---
name: website-preview-midjourney
version: 1.4.0
description: Inspect a website URL and create Midjourney-ready preview image prompts that fit its content and visual identity.
---

# Website Preview — Midjourney Prompt

## Purpose

Given a URL, inspect the page and create a Midjourney prompt for an image that feels art-directed for that website.

The normal interaction is deliberately simple:

- Required: a URL.
- Optional: extra requirements such as aspect ratio, subject exclusions, composition, medium, or mood.
- Infer sensible defaults. Do not make the user fill in a brief.

The image should feel native to the website without recreating the webpage, its UI, or its text.

## Default workflow

1. Inspect the exact URL and its rendered page using available browser/page-inspection tools.
2. Understand the page's subject and the role of the intended preview image.
3. Extract the visual identity: palette, contrast, medium, shapes, materials, lighting, texture, mood, and composition.
4. Infer the target image slot and aspect ratio from the page when possible. If unclear, default to `--ar 16:9`.
5. Create three genuinely distinct, ready-to-paste Midjourney prompts by default, each exploring a different creative direction while preserving the website's visual identity.
6. Keep analysis and the prompt clearly separated in the output format below.
7. Do not ask for optional details. User-specified requirements override inferred defaults.

## Website inspection

- Inspect the rendered page, not only its title, metadata, or search snippet.
- Examine the first viewport and relevant content/image area. Scroll further only when needed.
- Inspect desktop and mobile layouts when responsive behavior materially affects the image.
- When available, inspect CSS/design tokens and image dimensions for actual colors, fonts, gradients, borders, and surfaces.
- Prioritize the target image slot and dominant page design system. Ignore unrelated third-party widgets and isolated visual elements.
- Never claim a full visual inspection if only metadata or extracted text was available.

## What to infer

### Subject and intent

- What the page is about.
- The clearest visual idea that communicates its purpose.
- Whether the image is a link preview, card thumbnail, hero, article cover, project image, or background.
- What must remain legible at the likely display size.

### Visual identity

- Dominant background/surface colors and supporting colors.
- Accent colors, used with appropriate restraint.
- Light/dark balance, contrast, and saturation.
- Medium: photography, 3D, illustration, vector, collage, painterly, mixed media, etc.
- Shape language: geometric, organic, rounded, angular, modular, fluid, precise, irregular.
- Materials, lighting, texture, depth, and mood.
- Composition: centered or asymmetric, dense or sparse, whitespace, crop, and focal-point placement.

Use exact HEX values only when verified from CSS/design tokens or reliable sampling. Otherwise describe colors naturally. HEX values are hints, not a guarantee that Midjourney will reproduce exact brand colors.

Typography is only a supporting clue. Do not ask Midjourney to reproduce actual website typography, logos, headlines, or interface elements unless explicitly requested.

## Prompt construction

Write prompts in clear, concrete English by default, regardless of the language used to invoke the skill, unless the user asks for another language.

Build the prompt from:

1. A clear subject or visual concept relevant to the page.
2. Medium and execution (photographic, 3D, illustration, etc.).
3. Dominant and supporting palette.
4. Materials, lighting, and texture.
5. Composition, framing, viewpoint, crop, focal point, and negative space.
6. Relevant Midjourney parameters.

Translate the website's design into visual instructions. Avoid meta-instructions such as “match this website” or “use the colors from the URL”.

The image should represent the page, not merely reproduce its palette. Prefer one specific visual idea or metaphor over a generic assortment of themed objects.

Do not generate:

- Website screenshots, browser frames, UI mockups, dashboards, cards, or navigation.
- Readable text, logos, watermarks, or typography unless requested.
- Decorative clutter that competes with nearby page content.
- Generic stock imagery when the website has a distinctive visual language.

Use a restrained `--no` list only when it meaningfully prevents likely failure modes. Avoid excessively long negative lists.

## Aspect ratio and composition defaults

Infer the ratio from the actual image slot whenever possible. If the target cannot be identified, use `--ar 16:9`.

Useful starting points (write the parameter as integer `width:height` values):

