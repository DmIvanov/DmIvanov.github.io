---
title: Complexity part 5. Interfaces.
date: 2025-05-12
author: topolog
layout: post
permalink: /complexity-5-interfaces
image:
  path: images-posts/2025-05-12-complexity-5-interfaces/header.jpg
  thumbnail: images-posts/2025-05-12-complexity-5-interfaces/thumb-600.jpg
  caption: Photo by <a href="https://unsplash.com/@komarov?utm_content=creditCopyText&utm_medium=referral&utm_source=unsplash">Komarov Egor 🇺🇦</a> on <a href="https://unsplash.com/photos/a-close-up-of-a-control-panel-in-a-dark-room-sUiUI7FuT34?utm_content=creditCopyText&utm_medium=referral&utm_source=unsplash">Unsplash</a>
tags:
  - architecture
  - complexity
  - platform-agnostic
  - no code
categories:
  - Tech Blog
share: true

---

Software is made of boundaries. From functions to classes, modules to services, systems to organizations — every meaningful split in a codebase depends on one thing: the interface.

An **interface** defines how two parts of a system interact. It tells you what you can rely on, what you need to provide, and what you can expect in return. At its simplest, it might be a function signature. At its most complex, it’s an entire service API, an SDK, or a protocol between two teams in different time zones.

We design interfaces constantly, often without calling them that:

- Defining a function or data structure means deciding what’s public and what’s private.
- Splitting an app into modules shapes how features or services communicate.
- Connecting frontend and backend creates a contract for how data flows between them.
- Integrating third-party tools depends on remote APIs, SDKs, and configuration surfaces.
- UI components, design systems, and CLI tools all define interfaces for usage and composition.

## Why Interfaces Matter for Complexity

Interfaces are everywhere — some explicit, some implicit — and they shape how we write, test, extend, and reason about our systems. More than just technical artifacts like signatures or schemas, interfaces are how we **hide complexity, enforce separation, and communicate intent**. The complexity of a system doesn’t just come from how it works internally — it comes from **how much of that internal mess others have to understand just to use it**. Interfaces are the lever that controls that experience. They’re where complexity either gets absorbed — or spills over. Every time we define or consume an interface, we’re not just connecting code — we’re shaping how people think.

