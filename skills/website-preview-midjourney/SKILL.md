---
name: website-preview-midjourney
version: 1.0.0
description: Inspect a URL or website and create Midjourney prompts for preview, hero, card, or editorial images that visually belong to the page. Extract the page's visual identity, color palette, composition, and image language before writing prompts.
---

# Website Preview Image Director (Midjourney)

## Purpose

Turn a website URL into one or more strong, ready-to-paste Midjourney prompts for an image that feels art-directed for that specific page.

The goal is **visual integration, not a screenshot recreation**. The generated image should feel as though it was commissioned for the page: its colors, contrast, mood, materials, lighting, texture, and compositional rhythm should belong to the site's design system, while the image itself contributes a clear subject or idea.

Use this skill when a user provides a URL and asks for a Midjourney prompt, website preview image, link thumbnail, hero image, article cover, project thumbnail, or other image that should blend with a page.

## Core rules

1. **Inspect before prompting.** Do not infer the visual style from the domain name, logo, or page title alone when the page can be inspected.
2. **Match the page, not a generic aesthetic.** Translate observed design decisions into concrete visual direction: palette, contrast, light, medium, texture, geometry, density, and mood.
3. **Generate an image, not a webpage.** Unless explicitly requested, never prompt for browser chrome, UI cards, navigation, website screenshots, dashboards, interface mockups, or readable text.
4. **Keep the subject meaningful.** The image should communicate the page's topic or purpose, not merely reproduce its colors.
5. **Be precise about evidence.** Separate observed facts from interpretation. Never invent hex values, fonts, page content, or design traits.
6. **Preserve visual hierarchy.** The image must not compete with the page's headline, CTA, or surrounding content. Use negative space and controlled detail where appropriate.
7. **Avoid style-name shortcuts.** Describe visual properties directly instead of relying on vague labels, trend names, or named living artists.
8. **Make the result usable.** Return a clean primary prompt, useful variants, an aspect-ratio recommendation, and an optional reference-image workflow.

## Input contract

The minimum input is a URL. Interpret any additional user instructions as constraints.

Optional inputs:

- `image_role`: `open_graph`, `link_card`, `hero`, `article_cover`, `project_thumbnail`, `inline_illustration`, `background`, or another stated use.
- `subject`: a specific subject, concept, product, person, place, or scene that must appear.
- `aspect_ratio` or target dimensions.
- `style_intensity`: subtle / balanced / expressive.
- `must_include` and `must_avoid` constraints.
- `prompt_only`: return only the final prompt, with no analysis.

If the role is not specified, infer it from the page and its visible image slot. State the inference briefly. Do not block on optional information; make a sensible assumption and continue.

## Workflow

### 1. Inspect the exact URL

Use the available browser, browsing, or page-inspection tools.

- Open the exact URL, including its route and query parameters where relevant.
- Inspect the rendered page, not just search snippets or metadata.
- Capture the first viewport at desktop width (roughly 1440 px) and, when responsive image treatment could change the brief, a mobile viewport (roughly 390 px).
- Scroll far enough to understand the page's main content, visual sections, and any existing image treatment. Do not inspect unrelated routes unless they clarify the same design system.
- If available, inspect DOM/CSS or computed styles for actual color values, font families, gradients, borders, radii, shadows, and image dimensions.
- Check the intended image location. Identify whether the image sits on a plain surface, inside a colored card, over a gradient, within a collage, or alongside text.
- If the site has a consent overlay, dismiss it only when safe and ordinary. Do not sign in, submit forms, or perform unrelated actions.

Do not claim to have inspected a rendered page if only metadata or a text extraction was available.

### 2. Determine the image's job

Identify:

- What the page is about, in one sentence.
- What the image needs to communicate at a glance.
- The likely viewer and intended emotional response.
- The image role and display context.
- Whether the image is a focal point, supporting visual, thumbnail, or decorative background.
- The subject or visual metaphor that best expresses the page without duplicating its headline.

