Given a statement S, produce a hierarchical, MECE map of:

What S means (conceptual dependencies), and

Why S is true/valid (epistemic dependencies).

Apply the same two-branch procedure to every node recursively.

Definitions

Conceptual dependency: minimal concept/term/relation needed for the parent to be meaningful (“What must be understood?”).

Epistemic dependency: minimal fact/derivation/measurement that justifies the parent (“How do we know?”).

Primitive of definition: a basic term taken as given within a domain (e.g., identity, set membership).

Empirical primitive: a terminal observation/tautology with instrument + calibration identified.

MECE: siblings are mutually exclusive and collectively exhaustive.

Domain neutrality: don’t assume a field unless forced; annotate with [domain] when needed.

Controller (defaults)
max_depth: 10               # recursion depth cap per branch (default set to 10)
max_width: 12               # child cap per node before bucketing
domain: ""                  # e.g., "mathematics|measurement theory|computability"
output_format: "tree"       # "tree" | "json"
dedupe_threshold: 0.8
transport_required: auto    # auto|true|false
primitive_expansion_policy: "expand_all"   # "expand_all" | "allowlist_only"
axiom_systems:
  - "ZFC set theory"
  - "Type theory (HoTT)"
  - "Peano Arithmetic (PA)"
  - "Kolmogorov probability"
  - "IEEE-754 floating point"
  - "Unicode / ISO 10646"
root_primitives:
  - "identity"
  - "set membership"
evidence_mix_target:
  empirical_min: 0.0        # raise if domain implies measurement

Global Rules (apply at every node)

Two branches required.
Every node must have:

Conceptual dependencies (what it means), and

Epistemic dependencies (why it’s true/valid).
If a branch is intentionally empty, include a stopping_reason.

Recursion.
For each child, attach its own two branches and continue recursively.

Stopping (strict).

Conceptual stopping:
May stop only at root_primitives, or when leaving the domain and max_depth is hit. Otherwise, expand putative primitives into:

Formal bases (named axioms/definitions/standards), and

Implementation substrates (encodings, instruments, computational models).

Epistemic stopping:

May stop at empirical primitives only after naming:

instrument model,

calibration/traceability,

recorded observable (log/bitstring/tautology).

May stop at formal primitives only after naming:

the axiom system, and

the specific theorem/derivation invoked.

Always set stopping_reason ∈ {primitive_of_definition, empirical_primitive, out_of_scope, max_depth} for any leaf.

Operators (required on every child).
Allowed operators and branch whitelist:

Conceptual branch allowed: DEFINE | SPECIALIZE | DECOMPOSE | REDUCE | INSTRUMENT | TRANSPORT

Epistemic branch allowed: PROVE | COMPUTE | COUNTEREXAMPLE | INSTRUMENT | TRANSPORT
(Validator flags others as operator_mismatches.)

Failure modes (per-child).
Each node includes a failure_modes list; each item must:

reference exactly one child by label (child_ref), and

state how a minimal perturbation of that child falsifies the parent (mechanism, not rhetoric).

Transports (cross-frame coherence).

Use transport edges when multiple foundations/substrates appear (e.g., PA and ZFC):

{ "from": "<source>", "to": "<target>",
  "kind": "definitional|interpretation|conservativity|equiconsistency|implementation|calibration",
  "evidence": ["<proof/standard ids>"] }


Transports may appear inline (as TRANSPORT children) or in the dedicated Transports section of the node.

With transport_required = true, or auto and ≥2 distinct foundations/substrates detected in the subtree, the root must include a Transports section connecting all such pairs; else validator reports transport_gaps.

MECE enforcement.

Distinctness test: merge siblings if they can be merged without loss.

Coverage test: if removing any sibling breaks the parent’s sufficiency, restore it and record which test failed in the validator.

Granularity.

One idea per line; prefer relations (is-a / part-of / uses / implies).

Do not echo the parent text as a child without adding information.

No circularity / no branch mixing.

Evidence must not presuppose the parent.

Do not place evidence under conceptual, or concepts under epistemic.

Width/Depth guardrails.

Width guard: if a node would exceed max_width, insert a single DECOMPOSE: Taxonomy child that buckets siblings (each bucket ≤ max_width) and recurse inside buckets.

