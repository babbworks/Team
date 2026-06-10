# Babb Works

From Rust Belt, to Kalahari, to the Cosmos, Babb builds tools for work.

Babb is an industrial software company and an open-source foundry. We build practical software, standards, and hardware-oriented systems that help people start, run, and grow real operations under real constraints.

We build for the working world: individuals, small teams, shop floors, farms, field crews, and industrial operators whose work cannot pause for fragile software.

## Why Babb Exists

Most business and productivity tools are designed for stable desks, stable teams, and stable conditions. Much of the world does not work that way.

People who operate businesses in dynamic environments still need:

- Fast and legible tools
- Reliable records and communication
- Systems that work on low-cost hardware
- Software that functions in low-connectivity settings
- Open standards they can trust and build on

Babb exists to help businesses start, operate, and endure, while keeping the core systems of work open and interoperable.

## What We Build

We build across a scope that starts with command-line and personal workflows, and scales up to industrial operations and standards.

### Personal and Team Command-Line Operations

- [Workwarrior](https://github.com/babbworks/workwarrior): profile-based productivity system that unifies tasks, time, journals, and ledgers in the terminal.
- [Warrior](https://github.com/babbworks/warrior): mobile and desktop work management tool upstream from Workwarrior.
- [workpads-cli](https://github.com/babbworks/workpads-cli): command-line tools around Workpads workflows.

### Protocols and Standards

- [BitLedger](https://github.com/babbworks/bitledger): compact double-entry transmission protocol engineered for constrained environments.
- [BitPads](https://github.com/babbworks/bitpads): universal binary protocol family that extends accounting-grade transmission to broader operational contexts.
- [standards](https://github.com/babbworks/standards): central hub for maintained public standards and protocols.
- Standards repositories currently include:
  - [workpads-standard](https://github.com/babbworks/workpads-standard)
  - [bitpads-standard](https://github.com/babbworks/bitpads-standard)
  - [babbs-standard](https://github.com/babbworks/babbs-standard)
  - [BASICS-standard](https://github.com/babbworks/BASICS-standard)
  - [warrior-standard](https://github.com/babbworks/warrior-standard)

### Industrial and Enterprise Operations

- [Clarkware](https://github.com/babbworks/clarkware): on-the-floor manufacturing operations environment for technicians, supervisors, and integrated services.
- [ClarkChat](https://github.com/babbworks/clark-chat): persistent, type-aware conversation layer for operations, learning, and automation.
- [Workpads](https://github.com/babbworks/Workpads): business records protocol and applications, including mobile-first deployment paths.
- [SIMBA](https://github.com/babbworks/SIMBA): Service Integration for Mobile Business Applications.

### Small Utility and Organizational Tools

- [heybabb](https://github.com/babbworks/heybabb): terminal companion for navigating Babb tools and direction.
- [company-character](https://github.com/babbworks/company-character): framework for building organization-specific CLI assistants.
- [outstack](https://github.com/babbworks/outstack), [picowarrior](https://github.com/babbworks/picowarrior), [telux](https://github.com/babbworks/telux), and [Babb](https://github.com/babbworks/Babb): supporting products and experiments in the Babb ecosystem.

## Flagship Projects

The current flagship projects are:

- [Workwarrior](https://github.com/babbworks/workwarrior)
- [BitLedger](https://github.com/babbworks/bitledger)
- [BitPads](https://github.com/babbworks/bitpads)
- [Clarkware](https://github.com/babbworks/clarkware)

`heybabb` and `company-character` represent the class of compact, high-utility tools we build for enterprise operations and contributor onboarding.

## Technical Principles

Babb follows a technical philosophy shaped by constraints, clarity, and long-term interoperability.

1. **Terminal-first foundations**
   - We often build first for terminals and command lines, then expand upward to browser, mobile, and service surfaces.
   - Language, symbols, and explicit commands are treated as durable interfaces between humans and machines.

2. **Simple, efficient communication**
   - We prefer explicit protocols, lean formats, and observable behavior over opaque abstractions.
   - We optimize for legibility, reliability, and low overhead.

3. **Design for constraints**
   - Software should remain useful across unstable connectivity, older hardware, low-power environments, and varied operator skill levels.
   - We value offline-first and local-first patterns where appropriate.

4. **Hardware and firmware awareness**
   - We design with deployment realities in mind, including firmware constraints, low-level optimization paths, and embedded/edge execution contexts.
   - Assembly and Python are both first-class in our stack, alongside simple shell operations. Always testing languages for suitability.

5. **Open standards, maintained in public**
   - Standards and protocols are developed and maintained openly.
   - We support standards-track evolution so others can implement compatible systems without gatekeeping.

## Open Source Commitment

Babb is committed to keeping a permanent open-source version of every tool.

Commercial products and services may be developed around these systems, including cloud platforms, enterprise support, and hardware offerings, but core capabilities are developed upstream in open repositories whenever possible.

We treat open participation as infrastructure, not marketing.

## Contribution Model

Babb's primary audience is contributors.

We welcome participation across:

- Source code contributions
- Extensions and plugins
- Protocol and standards reviews
- Documentation and translation
- Testing in real operating environments

Babb takes responsibility for maintaining core integrity across tools, while providing structure, governance, and contribution rules so community work can compound safely.

## Mobile and Device Access

We are committed to affordable, widely available devices as first-class targets for work software.

This includes KaiOS-based mobile phones and similar low-cost devices in emerging markets. Babb will continue developing and shipping work applications that operate effectively on constrained mobile hardware.

## Position in the Moment

We are entering a period of reindustrialization and renewed space-era systems building while traditional forms of hard work remain foundational: farming, fabrication, transport, repair, field service, and local trade.

Babb is building for continuity across these realities: old and new infrastructure, local and global markets, terrestrial and off-world operations.

## Proposed Organization Roadmap (v0.1)

### Phase 1: Foundation and Cohesion

- Stabilize flagship tool experience and docs (`Workwarrior`, `BitLedger`, `BitPads`, `Clarkware`)
- Consolidate standards in [standards](https://github.com/babbworks/standards)
- Publish contribution rules and extension interfaces across core projects
- Improve cross-repo discoverability through `heybabb` and company docs

### Phase 2: Standards and Interoperability

- Advance `*-standard` repositories with versioning discipline and conformance guidance
- Publish reference implementations and interop fixtures
- Define compatibility profiles for constrained devices and low-bandwidth links

### Phase 3: Device and Field Deployment

- Expand mobile-first deployments, including KaiOS pathways
- Harden edge-ready and hardware-adjacent implementations
- Grow practical integrations between CLI tools, protocol layers, and operational systems
- Pioneer research into novel and long-standing mobile device devices with emphasis on operator autonomy.

### Phase 4: Cloud and Industrial Scale

- Deliver managed cloud services around open cores
- Extend enterprise deployment support for manufacturing and operations environments
- Maintain upstream-open development while introducing specialized products

### Phase 5: Long Arc

- Continue building systems that remain durable from local shops to remote regions to space-connected operations
- Keep standards free to use and practical to implement
- Protect interoperability as a public good

## Repositories

Primary repositories and standards currently include:

- [company-character](https://github.com/babbworks/company-character)
- [heybabb](https://github.com/babbworks/heybabb)
- [bitpads](https://github.com/babbworks/bitpads)
- [bitledger](https://github.com/babbworks/bitledger)
- [workwarrior](https://github.com/babbworks/workwarrior)
- [picowarrior](https://github.com/babbworks/picowarrior)
- [outstack](https://github.com/babbworks/outstack)
- [telux](https://github.com/babbworks/telux)
- [workpads-standard](https://github.com/babbworks/workpads-standard)
- [bitpads-standard](https://github.com/babbworks/bitpads-standard)
- [babbs-standard](https://github.com/babbworks/babbs-standard)
- [BASICS-standard](https://github.com/babbworks/BASICS-standard)
- [warrior-standard](https://github.com/babbworks/warrior-standard)
- [warrior](https://github.com/babbworks/warrior)
- [clark-chat](https://github.com/babbworks/clark-chat)
- [Workpads](https://github.com/babbworks/Workpads)
- [workpads-cli](https://github.com/babbworks/workpads-cli)
- [babbs](https://github.com/babbworks/babbs)
- [SIMBA](https://github.com/babbworks/SIMBA)
- [Babb](https://github.com/babbworks/Babb)
- [standards](https://github.com/babbworks/standards)

Full org repositories: [babbworks repositories](https://github.com/orgs/babbworks/repositories).

## Oath

Babb builds durable tools that help people and businesses lead, endure, and grow.

We serve the workers and economies of today and tomorrow with a timeless touch.

### Oath for Orbit

Getting a business launched requires tools, resources, and supports. Business owners are the ones who determine the markers of success, but the growth of sales is always required for survival. A business can be initiated and operated by people of all ages given timely access to personal assistance and education. Our electronic tools have become essential for day-to-day business and for extending access to commercial activity. However much we accelerate, though, our tools must harmonize with paper-based processes for communities that prefer such approaches. Looking out to the stars, the durable tools and technologies that can reliably operate far away are the same ones that can faithfully serve Earth's remotest regions. From the Midwest to the Kalahari to the Cosmos, Babb exists to help people and businesses to lead and endure. We are devoted to the development of Babb and its planetary and galactic growth. This is our shared oath to take the common spirit of business up into orbit and beyond.
