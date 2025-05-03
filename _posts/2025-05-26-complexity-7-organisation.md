---
title: Complexity part 7. Organisation.
date: 2025-05-26
author: topolog
layout: post
permalink: /complexity-7-organisation
image:
  path: images-posts/2025-05-26-complexity-7-organisation/header.jpg
  thumbnail: images-posts/2025-05-26-complexity-7-organisation/thumb-600.jpg
  caption: Photo by <a href="https://unsplash.com/@enginakyurt?utm_content=creditCopyText&utm_medium=referral&utm_source=unsplash">engin akyurt</a> on <a href="https://unsplash.com/photos/a-blue-sign-on-a-beach-fpLoxANQt0A?utm_content=creditCopyText&utm_medium=referral&utm_source=unsplash">Unsplash</a>
      
tags:
  - complexity
  - platform-agnostic
  - organisation
  - no code
categories:
  - Tech Blog
share: true
---

Misalignment and miscommunication are major, often underestimated sources of complexity in software projects. While previous chapters explored complexity within the codebase and within ourselves, this one shifts focus to the social layer: how teams, stakeholders, and individuals interact — or fail to — and how that influences the systems we build.


## Misalignment: When We Don’t Share the Same Mental Model

Misalignment happens when people involved in a project hold different understandings of what problem they’re solving, what success looks like, or how to get there. It’s one of the most subtle yet pervasive sources of accidental complexity — and it often goes unnoticed until the cost shows up in code.

What starts as a simple feature request can become an entangled mess when stakeholders, designers, and developers all interpret the task differently. Without a shared mental model, we build systems that reflect our assumptions — not the actual problem.

### Vision Misalignment

One of the most fundamental forms of misalignment is simply not agreeing on what we’re building or why. Different stakeholders — product managers, designers, developers, leadership — may have conflicting interpretations of the goals, user needs, or constraints.

- Sometimes, there’s no clearly defined problem at all — just an assumption that something “should” be done.
- In other cases, long-term strategic goals clash with short-term business needs.
- “Business rules” might be defined in multiple places — and none of them match.
- Product decisions are driven by opinions or hunches rather than data, leading to design churn and unclear priorities.

### Misaligned Priorities

Even when the vision is clear, different roles often optimize for different outcomes:

- Developers want clean code and test coverage, assuming they’re building production-ready systems.
- Product managers focus on MVP speed and user validation — assuming the work is disposable.
- One team might be optimizing for performance, another for flexibility, a third for time-to-market.

These differences aren’t inherently bad — they reflect healthy tension. But without explicit alignment, they lead to subtle mismatches in expectations, scope creep, and rushed compromises.

> Example: A developer spends extra time implementing a flexible architecture for future extensibility. The PM, assuming this is a throwaway prototype, is frustrated by the delay — and the result pleases no one.

### Ambiguity and Vague Requirements

A more granular form of misalignment appears in vague or ambiguous requirements. This often stems from incomplete transfer of context between those who define problems (PMs, designers) and those who implement solutions (developers).

- Assumptions go unstated.
- Domain details are missing or under-explained.
- Critical constraints are implied, not documented.
- Non-functional requirements are forgotten (security, performance, accessibility)

When this happens, developers are left to interpret the feature based on their own mental models. The result may function — but not match the intent.

> Example: A ticket says “add a price filter.” The developer adds a min–max input field. Later it turns out the business needed predefined price brackets (e.g., “Under $100”), and the implementation doesn’t fit the UI or backend model.

### Organizational Silos

Organizational silos are another form of misalignment — but instead of differing opinions, they stem from **a lack of shared visibility**. Different teams or roles work toward a common goal but do so in parallel, without enough coordination or understanding of each other's work. Everyone might be technically aligned on *what* to build, yet disconnected on *how*, *why*, or *under which constraints*. The result is a fractured system that’s more complex than it needs to be — not because the problem is hard, but because the communication is sparse.

You’ll see this when frontend and backend teams design an API without agreeing on data structures. When product and engineering align on features but not on edge cases. When designers hand off polished mockups that assume capabilities the platform doesn’t support. Or when QA and devs test and release features on different timelines. Each of these scenarios leads to misfit parts that need glue code, translation layers, or workarounds — all of which increase complexity without adding business value.

Silos often emerge naturally as teams specialize or grow. But the core issue is always the same: **lack of shared understanding in real time**. When teams don’t talk early and often, they make assumptions. And when those assumptions diverge, complexity fills the gap — not with better solutions, but with fragile connectors that patch over the misalignment.

###  Consequences of Misalignment

