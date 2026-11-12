# Spec 0006 — doctor: an unreadable sensitive-area declaration fails closed (B-4)

- Status: Approved
- Mode: lite (ANEW's own development)
- Plan: `.anew-dev/plans/0006-plan.md`
- Self-critique: done (G-7…G-14 folded in, approved by RifatISLER, 2026-11-12)
- Source: `.anew-dev/backlog.md` B-4 (spec 0002, review round 4: N-1, N-2, I-A; VERIFY F-3)

## Intent
A spec that touches a sensitive area says so with `Sensitive area: <name>` (`docs/domain.md`).
At ship, doctor reads that line to refuse moving a sensitive criterion out (R-08: a sensitive
criterion is proven, never deferred). Today doctor reads only a few spellings and treats every
other one as "not sensitive": measured on v1.24.1, nine plausible declarations pass silently —
`Sensitive area (s): x`, `Sensitive areas touched: x`, `Sensitive area → x`,
`## Sensitive area: x`, `> Sensitive area: x`, `| Sensitive area | x |`, `- Sensitive-area: x`,
`- Sensitive area (x)`, `- Sensitive area x` — and so does a declaration written in backticks
(G-9). One of them in a shipped spec lets a sensitive criterion be moved out with CI green. That
is the one direction doctor must never fail in.

Success means a declaration doctor cannot read makes the spec sensitive (fail closed) and doctor
names the line and the form to write instead, at ship and earlier, while the spec is still in
`specs/active/`; the spellings read correctly today keep their verdict, except the prose of
Requirement 6 (G-7); the rule is proven in both directions by selftest.

Deliberately not done: accepting more spellings (each new spelling is new parsing surface — one
form, and a message that asks for it); a canonical structured field instead of free text (its
own spec, longer term); skipping code fences and HTML comments (B-3); the other backlog items.

## Requirements
1. **A declaration line.** A line counts as a sensitive-area declaration when, after any leading
   Markdown — whitespace, a list bullet or number, blockquote `>`, heading `#`s, a table `|`,
   emphasis, a backtick — its text starts with "sensitive", then a space or a hyphen, then
   "area" (case-insensitive). (C-3, G-9)
2. **Readable declarations keep their verdict.** A declaration line is readable when its leading
   Markdown is only whitespace, at most one list bullet (`-` `*` `+`) or number (`1.` `1)`), and
   emphasis (`*` `_`), and the rest is one of the spellings read today: "sensitive area",
   "sensitive areas" or "sensitive area(s)", optional emphasis, then a separator `:` `—` `–` `-`,
   then a value. Any other leading Markdown (`>`, `#`, `|`, a backtick) makes the line unreadable,
   even with a correct value. A value starting with `none` or `n/a` (past emphasis, quotes or
   brackets) means not sensitive; any other value, an empty one included, means sensitive. (G-8)
3. **An unreadable declaration fails closed.** A declaration line (1) that is not readable (2)
   makes the spec sensitive, and doctor warns:
   `unreadable sensitive-area declaration: <path>:<line> — write "Sensitive area: <name>" or "Sensitive area: none"`.
   The warning is a plain WARN, never a gate, also under `--strict`; doctor gives one per
   unreadable line. At ship, only a moved-out criterion in the now-sensitive spec gates. (C-1,
   G-2, G-10)
4. **Several declarations.** A spec is sensitive when any of its declaration lines is sensitive
   or unreadable; `none` on one line never cancels another. (G-1)
5. **Earlier signal.** doctor also reads the spec files in `specs/active/` (`[0-9]*.md`, full
   and mini template alike; `README.md` is not a spec) and gives the same warning (3) there. In
   `specs/active/` it is a warning only — no gate, exit code unchanged. Specs in `specs/done/`
   that are not new ships are not read: they are never judged again. (C-2, G-4, G-5, G-11)
6. **Prose that looks like a declaration.** A prose line starting with "Sensitive areas are …"
   is a declaration line by (1) and therefore unreadable: the spec counts as sensitive and the
   warning tells the author how to rewrite it. This false gate is accepted — it only bites when
   a criterion is moved out, and the message names the fix. It changes one verdict asserted
   today: the selftest assertion that `Sensitive areas are not touched by this change.` does not
   make a spec sensitive is replaced by its opposite. This spec approves that change; no other
   existing assertion changes its expectation. (G-3, G-7)
7. **Docs say what doctor reads.** `docs/commands.md` (doctor, New ships) states rules 1–5 and
   rewords the criterion rule as "the first box followed by an id decides" (I-A);
   `docs/domain.md` (Sensitive areas comment) says doctor reads only
   `Sensitive area: <name>` / `Sensitive area: none`, written in English (G-12), and fails closed
   on anything else.

## User experience
none — no user interface (doctor's output lines are specified in Requirements 3 and 5).

## Constraints & out of scope
- Out of scope: new accepted spellings (C-4); a structured sensitive-area field; code fences,
  HTML comments, inline code inside a line (B-3); B-1, B-2, B-5, B-6, B-7.
- Out of scope: a localized label (e.g. `Hassas alan: Para`). The label is an English protocol
  field (`docs/conventions.md`, Always English); doctor does not read a translated one. (G-12)
- Unchanged: every other doctor rule and message; doctor's exit codes for specs without a
  declaration line.
- doctor stays POSIX sh + awk, byte-wise (`LC_ALL=C`), as today.
- Release: on `main` as v1.25.0 — Requirement 5 is a new warning, so new behavior — with a
  CHANGELOG entry and `.anew-version` per `README.md` (Versions and support). Whether
  Requirements 1–4 are backported to `lts/1.24` is decided at ship (`.anew-dev/README.md`);
  Requirement 5 never goes to the LTS line. (G-6, G-13)
- B-4 is closed in `.anew-dev/backlog.md` by the ship of this spec.

## Acceptance criteria
- [ ] AC-1 — Readable spellings keep their verdict in a new ship with a moved-out criterion:
  `- Sensitive area: Money`, `- Sensitive areas: Money, Identity`, `- **Sensitive area:** Money`,
  `- Sensitive area — Money` and `- Sensitive area:` (empty) → gate "a sensitive-area criterion
  cannot be moved out"; `- Sensitive area: none` and `- Sensitive area: n/a` → no such gate.
  Every existing sensitive-area assertion in `scripts/selftest` keeps its expectation (except the
  one AC-10 replaces). None of them gives the unreadable warning. (G-14)
- [ ] AC-2 — Each of the ten unreadable spellings (the nine listed in Intent and
  `` `Sensitive area: x` ``), in a new ship with a moved-out criterion → the gate "a
  sensitive-area criterion cannot be moved out", exit 1 under `--strict`.
- [ ] AC-3 — Each of the ten unreadable spellings → the warning of Requirement 3 with the spec's
  workspace-relative path and the line's number, one warning per unreadable line; in a new ship
  with no moved-out criterion, `--strict` exits 0. (G-10)
- [ ] AC-4 — An unreadable declaration in a spec in `specs/active/` → the same warning; no gate;
  doctor's exit code is the same as without the line.
- [ ] AC-5 — `specs/active/README.md`, or a spec already in `specs/done/` that is not a new ship,
  with an unreadable declaration line → no warning. (G-11)
- [ ] AC-6 — A spec with `- Sensitive area: none` and one unreadable declaration line, in a new
  ship with a moved-out criterion → the gate (Requirement 4).
- [ ] AC-7 — A spec with no declaration line → no new warning or gate; doctor's output is
  unchanged for it.
- [ ] AC-8 — `docs/commands.md` and `docs/domain.md` say what Requirement 7 lists.
- [ ] AC-9 — `./scripts/selftest` passes with an assertion for every spelling in AC-1, AC-2,
  AC-3 and AC-10, and `./scripts/doctor --strict` passes on this repository.
- [ ] AC-10 — `Sensitive areas are not touched by this change.` in a new ship with a moved-out
  criterion → the gate and the unreadable warning; the selftest assertion that expected no gate
  for it is replaced by this one (Requirement 6). (G-7)

## Definition of Done
- [ ] Every acceptance criterion mapped to proof (test or reproducible observation)
- [ ] `./scripts/doctor --strict` and `./scripts/selftest` green (ANEW's own check)
- [ ] Independent review done; real findings fixed, noise rejected with written rationale, deferred ones in `.anew-dev/backlog.md`
- [ ] Docs / ADRs updated if behavior or architecture changed
- [ ] Spec moved to `.anew-dev/done/` (it becomes immutable there)

## Scorecard (fill at ship — honest numbers make the process improvable)
| Metric | Value |
|---|---|
| Spec revisions | |
| Fix rounds | |
| Review findings: real / noise / deferred | |
| Regressions introduced | |
| Bugs escaped to production | |
