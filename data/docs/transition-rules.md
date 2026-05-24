# Yoga — Transition Rules

A living document of methodology rules that govern how poses connect to each other in the Yoga Maya curriculum. These rules apply to:

- Authoring `practice.entry_cues` and `practice.exit_cues` in asana YAMLs — no rule-violating transitions.
- Authoring `variations` — no rule-violating implicit transitions. A pose listed as a variation should be a true deepening of the same pose, not a separate pose that happens to share a stance or a Sanskrit root.
- Authoring `relationships.entry_transitions` and `relationships.exit_transitions` — same rule.
- The sequencing engine — any generated sequence must respect these rules.

The rules are derived from Yoga Maya's teaching methodology. They are not universal yoga rules — other lineages may allow transitions that Yoga Maya does not.

## Pose classes (reference)

The schema's `classification.primary_class` field uses these values:

- Open Standing Pose
- Neutral Standing Pose
- Standing Twist
- Seated Twist
- Core Work
- Arm Balance
- Inversion
- Forward Bend
- Seated Forward Bend
- Back Bend
- Side Bend

Rules below reference these class names.

---

## Rule 1 — Open Standing → Standing Twist must pass through Neutral

**Statement.** A practitioner cannot transition directly from an Open Standing Pose (e.g., Warrior 2, Trikonasana, Utthita Parsvakonasana) to a Standing Twist (e.g., Parivrtta Trikonasana, Parivrtta Parsvakonasana). The transition must pass through a Neutral Standing Pose — typically Tadasana or Uttanasana — first.

The reverse direction (Standing Twist → Open Standing) follows the same rule: pass through Neutral.