Misalignment doesn’t just lead to frustration or delays — it directly adds accidental complexity to the system. Here’s how:
- **Redundant logic:** Different teams or individuals implement the same rules in slightly different ways — leading to duplication, drift, and inconsistency across modules.
- **Glue code and workarounds:** Misunderstandings between teams (e.g. between frontend and backend, or between services) often result in a tangle of adapters, transformers, and “glue” logic that doesn’t solve a business problem — it just reconciles mismatched assumptions.
- **Overgeneralized solutions:** When trying to cover everyone’s interpretation of a problem, we often end up building something overly flexible — with extra flags, parameters, conditionals, or configuration options that few fully understand or use.
- **Rewrites and rework:** Code built under the wrong assumptions often needs to be torn down and redone, introducing churn and technical debt that could’ve been avoided with clearer alignment from the start.
- **Hidden complexity:** The reasons behind certain decisions become hard to trace, especially when they originate from conflicting or unclear expectations. Future developers are left guessing why something was built the way it was — and often afraid to touch it.

>❗ Misalignment doesn’t always show up as a visible bug. It shows up as extra logic, fragile workarounds, mismatched interfaces, and an uneasy feeling that “this is more complicated than it needs to be.”

The bottom line: every misunderstanding upstream multiplies complexity downstream. The longer it goes unaddressed, the more code it takes to compensate for it.


## Inconsistency: The Hidden Complexity Multiplier

Inconsistency across a project or organization might seem like a minor issue — a formatting style here, a naming variation there — but over time, it becomes a silent multiplier of complexity. It introduces friction in understanding, navigating, and contributing to code, particularly in growing teams or large codebases. Even when the code is technically correct, inconsistency forces developers to re-learn patterns, second-guess intentions, or stop to decipher why something was done differently.

You’ll see it in mismatched naming conventions, structural patterns that vary between modules, or different implementations of the same logic across features. One part of the app uses async/await, another uses callbacks. Some modules use MVVM, others don’t follow any pattern. Even small differences — like how date formatting is handled or where configuration lives — can add up.

Why is inconsistency so expensive? Because humans rely heavily on **pattern recognition** to reduce cognitive load. When patterns are familiar and predictable, we move faster and with more confidence. But inconsistency breaks that flow and introduces extra cognitive effort.

Here’s how inconsistency increases complexity:

- **Increased mental overhead**: Developers must constantly switch mental models when moving between modules or teams that barely resemble one another.
- **More room for error**: Inconsistent naming or structure makes it easier to misunderstand what a piece of code does — or miss important edge cases.
- **Harder navigation**: Without shared structure or naming, finding where something lives in the codebase takes longer.
- **Redundant discussions and PR churn**: Inconsistent style leads to debates in code reviews that could be solved by shared rules.
- **Slower onboarding**: Newcomers have more to learn, more context to absorb, and more habits to unlearn when every part of the system looks and behaves differently.
- **Decision fatigue**: Developers spend time making decisions about formatting, naming, or architecture that could be solved once through agreement or codestyle guides.

Consistency is more than just aesthetic — it's a design decision that makes software easier to understand, evolve, and scale. Establishing shared conventions, architectural patterns, naming rules, and tooling (like linters, templates, and CI pipelines) helps protect against unnecessary complexity. The goal isn’t rigid standardization, but clarity: when developers move across the codebase, they shouldn’t feel like they’ve entered a different world. Familiarity lowers mental load — and that’s one of the simplest ways to keep complexity in check.


## Organizational Design Shapes System Design

One of the most profound — yet often overlooked — sources of complexity is how our teams are structured. As Conway’s Law famously states:

> “Any organization that designs a system will produce a design whose structure is a copy of the organization's communication structure.”

In other words, your architecture mirrors your org chart. When teams are siloed, their systems become siloed. When teams struggle to collaborate, their code reflects that friction. And when teams are optimized for autonomy, their modules become independent — sometimes overly so.

This isn’t always bad. Conway’s Law isn’t a warning — it’s a description of reality. Aligning architecture with team boundaries can actually reduce coordination overhead. But the downside is clear: **organizational complexity breeds system complexity**. For example:

- A feature that spans multiple teams may require coordination between frontend, backend, infrastructure, and design — each with their own roadmaps and priorities.
- To respect team boundaries, logic might be split across services or modules that aren’t naturally separate — introducing glue code, abstractions, and coordination mechanisms.
- Autonomy can lead to redundancy — different teams solving similar problems in different ways, with different tools or assumptions.

What begins as a strategy to scale teams can lead to fragmented systems with duplicated logic, excessive generalization, and complex workflows that serve the organization’s shape more than the user’s needs.

Understanding this relationship helps us make better decisions. If we know the system will reflect the structure of our teams, we can shape those structures with intention. Cross-functional teams, clear shared ownership, and regular syncs between collaborators reduce the architectural scars of organizational misalignment.

And when reorganization isn't feasible, **awareness itself becomes a tool**. If we know that our team boundaries introduce complexity, we can invest in simplifying the seams: shared contracts, well-defined APIs, good documentation, and thoughtful glue code. Because sometimes, the best way to fix the architecture… is to fix how we work together.


## The Alignment–Chaos Spectrum

When it comes to organizational and system design, we are always balancing between two forces: **alignment** and **chaos**.

- Too little alignment, and we descend into chaos — duplicated solutions, incompatible modules, inconsistent APIs, and endless miscommunication.
- Too much alignment, and we fall into rigidity — stifling flexibility, creating one-size-fits-all solutions, and forcing awkward fits for problems that don’t belong in the same mold.

