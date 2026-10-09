# CLAUDE.md

## Project Overview

This project is a Java addon/plugin for **Meteor Client on Minecraft 26.2**.

The primary goal is to implement a finished, reliable automated mining module that reproduces the **observable behavior demonstrated in the reference YouTube video**, without copying or reconstructing proprietary source code.

The main module should provide automated mining with:

- User-selected mining targets.
- Baritone-based navigation/mining.
- Automatic inventory management using Shulker Boxes.
- Automatic Ender Chest interaction.
- Automatic tool repair through Mending/XP.
- Automatic switching between normal mining and repair mining.
- Automatic recovery and resumption after interruptions.
- Long-running unattended operation.
- Death/respawn/reconnect recovery where technically possible.

The implementation must be independently written from observed behavior and available public/project source code.

---

# Target Version

The current target is:

**Minecraft 26.2**

Do not assume APIs from older Minecraft, Fabric, Meteor Client, or Baritone versions.

The actual project template, Gradle configuration, dependency versions, mappings, and supplied Baritone source are the authoritative sources for APIs.

If the project configuration differs from assumptions in this document, follow the actual project configuration.

---

# Clean-Room / Behavioral Reimplementation

This is a behavior-driven reimplementation.

The reference implementation is **not available as source code**. Behavioral information comes from the observed YouTube video.

Do not attempt to recreate, decompile, extract, or reproduce proprietary implementation details that are not available to the project.

Instead:

1. Observe the externally visible behavior.
2. Describe the required behavior.
3. Design an independent implementation.
4. Use the project's APIs and supplied source code to determine how to implement it.
5. Test the resulting behavior.
6. Refine the implementation when new observations become available.

Do not claim that an implementation detail is part of the original addon unless it was actually observed or otherwise established.

---

# Source of Truth

When determining how something should be implemented, use this priority order:

1. Existing project source code.
2. The project's Gradle configuration and declared dependencies.
3. The supplied Baritone 26.2 source code.
4. Other exact-version project/reference material.
5. Public documentation for the exact versions in use.
6. General Java/Minecraft knowledge.

Never choose an API simply because it existed in another Minecraft or Baritone version.

When uncertain:

- Inspect the dependency.
- Search the project for existing usages.
- Inspect the supplied Baritone source.
- Confirm the exact method/class/signature.
- Then implement.

Do not guess.

---

# Baritone Source

A copy of the **Baritone source for Minecraft 26.2** may be supplied with the project.

It exists primarily as reference material for understanding the exact API and behavior available to this project.

Do not modify the vendored Baritone source unless explicitly instructed.

Prefer using the project's declared Baritone dependency/API rather than creating a modified Baritone fork.

When implementing Baritone integration:

1. Determine the exact Baritone version used by the project.
2. Inspect the supplied Baritone source.
3. Search for existing project usages.
4. Implement through a small abstraction layer.
5. Keep Baritone-specific code isolated from the main automation state machine.

---

# Project Organization

Prefer a structure conceptually similar to:

```text
project/
├── CLAUDE.md
├── build.gradle
├── gradle.properties
├── settings.gradle
├── src/
│   └── ...
├── reference/
│   └── REFERENCE_BEHAVIOR.md
└── vendor/
    └── baritone/
        └── ...
```

Do not force this exact structure if the supplied Meteor template uses another appropriate structure.

Follow the existing project's conventions.

---

# Reference Behavior

Maintain a separate behavioral reference document when useful:

```text
reference/REFERENCE_BEHAVIOR.md
```

This document should contain observations from the video and later testing.

Separate information into:

- **Observed** — directly visible or explicitly stated.
- **Inferred** — a reasonable conclusion required to implement observed behavior.
- **Unknown** — behavior that has not been established.
- **Implemented** — behavior implemented in this project.
- **Tested** — behavior verified in Minecraft.
- **Failed/Incomplete** — known limitations.

Do not turn guesses into requirements.

---

# Primary Module

The main automation module should be called `Miner` unless the existing project conventions require another name.

Conceptually it should operate as a state machine.

Suggested states:

```text
IDLE
NORMAL_MINING
INVENTORY_STORAGE
REPAIR_MINING
RECOVERY
PAUSED
ERROR
```

The exact implementation can differ if a cleaner architecture is appropriate.

The important requirement is that the module always has one clearly defined high-level objective.

---

# High-Level Priority

The automation should generally prioritize:

1. Critical recovery/error handling.
2. Death/respawn/reconnection recovery.
3. Inventory storage.
4. Tool repair.
5. Normal mining.
6. Idle/paused state.

