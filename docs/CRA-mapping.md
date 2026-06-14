# Mapping this project to the EU Cyber Resilience Act

The Cyber Resilience Act (Regulation (EU) 2024/2847) becomes
mandatory in 2027. Annex I sets the essential cybersecurity
requirements; Annex II sets the information and documentation a
product must carry. This template does not make a product
"CRA-compliant" by itself (compliance is an organisational and
process question), but it makes the *technical evidence* the CRA
asks for a by-product of the build, not a separate manual effort.

Each row below names a CRA clause, the evidence this repo produces
for it, and the command that produces it.

## Annex I, Part I, essential requirements

| CRA clause | What it asks for | Evidence here | Command |
| --- | --- | --- | --- |
| (2)(a) "secure by default" | The product ships with no unnecessary exposure | `main` declares only the capabilities it uses (`Stdio`, `Fs`); the type checker rejects any use beyond them | `capa --check main.capa` |
| (2)(b) protection from unauthorised access | Authority is bounded | The `Fs` capability is attenuated with `restrict_to("data/")`; code below that line cannot touch any other path | `capa --check main.capa` |
| (3)(e) "reducing attack surfaces" | Demonstrate minimised attack surface | The SBOM lists the **provably excluded** capabilities (`Net`, `Proc`, `Db`, `Env`, `Clock`, `Random`, `Unsafe`): the program cannot reach the network, spawn a process, etc., and the compiler proves it | `capa --cyclonedx main.capa` |
| (3)(f) minimise impact of incidents | Limit blast radius | Same capability bound: a compromised dependency cannot exceed the authority the type system grants it | `capa --cyclonedx main.capa` |

## Annex I, Part II, vulnerability handling

| CRA clause | What it asks for | Evidence here | Command |
| --- | --- | --- | --- |
| (1) SBOM | "a software bill of materials in a commonly used and machine-readable format" | CycloneDX 1.5 and SPDX 2.3 SBOMs emitted directly by the compiler | `capa --cyclonedx main.capa` / `capa --spdx main.capa` |
| (2) address and remediate vulnerabilities | A documented exploitability position | A VEX document (Vulnerability Exploitability eXchange) | `capa --vex main.capa` |
| (8) secure distribution | Integrity of the released artifact | SLSA L2 build provenance (Sigstore-attested) plus a GPG-signed tag | the `release.yml` workflow |
| (1)/(8) verifiable evidence | The evidence above is itself reproducible | Two builds of the same commit produce byte-identical artifacts on any OS, so an assessor can rebuild and diff to zero. CI proves it on every push by regenerating the pack and failing on any byte difference | `SOURCE_DATE_EPOCH=$(git show -s --format=%ct HEAD) capa --cyclonedx main.capa` (and the other emitters) |

## Annex II, information and instructions

| CRA item | Evidence here |
| --- | --- |
| 1 single point of contact | `SECURITY.md` |
| coordinated vulnerability disclosure policy | `SECURITY.md` |
| SBOM availability | attached to every GitHub release by `release.yml` |

## What is unique here

Most toolchains can emit an SBOM of *dependencies*. The thing the
CRA's attack-surface clauses (Part I (3)(e)/(f)) actually ask for is
harder: evidence that authority is bounded and that whole classes of
behaviour are *absent*. Because Capa makes capabilities part of the
type system, the SBOM can carry a machine-checkable list of
capabilities the program provably does not use. That negative proof
is the evidence a vulnerability scanner cannot give you, and it is
the reason this template exists.

The evidence is also reproducible by construction: two builds of the
same commit yield byte-identical artifacts on any operating system,
because every timestamp is pinned to the commit's date via
`SOURCE_DATE_EPOCH` and newlines are pinned to LF via `.gitattributes`.
CI enforces this on every push (it regenerates the pack and fails on
any byte difference), so the conformity evidence is not merely
available but independently verifiable: an assessor can rebuild it and
diff against what shipped to zero.

## A regulator-readable audit pack (optional)

The artifacts above are machine-readable. To turn them into a
Markdown audit pack plus a JSON attestation that a GRC platform can
ingest, pipe the SBOM through
[`capa_governance_pack`](https://github.com/nelsonduarte/capa_governance_pack):
it reads a CycloneDX SBOM, a governance policy, and a VEX list, and
emits `audit_pack.md` + `attestation.json`. A sample `policy.json`
lives in this repo to get you started.