**Rationale.** *[To be filled in — Yoga Maya's specific reasoning. Likely related to safe alignment of the SI joint, preparing the spine for rotation under load, and pedagogical clarity. Tamar to confirm.]*

**Examples.**

- ❌ Trikonasana → Parivrtta Trikonasana (directly)
- ✓ Trikonasana → Tadasana → Parivrtta Trikonasana

- ❌ Utthita Parsvakonasana → Parivrtta Parsvakonasana (directly)
- ✓ Utthita Parsvakonasana → Tadasana → Parivrtta Parsvakonasana

**Substring trap.** Pose names that share a Sanskrit root (Trikonasana / Parivrtta Trikonasana; Parsvakonasana / Parivrtta Parsvakonasana) are tempting to treat as variations of each other. They are not. The `parivrtta` prefix ("revolved") changes the pose's `primary_class` from Open Standing Pose to Standing Twist, which puts them on opposite sides of this rule. Always check `classification.primary_class`, not the name.

**How this surfaces in the schema.**

- A revolved pose must never appear as `from` or `to` of an entry/exit cue on an open standing pose, or vice versa.
- A revolved pose must never appear in the `variations` list of an open standing pose, or vice versa — even when the names share a substring.
- A revolved pose must never appear in `relationships.entry_transitions` or `relationships.exit_transitions` of an open standing pose, or vice versa.

---

## Rule 2 — Mirror semantics for cross-pose transitions

**Statement.** When both the source and destination of a transition are `classification.sides: two`, the transition either *preserves* or *swaps* which anatomical side is in play. A transition is **side-swapping** when the "active" leg in the source pose's canonical orientation becomes the "opposite" leg in the destination pose's canonical orientation. Side-swapping transitions must be marked with `mirror: true` on the relevant `entry_cues[]` or `exit_cues[]` alternative. Side-preserving transitions either omit the field or set `mirror: false`.

**Rationale.** Each pose's cues are authored from a single canonical orientation (typically "right side first"). The sequencer mirrors right ↔ left at display time to produce the second side. When transitioning between two `sides: two` poses, the sequencer needs to know whether the practitioner stays on the same side or switches sides — otherwise it cannot render the destination pose correctly.

The need arises because different poses have different natural canonical orientations. In Vrksasana the canonical "active" leg is the *lifted* leg (right foot on left inner thigh). In Virabhadrasana I the canonical "active" leg is the *front* leg (right foot forward). The natural transition between them — lifted leg becomes back leg — necessarily swaps which anatomical leg sits at each pose's canonical position.

**Examples.**

- **Vrksasana → Virabhadrasana I**: the lifted leg in Tree becomes the back leg in Warrior 1. Tree canonical is right-foot-up; Warrior 1 canonical is right-foot-forward. The right leg goes from "active" in Tree to "back" in Warrior 1 — a side-swap. **`mirror: true`.**
- **Vrksasana → High Lunge**: same logic — lifted leg becomes back leg. **`mirror: true`.**
- **Vrksasana → Utthita Hasta Padangusthasana**: lifted leg in Tree becomes extended leg in UHP. Same anatomical leg stays "active" in both canonicals. **`mirror: false`** (or omit).
- **High Lunge → Virabhadrasana I**: front leg stays the front leg; only the back heel position changes. **`mirror: false`.**
- **Utthita Trikonasana → Ardha Chandrasana**: front leg in Triangle becomes standing leg in Half Moon. Same anatomical leg stays "active." **`mirror: false`.**

**Consistency requirement.** When `pose_A`'s `exit_cues` entry to `pose_B` is marked `mirror: true`, `pose_B`'s `entry_cues` entry from `pose_A` (if authored) must also be marked `mirror: true`. The same fact lives in both YAMLs and they must agree. This is duplicative but lets each pose's YAML be self-sufficient as a reference document.

**Scope.** `mirror` is only meaningful when both the source and destination are `sides: two`. When either pose is `sides: one` (symmetric), the flag has no semantic effect and should be omitted.

**Not applicable to `relationships`.** `relationships.entry_transitions` and `relationships.exit_transitions` are graph topology only. The mirror flag lives in `entry_cues` / `exit_cues`. The sequencer derives mirror state from the cue blocks, not from the relationships graph.

---

## Future rules (candidates to discuss)

Placeholders for rules to flesh out as they come up:

- **Counter-pose conventions.** When a back bend should be followed by a forward bend (or vice versa) through neutral; the role of Balasana as a recovery pose; etc.
- **Hip-opener warm-up.** Whether deeper hip-openers (e.g., Pigeon, Frog) require warm-up sequences and what counts as adequate warm-up.
- **Inversion safety.** Contraindications, appropriate counter-poses after inversions (e.g., Tadasana before and after Sirsasana).
- **Seated ↔ standing transitions.** Conventions for how the practice moves between seated and standing portions.
- **Twist conventions.** Whether twists have ordering rules (e.g., always to the right first), and whether seated twists follow different transition rules than standing twists.
- **Vinyasa structure.** Whether Yoga Maya has a defined "core vinyasa" that always appears as a connector, and what its terms are.
- **Parent ↔ deepening pose conventions.** Whether a deepening pose (e.g., Sugarcane from Half Moon, Bound Half Moon from Bound Side Angle) should only be reachable from its parent pose, or whether other transitions are allowed.

---

## Schema authoring checklist

When populating a new asana YAML, run through this list:

1. For each `entry_cues[].from` and `exit_cues[].to`, identify the other pose's `primary_class` and confirm the transition is allowed under all rules above.
2. For each `variations[]` entry, confirm the listed pose is a true deepening — same pose family, same primary class — not a separate pose with a similar name.
3. For each `relationships.entry_transitions` and `relationships.exit_transitions` entry, apply the same check as (1).
4. For each `entry_cues[]` and `exit_cues[]` alternative where both source and destination are `sides: two`, determine whether the transition is side-swapping (Rule 2). If yes, set `mirror: true`.
5. For each side-swapping transition, verify that the corresponding entry on the other pose's YAML (if it exists) agrees: source's `exit_cues` and destination's `entry_cues` must have the same `mirror` value.
6. If unsure about a transition, route through a Neutral Standing Pose (Tadasana or Uttanasana) and let the sequencer chain through it.
