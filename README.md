# deepgrid-sku-compendium

Agent skill generated from *Deepgrid SKU Architecture Compendium v3* by Deepgrid Semi with [book-to-skill](https://github.com/virgiliojr94/book-to-skill).

It is a sheet-by-sheet knowledge base covering nine SKU architecture sheets, the D100 drone SoC, the DG SDV platform, the SiP composition and the node roadmap. For each sheet it gives the block detail, the target values, how much to trust each figure, arithmetic checks against the sheet's own numbers, and the contradictions to fix before a diligence read.

**Confidential source.** The content is synthesized summaries and analysis, not the original document text or figures, but it describes internal company architecture. Keep this repository private unless the owner has approved publication.

## Install

```bash
npx skills add https://github.com/shekerkamma/deepgrid-sku-compendium --skill deepgrid-sku-compendium
```

## Files

| File | Contents |
|---|---|
| `SKILL.md` | Contradiction list, reading rules, chapter and topic indexes |
| `chapters/ch01`–`ch14` | One chapter per sheet |
| `glossary.md` | Block and standard names used on the sheets |
| `patterns.md` | Reusable architecture and diligence patterns |
| `cheatsheet.md` | Per-SKU targets, evidence tiers, arithmetic checks, red flags |

Sibling skill: `deepgrid-sku-portfolio` (the business plan behind these sheets).
