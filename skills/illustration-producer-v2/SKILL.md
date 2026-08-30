---
name: illustration-producer-v2
description: Generate an approved storyboard's complete illustration batch in one consistent visual context and report successful paths and failed items.
---

# Illustration Producer V2

Use one fresh Agent to generate the complete approved illustration list while
holding character, palette, texture, composition language, and aspect ratio
consistent across the batch.

## Inputs

Require:

- the approved storyboard illustration rows and stable image ids;
- the approved style note;
- representative visual references;
- user-provided characters, products, logos, or other required assets;
- target aspect ratio and project output directory.

The storyboard decides which images exist. The approved style and references
decide how they look.

## One-Agent batch behavior

Keep the whole illustration batch in the same Agent context. Reuse the approved:

- palette and contrast;
- character and object references;
- line, texture, lighting, and depth treatment;
- composition grid and safe areas;
- naming and output conventions.

Select an available image-generation tool suited to the approved direction.
Generate in storyboard order when earlier outputs help keep later characters or
environments consistent. Do not create a separate Agent per image.

Name files by stable ids, for example `images/shot-003-main.png`, and keep all
paths project-relative.

## Failures

Retry a transient failed image operation once. After the second failure, retain
the completed batch items and return the failed image id with an actionable
reason. One failed image does not discard successful paths.

## Report

Return:

```json
{
  "successes": [
    {
      "id": "shot-003-main",
      "path": "images/shot-003-main.png",
      "note": "approved style applied"
    }
  ],
  "failures": [
    {
      "id": "shot-006-main",
      "reason": "actionable cause"
    }
  ],
  "paths": ["images/shot-003-main.png"]
}
```

Include only produced images in `paths`. The main task will use failures to
recommend retry, a tool change, a supplied resource, or a feasible skip.

## Targeted redo

For local redo, use a new targeted Agent with only:

- the named image ids and storyboard rows;
- the approved style note;
- the minimum successful batch references needed for consistency;
- the user's requested change.

Generate only those replacements and preserve previous paths until the user
accepts the new preview.