Prefer a specific, legible visual concept over a vague collection of thematic objects. For abstract or technical subjects, use one strong metaphor or a coherent material/form study rather than a literal pile of symbols.

If the page already has a clear subject image, identify what is missing from the visual story before proposing another one.

### 3. Extract the page's visual DNA

Create a compact design profile with the following dimensions.

**Color system**

- Primary background/surface color.
- Main text or darkest anchor color.
- Primary brand/accent color.
- Secondary/supporting colors.
- Gradient direction and stops, if present.
- Whether the palette is warm/cool, saturated/desaturated, high/low contrast, light/dark, monochrome/multicolor.
- Approximate color hierarchy: dominant base, supporting tones, and restrained accents.

Use exact HEX values only when they are available from CSS, design tokens, or reliable sampling of the rendered page. Label sampled or inferred values accordingly. If exact values cannot be established, use descriptive color names and say so. Do not pretend Midjourney will reproduce HEX values perfectly; use them as supporting anchors alongside natural-language color descriptions.

**Visual language**

- Photographic, 3D, vector, painterly, collage, diagrammatic, handcrafted, or mixed-media.
- Shape language: geometric, rounded, organic, angular, modular, fluid, precise, irregular, etc.
- Materials and surface qualities: glass, brushed metal, matte plastic, paper, grain, fabric, chrome, stone, soft gradients, etc.
- Lighting: soft diffused, hard directional, ambient, studio, dramatic, flat, backlit, luminous, etc.
- Texture and depth: clean/flat, subtle grain, tactile, layered, glossy, atmospheric, dimensional, etc.
- Density: sparse, balanced, intricate, maximalist.

**Composition and layout**

- Typical alignment: centered, editorial, asymmetric, grid-based, left/right weighted.
- Amount and location of whitespace.
- Whether the page uses large isolated objects, layered clusters, full-bleed imagery, cropped close-ups, or open landscapes.
- Typical image-to-background contrast and edge treatment.
- Any important safe area needed for overlaid text or nearby UI.

**Typography and brand cues**

Typography is a supporting clue, not something to reproduce in the image. Note broad qualities (for example, editorial serif, neutral grotesk, monospaced technical, rounded geometric) only if observed. Never ask Midjourney to render the site's actual font or copy unless the user explicitly needs text in the image.

### 4. Build an image art direction brief

Before writing the prompt, write a short internal brief with:

- **Concept:** the single visual idea.
- **Subject:** the main object, scene, or metaphor.
- **Style:** medium and visual language.
- **Palette:** dominant colors and accent placement, with observed HEX anchors when useful.
- **Lighting/materials:** the qualities that make it feel native to the page.
- **Composition:** camera/view, crop, subject placement, negative space, and depth.
- **Integration:** how it will sit beside or behind the page's existing UI and typography.
- **Avoid:** the most likely failure modes for this page.

The image should share enough visual DNA to feel integrated, but it should not be a literal copy of the page's design. Do not force every extracted design trait into one image; select the few that matter most.

### 5. Choose a composition for the actual image slot

Use the detected slot dimensions when available. Otherwise use these defaults as starting points, not rigid rules:

| Use                              |        Starting aspect ratio | Composition guidance                                                               |
| -------------------------------- | ---------------------------: | ---------------------------------------------------------------------------------- |
| Hero / banner                    | `16:9` or wider if supported | Strong, simple focal point; protect headline/CTA area; avoid busy edges            |
| Open Graph / link preview        |           `1.91:1` or `16:9` | Readable at small size; one clear idea; keep key subject near the center-safe area |
| Article / editorial cover        |               `3:2` or `4:5` | More narrative detail; clear focal subject; editorial crop                         |
| Project / portfolio card         |               `4:3` or `1:1` | Distinct silhouette and simple composition that survives a small crop              |
| Square visual / avatar-like tile |                        `1:1` | Centered or tightly framed subject                                                 |
| Decorative background            |       Match actual container | Low-detail, low-contrast region behind text; no competing focal point              |

