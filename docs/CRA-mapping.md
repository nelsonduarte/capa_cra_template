# Mapping this project to the EU Cyber Resilience Act

The Cyber Resilience Act (Regulation (EU) 2024/2847) becomes
mandatory in 2027. Annex I sets the essential cybersecurity
requirements; Annex II sets the information and documentation a
product must carry. This template does not make a product
"CRA-compliant" by itself (compliance is an organisational and
process question), but it has the compiler produce the *technical
evidence* the CRA asks for in CI, not as a separate manual effort.

Each row below names a CRA clause, the evidence this repo produces
for it, and the command that produces it.

## Annex I, Part I, essential requirements

| CRA clause | What it asks for | Evidence here | Command |
| --- | --- | --- | --- |
| (2)(a) "secure by default" | The product ships with no unnecessary exposure | `main` declares only the capabilities it uses (`Stdio`, `Fs`); the type checker refuses a call on a built-in capability that is not in scope | `capa --check main.capa` |
| (2)(b) protection from unauthorised access | Authority is narrowed | The `Fs` capability is attenuated with `restrict_to("data/")`; a path outside `data/` is refused at runtime by the narrowed `Fs` | `capa --check main.capa` |
| (3)(e) "reducing attack surfaces" | Demonstrate minimised attack surface | The SBOM lists, per function, the capabilities the compiler found no path to (`Net`, `Proc`, `Db`, `Env`, `Clock`, `Random`, `Unsafe`); no function in the program holds them | `capa --cyclonedx main.capa` |
| (3)(f) minimise impact of incidents | Limit blast radius | Same capability record: the capabilities each function holds, so a function that gains one shows up in the next SBOM diff | `capa --cyclonedx main.capa` |

## Annex I, Part II, vulnerability handling

| CRA clause | What it asks for | Evidence here | Command |
| --- | --- | --- | --- |
| (1) SBOM | "a software bill of materials in a commonly used and machine-readable format" | CycloneDX 1.5 and SPDX 2.3 SBOMs emitted directly by the compiler | `capa --cyclonedx main.capa` / `capa --spdx main.capa` |
| (2) address and remediate vulnerabilities | A documented exploitability position | A VEX document (Vulnerability Exploitability eXchange) | `capa --vex main.capa` |
| (8) secure distribution | Integrity of the released artifact | SLSA L2 build provenance (Sigstore-attested) plus a GPG-signed tag | the `release.yml` workflow |
| (1)/(8) verifiable evidence | The evidence above can be rebuilt and diffed | With `SOURCE_DATE_EPOCH` pinned, repeated builds of the same commit are meant to be byte-identical; CI checks it on every push by regenerating the pack and failing on any byte difference. A rebuild-and-diff is a check to run, not a guarantee | `SOURCE_DATE_EPOCH=$(git show -s --format=%ct HEAD) capa --cyclonedx main.capa` (and the other emitters) |

## Annex II, information and instructions

| CRA item | Evidence here |
| --- | --- |
| 1 single point of contact | `SECURITY.md` |
| coordinated vulnerability disclosure policy | `SECURITY.md` |
| SBOM availability | attached to every GitHub release by `release.yml` |

## What Capa adds here

Most toolchains can emit an SBOM of *dependencies*. The thing the
CRA's attack-surface clauses (Part I (3)(e)/(f)) actually ask for is
harder: evidence about a component's *authority*. Because Capa makes
capabilities part of the type system, the SBOM carries, per function,
the capabilities it holds and the ones the compiler found no path to,
derived from type-checked signatures rather than from a scan, and it is
the reason this template exists.

The evidence can also be rebuilt and diffed: every timestamp is pinned
to the commit's date via `SOURCE_DATE_EPOCH` and newlines are pinned to
LF via `.gitattributes`, and CI regenerates the pack on every push and
fails on any byte difference. A rebuild-and-diff is a check to run, not
a guarantee.

## A regulator-readable audit pack (optional)

The artifacts above are machine-readable. To turn them into a
Markdown audit pack plus a JSON attestation that a GRC platform can
ingest, pipe the SBOM through
[`capa_governance_pack`](https://github.com/nelsonduarte/capa_governance_pack):
it reads a CycloneDX SBOM, a governance policy, and a VEX list, and
emits `audit_pack.md` + `attestation.json`. A sample `policy.json`
lives in this repo to get you started.
