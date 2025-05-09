---
title: Complexity part 6. Human nature.
date: 2025-05-19
author: topolog
layout: post
permalink: /complexity-6-human-nature
image:
  path: images-posts/2025-05-19-complexity-6-human-nature/header.jpg
  thumbnail: images-posts/2025-05-19-complexity-6-human-nature/thumb-600.jpg
  caption: Photo by <a href="https://unsplash.com/@kutyawka?utm_content=creditCopyText&utm_medium=referral&utm_source=unsplash">Ekaterina Kuznetsova</a> on <a href="https://unsplash.com/photos/dummy-near-wall-GSoA6JYx2TQ?utm_content=creditCopyText&utm_medium=referral&utm_source=unsplash">Unsplash</a>
      
tags:
  - complexity
  - platform-agnostic
  - human-factor
  - psychology
  - no code
categories:
  - Tech Blog
share: true
---

When we talk about complexity in software, we often focus on technical causes — sprawling architectures, tight coupling, or poorly structured code. But underneath all of that lies a deeper source: human nature itself. The way we think, reason, decide, and collaborate directly shapes the systems we build. This article explores the human side of software complexity, not to eliminate it, but to understand it better and work with it more intentionally.

## Cognitive Limits

The entire topic of complexity in software ultimately stems from a fundamental truth: our cognitive capacity is limited.