Review this ordering during implementation if real-world behavior shows that another ordering is necessary.

Do not allow competing systems to independently control Baritone at the same time.

---

# Normal Mining

The user must be able to select **any appropriate Minecraft block(s)** as the mining target.

Do not hard-code Netherite, Diamonds, Tuff, or another resource as the only supported target.

The normal mining system should:

1. Read the user's selected target.
2. Determine appropriate blocks to mine.
3. Give the mining/navigation objective to Baritone.
4. Monitor progress.
5. Detect interruptions.
6. Resume the same objective after temporary interruptions.

Normal mining may be interrupted by:

- Inventory becoming full.
- Tool durability reaching the repair threshold.
- Death.
- Disconnect.
- Missing/unavailable target.
- Baritone failure.
- Missing required storage resources.
- User disabling the module.
- An unrecoverable error.

The original mining objective should be preserved so it can be resumed after temporary tasks finish.

---

# Baritone Controller

Do not spread direct Baritone API calls throughout the module.

Create an abstraction such as:

```text
BaritoneController
```

or an equivalent project-specific design.

Its responsibilities may include:

- Starting a mining objective.
- Stopping the current objective.
- Checking whether Baritone is active.
- Determining whether the current task completed/failed.
- Navigation to a location.
- Returning control to the Miner state machine.

The exact API must be determined from the supplied Baritone 26.2 source and project dependencies.

Only one high-level Baritone task should control the player at a time.

---

# Inventory Automation

The reference behavior shows automatic inventory management using Shulker Boxes.

When inventory becomes sufficiently full:

1. Pause normal mining.
2. Obtain an empty Shulker Box from an Ender Chest.
3. Place/use the Shulker Box.
4. Transfer appropriate inventory items into it.
5. Recover the Shulker Box as appropriate.
6. Continue normal mining.

The user specifically wants the implementation to **locate/place the storage setup automatically**, rather than requiring a permanently configured fixed position.

Do not assume a fixed Ender Chest coordinate.

Do not invent a destructive chest-breaking strategy based solely on ambiguous video narration.

Prefer safe, observable inventory/container interactions.

---

# Shulker Handling

The system should reproduce the behavior shown in the video rather than impose an arbitrary one-Shulker limitation.

Design the storage system so that it can:

- Detect available Shulker Boxes.
- Detect when an empty Shulker is required.
- Retrieve a Shulker from the appropriate inventory/container.
- Place it when required.
- Transfer items.
- Recover the Shulker.
- Continue operating after storage.

If multiple Shulkers are necessary for continuous operation, support them naturally rather than stopping after a single storage cycle.

Avoid losing items.

Avoid placing containers where they cannot be recovered.

---

# Ender Chest

The Ender Chest is the source for the storage Shulker workflow.

The system should be capable of locating/using an Ender Chest rather than requiring a hard-coded location.

Do not implement endless searching.

If an Ender Chest cannot be located or accessed:

- Stop the current action.
- Enter an appropriate recovery/error state.
- Provide a useful notification/log message.
- Do not repeatedly perform the same failed action every tick.

The exact Ender Chest search strategy should be implemented using capabilities actually available through Minecraft/Baritone/project APIs.

---

# Inventory Threshold

The exact inventory-full threshold shown by the original addon is not known.

Therefore:

- Do not pretend an exact percentage/count was observed.
- Make the threshold configurable.
- Choose a sensible default based on the actual inventory/storage behavior.
- Keep the logic isolated so the default can easily be adjusted after testing.

The storage trigger should occur before items can be lost or normal mining becomes unusable.

---

# Item Filtering

The exact item-filtering behavior of the reference addon is not completely known.

Do not hard-code an arbitrary filtering policy throughout the module.

Create an isolated inventory-management component so item selection can be changed without rewriting the storage system.

At minimum, the implementation should preserve the items that naturally result from the mining operation and avoid accidentally moving essential equipment.

---

# Infinity Miner / Tool Repair

The addon must provide an automated repair system similar to the behavior shown in the video.

Assume the player is using **one pickaxe**.

Monitor the actual durability of the active pickaxe.

The repair system should have configurable thresholds.

Reference behavior:

```text
Approximately 30% durability:
    Stop normal mining.
    Enter repair mode.

Approximately 99% durability:
    Stop repair mining.
    Resume the previous mining objective.
```

These values should be configurable rather than buried as constants.

Do not rely only on elapsed time or estimated blocks mined.

