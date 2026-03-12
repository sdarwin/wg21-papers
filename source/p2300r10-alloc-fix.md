# The Frame Allocator Is Not the Environment Allocator

Dietmar,

This builds on the previous document (p3552-tls-fix.md). The TLS
mechanism described there is correct but one design choice was
wrong: storing the frame allocator from `get_allocator(get_env(receiver))`
at connect time. The frame allocator should never come from the
environment.

---

## 1. Two Allocators, Two Purposes

The P2300 environment's `get_allocator` is a general-purpose
allocator. Any coroutine can query it:

```cpp
auto alloc = co_await ex::read_env(ex::get_allocator);
alloc.allocate(sizeof(MyObject));  // used for anything
```

This is the right design for general-purpose allocation. Users
put an allocator in the environment and any code in the chain can
use it for containers, strings, connection pools, whatever the
application needs.

A frame allocator is a different thing. Frame allocators exploit
a narrow pattern: coroutine frame sizes repeat, lifetimes nest,
and deallocation order mirrors allocation order. A recycling
frame allocator caches recently freed frames for immediate reuse.
P4003R0 Section 2.3 shows 3.1x speedup on MSVC over
`std::allocator`, and 1.28x over mimalloc.

These performance properties depend on the allocation pattern
being narrow. If arbitrary code can grab the frame allocator and
use it for a `std::vector` that grows to 10 MB, or a database
connection pool, or a JSON parser's scratch buffer, the pattern
is destroyed. The recycler's size classes no longer match. The
LIFO assumption breaks. The bounded pool overflows.

If the frame allocator is in the environment, this is exactly
what happens. Any coroutine in the chain can
`co_await read_env(get_allocator)` and use it for anything. The
environment does not distinguish between "allocator for frames"
and "allocator for everything else." There is one query and one
answer.

The frame allocator must be a separate channel:

- Not queryable from user code
- Not in the environment
- Read only by `promise_type::operator new`
- Propagated through TLS, restored at resume points

The environment's `get_allocator` remains what it is: a
general-purpose allocator for application use.

---

## 2. `with_frame_allocator`

The previous document's Section 3.3 stored the frame allocator
from the environment at connect time. Replace that. The frame
allocator is established at the launch site through TLS and
recovered from the frame itself, never from the environment.

### 2.1 The Algorithm

```cpp
auto with_frame_allocator(std::pmr::memory_resource* mr) {
    detail::set_frame_allocator(mr);
    return [](auto task) {
        detail::set_frame_allocator(nullptr);
        return std::move(task);
    };
}
```

Usage:

```cpp
with_frame_allocator(&pool)(my_server(sock))
```

### 2.2 Why This Is Safe

C++17 [expr.call] p5: "The postfix-expression is sequenced
before each expression in the expression-list."

In `with_frame_allocator(&pool)(my_server(sock))`:

1. `with_frame_allocator(&pool)` executes first - sets TLS,
   returns a callable
2. `my_server(sock)` executes second - `operator new` reads
   TLS, frame allocated with the pool
3. The callable is invoked with the task - clears TLS, returns
   the task as-is

TLS is set and cleared within a single expression. No guard to
misuse. No way to forget to clear it.

### 2.3 The Change from the Previous Document

The previous document's Section 3.3 said: at connect time, read
`get_allocator(get_env(receiver))` and store it in the promise.
This is replaced.

The promise recovers the frame allocator from its own frame. The
`operator new` in Section 3.2 of the previous document already
stashes the `memory_resource*` at the end of the frame. The
promise reads it back:

```cpp
std::pmr::memory_resource* recover_frame_allocator() noexcept {
    auto* self = coroutine_handle<promise_type>::from_promise(*this)
        .address();
    std::pmr::memory_resource* mr;
    std::memcpy(&mr,
        static_cast<char*>(self) + frame_size_,
        sizeof(mr));
    return mr;
}
```

This is the value that the resume-point restoration (Section 3.4
of the previous document) writes to TLS. It comes from the frame,
not from the environment.

The environment is not involved. `get_allocator` in the
environment is untouched. The two allocators are independent.

---

## 3. What the User Sees

Frame allocator only:

```cpp
std::pmr::monotonic_buffer_resource pool;

ex::sync_wait(
    with_frame_allocator(&pool)(my_server(sock)));
```

Frame allocator plus general-purpose environment allocator:

```cpp
std::pmr::monotonic_buffer_resource frame_pool;
std::pmr::polymorphic_allocator general_alloc(&app_pool);

ex::sync_wait(
    ex::write_env(
        with_frame_allocator(&frame_pool)(my_server(sock)),
        ex::env{ex::prop{ex::get_allocator, general_alloc}}));
```

Two allocators. Two channels. Independent. The coroutine chain
is unaware of both:

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
```

Every frame in the chain uses the frame pool. If any coroutine
needs the general-purpose allocator for application logic, it
queries the environment:

```cpp
auto alloc = co_await ex::read_env(ex::get_allocator);
std::pmr::vector<char> buf(alloc);
```

The frame allocator is not reachable from this query. It cannot
be misused.

---

## 4. Swappable Implementations

Because `with_frame_allocator` takes a `memory_resource*`, the
user is never locked into a particular frame allocator
implementation. A recycling allocator with size-class buckets is
optimal for coroutine frames - Capy ships one:

https://github.com/cppalliance/capy/blob/18c30d2197ac8804f9426005576d4f9b80a76135/include/boost/capy/ex/recycling_memory_resource.hpp

It uses power-of-two size classes (64 to 2048 bytes), a
thread-local pool for lock-free fast-path allocation, and a
global pool with a mutex for cross-thread block sharing.
Allocations above 2048 bytes bypass the pools. This is the kind
of allocator that exploits the narrow frame allocation pattern -
and the kind that breaks if arbitrary non-frame allocations
pollute it.

But the user can swap in any `memory_resource*` they want:

```cpp
// recycling allocator (production)
with_frame_allocator(get_recycling_memory_resource())(my_server(sock))

// monotonic buffer (testing, bounded memory)
std::pmr::monotonic_buffer_resource buf;
with_frame_allocator(&buf)(my_server(sock))

// tracking allocator (debugging)
tracking_memory_resource tracker;
with_frame_allocator(&tracker)(my_server(sock))
```

One call site, any strategy. The coroutine chain does not change.

---

## 5. Extensibility

How third-party task authors participate in frame allocator
propagation is your design space. The mechanism is in
`promise_type::operator new` - any task type that reads TLS in
its `operator new` and restores TLS at resume points
participates. You can expose the TLS accessors, define a
concept, or leave it as implementation detail.

P4003R0 Section 5 shows one approach. It is not the only one.
