---
name: software-definition-description-optimizer
description: Optimize Software Definition Description (SDD) markdown for clarity, consistency, and structure, starting with image-link validation and filename normalization.
---

# Software Definition Description Optimizer Skill

## Overview

This skill improves SDD markdown quality by tightening language, normalizing structure, and preserving technical intent.

Before any wording or structure edits, always validate image references and file names.

## Required First Step: Image Link Validation

Run this check first on the target markdown file:

1. Find all markdown image links, such as `![alt](./media/Some Image.png)`.
2. Confirm each referenced file exists, usually under `./media` relative to the markdown file.
3. Detect image file names containing spaces.
4. Rename image files to space-free names using kebab-case.
5. Update all matching markdown references to the new file names.

### Filename Rule

- Do not allow spaces in image file names.
- Use lowercase kebab-case for renamed images.
- Keep file extensions unchanged.

Examples:

- `System Diagram V2.png` -> `system-diagram-v2.png`
- `Login Flow Final.jpg` -> `login-flow-final.jpg`

## Optimization Scope

After image links are correct, optimize the SDD content:

1. Improve grammar and spelling without changing technical meaning.
2. Clarify ambiguous statements and remove redundancy.
3. Normalize headings and section ordering.
4. Keep terminology consistent across the document.
5. Preserve requirements, constraints, and acceptance intent.

## Preferred SDD Structure

Use or align to this structure when practical:

1. Purpose
2. Scope
3. Definitions and Acronyms
4. System Context
5. Functional Description
6. Non-Functional Requirements
7. Interfaces and Dependencies
8. Assumptions and Constraints
9. Traceability Notes

## Output Expectations

When this skill is used, return:

1. A short summary of image fixes (renamed files and updated links).
2. A short summary of wording and structure improvements.
3. The updated markdown content, preserving original intent.

## Guardrails

- Do not invent requirements.
- Do not remove technical details unless they are duplicates.
- Do not change numeric values, IDs, or thresholds without explicit instruction.
- Keep edits minimal, precise, and review-friendly.
