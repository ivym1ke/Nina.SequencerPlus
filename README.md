# Sequencer+ — experimental personal fork

> ## ⚠️ This is a 100% test fork, used only for experimentation.
>
> **This repository is not the Sequencer+ project.** It is one person's fork, used to build and try
> out unvalidated changes. It is **not** an official release, it is **not** maintained, it is
> **not** listed in the NINA plugin manifest repository, and nobody should treat a build from here
> as supported.
>
> **Do not run builds from this fork on production astrophotography equipment.** The changes here
> touch when a safety trigger re-arms and which expression engine the plugin binds to. No build from
> this fork has ever been loaded in a running NINA instance. Getting this wrong parks telescopes.
>
> If you came looking for the real thing, go to the community recovery repository:
> **[palmito9/Nina.SequencerPlus](https://github.com/palmito9/Nina.SequencerPlus)**. Everything below
> this box is that project's documentation, kept as-is.

## What is different from upstream

Forked from `cb4e7ae` (3.29.0.12). Two changes, both released from this fork and both unvalidated:

1. **`WhenUnsafe` re-arms after a short unsafe episode** (`3.29.0.13`).
   `WhenCommon.InterruptWhen()` sets `Triggered = true` on the restart path, but `Triggered` is only
   ever cleared inside `Execute()`, which that path never calls. If the monitor returns to safe
   before the restarted sequence reaches an instruction boundary, the latch sticks for the rest of
   the NINA session and silently disables the restart path — leaving the unsafe handler to park the
   mount while the sequence keeps exposing. Fixed by re-arming once the condition clears:

   ```csharp
   if (Triggered && Check()) { Triggered = false; }
   ```

2. **NCalc 5.2.11 → 7.0.0** (`3.29.0.14`). NINA 3.3 ships its own NCalc 7 and wins the assembly
   name race; NCalc 6 moved `Expression` from `NCalc.Sync` into `NCalc.Core`, so a v5-compiled
   plugin asks for a type that is no longer there and composition fails.

A separate branch, `fix/whenunsafe-triggered-latch`, carries the same two fixes on the older 3.26.x
tree (assembly `WhenPlugin`, namespaces `WhenPlugin.When.*`) for installs that predate the
Sequencer+ rename. That line additionally needs a `4.x` version number, because NINA 3.3 hardcodes
the old plugin's identifier in `PluginCompatibilityMap` with a minimum version of `4.0.0.0`.

### Status

**Unvalidated.** The reasoning behind both changes is source-level. Neither has been exercised
against a running NINA instance, on 3.2 or 3.3.

## Provenance and licence

Original plugin by **Marc Blank**, released under MPL-2.0. The community recovery that this forks
from is [palmito9/Nina.SequencerPlus](https://github.com/palmito9/Nina.SequencerPlus). This fork is
public to satisfy MPL-2.0 §3.2 — modified source must be available to anyone who receives a modified
binary. That is the only reason it is public.

---

## Introduction

This repository is a community recovery of the **Sequencer Powerups** plugin for N.I.N.A. (Nighttime Imaging 'N' Astronomy).

The plugin was originally developed and maintained by Marc Blank, who released it under the **Mozilla Public License 2.0 (MPL-2.0)**. At some point, he chose to remove the plugin from the N.I.N.A. plugin repository and delete the original GitHub repository, along with all associated documentation.

Under the Mozilla Public License 2.0, the license grant is irrevocable: anyone who received the software while it was publicly available retains the rights to use, study, modify, and redistribute it. The MPL also explicitly requires that source code be made available when binaries are distributed, meaning the community has both the legal right and, arguably, the standing to preserve and maintain this work.

This repository was reconstructed from a local clone of the original repository.

### Original Source Archive

A release attached to the initial recovery commit contains a complete archive of the original repository as it existed at v3.29.0.1, including all 51 branches. It is provided for historical reference and reproducibility.

The release includes:
- `WhenPlugin-source.zip` — original source at v3.29.0.1
- `WhenPlugin.dll` — the corresponding original binary
- `ExtendedNumerics.BigDecimal.dll`
- `Microsoft.Extensions.Logging.Abstractions.dll`
- `Microsoft.Extensions.Logging.Configuration.dll`
- `Microsoft.Extensions.Logging.Console.dll`
- `Microsoft.Extensions.Logging.dll`
- `Microsoft.Extensions.Logging.TraceSource.dll`
- `Microsoft.Extensions.Options.dll`
- `NCalc.Core.dll`
- `NCalc.Sync.dll`
- `Parlot.dll`
- `FIlter_Logger.flt`

### Branch Structure

The original repository contained 51 branches. To keep the repository manageable while preserving history, the following reorganisation was applied:

- Branches with unique commits were retained under a `historic/` prefix
- All other branches, whose commits were already reachable from the main line, were deleted
- `newdevelop` (the former stable branch) was renamed to `main`
- `main` (stale experiment) was renamed to `historic/develop-powerupslite`
- `develop` is a fresh branch created from `main`

**Preserved historic branches:** `historic/beta`, `historic/develop`, `historic/develop-powerupslite`, `historic/lite`, `historic/litedockable`, `historic/load`, `historic/master`, `historic/onerror`, `historic/smart`, `historic/smf`

The complete original branch set, including all 51 branches, is available in the `WhenPlugin-source.zip` archive attached to the initial recovery release.

---

## About Sequencer+

Sequencer+ is the continuation of Sequencer Powerups under a new name — a clean break required to avoid conflicts with the original plugin. It is a powerful plugin for N.I.N.A.'s Advanced Sequencer that brings programmatic control to your imaging sequences. Where the built-in sequencer provides a fixed set of instructions with static parameters, Sequencer+ lets you introduce variables, expressions, constants, conditional logic, reusable functions, and dynamic triggers — turning your imaging sequence into something closer to a programmable automation script.

It is particularly valuable for users who want to build sophisticated, reusable, and self-adapting sequences, especially for remote or unattended observatory operation.

### Variables and Constants

At the heart of Sequencer+ is a variable system. You can define **Constants** (fixed key-value pairs) and **Variables** (values that can change during sequence execution) and reference them throughout your sequence using expressions. This means you can define exposure time or gain once at the top of your sequence, and have every instruction that needs it read from that single definition — no more hunting through dozens of instructions to change one value.

### Expressions

Most numeric parameters in enhanced instructions accept **Expressions** — mathematical or logical formulas that are evaluated at runtime. Expressions can reference variables, constants, and live NINA data such as current HFR, focuser position, or temperature. This allows sequences to adapt dynamically to conditions as they evolve through the night.

### Enhanced Instructions

Sequencer+ provides enhanced versions of many built-in NINA instructions, identified by a `+` suffix (e.g. `Smart Exposure+`, `Cool Camera+`, `Switch Filter+`, `Wait for Time+`, `Annotation+`). These enhanced versions accept Expressions in their parameter fields, allowing dynamic rather than static configuration. For example, `Cool Camera+` can set the target temperature from a variable rather than a hard-coded value.

### Template by Reference

One of the most powerful features is **Template by Reference**, which turns NINA templates into something analogous to subroutine calls. Rather than embedding a copy of a template's instructions into every sequence that needs them, Template by Reference keeps a single master template and references it by name. When the master template changes, every sequence using it updates automatically. This dramatically reduces the maintenance burden for users who manage complex or multiple sequences.

### Functions

Building on Template by Reference, Sequencer+ introduces **Functions**: templates that can be called with up to six arguments and can return a value by setting a variable. Functions can even be recursive. This makes it possible to build genuinely reusable logic, such as computing a value, making a decision, or encapsulating a complex multi-step procedure that varies based on input.

### Triggers

Sequencer+ adds four new **Triggers** to complement NINA's built-in ones:

- **When / When Becomes Unsafe** — unlike standard NINA triggers which only fire between instructions, these react within seconds of a safety condition changing, enabling rapid response to weather or observatory safety events.
- **DIY Trigger** — lets you separate the *condition* of an existing trigger from its *action*, so you can reuse a trigger's detection logic while substituting your own response instructions.
- **Autofocus Trigger** — a one-time trigger you can drop into a running sequence to fire an autofocus as soon as the current instruction completes, useful when you spot a problem during an unattended session.
- **Interrupt Trigger** — similar to Autofocus Trigger but lets you specify any instruction(s) to execute, rather than being limited to autofocus.

### DIY Meridian Flip

A fully customisable version of the meridian flip. Rather than executing a fixed flip procedure, DIY Meridian Flip exposes all the individual steps — waiting to pass the meridian, performing the flip, recentering, resuming guiding — as editable instructions inside a container. You can reorder, add, or remove steps to suit your exact setup and workflow.

### External Script+

An enhanced version of NINA's External Script instruction with two key additions: variable values can be passed into the called batch script, and a single return value can be received back into the sequence via the reserved word `EXITCODE`. This opens the door to integrating external tools, logging systems, or custom hardware control into a NINA sequence.

---

## License

Mozilla Public License 2.0 — see [LICENSE](LICENSE).

## Contributing

**Not here.** This fork is a personal scratch space and does not accept contributions. The community
recovery project is [palmito9/Nina.SequencerPlus](https://github.com/palmito9/Nina.SequencerPlus);
anything raised here will not reach it.