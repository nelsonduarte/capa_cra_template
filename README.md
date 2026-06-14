# capa_cra_template

A starter [Capa](https://github.com/nelsonduarte/capa-language)
project that is **CRA-ready from commit zero**. Clone it, replace the
sample program with your own, and you inherit a build that emits the
technical evidence the EU Cyber Resilience Act asks for: a software
bill of materials, a documented attack surface, a VEX exploitability
position, and signed build provenance, all as a by-product of
compiling, not a separate manual effort.

## Why a template

Every modern toolchain can emit an SBOM of dependencies. The CRA's
attack-surface clauses (Annex I, Part I, (3)(e)/(f)) ask for
something harder: evidence that a component's authority is *bounded*
and that whole classes of behaviour are *absent*. Capa makes
capabilities part of the type system, so the compiler can prove and
record which capabilities a program does **not** use. This template
wires that proof into CI and releases so you do not have to.

See [docs/CRA-mapping.md](docs/CRA-mapping.md) for the clause-by-
clause mapping.

## What is in here

```
main.capa                  the sample program (capability-bounded)
data/components.txt        sample input so it runs out of the box
capa.toml                  package manifest (toolchain version pin)
policy.json                governance policy (for the optional audit pack)
docs/CRA-mapping.md        CRA clause -> evidence -> command
SECURITY.md                disclosure contact + supply-chain model
.github/workflows/ci.yml   check, run, regenerate artifacts on every push
.github/workflows/release.yml  tag -> SBOM + VEX + SLSA provenance on the release
```

## Use it

```
capa --check main.capa     # type + capability check
capa --run main.capa       # run against data/components.txt
capa --fmt-check main.capa # formatting gate
```

Generate the conformity artifacts locally:

```
capa --cyclonedx  main.capa > sbom.cyclonedx.json
capa --spdx       main.capa > sbom.spdx.json
capa --vex        main.capa > vex.json
capa --provenance main.capa > provenance.json
```

## Reproducible by construction

Two builds of the same commit produce **byte-identical** artifacts, on
any operating system. This is a CRA-relevant property: the evidence is
not just available, it is verifiable and reproducible, so an assessor
can rebuild it and diff against what shipped and get zero differences.

CI proves this automatically: after generating the five artifacts it
regenerates them a second time (same commit, same epoch, a different
`PYTHONHASHSEED`) and fails the build if any byte differs.

To reproduce locally, pin the build timestamp to the commit before
running the emitters:

```
export SOURCE_DATE_EPOCH=$(git show -s --format=%ct HEAD)
capa --cyclonedx  main.capa > sbom.cyclonedx.json
capa --spdx       main.capa > sbom.spdx.json
capa --vex        main.capa > vex.json
capa --provenance main.capa > provenance.json
capa --manifest   main.capa > manifest.json
```

Determinism comes from two pieces: `SOURCE_DATE_EPOCH` (the
reproducible-builds.org convention) pins every timestamp field to one
instant instead of the wall clock, and `.gitattributes` pins newlines
to LF so a CRLF checkout cannot perturb the bytes. Without
`SOURCE_DATE_EPOCH` the emitters use real wall-clock time and two
builds diverge by design.

See the capabilities the program provably cannot reach (a capability
is excluded program-wide only when every function excludes it, so this
intersects the per-function proofs):

```
capa --manifest main.capa | python -c "import json,sys; \
fns=json.load(sys.stdin)['functions']; \
sets=[set(f['provably_excluded_capabilities']) for f in fns]; \
print('\n'.join(sorted(set.intersection(*sets))))"
```

## Make it yours

1. Rename the package in `capa.toml` and the project in this README.
2. Replace `main.capa` with your program. Declare only the
   capabilities you need; attenuate them (`restrict_to`) as early as
   possible.
3. Update `data/` (or drop it) and `policy.json`.
4. Set your disclosure contact in `SECURITY.md`.
5. Optional: commit your GPG public key as `publisher.asc` to turn on
   tag-signature verification in `release.yml`.
6. Tag a release (`git tag -s v0.1.0 && git push --tags`); CI attaches
   the conformity pack and SLSA provenance to it.

## A regulator-readable audit pack (optional)

The artifacts above are machine-readable. To turn them into a
Markdown audit pack plus a JSON attestation for a GRC platform, run
[`capa_governance_pack`](https://github.com/nelsonduarte/capa_governance_pack)
over the SBOM, `policy.json`, and `vex.json`.

## License

Apache-2.0. See [LICENSE](LICENSE).