For Open Graph previews, remember that the image may be cropped by social platforms. Keep the core subject away from extreme edges and avoid tiny details that disappear at thumbnail scale.

If the user provides exact dimensions, prioritize those dimensions over defaults.

### 6. Write the Midjourney prompt

Write prompts in clear, concrete English by default. Use the user's preferred language if they ask. Favor compact, information-dense descriptions over long instruction paragraphs.

Build the prompt in this order:

1. **Subject and concept** — what the image actually depicts.
2. **Medium and style** — photographic/3D/illustration/etc. plus distinctive execution.
3. **Environment and material** — only what supports the idea.
4. **Palette** — dominant colors first, accent colors second; include HEX anchors only as secondary hints.
5. **Lighting and texture** — specific qualities rather than generic words such as “beautiful” or “high quality”.
6. **Composition** — framing, viewpoint, placement, negative space, crop, and depth.
7. **Integration cues** — how it relates to the page's visual language, such as restrained saturation, soft-edged geometry, fine grain, or crisp high-contrast forms.
8. **Parameters** — append supported parameters at the end.

The prompt should describe the desired image directly. Avoid meta-instructions such as “make this match the website”, “use the colors from the URL”, or “create a preview for this webpage”. Translate those requests into actual visual properties.

Use positive descriptions first. When appropriate, add a restrained `--no` list for common unwanted output such as text, watermark, logo, UI, browser frame, collage, or border. Avoid huge negative lists that can distract from the subject.

### 7. Select parameters conservatively

Always check the current Midjourney parameter documentation if browsing is available and the exact syntax or model compatibility is uncertain. Midjourney parameters can change by model version.

- Use `--ar` to match the image slot.
- Use `--stylize` / `--s` only when it helps. For brand-faithful results, keep stylization modest; raise it only when the page's art direction is expressive.
- Consider `--raw` when closer adherence to the requested visual direction is important and the selected model supports it.
- Use `--no` sparingly for unwanted text, watermarks, logos, browser frames, or UI elements.
- Do not hardcode a model version unless the user requests one or confirms the model they use. Otherwise rely on the user's Midjourney default and mention that the prompt is model-version agnostic.
- Do not invent Style Reference codes, image URLs, seeds, or personalization profile IDs.

### 8. Add a Style Reference workflow when useful

If the agent can capture a screenshot or the user can provide one, recommend using a clean screenshot of the relevant page as a **Style Reference**, not as the subject of the image.

- Prefer a crop that shows the site's colors, surfaces, spacing, gradients, and visual atmosphere without letting large amounts of text or UI dominate the reference.
- A full-page screenshot can be useful when the visual language is consistent, but a crop of the target section is usually more focused.
- In Midjourney's web interface, attach the screenshot using the Style Reference control.
- In a prompt workflow that supports URL-based Style References, use `--sref <publicly-accessible-image-url>` and optionally adjust `--sw` if supported by the chosen model.
- Do not claim a local screenshot is publicly accessible. Do not fabricate a hosted URL. If the image has not been uploaded/hosted, explain that the user must attach it manually.
- Use a Style Reference for visual treatment. Use an Image Prompt only when the user also wants the reference to influence composition/content; it can pull in unwanted layout elements from a screenshot.

A Style Reference can help Midjourney carry over colors, medium, texture, and lighting, but it is not a pixel-perfect brand-token lock. Keep the textual prompt's palette and composition instructions.

## Output format

Unless `prompt_only` is true, return the following sections in this order.

### 1. Page read

- URL inspected and whether the rendered page was accessible.
- Page purpose and inferred image role.
- One-sentence concept for the proposed image.

### 2. Visual DNA

A concise list or table containing:

