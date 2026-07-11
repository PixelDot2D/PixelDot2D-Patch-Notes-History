# PixelDot2D Core Framework Patch Notes History
Full Patch Note History for PixelDot2D Core Framework.

**Available on the:** [Unity Asset Store](https://assetstore.unity.com/packages/tools/utilities/pixeldot2d-core-framework-370674)

> [!IMPORTANT]
> **Modular Deployment & Update Lifecycle Rules:**
> 
> - **Asmdef Isolation:** Every sub-library within the PixelDot2D ecosystem is isolated into its own explicit Assembly Definition (`.asmdef`). This means framework updates are never "all-or-nothing." You can selectively merge or update specific modules (Combat, Platformer, Modular Character) at your own leisure, or choose to lock a sub-library to an older version indefinitely if it is already tightly coupled to your active project loop.
> - **The Core Requirement:** While downstream sub-libraries are structurally independent of one another, the **Core module serves as the foundational dependency** for all systems. To ensure framework stability, Core must always be kept updated to match the minimum version requirements of any active sub-library.
> - **A Note on Custom Monolithic Coupling:** If you have manually removed the internal `.asmdef` files to intentionally break down assembly boundaries and tightly couple a sub-library directly into your own project's monolithic assembly namespace, be advised that package updates cannot automate this. A manual code merge will be required to maintain structural compatibility with the updated Core layer.

## Table Of Contents

- [Patch 2.1.0](#patch-210)
  - [Core Updates](#core-updates-patch-2-1-0)
  - [Combat Updates](#combat-updates-patch-2-1-0)
  - [Modular Character Updates](#modular-character-updates-patch-2-1-0)
  - [Critical Refactor Notice](#critical-architecture-update-unified-data-pipeline-refactor-patch-2-1-0)


- [Patch 2.0](#patch-20)
  - [Core Updates](#core-updates-patch-2-0)
  - [Combat Updates](#combat-updates-patch-2-0)
  - [Platformer Updates](#platformer-updates-patch-2-0)
  - [Modular Character Sub-Library](#modular-character-sub-library-patch-2-0)

---
## Patch 2.1.0

### Core Updates <a name="core-updates-patch-2-1-0"></a>

#### Stats Ecosystem:
- **Safe Stat Overwrites:** Upgraded the internal safe modifier method lookup pipeline to elegantly overwrite existing entries if a matching `EntityID` is already registered. This replaces the previous strict rejection pattern and enables seamless, dynamic refreshing of active modifiers originating from the exact same source.
- **Buff & Debuff Neutralization:** Implemented standalone Buff and Debuff immunity systems. When active, these states completely isolate a targeted stat entity from external calculation passes, entirely negating positive or negative modifiers according to their respective structural scopes.
- **Reference-Counted Stat Immunities:** Added full support for stacking additive status immunities across multiple active sources. Modifiers are managed via a non-destructive reference-counting pipeline, ensuring that tracking sources do not introduce destructive side effects to one another (e.g., stripping a temporary potion immunity will safely preserve a permanent equipment immunity).

#### Save and Load Serialization Architecture:
- **Automated Low-Friction Serialization:** Overhauled the `SaveAndLoadManager` to maximize developer ease-of-use without sacrificing raw disk I/O performance, defensive error handling, or stream stability. Subsystems are now registered using a simple Inspector drag-and-drop workflow. At runtime, the manager automatically validates, captures, and sequences the data stream internally via `Enum_ISaveableAndLoadableKey` sorting rules, completely eliminating manual structural maintenance.
- **Assembly Isolation:** While the `SaveAndLoadManager` has always resided inside the core framework Assembly Definition (`.asmdef`), it is now fully decoupled via loose enum mappings and interface hooks. This guarantees that Core remains strictly isolated, comfortably saving and loading data across external sub-libraries and custom user-space systems without requiring upstream assembly references.

#### RB2DMovementManager Optimization:
- **Hybrid Velocity Pipeline:** Upgraded the velocity calculation pipeline from strictly relying on the `ScriptableObject` value to dynamically picking between the configuration asset or the caller’s runtime value.
- **Absolute Control:** Passing a multiplier of `Vector2.one` allows the `ScriptableObject`'s raw configuration values to drive the final velocity calculations entirely.
- **Caller Control:** Passing a dynamic `Vector2` runtime value (such as live character stats or randomized multi-axis projectile variance) drives the final output, while setting the corresponding `ScriptableObject`'s base axis speeds to `1.0f` acts as a clean, unscaled multiplier baseline.
- **Axis Locking:** Setting any specific axis speed to `0.0f` within either the configuration asset or the incoming multiplier vector completely isolates, locks, or ignores movement on that plane via direct zero-multiplication.

#### Low-Overhead Native Collision Matrix:
- **Custom Collision System:** Added a high-performance collision matrix to completely bypass Unity's native Box2D performance cost and lifecycle inconsistencies.
- **Traditional Trigger Lifecycles:** Maintains full `OnTriggerEnter`, `OnTriggerStay`, and `OnTriggerExit` lifecycle simulations, driven entirely by manual, predictable update pumps inside your fixed layout loop.
- **Batched Data Payloads:** Combines all frame intersections into a single event call, separating results into dedicated collections for valid targets and obstacles simultaneously.
- **Total Execution Control:** Grants developers absolute control over sorting priority, allowing you to check obstacle lines first for early deactivation or focus on valid target logic instantly.
- **Dual Evaluation Modes:** Added support for two distinct collision check resolutions selectable directly via the Inspector:
  - `PerGameObject`: Automatically filters out duplicate hits originating from redundant sub-colliders.
  - `PerCollider`: Tracks individual sub-collider components independently, treating each instance as a unique spatial interaction for precise, locational hitbox mapping.
- **Dynamic Physics Casting:** Updated Box, Capsule, and Line shapes to optionally factor in the origin's direct rotation or remain globally isolated from local orientation changes.
- **Custom Pivot Offsets:** Added configuration options to allow casting shapes to explicitly use the origin transform as a custom rotational pivot point.

#### Sub-Library Alignment:
- **Signature Standardization:** Standardized the `Init` method signatures for `RB2DMovement_ModularCharacter` and `RB2DMovement_CombatManager` to require identical core arguments. This strict architectural alignment ensures that upgrading a standard character to a combat-ready state is as simple as swapping a variable, providing a clean path for a library merge if desired.

### Combat Updates <a name="combat-updates-patch-2-1-0"></a>

- **Advanced Projectile Upgrades:** Integrated the `MultiWeaponBlueprint` architecture directly into individual projectile entities to drastically expand their utility.
- **Multi-Weapon Sequencing:** Any complex weapon behavior capable of being structured inside a `MultiWeaponBlueprint` configuration can now be natively deployed by a projectile directly from the Inspector.
- **Chain of Command Execution:** Implemented a unified forwarding pipeline that securely passes the root entity source across deeply nested projectile layers. This ensures downstream targeting, tracking filters, and damage calculations always route back cleanly to the original instigator.
- **Dynamic Layer Relocation:** Shifted valid target and obstacle layer tracking out of static configuration `ScriptableObjects` and directly onto the `IWeaponizable` interface. This fully uncouples targeting constraints from fixed asset data and grants entities absolute authority over their own spatial detection parameters at runtime. Developers can now easily implement dynamic gameplay mechanics such as charm effects, temporary faction swaps, or status-driven accuracy modifiers—such as completely zeroing out obstacle layers (e.g., dropping a Wall layer bitmask to `0`) to seamlessly execute piercing projectile upgrades.

### Modular Character Updates <a name="modular-character-updates-patch-2-1-0"></a>

- **Reference-Counted Status Immunities:** Upgraded the passive status execution layer to natively support non-destructive immunity tracking via unique `EntityId` mapping. Active status protections originating from overlapping, multi-layered sources (such as an item, an active passive, and a temporary consumable potion simultaneously) now register independently within a pre-warmed tracking dictionary. When a temporary effect expires or is cleanly stripped, the system triggers an optimized baseline cache sweep, safely preserving remaining active gameplay immunities without allowing temporary duration ends to leave behind data leaks or accidentally clear permanent equipment or baseline protection.
- **State Factory Validation Lifecycle:** Updated the `ModularCharacterController` pipeline to allow state instances to register and preserve their originating `ScriptableObject` factory references. This introduces a secure, deterministic verification mechanism that allows external systems to safely cache active slots, hot-swap behaviors, and accurately validate ownership bounds before reverting modifications. This structural anchor provides immediate native support for complex runtime events like temporary passives and equipment configuration swaps.
- **Dynamic World-Space Inversion:** Implemented an array of relative orientation methods to completely uncouple entity translational physics from Unity's global static coordinates. By tracking and scaling localized axis signs through basis vectors, developers can cleanly trigger advanced spatial modifications—such as completely reversing a player's directional control layout or flipping relative environmental gravity parameters—via simple runtime vector adjustments.
- **New Passive Execution – Stat Immunity:** Introduced a versatile execution type fully integrated with all structural passive Gates. This new module grants passive Cog sequences direct authority over an entity's Stat immunity layer. Developers can now orchestrate runtime execution passes that dynamically trigger absolute isolation from buffs, debuffs, or both simultaneously, driven entirely by `ScriptableObject` data configurations.
- **New Passive Execution – Debuff Immunities:** Introduced a specialized execution type integrated with all structural passive Gates. This Cog enables passive sequences to instantly grant comprehensive debuff immunities based on specific mechanical or damage-school flags mapped in the asset data.
- **New Passive Execution – Change State:** Added a state-transition Cog that triggers a runtime state change upon satisfying a defined Gate Cog while securely caching the pre-existing state context. Upon reaching the passive’s designated Exit Cog, the execution layer evaluates the current state; if an external source has since overridden the state category or assumed state ownership, the reversion gracefully aborts. This allows temporary state-altering passives (such as gliding, hovering, or dashing) to safely self-clean without disrupting newer, high-priority state overrides.


## Critical Architecture Update: Unified Data Pipeline Refactor <a name="critical-architecture-update-unified-data-pipeline-refactor-patch-2-1-0"></a>

### What Changed?
We have completely overhauled how telemetry and calculation data flow through our modification hooks, states, and passive systems. Instead of passing loose, individual variables (such as raw floats, separate game references, and isolated bitmasks) across verbose method signatures, all core data is now packaged into a single, high-performance struct parameter block.

### Why Was This Done?
- **Infinite Extensibility without Breaking Changes:** Transitioning to a parameter block struct completely future-proofes your custom APIs. If you or PixelDot2D need to introduce new combat telemetry fields down the line (such as critical hit multipliers, status effects, or elemental school flags), you can simply expand the struct definition. All existing method signatures remain fully intact, completely eliminating the need for cascading code rewrites across your project.
- **The Unified Cross-Library Bridge:** This structural shift aligns perfectly with the underlying design of our Combat Sub-Library. If you choose to merge the Modular Character and Combat sub-libraries together, they now natively speak the same language and share the exact same structural data blueprint. This turns a historically complex multi-system integration into a straightforward variable swap.

### Crucial Upgrade Instructions
> [!WARNING]
> Because this refactor modifies the core signatures across our passives, gates, and states, updating your project package will cause localized compilation errors if you have heavily overridden or customized these files. 

- **If you have NOT added custom code:** Simply allow the package update to overwrite your local files completely. The project will compile cleanly out of the box with the new struct implementations.
- **If you HAVE written custom logic:** A manual script merge will be required. You will simply need to update your custom hook method overrides to accept the new struct parameter and read your logic values directly from its fields.
- **Ready to Merge Libraries:** If you decide to merge the two libraries, we have included a meticulously detailed, heavily commented integration blueprint directly within the codebase. To view the step-by-step instructions on how to seamlessly bridge these two packages together, inspect `DamageData.cs` or `DamageReport.cs` located within your Combat structs directory.


---

## Patch 2.0 

### Core Updates <a name="core-updates-patch-2-0"></a>

- **Thread Safety:** Added thread-safe handling for Unity Objects during Preload, Pre-save, Save Complete, and Load Complete cycles.
- **Zero-GC Spatial Sensor Module:** Integrated a centralized suite of high-performance spatial sensor utilities supporting Box, Circle, Capsule, and Line casts. These standardized sensors drive environmental awareness across all sub-libraries with zero runtime allocation overhead and integrated Editor Debug Visuals. Following the framework's strict rule of open extensibility, the sensor module is fully architected to support expansion into custom shapes.
- **Coordinate Virtualization (RB2D Mover):** Refactored the core movement solver to utilize an internal Basis Mapping system. Objects can now define their own local coordinate space (Right/Up vectors) independently of Unity’s Transform component. This allows entities to handle complex, genre-specific movement behaviors, such as 2D side-scrolling sprite flips, via external vector injection without altering physical GameObject orientation or breaking mathematical calculations.

### Combat Updates <a name="combat-updates-patch-2-0"></a>

- **New Execution Types:** Added Box and Capsule casting for virtual weapon execution.
- **Optimized Native Physics Execution:** Projectiles have been completely decoupled from traditional Collider2D components. All spatial detection now utilizes direct native C++ calls via `Physics2D.Cast` (supporting Box, Capsule, and Circle shapes). This bypasses the overhead of Unity’s internal physics solver for maximum performance.
- **Virtual Collision Lifecycle & Overlap Tracking:** Implemented a zero-allocation, math-driven physics solver that perfectly emulates Unity’s `OnEnter`, `OnStay`, and `OnExit` hooks without physical colliders. By utilizing an internal persistent buffer, the system enforces Overlap Memoization to prevent multi-hit frame spikes while natively supporting Dynamic Re-entry Detection if a target leaves and re-enters the virtual volume.
- **Deterministic Feedback Filtering:** Implemented a specialized `OnObstacleHitOnly` event hook, allowing developers to cleanly isolate environmental particle feedback (such as wall sparks) from high-priority combat visual effects (such as blood splatters).
- **Stripped Editor Debugging:** All custom virtual collision profiles include live visual debugging. These utilities are strictly wrapped inside `#if UNITY_EDITOR` preprocessor directives, guaranteeing absolute zero CPU or memory overhead in production builds.
- **Animation Frame-Driven Combat Synchronization:** Integrated the core animation pipeline directly into `RB2DMovement_CombatManager`. The system filters combat execution boundaries based on the active animation frame using an O(1) constant-time lookup structure. Leaving constraint frames empty enables relentless, multi-frame offensive onslaughts, while specifying explicit frame indices grants total authority over frame-perfect combat execution timings.

### Platformer updates <a name="platformer-updates-patch-2-0"></a>

- **Centralized Collision Migration:** Fully migrated actor collision detection to the new Core Physics Module for optimized execution footprints and unified visual debugging.
- **Global Standardization:** Moved `Enum_PlatformerFacingDirection` to Core and renamed it to `Enum_SideScrollerFacingDirection` for universal use.
- **Updated Layer Naming Convention:** `Pushable` is now `Interactable`.

### NEW Modular Character Sub-Library <a name="modular-character-sub-library-patch-2-0"></a>

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
