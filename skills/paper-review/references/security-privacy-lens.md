<!-- Adapted in part from Security Conference Paper Skills, sec-conf-write (MIT; Copyright (c) 2026 Security Conference Paper Skills contributors), modified by the KIT Privacy & Security group. Licence texts and credits: NOTICE.md and LICENSES/ in this skill folder. -->
# Security & privacy lens

Apply in Step 3 of the review procedure. Findings go into the normal Weaknesses list with a
severity. Venue rules change every cycle: confirm each venue note below in the live call for
papers (procedure Step 6) before reporting it.

## 1. Threat-model card (fill in; put it in the review)

| Item | What the paper says (location) | Problem? |
|---|---|---|
| Violated or enforced property (confidentiality, integrity, privacy, ...) | | |
| Attacker: position and capabilities | | |
| Attacker: limitations (what it cannot do) | | |
| Protected asset and trust boundary | | |
| Trusted components and assumptions (TCB, honest parties, white-box access, ...) | | |
| Out-of-scope cases | | |

- Missing or vague items ("a malicious attacker") are Major. A threat model that makes the
  attack trivial or the defense irrelevant in practice is Blocking or Major.
- Realistic attacker and justified assumptions, followed by method and evaluation? Why do
  existing work and tools miss this? What shows the result is real, prevalent or worth fixing?

## 2. Checks by contribution type

- **Defense / mitigation:** evaluated against an **adaptive attacker** who knows the defense?
  Only non-adaptive or fixed attacks is Major (often Blocking for ML defenses). Bypass
  analysis, security argument or proof sketch? Overhead in absolute numbers? False blocking
  of legitimate behaviour?
- **Attack:** new, or a known technique on a new target? Preconditions common enough to
  matter? End-to-end success shown on real systems/versions?
- **Vulnerability class / analysis tool:** precise violation condition? Ground truth for
  precision and recall? Causes of false positives and negatives? Baseline tools configured fairly?
- **Measurement:** data source, time range, filters and counts stated? Labeling rules and
  disagreement handling? Claims limited to the sampled population?
- **ML for security / security of ML:** is the security problem primary, or is it mainly ML
  benchmarking? Train/test leakage (time, same app/family)? Concept drift?
- **Crypto protocols / PETs (MPC, HE, PIR, ZK, searchable encryption, anonymity systems):**
  security model stated (semi-honest/malicious, corruption threshold, (non-)colluding
  servers)? Ideal functionality or game, and leakage profile, stated and proven? Costs under
  LAN and WAN, against the state of the art at the same security level?
- **Privacy attacks (membership/attribute inference, reconstruction):** TPR at low FPR, not
  only accuracy or AUC? Members and non-members from the same distribution (a "blind"
  baseline that ignores the model should be near chance)? Realistic attacker prior?
- **User studies:** sample size and recruitment platform, pre-registration or a power
  argument, ecological validity, deception and debriefing, compensation?
- **SoK:** new insight beyond a taxonomy, transparent corpus selection? Venue: CCS 2026 took
  no SoK or survey papers; S&P and PETS want a "SoK:" title prefix.

## 3. Impact accounting

Keep separate, and check the paper does too: raw alerts, author-confirmed findings,
vendor-confirmed bugs, fixes, CVEs or advisories, pending reports. Alert counts presented as
vulnerabilities, or percentages without absolute counts, are Major.

## 4. Disclosure, ethics and misuse

- **Disclosure:** vendors told? Status and timeline stated? IEEE S&P requires disclosure no
  later than the rebuttal deadline and no full CVE identifiers in the anonymous submission.
  Unpatched exploit details redacted?
- **Live systems:** experiments on real services, networks or users: permission stated? Harm limited?
- **Human subjects / personal data:** IRB or ethics-board approval or waiver stated (NDSS: an
  exemption alone may not be enough)? Consent, storage, retention, de-identification?
- **Misuse / dual use:** tool and attack papers discuss misuse and mitigation (S&P asks for it)?
- "No ethical concerns" despite live systems, vulnerabilities, user data or exploit tooling
  is Major; ethics can be a reject reason on its own at S&P, NDSS and USENIX.

## 5. Open science and artifacts

- USENIX Security and CCS require an **Open Science appendix**: which artifacts, where, and
  why any are withheld. Missing it is Blocking when the live CFP requires it.
- Artifact links anonymized, no tracking, not updated after the deadline (S&P)? Paper complete
  without optional appendices (USENIX reviewers need not read them)?

## 6. Venue fit (confirm in the live CFP)

- **CCS:** track chosen and justified; practical importance to security and privacy.
- **IEEE S&P:** clear security/privacy application; theory papers show relevance to practice.
- **NDSS:** clear relation to real networks and systems; not an ML paper with a security wrapper.
- **USENIX Security:** finished and complete; ethics judged for conduct and implications.
- **PETS/PoPETs:** strong tie to privacy in digital systems, with the real-world privacy link
  on the first page. Evaluated only on toy or non-privacy datasets (MNIST, CIFAR, ...)? Then a
  marked paragraph must justify why results carry over.
- **AI-use disclosure:** CCS section or acknowledgement, S&P submission field, NDSS statement;
  USENIX '27 needs none (authors vouch for AI output), so do not flag its absence there.

## 7. Differential privacy checks (if the paper uses DP)

- **DP model:** central, local, shuffle or distributed? Who sees raw data, and does that match
  the threat model?
- **Privacy unit and neighbouring relation** (record, user, device, time window; add/remove vs
  replace)? Does it match what needs protecting (e.g. users with many records)?
- **ε and δ** for the whole release (not per step, round or query; RDP/zCDP converted to
  (ε, δ)); δ well below 1/n; large ε values explained?
- **Full accounting:** does the budget cover all uses of private data: hyperparameter tuning,
  model selection, preprocessing (clipping bounds, domains, category lists taken from the
  data), repeated releases? Tuning on private data outside the budget is Major.
- **Public data:** really public, and disjoint from the private data? People in an overlap get
  no protection; was this checked?
- **Composition and accountant** named? Subsampling amplification matches the sampler actually
  implemented (Poisson vs shuffling / fixed batches)?
- **Implementation:** floating-point attacks on naive Laplace/Gaussian sampling, secure RNG,
  timing side channels. Library or own code? Any empirical privacy audit?
- **Comparisons at equal ε** (same unit, δ and accountant); non-private baseline shown;
  utility with error bars over the noise.
- **Wording:** "privacy-preserving", "anonymous" or "synthetic so private" without a formal
  guarantee is Major. Membership-inference results complement but do not prove DP.

## Lens output

In the review's "Security & privacy lens" section: the threat-model card, then a table
`Check | Result | W#` of at most 6 rows (the most important checks); problems are W#. List
sections that do not apply in one line ("Not applicable: §3, §7").
