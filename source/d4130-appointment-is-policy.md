---
title: "Appointment Is Policy"
document: P4130R0
date: 2026-03-21
reply-to:
  - "Vinnie Falco <vinnie.falco@gmail.com>"
  - "C++ Alliance Proposal Team"
audience: WG21
---

## Abstract

The convener appoints every chair. The chairs control every schedule. The schedule determines what ships.

The system produces results. The 2026 convenership transition is the first such moment in a generation. How the new convener uses these powers will shape the next decade of C++.

---

## Revision History

### R0: March 2026 (post-Croydon mailing)

- Initial version.

---

## 1. Disclosure

The author has papers in the WG21 mailing and participates in the committee process. The author developed and maintains [Corosio](https://github.com/cppalliance/corosio)<sup>[1]</sup> and [Capy](https://github.com/cppalliance/capy)<sup>[2]</sup> and is a co-author of [P2469R0](https://wg21.link/p2469r0)<sup>[3]</sup>. The author's preferred asynchronous model competes with [P2300R10](https://wg21.link/p2300r10)<sup>[4]</sup> (`std::execution`). Coroutine-native I/O cannot express compile-time work graphs. That is a real limitation. The author is telling you this so you can calibrate everything that follows.

This paper is a companion to [P4129R1](https://wg21.link/p4129r1)<sup>[5]</sup>, which documents the dynamics of WG21 voting, and [P4050R0](https://wg21.link/p4050r0)<sup>[6]</sup>, which documents process failure modes in large-scale standardization. Those papers examine the mechanism. This paper examines the role that operates it - its structural properties and the opportunities they create.

Every decision examined was made by experienced practitioners under real constraints. The intent is to help the committee understand its own structure.

---

## 2. What the System Produces

The committee's achievements are visible in the published record.

**Schedule predictability.** C++14, C++17, C++20, and C++23 shipped on time. Four consecutive on-time releases. No previous era of C++ standardization achieved this. The earlier model produced C++98, then a five-year gap to C++03 (a bug-fix release), then an eight-year gap to C++11. The three-year cadence established by [P1000R6](https://wg21.link/p1000r6)<sup>[11]</sup> transformed C++ from a language that shipped once a decade to one that ships every three years. Voutilainen writes in 2019, in [P0592R3](https://wg21.link/p0592r3)<sup>[23]</sup> - and the committee delivered on every count:

> "C++ ships on time. C++11 shipped late, C++14 shipped on time, C++17 shipped on time, C++20 will ship on time, and C++23 will ship on time."

**Compiler tracking.** By C++17, major compilers were shipping conforming implementations within months of publication. The predictable cadence transformed the relationship between the standard and its implementations. Sutter writes<sup>[24]</sup>:

> "After C++98 shipped, it took 5 years before we saw the first complete conforming compiler that implemented all language features; after C++11 shipped, it took 3 years; when C++14 shipped, it took months."

Sutter writes<sup>[25]</sup>:

> "Never before in the history of C++ has the standard maintained such high specification quality, never before has it released on a regular predictable schedule... and as we removed that quality and schedule uncertainty, it's no accident that never before have all the major compilers tracked the standard so closely, which benefits the whole worldwide C++ community."

The pattern held through C++17. C++20's larger feature set strained implementation timelines.

**Consensus as a brake.** The consensus model prevents premature adoption. The SA votes that blocked `std::execution` for C++23 gave the proposal two additional years of refinement. The contracts removal from C++20 - plenary vote 68/0/4 ([P1823R0](https://wg21.link/p1823r0)<sup>[15]</sup>) - demonstrated that the safety valve works when the evidence is clear. The consensus threshold protects the standard from features that are not ready. Josuttis, Voutilainen, Orr, Vandevoorde, Spicer, and Di Bella write in [P1823R0](https://wg21.link/p1823r0)<sup>[15]</sup>:

> "Part of the train model must be an agreement that if there is not consensus that a feature is in acceptable form when the train is about to depart that the feature must be taken off the train and that it has to catch the next train."

**Volunteer work.** The C++ standard is produced by volunteers. The technical quality of the work - modules, concepts, ranges, coroutines, `std::expected`, `std::print`, `std::mdspan` - speaks for itself. The people who produce this work do so because they care about the language. Stroustrup writes in his foreword to the standard<sup>[26]</sup>:

> "The C++ community is now fortunate enough to have several high-quality implementations that closely approximate the standard... The community owes thanks to the compiler and standard library writers - many of whom are members of the committee or represented there by colleagues."

**Open participation.** Anyone can write a paper. Anyone can attend a meeting. The barriers to participation are time and travel, not credentials. Stroustrup states at CppCon 2021<sup>[27]</sup>:

> "Having an ISO committee allows people from competing organizations to collaborate on important things for their organizations. We have been doing lots of things that couldn't have been done without the committee."

The 2026 convenership transition is itself evidence of the system working. Conveners serve terms. Terms end. New conveners bring new perspectives.

Bjarne Stroustrup, the creator of C++, assessed the pace in [P1962R0](https://wg21.link/p1962r0)<sup>[16]</sup>:

> "We can do much or do less fast. We cannot do both and maintain quality and coherence."

The system produces these results. Its own leaders have noted the constraints. The following sections document the structure that produced them, and what the committee's officers have said about where the pace creates strain.

---

## 3. The Convener's Formal Powers

The WG21 convener's powers are documented in three layers: ISO directives, WG21 standing documents, and the committee's own published descriptions.

### 3.1 ISO Directives

The ISO/IEC Directives, Part 1, as modified by the Consolidated JTC 1 Supplement, define the working group convenor role<sup>[7]</sup>:

> "A working group operates by consensus, reports and gives recommendations, if any, to its parent committee through a Convenor appointed by the parent committee. The Working Group Convenor shall act in a purely international capacity."

Working group convenors are appointed for up to three-year terms with no limit on reappointment<sup>[7]</sup>.

### 3.2 Standing Document 3

[SD-3](https://isocpp.org/std/standing-documents/sd-3-study-group-organizational-information)<sup>[8]</sup> governs study group creation:

> "SGs and their chairs are administratively appointed by the convener at or between WG21 meetings."

The convener also disbands study groups:

> "Once the convener determines that an SG has completed its assigned work or cannot make further progress, he administratively disbands the SG at or between WG21 meetings."

### 3.3 Standing Document 4

[SD-4](https://isocpp.org/std/standing-documents/sd-4-wg21-practices-and-procedures)<sup>[9]</sup> extends the appointment power beyond study groups to all subgroups:

> "Subgroup chairs are appointed by the convener, and are selected to match the current needs of the subgroup. They have no fixed term."

### 3.4 The Committee's Own Description

The committee's public description of its structure states<sup>[10]</sup>:

> "The committee is organized into a three-stage pipeline consisting of several subgroups, each run by the indicated chairperson, operating with the authority of the convenor."

The convener's officer description reads<sup>[10]</sup>:

> "The convener determines consensus, chairs the WG, sets the WG meeting schedule ('convenes' meetings), appoints Study Groups, and is responsible to higher levels of ISO (SC22, JTC1, and ITTF) for the WG's work."

### 3.5 Summary of Documented Powers

The convener's documented powers, drawn from the sources above:

- Appoints all subgroup chairs (SD-4)
- Appoints all study group chairs (SD-3)
- Disbands study groups (SD-3)
- Determines consensus (isocpp.org)
- Chairs plenary (isocpp.org)
- Sets the meeting schedule (isocpp.org)
- All subgroups operate "with the authority of the convenor" (isocpp.org)

[P4129R1](https://wg21.link/p4129r1)<sup>[5]</sup> documents that the chair controls poll wording and that different framings of the same question produce different vote distributions. The convener chairs plenary.

---

## 4. Institutional Wisdom

Filmed interviews for the Boost documentary, conducted in November 2025 at Kona, provide on-camera reflections from senior committee members on how the convener's structural powers operate in practice<sup>[12]</sup><sup>[13]</sup>. These are experienced practitioners sharing institutional knowledge.

### 4.1 On the Importance of Chair Selection

John Spicer, EDG President and US National Body Chair, a committee member for thirty-three years<sup>[12]</sup>:

> "he picks working group chairs, and working group chairs pretty much stay in their position until they don't wanna do it anymore."

> "the chairs are definitely really vital positions and I think, in for technical issues, the chairs probably have more, you know, potentially more influence than [the convener] has over what goes on. So I think the selection of chairs is a very important part of the process."

The convener appoints the chairs.

### 4.2 On Scheduling Power

Jens Maurer, CWG Chair, a committee member since approximately 2000<sup>[13]</sup>:

> "chairs of subgroups get the power to select certain papers that they schedule and others that they don't schedule"

[P4129R1](https://wg21.link/p4129r1)<sup>[5]</sup> documents that scheduling determines who is in the room, and that attendance composition affects vote outcomes.

### 4.3 On the Succession Mechanism

Maurer describes how chairs are currently selected<sup>[13]</sup>:

> "formally, yes, you are chosen by the convener... I don't think there has actually been a competition sort of thing... he usually asks the outgoing chairperson who could be the successor."

Spicer describes how the convener's role itself has evolved<sup>[12]</sup>:

> "the convener's responsibilities were much less than what they are today. And some of that is from procedural changes over the years..."

The role has expanded substantially since its inception. The informal succession mechanism - outgoing chair recommends, convener appoints - has remained consistent.

---

## 5. What the Published Record Shows

The committee's own officers, direction-setters, and implementers have published papers identifying where the strengths documented in Section 2 reach their limits. Each exhibit below is a published WG21 paper authored by senior committee members.

### 5.1 Coherence

The Direction Group writes in [P2000R3](https://wg21.link/p2000r3)<sup>[17]</sup>:

> "We have no shared aims, no shared taste... The alternative is a dysfunctional committee producing an incoherent language."

The same body writes in [P0939R4](https://wg21.link/p0939r4)<sup>[18]</sup>:

> "In the years leading up to the 1998 standard, the committee members often reminded themselves of the story of the Vasa, the beautiful 17th century battleship that sank on its maiden voyage because of (among other things) insufficient work on its foundation and excessive late additions."

The Direction Group published these assessments across four years.

### 5.2 Defect Load

Nina Ranns writes in the 2021 admin telecon minutes ([N4890](https://www.open-std.org/jtc1/sc22/wg21/docs/papers/2021/n4890.pdf))<sup>[19]</sup>:

> "This time we have quite a few 'defects' in C++20 that are too large to be handled via the issues list, so they are being fixed by proposals... At this time, there are no nonexperimental implementations of these features shipped, so this is our last chance to get them fixed before the defects ship."

C++14 and C++17 carried smaller features and produced fewer defects. C++20 carried modules, coroutines, concepts, and ranges. The defects were proportional to the feature load.

### 5.3 Implementation Capacity

The compiler tracking documented in Section 2 held through C++17. Eighteen implementers from GCC, Clang, MSVC, EDG, libc++, and libstdc++ write in [P3962R0](https://wg21.link/p3962r0)<sup>[20]</sup>:

> "When features accumulate faster than they can be fully implemented, integrated, and adjusted, they stack on top of incomplete or inconsistent foundations, further increasing the risk of non-portable code."

> "We would like the committee to consider ways of slowing down the addition of features into the standard to allow implementers to catch up, and to allow the existing features to improve in quality."

### 5.4 Deadline Pressure

Ville Voutilainen writes in [P3608R0](https://wg21.link/p3608r0)<sup>[21]</sup>:

> "And all this with our noses pressed against a deadline."

The three-year cadence was designed to eliminate "now or never" pressure. The pressure changed form.

---

## 6. Opportunities for Visible Fairness

The committee's work depends on trust. Trust depends on transparency. The convenership has been exercised with integrity - the results in Section 2 speak for themselves. At the same time, institutional arrangements that depend on the good character of the officeholder are structurally fragile. A system where appointments, scheduling, and role boundaries are visibly fair earns trust automatically - from current members, from newcomers, and from the broader C++ community watching from outside.

Concentrated appointment authority has genuine advantages. It enables decisive action without political deadlock over chair selection. It provides continuity across the committee's long development cycles. A single officer who knows every subgroup's needs can match chairs to challenges faster than any selection process. These properties may have contributed to the results documented in Section 2. The directions that follow do not replace this authority. They make its exercise visible.

The 2026 convenership transition is an opportunity. Each direction below uses powers the convener already has. None requires a rule change. The committee evaluates the tradeoffs. [P4134R0](https://wg21.link/p4134r0)<sup>[14]</sup> presents Directions G and H with additional detail.

### 6.1 Open Competition for Chair Appointments

The informal succession mechanism documented in Section 4.3 - outgoing chair recommends, convener appoints - has served the committee for decades. An open call for candidates ensures the convener's choice draws from the widest possible pool.

- **What it changes.** Subgroup and study group chairs are selected through a process that includes a public call for candidates and a brief statement of qualifications from each candidate. The convener selects from the candidate pool
- **What it costs.** A selection process that runs once per term

### 6.2 Voluntary Role Separation

Voluntary role separation makes the convenership's independence visible. The separation is a norm, not a restriction.

- **What it changes.** The convener does not simultaneously serve as chairman of a foundation or as an active paper author on proposals under committee consideration. The separation is a voluntary norm
- **What it costs.** The convener foregoes two activities

### 6.3 Scheduling Transparency

Jonathan M&uuml;ller writes in [P3801R0](https://wg21.link/p3801r0)<sup>[22]</sup>: "The paper was forwarded after the feature freeze deadline in a telecon." Published criteria make scheduling decisions visible.

- **What it changes.** Chairs publish their scheduling criteria before each meeting - what papers are scheduled, what papers are deferred, and why
- **What it costs.** A brief pre-meeting note per subgroup

### 6.4 Domain Coverage Checks

- **What it changes.** Before consequential decisions, the chair verifies that stakeholders from the affected domain are present and have had an opportunity to participate. [P4050R0](https://wg21.link/p4050r0)<sup>[6]</sup> describes the full framework
- **What it costs.** One additional question before a consequential poll: "Are the affected stakeholders represented?"

### 6.5 Term Rotations for Chairs

The implementers recommend in [P3962R0](https://wg21.link/p3962r0)<sup>[20]</sup> "longer standardization cycles or alternating feature-focused and consolidation-focused releases." Term rotation is one mechanism the convener can use to pace the workload.

- **What it changes.** Chairs serve defined terms with the expectation of rotation. Reappointment remains available when continuity serves the committee
- **What it costs.** The convener periodically identifies and onboards new chairs

A reasonable observer, examining these arrangements without personal knowledge of the individuals involved, should be able to verify that the process is fair. These measures make that verification straightforward.

The C++ standardization process has produced a language used by millions, through volunteer effort, under constraints that most organizations would find paralyzing - no dedicated funding, no management hierarchy, hundreds of competing interests, and a consensus model that gives every participant a voice. The committee that produced concepts, ranges, coroutines, and modules can produce transparent governance. The tools are already in the convener's hands.

---

## Acknowledgements

The author thanks the Boost documentary team, led by Ray Nowosielski, for the filmed interviews cited in Section 4. The author thanks John Spicer and Jens Maurer for their willingness to describe the committee's structural dynamics on camera - sharing institutional knowledge publicly is itself an act of stewardship. The author thanks Joaqu&iacute;n M L&oacute;pez Mu&ntilde;oz and Peter Dimov for their critique of [P4129R1](https://wg21.link/p4129r1)<sup>[5]</sup>, which informed the analytical approach used here.

---

## References

- [1] Corosio: https://github.com/cppalliance/corosio
- [2] Capy: https://github.com/cppalliance/capy
- [3] [P2469R0](https://wg21.link/p2469r0): "Asio and Sender/Receiver." https://wg21.link/p2469r0
- [4] [P2300R10](https://wg21.link/p2300r10): "`std::execution`." Dominiak, Evtushenko, Baker, Teodorescu, Howes, Shoop, Garland, Niebler, Lelbach. https://wg21.link/p2300r10
- [5] [P4129R1](https://wg21.link/p4129r1): "The Dynamics of Voting in WG21." Falco. https://wg21.link/p4129r1
- [6] [P4050R0](https://wg21.link/p4050r0): "Failure Modes in Large-Scale Standardization." Falco. https://wg21.link/p4050r0
- [7] ISO/IEC Directives, Part 1, Consolidated JTC 1 Supplement 2023, Section 1.12. https://jtc1info.org/wp-content/uploads/2023/11/ISO-IEC-Consolidated-JTC-1-Supplement-2023.pdf
- [8] SD-3: "Study Group Organizational Information." https://isocpp.org/std/standing-documents/sd-3-study-group-organizational-information
- [9] SD-4: "WG21 Practices and Procedures." https://isocpp.org/std/standing-documents/sd-4-wg21-practices-and-procedures
- [10] "The Committee." https://isocpp.org/std/the-committee
- [11] [P1000R6](https://wg21.link/p1000r6): "C++ IS schedule." Sutter. https://wg21.link/p1000r6
- [12] Spicer, J. Filmed interview for the Boost documentary, November 2025, Kona, HI. Filmed by Ray Nowosielski.
- [13] Maurer, J. Filmed interview for the Boost documentary, November 4, 2025, Kona, HI. Filmed by Ray Nowosielski. https://vimeo.com/1140852773/f4f135ef24
- [14] [P4134R0](https://wg21.link/p4134r0): "A Better WG21." Falco. https://wg21.link/p4134r0
- [15] [P1823R0](https://wg21.link/p1823r0): "Remove Contracts from C++20." Josuttis, Voutilainen, Orr, Vandevoorde, Spicer, Di Bella. https://wg21.link/p1823r0
- [16] [P1962R0](https://wg21.link/p1962r0): "How can you be so certain?" Stroustrup. https://wg21.link/p1962r0
- [17] [P2000R3](https://wg21.link/p2000r3): "Direction for ISO C++." Stroustrup, Hinnant, Orr, Vandevoorde, Wong. https://wg21.link/p2000r3
- [18] [P0939R4](https://wg21.link/p0939r4): "Direction for ISO C++." Stroustrup, Sutter, Vandevoorde. https://wg21.link/p0939r4
- [19] N4890: "WG21 2021-05 Admin telecon minutes." Ranns. https://www.open-std.org/jtc1/sc22/wg21/docs/papers/2021/n4890.pdf
- [20] [P3962R0](https://wg21.link/p3962r0): "Implementation reality of WG21 standardization." Ranns, Keane, Serebrennikov, Ballman, Sandoe, Caves, DaCamara, Dos Reis, Brito, Meerwald, Xu, Yaghmour, Miller, Childers, Waffl3x, Cardoso Lopes, Tong, Dionne. https://wg21.link/p3962r0
- [21] [P3608R0](https://wg21.link/p3608r0): "C++26 is not ready." Voutilainen. https://wg21.link/p3608r0
- [22] [P3801R0](https://wg21.link/p3801r0): "Concerns about the design of `std::execution::task`." M&uuml;ller. https://wg21.link/p3801r0
- [23] [P0592R3](https://wg21.link/p0592r3): "To boldly suggest an overall plan for C++23." Voutilainen. https://wg21.link/p0592r3
- [24] Sutter, H. "Trip report: Summer ISO C++ standards meeting (Oulu)." 2016. https://herbsutter.com/2016/06/30/trip-report-summer-iso-c-standards-meeting-oulu/
- [25] Sutter, H. "Trip report: Winter ISO C++ standards meeting." 2016. https://herbsutter.com/2016/03/11/trip-report-winter-iso-c-standards-meeting/
- [26] Stroustrup, B. "Foreword: Serving the C++ Community." https://stroustrup.com/standard-foreword.pdf
- [27] CppCon 2021: "C++ Standards Committee - Fireside Chat Panel." Stroustrup, Sutter, et al. https://www.youtube.com/watch?v=HQ7_jbN0Svc