Use the actual tool durability available from Minecraft's item state.

---

# Repair Target Selection

The reference behavior indicates dimension-dependent repair mining.

In the Nether, the demonstrated repair target is:

```text
Nether Quartz Ore
```

For the Overworld, the exact original block priority/list is unknown.

Therefore the implementation should **not invent a claim that a particular list reproduces the original addon**.

Instead, design a repair-target system that can:

1. Detect the current dimension.
2. Select an appropriate XP-producing repair target for that dimension.
3. Prefer the best available target.
4. Fall back to another valid XP source when appropriate.
5. Optionally support XP Bottles as an alternative repair mechanism.

The repair target selection should be isolated behind something similar to:

```text
RepairTargetManager
```

This should allow future adjustment without rewriting the repair state machine.

---

# XP Bottles

The repair system may use XP Bottles when available and appropriate.

Do not make XP Bottles mandatory unless required by the final observed/tested behavior.

If XP Bottles are implemented:

- Detect whether they are available.
- Use them only when appropriate.
- Do not waste them unnecessarily.
- Do not create a busy loop around item usage.
- Verify that the pickaxe actually receives durability repair.

---

# Mending Behavior

The repair system should account for the fact that Mending repairs the tool through collected XP.

Do not assume that merely mining an XP-producing block means the pickaxe is being repaired.

Monitor:

- Pickaxe durability.
- Player XP/orbs where relevant.
- Whether the pickaxe is still the active tool.
- Whether repair progress is occurring.

If repair is not progressing, enter an appropriate recovery/error condition rather than mining indefinitely.

---

# Resuming Mining

After repair or storage:

1. Restore the previous high-level mining objective.
2. Confirm that required conditions are satisfied.
3. Resume mining automatically.

The user should not need to manually restart the mining module after a normal storage or repair cycle.

The system should preserve enough state to know what it was doing before an interruption.

---

# Death / Respawn / Disconnect Recovery

Unattended long-running operation is a core requirement.

The Miner should be designed to recover where technically possible from:

- Player death.
- Respawn.
- Temporary disconnect.
- Reconnection.
- Chunk loading.
- Temporary server lag.
- Temporary Baritone failure.
- Temporary missing target/path.
- GUI/container desynchronization.

Do not assume recovery is always possible.

When recovery cannot safely continue:

- Stop automation.
- Enter an error/recovery state.
- Notify the user.
- Avoid destructive or repetitive behavior.

Do not create an infinite retry loop.

---

# Server/Network Safety

2b2t-style environments may involve:

- Significant latency.
- Delayed inventory updates.
- Delayed block updates.
- Chunk loading delays.
- Container synchronization delays.

Never assume that an interaction succeeded merely because the interaction was requested.

Prefer:

```text
request action
    ↓
wait for observable state change
    ↓
verify result
    ↓
continue
```

over:

```text
click
wait arbitrary fixed delay
assume success
```

Fixed delays may be used where technically necessary, but state-based confirmation should be preferred.

---

# Anti-Busy-Loop Rule

Do not perform the same interaction every tick.

For example, do not:

```text
every tick:
    click chest
```

Instead:

```text
request chest interaction
wait for expected state
verify
continue or retry with a bounded retry policy
```

Every retry should have a reason.

---

# GUI / Container Handling

Inventory and container automation must account for:

- Opening screens.
- Closing screens.
- Slot updates.
- Item movement.
- Container synchronization.
- Player movement.
- Unexpected GUI closure.
- Missing expected container.
- Full containers.
- Network delay.

Never blindly assume a slot contains the expected item after an interaction.

Verify inventory state before proceeding.

---

# Error Handling

Errors should be explicit and recoverable where possible.

Examples:

```text
No mining target found
No suitable pickaxe
Pickaxe cannot be repaired
No suitable XP source
No XP Bottles available
No empty Shulker available
Ender Chest unavailable
Container interaction failed
Baritone task failed
Unable to navigate
Player died
Disconnected
Unexpected GUI state
```

For each error, determine whether the correct response is:

- Retry.
- Recover.
- Pause.
- Notify.
- Disable automation.

Do not silently continue in an invalid state.

---

# Settings

Expose useful behavior through Meteor module settings.

At minimum, consider:

```text
Mining target
Inventory storage enabled
Inventory threshold
Infinity Miner enabled
Repair start threshold
Repair stop threshold
Repair target configuration
XP Bottle repair enabled
Recovery/reconnect behavior
```

Do not expose unnecessary internal implementation details.

