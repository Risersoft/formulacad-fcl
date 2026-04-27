# Contributing to FormulaCAD FCL — Language, Samples, and Library

Thanks for your interest in contributing.

This repository contains the FCL specification, sample models, and the standard component library used by FormulaCAD.

Models defined here are executed and rendered by the FormulaCAD platform. The platform itself is not part of this repository.

Before you start, it's useful to understand how this repository relates to the FormulaCAD platform:

- **This repository is the publication channel.** Changes here are reviewed and then propagated into FormulaCAD's canonical cloud database, which is what the platform runs against.
- **The cloud database is the runtime.** Users of FormulaCAD author their own components through the web editor or desktop client; those components live in their own namespaces and are not published here.
- **Your contribution affects downstream users.** A merged correction to a library component updates the canonical cloud version on deployment, and every assembly that references it picks up the change.

Because contributions affect downstream users, we review carefully and prioritize correctness.

---

## What you can contribute

**Library component corrections.**  
If a dimension in a standard section is incorrect against the published standard, or a cross-section template produces incorrect geometry, submit a PR fixing it. This is the most common and highest-value contribution.

**Specification clarifications.**  
If language in `context.md`, `rules.catalog.json`, or `functions.json` is ambiguous or incorrect, propose clearer wording.

**New sample models.**  
If a useful FCL pattern is not well represented in the `Samples/` directory, contribute a complete model that demonstrates it clearly.

**New regional standards.**  
If you want to propose adding JIS, GB, AS/NZS, BS, or another catalog, open an issue first to discuss scope and sourcing. Avoid starting large contributions without prior discussion.

**Function library specification extensions.**  
If a function would be genuinely useful and isn't documented in `functions.json`, propose an addition to the function library specification. The proposal should describe the function's name, parameters, return value, behavior, and expected edge cases.

You may also include an implementation sketch or reference algorithm where helpful, but note that `functions.json` defines the specification only. Actual function execution is implemented within the FormulaCAD platform.

---

## What you can't contribute here

**FCL language implementation changes.**  
The parser, engine, and runtime are not part of this repository. If you think there is a language design issue, open an issue to discuss.

**Platform features.**  
Requests or changes to the FormulaCAD platform (web editor, desktop client, cloud service, rendering) are not implemented through this repository.

**Private catalogs.**  
If your organization has a proprietary component library, that belongs in your own FormulaCAD namespace, not in this repository.

---

## Before you open a PR

1. **Check the issue tracker.** Someone may already be working on the same thing.  
2. **Open an issue for anything non-trivial.** Dimensional corrections are usually safe to PR directly. Larger changes should start with discussion.  
3. **Verify against the source standard.** Cite the specific clause, table, or page that supports your change.  
4. **Fork and branch.** One logical change per PR.

---

## Correcting a library component

This is the most frequent contribution type.

**1. Identify the error.**  
Open the relevant `index.json` (e.g., `Library/EN/index.json`) and compare parameters against the published standard.

**2. Verify your source.**  
Use authoritative standards (EN, IS, AISC, DIN, etc.). Commercial catalogs are acceptable secondary references where necessary.

**3. Make the correction.**  
Edit the `params` values. Keep naming and structure consistent.

**4. Check template impact.**  
Most corrections are data-only. Structural changes to FCL templates should start with an issue.

**5. Submit the PR.**  
Include:
- Source standard and reference  
- Values changed (before → after)  
- Whether the change is data-only or structural  

**Example PR description:**

> Correcting root radius of `EN.IPE240` from 12mm to 15mm per EN 10365 Table 3.2. Template unchanged; data only.

We merge quickly when the source citation is clear and the change is data-only.

---

## Proposing a new standard catalog

Open an issue first.

Include:

- **Standard and scope** (e.g., JIS G 3192 hot-rolled sections)  
- **Estimated size** (families and components)  
- **Source availability** (published vs commercial)  
- **Template reuse vs new templates**

Once agreed, you can submit structured PRs.

---

## Writing new FCL samples

The `Samples/` directory contains complete, working models.

New samples should:

- Be complete and buildable in FormulaCAD  
- Demonstrate a specific pattern clearly  
- Follow existing conventions  
- Be tested before submission  

Keep samples focused — one model should illustrate one idea.

---

## Specification changes

Changes to `context.md`, `rules.catalog.json`, or `functions.json` affect all users and AI systems grounded on this repository.

- Wording clarifications are generally safe  
- Structural changes should start with an issue  

If the specification contradicts observed behavior, file an issue with both spec text and actual behavior.

---

## Issues and feedback

GitHub issues are the primary way to ask questions, share ideas, or suggest improvements.

You can open an issue for:

- Suggestions for improvements (language, samples, library, or overall workflow)  
- Questions about FCL usage  
- Ideas for new samples, functions, or library components  
- Problems with sample models (`Samples/`)  
- Errors or inconsistencies in the library  
- Issues with the FormulaCAD sample viewer or platform behavior  

If you’re unsure, go ahead and open an issue — we’ll guide you or redirect as needed.

---

## Review and merge process

**Who reviews.**  
Risersoft maintainers review all PRs, with domain expertise as needed.

**Review criteria:**
- Correctness against source  
- Consistency with existing patterns  
- Backward compatibility  
- Clarity  

**Review time.**  
We aim to review simple data corrections within a few days. Larger contributions may take longer.

**After merge.**  
Changes are propagated into the FormulaCAD cloud library on deployment and affect downstream users.

---

## Licensing your contribution

By submitting a pull request, you agree that your contribution is licensed under the repository's MIT License.

If your contribution is derived from published standards, you are licensing your transformation of the data into FCL form — not the original standard.

---

## Code of conduct

Keep discussions constructive. Technical disagreement is fine; disrespectful behavior is not.

---

## Getting help

- Unsure if a contribution is appropriate? Open an issue  
- Questions about FCL? See https://formulacad.com/fcl-language  
- Want to discuss ideas? Open an issue with `discussion` tag  

Thanks for contributing.
```

