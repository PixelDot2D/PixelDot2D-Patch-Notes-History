# PixelDot2D Core Framework Patch Notes History
Full Patch Note History for PixelDot2D Core Framework.

**Available on the:** [Unity Asset Store](https://assetstore.unity.com/packages/tools/utilities/pixeldot2d-core-framework-370674)

## Table Of Contents

- [Patch 2.0](#patch-20)
  - [Core Updates](#core-updates)
  - [Combat Sub-Library](#combat-sub-library)
  - [Platformer Sub-Library](#platformer-sub-library)
  - [Modular Character Sub-Library](#new-modular-character-sub-library)

---

## Patch 2.0

### Core Updates

- **Thread Safety:** Added thread-safe handling for Unity Objects during Preload, Pre-save, Save Complete, and Load Complete cycles.
- **Zero-GC Spatial Sensor Module:** Integrated a centralized suite of high-performance spatial sensor utilities supporting Box, Circle, Capsule, and Line casts. These standardized sensors drive environmental awareness across all sub-libraries with zero runtime allocation overhead and integrated Editor Debug Visuals. Following the framework's strict rule of open extensibility, the sensor module is fully architected to support expansion into custom shapes.
- **Coordinate Virtualization (RB2D Mover):** Refactored the core movement solver to utilize an internal Basis Mapping system. Objects can now define their own local coordinate space (Right/Up vectors) independently of Unity’s Transform component. This allows entities to handle complex, genre-specific movement behaviors, such as 2D side-scrolling sprite flips, via external vector injection without altering physical GameObject orientation or breaking mathematical calculations.

### Combat Sub-Library

- **New Execution Types:** Added Box and Capsule casting for virtual weapon execution.
- **Optimized Native Physics Execution:** Projectiles have been completely decoupled from traditional Collider2D components. All spatial detection now utilizes direct native C++ calls via `Physics2D.Cast` (supporting Box, Capsule, and Circle shapes). This bypasses the overhead of Unity’s internal physics solver for maximum performance.
- **Virtual Collision Lifecycle & Overlap Tracking:** Implemented a zero-allocation, math-driven physics solver that perfectly emulates Unity’s `OnEnter`, `OnStay`, and `OnExit` hooks without physical colliders. By utilizing an internal persistent buffer, the system enforces Overlap Memoization to prevent multi-hit frame spikes while natively supporting Dynamic Re-entry Detection if a target leaves and re-enters the virtual volume.
- **Deterministic Feedback Filtering:** Implemented a specialized `OnObstacleHitOnly` event hook, allowing developers to cleanly isolate environmental particle feedback (such as wall sparks) from high-priority combat visual effects (such as blood splatters).
- **Stripped Editor Debugging:** All custom virtual collision profiles include live visual debugging. These utilities are strictly wrapped inside `#if UNITY_EDITOR` preprocessor directives, guaranteeing absolute zero CPU or memory overhead in production builds.
- **Animation Frame-Driven Combat Synchronization:** Integrated the core animation pipeline directly into `RB2DMovement_CombatManager`. The system filters combat execution boundaries based on the active animation frame using an O(1) constant-time lookup structure. Leaving constraint frames empty enables relentless, multi-frame offensive onslaughts, while specifying explicit frame indices grants total authority over frame-perfect combat execution timings.

### Platformer Sub-Library

- **Centralized Collision Migration:** Fully migrated actor collision detection to the new Core Physics Module for optimized execution footprints and unified visual debugging.
- **Global Standardization:** Moved `Enum_PlatformerFacingDirection` to Core and renamed it to `Enum_SideScrollerFacingDirection` for universal use.
- **Updated Layer Naming Convention:** `Pushable` is now `Interactable`.

### NEW Modular Character Sub-Library

Built for entities requiring real-time evolution, mutation, and complete runtime restructuring. The framework enforces atomic control over individual behaviors—enabling developers to seamlessly inject, hot-swap, or strip character logic on the fly across any genre utilizing a standard 2D physics plane, including side-scrollers, 2.5D hybrids, top-down shooters, space simulators, and more.

To ensure absolute stability during complex, multi-state reconfigurations, all transition mutations are deferred through an isolated Queue System that completely neutralizes race conditions and null-reference exceptions. This architecture enables absolute, zero-code change extensibility: entirely new states can be introduced and integrated into existing character behaviors without altering a single line of pre-existing code inside the orchestrator. By leveraging the data-driven input pipeline and automated flyweight factory, developers can map complex transitions into newly authored states directly from the Unity Inspector using pre-built blueprint templates engineered for any movement profile a native `Rigidbody2D` can execute.

#### Key Technical Features:

- **Plain C# Architecture:** Runs via manual updates completely outside `MonoBehaviour` hierarchy overhead to ensure zero runtime GC spikes.
- **Frictionless Extensibility:** Add new logic seamlessly using pre-built flyweight and factory Patterns without touching the core controller.
- **Scripted Composition States:** Mix, match, and orchestrate complex entity state lifecycles directly inside the editor using interchangeable `ScriptableObjects`.
- **Intelligent Removal & Fallback:** Safely strip states at runtime with automatic fallback safeguards to a default Master Blueprint.
- **Interface-Driven Pipeline:** Requires only a single, thin interface (`IModularCharacter`), completely eliminating forced inheritance.
- **Virtual Lazy State Pooling:** Internal memory management that only keeps required states in memory, recycling instances to preserve cache locality.
- **Modular "Lego-Style" Passives:** Deconstruct passives into interchangeable `ScriptableObject` cogs to build complex gameplay synergies with zero code changes.
- **Infinite Reusability:** One universal controller to drive players, enemies, AI companions, or any 2D entity.

---
*Copyright 2026 © PixelDot2D - All Rights Reserved | Contact: PixelDot2D@gmail.com*