Defaults should be sensible but should not falsely claim to match unknown reference values.

---

# Architecture

Prefer small, independently testable components.

A reasonable conceptual architecture is:

```text
Miner
 ├── MinerState
 ├── MinerController
 ├── BaritoneController
 ├── MiningTask
 ├── StorageTask
 ├── RepairTask
 ├── InventoryManager
 ├── DurabilityManager
 ├── RepairTargetManager
 └── RecoveryManager
```

This is a guideline, not a requirement.

Do not create classes merely to satisfy this diagram if a simpler design is clearly better.

The important requirement is separation of responsibilities.

---

# State Machine Rules

Only one high-level task should own the player's automation at a time.

For example:

```text
NORMAL_MINING
      │
      ├── inventory full ──────> INVENTORY_STORAGE
      │                              │
      │                              └──> NORMAL_MINING
      │
      ├── durability low ──────> REPAIR_MINING
      │                              │
      │                              └──> NORMAL_MINING
      │
      ├── death/disconnect ────> RECOVERY
      │                              │
      │                              └──> NORMAL_MINING
      │
      └── fatal error ─────────> ERROR
```

Do not allow normal mining, repair mining, and storage navigation to simultaneously issue conflicting Baritone commands.

---

# User Disable

Disabling the Miner must cleanly stop automation.

On disable:

- Stop/clear Baritone objectives.
- Stop active automation.
- Close or safely exit relevant GUI states.
- Do not leave background tasks running.
- Do not continue moving or interacting after the module is disabled.

The player must regain normal manual control.

---

# Notifications and Logging

Provide useful Meteor notifications for major state transitions and failures.

Examples:

```text
Starting mining
Inventory full — storing items
Starting tool repair
Tool repaired — resuming mining
Searching for Ender Chest
Storage failed
Repair failed
Player died — attempting recovery
Reconnected — attempting recovery
Miner stopped
```

Avoid notification spam.

Detailed diagnostics should be available through debug logging rather than constantly notifying the user.

---

# Version Discipline

This project targets Minecraft 26.2.

Do not introduce code based on:

- Older Minecraft APIs.
- Newer Minecraft APIs.
- Different Fabric mappings.
- Different Meteor Client versions.
- Different Baritone versions.

If an API is uncertain, inspect the actual project.

Do not "fix" compilation errors by blindly changing method names until something compiles.

Determine the correct API first.

---

# Dependency Discipline

Do not add dependencies unless they are actually necessary.

Before adding a dependency:

1. Check whether the project already provides the required functionality.
2. Check the existing Gradle configuration.
3. Check the supplied source.
4. Prefer the project's existing dependency ecosystem.

Do not replace or upgrade dependencies merely to make implementation easier without explicit justification.

---

# Code Quality

The finished plugin should be production-oriented.

Prioritize:

- Correctness.
- Reliability.
- Clear state transitions.
- Safe recovery.
- Maintainability.
- Minimal unnecessary complexity.
- Proper cleanup.
- Useful logging.
- Compatibility with the exact project versions.

Avoid:

- Giant classes containing all automation logic.
- Tick-based spaghetti state machines.
- Blind delays.
- Infinite retries.
- Silent failures.
- Hard-coded coordinates.
- Hard-coded target resources.
- Version-specific guesses.
- Unnecessary abstractions.

---

# Development Workflow

When working on this project:

## 1. Inspect First

Before changing code:

- Inspect the project structure.
- Inspect Gradle configuration.
- Identify exact Minecraft/Fabric/Meteor/Baritone versions.
- Inspect existing addon/module patterns.
- Inspect supplied Baritone source.
- Search for relevant existing APIs.

Do not immediately start writing implementation code.

## 2. Plan

Before substantial changes, identify:

- What component owns the behavior.
- What state transitions are required.
- What APIs are needed.
- What behavior is known versus inferred.
- What can be tested.

## 3. Implement Incrementally

Implement one meaningful subsystem at a time.

Recommended order:

1. Project/module integration.
2. Baritone abstraction.
3. Basic user-selected mining.
4. Durability detection.
5. Repair state machine.
6. Inventory detection.
7. Shulker/Ender Chest storage.
8. Recovery.
9. Combined long-running automation.
10. Polish and configuration.

## 4. Compile Frequently

After meaningful changes:

- Run the project's build/compile task.
- Fix actual compiler/API errors.
- Do not accumulate a large number of unverified changes.

## 5. Test In-Game

Where practical, verify behavior in Minecraft 26.2.

Test individual systems before testing the complete automation loop.

