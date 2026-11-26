---
name: website-preview-midjourney
version: 1.1.0
description: Turn a website URL into a Midjourney prompt for an image that fits the page's subject, visual identity, and intended image slot.
---

# Website Preview — Midjourney Prompt

## Purpose

Given a URL, inspect the page and produce a ready-to-paste Midjourney prompt for a preview image that feels art-directed for that website.

The default interaction should be simple:

- **Required input:** a URL.
- **Optional input:** any specific requirements the user wants to add (for example, `--ar 4:3`, no people, more abstract, include a particular object, or leave room for a title).
- Do not ask the user to fill in a brief. Infer sensible defaults from the page and proceed.

The image should feel native to the website without recreating the webpage, its UI, or its text.

## Default behavior

Unless the user overrides something:

1. Inspect the exact URL and its rendered page.
2. Infer what the page is about and what kind of preview image would best represent it.
3. Extract the page's visual identity: dominant and supporting colors, contrast, medium, shapes, materials, lighting, texture, mood, and composition.
4. Infer the image's role and dimensions from the visible layout or image slot.
5. Create **one strong, ready-to-paste Midjourney prompt** that combines the page's subject with its visual identity.
6. Use a sensible aspect ratio based on the target slot. If no slot or use case can be inferred, default to `--ar 16:9`.
7. Keep the prompt focused and concise. Do not return multiple variants, a long design audit, or a tutorial unless the user asks.
8. Include only useful Midjourney parameters. Do not hardcode a model version, seed, Style Reference code, or personalization profile.
9. If a screenshot can be captured, consider recommending it as a Style Reference when it would materially improve visual matching. Do not make this a prerequisite or invent a public image URL.

## Website inspection

Use available browser, browsing, or page-inspection tools to inspect the exact URL.

- Inspect the rendered page, not only its title, metadata, or search snippet.
- Look at the first viewport and the relevant content/image area. Scroll further only when needed to understand the page.
- Inspect desktop and mobile layouts when responsive behavior materially affects the image.
- When available, inspect CSS/design tokens and image dimensions for actual colors, fonts, gradients, borders, and surfaces.
- Prioritize the target image slot and the page's dominant design system. Ignore unrelated third-party widgets and one-off visual elements.
- Never claim a full visual inspection if only metadata or extracted text was available.

## What to extract

Build a compact internal visual profile. Focus on traits that should visibly influence the generated image.

### Subject and intent

- What the page is about.
- The clearest visual idea that communicates its purpose.
- Whether the image is a link preview, card thumbnail, hero, article cover, project image, or background.
- What needs to remain legible at the likely display size.

### Visual identity

- Dominant background/surface colors and supporting colors.
- Accent color(s), used with appropriate restraint.
- Light/dark balance, contrast, and saturation.
- Medium: photography, 3D, illustration, vector, collage, painterly, mixed media, etc.
- Shape language: geometric, organic, rounded, angular, modular, fluid, precise, irregular.
- Materials, lighting, texture, depth, and overall mood.
- Composition: centered or asymmetric, dense or sparse, whitespace, crop, and focal-point placement.

Use exact HEX values only when verified from CSS/design tokens or reliable sampling. Otherwise describe colors naturally. HEX values are hints, not a guarantee that Midjourney will reproduce exact brand colors.

Typography is only a supporting clue. Do not ask Midjourney to reproduce the website's actual typography, logo, headline, or interface unless explicitly requested.

## Prompt construction

Write the prompt in clear, concrete English by default, regardless of the language used to invoke the skill, unless the user asks for another language.

Use this order, adapting it to the image:

1. **Subject/concept:** one clear visual idea relevant to the page.
2. **Medium and execution:** photographic, 3D, illustration, etc., with specific visual qualities.
3. **Palette:** dominant colors first, accent colors second.
4. **Materials, lighting, and texture:** the traits that make the image feel visually related to the site.
5. **Composition:** framing, viewpoint, crop, focal point, negative space, and safe areas.
6. **Parameters:** aspect ratio and any other relevant supported settings.

Translate the website's design into visual instructions. Do not write meta-instructions such as “match this website” or “use the colors from the URL”.

The image should represent the page, not merely reproduce its palette. Choose a specific subject or visual metaphor rather than a generic assortment of themed objects.

Do not generate:

- Website screenshots, browser frames, UI mockups, dashboards, cards, or navigation.
- Readable text, logos, watermarks, or typography unless requested.
- Decorative clutter that competes with nearby page content.
- A generic stock-image aesthetic when the website has a distinctive visual language.

Use a restrained `--no` list only when it meaningfully prevents likely failure modes.

## Aspect ratio and composition defaults

Infer the ratio from the actual image slot whenever possible. If the target cannot be identified, use `--ar 16:9`.

