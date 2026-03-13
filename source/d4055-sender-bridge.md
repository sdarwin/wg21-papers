---
title: "Consuming Senders from Coroutine-Native Code"
document: D4055R0
date: 2026-03-13
reply-to:
  - "Vinnie Falco <vinnie.falco@gmail.com>"
  - "Steve Gerbino <steve@gerbino.co>"
audience: LEWG
---

## Abstract

An `IoAwaitable` bridge ([P4003R0](https://wg21.link/p4003r0)<sup>[3]</sup>) consumes `std::execution` senders with inline operation state, correct stop token propagation, and automatic executor dispatch-back. The bridge is one class template. The complete implementation is in Appendix A.

---

## Revision History

### R0: March 2026 (post-Croydon mailing)

- Initial version.

---

## 1. Disclosure

The authors developed and maintain [Capy](https://github.com/cppalliance/capy/tree/f0466466e63baf0cc3d6034bc35eec24694f5d16)<sup>[4]</sup> and [Corosio](https://github.com/cppalliance/corosio)<sup>[6]</sup> and believe coroutine-native I/O is the correct foundation for networking in C++. The findings in this paper are structural and hold regardless of which library implements the coroutine-native layer. This paper is one of a suite of four that examines the relationship between compound I/O results and the sender three-channel model. The companion papers are [D4053R0](https://wg21.link/d4053r0)<sup>[2]</sup>, "Sender I/O: A Constructed Comparison"; [D4054R0](https://wg21.link/d4054r0)<sup>[11]</sup>, "Two Error Models"; and [D4056R0](https://wg21.link/d4056r0)<sup>[12]</sup>, "Producing Senders from Coroutine-Native Code." The authors provide information, ask nothing, and serve at the pleasure of the chair. [Capy](https://github.com/cppalliance/capy/tree/f0466466e63baf0cc3d6034bc35eec24694f5d16)<sup>[4]</sup> is a coroutine primitives library. It contains no sockets, no timers, and no platform-specific I/O APIs. Networking is in [Corosio](https://github.com/cppalliance/corosio)<sup>[6]</sup>, which is not used in this paper. The bridge depends on [Capy](https://github.com/cppalliance/capy/tree/f0466466e63baf0cc3d6034bc35eec24694f5d16)<sup>[4]</sup> and `beman::execution`<sup>[5]</sup>, a community implementation of `std::execution` ([P2300R10](https://wg21.link/p2300r10)<sup>[1]</sup>). Source code links refer to pinned commit [f046646](https://github.com/cppalliance/capy/tree/f0466466e63baf0cc3d6034bc35eec24694f5d16)<sup>[4]</sup>.

---

## 2. The Bridge

```cpp
capy::task<int> compute(auto sched)
{
    int result = co_await capy::await_sender(
        ex::schedule(sched)
            | ex::then([] {
                std::cout
                    << "  sender running on thread "
                    << std::this_thread::get_id()
                    << "\n";
                return 42 * 42;
            }));

    std::cout
        << "  coroutine resumed on thread "
        << std::this_thread::get_id() << "\n";

    co_return result;
}
```

`await_sender` returns a `sender_awaitable` that satisfies `IoAwaitable` ([P4003R0](https://wg21.link/p4003r0)<sup>[3]</sup>). Any coroutine type whose promise propagates `io_env` through `await_suspend(h, io_env const*)` can use it. `capy::task` is one such type. It is not the only one.

Appendix A contains the complete bridge implementation. It was tested against `beman::execution`<sup>[5]</sup>.

---

## 3. Demonstration

```
main thread: 32208
  sender running on thread 9560
  coroutine resumed on thread 34356
result: 1764
```

The sender executed on thread 9560, the `beman::execution::run_loop` thread. The coroutine resumed on thread 34356, the Capy thread pool. The value 1764 crossed the bridge. The bridge allocated zero bytes on this path.

---

## 4. What the Bridge Does

The bridge consumes any `std::execution` sender. Scheduler hops work. Stop token propagation works. `set_value`, `set_error`, and `set_stopped` are handled. The operation state is stored inline. Any sender pipeline - `when_all`, `then`, `let_value`, `on` - works through the bridge.

An `IoAwaitable` coroutine can suspend into a sender pipeline that schedules work on a GPU, a thread pool, or any other execution context. When the sender completes, the bridge dispatches the resumption back to the coroutine's originating executor (`env_->executor.post(cont_)`). The coroutine resumes in the correct context with the sender's result, regardless of where the sender executed. The bridge is the interop path for coroutine-native I/O code that needs to offload compute to a GPU scheduler and resume on the I/O event loop.

The bridge does not use `execution::task`.

---

## 5. What the Bridge Does Not Require

| Property                                | `execution::task` | Bridge |
| --------------------------------------- | ------------------ | ------ |
| Routine I/O errors become exceptions    | Yes                | No     |
| Type erasure on connect                 | Yes                | No     |
| `dispatch` adapter for compound results | Yes                | No     |
| Zero allocations in the bridge          | No                 | Yes    |

`std::execution::task` is not necessary to consume senders.

---

## 6. The Narrowest Abstraction

[Capy](https://github.com/cppalliance/capy/tree/f0466466e63baf0cc3d6034bc35eec24694f5d16)<sup>[4]</sup> is a coroutine primitives library. It provides `task`, executors, stop tokens, frame allocators, and buffer sequences. It contains no sockets, no timers, and no platform-specific I/O APIs. Networking is in [Corosio](https://github.com/cppalliance/corosio)<sup>[6]</sup>, a separate library that depends on [Capy](https://github.com/cppalliance/capy/tree/f0466466e63baf0cc3d6034bc35eec24694f5d16)<sup>[4]</sup>.

The sender bridge depends on [Capy](https://github.com/cppalliance/capy/tree/f0466466e63baf0cc3d6034bc35eec24694f5d16)<sup>[4]</sup> and `std::execution`. It does not depend on [Corosio](https://github.com/cppalliance/corosio)<sup>[6]</sup>. It does not depend on any platform I/O API. A user who needs I/O adds [Corosio](https://github.com/cppalliance/corosio)<sup>[6]</sup>. A user who needs sender interop adds the bridge. A user who needs neither does not pay for either. The coroutine type does not change between these configurations.

---

## 7. Acknowledgments

The authors thank Dietmar K&uuml;hl for `beman::execution`<sup>[5]</sup> and for the channel-routing enumeration in [P2762R2](https://wg21.link/p2762r2)<sup>[7]</sup>, Micha&lstrok; Dominiak, Eric Niebler, and Lewis Baker for `std::execution`, Chris Kohlhoff for identifying the partial-success problem in [P2430R0](https://wg21.link/p2430r0)<sup>[8]</sup>, Kirk Shoop for the completion-token heuristic analysis in [P2471R1](https://wg21.link/p2471r1)<sup>[9]</sup>, Fabio Fracassi for [P3570R2](https://wg21.link/p3570r2)<sup>[10]</sup>, Ville Voutilainen and Jens Maurer for reflector discussion on dispatch patterns, Herb Sutter for identifying the need for tutorials and documentation, Mark Hoemmen for insights on `std::linalg` and the layered abstraction model, and Peter Dimov for the refined channel mapping.

---

## References

1. [P2300R10](https://wg21.link/p2300r10) - "std::execution" (Micha&lstrok; Dominiak et al., 2024). https://wg21.link/p2300r10

2. [D4053R0](https://wg21.link/d4053r0) - "Sender I/O: A Constructed Comparison" (Vinnie Falco, Steve Gerbino, 2026). https://wg21.link/d4053r0

3. [P4003R0](https://wg21.link/p4003r0) - "Coroutines for I/O" (Vinnie Falco, Steve Gerbino, Mungo Gill, 2026). https://wg21.link/p4003r0

4. [cppalliance/capy](https://github.com/cppalliance/capy/tree/f0466466e63baf0cc3d6034bc35eec24694f5d16) - Coroutine primitives library. Commit f046646. https://github.com/cppalliance/capy/tree/f0466466e63baf0cc3d6034bc35eec24694f5d16

5. [bemanproject/execution](https://github.com/bemanproject/execution) - Community implementation of `std::execution`. https://github.com/bemanproject/execution

6. [cppalliance/corosio](https://github.com/cppalliance/corosio) - Coroutine-native networking library. https://github.com/cppalliance/corosio

7. [P2762R2](https://wg21.link/p2762r2) - "Sender/Receiver Interface For Networking" (Dietmar K&uuml;hl, 2023). https://wg21.link/p2762r2

8. [P2430R0](https://wg21.link/p2430r0) - "Partial success scenarios with P2300" (Chris Kohlhoff, 2021). https://wg21.link/p2430r0

9. [P2471R1](https://wg21.link/p2471r1) - "NetTS, ASIO and Sender Library Design Comparison" (Kirk Shoop, 2021). https://wg21.link/p2471r1

10. [P3570R2](https://wg21.link/p3570r2) - "Optional variants in sender/receiver" (Fabio Fracassi, 2025). https://wg21.link/p3570r2

11. [D4054R0](https://wg21.link/d4054r0) - "Two Error Models" (Vinnie Falco, 2026). https://wg21.link/d4054r0

12. [D4056R0](https://wg21.link/d4056r0) - "Producing Senders from Coroutine-Native Code" (Vinnie Falco, Steve Gerbino, 2026). https://wg21.link/d4056r0

---

## Appendix A. Bridge Implementation

```cpp
#include <boost/capy/ex/io_env.hpp>

#include <beman/execution/execution.hpp>

#include <cassert>
#include <coroutine>
#include <cstring>
#include <exception>
#include <new>
#include <stop_token>
#include <stdexcept>
#include <tuple>
#include <type_traits>
#include <utility>
#include <variant>

namespace boost::capy {

namespace detail {

struct stopped_t {};

struct bridge_env
{
    std::stop_token st_;

    auto query(
        beman::execution::get_stop_token_t const&)
            const noexcept
    {
        return st_;
    }
};

template<class Sender>
using sender_single_value_t =
    beman::execution::value_types_of_t<
        Sender,
        bridge_env,
        std::tuple,
        std::type_identity_t>;

template<class ValueTuple>
struct bridge_receiver
{
    using receiver_concept =
        beman::execution::receiver_t;

    std::variant<
        std::monostate,
        ValueTuple,
        std::exception_ptr,
        stopped_t>*         result_;
    std::coroutine_handle<> cont_;
    io_env const*           env_;

    auto get_env() const noexcept -> bridge_env
    {
        return {env_->stop_token};
    }

    template<class... Args>
    void set_value(Args&&... args) && noexcept
    {
        result_->template emplace<1>(
            std::forward<Args>(args)...);
        env_->executor.post(cont_);
    }

    template<class E>
    void set_error(E&& e) && noexcept
    {
        if constexpr (
            std::is_same_v<
                std::decay_t<E>,
                std::exception_ptr>)
            result_->template emplace<2>(
                std::forward<E>(e));
        else
            result_->template emplace<2>(
                std::make_exception_ptr(
                    std::forward<E>(e)));
        env_->executor.post(cont_);
    }

    void set_stopped() && noexcept
    {
        result_->template emplace<3>(
            stopped_t{});
        env_->executor.post(cont_);
    }
};

} // namespace detail

template<class Sender>
struct sender_awaitable
{
    using value_tuple =
        detail::sender_single_value_t<Sender>;
    using receiver_type =
        detail::bridge_receiver<value_tuple>;
    using op_state_type = decltype(
        beman::execution::connect(
            std::declval<Sender>(),
            std::declval<receiver_type>()));

    Sender sndr_;

    std::variant<
        std::monostate,
        value_tuple,
        std::exception_ptr,
        detail::stopped_t> result_{};

    alignas(op_state_type)
    unsigned char op_buf_[sizeof(op_state_type)];
    bool op_constructed_ = false;

    explicit sender_awaitable(Sender sndr)
        : sndr_(std::move(sndr))
    {
    }

    sender_awaitable(sender_awaitable&& o)
        noexcept(
            std::is_nothrow_move_constructible_v<
                Sender>)
        : sndr_(std::move(o.sndr_))
    {
        assert(!o.op_constructed_);
    }

    sender_awaitable(
        sender_awaitable const&) = delete;
    sender_awaitable& operator=(
        sender_awaitable const&) = delete;
    sender_awaitable& operator=(
        sender_awaitable&&) = delete;

    ~sender_awaitable()
    {
        if(op_constructed_)
            std::launder(
                reinterpret_cast<op_state_type*>(
                    op_buf_))->~op_state_type();
    }

    bool await_ready() const noexcept
    {
        return false;
    }

    std::coroutine_handle<>
    await_suspend(
        std::coroutine_handle<> h,
        io_env const* env)
    {
        ::new(op_buf_) op_state_type(
            beman::execution::connect(
                std::move(sndr_),
                receiver_type{
                    &result_, h, env}));
        op_constructed_ = true;
        beman::execution::start(
            *std::launder(
                reinterpret_cast<
                    op_state_type*>(
                        op_buf_)));
        return std::noop_coroutine();
    }

    auto await_resume()
    {
        if(result_.index() == 2)
            std::rethrow_exception(
                std::get<2>(result_));
        if(result_.index() == 3)
            throw std::runtime_error(
                "sender completed with "
                "set_stopped");

        if constexpr (
            std::tuple_size_v<
                value_tuple> == 0)
            return;
        else if constexpr (
            std::tuple_size_v<
                value_tuple> == 1)
            return std::get<0>(
                std::get<1>(
                    std::move(result_)));
        else
            return std::get<1>(
                std::move(result_));
    }
};

template<class Sender>
auto await_sender(Sender&& sndr)
{
    return sender_awaitable<
        std::decay_t<Sender>>(
            std::forward<Sender>(sndr));
}

} // namespace boost::capy
```
