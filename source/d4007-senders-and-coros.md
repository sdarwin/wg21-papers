---
title: "`std::execution::task` Shipping Gate"
document: D4007R1
date: 2026-03-11
reply-to:
  - "Vinnie Falco <vinnie.falco@gmail.com>"
  - "Mungo Gill <mungo.gill@me.com>"
audience: LEWG
---

## Abstract

`std::execution::task` ([P3552R3](https://wg21.link/p3552r3)<sup>[1]</sup>) has open defects identified by national ballot comments, LWG issues, and published papers. This paper lists criteria that must be resolved to ship `task` in C++26. If all are resolved, ship. If any remain, apply the proposed wording in Section 5 to remove `task` from the working draft.

---

## Revision History

### R1: March 2026 (post-Croydon mailing)

* Complete rewrite as a criteria paper.

### R0: February 2026 (pre-Croydon mailing)

* Original analysis of structural gaps. See [P4007R0](https://wg21.link/p4007r0)<sup>[2]</sup>.

---

## 1. Disclosure

The authors developed [P4003R0](https://wg21.link/p4003r0)<sup>[3]</sup> ("Coroutines for I/O") and [P4007R0](https://wg21.link/p4007r0)<sup>[2]</sup> ("Senders and Coroutines"). A coroutine-only design cannot express compile-time work graphs, does not support heterogeneous dispatch, and assumes a cooperative runtime. Those are real costs. The defects documented below hold regardless of whether P4003 is the right design or any design at all.

---

## 2. Fixed After Ship

`task`'s `promise_type` is a class template instantiated in user code. Its `operator new`, allocator selection, stop token storage, environment forwarding, and destruction ordering can change between standard revisions without binary incompatibility. These issues are fixable post-ship: [Unusual Allocator Customisation](https://wg21.link/p3796r1)<sup>[4]</sup>, [Flexible Allocator Position](https://wg21.link/p3796r1)<sup>[4]</sup>, [Shadowing The Environment Allocator](https://wg21.link/p3796r1)<sup>[4]</sup>, [Stop Source Always Created](https://wg21.link/p3796r1)<sup>[4]</sup>, [Stop Token Default Constructible](https://wg21.link/p3796r1)<sup>[4]</sup>, [Task Not Actually Lazily Started](https://wg21.link/p3796r1)<sup>[4]</sup>, [Frame Destroyed Too Late](https://wg21.link/p3796r1)<sup>[4]</sup>, [No Default Arguments](https://wg21.link/p3796r1)<sup>[4]</sup>, [`unhandled_stopped` Not `noexcept`](https://wg21.link/p3796r1)<sup>[4]</sup>, [Environment Design Inefficient](https://wg21.link/p3796r1)<sup>[4]</sup>, [Non-Sender Awaitables Unsupported](https://wg21.link/p3796r1)<sup>[4]</sup>, [Future Language Feature Could Avoid `co_yield`](https://wg21.link/p3796r1)<sup>[4]</sup>, [No TLS Capture/Restore Hook](https://wg21.link/p3796r1)<sup>[4]</sup>, [`return_value`/`return_void` Have No Specification](https://wg21.link/p3796r1)<sup>[4]</sup>, [`co_return { args... }` Unsupported](https://wg21.link/p3796r1)<sup>[4]</sup>, [`change_coroutine_scheduler` Requires Assignable Scheduler](https://wg21.link/p3796r1)<sup>[4]</sup>, [Sender-Unaware Coroutines Cannot `co_await` a `task`](https://wg21.link/p3796r1)<sup>[4]</sup>, [Missing Rvalue Qualification](https://wg21.link/p3796r1)<sup>[4]</sup>, [Parameter Lifetime Is Surprising](https://wg21.link/p3801r0)<sup>[5]</sup>, [No Protection Against Dangling References](https://wg21.link/p3801r0)<sup>[5]</sup>, [`co_yield with_error` Is Clunky](https://wg21.link/p3801r0)<sup>[5]</sup>, [`co_await schedule(sch)` Is an Expensive No-Op](https://wg21.link/p3801r0)<sup>[5]</sup>, [Coroutine Cancellation Is Ad-Hoc](https://wg21.link/p3801r0)<sup>[5]</sup>.

`affine_on` semantics, rescheduling behavior, and algorithm customisation are specification-level concerns. Tightening requirements or adding default implementations does not change any published interface. These issues are fixable post-ship: [`affine_on` Default Implementation Lacks Specification](https://wg21.link/p3796r1)<sup>[4]</sup>, [`affine_on` Semantics Not Clear](https://wg21.link/p3796r1)<sup>[4]</sup>, [`affine_on` Shape May Not Be Correct](https://wg21.link/p3796r1)<sup>[4]</sup>, [`affine_on` Shouldn't Forward Stop Requests](https://wg21.link/p3796r1)<sup>[4]</sup>, [`affine_on` Customisation For Other Senders](https://wg21.link/p3796r1)<sup>[4]</sup>, [Starting a `task` Reschedules Unconditionally](https://wg21.link/p3796r1)<sup>[4]</sup>, [Resuming After a `task` Reschedules Unnecessarily](https://wg21.link/p3796r1)<sup>[4]</sup>, [`bulk` vs. `task_scheduler`](https://wg21.link/p3796r1)<sup>[4]</sup>, [No Completion Scheduler](https://wg21.link/p3796r1)<sup>[4]</sup>, [`with_awaitable_senders` Unused](https://wg21.link/p3796r1)<sup>[4]</sup>.

---

## 3. Ships Broken

The criteria in this section are items where shipping forecloses the fix.

| Criterion             | References                                                                                                                                                                                                                                                                  | Fixed |
|-----------------------|-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|:-----:|
| Allocator Timing      | [P4007R0](https://wg21.link/p4007r0)<sup>[2]</sup> S5, [D3980R0](https://isocpp.org/files/papers/D3980R0.html)<sup>[6]</sup>, [P3796R1](https://wg21.link/p3796r1)<sup>[4]</sup>, [LWG 4356](https://cplusplus.github.io/LWG/issue4356)<sup>[7]</sup>, [US 254-385](https://github.com/cplusplus/nbballot/issues/960)<sup>[8]</sup> | no    |
| Allocator Propagation | [P4007R0](https://wg21.link/p4007r0)<sup>[2]</sup> S5, [D3980R0](https://isocpp.org/files/papers/D3980R0.html)<sup>[6]</sup>, [P3796R1](https://wg21.link/p3796r1)<sup>[4]</sup>                                                                                          | no    |
| Complicated Success   | [P4007R0](https://wg21.link/p4007r0)<sup>[2]</sup> S3, [P2430R0](https://wg21.link/p2430r0)<sup>[9]</sup>                                                                                                                                                                  | no    |
| Exception Hygiene     | [P4007R0](https://wg21.link/p4007r0)<sup>[2]</sup> S3.7                                                                                                                                                                                                                    | no    |
| Error Return          | [P4007R0](https://wg21.link/p4007r0)<sup>[2]</sup> S4, [P3950R0](https://wg21.link/p3950r0)<sup>[10]</sup>, [P3801R0](https://wg21.link/p3801r0)<sup>[5]</sup>, [P1713R0](https://wg21.link/p1713r0)<sup>[11]</sup>                                                        | no    |
| Symmetric Transfer    | [P2583R3](https://wg21.link/p2583r3)<sup>[12]</sup>, [US 246-373](https://github.com/cplusplus/nbballot/issues/948)<sup>[13]</sup>, [LWG 4348](https://cplusplus.github.io/LWG/issue4348)<sup>[14]</sup>, [P3801R0](https://wg21.link/p3801r0)<sup>[5]</sup>, [P3796R1](https://wg21.link/p3796r1)<sup>[4]</sup> | no    |

- **Allocator Timing.** The user can specify either a type `Allocator` or pointer to `pmr::memory_resource` at the launch site and this allocator is injected into the environment and used to allocate the coroutine frame of the launched task; if the launch function implementation creates any trampolines, those coroutine frames are also allocated using the user's specified allocator.

- **Allocator Propagation.** The allocator used in the launch function is invisibly propagated to all child tasks.

- **Complicated Success.** I/O outcomes (`error_code`, value pair) map cleanly to the sender's three-channel design, without data loss, and without foreclosing a subset of downstream sender algorithms.

- **Exception Hygiene.** No exceptions thrown for transient errors like `ECONNRESET` or `ssl::stream_truncated`.

- **Error Return.** Users write `co_return` to propagate error codes to the caller, not `co_yield`.

- **Symmetric Transfer.** `sender-awaitable::await_suspend` returns `void`, so co_awaiting a synchronously completing sender grows the stack by one frame per completion with no upper bound.

---

## 4. Straw Polls

1. "The committee is comfortable shipping `std::execution::task` in C++26 with the defects listed in Section 3 unresolved."

2. "Apply the proposed wording in Section 5."

---

## 5. Proposed Wording

TBD

---

## References

### WG21 Papers

1. [P3552R3](https://wg21.link/p3552r3) - "Add a Coroutine Task Type" (Dietmar K&uuml;hl, Maikel Nadolski, 2025). https://wg21.link/p3552r3
2. [P4007R0](https://wg21.link/p4007r0) - "Senders and Coroutines" (Vinnie Falco, Mungo Gill, 2026). https://wg21.link/p4007r0
3. [P4003R0](https://wg21.link/p4003r0) - "Coroutines for I/O" (Vinnie Falco, Steve Gerbino, Mungo Gill, 2026). https://wg21.link/p4003r0
4. [P3796R1](https://wg21.link/p3796r1) - "Coroutine Task Issues" (Dietmar K&uuml;hl, 2025). https://wg21.link/p3796r1
5. [P3801R0](https://wg21.link/p3801r0) - "Concerns about the design of std::execution::task" (Jonathan M&uuml;ller, 2025). https://wg21.link/p3801r0
6. [D3980R0](https://isocpp.org/files/papers/D3980R0.html) - "Task's Allocator Use" (Dietmar K&uuml;hl, 2026). https://isocpp.org/files/papers/D3980R0.html
7. [LWG 4356](https://cplusplus.github.io/LWG/issue4356) - "connect() should use get_allocator(get_env(rcvr))". https://cplusplus.github.io/LWG/issue4356
8. [US 254-385](https://github.com/cplusplus/nbballot/issues/960) - C++26 NB ballot comment. https://github.com/cplusplus/nbballot/issues/960
9. [P2430R0](https://wg21.link/p2430r0) - "Partial success scenarios with P2300" (Chris Kohlhoff, 2021). https://wg21.link/p2430r0
10. [P3950R0](https://wg21.link/p3950r0) - "return_value & return_void Are Not Mutually Exclusive" (Robert Leahy, 2025). https://wg21.link/p3950r0
11. [P1713R0](https://wg21.link/p1713r0) - "Allowing both co_return; and co_return value; in the same coroutine" (Lewis Baker, 2019). https://wg21.link/p1713r0
12. [P2583R3](https://wg21.link/p2583r3) - "Symmetric Transfer and Sender Composition" (Mungo Gill, Vinnie Falco, 2026). https://wg21.link/p2583r3
13. [US 246-373](https://github.com/cplusplus/nbballot/issues/948) - C++26 NB ballot comment. https://github.com/cplusplus/nbballot/issues/948
14. [LWG 4348](https://cplusplus.github.io/LWG/issue4348) - "task doesn't support symmetric transfer". https://cplusplus.github.io/LWG/issue4348

### Other

15. [P2300R10](https://wg21.link/p2300r10) - "std::execution" (Micha&lstrok; Dominiak et al., 2024). https://wg21.link/p2300r10
16. [C++ Working Draft](https://eel.is/c++draft/) - (Richard Smith, ed.). https://eel.is/c++draft/