- Open Graph / link preview: `--ar 191:100` (approximately 1.91:1) or `--ar 16:9`.
- Hero/banner: `--ar 16:9` or a wider integer ratio if requested.
- Article/editorial cover: `--ar 3:2` or `--ar 4:5`.
- Project/card thumbnail: `--ar 4:3` or `--ar 1:1`.
- Square tile: `--ar 1:1`.

### Strict aspect-ratio validation

Midjourney aspect-ratio parameters must use the form `--ar width:height`.

- **Use positive whole numbers on both sides of the colon. Do not use decimal values.**
- Never output malformed ratios such as `--ar 19.1:1`, `--ar 1.91:1`, `--ar 16/9`, `--ar 1,91:1`, or `--ar 1.91`.
- For a target ratio expressed as a decimal, convert it to an integer ratio before writing the prompt. For example, 1.91:1 becomes `--ar 191:100`; 1.5:1 becomes `--ar 3:2`; 1.333:1 becomes `--ar 4:3`.
- When the source image slot has known pixel dimensions, use those dimensions as `width:height` and reduce by their greatest common divisor where practical. For example, 1200×630 becomes `--ar 40:21`.
- If an exact conversion would create unnecessarily large numbers, use a close, simple integer ratio such as `--ar 2:1` or `--ar 16:9`.
- If the intended ratio cannot be established confidently, use the safe default `--ar 16:9`.
- Validate every prompt independently. Because the skill returns three prompts by default, all three must contain a valid, complete `--ar width:height` parameter.
- Preserve a user-specified ratio, but normalize decimal input into integer width:height form. If the user explicitly provides `--ar 4:3`, keep it unchanged.

When an image may be cropped by a card or social platform, keep the main subject inside a center-safe area and avoid tiny details.

User-specified requirements always override inferred defaults. Preserve other defaults unless explicitly changed. For example, if the user says `--ar 4:3`, change the ratio but retain the website analysis, visual matching, and subject selection.

## Style Reference guidance

A screenshot of the website can help Midjourney match its visual language.

- Recommend a clean screenshot crop showing the page's colors, surfaces, gradients, and overall atmosphere, ideally near the target image slot.
- Use it as a Style Reference when the goal is to borrow visual treatment.
- Use an Image Prompt only if the screenshot should also influence subject or composition; otherwise it may cause Midjourney to reproduce unwanted page layout.
- If a public screenshot URL is genuinely available and usable, include `--sref <url>` and optionally `--sw` only when supported by the current model.
- If no public URL exists, say briefly that the user can attach the screenshot manually as a Style Reference. Never invent a URL or claim a local screenshot is hosted.
- The text prompt must stand on its own; screenshot attachment is optional.

Check current Midjourney documentation when parameter syntax or model compatibility is uncertain. Do not hardcode a model version, seed, Style Reference code, or personalization profile.

## Mandatory response format

Always use the following hierarchy. The user must be able to identify and copy each Midjourney prompt immediately. Preserve the detailed visual analysis; the goal is to make the copy-ready prompts easier to find, not to remove creative exploration or process information.

### 1. Visual analysis

Give a concise but useful summary of:

- **Page / subject:** what the page is about and what the image should communicate.
- **Visual identity:** palette, medium, shapes/materials, lighting/texture, and mood.
- **Composition:** intended framing and aspect ratio, including the evidence used to infer it.

This section may contain process details and caveats, but keep it readable. Do not let it bury the final prompt.

### 2. Midjourney prompts

This must be the most visually prominent and easiest-to-copy section of the response.

- Use the exact heading `## Midjourney prompts`.
- Produce **three prompts by default**. The user should not need to request variants.
- The three prompts must explore genuinely different creative directions, not merely rephrase the same concept or swap a few adjectives.
- Use these creative roles as a starting point:
  1. **Closest match:** the strongest and most faithful translation of the website's subject and visual identity.
  2. **Creative interpretation:** a more unexpected, expressive, or metaphorical concept that still feels native to the website.
  3. **Alternative direction:** a meaningfully different composition, visual metaphor, or medium that remains appropriate for the page.
