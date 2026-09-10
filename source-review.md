# Source review: signatures, web archives, and witnesses

Checked 2026-09-10 using public primary sources. Standards and documentation establish the claims below; the hypothetical failure scenarios and suggested acceptance checks are this review's defensive analysis. No proof was fabricated and no external submission was made.

## 1. Signed attestations

**RFC 9421, HTTP Message Signatures (February 2024).**

- Section 3 says successful verification establishes semantic equivalence for the subset of HTTP message components that was signed. Sections 7 and 7.3.4 additionally require the verifier to check that the key, algorithm, time boundaries, and covered content fit the application. A mathematically valid signature alone does not establish the appropriate signer.
- Section 1.1 defines signature creation time as a time asserted by the signer. It is not an independently observed task-execution or artifact-creation time.

Canonical links: [covered components, section 3](https://www.rfc-editor.org/rfc/rfc9421.html#section-3); [identity/context, section 7.3.4](https://www.rfc-editor.org/rfc/rfc9421.html#section-7.3.4); [creation time definition, section 1.1](https://www.rfc-editor.org/rfc/rfc9421.html#section-1.1).

**Sigstore, Verifying Signatures — Keyless verification using OpenID Connect.**

- Identity-based verification supplies the expected certificate identity and expected OIDC issuer; accepting an arbitrary certificate is not equivalent to authenticating the intended producer.
- Cosign's normal image verification also compares the signed image digest with the image being checked. Signature verification and application claim checks are distinct checks.

Canonical link: [Sigstore identity and digest verification](https://docs.sigstore.dev/cosign/verifying/verify/#keyless-verification-using-openid-connect).

**Conceptual false-evidence scenario:** a genuine signer attests to a successful run using an unchecked self-report. The signature is authentic, but the claimed execution was not observed. This requires no broken cryptography.

**Defensive verification:** validate the intended signer and covered statement; bind the statement to the assigned task, expected inputs and exact output digest. Require a trusted execution record or separate observation for the execution claim. A fresh challenge can address replay, but cannot make a dishonest statement true. Do not describe a signer-supplied timestamp as independent time evidence.

## 2. Third-party web archives

**RFC 7089, Memento (December 2013).**

- Section 2.1.1 says `Memento-Datetime` identifies the datetime of the prior resource state represented by the response; the `original` relation identifies the original resource. Section 4.5.6 requires those values to remain stable for the Memento.
- Section 7 explicitly leaves archive veracity outside the protocol's scope. Even honest archives may hold different states for the same resource and datetime because the original server varies responses by client IP, headers or other factors.

Canonical links: [datetime meaning, section 2.1.1](https://www.rfc-editor.org/rfc/rfc7089.html#section-2.1.1); [stable archive metadata, section 4.5.6](https://www.rfc-editor.org/rfc/rfc7089.html#section-4.5.6); [trust limits, section 7](https://www.rfc-editor.org/rfc/rfc7089.html#section-7).

**Internet Archive, Using the Wayback Machine.**

- The help page explains that JavaScript requiring the original server can fail in an archived replay.
- Incomplete archived navigation may select another available capture date or a live-web resource. Check the actual capture date and destination of each relevant resource rather than assuming the initial page's date covers everything displayed.

Canonical link: [Wayback capture and replay limitations](https://help.archive.org/help/using-the-wayback-machine/), questions “Why are some sites harder to archive than others?” and “How did I end up on the live version of a site?”

**Conceptual false-evidence scenario:** a real archive preserves a page saying a task succeeded, but its result text is merely the publisher's assertion. The archive can corroborate what was available to its crawler without corroborating the underlying execution. A replay containing material from different dates can further mislead a reviewer.

**Defensive verification:** retrieve the archive record directly, verify original URL and actual capture metadata, inspect redirects and important dependencies, and distinguish captured response content from live or later resources. Describe the archive as evidence of an observed representation, subject to archive trust and capture limitations; obtain execution evidence separately. Capture time does not establish when the artifact was first created.

## 3. Third-party witnesses and transparency logs

**SLSA v1.2, Build: Verifying artifacts.**

- Step 1 requires verifying the provenance envelope against configured trust roots, comparing the statement's `subject` with the artifact digest, checking the predicate type, and evaluating recognized signing identities together with `builder.id`.
- Step 2 compares expected builder, canonical source repository, build type and external parameters. Even Build L3 assumes a trusted build platform; the stated protection does not cover compromise of the platform itself.

Canonical links: [signature, subject and builder verification](https://slsa.dev/spec/v1.2/verifying-artifacts#step-1-check-slsa-build-level); [expected source and parameters](https://slsa.dev/spec/v1.2/verifying-artifacts#step-2-check-expectations).

**Sigstore, Rekor.**

- Rekor stores signed supply-chain metadata and exposes entry inclusion proofs and log-integrity verification.
- Its auditing section describes monitoring append-only consistency and identities; `omniwitness` is named as a log-auditing option. This is witnessing the log's behavior, not witnessing the real-world work described by each entry.

Canonical links: [Rekor purpose and inclusion](https://docs.sigstore.dev/logging/overview/); [log auditors and witnesses](https://docs.sigstore.dev/logging/overview/#auditing-the-public-instance).

**Conceptual false-evidence scenario:** a signed self-report appears in an authentic transparency log and several services reproduce it. Multiple copies of one claim do not become multiple independent observations of execution. A log witness's signature can concern log consistency rather than the task.

**Defensive verification:** verify the entry, signer, subject digest and expected provenance, then identify exactly what each third party observed and who controls its evidence collection. For an independent execution witness, require a separately controlled observer's direct record of the relevant run and resulting artifact. Do not claim that log inclusion, consistency, or a log-recorded time proves factual execution or the artifact's original creation time.

Editorial distinction: “signature valid,” “entry included,” “archived representation observed,” and “execution independently observed” are different conclusions. State only the conclusion supported by the verified evidence and its trust assumptions.
