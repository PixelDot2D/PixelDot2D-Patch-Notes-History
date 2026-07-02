# PixelDot2D-Patch-Notes-History
Full Patch Note Histroy for the Framework.



## Table Of Contents
*Press Ctrl + F for ease of finding specific notes.*

- [Patch 2.0](#patch-20)
  - [Core Updates](#core-updates)
  - [Combat Sub-Library](#combat-sub-library)
  - [Platformer Sub-Library](#platformer-sub-library)
  - [Modular Character Sub-Library](#modular-character-sub-library)

---

## Patch 2.0

### ⚙️ Core Updates

- **Thread Safety:** Added thread-safe handling for Unity Objects during Preload, Pre-save, Save Complete, and Load Complete cycles.
- **Zero-GC Spatial Sensor Module:** Integrated a centralized suite of high-performance spatial sensor utilities supporting Box, Circle, Capsule, and Line casts. These standardized sensors drive environmental awareness across all sub-libraries with zero runtime allocation overhead and integrated Editor Debug Visuals. Following the framework's strict rule of open extensibility, the sensor module is fully architected to support expansion into custom shapes.
- **Coordinate Virtualization (RB2D Mover):** Refactored the core movement solver to utilize an internal Basis Mapping system. Objects can now define their own local coordinate space (Right/Up vectors) independently of Unity’s Transform component. This allows entities to handle complex, genre-specific movement behaviors, such as 2D side-scrolling sprite flips, via external vector injection without altering physical GameObject orientation or breaking mathematical calculations.

### ⚔️ Combat Sub-Library

- **New Execution Types:** Added Box and Capsule casting for virtual weapon execution.
- **Optimized Native Physics Execution:** Projectiles have been completely decoupled from traditional Collider2D components. All spatial detection now utilizes direct native C++ calls via `Physics2D.Cast` (supporting Box, Capsule, and Circle shapes). This bypasses the overhead of Unity’s internal physics solver for maximum performance.
- **Virtual Collision Lifecycle & Overlap Tracking:** Implemented a zero-allocation, math-driven physics solver that perfectly emulates Unity’s `OnEnter`, `OnStay`, and `OnExit` hooks without physical colliders. By utilizing an internal persistent buffer, the system enforces Overlap Memoization to prevent multi-hit frame spikes while natively supporting Dynamic Re-entry Detection if a target leaves and re-enters the virtual volume.
- **Deterministic Feedback Filtering:** Implemented a specialized `OnObstacleHitOnly` event hook, allowing developers to cleanly isolate environmental particle feedback (such as wall sparks) from high-priority combat visual effects (such as blood splatters).
- **Stripped Editor Debugging:** All custom virtual collision profiles include live visual debugging. These utilities are strictly wrapped inside `#if UNITY_EDITOR` preprocessor directives, guaranteeing absolute zero CPU or memory overhead in production builds.
- **Animation Frame-Driven Combat Synchronization:** Integrated the core animation pipeline directly into `RB2DMovement_CombatManager`. The system filters combat execution boundaries based on the active animation frame using an O(1) constant-time lookup structure. Leaving constraint frames empty enables relentless, multi-frame offensive onslaughts, while specifying explicit frame indices grants total authority over frame-perfect combat execution timings.

### 🏃 Platformer Sub-Library

- **Centralized Collision Migration:** Fully migrated actor collision detection to the new Core Physics Module for optimized execution footprints and unified visual debugging.
- **Global Standardization:** Moved `Enum_PlatformerFacingDirection` to Core and renamed it to `Enum_SideScrollerFacingDirection` for universal use.
- **Updated Layer Naming Convention:** `Pushable` is now `Interactable`.

### 📦 NEW: Modular Character Sub-Library
*(Details coming soon)*

---
*Copyright 2026 © PixelDot2D - All Rights Reserved | Contact: PixelDot2D@gmail.com*
