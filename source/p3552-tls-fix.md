# Invisible Allocator Propagation for `std::execution::task`

Dietmar,

This describes a fix for frame allocator propagation in your
`task`. The mechanism is drawn from P4003R0 Section 5. It uses
thread-local storage as a write-through cache, applied to the
sender environment.

---

## 1. Your Current Approach

Your working implementation in `bemanproject/task` uses a
`defer_frame` wrapper to extract the allocator from the sender
environment before creating the coroutine frame:

```cpp
template <typename Task>
struct defer_frame
{
    Task task;
    template <typename... Arg>
    auto operator()(Arg&&... arg) const {
        return ex::let_value(
            ex::read_env(ex::get_allocator),
            [&task=this->task,
             ...a=std::forward<Arg>(arg)](auto alloc) {
                return std::invoke(
                    task, std::allocator_arg, alloc,
                    std::move(a)...);
            });
    }
};
```

The mechanism:

1. `read_env(get_allocator)` extracts the allocator from the
   receiver's environment
2. `let_value` passes it to a lambda that calls the coroutine
   factory with `std::allocator_arg`
3. The coroutine's `operator new` receives the allocator through
   the parameter list

This works for the immediate coroutine. It does not work for child
coroutines. When the coroutine body calls `co_await child_task()`,
the child's frame is allocated before any sender machinery can
intervene. The child uses global `operator new`. As you noted, the
example "does use a memory allocation using global operator new
from the [hidden] coroutine awaiting an awaiter."

`defer_frame` has a second limitation: it requires the user to
wrap every coroutine factory. The allocator does not propagate
through `co_await`. Each call site must participate.

---

## 2. Ergonomic Cost

With `defer_frame`, the user writes:

```cpp
static constexpr defer_frame t0(
    [](auto, auto, int i) -> ex::task<void> {
        std::cout << " i=" << i << "\n";
        auto a = co_await ex::read_env(ex::get_allocator);
        a.deallocate(a.allocate(1), 1);
    });

ex::sync_wait(
    ex::write_env(
        t0(17),
        ex::env{ex::prop{ex::get_allocator, alloc}}));
```

Every coroutine must be wrapped in `defer_frame`. Every call site
must pass the allocator explicitly. Child coroutines that the
wrapped coroutine calls via `co_await` do not receive the
allocator at all.

A call chain makes the cost visible:

```cpp
// What the user must write today
static constexpr defer_frame do_read(
    [](auto, auto, socket& s, buffer& buf) -> ex::task<void> {
        co_await async_read(s, buf);
    });

static constexpr defer_frame do_process(
    [](auto, auto, socket& s) -> ex::task<void> {
        buffer buf;
        co_await do_read(s, buf);      // must also be defer_frame
        process(buf);
    });

ex::sync_wait(
    ex::write_env(
        do_process(sock),
        ex::env{ex::prop{ex::get_allocator, alloc}}));
```

Compare with what the user should write:

```cpp
ex::task<void> do_read(socket& s, buffer& buf) {
    co_await async_read(s, buf);
}

ex::task<void> do_process(socket& s) {
    buffer buf;
    co_await do_read(s, buf);
    process(buf);
}

ex::sync_wait(
    ex::write_env(
        do_process(sock),
        ex::env{ex::prop{ex::get_allocator, alloc}}));
```

Same logic. No wrappers. No `allocator_arg`. No template
deduction at every call site. The allocator propagates invisibly.

P4003R0 Section 5.2 documents the same ergonomic problem with the
`allocator_arg_t` approach: the allocator becomes viral, infecting
every signature in the call chain. `defer_frame` moves the
infection from the parameter list to the call-site wrapper, but
the virus is the same.

---

## 3. The Fix

The fix has four parts. All changes are inside your `task`'s
`promise_type`.

### 3.1 TLS Accessors

A thread-local pointer serves as a write-through cache for the
current frame allocator. No dynamic initializer - the
thread-local is zero-initialized by the loader.

```cpp
namespace execution::detail {

inline std::pmr::memory_resource*&
frame_allocator_ref() noexcept {
    static thread_local std::pmr::memory_resource* mr = nullptr;
    return mr;
}

std::pmr::memory_resource*
get_frame_allocator() noexcept {
    return frame_allocator_ref();
}

void set_frame_allocator(
    std::pmr::memory_resource* mr) noexcept {
    frame_allocator_ref() = mr;
}

} // namespace execution::detail
```

### 3.2 `operator new` / `operator delete`

The promise reads TLS during frame allocation. The
`memory_resource*` used for allocation is stored at the end of the
frame so `operator delete` can recover it.

