---
title: "The Sender Sub-Language"
document: D4014R2
date: 2026-03-28
reply-to:
  - "Vinnie Falco <vinnie.falco@gmail.com>"
  - "Mungo Gill <mungo.gill@me.com>"
audience: LEWG
---

## Abstract

C++26 ships thirty sender algorithms that collectively replace sequential statements, local variables, error handling, and iteration with library-level equivalents rooted in continuation-passing style.

This paper is a progressive tutorial. It introduces every sender algorithm in the C++26 working draft ([N5014](https://www.open-std.org/jtc1/sc22/wg21/docs/papers/2025/n5014.pdf))<sup>[1]</sup>, one at a time, from the simplest to the most complex. Each algorithm is defined, demonstrated in a working example, and explained. After the explanation, the equivalent C++ program appears without commentary. The theoretical foundations are presented first - the lambda calculus, continuation-passing style, and monadic composition that give the Sub-Language its structure - followed by the algorithms themselves, grouped by function and ordered by escalating complexity. The final sections cover the `task` coroutine type ([P3552R3](https://wg21.link/p3552r3))<sup>[2]</sup> and the composition patterns that emerge when senders and coroutines interleave.

The evidence is public. The conclusions are the reader's.

---

## Revision History

### R2: March 2026

- Complete rewrite as a progressive tutorial.
- Covers all thirty sender algorithms in the C++26 working draft.
- Adds coverage of `task` and sender-coroutine composition.
- Replaces the advocacy framing of R0/R1 with an inform-paper structure.

### R1: March 2026 (pre-Croydon mailing)

- Added trade-off analysis, nvexec precedent, concurrent selection gap, and timeline.

### R0: March 2026 (pre-Croydon mailing)

- Initial version.

---

## 1. Disclosure

The author maintains [Boost.Beast](https://github.com/boostorg/beast)<sup>[3]</sup>, [Corosio](https://github.com/cppalliance/corosio)<sup>[4]</sup>, and [Capy](https://github.com/cppalliance/capy)<sup>[5]</sup>. The author believes coroutine-native I/O is the correct foundation for networking in C++. A coroutine-native design cannot express compile-time work graphs. That is a genuine limitation, and domains that need compile-time work graphs - GPU dispatch, heterogeneous execution, high-frequency trading - are right to use a model that provides them.

`std::execution` ([P2300R10](https://wg21.link/p2300r10))<sup>[6]</sup> provides compile-time sender composition, structured concurrency guarantees, zero-allocation pipelines in steady state, and a customization point model that enables heterogeneous dispatch across execution contexts. The P2300 authors built a framework grounded in four decades of programming language research - the lambda calculus, continuation-passing style, monadic composition, and algebraic effect channels - and delivered it as a working C++ library that ships in production at NVIDIA and Citadel Securities<sup>[7]</sup>. The [stdexec](https://github.com/NVIDIA/stdexec)<sup>[8]</sup> reference implementation demonstrates performance on par with hand-written CUDA<sup>[9]</sup>. The engineering depth is real. The theoretical foundations are sound. The achievement deserves the recognition it has earned.

This paper is a tutorial. It shows how each of the thirty sender algorithms in C++26 works, what each one is for, and what the equivalent C++ program looks like. The author provides information, asks nothing, and serves at the pleasure of the chair.

The author asks for nothing.

---

## 2. Theoretical Foundations

The Sender Sub-Language is not merely a fluent API. It is continuation-passing style expressed as composable value types, drawing on techniques refined across four decades of programming language research. Understanding where the pieces come from makes the tutorial that follows easier to absorb.

### 2.1 Continuations and CPS

The [Lambda Papers](https://en.wikisource.org/wiki/Lambda_Papers)<sup>[10]</sup> (Steele and Sussman, 1975-1980) formalized continuation-passing style: every function receives an explicit continuation representing "what happens next." Instead of returning a value to its caller, a function passes its result forward into the continuation. The sender protocol works the same way. `connect(sndr, rcvr)` reifies the continuation - it binds a sender to its receiver, producing an operation state that holds the entire work graph as a concrete type. `start(op)` evaluates it.

Gordon Plotkin's 1975 paper on [call-by-value lambda calculus](https://doi.org/10.1016/0304-3975(75)90017-1)<sup>[11]</sup> established that CPS makes evaluation order explicit in the term structure. This is why optimizing compilers - [SML/NJ](https://www.smlnj.org/)<sup>[12]</sup>, [GHC](https://www.haskell.org/ghc/)<sup>[13]</sup>, [Chicken Scheme](https://www.call-cc.org/)<sup>[14]</sup> - use CPS as their intermediate representation, and why the Sender Sub-Language can build zero-allocation pipelines and compile-time work graphs.

### 2.2 Monads

Eugenio Moggi's 1991 paper ["Notions of Computation and Monads"](https://doi.org/10.1016/0890-5401(91)90052-4)<sup>[15]</sup> showed that monads structure computation with effects. Two operations are sufficient: `return` (lift a value into the computational context) and `bind` (sequence one computation into the next). In the Sender Sub-Language, `just(x)` is monadic return and `let_value(f)` is monadic bind. `then(f)` is the functor lift - `fmap` - a specialization where the function returns a plain value rather than a new sender.

### 2.3 Delimited Continuations and Algebraic Effects

Olivier Danvy and Andrzej Filinski's 1990 paper ["Abstracting Control"](https://doi.org/10.1145/91556.91622)<sup>[16]</sup> introduced delimited continuations and the shift/reset operators, enabling multiple effect channels within a single computation. The sender three-channel model - `set_value`, `set_error`, `set_stopped` - is a fixed algebraic effect system. Each channel carries a distinct semantic role, and the composition algebra dispatches on which channel a result arrives on.

Timothy Griffin's 1990 paper ["A Formulae-as-Types Notion of Control"](https://doi.org/10.1145/96709.96714)<sup>[17]</sup> connected control operators to type systems. The sender completion signatures - the type-level declaration of which channels a sender may complete on and with what argument types - are this connection made concrete in C++.

### 2.4 The Mapping

| Sender concept                            | Theoretical origin        |
| ----------------------------------------- | ------------------------- |
| `just(x)`                                 | Monadic return / `pure`   |
| `let_value(f)`                            | Monadic bind (`>>=`)      |
| `then(f)`                                 | Functor `fmap`            |
| `set_value` / `set_error` / `set_stopped` | Algebraic effect channels |
| `connect(sndr, rcvr)`                     | CPS reification           |
| `start(op)`                               | CPS evaluation            |
| Completion signatures                     | Type-level sum types      |

The names are not arbitrary. The P2300 authors chose them with care, and the choices reflect genuine scholarship. `just` echoes Haskell's `Just`. `let_value` mirrors monadic bind. The three completion channels form a closed algebraic effect system. The framework is grounded in real theory, and the theory is worth knowing before the tutorial begins.

---

## 3. Value Lifting

The simplest sender algorithms lift values into the sender context. They create senders from nothing - no scheduler, no I/O, no computation. The value is already known.

### 3.1 `just`

`just(vs...)` creates a sender that completes immediately with the given values on the value channel.

```cpp
auto sndr = just(42);
```

This is monadic return - the `pure` operation that lifts a value into the sender context. The sender completes synchronously, inline, with no allocation and no scheduling. It is the entry point into every pipeline.

```cpp
int v = 42;
```

### 3.2 `just_error`

`just_error(e)` creates a sender that completes immediately with the given error on the error channel.

```cpp
auto sndr = just_error(
    make_error_code(errc::invalid_argument));
```

`just_error` lifts an error into the sender context the way `just` lifts a value. The error is delivered to the first error handler downstream - an `upon_error` or `let_error` in the pipeline.

```cpp
throw system_error(
    make_error_code(errc::invalid_argument));
```

### 3.3 `just_stopped`

`just_stopped()` creates a sender that completes immediately on the stopped channel. No value is produced and no error is reported. The operation was cancelled.

```cpp
auto sndr = just_stopped();
```

The stopped channel is the third path in the sender completion algebra - distinct from both success and failure. It represents cancellation as a first-class concept. Regular C++ has no dedicated cancellation channel; the three-channel model is one of the properties the Sub-Language adds.

---

## 4. Consuming a Pipeline

### 4.1 `sync_wait`

`sync_wait(sndr)` blocks the current thread until the sender completes, then returns the result as an `optional<tuple<...>>`.

```cpp
auto sndr = just(6, 7)
          | then([](int a, int b) {
                return a * b;
            });
auto [result] =
    sync_wait(std::move(sndr)).value();
// result == 42
```

`sync_wait` is the bridge between the Sender Sub-Language and regular C++. It is the only sender consumer in the standard - the single point where a sender pipeline produces a value that ordinary code can hold. Everything upstream of `sync_wait` is lazy. Nothing executes until `sync_wait` drives the pipeline to completion.

```cpp
int result = 6 * 7;
```

---

## 5. Transformation

The transformation algorithms apply a function to a sender's completion and produce a new completion. The function returns a value, not a sender.

### 5.1 `then`

`then(f)` applies `f` to the values produced by the predecessor sender. The return value of `f` becomes the new value completion.

```cpp
auto sndr = just(3, 4)
          | then([](int a, int b) {
                return a * a + b * b;
            });
// completes with set_value(25)
```

`then` is functor `fmap`. It transforms values without changing the structure of the pipeline - the sender still completes on the value channel, with a different value. Errors and stopped signals pass through untouched.

```cpp
int result = 3 * 3 + 4 * 4;
```

### 5.2 `upon_error`

`upon_error(f)` applies `f` to the error produced by the predecessor sender. The return value of `f` becomes a value completion - the error is recovered.

```cpp
auto sndr = just_error(
        make_error_code(errc::connection_reset))
      | upon_error([](error_code ec) {
            return -1;
        });
// completes with set_value(-1)
```

`upon_error` is the error-channel counterpart of `then`. Where `then` transforms values, `upon_error` transforms errors into values - the result crosses channels, converting an error completion into a value completion. Values and stopped signals pass through untouched.

```cpp
int result;
try {
    throw system_error(
        make_error_code(
            errc::connection_reset));
} catch (system_error const&) {
    result = -1;
}
```

### 5.3 `upon_stopped`

`upon_stopped(f)` applies `f` when the predecessor sender is cancelled. The return value of `f` becomes a value completion.

```cpp
auto sndr = just_stopped()
          | upon_stopped([] {
                return 0;
            });
// completes with set_value(0)
```

`upon_stopped` converts a cancellation into a value. It is used when a pipeline needs a fallback result in the event of cancellation rather than propagating the stopped signal.

---

## 6. Monadic Composition

The `let_*` family is the complexity step. Where `then` applies a function that returns a value, `let_value` applies a function that returns a sender. The function constructs the next stage of the pipeline at runtime. This is monadic bind.

### 6.1 `let_value`

`let_value(f)` applies `f` to the values produced by the predecessor. `f` returns a new sender, and that sender's completion becomes the completion of the composed pipeline.

```cpp
auto sndr = just(std::string("hello"))
          | let_value([](std::string s) {
                return just(s + " world")
                     | then([](std::string r) {
                           return r.size();
                       });
            });
// completes with set_value(11)
```

`let_value` is monadic bind. The function receives the predecessor's values and returns an entirely new sender - which may itself be a multi-stage pipeline. The result is a sender whose completion type is determined by the inner sender, not by the function's return type directly. This is what makes `let_value` more powerful than `then`: the next stage of computation is itself a sender. The pattern is straightforward once the distinction between returning a value and returning a sender is clear.

```cpp
std::string s = "hello";
s += " world";
size_t result = s.size();
```

### 6.2 `let_error`

`let_error(f)` applies `f` to the error produced by the predecessor. `f` returns a new sender. The error is recovered by routing into a new computation.

```cpp
auto sndr = just_error(
        make_error_code(errc::timed_out))
      | let_error([](error_code ec) {
            return just(ec.message());
        });
// completes with set_value("timed out")
```

`let_error` is the error-channel counterpart of `let_value`. Where `upon_error` transforms an error into a value, `let_error` transforms an error into a sender - enabling recovery paths that are themselves multi-stage pipelines.

```cpp
std::string result;
try {
    throw system_error(
        make_error_code(errc::timed_out));
} catch (system_error const& e) {
    result = e.code().message();
}
```

### 6.3 `let_stopped`

`let_stopped(f)` applies `f` when the predecessor is cancelled. `f` returns a new sender that replaces the cancellation.

```cpp
auto sndr = just_stopped()
          | let_stopped([] {
                return just(
                    std::string("recovered"));
            });
// completes with set_value("recovered")
```

`let_stopped` converts a cancellation into a new computation. The function receives no arguments - the stopped channel carries no data - and returns a sender whose completion replaces the stopped signal. The reader will recognize the pattern: each `let_*` variant handles one channel and routes its result into a new sender.

---

## 7. Execution Contexts

The execution context algorithms control where work runs. A scheduler represents an execution resource - a thread pool, an I/O context, a GPU stream - and these algorithms move work between them.

### 7.1 `schedule`

`schedule(sch)` produces a sender that, when started, completes on the specified scheduler's execution context.

```cpp
auto sndr = schedule(pool.get_scheduler())
          | then([] {
                return handle_request();
            });
```

`schedule` creates a sender from a scheduler. The sender completes with no values - it serves purely as a transition point. Work piped after `schedule` runs on the scheduler's context. This is how a pipeline enters a specific execution resource.

```cpp
pool.post([&] {
    handle_request();
});
```

### 7.2 `starts_on`

`starts_on(sch, sndr)` arranges for the given sender to begin execution on the specified scheduler.

```cpp
auto sndr = starts_on(
    pool.get_scheduler(),
    just(request) | then([](auto req) {
        return handle(req);
    }));
```

`starts_on` takes an existing sender and ensures it starts on a particular context. The sender's entire execution begins on the given scheduler.

```cpp
pool.post([&] {
    auto result = handle(request);
});
```

### 7.3 `continues_on`

`continues_on(sch)` transitions execution to a different scheduler after the predecessor completes.

```cpp
auto sndr =
    starts_on(io_ctx.get_scheduler(),
              async_read(socket, buf))
  | continues_on(pool.get_scheduler())
  | then([](auto data) {
        return parse(data);
    });
```

`continues_on` is the explicit context transition. When the predecessor completes, execution resumes on the specified scheduler. This is how a pipeline moves work between contexts - read on the I/O thread, parse on the thread pool.

```cpp
auto data = read(socket);
pool.post([&] {
    auto result = parse(data);
});
```

### 7.4 `on`

`on(sch, sndr)` runs the sender on the specified scheduler, then returns execution to the caller's context.

```cpp
auto sndr = on(
    pool.get_scheduler(),
    just(request) | then([](auto req) {
        return handle(req);
    }));
```

`on` is a round-trip. It transitions to the given scheduler, runs the sender, and transitions back. The caller does not need to track which context the work ran on - the result arrives on the original context.

```cpp
auto result = handle(request);
```

### 7.5 `affine_on`

`affine_on(sndr, sch)` adapts a sender to complete on the specified scheduler, skipping the transition if the sender already completes there.

```cpp
auto sndr = some_operation()
          | affine_on(pool.get_scheduler());
```

`affine_on` was introduced by [P3552R3](https://wg21.link/p3552r3)<sup>[2]</sup> as the scheduler affinity primitive. It behaves like `continues_on` but can avoid the scheduling overhead when the predecessor already completes on the target scheduler. This optimization is what makes scheduler affinity practical for coroutine `task` - the `await_transform` injects `affine_on` around every `co_await`ed sender, and when the sender completes on the correct scheduler, no rescheduling occurs. We will return to this in Section 12.

```cpp
auto result = some_operation();
```

### 7.6 `schedule_from`

`schedule_from(sch, sndr)` ensures that the sender's completion signals arrive on the specified scheduler.

```cpp
auto sndr = schedule_from(
    pool.get_scheduler(),
    async_read(socket, buf));
```

`schedule_from` is an implementer-facing algorithm. It guarantees that the completion - whether value, error, or stopped - is delivered on the given scheduler. Where `continues_on` transitions the value channel, `schedule_from` transitions all three channels. Most programmers will use `continues_on` or `on`; `schedule_from` exists for algorithm authors who need precise control over completion context.

```cpp
auto data = read(socket);
```

---

## 8. Environment

Senders execute within an environment - a queryable set of key-value pairs carried by the receiver. The environment provides context that the pipeline's algorithms can read: which scheduler to use, which allocator to use, whether cancellation has been requested. The following algorithms manipulate that environment.

### 8.1 `read_env`

`read_env(query)` creates a sender that reads a value from the receiver's environment and completes with it on the value channel.

```cpp
auto sndr = read_env(get_scheduler)
          | let_value([](auto sch) {
                return schedule(sch)
                     | then([] {
                           return load_config();
                       });
            });
```

`read_env` is the introspection primitive. The sender does not carry the environment value itself - it reads the value from the receiver at connection time, when the environment is known. This enables context-dependent behavior: the same pipeline can behave differently depending on which scheduler, allocator, or stop token the enclosing context provides. The composition reads naturally once the receiver's role as environment carrier is understood.

```cpp
auto config = load_config();
```

### 8.2 `write_env`

`write_env(env, sndr)` wraps a sender so that downstream receivers see additional or overridden environment entries.

```cpp
auto sndr = write_env(
    make_env(get_allocator, pool_alloc),
    process_file(path));
```

`write_env` overlays entries onto the receiver's environment. The wrapped sender and all of its children see the modified environment. This is how a pipeline provides a custom allocator, overrides a scheduler, or injects application-specific context without threading parameters through every function signature.

```cpp
auto result = process_file(path);
```

### 8.3 `unstoppable`

`unstoppable(sndr)` wraps a sender so that it does not receive stop requests from the parent context.

```cpp
auto sndr = unstoppable(
    commit_transaction(db, txn));
```

`unstoppable` shields a sender from cancellation. The wrapped sender runs to completion regardless of whether the parent pipeline has been stopped. This is used for operations that must not be interrupted - transaction commits, resource cleanup, finalization steps.

```cpp
commit_transaction(db, txn);
```

---

## 9. Structured Concurrency

The structured concurrency algorithms express fork-join parallelism and sender sharing. This is where the Sender Sub-Language provides something C++ statements alone do not: concurrent execution with structured lifetime guarantees.

### 9.1 `when_all`

`when_all(sndrs...)` starts all of the given senders concurrently and completes when every sender has completed. The value completions are concatenated.

```cpp
auto sndr = when_all(
    fetch_user(user_id),
    fetch_orders(user_id),
    fetch_preferences(user_id))
  | then([](auto user, auto orders,
            auto prefs) {
        return build_profile(
            user, orders, prefs);
    });
```

`when_all` is genuine structured concurrency. All child senders start together, run concurrently, and the join point is the `when_all` sender's completion. If any child completes with an error or is stopped, the remaining children are cancelled. The lifetime guarantee is structural - no child outlives the join point. Once the programmer has internalized the pattern, the intent is clear: three fetches, run concurrently, results combined.

```cpp
auto user = fetch_user(user_id);
auto orders = fetch_orders(user_id);
auto prefs = fetch_preferences(user_id);
auto profile =
    build_profile(user, orders, prefs);
```

### 9.2 `when_all_with_variant`

`when_all_with_variant(sndrs...)` is semantically equivalent to `when_all(into_variant(sndrs)...)`. It allows child senders with different value completion signatures.

```cpp
auto sndr = when_all_with_variant(
    fetch_json(url_a),
    fetch_binary(url_b))
  | then([](auto json_var, auto bin_var) {
        return combine(json_var, bin_var);
    });
```

`when_all` requires all children to have the same value completion signature. `when_all_with_variant` relaxes this constraint by wrapping each child's value completion in a `variant`. The result types are heterogeneous, and the programmer destructures the variants at the join point.

```cpp
auto json = fetch_json(url_a);
auto binary = fetch_binary(url_b);
auto result = combine(json, binary);
```

### 9.3 `split`

`split(sndr)` converts a single-shot sender into a multi-shot sender. The result is shared: multiple downstream pipelines can consume the same sender's completion.

```cpp
auto shared = split(fetch_data(url));
auto sndr = when_all(
    shared | then([](auto data) {
        return analyze(data);
    }),
    shared | then([](auto data) {
        return archive(data);
    }));
```

`split` allocates shared state - the sender's result is stored once and distributed to all consumers. This is the sender equivalent of binding a value to a local variable and using it in multiple expressions. The allocation is the cost; the sharing is the benefit.

```cpp
auto data = fetch_data(url);
auto analysis = analyze(data);
auto archived = archive(data);
```

---

## 10. Signal Adaptation and Data Parallelism

The signal adaptation algorithms reshape completion signatures - converting between channels, wrapping values in standard library types, and collapsing multiple completion paths into one. The data parallelism algorithms fan work out across an index space. Together, they handle the batch-processing and data-transformation patterns that arise in database operations, ETL pipelines, and scientific computing.

### 10.1 `into_variant`

`into_variant` adapts a sender with multiple value completion signatures into one that completes with a single `variant` of `tuple`s.

```cpp
auto sndr = query_database(stmt)
          | into_variant;
// if the query can complete with either
// set_value(row) or set_value(int),
// now completes with
// set_value(variant<tuple<row>,
//                   tuple<int>>)
```

`into_variant` collapses heterogeneous value completions into a single variant type. The result is a sender with exactly one value completion signature. This is used when a sender's multiple value paths must be unified - for example, before passing to `when_all`, which requires a single value completion signature from each child.

```cpp
auto result = query_database(stmt);
```

### 10.2 `stopped_as_optional`

`stopped_as_optional` converts a sender's value completion into an `optional` and its stopped completion into an empty `optional`.

```cpp
auto sndr = dequeue(work_queue)
          | stopped_as_optional;
// value(item)   -> value(optional(item))
// stopped()     -> value(optional())
```

`stopped_as_optional` is the bridge between the stopped channel and ordinary value processing. A queue that reports "closed" via `set_stopped` becomes a sender that completes with an empty `optional` - the programmer handles both cases in the value channel with a single `if`.

```cpp
auto item = dequeue(work_queue);
// returns optional<item_type>
```

### 10.3 `stopped_as_error`

`stopped_as_error(e)` converts a sender's stopped completion into an error completion carrying `e`.

```cpp
auto sndr = timed_operation(deadline)
          | stopped_as_error(
                make_error_code(
                    errc::timed_out));
```

`stopped_as_error` reclassifies cancellation as an error. The stopped channel is eliminated from the completion signatures and replaced by an error. This is used when the downstream pipeline handles errors but not cancellation - the reclassification unifies the two failure paths.

```cpp
auto result = timed_operation(deadline);
if (timed_out)
    throw system_error(
        make_error_code(errc::timed_out));
```

### 10.4 `bulk`

`bulk(policy, shape, f)` invokes `f(i, args...)` for each index `i` in the range `[0, shape)`, where `args` are the values produced by the predecessor sender.

```cpp
auto sndr = just(records)
          | bulk(std::par, records.size(),
                [](size_t i, auto& recs) {
                    recs[i].score =
                        compute_score(recs[i]);
                });
```

`bulk` is the data-parallel primitive. The execution policy (`std::par`, `std::seq`) controls whether the invocations may run concurrently. The `parallel_scheduler` provides a customized implementation that executes the index space across the system thread pool. The design is remarkably expressive - a single algorithm covers sequential iteration, parallel map, and GPU-dispatched computation depending on the scheduler and policy.

```cpp
std::for_each(std::execution::par,
    records.begin(), records.end(),
    [](auto& r) {
        r.score = compute_score(r);
    });
```

### 10.5 `bulk_chunked`

`bulk_chunked(policy, shape, f)` partitions the index space into chunks and invokes `f` once per chunk with the chunk's index range.

```cpp
auto sndr = just(data)
          | bulk_chunked(std::par,
                data.size(),
                [](auto chunk, auto& d) {
                    for (auto i : chunk)
                        d[i] = transform(d[i]);
                });
```

`bulk_chunked` gives the implementation control over how the index space is divided. The `parallel_scheduler` uses this to distribute work across threads in chunks sized for the hardware. The callable receives a range of indices rather than a single index, enabling vectorization and cache-friendly access patterns within each chunk.

```cpp
std::for_each(std::execution::par,
    data.begin(), data.end(),
    [](auto& x) { x = transform(x); });
```

### 10.6 `bulk_unchunked`

`bulk_unchunked(policy, shape, f)` invokes `f` once per index with no chunking guarantees.

```cpp
auto sndr = just(pixels)
          | bulk_unchunked(std::par,
                pixels.size(),
                [](size_t i, auto& px) {
                    px[i] = apply_filter(px[i]);
                });
```

`bulk_unchunked` is the unchunked counterpart. The callable receives one index at a time. The implementation may still batch invocations internally, but the callable's interface is per-index. Each piece of the `bulk` family fits together with the precision one expects from a well-engineered system: `bulk` dispatches to `bulk_chunked` by default, which the scheduler can further specialize.

```cpp
std::for_each(std::execution::par,
    pixels.begin(), pixels.end(),
    [](auto& p) { p = apply_filter(p); });
```

---

## 11. Async Scopes

The async scope algorithms manage the lifetime of dynamically spawned work. A `counting_scope` or `simple_counting_scope` tracks outstanding operations and provides a join point where the caller waits for all spawned work to complete. The scope token is the handle through which senders are associated with a scope.

### 11.1 `associate`

`associate(token, sndr)` ties a sender's lifetime to a scope. The scope will not complete its join until the associated sender has completed.

```cpp
counting_scope scope;
auto token = scope.get_token();

for (auto& conn : accepted_connections) {
    auto sndr = associate(token,
        starts_on(pool.get_scheduler(),
                  handle_connection(conn)));
    // sndr is fire-and-forget within the scope
}
// scope.join() waits for all connections
```

`associate` is the structured spawn. The sender runs independently - it is not piped into a continuation - but its lifetime is bounded by the scope. When the scope joins, all associated senders must have completed. This is how a server manages connection lifetimes: each connection is associated with the scope, and shutdown waits for all connections to drain.

```cpp
for (auto& conn : accepted_connections)
    pool.post([&] {
        handle_connection(conn);
    });
pool.join();
```

### 11.2 `spawn_future`

`spawn_future(token, sndr)` spawns a sender into a scope and returns a new sender that completes with the spawned sender's result.

```cpp
auto sndr = spawn_future(token,
    starts_on(pool.get_scheduler(),
              fetch_data(key)))
  | then([](auto data) {
        return process(data);
    });
```

`spawn_future` is `associate` with a return channel. The spawned sender begins executing immediately, and the returned sender completes with the result when it is ready. This is the sender equivalent of `std::async` - fire off work, get a handle to the result, consume it later.

```cpp
auto future = std::async(
    std::launch::async,
    [&] { return fetch_data(key); });
auto result = process(future.get());
```

---

## 12. The `task` Coroutine Type

[P3552R3](https://wg21.link/p3552r3)<sup>[2]</sup> adds `execution::task<T, C>` to C++26 - a coroutine type that is also a sender. The `task` is the bridge between coroutine-style `co_await` and the sender pipeline model. The integration between the two worlds is seamless: a `task` can `co_await` any sender, and a `task` can be used as a sender in any pipeline.

### 12.1 `task<T>`

`task<T>` declares a coroutine that produces a value of type `T` on the value channel when it completes.

```cpp
task<int> the_answer() {
    co_return 42;
}

int main() {
    auto [v] =
        sync_wait(the_answer()).value();
    // v == 42
}
```

A `task` is lazy - the coroutine body does not execute until the task is connected to a receiver and started. It is a sender: it can be piped, composed, passed to `sync_wait`, or used as a child of `when_all`. The completion signatures are `set_value_t(T)`, `set_error_t(exception_ptr)`, and `set_stopped_t()`.

### 12.2 `co_await` a Sender

Inside a `task`, any sender can be `co_await`ed. The sender is connected to an internal receiver, started, and the result is delivered as the value of the `co_await` expression.

```cpp
task<int> add_one() {
    int v = co_await just(41);
    co_return v + 1;
}
```

When the sender completes with `set_value(args...)`, the `co_await` expression produces the values. A single value is returned directly. Multiple values are returned as a `tuple`. A sender with no value arguments produces `void`. If the sender completes with `set_error`, the error is thrown as an exception. If it completes with `set_stopped`, the coroutine is cancelled without resuming.

### 12.3 `task_scheduler`

`task_scheduler` is a type-erased scheduler used by `task` for scheduler affinity. When a `task` is connected to a receiver, the scheduler is obtained from the receiver's environment via `get_scheduler(get_env(rcvr))` and stored in the `task_scheduler`.

```cpp
task<void> work() {
    co_await async_read(socket, buf);
    // resumes on the task's scheduler
    process(buf);
}

sync_wait(
    starts_on(pool.get_scheduler(),
              work()));
```

The `task_scheduler` uses small-object optimization to avoid allocation for common scheduler types. The type erasure is the cost of not knowing the scheduler type at coroutine definition time. The programmer has everything they need: the scheduler is obtained automatically, the affinity is maintained transparently, and the type erasure overhead is minimal.

### 12.4 `inline_scheduler`

`inline_scheduler` completes immediately on the calling thread. Using it as the task's scheduler type disables scheduler affinity.

```cpp
struct no_affinity {
    using scheduler_type = inline_scheduler;
};

task<void, no_affinity> fast_path() {
    co_await async_op();
    // no rescheduling - resumes wherever
    // the sender completed
}
```

Disabling affinity removes the rescheduling overhead at the cost of the guarantees affinity provides. The programmer who understands the implications is free to make this choice.

### 12.5 Scheduler Affinity

The `task`'s promise type injects `affine_on` around every `co_await`ed sender via `await_transform`. The effect: after each `co_await`, execution resumes on the task's scheduler regardless of where the sender completed.

```cpp
task<void> affine_demo() {
    // running on pool's scheduler
    co_await async_read(socket, buf);
    // still on pool's scheduler
    auto result = parse(buf);
    co_await async_write(socket, result);
    // still on pool's scheduler
}
```

Scheduler affinity means the programmer can reason about execution context the same way they reason about synchronous code - each line runs on the same context as the line before it. The `affine_on` insertion is invisible. The rescheduling happens only when necessary - if the sender already completes on the correct scheduler, `affine_on` skips the transition.

### 12.6 Allocator Support

The context parameter `C` configures allocator support. The allocator type is declared via `C::allocator_type` and is used for the coroutine frame allocation.

```cpp
struct alloc_ctx {
    using allocator_type =
        pmr::polymorphic_allocator<byte>;
};

task<int, alloc_ctx> allocated_work(
        allocator_arg_t,
        pmr::polymorphic_allocator<byte>,
        int input) {
    co_return input * 2;
}

pmr::monotonic_buffer_resource pool;
auto sndr = allocated_work(
    allocator_arg,
    pmr::polymorphic_allocator<byte>{&pool},
    21);
```

The allocator is available to child operations through the receiver's environment via `get_allocator`. The design supports environments where heap allocation is prohibited - the same context parameter that controls the scheduler type controls the allocator type.

### 12.7 `co_yield with_error(e)`

`co_yield with_error(e)` reports an error on the error channel without throwing an exception.

```cpp
task<void> validate(request const& req) {
    if (!req.valid())
        co_yield with_error(
            make_error_code(
                errc::invalid_argument));
    co_return;
}
```

This is how a `task` delivers a typed error to the sender composition algebra without relying on exceptions. The coroutine is suspended and completes with `set_error(e)`. The downstream pipeline handles the error through `upon_error` or `let_error`. This is a feature of the design: errors can be reported precisely, with the type preserved, and the composition algebra participates.

### 12.8 Cancellation

`co_await just_stopped()` cancels the coroutine. The coroutine completes with `set_stopped()`.

```cpp
task<void> check_cancel() {
    auto token = co_await
        read_env(get_stop_token);
    if (token.stop_requested())
        co_await just_stopped();
    co_return;
}
```

Any sender that completes with `set_stopped` cancels the `task`. The coroutine is never resumed - local variables are destroyed and the task completes on the stopped channel. The `task`'s stop token is linked to the parent context's stop token, so cancellation propagates structurally.

### 12.9 Error Channel Mapping

When a `task` `co_await`s a sender that completes with `set_error(ec)`, the error is delivered as a thrown exception.

```cpp
task<void> error_mapping_demo() {
    try {
        co_await async_connect(
            socket, endpoint);
    } catch (system_error const& e) {
        // set_error(ec) became throw
        log("connect failed: {}",
            e.code().message());
    }
}
```

This is how the three-channel model composes with coroutines. The value channel becomes the `co_await` return value. The error channel becomes a thrown exception. The stopped channel becomes cancellation. The mapping is clean and the programmer can use familiar `try`/`catch` for error handling. The composition algebra built on `set_error` integrates with coroutines through the exception mechanism - the two error models meet at the `co_await` boundary.

### 12.10 Compound Results

When an I/O operation returns both a status code and a byte count, the compound result can stay on the value channel. Both values arrive at the `co_await` expression.

```cpp
task<void> read_with_status() {
    auto [ec, n] = co_await
        async_read(socket, buf);
    if (!ec) {
        process(buf, n);
    } else if (
            ec == errc::connection_reset) {
        cleanup(n);
    }
}
```

Both values are visible. The programmer branches with `if`. The composition algebra - `retry`, `upon_error`, `when_all` - does not participate in this dispatch, because the result stayed on the value channel. This is the trade-off: data preservation or composition algebra participation, not both simultaneously. The programmer always has the option of writing sender code directly instead of using the coroutine, gaining access to the full composition algebra at the cost of the programming model documented in Sections 3 through 11.

Alternatively, the compound result can be bundled into the error type:

```cpp
struct io_result {
    error_code ec;
    size_t bytes;
};

task<void> bundled_error() {
    auto [ec, n] = co_await
        async_read(socket, buf);
    if (ec)
        co_yield with_error(
            io_result{ec, n});
    process(buf, n);
}
```

The composition algebra now participates - `retry` sees the error, `upon_error` can handle it. Every `upon_error` handler downstream must accept `io_result` alongside any other error types in the pipeline. The data survives the channel crossing because it is inside the error object. Both approaches are legitimate design choices. The programmer evaluates the trade-off for the domain at hand.

### 12.11 Pipelines Inside Coroutines

A sender pipeline can be `co_await`ed as a single expression inside a `task`.

```cpp
task<string> fetch_and_transform(url u) {
    auto result = co_await (
        async_fetch(u)
      | then([](auto resp) {
            return resp.body();
        })
      | then([](auto body) {
            return compress(body);
        }));
    co_return result;
}
```

The pipeline is composed using the pipe operator, then `co_await`ed as a unit. The `task` provides the execution context; the pipeline provides the composition. The integration between coroutines and senders is seamless - each model contributes its strength.

### 12.12 Tasks as Senders

A `task` is a sender. It can be used anywhere a sender is expected - piped into `then`, passed to `when_all`, composed with any algorithm from Sections 3 through 11.

```cpp
auto sndr = fetch_and_transform(my_url)
          | then([](string compressed) {
                return store(compressed);
            });
auto [stored] =
    sync_wait(std::move(sndr)).value();
```

The `task` enters the sender world as a first-class participant. The coroutine's completion signatures become the sender's completion signatures. The scheduler affinity, allocator support, and error handling all compose through the sender protocol. The programmer has everything they need.

---

## 13. Composition

The final section demonstrates how senders and coroutines compose together in a realistic program. The example uses four layers, each building on the previous, combining the algorithms from this tutorial into a single system. The resulting program is readable - each layer is a natural application of the patterns introduced earlier.

### 13.1 Sensor Fusion

A `task` that `co_await`s a structured-concurrency pipeline to read three sensors in parallel and fuse the results.

```cpp
task<sensor_data> read_sensors(
        sensor_array const& sensors) {
    auto [lidar, radar, camera] = co_await
        when_all(
            starts_on(
                sensors.lidar_ctx(),
                async_read(
                    sensors.lidar()))
            | then([](raw_data raw) {
                  return decode_lidar(raw);
              }),
            starts_on(
                sensors.radar_ctx(),
                async_read(
                    sensors.radar()))
            | then([](raw_data raw) {
                  return decode_radar(raw);
              }),
            starts_on(
                sensors.camera_ctx(),
                async_read(
                    sensors.camera()))
            | then([](raw_data raw) {
                  return decode_camera(raw);
              }));
    co_return fuse(lidar, radar, camera);
}
```

Three sensors, three execution contexts, one `when_all`. The `task` provides scheduler affinity; the pipeline provides concurrency. The fused result is a single `sensor_data` value.

### 13.2 Collision Detection

The `task` from 13.1 is used as a sender inside a pipeline that evaluates whether immediate braking is required.

```cpp
auto collision_pipeline(
        sensor_array const& sensors,
        vehicle_state const& state) {
    return read_sensors(sensors)
         | then([&](sensor_data data) {
               return detect_obstacles(
                   data, state);
           })
         | let_value(
               [](obstacle_set obstacles) {
               if (obstacles.imminent())
                   return just(brake_cmd{
                       obstacles.severity()});
               return just(
                   brake_cmd::none());
           });
}
```

The `task` is a sender. The pipeline consumes it with `then` and `let_value`. The branching inside `let_value` returns different `brake_cmd` values - both branches return the same sender type, so no type erasure is needed.

### 13.3 Actuator Command

A `task` that `co_await`s the collision pipeline and sends brake commands on a real-time scheduler.

```cpp
task<actuator_result> actuate_brakes(
        sensor_array const& sensors,
        vehicle_state const& state,
        brake_controller& brakes) {
    auto cmd = co_await
        collision_pipeline(sensors, state);
    if (cmd.severity() > 0) {
        co_await starts_on(
            brakes.realtime_ctx(),
            async_write(
                brakes.channel(), cmd));
        co_return
            actuator_result::applied(cmd);
    }
    co_return actuator_result::none();
}
```

The pipeline from 13.2 is `co_await`ed inside a `task`. The brake command is issued on a real-time scheduler - a context transition from the sensor-processing context to the actuator context.

### 13.4 Failover

The `task` from 13.3 is used as a sender inside a pipeline that handles sensor failure with emergency braking.

```cpp
auto safety_controller(
        sensor_array const& sensors,
        vehicle_state const& state,
        brake_controller& brakes,
        counting_scope& scope) {
    return associate(
        scope.get_token(),
        actuate_brakes(
            sensors, state, brakes)
      | let_error(
            [&](error_code ec) {
            return starts_on(
                brakes.realtime_ctx(),
                async_write(
                    brakes.channel(),
                    brake_cmd::emergency()))
              | then([ec](auto) {
                    log("sensor failure: "
                        "{}, emergency "
                        "brake applied",
                        ec);
                    return actuator_result
                        ::emergency();
                });
        })
      | upon_stopped([&] {
            log("controller stopped");
            return
                actuator_result::none();
        }));
}
```

Four layers. A sender pipeline containing a `task` containing a sender pipeline containing a `task` containing a sender pipeline. Each layer is individually reasonable - a natural application of the algorithms this tutorial has introduced. The structure is intuitive. The composition follows the patterns established in the preceding sections. The tutorial has, in a sense, been unnecessary - the Sub-Language is its own best teacher.

### 13.5 The Equivalent Program

```cpp
task<actuator_result> safety_controller(
        sensor_array const& sensors,
        vehicle_state const& state,
        brake_controller& brakes) {
    try {
        auto data = co_await
            read_all_sensors(sensors);
        auto obs =
            detect_obstacles(data, state);
        if (obs.imminent()) {
            auto cmd =
                brake_cmd{obs.severity()};
            co_await brakes.apply(cmd);
            co_return
                actuator_result::applied(
                    cmd);
        }
        co_return actuator_result::none();
    } catch (...) {
        co_await
            brakes.emergency_stop();
        co_return
            actuator_result::emergency();
    }
}
```

---

## 14. Conclusion

This tutorial has progressed from the simplest sender algorithm - `just(42)` - to a four-layer composition of sender pipelines and coroutines managing sensor fusion, collision detection, actuator commands, and failover in a safety-critical control system. Some readers may find the later examples challenging. There is no rush. The examples reward careful study and repeated reading. The Sender Sub-Language is C++26's asynchronous programming model - the standard's answer to structured concurrency and heterogeneous execution. The programmer who masters it has mastered the model the standard provides. With patience and practice, every pattern in this tutorial becomes familiar.

Two models, each correct for its domain, is a stronger standard than one model asked to serve both.

---

## Acknowledgments

The authors thank the P2300 authors - Micha&lstrok; Dominiak, Georgy Evtushenko, Lewis Baker, Lucian Radu Teodorescu, Lee Howes, Kirk Shoop, Michael Garland, Eric Niebler, and Bryce Adelstein Lelbach - for building `std::execution` and for the theoretical depth that informs this tutorial. The authors also thank Dietmar K&uuml;hl and Maikel Nadolski for the `task` coroutine type, Steve Downey for the sender-examples that informed our understanding, and Peter Dimov for editorial guidance.

---

## References

1. [N5014](https://www.open-std.org/jtc1/sc22/wg21/docs/papers/2025/n5014.pdf). Thomas K&ouml;ppe (ed.). "Working Draft, Standard for Programming Language C++." 2025. https://www.open-std.org/jtc1/sc22/wg21/docs/papers/2025/n5014.pdf

2. [P3552R3](https://wg21.link/p3552r3). Dietmar K&uuml;hl, Maikel Nadolski. "Add a Coroutine Task Type." 2025. https://wg21.link/p3552r3

3. [Boost.Beast](https://github.com/boostorg/beast). Vinnie Falco. HTTP and WebSocket library. https://github.com/boostorg/beast

4. [Corosio](https://github.com/cppalliance/corosio). Vinnie Falco. Coroutine-native networking library. https://github.com/cppalliance/corosio

5. [Capy](https://github.com/cppalliance/capy). Vinnie Falco. Coroutine I/O primitives. https://github.com/cppalliance/capy

6. [P2300R10](https://wg21.link/p2300r10). Micha&lstrok; Dominiak, Georgy Evtushenko, Lewis Baker, Lucian Radu Teodorescu, Lee Howes, Kirk Shoop, Michael Garland, Eric Niebler, Bryce Adelstein Lelbach. "std::execution." 2024. https://wg21.link/p2300r10

7. Herb Sutter. ["Living in the Future: Using C++26 at Work"](https://herbsutter.com/2025/04/23/living-in-the-future-using-c26-at-work/). 2025. https://herbsutter.com/2025/04/23/living-in-the-future-using-c26-at-work/

8. [stdexec](https://github.com/NVIDIA/stdexec). NVIDIA's reference implementation of `std::execution`. https://github.com/NVIDIA/stdexec

9. ["New C++ Sender Library Enables Portable Asynchrony"](https://www.hpcwire.com/2022/12/05/new-c-sender-library-enables-portable-asynchrony/). HPC Wire, 2022. https://www.hpcwire.com/2022/12/05/new-c-sender-library-enables-portable-asynchrony/

10. Guy Steele and Gerald Sussman. [*The Lambda Papers*](https://en.wikisource.org/wiki/Lambda_Papers). MIT AI Memo series, 1975-1980. https://en.wikisource.org/wiki/Lambda_Papers

11. Gordon Plotkin. ["Call-by-name, call-by-value and the lambda-calculus"](https://doi.org/10.1016/0304-3975(75)90017-1). *Theoretical Computer Science*, 1(2):125-159, 1975. https://doi.org/10.1016/0304-3975(75)90017-1

12. [SML/NJ](https://www.smlnj.org/). Standard ML of New Jersey. https://www.smlnj.org/

13. [GHC](https://www.haskell.org/ghc/). Glasgow Haskell Compiler. https://www.haskell.org/ghc/

14. [Chicken Scheme](https://www.call-cc.org/). Scheme implementation using CPS compilation. https://www.call-cc.org/

15. Eugenio Moggi. ["Notions of Computation and Monads"](https://doi.org/10.1016/0890-5401(91)90052-4). *Information and Computation*, 1991. https://doi.org/10.1016/0890-5401(91)90052-4

16. Olivier Danvy and Andrzej Filinski. ["Abstracting Control"](https://doi.org/10.1145/91556.91622). *LFP*, 1990. https://doi.org/10.1145/91556.91622

17. Timothy Griffin. ["A Formulae-as-Types Notion of Control"](https://doi.org/10.1145/96709.96714). *POPL*, 1990. https://doi.org/10.1145/96709.96714

18. [P3796R1](https://wg21.link/p3796r1). Dietmar K&uuml;hl. "Coroutine Task Issues." 2025. https://wg21.link/p3796r1

19. [P3826R3](https://wg21.link/p3826r3). Eric Niebler. "Fix Sender Algorithm Customization." 2026. https://wg21.link/p3826r3

20. [P2079R10](https://www.open-std.org/jtc1/sc22/wg21/docs/papers/2025/p2079r10.html). Lucian Radu Teodorescu, Ruslan Arutyunyan, Lee Howes, Michael Voss. "Parallel Scheduler." 2025. https://www.open-std.org/jtc1/sc22/wg21/docs/papers/2025/p2079r10.html

21. [sender-examples](https://github.com/steve-downey/sender-examples). Steve Downey. Example code for C++Now talk. https://github.com/steve-downey/sender-examples

22. Bjarne Stroustrup. *The Design and Evolution of C++*. Addison-Wesley, 1994. ISBN 0-201-54330-3.

23. Bartosz Milewski. [*Category Theory for Programmers*](https://github.com/hmemcpy/milewski-ctfp-pdf). 2019. https://github.com/hmemcpy/milewski-ctfp-pdf

24. Simon Marlow (ed.). [*Haskell 2010 Language Report*](https://www.haskell.org/onlinereport/haskell2010/). 2010. https://www.haskell.org/onlinereport/haskell2010/