## 6. Refine

When testing reveals behavior that differs from the reference:

- Record the observation.
- Determine whether the difference is known, inferred, or unknown.
- Adjust the implementation.
- Avoid rewriting unrelated systems.

---

# Testing Strategy

At minimum, test:

### Basic Mining

- Select arbitrary block target.
- Start Miner.
- Confirm Baritone mines target.
- Disable Miner.
- Confirm automation stops cleanly.

### Durability

- Start with partially damaged pickaxe.
- Verify durability monitoring.
- Cross repair threshold.
- Verify transition to repair mode.

### Repair

- Nether Quartz repair target in Nether.
- Appropriate Overworld XP target.
- XP Bottle fallback where enabled.
- Verify actual durability increases.
- Verify return to normal mining near repair completion.

### Storage

- Fill inventory.
- Detect storage condition.
- Locate/access Ender Chest.
- Obtain Shulker.
- Place/use Shulker.
- Transfer items.
- Recover Shulker.
- Resume mining.

### Combined

Test:

```text
mine
→ inventory storage
→ resume
→ low durability
→ repair
→ resume
→ inventory storage
→ resume
```

### Recovery

Test, where possible:

- Death.
- Respawn.
- Disconnect.
- Reconnect.
- Temporary Baritone failure.
- Missing target.
- Missing Shulker.
- Missing Ender Chest.
- Unexpected GUI closure.

---

# Unknown Behavior

The following reference details are currently unknown and must not be falsely presented as established facts:

- Exact inventory-full threshold.
- Exact inventory item-filtering rules.
- Exact Ender Chest search strategy.
- Exact Shulker placement location.
- Exact Shulker recovery procedure.
- Whether storage uses one or multiple Shulkers in every situation.
- Exact Overworld XP block priority.
- Exact XP Bottle behavior.
- Exact Baritone settings used by the original addon.
- Exact death/reconnect behavior of the original addon.
- Exact tool-selection logic beyond the current one-pickaxe assumption.
- Any behavior not observable from the reference material.

When one of these matters to implementation, choose a safe, configurable, independently implemented behavior and clearly mark it as an implementation decision rather than a fact about the original addon.

---

# Do Not Overfit to the Video

The video is a behavioral reference, not an implementation specification.

If the video demonstrates:

> "The addon automatically stores items in a Shulker."

Implement the behavior.

Do not assume:

> "Therefore the original addon must have used class X, method Y, a specific Baritone goal, or a specific inventory algorithm."

Those are implementation details unless independently established.

---

# Do Not Ask the User to Implement the Code

The goal is a **finished plugin**, not a Java learning exercise.

When implementing a feature:

- Make the required code changes.
- Compile the project.
- Fix errors.
- Test what can be tested.
- Explain only important decisions and limitations.
- Do not stop at pseudocode when actual implementation is possible.
- Do not ask the user to write routine Java code themselves.

If information is genuinely missing, make the safest reasonable implementation and clearly identify the assumption.

Only ask the user when the missing information materially prevents a correct implementation.

---

# Scope Discipline

Do not modify unrelated parts of the project.

Keep changes focused on the Miner/addon functionality.

Before modifying an existing class, understand why it exists and whether another component depends on it.

Do not perform broad refactors merely for stylistic reasons.

---

# Definition of Done

The project is considered complete when the finished addon can reliably:

1. Run on the target Minecraft 26.2 project.
2. Allow the user to select mining targets.
3. Automatically mine those targets.
4. Detect a low-durability pickaxe.
5. Automatically switch to an appropriate XP repair strategy.
6. Repair the pickaxe.
7. Resume the previous mining target.
8. Detect a sufficiently full inventory.
9. Automatically obtain/use Shulker storage through an Ender Chest.
10. Transfer mining items into storage.
11. Resume mining.
12. Continue these cycles without requiring keyboard input.
13. Recover from expected interruptions where technically possible.
14. Stop cleanly when disabled.
15. Avoid destructive behavior when the required state cannot be established.
16. Compile successfully against the exact project dependencies.
17. Have no known critical state-machine or automation loops.

---

# Final Development Principle

Build for **behavioral reliability**, not superficial similarity.

When information is missing:

1. Identify what is actually known.
2. Identify what is unknown.
3. Implement the safest reasonable abstraction.
4. Make uncertain behavior configurable where practical.
5. Test it.
6. Refine it when new evidence becomes available.

The objective is a robust, finished Minecraft 26.2 Miner addon that provides the observed automation behavior through an independent implementation.