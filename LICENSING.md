# Jikji Licensing

Jikji is distributed under a controlled dual-licensing model.

## Default License

Unless you have a separate written license grant from the copyright holder,
Jikji is licensed under the **GNU General Public License, version 3 only**
(`GPL-3.0-only`). The complete license text is in [`LICENSE`](LICENSE).

## Apache 2.0 Alternative

The copyright holder may grant a specific recipient permission to use a
specified Jikji release or commit under the **Apache License, Version 2.0**.
That permission is not automatic: it must be provided separately and in
writing. Contact the author through the project contact page at
<https://jikji-labs.com/contact.html> to request it.
The unmodified license text used for named-grant artifacts is retained at
[`LICENSES/Apache-2.0.txt`](LICENSES/Apache-2.0.txt).

Every Apache 2.0 grant includes Jikji's [`NOTICE`](NOTICE). Distributions of
covered derivative works must preserve the NOTICE attribution in one of the
forms permitted by section 4(d) of Apache 2.0. In particular, the distribution
must continue to identify Jikji as the original work. This uses Apache 2.0's
standard NOTICE mechanism and does not modify the Apache 2.0 license text.

If no separate written Apache 2.0 grant identifies both your organization and
the covered Jikji version, the GPL-3.0-only terms apply.

Named-grant binary artifacts are produced only with the explicit
`JIKJI_ARTIFACT_LICENSE=Apache-2.0`, `JIKJI_APACHE_GRANT_ID`, and
`JIKJI_APACHE_GRANT_RECIPIENT` release inputs. A real release additionally
requires a detached, independently signed public grant attestation whose
coverage exactly names the full commit, public commit, and version. The grant
authority public key must match an independently managed fingerprint. Such an
artifact contains the Apache License 2.0 copy as `LICENSE` and records the
public grant binding in its provenance, NOTICE, SBOM, and bundled attestation.
The private written grant is never bundled.

The attestation's validity interval authorizes release packaging; it is not a
new condition on Apache 2.0, does not expire rights already granted under the
separate written grant, and does not restrict downstream recipients. An Apache
dry run without a signed attestation is marked synthetic and non-promotable.

## Why Requests Are Reviewed

The request process exists so the author can understand how early deployments
use Jikji, which operating environments matter, and what support and product
requirements emerge before the licensing model is opened more broadly.
Contact is a request process for the alternative grant; it is not an additional
condition on GPL use, and it does not add restrictions to Apache 2.0 after a
valid Apache 2.0 grant has been issued.

## Contributions

Alternative licensing requires the project to preserve the right to relicense
all contributions. Until a contributor agreement covering that right is
published, external contributions require explicit maintainer approval before
work begins. This avoids accepting code under terms that the project cannot
later license consistently.

This file describes the project's intended licensing process. It is not legal
advice; the final grant and contributor process should be reviewed by counsel.