**Neither extreme is healthy.** Complexity grows both when teams are too independent and when they are too tightly bound.

In a startup, a bit of chaos is survivable — even necessary. Agility matters more than perfect structure. In a large corporation, alignment is critical to avoid duplicated effort and to maintain coherence across massive systems. But even there, too much standardization can become its own kind of complexity: developers spend more time conforming to rules than solving real problems.

Finding the right point on the alignment–chaos spectrum is not about enforcing rigid standards or letting everything evolve organically. It’s about **choosing where consistency matters most** — and where flexibility should be preserved.

Some helpful questions to guide these choices:
- Which areas are critical for our product (UI, security, fast feature delivery, API design, rich documentation)?
- Where does inconsistency create real cognitive load for developers or users?
- Where does forced consistency create unnecessary complexity or cost?
- Which areas can tolerate divergence (internal module structure, small utility patterns)?

Alignment should focus on **principles and shared understanding**, not on micromanaging every decision. For example:
- Agreeing that all APIs should be versioned, even if different services use slightly different tooling.
- Agreeing that features should be modular and testable, without forcing the same exact folder structure everywhere.
- Agreen that all the features of the mobile app use common modules/libraries/approaches for networking, encryption, analytics, UI animation, and so on.

When we treat alignment and autonomy not as absolutes but as a spectrum to be tuned based on context, we gain a powerful tool to manage complexity consciously — rather than reactively.

### The Challenge of Scaling Alignment

When an organization grows, two natural forces come into play:

- **Autonomy needs to increase** to keep teams agile and motivated. Each team needs the space to make local decisions without endless cross-team negotiation.
- **Alignment becomes harder** because there are more moving parts, more diverse needs, and longer communication paths.

Finding the right balance between alignment and flexibility isn’t a one-time decision. As organizations scale, what once made sense at 5 people might break down at 50, and what worked at 50 might crumble at 500. New teams form, responsibilities shift, and technical debt compounds. Alignment must be treated as an evolving, living system — something we revisit regularly to keep complexity manageable without stifling autonomy.

### Familiarity Is Not Simplicity

When you've been in the same organization or working on the same project for a long time, you naturally adapt to its quirks — even the overly complex ones. But just because something *feels* familiar doesn’t mean it’s *simple*. Familiarity masks complexity. You stop noticing the awkward workflows, overloaded abstractions, or brittle architecture — not because they’ve become better, but because you've built mental shortcuts around them.

That’s why it’s important to listen to newcomers. Fresh eyes often spot convoluted patterns or redundant processes that long-timers no longer question. If they point out areas of confusion or unnecessary indirection, take it seriously — it might be time to clean things up.

A related trap is unclear responsibility. When ownership boundaries aren't well defined, work gets messy — not just in the code, but across the whole development process. And messy systems, organizational or technical, are always harder to navigate, reason about, and maintain.


## Strategies for Evolving Alignment

- **Principle-Based Governance**  
  Instead of hard rules for every decision, define a few strong principles that guide teams in making their own aligned choices (e.g., "Favor explicit over implicit dependencies" or "Prioritize local reasoning in APIs").

- **Lightweight Alignment Forums**  
  Create spaces (like architecture circles, guilds, or working groups) where interested developers discuss and update shared practices — without heavy-handed command-and-control.

- **Versioning and Layered Contracts**  
  Allow systems to evolve by versioning APIs, internal modules and services, and shared libraries. Not everything has to align instantly — progressive adoption is key.

- **Clear Boundaries and Ownership**  
  Explicitly define system ownership. When boundaries are clear, teams can innovate internally while maintaining stable interfaces externally.

- **Celebrating Simplicity**  
  Recognize and reward teams that *remove* complexity, not just those who add impressive-looking abstractions. Make simplicity part of your technical culture.

Ultimately, alignment isn't about creating the One True Way. It's about minimizing the unnecessary complexity that grows when communication, expectations, and responsibilities drift apart — while leaving space for the necessary complexity that arises from solving real-world problems.


## Conclusion

Misalignment, inconsistency, and siloed collaboration don’t just slow us down — they silently shape our systems, making them harder to understand, evolve, and maintain. Complexity doesn’t come only from within the code — it emerges from the spaces between people: from miscommunication, vague expectations, inconsistent decisions, and disconnected teams. These invisible dynamics leave very real traces in every layer of our systems.

If we want to reduce complexity, we can’t focus solely on technical design. We have to shape the environment in which software is designed. That means building strong feedback loops, aligning on principles over rigid processes, and fostering a culture of collaboration and mutual understanding. It means recognizing that organizational structure, communication practices, and cultural defaults all influence the clarity — or confusion — of what we build.

Every organization must find its own balance between alignment and chaos. Too little alignment, and inconsistency and redundancy take over. Too much, and flexibility disappears. But if we invest in clear boundaries, shared principles, and continuous conversation, alignment becomes not a constraint, but a force multiplier. The real challenge isn’t just writing better code — it’s creating the conditions where better code can be written.
