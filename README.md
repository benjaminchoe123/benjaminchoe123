# Benjamin Choe

Information Systems student at UMBC. I build detection and threat-intelligence tooling, and I'm
studying for CompTIA Security+.

Everything below is one project split in two halves, and the interesting part is the gap
between them:

> **What is actually being exploited?** — and — **would I catch it?**

---

## The measurement I'd want to be asked about

My pipeline records every MITRE ATT&CK technique it observes in live threat feeds. `ruleproof`
records every technique I can *prove* a detection fires on. Neither number means much alone.
Together, as of 2026-08-26:

| | |
|---|---|
| ATT&CK techniques observed in live data | **43** |
| …with a demonstrated detection | **15** |

Two things that number surfaced, both of which are criticisms of my own work:

- **The single most-observed technique has no detection at all.** T1190, *Exploit
  Public-Facing Application*, appears in 25 threat notes — over six times the next-largest
  undetected technique, and about twice the next-most-observed one. That's partly honest (a catalogue of exploited CVEs *is* mostly T1190, and it can't
  be caught by one generic rule) but a coverage percentage that quietly omits it is flattering
  itself.
- **Two of my rules detect things my pipeline has never seen.** I chose them from general
  detection knowledge instead of from the evidence I was already collecting. The two projects
  weren't talking to each other until I measured this.

Then I acted on it. `gap` named T1571 as the most-observed undetected technique, so I wrote that
rule next; then T1219, T1105 and T1189 — each one the next the tool ranked. Coverage moved
**26% → 35%**. Measure, rank, write the rule the measurement asked for, re-measure — one command
at each step, and CI fails the build if that number regresses, so it stays true without me.

**T1190 stays undetected on purpose**, and I would rather explain that than paper over it.
Exploiting a public-facing application looks different for every product, so a single generic
rule would have to pattern-match invented exploit strings — a rule that never fires, hiding a
gap. It is where generic detection stops and product-specific detection has to start. Leaving it
visible in the number is the honest choice.

Both numbers come from `ruleproof gap`, so you can reproduce them rather than take my word:

```console
$ ruleproof gap rules ../threat-intel-pipeline/vault/threats
Observed techniques      : 43
Demonstrated by rules    : 10
Observed AND detected    : 15  (35%)
Observed, NOT detected   : 28
```

---

## Repositories

### [threat-intel-pipeline](https://github.com/benjaminchoe123/threat-intel-pipeline) — what is being exploited
Ingests five public threat feeds daily (CISA KEV, abuse.ch ThreatFox/URLhaus/MalwareBazaar,
malware-traffic-analysis.net), deduplicates against SQLite state, enriches every indicator
through VirusTotal and AbuseIPDB, and builds a linked knowledge graph of threats, malware
families and ATT&CK techniques. Drafts a weekly analyst report that **cannot auto-publish**
unless every CVE and ATT&CK ID it cites traces back to that week's evidence.

*Built for auditability rather than automation:* every AI-generated claim is logged against its
source and schema-invalid output is quarantined instead of published. **413 tests · CI · CodeQL
· secret scanning.**

It also broke on me. Enrichment failed silently for two weeks while still reporting success —
[the postmortem is in the repo](https://github.com/benjaminchoe123/threat-intel-pipeline/blob/main/docs/POSTMORTEM-2026-08-enrichment-outage.md),
along with the health check and standalone watchdog I shipped so it's detectable next time.

### [ruleproof](https://github.com/benjaminchoe123/ruleproof) — would you catch it
Unit tests for Sigma detection rules. Each rule has to prove it fires on the attack it was
written for *and* stays silent on benign lookalikes — reported separately, because a rule that
misses an attack and a rule that fires on ordinary activity fail in opposite directions, and the
second is how a detection gets muted in production.

Built on one idea: **untested is a result, not an absence.** A rule shipped with no tests fails
the build, and ATT&CK coverage counts only techniques with a *passing test*. **103 tests · 10 rules · 93 cases · CI ·
mutation-checked** by deliberately loosening each rule to confirm the tests catch it.

Using it on itself found two things worth more than the features. A test file whose rule had been
deleted reported green and exit 0 — the project's own thesis inverted, now caught. And a mutation
*survived*: loosening a filename match from `endswith` to `contains` failed no test, because the
near-miss negative guarding it never contained the string at all. The suite was green, the rule
was correct, and one of its constraints was protected by a test that asserted nothing.

The rule I am most willing to defend is the one that made me re-examine a decision I had already
published. T1189 (drive-by compromise) is delivery-side and lure-dependent in exactly the way
T1190 is product-dependent — so if "web technique" were why I left T1190 alone, T1189 should be
exempt too. It is not: it converges on a fixed observable regardless of the lure. The question
turned out to be **whether a technique ends somewhere that does not depend on the product being
attacked**, and having a rule on each side of that line is what keeps the T1190 gap a judgement
rather than an excuse.

### [dev-crew](https://github.com/benjaminchoe123/dev-crew) — a systems-design experiment
Four AI agents — architect, coder, tester, manager — that build software while communicating
only through an append-only SQLite message bus, which makes any run replayable from the log.
Separation of duties is enforced by **which tools each role is given**, not by asking the agents
nicely: the tester can author tests but cannot edit implementation code, and the manager has no
write access at all. **50 tests · CI on Windows and Linux.**

---

## How I work

- **Test-first.** Every repo above was written that way, and the test counts are real — run them.
- **I mutation-check my own tests.** A passing suite proves nothing until you've broken the code
  on purpose and watched the right test fail.
- **I publish what went wrong.** The outage postmortem is in the repo, not deleted, and the
  coverage number above is deliberately unflattering.

📍 Baltimore, MD · 📧 bchoe2@umbc.edu · [LinkedIn](https://linkedin.com/in/benjamin-choe-178294387)