```cpp
// In task<T>::promise_type

static void* operator new(std::size_t size) {
    auto* mr = detail::get_frame_allocator();
    if (!mr)
        mr = std::pmr::new_delete_resource();

    auto total = size + sizeof(std::pmr::memory_resource*);
    void* raw = mr->allocate(total, alignof(std::max_align_t));
    std::memcpy(
        static_cast<char*>(raw) + size, &mr, sizeof(mr));
    return raw;
}

static void operator delete(void* ptr, std::size_t size) {
    std::pmr::memory_resource* mr;
    std::memcpy(
        &mr, static_cast<char*>(ptr) + size, sizeof(mr));
    auto total = size + sizeof(std::pmr::memory_resource*);
    mr->deallocate(ptr, total, alignof(std::max_align_t));
}
```

When no frame allocator has been established (TLS is null), the
fallback is `new_delete_resource` - identical behavior to a
coroutine with no custom `operator new`. See P4003R0 Section 5.3
for the rationale on why `new_delete_resource` is preferred over
`get_default_resource`.

### 3.3 Store the Allocator at `connect` Time

When `connect(task_sender, receiver)` is called, your promise has
access to the receiver's environment. Read the allocator and store
it:

```cpp
// In the task's operation state, during connect:

template <typename Receiver>
void connect_impl(Receiver&& rcvr) {
    auto alloc = get_allocator(get_env(rcvr));
    promise_.stored_allocator_ = alloc.resource();
}
```

The stored allocator is a `std::pmr::memory_resource*`. Type
erasure happens once, at connect time.

### 3.4 Restore TLS at Every Resume Point

Before the coroutine body executes - and before any child
coroutine can be called - TLS must hold the correct allocator.
Two resume points matter:

**`initial_suspend` resume.** When `start()` resumes the
coroutine after initial suspension, the promise restores TLS:

```cpp
struct initial_awaiter {
    promise_type* promise_;

    bool await_ready() noexcept { return false; }
    void await_suspend(coroutine_handle<>) noexcept {}

    void await_resume() noexcept {
        detail::set_frame_allocator(
            promise_->stored_allocator_);
    }
};

auto initial_suspend() noexcept {
    return initial_awaiter{this};
}
```

**`await_transform` resume.** Every `co_await` in the body passes
through `await_transform`. The returned awaiter restores TLS in
its `await_resume`:

```cpp
template <typename A>
auto await_transform(A&& a) {
    struct restoring_awaiter {
        // ... wraps the original awaiter ...

        decltype(auto) await_resume() {
            detail::set_frame_allocator(
                promise_->stored_allocator_);
            return inner_.await_resume();
        }
    };
    return restoring_awaiter{
        as_awaitable(
            affine_on(std::forward<A>(a), /*sched*/),
            *this),
        this};
}
```

After `await_resume` sets TLS, the coroutine body continues.
When it calls `child_task(args...)`, the child's `operator new`
reads TLS and allocates with the correct allocator. The child
never knows the allocator exists.

P4003R0 Section 5.4 ("The Window") analyzes why this is safe:
TLS is valid between a coroutine's resumption and its next
suspension point. `operator new` for a child coroutine executes
within this window.

---

## 4. The Result

Launch site - allocator established once:

```cpp
std::pmr::monotonic_buffer_resource pool;
std::pmr::polymorphic_allocator alloc(&pool);

ex::sync_wait(
    ex::write_env(
        my_server(sock),
        ex::env{ex::prop{ex::get_allocator, alloc}}));
```

Coroutine chain - no allocator awareness:

```cpp
ex::task<void> my_server(socket& sock) {
    for (;;) {
        auto conn = co_await accept(sock);
        co_await handle_connection(conn);
    }
}

ex::task<void> handle_connection(connection& conn) {
    auto req = co_await read_request(conn);
    auto resp = process(req);
    co_await write_response(conn, resp);
}

ex::task<response> read_request(connection& conn) {
    buffer buf;
    co_await async_read(conn, buf);
    co_return parse(buf);
}
```

Every `task` in the chain allocates its frame with the pool
allocator. No `defer_frame`. No `allocator_arg`. No template
parameters. The allocator propagates from the launch site through
TLS, restored at every resume point by the promise machinery.

---

## 5. Why TLS Is Safe Here

The likely objection: TLS does not exist on GPU devices, and
`std::execution` is designed for heterogeneous dispatch.

Coroutines are not supported in nvcc device code. The CUDA C++
Language Support table (v13.1, December 2025) lists coroutines as
**NOT SUPPORTED** for device code. Your `task` is a coroutine
type. It will never execute on a GPU.

TLS inside your `promise_type` costs nothing for the GPU path
because the GPU path does not use `task`. Sender algorithms that
execute on GPUs use struct receivers with void-returning
completions. They have no `promise_type`. They have no
`operator new`. TLS does not touch them.

The TLS mechanism is scoped to coroutine types. It does not
affect the sender protocol, the receiver protocol, or any sender
algorithm. It lives entirely inside the promise machinery of a
type that already requires a cooperative, single-threaded
execution window between suspension points.

P4003R0 Section 5 provides the full rationale, including the
window analysis, the per-chain scoping model, and the portability
considerations.
