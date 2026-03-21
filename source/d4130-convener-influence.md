---
title: "The Convenership's Structural Influence Over WG21"
document: P4130R0
date: 2026-03-21
reply-to:
  - "Vinnie Falco <vinnie.falco@gmail.com>"
audience: WG21
---

## Abstract

The convener appoints every chair. The chairs control every schedule. The schedule determines what ships.

This paper documents the structural powers of the WG21 convenership as exercised from 2002 through 2025, the longest continuous tenure in the committee's history. It traces the documented connections between the convener's formal powers, the Standard C++ Foundation's funding and board composition, corporate affiliations of appointed chairs, and the outcomes of major standardization decisions including coroutines, sender/receiver, and the Networking TS. Every exhibit is drawn from published ISO documents, WG21 standing documents, Foundation disclosures, public corporate records, or filmed testimony from senior committee members. The paper presents documented facts. The conclusions are the reader's.

---

## Revision History

### R0: March 2026 (post-Croydon mailing)

- Initial version.

---

## 1. Disclosure

The author has papers in the WG21 mailing and participates in the committee process. The author developed and maintains [Corosio](https://github.com/cppalliance/corosio)<sup>[1]</sup> and [Capy](https://github.com/cppalliance/capy)<sup>[2]</sup> and is a co-author of [P2469R0](https://wg21.link/p2469r0)<sup>[3]</sup>. The author's preferred asynchronous model competes with [P2300R10](https://wg21.link/p2300r10)<sup>[4]</sup> (`std::execution`). The author supports C++20 stackless coroutines (frame-opaque), the stackful coroutines championed by Nat Goodspeed and Oliver Kowalke ([P0876](https://wg21.link/p0876)<sup>[27]</sup>), and the frame-visible stackless coroutines that the committee rejected. The author's position is that C++ needs all three because each style of coroutine serves a different domain. These opinions did not influence the conclusions of this paper. The author is telling you this so you can calibrate everything that follows.

This paper is a companion to [P4129R1](https://wg21.link/p4129r1)<sup>[5]</sup>, which documents the dynamics of WG21 voting, and [P4050R0](https://wg21.link/p4050r0)<sup>[6]</sup>, which documents process failure modes in large-scale standardization. Those papers examine the mechanism. This paper examines the officer who operates it.

The analysis is structural, not personal. Every decision examined was made by experienced practitioners under real constraints. The intent is to document the published record, not to assign blame.

---

## 2. The Convener's Formal Powers

The WG21 convener's powers are documented in three layers: ISO directives, WG21 standing documents, and the committee's own published descriptions.

### 2.1 ISO Directives

The ISO/IEC Directives, Part 1, as modified by the Consolidated JTC 1 Supplement, define the working group convenor role<sup>[7]</sup>:

> "A working group operates by consensus, reports and gives recommendations, if any, to its parent committee through a Convenor appointed by the parent committee. The Working Group Convenor shall act in a purely international capacity."

Working group convenors are appointed for up to three-year terms with no limit on reappointment<sup>[7]</sup>.

### 2.2 Standing Document 3

[SD-3](https://isocpp.org/std/standing-documents/sd-3-study-group-organizational-information)<sup>[8]</sup> governs study group creation:

> "SGs and their chairs are administratively appointed by the convener at or between WG21 meetings."

The convener also disbands study groups:

> "Once the convener determines that an SG has completed its assigned work or cannot make further progress, he administratively disbands the SG at or between WG21 meetings."

### 2.3 Standing Document 4

[SD-4](https://isocpp.org/std/standing-documents/sd-4-wg21-practices-and-procedures)<sup>[9]</sup> extends the appointment power beyond study groups to all subgroups:

> "Subgroup chairs are appointed by the convener, and are selected to match the current needs of the subgroup. They have no fixed term."

### 2.4 The Committee's Own Description

The committee's public description of its structure states<sup>[10]</sup>:

> "The committee is organized into a three-stage pipeline consisting of several subgroups, each run by the indicated chairperson, operating with the authority of the convenor."

The convener's officer description reads<sup>[10]</sup>:

> "The convener determines consensus, chairs the WG, sets the WG meeting schedule ('convenes' meetings), appoints Study Groups, and is responsible to higher levels of ISO (SC22, JTC1, and ITTF) for the WG's work."

### 2.5 Summary of Documented Powers

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

## 3. The Tenure

Herb Sutter served as WG21 convener from 2002 through 2025, with a brief hiatus in 2008-2009 when P.J. Plauger served for approximately one year<sup>[11]</sup><sup>[12]</sup>. Sutter described the hiatus as "brief" and his tenure as continuous "since 2002"<sup>[13]</sup>. His term expired December 31, 2025. Guy Davidson succeeded him effective January 1, 2026<sup>[13]</sup>.

Twenty-two years. The longest continuous convenership in WG21's history.

During this period, Sutter simultaneously held the following roles:

| Role                                    | Period          | Source                                 |
| --------------------------------------- | --------------- | -------------------------------------- |
| WG21 Convener                           | 2002-2025       | isocpp.org<sup>[10]</sup>              |
| Microsoft, Developer Division           | 2002-2024       | Microsoft press<sup>[14]</sup>         |
| Chairman & President, Std C++ Foundation | 2014-present   | isocpp.org/about<sup>[15]</sup>        |
| Citadel Securities, Technical Fellow    | Nov 2024-present | Sutter blog<sup>[16]</sup>            |
| Paper author (P0709, P0515, P0707, P1000, P3081) | Various | open-std.org<sup>[17]</sup>     |

The convener is expected to "act in a purely international capacity"<sup>[7]</sup>. The convener was employed by Microsoft for 22 of those years.

---

## 4. The Interlocking Boards

Three organizations orbit the C++ standardization process. Their boards overlap.

### 4.1 Standard C++ Foundation

The Standard C++ Foundation is a Washington 501(c)(6) entity. Its board, as listed on [isocpp.org/about](https://isocpp.org/about)<sup>[15]</sup>:

| Name              | Foundation Role            |
| ----------------- | -------------------------- |
| Herb Sutter       | Chairman, President        |
| Bjarne Stroustrup | Treasurer                  |
| Nina Ranns        | Vice-President, Marketing and IP |
| Inbal Levi        | Secretary                  |
| Michael Wong      | Director                   |

The Foundation controls isocpp.org, github.com/cplusplus, github.com/isocpp, and CppCon<sup>[15]</sup>. CppCon has been held annually since 2014<sup>[18]</sup>.

### 4.2 Boost Foundation

The Boost Foundation board, as listed on [sites.google.com/boost.org](https://sites.google.com/boost.org/boost-foundation/contact)<sup>[19]</sup>:

| Name            | Boost Foundation Role                   |
| --------------- | --------------------------------------- |
| Jeff Garland    | Executive Director, Standardization Chair |
| Inbal Levi      | Director, Summer of Code Chair          |

### 4.3 WG21 Direction Group

The Direction Group, as documented in [P2000R5](https://wg21.link/p2000r5)<sup>[20]</sup> and [isocpp.org/std/the-committee](https://isocpp.org/std/the-committee)<sup>[10]</sup>:

| Name                  | Affiliation          |
| --------------------- | -------------------- |
| Bjarne Stroustrup     |                      |
| Roger Orr             |                      |
| Daveed Vandevoorde    | Nvidia               |
| Michael Wong          |                      |
| Jeff Garland          | CrystalClear Software |
| Paul E. McKenney      |                      |

### 4.4 Cross-Membership

| Person          | Std C++ Foundation     | Boost Foundation                | WG21 Role                          | Employer  |
| --------------- | ---------------------- | ------------------------------- | ---------------------------------- | --------- |
| Inbal Levi      | Secretary              | Director, Summer of Code Chair  | LEWG Chair, Israeli NB Chair       | Microsoft |
| Michael Wong    | Director               |                                 | Direction Group, SG14/SG19 Chair   |           |
| Jeff Garland    |                        | Executive Director, Std. Chair  | Direction Group, Vice-Convener, LWG Vice-Chair | CrystalClear |
| Nina Ranns      | Vice-President         |                                 | Vice-Convener, SG22 Chair          |           |
| Bjarne Stroustrup | Treasurer            |                                 | Direction Group                     |           |

The convener appointed the chairs. The Foundation chairman is the convener. The Foundation board members hold chair positions.

---

## 5. The Microsoft Thread

Herb Sutter joined Microsoft's Developer Division on March 13, 2002<sup>[14]</sup>. He left Microsoft on November 11, 2024, after "over 22 years"<sup>[16]</sup>. He was WG21 convener for the entirety of his Microsoft tenure.

### 5.1 Coroutines

Gor Nishanov, a Microsoft engineer, was the primary author of the C++ coroutines proposal<sup>[21]</sup>. The coroutines design uses a stackless, compiler-transform approach.

| Date              | Event                                                        | Source                    |
| ----------------- | ------------------------------------------------------------ | ------------------------- |
| 2014              | Nishanov's coroutines proposal enters WG21                   | P0057<sup>[22]</sup>      |
| 2015 SP2 (2016)   | MSVC ships coroutines TS support - first shipping vendor     | P0912R5<sup>[21]</sup>    |
| 2017              | Clang 5 ships coroutines TS support                          | P0912R5<sup>[21]</sup>    |
| June 2018         | Rapperswil plenary: merge fails (29/15/8, 4 NB opposed)     | N4781<sup>[23]</sup>      |
| November 2018     | San Diego plenary: merge fails (34/17/10, 6 NB opposed)     | N4802<sup>[24]</sup>      |
| July 2019         | Cologne: coroutines merged into C++20                        | Sutter trip report<sup>[25]</sup> |

Bulgaria's national body comment (BG050) raised design concerns. France raised concerns about the heap allocation requirement. Both were rejected<sup>[24]</sup>.

John Spicer, EDG President and US National Body Chair, described the process in a filmed interview<sup>[26]</sup>:

> "Coroutines was developed by Microsoft. Gor [Nishanov]... he's a really brilliant engineer for Microsoft. And he was very enthusiastic about this feature... there was a lot of controversy over exactly, which people had some concerns with his approach and people kept trying to come up with alternate approaches... nobody's come up with something that we like better. So we're gonna go with this."

### 5.2 Stackful Coroutines

[P0876R0](https://wg21.link/p0876r0)<sup>[27]</sup> (`fiber_context`, stackful coroutines) was introduced in February 2018. The proposal has been revised at least twenty times (through R20 as of February 2025). Stackful coroutines are well-understood and multiply-implemented. They have not been standardized.

---

## 6. The Foundation Thread

The Standard C++ Foundation, chaired by Herb Sutter, funded Eric Niebler's development of the ranges library. The range-v3 `CREDITS.md` file acknowledges<sup>[28]</sup>:

| Contributor                  | Contribution                                 |
| ---------------------------- | -------------------------------------------- |
| The Standard C++ Foundation  | A generous grant supporting my Ranges work   |

Niebler went on to become a principal author of [P2300R10](https://wg21.link/p2300r10)<sup>[4]</sup> (`std::execution`, the sender/receiver framework). His career path traversed BoostPro, freelance consulting, Microsoft Research, Facebook/Meta, and Nvidia, where he is currently a Distinguished Engineer<sup>[29]</sup>.

The Foundation also sponsors WG21 meetings. Jens Maurer, the CWG Chair, described the arrangement in a filmed interview<sup>[30]</sup>:

> "The sponsor of this meeting is the C++ Foundation, which is a foundation chaired by Herb Sutter, the convener... And the proceeds from that conference are taken to fund the foundation."

The conference is CppCon. The Foundation is chaired by the convener. The Foundation funds the meetings. The Foundation funded the ranges work. The ranges author became a sender/receiver author.

---

## 7. The P2300 Thread

[P2300R10](https://wg21.link/p2300r10)<sup>[4]</sup> (`std::execution`) was adopted into the C++26 working paper at the St. Louis meeting in July 2024<sup>[31]</sup>. The paper has nine authors.

### 7.1 Authors and Affiliations

| Author                    | Known Affiliation     | WG21 Role                    |
| ------------------------- | --------------------- | ---------------------------- |
| Micha&lstrok; Dominiak    |                       |                              |
| Georgy Evtushenko         |                       |                              |
| Lewis Baker               | Facebook/Meta         |                              |
| Lucian Radu Teodorescu    |                       |                              |
| Lee Howes                 | Facebook/Meta         |                              |
| Kirk Shoop                | Facebook/Meta         |                              |
| Michael Garland           | Nvidia                |                              |
| Eric Niebler              | Nvidia                |                              |
| Bryce Adelstein Lelbach   | Nvidia                | Former LEWG Chair            |

Three authors were at Facebook/Meta. Two were at Nvidia. The LEWG chair - the chair of the subgroup that reviewed P2300 - was simultaneously a co-author of P2300.

### 7.2 The LEWG Chair

Bryce Adelstein Lelbach served as LEWG chair, appointed by the convener. He was a co-author of P2300. He was employed by Nvidia as HPC Programming Models Architect. He later ran for convener<sup>[32]</sup>.

His successor as LEWG chair is Inbal Levi, who is employed by Microsoft, serves as Secretary of the Standard C++ Foundation, and serves as Director of the Boost Foundation<sup>[15]</sup><sup>[19]</sup>.

---

## 8. The Citadel Thread

| Date              | Event                                                      |
| ----------------- | ---------------------------------------------------------- |
| July 2024         | P2300 adopted for C++26 (St. Louis)<sup>[31]</sup>         |
| November 11, 2024 | Sutter joins Citadel Securities<sup>[16]</sup>             |

Four months separate these events.

Sutter wrote from Citadel in April 2025<sup>[33]</sup>:

> "One thing I've appreciated here at Citadel is how aggressively the key advances are being adopted in our live trading systems. We already use C++26's `std::execution` in production for an entire asset class, and as the foundation of our new messaging infrastructure. That's possible because we've had our own in-house implementation running for several years now - thanks, Ga&scaron;per and Bronek!"

And<sup>[33]</sup>:

> "It's illuminating; personally, I'm learning more about `std::execution` now that I'm in an environment where it's being used for real."

Citadel Securities was a CppCon 2024 Silver sponsor. Other financial firms among CppCon 2024 sponsors include Bloomberg (Platinum), Hudson River Trading, Jump Trading, Optiver, and Susquehanna (Gold)<sup>[34]</sup>.

The former convener now works at a company that uses `std::execution` in production and promotes it publicly.

---

## 9. The Networking Deadlock

[N1925](https://www.open-std.org/jtc1/sc22/wg21/docs/papers/2005/n1925.pdf)<sup>[35]</sup> proposed networking for C++ in December 2005. The Networking TS, based on Boost.Asio (Chris Kohlhoff), was published as ISO/IEC TS 19282:2018. It has not been merged into the C++ standard.

The October 2021 electronic polls ([P2453R0](https://wg21.link/p2453r0)<sup>[36]</sup>) produced the following results on the same topic, in the same poll cycle, from the same voters:

| Poll                                          | SF | WF |  N | WA | SA | Outcome             |
| --------------------------------------------- | -- | -- | -- | -- | -- | ------------------- |
| Asio is a good basis for most async use cases |  5 | 10 |  6 | 14 | 18 | Weak consensus agst |
| S/R is a good basis for most async use cases  | 24 | 16 |  3 |  6 |  3 | Consensus in favor  |
| Stop pursuing Networking TS                   | 13 | 13 |  8 |  6 | 10 | No consensus        |
| Networking should be based on S/R             | 17 | 11 | 10 |  4 |  6 | Weak consensus      |
| Acceptable to ship networking without TLS     |  9 | 13 |  5 |  6 | 13 | No consensus        |

The committee voted against Asio as a general async model. The committee voted for sender/receiver as a general async model. The committee could not reach consensus to stop pursuing the Networking TS. The committee could not reach consensus to ship networking without TLS.

The practical effect: C++ has had no standard networking library for twenty-one years.

The beneficiaries of sender/receiver standardization include the employers of the P2300 authors (Section 7). The Networking TS, which would have standardized a competing asynchronous model, remains blocked.

---

## 10. Chair Appointment Patterns

All subgroup chairs are appointed by the convener (Section 2). Under Sutter's tenure, the following chairs served in key positions. Affiliations are from public records as of January 2026<sup>[10]</sup>.

### 10.1 LEWG (Library Evolution)

LEWG reviews P2300 and networking proposals.

| Chair                     | Affiliation | Other Roles                                                  |
| ------------------------- | ----------- | ------------------------------------------------------------ |
| Bryce Adelstein Lelbach   | Nvidia      | P2300 co-author, convener candidate                          |
| Inbal Levi                | Microsoft   | Std C++ Foundation Secretary, Boost Foundation Director, Israeli NB Chair |

### 10.2 EWG (Evolution)

| Chair                | Affiliation | Notes                                      |
| -------------------- | ----------- | ------------------------------------------ |
| Ville Voutilainen    |             | Authored P2464 arguing against Networking TS |
| Erich Keane          | Nvidia      |                                            |

### 10.3 LWG (Library Wording)

| Chair            | Affiliation          | Other Roles                                              |
| ---------------- | -------------------- | -------------------------------------------------------- |
| Jonathan Wakely  | IBM                  |                                                          |
| Jeff Garland     | CrystalClear Software | Boost Foundation Executive Director, Direction Group, Vice-Convener |

### 10.4 CWG (Core Language Wording)

| Chair       | Affiliation | Notes                                                  |
| ----------- | ----------- | ------------------------------------------------------ |
| Jens Maurer |             | Committee member since ~2000, acknowledged social dynamics on camera |

### 10.5 Corporate Concentration

| Corporation       | Positions Held                                                                  |
| ----------------- | ------------------------------------------------------------------------------- |
| Nvidia            | EWG Chair, Direction Group rotating chair, former LEWG Chair, P2300 co-authors  |
| Microsoft         | LEWG Chair, former Convener (22 years)                                          |
| Citadel Securities | Former Convener (joined November 2024)                                         |

---

## 11. What the Witnesses Say

Filmed interviews for the Boost documentary, conducted in November 2025 at Kona, provide on-camera testimony from senior committee members<sup>[26]</sup><sup>[30]</sup><sup>[37]</sup>. These are first-person statements, not hearsay.

### 11.1 On Chair Appointments

John Spicer, EDG President and US National Body Chair, a committee member for thirty-three years<sup>[26]</sup>:

> "he picks working group chairs, and working group chairs pretty much stay in their position until they don't wanna do it anymore."

Jens Maurer, CWG Chair, a committee member since approximately 2000<sup>[30]</sup>:

> "formally, yes, you are chosen by the convener... Herb Sutter has been convener for ages and Herb Sutter essentially appoints those chair roles... I don't think there has actually been a competition sort of thing... he usually asks the outgoing chairperson who could be the successor."

Two independent witnesses. The same mechanism described. No open competition.

### 11.2 On the Power of Chairs

Spicer<sup>[26]</sup>:

> "the chairs are definitely really vital positions and I think, in for technical issues, the chairs probably have more, you know, potentially more influence than Herb has over what goes on. So I think the selection of chairs is a very important part of the process."

The convener appoints the chairs. The chairs have more technical influence than the convener. The selection of chairs is "a very important part of the process."

### 11.3 On the Expansion of the Convener Role

Spicer<sup>[26]</sup>:

> "when Herb first started as convener, the convener's responsibilities were much less than what they are today. And some of that is from procedural changes over the years... But also part of that is, you know, kind of what Herb brought into the process and how he did the job."

### 11.4 On Favoritism and Scheduling

Maurer was asked whether proposals advance based on personal relationships rather than technical merit. He paused. He grimaced. He said<sup>[30]</sup>:

> "the question of whether there's favoritism towards certain proposals regarding personal acquaintances certainly needs careful choice of words in the response"

He then described the mechanism<sup>[30]</sup>:

> "chairs of subgroups get the power to select certain papers that they schedule and others that they don't schedule... being good friends with a chair is helpful in these situations"

### 11.5 On the Separation of Roles

Spicer acknowledged the tension<sup>[26]</sup>:

> "He's been a technical contributor and he is also been the convener and I think he does a pretty good job of trying to keep those things separate, as separate jobs, because you don't want, nobody wants him to be using his position to influence technical position or at least something that he's presenting."

### 11.6 On the Political Perception

Daisy Hollman, Sandia National Labs<sup>[37]</sup>:

> "I think there are people who would sit in this chair and say Herb is a very good politician and knows the right things to say in order to get his agenda accomplished. And I genuinely don't believe that."

Hollman also described an instance of the convener exercising authority over a subordinate<sup>[37]</sup>:

> "I did because I went over Jon's head and told Herb that I wasn't gonna do it unless. And Herb said, we're gonna do this."

### 11.7 On Corporate Proposals

Hollman<sup>[37]</sup>:

> "there's not as much funny business going on there as you think. I think that quite often the reason why proposals from Microsoft, from Google, from Apple see a lot of attention is because they are good."

---

## 12. What the Evidence Does Not Show

The evidence presented in this paper is structural. It documents correlations, not causation. The following limitations apply.

There is no public evidence that Sutter voted on proposals in subgroups or used his position to direct specific technical outcomes. Spicer's testimony (Section 11.5) states that Sutter "does a pretty good job of trying to keep those things separate." Hollman's testimony (Section 11.6) explicitly defends Sutter against the political characterization.

The coroutines design had genuine technical merit. Nishanov's proposal accumulated years of implementation experience across multiple compilers before adoption. As Spicer noted: "he's a really brilliant engineer" and "nobody's come up with something that we like better"<sup>[26]</sup>.

P2300 had broad committee support. The October 2021 poll showed 24 SF and 16 WF votes for sender/receiver as an async basis<sup>[36]</sup>. That support extended well beyond any single company.

The C++ expert community is small. Overlapping roles are partly inevitable when the same people who build libraries also serve on foundation boards and chair subgroups. Talented people work at major companies. The Foundation funds good work. Not every connection implies coordination.

Sutter's train model - the three-year release cadence established by [P1000R6](https://wg21.link/p1000r6)<sup>[38]</sup> - was genuinely beneficial for C++ delivery. The predictable schedule transformed C++ from a language that shipped once a decade to one that ships every three years.

The consensus model's structural properties, documented in [P4129R1](https://wg21.link/p4129r1)<sup>[5]</sup>, explain many outcomes without requiring personal influence. Chair discretion over poll wording, attendance-dependent outcome reversals, and the disproportionate weight of SA votes under the consensus threshold are structural features of the mechanism, not evidence of manipulation.

Jeff Garland's Boost Foundation and Direction Group roles predate the current convener transition. His appointment as Vice-Convener reflects continuity, not necessarily patronage.

Correlation is not causation. The reader who examines the record in Sections 2 through 11 will form their own assessment of what the correlations mean.

---

## Acknowledgements

The author thanks the Boost documentary team, led by Ray Nowosielski, for the filmed interviews cited in Section 11. The author thanks John Spicer, Jens Maurer, Daisy Hollman, and the other interview subjects for their candor on camera. The author thanks Joaqu&iacute;n M L&oacute;pez Mu&ntilde;oz and Peter Dimov for their critique of [P4129R1](https://wg21.link/p4129r1)<sup>[5]</sup>, which informed the analytical approach used here.

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
- [11] Sutter, H. "September 2008 ISO C++ Standards Meeting." Blog post, October 28, 2008. https://herbsutter.com/2008/10/28/september-2008-iso-c-standards-meeting-the-draft-has-landed-and-a-new-convener/
- [12] Sutter, H. "Trip Report: October 2009 ISO C++ Standards Meeting." Blog post, November 11, 2009. https://herbsutter.com/2009/11/11/trip-report-october-2009-iso-c-standards-meeting/
- [13] Sutter, H. "Trip Report: November 2025 ISO C++ Standards Meeting (Kona, USA)." Blog post, November 10, 2025. https://herbsutter.com/2025/11/10/trip-report-november-2025-iso-c-standards-meeting-kona-usa/
- [14] "ISO/ANSI C++ Standards Committee Secretary Herb Sutter Joins Microsoft's Developer Division." Microsoft News, March 13, 2002. https://news.microsoft.com/2002/03/13/isoansi-c-standards-committee-secretary-herb-sutter-joins-microsofts-developer-division/
- [15] "About the Standard C++ Foundation." https://isocpp.org/about
- [16] Sutter, H. "A New Chapter, and Thoughts on a Pivotal Year for C++." Blog post, November 11, 2024. https://herbsutter.com/2024/11/11/a-new-chapter-and-a-pivotal-year-for-cpp/
- [17] Papers authored or co-authored by Herb Sutter: [P0709R4](https://wg21.link/p0709r4) (Zero-overhead deterministic exceptions), [P0515R3](https://wg21.link/p0515r3) (Consistent comparison, with Maurer and Brown), [P0707R0](https://wg21.link/p0707r0) (Metaclasses), [P1000R6](https://wg21.link/p1000r6) (C++ IS schedule), [P3081R2](https://wg21.link/p3081r2) (Core safety profiles). https://open-std.org
- [18] "Standard C++ Foundation Annual Report for Fiscal Year 2024." https://isocpp.org/blog/2024/10/standard-cpp-foundation-annual-report-for-fiscal-year-2024
- [19] "Boost Foundation - Contact." https://sites.google.com/boost.org/boost-foundation/contact
- [20] [P2000R5](https://wg21.link/p2000r5): "Direction for ISO C++." Stroustrup, Hinnant, Orr, Vandevoorde, Wong. https://wg21.link/p2000r5
- [21] [P0912R5](https://wg21.link/p0912r5): "Merge Coroutines TS into C++20." Nishanov. https://wg21.link/p0912r5
- [22] [P0057R5](https://wg21.link/p0057r5): "Wording for Coroutines." Nishanov, Maurer, Smith, Vandevoorde. https://wg21.link/p0057r5
- [23] N4781: WG21 2018-06 Rapperswil Minutes. https://www.open-std.org/jtc1/sc22/wg21/docs/papers/2018/n4781.pdf
- [24] N4802: WG21 2018-11 San Diego Minutes. https://www.open-std.org/jtc1/sc22/wg21/docs/papers/2019/n4802.pdf
- [25] Sutter, H. "Trip Report: Summer ISO C++ Standards Meeting (Cologne)." Blog post, July 20, 2019. https://herbsutter.com/2019/07/20/trip-report-summer-iso-c-standards-meeting-cologne/
- [26] Spicer, J. Filmed interview for the Boost documentary, November 2025, Kona, HI. Filmed by Ray Nowosielski.
- [27] [P0876R0](https://wg21.link/p0876r0): "`fiber_context` - fibers without scheduler." Kowalke. https://wg21.link/p0876r0
- [28] range-v3 CREDITS.md. https://github.com/ericniebler/range-v3/blob/master/CREDITS.md
- [29] Niebler, E. "About." https://ericniebler.com/about/
- [30] Maurer, J. Filmed interview for the Boost documentary, November 4, 2025, Kona, HI. Filmed by Ray Nowosielski. https://vimeo.com/1140852773/f4f135ef24
- [31] Sutter, H. "Trip Report: Summer ISO C++ Standards Meeting (St Louis, MO, USA)." Blog post, July 2, 2024. https://herbsutter.com/2024/07/02/trip-report-summer-iso-c-standards-meeting-st-louis-mo-usa/
- [32] Lelbach, B. A. "C++ Convenor." Campaign page. https://brycelelbach.github.io/cpp_convenor/
- [33] Sutter, H. "Living in the Future: Using C++26 at Work." Blog post, April 23, 2025. https://herbsutter.com/2025/04/23/living-in-the-future-using-c26-at-work/
- [34] CppCon 2024 sponsor listings. https://cppcon.org
- [35] N1925: "Networking Library Proposal for TR2." December 4, 2005. https://www.open-std.org/jtc1/sc22/wg21/docs/papers/2005/n1925.pdf
- [36] [P2453R0](https://wg21.link/p2453r0): "2021 October Library Evolution Poll Outcomes." https://wg21.link/p2453r0
- [37] Hollman, D. Filmed interview for the Boost documentary, November 2025, Kona, HI. Filmed by Ray Nowosielski.
- [38] [P1000R6](https://wg21.link/p1000r6): "C++ IS schedule." Sutter. https://wg21.link/p1000r6
