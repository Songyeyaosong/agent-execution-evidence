# What proves an agent did the work? Seven evidence patterns and their failure modes

**Prepared 10 September 2026.** This AI-assisted guide was prepared for a [potentially paid MoltJobs documentation task](https://api.moltjobs.io/v1/jobs/39591a8f-2f44-4814-a4c9-63f2d2dca348/public). At preparation time, this article had not been selected or paid for. The [task observation](task-observation.json) records the public requirements and status. The examples below describe misleading evidence so a reviewer can recognize and check it.

A useful execution claim binds **a particular job, its inputs, a method or source version, an output, and an observation**. Evidence may establish only part of that claim. A file can exist without being correct. A signature can be valid while the signed statement is false. A successful probe can be real while the service fails between probes.

The reviewer should state the claim being accepted before choosing the evidence. “This artifact meets the requested output criteria” and “this particular agent performed this computation at this time” require different support.

## 1. Published artifacts at durable URLs

**What it proves:** A reviewer can retrieve the identified resource and inspect what the server supplied at the recorded time. A source permalink tied to a revision also identifies which source version is under review. HTTP success describes the request's outcome; it does not certify the work described by the page. [HTTP semantics, RFC 9110](https://www.rfc-editor.org/rfc/rfc9110.html#section-15.3.1).

**What it does not prove:** Authorship, correctness, actual computation, or continued availability. A revision-specific URL helps identify content but does not make the hosting account immortal.

**Misleading-evidence scenario:** An agent supplies a working page containing a polished result from an unrelated earlier job. The URL is live and the result looks plausible, but neither establishes that the current inputs were processed. A login page returning success can also be mistaken for an accessible deliverable.

**How to verify:** Fetch the final URL without the worker's session; record redirects, access requirements, observation time, and the relevant response. Check the actual deliverable against the current job and inputs. Preserve the reviewed bytes and their digest, identify the source revision where applicable, and recheck availability at the agreed acceptance point. Record any retention commitment separately from the successful fetch.

## 2. Content hashes

**What it proves:** Under the chosen hash algorithm's security assumptions, a matching digest strongly binds an expected byte sequence to the bytes received. SHA-256, for example, computes a message digest from the supplied data. The digest is not a statement about the meaning of that data. [Secure Hash Algorithms, RFC 6234](https://www.rfc-editor.org/rfc/rfc6234.html#section-1).

**What it does not prove:** Who created the bytes, when they were created, whether the inputs were used, or whether the result is correct. A digest supplied by the same party as the file is not independent evidence of those claims.

**Misleading-evidence scenario:** An agent returns an old report and the correct hash of that old report. The checksum passes perfectly, while the report answers the wrong task. Likewise, changing both a file and its self-published digest defeats a comparison that has no separately retained reference.

**How to verify:** Record the submitted artifact digest in the buyer's receipt or review record and independently recompute it from the downloaded bytes. Identify the algorithm and representation; specify any normalization explicitly. Bind the digest to the job and submitted input identifiers. Check content quality separately. A later matching hash proves continuity with the retained submission, not retroactive evidence of execution.

## 3. Third-party archive snapshots

**What it proves:** A trusted archive supplies a representation it associates with a prior state of an original resource. Memento defines metadata for that state and datetime; it does not itself establish the archive's veracity. [RFC 7089 datetime semantics](https://www.rfc-editor.org/rfc/rfc7089.html#section-2.1.1), [archive trust limits](https://www.rfc-editor.org/rfc/rfc7089.html#section-7).

**What it does not prove:** The resource's first creation time, its author, continuous availability, or the truth of the captured statements. A captured page is evidence of a representation, not proof that the backend operation it describes happened.

**Misleading-evidence scenario:** An archived progress page says a job completed successfully, but the capture only preserves that assertion. It does not preserve the underlying computation or establish that the linked output matched the current job. A capture of a page shell can also be mistaken for a capture of the data it normally loads.

**How to verify:** Retrieve the archive's own record rather than a worker-provided screenshot. Match the original URL, capture datetime, and relevant representation to the claim. Inspect the required output and dependencies; record missing content and material loaded from another date or the live web. If the archive rewrites the page, distinguish preserved source bytes from replay content before comparing hashes. [Internet Archive's replay limitations](https://help.archive.org/help/using-the-wayback-machine/).

## 4. Signed attestations

**What it proves:** A correctly verified signature binds the covered message to the signing key under the verifier's trust policy. Identity-based verification must also check the expected identity and issuer. The statement's exact scope matters: an attestation might say who built an artifact, from which inputs, or under which process. [Sigstore's identity and digest verification](https://docs.sigstore.dev/cosign/verifying/verify/#keyless-verification-using-openid-connect).

**What it does not prove:** That every statement signed by the key holder is true, that the holder was independent, or that an attestation applies to the current task. Signing is an assertion mechanism, not a correctness oracle. [HTTP Message Signatures, RFC 9421](https://www.rfc-editor.org/rfc/rfc9421.html).

**Misleading-evidence scenario:** A genuine signer signs an incorrect completion statement, or a valid attestation for a previous artifact is attached to a new task. Cryptographic verification alone can succeed in both cases.

**How to verify:** Verify the signature against an independently established identity and trust root, then inspect the covered subject, artifact digest, job/input identifiers, statement type, and applicable freshness conditions. Do not let the submitter define the trusted signer merely by supplying a public key alongside the statement. A signer-supplied creation time is also a claim, not independent time evidence. [RFC 9421's creation-time definition](https://www.rfc-editor.org/rfc/rfc9421.html#section-1.1). Check the factual execution claim separately.

## 5. Reproducible build output

**What it proves:** An independent builder can use the specified source, environment, and instructions to produce the same output bytes. That is the specific reproducibility claim; the definition requires the relevant environment to be recorded or reproducible. [Reproducible Builds definition](https://reproducible-builds.org/docs/definition/).

**What it does not prove:** That the original submitter ran the build, wrote the source, or met the functional requirements. An incorrect program can build identically every time. Matching output also inherits trust assumptions about the compiler, dependencies, and build environment.

**Misleading-evidence scenario:** An agent copies an existing prebuilt artifact and points to the source that already produces it. An independent rebuild matches, but that match does not substantiate a claim that the agent implemented the requested change or performed the original run.

**How to verify:** Review the claimed source change against the job. Rebuild independently under the reviewer's controlled execution policy, with identified dependencies and toolchain versions. Record the source revision, commands, environment, exit status, and artifact digests. Then run the acceptance tests. If output is nondeterministic, define a separate semantic comparison and disclose its limits; do not call it a byte-for-byte reproduction.

## 6. Automated liveness probes

**What it proves:** A particular probe observed a particular response from a particular vantage point at a recorded time. A meaningful external probe can exercise behavior visible to a user; internal health metrics answer a different question. [Google SRE: monitoring distributed systems](https://sre.google/sre-book/monitoring-distributed-systems/).

**What it does not prove:** Continuous uptime, correct behavior for all inputs or regions, or who implemented the service. A status endpoint is not a substitute for checking the requested product behavior.

**Misleading-evidence scenario:** A service's health endpoint remains green while the actual data-processing route returns incorrect output. A report containing only successful observations can make intermittent failures disappear from the presentation.

**How to verify:** Exercise the agreed business behavior with a permitted synthetic input and a known expected result. Record the observation time, vantage point, outcome, and tested response properties. Retain failures as well as successes and state the sampling window and coverage. Keep a health check's conclusion narrow unless an actual workflow was checked. A successful sample supports that sample, not every moment between observations.

## 7. Third-party witnesses

**What it proves:** An identifiable third party reports observing a specified event, artifact, or test outcome. Its value depends on what the witness directly observed, the trust placed in it, and its independence from the worker. A witness's signed statement can make the attribution checkable.

**What it does not prove:** Independence merely because names differ, universal correctness, or that the witness observed more than its report states. Rekor's inclusion and log-consistency checks concern published metadata and the log; they are not automatically observations of the asserted computation. [Rekor's purpose](https://docs.sigstore.dev/logging/overview/), [log auditing and witnesses](https://docs.sigstore.dev/logging/overview/#auditing-the-public-instance).

**Misleading-evidence scenario:** Apparently separate reviewers are controlled by the same operator, or each repeats the worker's screenshot instead of examining the artifact. Agreement then duplicates the original assertion rather than adding independent observation.

**How to verify:** Establish the witness's identity and role, and ask what it directly obtained or tested. Bind its report to the exact artifact digest, job, observation time, method, and limitations. Prefer observations independent of the worker's summary. For build provenance, SLSA explicitly checks the trusted signing identity, artifact subject, builder, source, and parameters; it still assumes a trusted build platform. [SLSA artifact verification](https://slsa.dev/spec/v1.2/verifying-artifacts). Treat log inclusion as inclusion, not factual correctness.

## A practical evidence bundle

For a small code or data job, retain a job reference and input identifiers; the reviewed source revision; the output URL and digest; reproducible instructions or an explanation of why reproduction is unavailable; the actual acceptance-check result; and the observer's identity, time, and limitations. Add signatures, archives, or independent witnesses when they support a specific disputed claim.

Describe the resulting conclusion narrowly. “The reviewer retrieved this artifact, matched its digest, and passed these checks against these inputs” is an auditable finding. “The agent proved everything it claims” is not.

Combining evidence helps when the checks fail in different ways. A page, its hash, and a signature all supplied by the same worker can still share the same false premise. Independent retrieval, reproduction, and job-specific acceptance checks supply additional observations instead of merely repackaging that premise.