Depth guard: require at least one path from the root to a terminal leaf with stopping_reason ∈ {primitive_of_definition, empirical_primitive}; otherwise emit a coverage_violations entry (“no terminal justification reached”).

Output Format — Tree
<Label>
 ├─ Conceptual dependencies
 │    ├─ <OPERATOR>: <Child 1>
 │    │    ├─ Conceptual dependencies ...
 │    │    ├─ Epistemic dependencies ...
 │    │    └─ Failure modes ...
 │    └─ <OPERATOR>: <Child 2>
 │         ├─ Conceptual dependencies ...
 │         ├─ Epistemic dependencies ...
 │         └─ Failure modes ...
 ├─ Epistemic dependencies
 │    ├─ <OPERATOR>: <Evidence 1> [provenance: ...]
 │    │    ├─ Conceptual dependencies ...
 │    │    ├─ Epistemic dependencies ...
 │    │    └─ Failure modes ...
 │    └─ <OPERATOR>: <Evidence 2> [provenance: ...]
 │         ├─ Conceptual dependencies ...
 │         ├─ Epistemic dependencies ...
 │         └─ Failure modes ...
 └─ Transports
      ├─ {from: ..., to: ..., kind: ..., evidence: [...]}
      └─ ...


If a branch is empty due to stopping, write — (primitive) or — (out of scope) and set stopping_reason on that node.

Output Format — JSON

(Operator and stopping_reason are mandatory.)

{
  "root": {
    "label": "string",
    "kind": "statement|concept|evidence|method|relation",
    "conceptual": [ /* Node */ ],
    "epistemic":  [ /* Node */ ],
    "failure_modes": [ /* Failure */ ],
    "transports": [ /* TransportEdge */ ],
    "notes": "optional string"
  },
  "metrics": {
    "atomicity": 0.0,
    "mece_overlap": 0.0,
    "max_depth_used": 0,
    "branching_hist": { "0": 1 },
    "evidence_mix": { "empirical": 0.0, "formal": 1.0, "other": 0.0 }
  },
  "validation_report": {
    "missing_branches": [],
    "leaves_without_stopping_reason": [],
    "cycles_detected": [],
    "coverage_violations": [],
    "concept_evidence_misplacements": [],
    "operator_mismatches": [],
    "transport_gaps": [],
    "provenance_missing": []
  }
}


Node schema:

{
  "label": "string",
  "operator": "DEFINE|SPECIALIZE|DECOMPOSE|REDUCE|TRANSPORT|INSTRUMENT|PROVE|COMPUTE|COUNTEREXAMPLE",
  "kind": "concept|evidence|method|relation|law|definition|measurement|dataset|derivation",
  "conceptual": [ /* Node */ ],
  "epistemic":  [ /* Node */ ],
  "failure_modes": [
    { "child_ref": "string", "perturbation": "string", "falsifies": "string" }
  ],
  "transports": [
    { "from": "string", "to": "string",
      "kind": "definitional|interpretation|conservativity|equiconsistency|implementation|calibration",
      "evidence": ["string"] }
  ],
  "provenance": ["string"],
  "stopping_reason": "none|primitive_of_definition|empirical_primitive|out_of_scope|max_depth",
  "domain": "optional string"
}

Provenance (mandatory for epistemic leaves)

Formal: ["theory:<id>", "theorem:<id>"]

Empirical: ["standard:<id>", "device:<model>", "certificate:<id>"]

Computation: ["impl:<lang|lib>", "env:<isa/os>", "artifact:<hash|path>"]

Metrics (computed)

atomicity ∈ [0,1] — brevity & term-uniqueness.

mece_overlap ∈ [0,1] — average Jaccard overlap of sibling glossaries (must be ≤ 1 - dedupe_threshold).

max_depth_used: int

branching_hist: { depth: count }

evidence_mix: { empirical: f, formal: f, other: f }

Post-Generation Validator (must be empty for valid output)

missing_branches: [paths]

leaves_without_stopping_reason: [paths]

cycles_detected: [paths]

coverage_violations: [{parent_path, removed_child, reason}]

concept_evidence_misplacements: [paths]

operator_mismatches: [paths]

transport_gaps: [pairs to be connected]

provenance_missing: [epistemic leaf paths]

Execution note: The engine outputs only the decomposition, plus metrics and validation_report. It must not add a concluding narrative.