Our working memory can only hold a small number of items at once — a constraint famously known as the 7±2 rule. (_We've shared more on that in the [introduction to this topic](/complexity-0-introduction)_)

Because of this, we’re forced to organize code in ways that reduce the number of things we have to think about at the same time. That’s why we spend so much time discussing how to design our modules, structure abstractions, and align the level of detail with the context in which the code is used. (_We’ve explored these topics in earlier chapters: [part 1](/complexity-1-decisions-in-code), [part 2](/complexity-2-logic-code-disctribution), [part 3](/complexity-3-problem-solution-mismatch), [part 4](/complexity-4-abstraction), [part 5](/complexity-5-interfaces), check them out for a deeper dive._)

But memory limitations are only one part of the story. There are several other traits of human cognition that shape how we write and experience code:

### Difficulty with State Tracking

Humans are notoriously bad at holding multiple possible states in mind — especially when those states are implicit. When reading code, we often have to mentally simulate different paths: "What happens if this flag is true?", "Is this variable already set?", "Which branch are we in right now?" As the number of possible states grows, our ability to reason about the system drops sharply. This is why deeply stateful systems, boolean flags, and side-effect-heavy logic can become so cognitively expensive — even when they’re technically correct.

### Poor Handling of Indirection

Indirection is a powerful tool — we use it to make code modular, abstract, and reusable. But every level of indirection makes it harder to trace what’s actually happening. When a function calls a method on a dependency, which is a protocol-backed abstraction that calls another injected service, we’ve added multiple hops for the brain to follow. Protocols, callbacks, dependency injection, and decorators are all useful — but they come at a cost. Too much indirection breaks our intuitive grasp of control flow and turns even simple tasks into a scavenger hunt.

### Context Switching Is Expensive

Whenever we switch from one file, type, or layer to another, we need to load a new mental model. That mental juggling slows us down and increases the chance of mistakes. If your current task requires bouncing between the view, the model, two unrelated services, and a shared utility, the mental overhead becomes significant. This is why we care so much about **cohesion** and **coupling**. When related logic is kept close together, and unrelated logic is clearly separated, we can stay focused on a task without constantly reorienting ourselves.

### Preference for Linear Thinking

We’re much better at understanding sequences than branches. Code that reads like a clear, step-by-step story is far easier to follow than logic that fans out into deeply nested conditionals or recursive calls. When possible, flat, linear flows reduce mental strain. This doesn’t mean avoiding branching entirely, but it does mean we should be mindful of how deep and how scattered that branching becomes — especially when the logic is spread across multiple files or functions.

### Cognitive Fatigue

Even the clearest code becomes hard to work with when we’re tired, stressed, or under pressure — which is often the case in real development. What seems like a minor complexity in the morning can feel like a blocker at 6 PM after a day of meetings and bug triaging. This is why we should aim to write code that’s easy to understand not just in theory, but in practice — under less-than-ideal conditions. Clear, localized, well-named logic helps future you (or your teammates) work more effectively when mental energy is low.

---

## Cognitive Biases

All human beings are biased. We don’t perceive the world exactly as it is — we filter it through years of experience, instinct, and evolved mental shortcuts. These shortcuts, known as **cognitive biases**, help us make decisions quickly and efficiently, especially under uncertainty. They evolved to help our ancestors survive — to avoid danger, spot opportunities, and act fast when time mattered most.

In many real-world situations, these instincts are helpful. They allow us to judge quickly, act decisively, and reduce decision fatigue. But in software development — where the problems are abstract, the consequences delayed, and the systems complex — these same instincts can easily mislead us.

![](/images-posts/2025-05-19-complexity-6-human-nature/cognitive-biases.png)

Books like *Thinking, Fast and Slow* by Daniel Kahneman and *Predictably Irrational* by Dan Ariely explore the nature of these biases in depth, revealing just how irrational even our most "logical" thinking can be. Let’s look at a few common cognitive biases and how they influence the decisions we make in our work — often adding accidental complexity to our systems.

### Complexity Bias

We tend to assume that complex problems require complex solutions. Simple answers feel naive, incomplete, or unworthy of real-world use — even when they’re exactly what’s needed.

For example, we might skip using a basic dictionary or an enum to model a small state machine and instead introduce a full-blown state pattern with protocols, coordinators, and factories — not because the problem demands it, but because it *feels* like a better-engineered solution. The result? More indirection, more surface area, and more mental effort to maintain something that could have been straightforward.

### Simplicity Bias

At the other extreme, we sometimes believe that *one* simple idea can solve all our problems. This is the “silver bullet” mindset — believing that a new framework, language, or architecture will fix everything.

We see this when teams latch onto new technologies and try to apply them everywhere. But reality rarely conforms to a single tool. For instance, switching to a reactive programming framework might simplify some flows but introduce unnecessary complexity in others. Overcommitting to a universal solution often leads to misfit abstractions and workarounds that break the simplicity we were after.

### Foreseeing the Future

Our ability to imagine the future helped humans survive. But in software, our predictions are often wrong.

We design for imagined scale, performance, and reusability — and build systems that are far more complex than they need to be. We add abstraction layers, generalize interfaces, or design elaborate plugin architectures for a feature that might never grow. The future-proof solution feels responsible in the moment — but if that future never arrives, we’re left maintaining complexity that no longer serves a purpose.

The same happens with **premature optimization**. We spend days crafting modular architecture, separating every concern, or making our scroll views buttery smooth — in an MVP no one has used yet. We optimize for problems we *might* face, rather than focusing on what matters now.

### Conformism and Cargo Cults

Humans are social creatures. We learn by imitation — especially from those we perceive as successful. But copying without understanding is risky.

In software, this manifests as **cargo culting** — adopting tools or practices simply because others (often big companies) do. One famous example: Airbnb adopted React Native for their mobile app, later found it misaligned with their needs, and moved away from it. Yet many teams continue to use React Native *because* Airbnb once did, not considering whether it’s right for their own scale and context.

### Anchoring Bias

Once we make an initial choice, we tend to stick with it — even when the context changes.

Let’s say a team has used UIKit for a decade. Even as SwiftUI matures and becomes a better fit for new features, they continue to build everything the old way — not because it's better, but because it's familiar. The initial decision becomes an unspoken prerequisite. We stop evaluating it critically and treat it as a fixed point in our reasoning.

### Confirmation Bias

We naturally seek information that supports what we already believe and ignore what contradicts it.

If we think a particular architecture is superior, we’ll highlight examples where it worked and dismiss the ones where it didn’t. This bias slows down learning and locks us into suboptimal patterns. It makes us blind to flaws in our own code and resistant to alternative perspectives — both of which are essential to reducing complexity.

### Sunk Cost Fallacy

This bias leads us to continue investing in something because we’ve already put time, effort, or resources into it — even when it no longer makes sense.

For example, a team might recognize that Core Data is no longer meeting their needs. But because they’ve already built everything around it, switching feels like admitting failure or "wasting" past work. So they keep investing in the old system, layering on patches and workarounds, increasing complexity with every iteration — instead of making a clean break.

### Pattern Recognition (and Misapplication)

Our brains are wired to spot patterns — it's how we learn. But sometimes we see patterns where none exist, or we apply old solutions to new problems that only *look* similar.

For instance, seeing that two features share a similar flow, we abstract their logic into a shared module. But small differences begin to emerge — and soon the abstraction becomes a tangle of conditionals trying to accommodate both. What started as an effort to simplify ends up more complex than the originals.

---

## Developers’ Specifics: When Complexity Becomes a Temptation

Beyond the general cognitive biases we all share, software developers have their own unique tendencies — shaped by the nature of the work, the culture of the industry, and the psychology of technical problem-solving. These tendencies don’t just tolerate complexity — they sometimes *invite* it.

### Simple Solutions Are Boring

Many developers, especially experienced ones, find little excitement in straightforward answers. A problem that can be solved in ten lines of code might not feel “interesting” enough. Instead, we look for elegant patterns, clever abstractions, or novel techniques — not because they’re needed, but because they’re *fun*. This often results in solutions that are harder to read, test, or maintain — even though they were more enjoyable to build.

### The God Syndrome

There’s a subtle, almost mythic appeal to mastering complexity. Building something deeply abstract or highly optimized — something only you or a few others truly understand — can feel like a power move. The more sophisticated our code becomes, the more control and intellectual dominance we might feel. But this illusion of mastery can be dangerous: systems built for intellectual satisfaction often become fragile, opaque, and intimidating for others.

### Social Reinforcement of Complexity

Complexity is frequently rewarded in various professional domains from academia to industry, and software development is also among them. In code reviews, presentations, or open-source projects, intricate solutions are often met with admiration — even when a simpler alternative would be more maintainable. In tech culture, solving something in a clever way is often valued more than solving it in a *clear* way. This dynamic reinforces habits that prioritize impressiveness over simplicity.

### Reinventing the Wheel

For many developers (myself included), solving problems from scratch is one of the most enjoyable parts of the job. Even when well-tested libraries, frameworks, or even components from our own projects already exist to solve the problem, we sometimes choose to build a new solution. We justify it as being more “tailored to our needs,” but it often duplicates effort and introduces new maintenance burdens. Reinvention gives us a sense of ownership and technical fulfillment — but it can also add unnecessary complexity to systems that didn’t require it.

### CV-Driven Development

Sometimes, complexity isn’t introduced for technical reasons at all — but for career visibility. Adopting trendy architectures, experimental tools, or heavyweight frameworks can be justified as “future-proofing” or “scaling,” when in reality they’re resume boosters. We rationalize that these technologies will “look good on GitHub” or during interviews — even if they overcomplicate the current product.

---

## Personal Traits and Complexity

Beyond the biases, there’s another important factor influencing the complexity of our solutions — our individual personalities.

We all bring a unique mix of temperament, preferences, strengths, and fears to our work. These personal traits shape how we interpret problems, what kind of solutions we gravitate toward, and how we approach uncertainty. The result? Even with the same technical skills and constraints, two developers may solve the same problem in entirely different ways — with wildly different complexity profiles.

Here are some of the personality-driven behaviors that subtly (or not-so-subtly) influence the decisions we make.

### Risk Tolerance and Change Aversion

Some developers thrive on experimentation, eager to adopt new tools, rewrite old modules, or rethink established structures. Others prefer stability — valuing predictability, reliability, and minimizing disruption.

- High risk-tolerance can lead to innovation and architectural progress — but also to instability and churn.
- Low risk-tolerance protects against breaking changes, but can entrench legacy systems and block necessary evolution.

> 🧩 *Example: One developer pushes to migrate a legacy feature to Swift Concurrency. Another resists, citing edge cases and team ramp-up time. The resulting compromise is a hybrid system that’s more complex than either approach on its own.*

### Perfectionism vs. Pragmatism

Perfectionists strive for clean abstractions, polished code, and comprehensive solutions — sometimes to the point of over-engineering. Pragmatists, on the other hand, optimize for getting things done with minimal friction.

- Perfectionism can lead to elegant but bloated designs.
- Pragmatism can lead to faster delivery but more technical debt — especially when context is ignored.

> 🧩 *Example: A team creates a generic plugin-based form engine that can handle any field type. The product only needs two. The overhead ends up outweighing the benefit.*

### Detail-Oriented vs. Big Picture Thinkers

Some developers obsess over implementation detail — performance, edge cases, data layout. Others focus on architecture, workflows, and long-term design.

- Detail-oriented devs may introduce low-level complexity in the name of precision.
- Big-picture thinkers may miss technical pitfalls while chasing architectural ideals.

> 🧩 *Example: One dev optimizes data caching with custom flags and serialization. Another finds it too hard to use and starts building a parallel system — doubling the complexity.*

### Control-Oriented vs. Delegators

Control-oriented developers prefer to own the entire stack, building their own solutions for predictability. Delegators trust libraries, services, and abstraction to do the heavy lifting.

- Too much control leads to reinvention and high maintenance.
- Too much delegation can result in black-box dependencies and unexpected edge cases.

> 🧩 *Example: A team builds a custom HTTP client for “full control.” Later, they struggle to add standard features like retries and authentication that a third-party library already offered out of the box.*

### Confidence and Experience Level

Confidence shapes how developers make trade-offs. Less experienced developers may over-rely on frameworks or patterns they've seen in tutorials. More experienced ones may overgeneralize from past projects — even when the context has changed.

> 🧩 *Example: A junior developer uses a `BaseViewModel` pattern because it's common in examples — even though the app doesn't need it. A senior developer insists on custom DI infrastructure because “that's what worked in the past,” ignoring better options.*

### Communication Style and Assertiveness

In team environments, not all decisions are made by the most thoughtful person — sometimes they’re made by the most vocal one. Developers who are confident and articulate may steer teams toward or away from certain solutions. Those who are quieter may struggle to challenge decisions — even when they spot emerging complexity.

> 🧩 *Example: A persuasive developer pushes Clean Architecture for a simple app. The team complies, but ends up with fragmented modules and ceremony-heavy code that no one enjoys working with.*

---

## Navigating Complexity with Self-Awareness

At its core, software complexity isn’t just a technical challenge — it’s a human one. Our cognitive limits, personal traits, biases, and emotional instincts are deeply woven into the systems we build. We like to imagine ourselves as rational, logic-driven engineers, but the truth is more nuanced. Every decision — from architecture to variable naming — is filtered through the lens of how we think, feel, percieve the world and relate to others.

### Acknowledging Human Nature

The first step toward managing this complexity is simple: **awareness**. You can’t remove cognitive biases or overcome your working memory limits — but you can notice them. By naming our instincts, we give ourselves a moment to pause, reflect, and respond with intention instead of habit. Recognizing that our brains are optimized for survival, not systems design, helps us work with our limitations rather than against them.

### Recognize the Invisible Forces

Before reaching for a solution, step back. Ask yourself: *Why am I really making this choice?*  
Is this abstraction for clarity — or for cleverness?  
Am I resisting a change because it’s risky — or because it’s genuinely unnecessary?  
Am I repeating a familiar pattern just because it’s comfortable?

And then look beyond yourself:  
Is the technical direction being shaped by thoughtful discussion — or by the loudest voice in the room?  
Are diverse perspectives being heard, especially from those who may be quieter or newer to the team?

### Build Systems That Guide Behavior

Not all complexity comes from individual behavior. Sometimes it’s baked into the culture, the process, or the tooling. That’s why we need to **intentionally shape our environment** to support good decisions:

- Favor conventions and templates that encourage clarity over cleverness.
- Keep interfaces small, names specific, and responsibilities narrow by default.
- Use boundaries — both architectural and social — to prevent complexity from leaking across the system.

### Create Safe Spaces for Discussion and Learning

Healthy teams surface biases early, not late. They ask hard questions in design reviews, reflect together after launches, and encourage curiosity over defensiveness. Pairing different personality types — perfectionists with pragmatists, cautious thinkers with bold innovators — often reveals blind spots and balances extremes.

And most importantly: foster a culture where challenging an idea doesn’t feel like challenging a person. That’s how we move from defending complexity to dismantling it.

### Use Structured Decision-Making Tools

Cognitive shortcuts often lead us to pick familiar tools or repeat past solutions without critical evaluation. Checklists, decision matrices, or even just a shared “consider before you abstract” checklist can help you avoid these defaults and consider meaningful alternatives.

### Rely on Data, Not Just Intuition

Opinions are loud, but data is quieter and more reliable. When deciding whether to refactor, optimize, or adopt a new tool, ground the discussion in evidence: performance metrics, usage data, real-world constraints. Knowing the difference between a real pattern and a coincidental one is a superpower — and statistics are the map.

### Apply Occam’s Razor

Finally, when two solutions solve the same problem equally well, **choose the simpler one**. The one with fewer assumptions, fewer moving parts, and fewer things that can go wrong. Simplicity isn’t just elegance — it’s strategic clarity.

---

## Final thoughts

Software complexity often looks like a technical problem — but it’s deeply human. It reflects how we think, how we collaborate, and how we cope with uncertainty. Code is written for humans first, machines second — and our brains are the ultimate interpreter. The best software isn’t just technically sound; it’s built by people who understand not only the system, but themselves. With self-awareness, empathy, and clear intent, we can create systems that are not only powerful, but also maintainable, adaptable, and human-friendly.

Cognitive limits, biases, and personal tendencies all influence the way we design and evolve software. They shape how we frame problems, what solutions we reach for, and which trade-offs we accept. Left unchecked, they lead to solutions that don’t quite fit — adding complexity not because the problem demanded it, but because we misunderstood the problem. When we recognize these human influences, we gain the power to catch ourselves — and each other — in the act of unintentionally making things harder. That’s how we move from managing code to managing complexity.