- **Interfaces create abstraction boundaries**  
  They draw the line between what’s inside and what’s outside. A good interface reduces mental load. A bad one forces you to think about too much at once.  
  _([More on abstraction](https://dmtopolog.com/complexity-4-abstraction))_

- **Interfaces act as contracts**  
  They’re promises between parts of the system. The stronger and clearer the contract, the more confidently teams can work independently. Weak or unstable contracts breed uncertainty — and with it, defensive code, extra checks, and over-engineering.

- **Interfaces are where complexity compounds**  
  Especially at team and system boundaries. Misalignment between frontend/backend, client/server, or feature/service often leads to glue code, mismatched assumptions, and integration bugs.

- **Interfaces shape how we use a system**  
  They guide interactions and encode design intent. Clear, consistent interfaces make code easier to use and extend. Confusing ones lead to workarounds, forks, and rewrites.

- **Interfaces define the scope of reasoning**  
  A well-encapsulated interface lets you work with a system without understanding its internals. If you constantly need to peek inside to know how to use it, the interface isn’t doing its job.

- **Interfaces determine coupling**  
  When an interface leaks implementation details, it increases coupling. Tightly coupled systems are harder to change and more fragile over time.


## What Makes an Interface Good or Bad?

Not all interfaces are created equal. Some make systems easy to use, change, and reason about. Others create friction at every step. A good interface reduces complexity — a bad one amplifies it.

Here are some key properties that distinguish good interfaces from bad ones, along with examples that show how they play out in practice.

### 1. **Clarity vs. Ambiguity**

**Good interfaces** are clear in intent and usage. You can tell what they do, what inputs they expect, and what outputs they produce — without reading the internal implementation.

**Bad interfaces** are ambiguous. They require trial-and-error, reading the source, or asking someone else to understand how to use them correctly.

> 🧩 *Example:*
> - ✅ `func sendEmail(to address: EmailAddress)` — clear intent.
> - ❌ `func process(_ input: Any)` — unclear purpose, undefined expectations.


### 2. **Minimal Surface Area vs. Overexposure**

**Good interfaces** expose just enough to do the job — and no more. They present a minimal, focused contract.

**Bad interfaces** leak internal details or expose too many options “just in case,” increasing cognitive load and coupling.

> 🧩 *Example:*
> - ✅ `UserService.fetchProfile(for id: UserID)`
> - ❌ `UserRepository.load(with options: [String: Any], source: DataSourceType, cachePolicy: CacheLevel)`


### 3. **Consistency vs. Surprise**

**Good interfaces** behave consistently with the rest of the system. They follow naming, structure, and behavioral conventions that match other components.

**Bad interfaces** break expectations, introduce some unintended side effects.

> 🧩 *Example:*
> - ✅ A `get()` method that performs a read-only fetch.
> - ❌ A `get()` method that also modifies internal state or triggers side effects.


### 4. **Stability vs. Volatility**

**Good interfaces** are stable over time. They evolve carefully and intentionally.

**Bad interfaces** change frequently — even in backwards-incompatible ways — forcing every dependent component to adapt.

> 🧩 *Example:*
> - ✅ A versioned API with deprecated fields clearly marked.
> - ❌ A shared protocol that changes weekly, breaking multiple modules downstream.


### 5. **Focused Responsibility vs. Overgeneralization**

**Good interfaces** do one thing well. They’re easy to explain in a sentence.

**Bad interfaces** try to be everything to everyone — and end up doing none of it cleanly.

> 🧩 *Example:*
> - ✅ `AnalyticsService.track(event:)`
> - ❌ `CoreSystemManager.handleEventAndState(for user: SessionContext?)`


### 6. **Ease of Use vs. Defensive Usage**

**Good interfaces** guide the user toward correct usage. They make the easy thing the right thing.

**Bad interfaces** require defensive code, special flags, or knowledge of edge cases to avoid misuse.

> 🧩 *Example:*
> - ✅ `ImageLoader.load(from: URL, placeholder: UIImage?)`
> - ❌ `ImageHandler(action: .fetch, config: [:], callback: ((Result<Any, Error>) -> Void)?)`


### 7. **Encapsulation vs. Leakage**

**Good interfaces** hide complexity behind a clean surface.

**Bad interfaces** force consumers to know too much about what's behind the curtain — exposing implementation details, states, or dependencies that don’t belong outside.

> 🧩 *Example:*
> - ✅ `Session.start()`
> - ❌ `Session.prepare(userContext: UserContext, cache: CacheStore, options: [String: Any])`


### 8. **Context-Appropriate vs. Internally Driven**

**Good interfaces** reflect the mental model of the consumer. They speak the language of the problem domain, not the internal implementation.

**Bad interfaces** mirror how the system is built rather than how it’s used — exposing low-level or irrelevant details to the caller.

> 🧩 *Example:*
> - ✅ `Cart.totalPrice(in currency: CurrencyCode)`
> - ❌ `Cart.getAmounts(applyTax: Bool, applyDiscount: Bool, useCachedRate: Bool)`


Good interfaces make code easier to read, use, change, and test. They absorb complexity — so it doesn’t spread. Bad interfaces do the opposite: they force complexity outward, making every consumer deal with it.

That’s why interface design isn’t just a technical detail. It’s one of the highest-leverage tools we have for managing complexity at scale.


## Mitigating Complexity with Better Interfaces

Interfaces are one of the most powerful tools we have for managing complexity — but only if we design and evolve them with care. A poorly thought-out interface pushes complexity onto the user, while a good one hides it, clarifies intent, and enables safe, confident usage. Below are practical guidelines to help you shape interfaces that reduce friction, lower cognitive load, and evolve gracefully over time.

- **Keep them minimal**  
  Expose only what’s necessary. The smaller the interface, the less there is to understand, misuse, or change. Simpler interfaces are easier to learn, test, and maintain.

- **Be consistent**  
  Use consistent naming, parameter order, data shapes, and error handling across all your interfaces. Familiar, predictable patterns reduce mental effort and speed up comprehension.

- **Separate concerns**  
  Avoid multipurpose interfaces. Each interface should have a focused responsibility — doing one thing well. Separation enables reuse without unintended coupling.

- **Make them self-documenting**  
  Choose clear, intention-revealing names, types, and constraints so the interface explains itself at a glance. The best interfaces require minimal external documentation to understand.

- **Invest in documentation**  
  Self-documentation helps, but it’s not enough — especially for shared or public interfaces. Clarify not just *what* the interface does, but *why* it exists and how it should be used.

- **Design for evolution**  
  Interfaces tend to stick around. Design them with future growth in mind. Use techniques like optional parameters, clear versioning, and compatibility layers to support change without disruption.

- **Use deep, not shallow modules**  
  Favor interfaces that offer high value with minimal surface area. A deep module hides complexity behind a simple API (e.g., a `start()` method that kicks off an entire flow), while a shallow one forces the consumer to manage internals themselves.

- **Fail loudly and early**  
  If an interface is used incorrectly, it should produce a clear error — not silently fail or allow invalid input to slip through. Defensive code at the boundary reduces bugs downstream.

- **Stabilize contracts**  
  Once an interface is shared — especially across teams or services — treat it like a contract. Changing it carelessly creates ripple effects that increase complexity everywhere it’s used.

- **Hide internal complexity**  
  Push the messy logic behind the interface. The more implementation details you can abstract away, the simpler it is for consumers to use the interface correctly and confidently.

- **Design for the consumer, not the implementation**  
  Structure the interface around how people use it — not how the system stores or calculates data. Consider the consumer’s mental model, what they know, and what will make their job easiest.

- **Create collaborative design processes**  
  Interfaces shouldn’t be designed in isolation. Run interface or API reviews that include stakeholders from both sides of the boundary — those who build it and those who use it. This reduces the chance of costly mismatches.

- **Test interfaces from the outside**  
  Use integration or contract testing to validate that your interfaces are usable and resilient. If a consumer test is hard to write or read, it may signal that the interface needs simplification.
