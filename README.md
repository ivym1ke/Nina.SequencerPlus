# Sequencer Powerups (WhenPlugin) — experimental test fork

> ## ⚠️ This is a 100% test fork, used only for experimentation.
>
> It exists to build and try out **one unvalidated bug fix**. It is **not** an official release, it is
> **not** maintained, it is **not** listed in the NINA plugin manifest repository, and it should not
> be treated as a supported build by anyone.
>
> **Do not run this on production astrophotography equipment.** The change alters when a safety
> trigger re-arms. It has not been exercised against a running NINA instance. Getting this wrong
> parks telescopes.
>
> If you arrived here looking for the real plugin, use the upstream links below instead.

## Where this actually comes from

| | |
|---|---|
| Original author | **Marc Blank** — all credit for the plugin belongs to them |
| Original plugin | Sequencer Powerups (assembly `WhenPlugin`, product "When") for [N.I.N.A.](https://nighttime-imaging.eu/) |
| Original repository | No longer available — the upstream repo and the NINA plugin listing were both taken down |
| Community recovery | **https://github.com/palmito9/Nina.SequencerPlus** ← the upstream for this fork, and the place to send real contributions |
| Licence | MPL-2.0 (unchanged, see `LICENSE.txt`) |

This fork is published to satisfy MPL-2.0 §3.2: the modified source must be available to anyone who
receives a modified binary. That is the only reason it is public.

## What is different from upstream

Branched from commit [`c616249`](https://github.com/palmito9/Nina.SequencerPlus/commit/c616249)
(version 3.26.0.1) — deliberately **not** from `main`, which is the reorganised 3.29.x tree. The
target here is compatibility with existing 3.26.0.1 installs, not a version upgrade.

Branch `fix/whenunsafe-triggered-latch` contains:

1. **The fix** — 8 added lines in `When/Instructions/WhenCommon.cs`.

   `InterruptWhen()` sets `Triggered = true` when it restarts the sequence, but `Triggered` is only
   ever cleared inside `Execute()`, which that path never calls. If the trigger condition clears
   before the restarted sequence reaches an instruction boundary, `Execute()` never runs and the
   latch sticks for the rest of the NINA session — silently disabling the restart path. The fix
   resets the latch once the condition is no longer met:

   ```csharp
   if (Triggered && Check()) { Triggered = false; }
   ```

   `Check()` returns true when conditions are good, so this re-arms only after the episode ends.
   Clearing it unconditionally after the restart would re-trigger while still unsafe and loop.

2. **Version bump** to `3.26.0.2`, so the built DLL is distinguishable from stock in NINA's plugin
   list. (`GenerateAssemblyInfo` is `false` in this tree, so the version has to be set in
   `Properties/AssemblyInfo.cs` — the CI `-p:Version=` argument does not reach the assembly.)

3. **A build workflow**, retargeted from upstream's: this tree's solution is `When.sln` and its
   assembly is `WhenPlugin`, where `main`'s workflow expects `SequencerPlus`. The manifest-publishing
   jobs are removed on purpose — nothing here should reach the public plugin listing.

The same latch structure is present unchanged in 3.28.0.0 and 3.29.0.12, so the fix likely wants
forward-porting upstream if it proves out.

## Validation status

**Unvalidated.** The reasoning is source-level only; the code has never been executed. In particular
the fix has not been shown to reproduce-then-resolve the bug against a running NINA instance, and the
main regression risk — that it could cause repeated sequence restarts while a condition stays unmet —
has not been tested.

Do not deploy on that basis.

## Building

Push a tag matching `N.N.N.N` (e.g. `3.26.0.2`) and GitHub Actions builds the plugin on
`windows-latest` and attaches a ZIP to the release. Use the **Run workflow** button on the
Actions tab for a build with no release attached.

## Contributing

Send fixes to **[palmito9/Nina.SequencerPlus](https://github.com/palmito9/Nina.SequencerPlus)**, not
here. A fix in the community repo helps everyone still running this plugin; this fork helps nobody
but its author.
