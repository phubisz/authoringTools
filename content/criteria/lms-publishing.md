---
name: LMS Publishing Options
type: criteria
tags: [criteria, lms, scorm, xapi, publishing, standards]
---

# LMS Publishing Options

## Definition

LMS Publishing Options refers to the **breadth, reliability, and flexibility of standards-compliant output formats** a tool can generate for delivery through Learning Management Systems. Key standards include SCORM 1.2, SCORM 2004, xAPI (Tin Can), AICC, cmi5, and HTML5. Also relevant: direct hosting options, live-update capabilities, and multi-LMS compatibility.

**Why it matters**: Publishing incompatibility is a critical risk. A course that doesn't track correctly in an LMS breaks learner completion data, assessment scores, and compliance reporting. Teams integrating with legacy LMS platforms must verify format support.

---

## Rankings

| Tool | Score | Notes |
|------|-------|-------|
| [[tools/articulate-360\|Articulate 360]] | ⭐⭐⭐⭐⭐ (5) | SCORM 1.2/2004, xAPI, AICC, cmi5, HTML5; broadest compatibility |
| [[tools/ispring-suite\|iSpring Suite]] | ⭐⭐⭐⭐⭐ (5) | SCORM 1.2/2004, xAPI, AICC; known for reliable PPT-to-SCORM fidelity |
| [[tools/lectora\|Lectora]] | ⭐⭐⭐⭐⭐ (5) | SCORM 1.2/2004, xAPI, AICC, PENS, HTML5; legacy format depth |
| [[tools/dominknow\|dominKnow]] | ⭐⭐⭐⭐⭐ (5) | SCORM 1.2/2004, xAPI, AICC, HTML5, PDF; single-source multi-format |
| [[tools/easygenerator\|Easygenerator]] | ⭐⭐⭐⭐ (4) | SCORM + xAPI; dynamic SCORM; built-in LMS-lite hosting |
| [[tools/mindsmith\|Mindsmith]] | ⭐⭐⭐⭐ (4) | Dynamic remote-hosted SCORM; live updates post-publish; xAPI |
| [[tools/elucidat\|Elucidat]] | ⭐⭐⭐⭐ (4) | SCORM 1.2/2004, xAPI, HTML5; strong LMS integrations |
| [[tools/gomo-learning\|Gomo Learning]] | ⭐⭐⭐⭐ (4) | SCORM 1.2/2004, xAPI; built-in xAPI analytics dashboard |
| [[tools/parta-io\|Parta.io]] | ⭐⭐⭐ (3) | SCORM + xAPI; less documented for AICC/cmi5; not a primary focus |

---

## Key Insight

> **Mindsmith's dynamic remote-hosted SCORM is a unique innovation**: the SCORM package is a wrapper pointing to a live URL — meaning course updates are instantly available to all learners without re-uploading to the LMS. This is particularly valuable for rapidly-updated content (product training, policy changes). Traditional tools require a re-upload cycle for every content change.

---

## Standards Reference

| Standard | Description | Relevant Tools |
|----------|-------------|----------------|
| SCORM 1.2 | Oldest, most widely supported | All tools |
| SCORM 2004 | Improved sequencing; less universal | Articulate, iSpring, Lectora, dominKnow |
| xAPI (Tin Can) | Modern; tracks any learning activity | All cloud tools |
| AICC | Legacy aviation standard; still required by some orgs | Articulate, iSpring, Lectora, dominKnow |
| cmi5 | Modern LMS integration standard | Articulate 360 |
| HTML5 | Direct browser delivery; no plugin | All tools |

---

## Tool Commentary

### Articulate 360
The broadest publishing support in this comparison. cmi5 (modern LMS standard) support is exclusive in this group. Review 360 enables stakeholder preview without LMS deployment. The de facto standard for enterprise LMS publishing.

### iSpring Suite
Known for producing exceptionally clean, reliable SCORM output from PowerPoint source files. The PPT-to-SCORM pipeline has a track record of high LMS compatibility. iSpring Space provides cloud hosting as an alternative to LMS deployment.

### Lectora
Includes PENS (Package Exchange Notification Services) — a less common but occasionally required standard in government and regulated contexts. Strong accessibility in SCORM-tracked assessments.

### dominKnow
Single-source publishing: one course can be simultaneously published as SCORM, xAPI, and HTML5 for different delivery contexts. Reduces maintenance overhead for multi-format deployments.

### Mindsmith
Remote-hosted SCORM differentiates Mindsmith: the LMS receives a package that always points to the current version of the course. No re-publish/re-upload cycle when content changes. Ideal for organizations with frequently-updated training content.

### Gomo Learning
Built-in xAPI analytics dashboard is a notable differentiator — organizations can track learner behavior without a separate Learning Record Store (LRS), reducing infrastructure requirements.