Useful starting points:

- Open Graph / link preview: `--ar 1.91:1` or `--ar 16:9`.
- Hero/banner: `--ar 16:9` or wider if the user specifies it.
- Article/editorial cover: `--ar 3:2` or `--ar 4:5`.
- Project/card thumbnail: `--ar 4:3` or `--ar 1:1`.
- Square tile: `--ar 1:1`.

When an image may be cropped by a card or social platform, keep the main subject inside a center-safe area and avoid tiny details.

User-specified requirements always override inferred defaults. Preserve other defaults unless the user explicitly changes them. For example, if the user says `--ar 4:3`, change the aspect ratio but keep the website analysis, visual matching, subject selection, and other suitable defaults.

## Style Reference guidance

A screenshot of the website can help Midjourney match its visual language.

- Recommend a clean screenshot crop showing the page's colors, surfaces, gradients, and overall atmosphere, ideally near the target image slot.
- Use it as a **Style Reference** when the goal is to borrow visual treatment.
- Use an Image Prompt only if the user also wants the screenshot to influence subject or composition; otherwise it may cause Midjourney to reproduce unwanted page layout.
- If a public screenshot URL is genuinely available and usable, a prompt may include `--sref <url>` and optionally `--sw` when supported by the current model.
- If no public URL exists, tell the user briefly that they can attach the screenshot manually as a Style Reference. Never invent a URL or claim that a local screenshot is hosted.
- Do not make screenshot attachment mandatory. The text prompt must stand on its own.

Check current Midjourney documentation when exact parameter syntax or model compatibility is uncertain. Avoid hardcoding a model version unless the user requests it.

## Output

### Default output

Return only:

1. A short one-line description of the visual direction (optional if `prompt_only` is requested).
2. One fenced text block containing the final Midjourney prompt, ready to paste.
3. A brief Style Reference note only when a screenshot would materially improve the result.

Do not include a full visual audit, variants, lengthy explanation, or generic iteration advice by default.

### If the user asks for more detail

You may additionally provide:

- A concise visual profile (palette, medium, lighting/materials, composition, mood).
- Two genuinely different prompt alternatives.
- A short explanation of the art direction.
- Specific iteration suggestions based on a generated result.
- A more detailed Style Reference workflow.

## Handling user overrides

Treat any extra instructions supplied alongside the URL as prompt constraints, not as a replacement for the skill's default workflow.

Examples:

- `URL --ar 4:3` → use a 4:3 composition.
- `URL, no people` → exclude people.
- `URL, more abstract` → use a more abstract visual concept while preserving the site's visual identity.
- `URL, leave space on the left for a title` → place the main subject to the right and preserve clean negative space on the left.
- `URL, make it more photographic` → change the medium, but retain the site's palette and visual character.

If a user requirement conflicts with the website's apparent design, follow the user's explicit requirement and preserve as much of the site's identity as possible.

Do not ask follow-up questions for optional details. Make a reasonable decision and continue. Ask one concise question only if a missing detail makes it impossible to choose between materially different outputs.

## Failure and fallback behavior

If the URL is inaccessible, blocked, or requires authentication:

1. State briefly what could and could not be inspected.
2. Use available metadata, Open Graph information, favicon/logo, accessible assets, or reliable public page information as supporting evidence.
3. Do not describe metadata-only inspection as a visual audit.
4. Produce a provisional prompt using verified information and mark the visual match as lower confidence.
5. Ask for a screenshot only if visual matching is central and the available evidence is inadequate.

If the page loads but CSS cannot be inspected, estimate colors from the rendered page and treat them as approximations. If there is no screenshot, avoid exact HEX values.

If the page contains conflicting design signals, prioritize the target image slot and dominant page style over isolated banners, logos, or embedded third-party content.

## Quality checklist

Before returning the prompt, verify:

- The URL and relevant page content were inspected to the level claimed.
- The image has a clear subject relevant to the page.
- The palette and visual treatment reflect observed page characteristics.
- The composition suits the target image slot and likely crop.
- The image is not accidentally a recreation of the website or its UI.
- User-specified constraints have been applied.
- The prompt is concise, complete, and ready to paste.
- Parameters are supported or conservatively chosen.

## Official Midjourney references

Consult these when parameter support or reference behavior needs checking:

- Prompt basics: https://docs.midjourney.com/hc/en-us/articles/32023408776205-Prompt-Basics
- Parameter list: https://docs.midjourney.com/hc/en-us/articles/32859204029709-Parameter-List
- Style Reference: https://docs.midjourney.com/hc/en-us/articles/32180011136653-Style-Reference
- Image Prompts: https://docs.midjourney.com/hc/en-us/articles/32040250122381-Image-Prompts
