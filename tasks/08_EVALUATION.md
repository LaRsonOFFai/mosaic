# Task M8 — Reproducible evaluation harness

## Operating mode

Plan first. Keep evaluation local or on operator-controlled lab infrastructure. Do not scan or probe third-party systems.

## Goal

Create a reproducible framework to measure functionality, overhead, latency, and selected traffic-classification features of MOSAIC modes against the project's own ordinary decoy baseline.

## Required design document

Create `docs/EVALUATION_METHOD.md` specifying:

- research question;
- environments and ownership boundary;
- dataset generation;
- labels;
- train/validation/test split without session leakage;
- features;
- models;
- metrics;
- confidence/variance reporting;
- limitations;
- reproducibility metadata;
- data-retention and privacy policy.

## Required implementation

Add `tools/evaluation/` containing a clearly documented offline workflow that:

1. imports packet/event metadata from local captures or simulator output;
2. removes payload contents and unnecessary identifiers;
3. groups data by session;
4. extracts features such as size, direction, inter-arrival, burst length, and duration;
5. separates sessions across train/test partitions;
6. trains at least one simple interpretable baseline classifier;
7. reports ROC-AUC and TPR at selected low FPR values;
8. reports useful throughput, padding overhead, and added latency;
9. saves configuration, random seed, code revision, and summary results;
10. generates a Markdown report with caveats.

Prefer a small auditable pipeline. Do not build an automated adversarial evasion optimizer in this milestone.

## Dataset classes

At minimum:

- ordinary decoy application without MOSAIC;
- MOSAIC fast mode;
- balanced mode;
- strict mode;
- optional synthetic no-encryption local baseline clearly separated from security conclusions.

Do not use copyrighted or private third-party packet captures without permission.

## Required tests

- feature extraction on fixed miniature fixtures;
- no session leakage across splits;
- deterministic split for seed;
- metrics on a known toy dataset;
- malformed input handling;
- payload fields are not retained in derived datasets;
- report includes limitations even when results are favorable;
- end-to-end smoke test on generated simulator metadata.

## Interpretation rules

The report must not say “undetectable” or “bypasses every DPI.” It must state:

- exact dataset and environment;
- classifier family;
- sample count;
- FPR/TPR operating points;
- overhead and latency tradeoff;
- uncertainty and limitations;
- whether results reproduce across seeds.

## Constraints

- No public network scanning.
- No active probing of unrelated servers.
- No automated generation of attack traffic.
- No collection of other users' payloads.
- No model that automatically mutates the protocol to evade a target system.

## Acceptance criteria

A clean local run generates labeled metadata, trains the baseline classifier, emits performance/classification metrics, and creates a reproducible report. Results are phrased as bounded experimental findings. Only M8 is implemented.