- Adapt those roles when another set of three directions would better suit the page. Do not force an unsuitable direction merely to satisfy a label.
- Give each prompt its own clearly numbered subheading, such as `### 1. Closest match`, `### 2. Creative interpretation`, and `### 3. Alternative direction`.
- Put each complete prompt in its own fenced code block with a `text` language tag.
- Each code block must contain only that prompt: no introduction, labels, quotation marks, commentary, bullet points, or explanations.
- Each prompt must be independently usable: include its subject, visual treatment, composition, and all required Midjourney parameters. The user must be able to copy any single code block and paste it directly into Midjourney without combining it with another prompt or with the analysis.
- Apply all user-specified requirements consistently to all three prompts unless the user explicitly asks for a particular requirement to apply only to one direction.
- Include the inferred or requested aspect ratio in every prompt. Do not omit parameters from later variants just because they appeared in the first.
- Do not put analysis, caveats, optional edits, or notes inside any prompt code block.
- If the user explicitly requests a different number of prompts, follow that request. If they explicitly request only one, produce one. If they request more than three, provide the requested number while keeping each direction meaningfully distinct.
- Do not use a WritingBlock or other UI format instead of code blocks; code blocks are the copy-ready prompts.

### 3. Style reference

If useful, provide the screenshot path or URL exactly as observed and explain in one or two sentences whether it should be attached as a Style Reference or Image Prompt. If no useful reference exists, omit this section.

### 4. Notes

Put caveats, assumptions, optional prompt edits, and uncertainties here. Clearly distinguish optional suggestions from the final prompt. Do not ask the user to manually edit the prompt unless explaining a genuinely optional adjustment.

The default response should include useful analysis, three distinct copy-ready prompts, relevant Style Reference recommendations, and material notes. Avoid lengthy tutorials or generic advice. Do not shorten the creative exploration simply to make the output more compact.

## Handling user overrides

Treat instructions supplied alongside the URL as constraints layered on top of the defaults.

Examples:

- `URL --ar 4:3` → use a 4:3 composition.
- `URL, no people` → exclude people.
- `URL, more abstract` → use a more abstract concept while preserving the site's visual identity.
- `URL, leave space on the left for a title` → place the subject to the right and preserve clean negative space on the left.
- `URL, make it more photographic` → change the medium, retaining the site's palette and visual character.

If a requirement conflicts with the website's apparent design, follow the explicit user requirement and preserve as much of the site's identity as possible.

## Failure and fallback behavior

If the URL is inaccessible, blocked, or requires authentication:

1. State briefly what could and could not be inspected.
2. Use available metadata, Open Graph information, favicon/logo, accessible assets, or reliable public page information as supporting evidence.
3. Do not describe metadata-only inspection as a visual audit.
4. Produce a provisional prompt using verified information and mark visual matching as lower confidence.
5. Ask for a screenshot only if visual matching is central and available evidence is inadequate.

If CSS cannot be inspected, estimate colors from the rendered page and describe them as approximations. If no screenshot is available, avoid exact HEX values.

If the page contains conflicting design signals, prioritize the target image slot and dominant page style over isolated banners, logos, or embedded third-party content.

## Quality checklist

Before returning:

- The URL and relevant page content were inspected to the level claimed.
- The image has a clear subject relevant to the page.
- Palette and visual treatment reflect observed page characteristics.
- Composition suits the target image slot and likely crop.
- The image does not recreate the website or its UI.
- User-specified constraints are applied.
- Three genuinely distinct prompt directions are provided by default, unless the user requests a different number.
- Every prompt is complete, self-contained, and contained in its own easy-to-copy code block.
- Each prompt includes a valid integer `--ar width:height` parameter; neither side contains decimals.
- Decimal aspect ratios have been converted to an integer ratio (for example, 1.91:1 → 191:100).
- Each prompt's aspect-ratio syntax has been checked independently.
- Analysis and caveats are outside all prompt code blocks.
- Parameters are supported or conservatively chosen.

## Official Midjourney references

Consult these when parameter support or reference behavior needs checking:

- Prompt basics: https://docs.midjourney.com/hc/en-us/articles/32023408776205-Prompt-Basics
- Parameter list: https://docs.midjourney.com/hc/en-us/articles/32859204029709-Parameter-List
- Style Reference: https://docs.midjourney.com/hc/en-us/articles/32180011136653-Style-Reference
- Image Prompts: https://docs.midjourney.com/hc/en-us/articles/32040250122381-Image-Prompts
