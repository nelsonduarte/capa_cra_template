# Security policy

`capa_cra_template` is a starter Capa project. Its security story is
structural: the program is written in [Capa](https://github.com/nelsonduarte/capa-language),
so per-function capability discipline is enforced by the type
checker. `main` declares only `Stdio` and `Fs`, the `Fs` capability
is attenuated to `data/`, and the SBOM the compiler emits
(`capa --cyclonedx main.capa`) proves that `Net`, `Proc`, `Db`,
`Env`, `Clock`, `Random`, and `Unsafe` are unreachable.

## Reporting a vulnerability

Email **security@example.com** with the subject prefix
`[security] capa_cra_template: ...`. Replace this address with your
own single point of contact before you ship; the EU CRA (Annex II)
requires a contact for coordinated vulnerability disclosure.

Include a reproducer if possible: the input data, the command line,
and the observed vs. expected behaviour. We aim to acknowledge
reports within 5 working days.

## Supply-chain integrity

Each tagged release carries:

- a CycloneDX and an SPDX SBOM emitted by the Capa compiler;
- a VEX exploitability document;
- SLSA L2 build provenance, attested through Sigstore and published
  to the Rekor transparency log.

Verify a downloaded tarball:

```
gh attestation verify capa_cra_template-<tag>.tar.gz --owner <you>
```

To add tag-signature verification, commit your public key as
`publisher.asc`; the release workflow then runs `git verify-tag`
before building.