- Palette: colors and HEX values where verified; mark approximations.
- Visual medium and shape language.
- Lighting, materials, and texture.
- Composition and whitespace.
- Overall mood.
- Confidence: high / medium / low, with a short reason.

### 3. Primary Midjourney prompt

Provide one ready-to-paste prompt in a fenced text block. Keep the parameters at the end. Do not place explanatory notes inside the prompt.

### 4. Variants

Provide two optional directions:

- **Closer match:** more restrained, closer to the site's existing design language.
- **More expressive:** a stronger creative interpretation that still respects the page palette and visual identity.

Each variant should change the art direction or composition meaningfully, not merely swap adjectives.

### 5. Reference setup

State whether attaching a screenshot as a Style Reference is recommended. If yes, say exactly which part of the page to capture and how to attach it. Only include a `--sref` example if a real, usable public image URL exists; otherwise explain the manual upload step.

### 6. Quick iteration notes

Give up to three targeted adjustments the user can make after seeing the first generation, such as:

- If the result is too colorful, reduce saturation and make the accent color more selective.
- If it feels detached from the page, strengthen the background/surface color and material cues.
- If it is too busy, simplify the subject count and increase negative space.

Do not provide generic advice that does not relate to the page's actual design.

## Failure and fallback behavior

### Page inaccessible

If the URL is blocked, requires authentication, fails to load, or cannot be rendered:

1. Say exactly what could and could not be inspected.
2. Try page metadata, Open Graph tags, favicon/logo, accessible CSS/assets, or a reliable public copy if available.
3. Do not represent metadata-only findings as a full visual audit.
4. Produce a provisional prompt using only verified information and label the visual direction as low confidence.
5. Ask for a screenshot only when visual fidelity is central and the current evidence is insufficient. Do not stop the task if a useful provisional prompt can still be produced.

### Incomplete visual evidence

When CSS tokens cannot be inspected, estimate colors from the rendered screenshot and label them as sampled estimates. If there is no screenshot, avoid exact HEX values entirely.

### Ambiguous purpose

Infer the most likely image role from the page layout. State the assumption and proceed. Ask one focused question only when two materially different outputs are equally plausible and choosing incorrectly would waste the generation.

### Conflicting design signals

Prioritize the target image slot and the page's dominant visual system over isolated elements such as a colorful logo, one-off banner, or third-party embedded widget. Explain any meaningful conflict briefly.

## Quality checklist

Before returning the prompt, verify:

- [ ] The page was inspected to the level claimed.
- [ ] The proposed subject clearly relates to the page.
- [ ] The image shares the site's actual palette and visual language.
- [ ] Exact colors are evidence-based or explicitly marked as estimates.
- [ ] The composition fits the target slot and survives likely cropping.
- [ ] The image leaves appropriate space for nearby text/UI.
- [ ] The prompt asks for an image, not a webpage or interface screenshot.
- [ ] Unwanted text, logos, watermarks, and UI are excluded when appropriate.
- [ ] Parameters are valid for the selected Midjourney model, or kept conservative/model-agnostic.
- [ ] The primary prompt is ready to paste without editing placeholders.

## Example invocation

> Use the Website Preview Image Director skill on `https://example.com/work/project-alpha`. Create a 16:9 project-card preview. The image should represent the project, use the page's visual identity, and leave enough negative space for a title overlay. Return the primary prompt and two variants.

## Official Midjourney references

Check these pages when parameter support or reference behavior needs verification:

- Prompt basics: https://docs.midjourney.com/hc/en-us/articles/32023408776205-Prompt-Basics
- Parameter list: https://docs.midjourney.com/hc/en-us/articles/32859204029709-Parameter-List
- Style Reference: https://docs.midjourney.com/hc/en-us/articles/32180011136653-Style-Reference
- Image Prompts: https://docs.midjourney.com/hc/en-us/articles/32040250122381-Image-Prompts
