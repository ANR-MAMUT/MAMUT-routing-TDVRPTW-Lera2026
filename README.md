# Lera2026 TDVRPTW benchmark family (TD-IGP-Gehring&Homberger, 200–1000 customers)

Satellite benchmark repository of [MAMUT-routing](https://github.com/ANR-MAMUT/MAMUT-routing) (ANR MAMUT, ANR-22-CE22-0016), mounted there as `benchmarks/TDVRPTW/Lera2026`. Self-contained: instances and best-known solutions ship together; the time-dependent travel times are stored as a compact IGP specification (`igp-profile` model) and materialize deterministically on load — no committed arrival-time-function sidecars (a complete n=1000 ATF sidecar would weigh ~60–90 MB gzipped).

The family is named in honour of **Gonzalo Lera-Romero**, whose open-source exact TDVRPTW solver lineage (Lera-Romero et al. 2020) anchors modern work on the problem. The name is purely honorific: Lera-Romero is not an author of this family, and it is distinct from the Lera-Romero branch-price-and-cut code vendored elsewhere in the MAMUT ecosystem.

## What this family is — and what it is not

This family does for time-dependent routing what Gehring & Homberger (1999) did for Solomon's VRPTW testbed: it scales the classic IGP time-dependent model (Ichoua, Gendreau, Potvin 2003) from 100 to 200–1000 customers. Base data are the **Gehring & Homberger VRPTW instances** (n = 200, 400, 600, 800, 1000; 60 per size) exactly as curated in MAMUT-routing's `Sintef2008` family (origin `GehHom1999`): coordinates, demands, service times, fleet sizes, capacities and time windows carried over verbatim except for the minimal time-window repair described below. Travel speeds follow **IGP 2003 Table 1** (3 arc categories, average speed ≈ 1) placed in the **Dabia et al. (2013) five-period skeleton** (rush periods at 20–30% and 70–80% of the original horizon, profile `[mid, rush, mid, rush, mid]` per category) — the period placement that protects G&H's early tight deadlines, adopted after a full feasibility audit.

It is therefore **not** the raw G&H testbed (results are not comparable with static VRPTW literature), and **not** the historical IGP 2003 experimental setup either (that used Solomon-100 with soft time windows and three equal periods). The Solomon-based IGP testbed at n ≤ 100 lives in the `Dabia2013` family of MAMUT-routing.

## Two tiers: canonical S2 core + S1/S3 intensity ladder

Scenario = rush slowdown factor a from IGP 2003 Table 1 (rush speed = midday speed ÷ a per category):

- **`S2/` (a = 2), the canonical core**: all 300 G&H bases — every G&H instance has exactly one canonical TD twin (the Dabia2013 philosophy).
- **`S1/` (a = 1.5) and `S3/` (a = 4), the intensity ladder**: the deterministic subset `_1`, `_5`, `_10` of each class (C1/C2/R1/R2/RC1/RC2), 18 per size, 90 per scenario. A base's S1/S2/S3 triplet shares one arc-category assignment, so the three instances differ only in the speed values — a controlled congestion ladder on identical geometry. S3 is the extreme tier: its time-window repair is substantial (see below); treat it as a stress benchmark.

480 instances total, named `Lera-<GH name>-S<k>` under `S<k>/n=<customers>/`.

## Minimal time-window repair (read before comparing anything)

G&H deadlines assume unit travel speed; under IGP speeds (slow category ≈ 0.33–0.67 at S2) some customers become unreachable by **any** route, which is ill-posed under hard time windows. Each instance therefore carries the **minimal deterministic repair**: a customer deadline is lifted to the earliest possible time-dependent arrival (`l_i' = max(l_i, ceil(α_0i(0)))`) and the depot deadline to the worst singleton return; the five periods stay anchored to the *original* horizon (the last period extends), so repairs never change travel times. Repair magnitudes are recorded per instance in `metadata.tw_repair` (S2 lifts ~a few deadlines per affected instance; S3 substantially more). Instances are exactly reproducible from `Sintef2008` + this rule.

## Format (`igp-profile` model)

- `<Name>.vrp.json` — MAMUT shape: depot 0, coordinates/demands/service times/time windows/fleet from the G&H base, `horizon` `[0, repaired depot due date]`, and a `td` block `{model: "igp-profile", time_periods, speeds, categories_path, categories_sha256, atf_sha256}`. The stored `time_periods` floats are canonical.
- `<Base>.igp.json.gz` — arc-category sidecar (`format: "mamut-td-igp-categories"`): explicit symmetric category matrix over the complete graph as digit strings (seeded uniform, seed recipe in instance metadata), shared by the scenario triplet of a base name, hashed over its uncompressed canonical bytes.
- **No ATF sidecar**: the canonical arrival-time functions (one non-decreasing continuous PWL function per arc, exact IGP consolidation with the last period's speed extended beyond the horizon) are **materialized on load** by `mamut_routing_lib.td` (≥ the `igp-profile`-capable release; see MAMUT-routing) from coordinates (Euclidean `sqrt(dx*dx+dy*dy)`), categories, periods and speeds. `atf_sha256` pins the materialized canonical bytes exactly as a committed sidecar's hash would — integrity is verifiable offline from published data alone.
- `<Name>.bks.Duration.json` — best known solution under the `Duration` objective; costs are always the canonical checker's output (`mamut_routing_lib.td.check_td_solution`).

Population pipeline: `populate_td_lera2026` v1 (curation tooling maintained outside this repository; links will be published with the benchmark paper). Every instance was cross-validated against the direct forward Ichoua loop and hard-audited for post-repair singleton feasibility.

## Provenance and licensing

Underlying data: Gehring & Homberger (1999) VRPTW instances (as curated in MAMUT-routing `Sintef2008`); time-dependent model: Ichoua, Gendreau, Potvin (2003), EJOR 144(2), 379–396; period skeleton: Dabia et al. (2013). MAMUT-authored curation artifacts under the MIT license; this notice does not relicense the underlying third-party benchmark definitions.
