---
name: prompt-images
description: >
  Prompting techniques for AI image generation and editing models on Replicate.
  Use when writing prompts for image models, choosing which image model to use
  for a task, or building image generation features. Covers FLUX, Seedream,
  Nano Banana Pro, Ideogram, Recraft, Imagen, and Stable Diffusion.
---

# Prompting image models on Replicate

Distilled from Replicate's blog posts on prompting image models (2024-2026). All models referenced are available on Replicate.

## Universal prompting principles

These techniques work across most modern image generation models.

### Use natural language, not keyword lists

Write full sentences describing what you want. Modern models (FLUX, Seedream 5, Nano Banana Pro, Imagen 4) understand grammar and context far better than keyword-stuffed prompts.

Good: "A woman standing in a Tokyo alleyway at dusk, neon signs reflecting off wet pavement"
Bad: "woman, Tokyo, alleyway, dusk, neon, wet pavement"

### Be specific and unambiguous

Name exact colors, materials, lighting setups, camera equipment, and spatial relationships. Vague terms like "make it better" or "artistic" give unpredictable results.

Good: "A brutalist concrete building reflected in a perfectly still puddle after rain. A single figure with a red umbrella walks along the edge, the only color in an otherwise monochrome scene. Overcast sky, flat diffused light, tilt-shift lens effect on the edges."
Bad: "Cool building with a person near it, rainy day"

### Specify what to keep when editing

When editing images, explicitly state what should remain unchanged. Use phrases like "keeping the pose and expression unchanged" or "maintain the original composition."

### Use quotation marks for text rendering

When you want specific text rendered in an image, wrap it in double quotes within the prompt:

"Design a poster with the title \"BLUE NOTE SESSIONS\" in bold condensed sans-serif"

### Start simple, then iterate

Begin with basic changes. Test small edits first, then build on what works. Most editing models support iterative editing, so take advantage of that.

### Name subjects directly

Use descriptive phrases like "the woman with short black hair" or "the red car." Avoid pronouns, which are often too ambiguous for image models.

### Use photographic language

Modern models understand camera and photography terminology deeply. Reference specific film stocks (Kodak Portra 800), lens characteristics (50mm Summilux wide open), lighting setups (Rembrandt lighting), and shooting techniques (shallow depth of field, golden hour).

### Prompt length matters

Most modern models accept very long prompts (SD3: 10,000+ characters, Recraft V4: 10,000 characters, FLUX.2: 32K tokens). Use the space. Long descriptive prompts with clear structure outperform short ones. But be aware: the longer and more complex the prompt, the more likely something will be missed.

### Choose verbs carefully

Words like "transform" suggest a full rework. If you want targeted changes, use specific actions like "change the clothes to a blue jacket" or "replace the background with a beach." "Put him on the beach" is too vague. Better: "Change the background to a beach while keeping the person in the exact same position, maintain identical subject placement, camera angle, framing, and perspective. Only replace the environment around them."


## Model-specific guidance

### FLUX family (Black Forest Labs)

FLUX uses flow matching rather than diffusion. This produces a distinctive "flow" aesthetic with organic movement and fluidity.

**FLUX.1 Kontext** (editing model):
- Best in class for editing images using text prompts
- Better and cheaper than gpt-image-1 for most editing tasks
- Comes in pro ($0.04, 5s), max ($0.08, 7s), and dev ($0.025, 4s) variants
- Iterative editing is a core strength: make small changes, build on results
- For text editing, use the pattern: "Change 'old text' to 'new text'"
- For style transfer, name the exact style: "impressionist painting," "watercolor sketch," or reference specific movements and artists
- Control composition explicitly: say if you want to keep camera angle or framing
- Struggles with object removal (leaves artifacts). Use other models for that.

**FLUX.1 Tools** (control/steerability):
- Fill: inpainting and outpainting. Great at text inpainting (mask text, prompt for new text, matches original style)
- Canny: edge detection conditioning for structure-preserving generation
- Depth: depth map conditioning for realistic perspective
- Redux: generate variations of images, also supports text prompts for restyling

**FLUX.2** (latest generation):
- Multi-reference support: up to 8 input images simultaneously
- Character, product, and style consistency across references
- Better text rendering, hands, faces, fabrics than FLUX.1
- Structured prompting for programmatic workflows
- 32K prompt tokens for detailed creative direction
- Supports generative expand/crop for post-generation edits
- Pro ($0.015+/megapixel, 6s), Flex (higher quality, 22s), Dev (open source, 2.5s)

**FLUX fine-tunes / LoRAs**:
- Use trigger words from your trained model in every prompt
- Combine multiple LoRAs by setting `extra_lora` and `extra_lora_scale` parameters
- Balance multiple LoRAs with scales between 0.9 and 1.1
- Generate synthetic training data from outputs: generate many images, pick the best, retrain
- Use consistent-character model to generate training data from a single reference image

### Seedream 5.0 (ByteDance)

- Understands photographic language deeply: film stocks, lens characteristics, lighting setups
- Use natural language, not keyword lists
- Use double quotation marks around text you want rendered in the image
- Supports example-based editing: provide a before/after pair (Image 1 and Image 2), then a third image. The model infers the transformation and applies it to the third image. No text description of the edit needed.
- Example-based editing works for: material swaps, scene changes (day to night), style transfers
- Multi-step reasoning: can classify objects, sort items, apply sequential logic from a single prompt
- Can understand visual markers (arrows, bounding boxes, colored regions) drawn on input images, then follow instructions referencing them
- Multi-image generation: ask for "a series" or "a set" or specify a grid layout for consistent style and character continuity across panels
- Strong domain knowledge: architecture (floor plan to render), science (ecosystem diagrams), food (nutritional annotations)
- For complex edits, show don't tell: the example-based editing approach works better than trying to describe the transformation in words

### SeedEdit 3.0 (ByteDance)

- Tends to restrict itself to the initial composition, making it difficult to prompt a new angle or scene
- Outputs are typically softer and can look more AI-generated
- Good at background editing: natural lighting and believable placement
- Good cheap alternative when gpt-image-1 is too expensive and Kontext isn't working

### Nano Banana Pro (Google)

- Built on Gemini architecture: has intermediary reasoning layers between input and output
- Best-in-class text adherence: input any text and the model renders it word for word
- Text adherence is maintained even with stylistic transformations
- Handles up to 14 reference images simultaneously for character consistency
- Baked-in logic: can read text in images, solve math problems, interpret code, generate infographics from documents
- Great for: infographics, magazine layouts, posters, annotated diagrams, educational visuals, app mockups
- Can render code snippets accurately
- Characters persist across multiple iterative generations with minimal prompting
- World knowledge: can identify landmarks from GPS coordinates, understands real-world context
- Not connected to the internet, so no real-time information
- Weakest at: background editing (tends to cut out subjects poorly), object removal

### Imagen 4 (Google)

- Fine detail rendering: textures, water droplets, animal fur
- Wide style versatility: photorealistic to abstract art to illustrations
- Strong typography rendering
- Precise and detailed prompts are key: specify subjects, environments, artistic styles, and compositional elements
- Consider using an LLM to enhance your prompt before passing it to Imagen 4
- Supports common aspect ratios via `aspect_ratio` parameter

### Ideogram 3.0

- Three variants: Turbo (fast iteration), Balanced (compromise), Quality (highest fidelity)
- Best-in-class typography and design layout
- Handles long or stylized phrases with precision
- Style transfer via reference images: upload visuals capturing the look you want (color palette, texture, composition, mood) and the model matches that aesthetic
- Useful for graphic design, marketing, posters, product shots with branding, ads with copy
- Strong realism: ranked highest in ELO scoring for realism, alignment, and versatility

### Ideogram v2

- Strong inpainting capabilities
- When Magic Prompt is off: describe the whole scene, not just the inpainted region
- When Magic Prompt is on: the model rewrites your prompt based on both the original prompt and the image, so you can focus on just the edit region
- If you only describe the inpainting region, the model emphasizes the prompt more, which can produce better results

### Recraft V4

- Designed around "design taste": makes intentional choices about composition, lighting, color
- Images feel art-directed rather than generic, even from simple prompts
- Typography is treated as a structural part of the image, not an afterthought
- Only model that produces native SVG output: real editable vector files with clean paths
- Four variants: V4 (raster, ~10s), V4 Pro (high-res raster, ~28s), V4 SVG (vector, ~15s), V4 Pro SVG (high-res vector, ~30s)
- Accepts prompts up to 10,000 characters
- Good at: editorial design, commercial product photography, macro detail, icon sets, stylized illustration

### Stable Diffusion 3

- Do NOT use negative prompts. SD3 was not trained with them. They introduce noise rather than removing unwanted elements.
- Use long, descriptive natural English sentences. No more 77-token CLIP limit.
- Can pass different prompts to each of the three text encoders (two CLIP + T5-XXL)
- Recommended settings: 28 steps, CFG 3.5-4.5, dpmpp_2m sampler with sgm_uniform scheduler, shift 3.0
- If outputs look "burnt" (too much contrast), lower the CFG
- Resolution: best at ~1 megapixel. Common ratios: 1024x1024 (1:1), 1344x768 (16:9), 832x1216 (2:3)
- Subjects can change dramatically at different step values (age, gender, ethnicity may shift as steps increase)
- Lower CFG makes outputs more similar across text encoder options, so you can skip the large T5 encoder at low CFG
- Shift parameter: higher values handle noise better at higher resolutions. 3.0 is default, 6.0 also scores well. Lower values (1.5-2.0) give a more raw look.


## Task-specific guidance

### Text rendering

Best models: Nano Banana Pro, Seedream 5, FLUX Kontext, Recraft V4, Ideogram 3

- Wrap desired text in double quotation marks within the prompt
- Stick to readable fonts. Highly stylized text may not work as well.
- When editing text: use "Change 'X' to 'Y'" pattern for best results
- Match text length when possible: big shifts in length can change layout
- Be explicit about preserving font style if it matters
- For complex typography (posters, editorial layouts): Recraft V4 and Ideogram 3 treat text as part of the composition rather than stamping it on top
- Nano Banana Pro maintains text accuracy even during style transformations
- FLUX Fill can inpaint text: mask the text region, prompt with new text, and it matches the original style

### Style transfer

Best models: FLUX Kontext (pro), Nano Banana Pro, Seedream 5, Ideogram 3

- Name the exact style: "impressionist painting," "1960s pop art," "Sumi-e ink wash"
- Reference specific artists or movements for clearer guidance
- If a style label doesn't work, describe its key traits: "visible brushstrokes, thick paint texture, rich color depth"
- State what should stay the same: "keep the original composition"
- For Seedream 5: use example-based editing (show before/after pair) when the style is hard to describe
- For Ideogram 3: upload style reference images to capture color palette, texture, mood
- Do NOT use Runway Gen-4 for stylistic tasks. It cannot restyle scenes.

### Character consistency

Best models: Runway Gen-4 Image (photos), FLUX Kontext (creative), FLUX.2 (multi-reference), Nano Banana Pro (up to 14 refs)

- Start with a clear reference description: "the woman with short black hair and green eyes"
- Say what's changing (setting, activity, style) and what should stay the same (face, expression, clothing)
- For photos: start with Runway Gen-4 Image, fall back to Kontext Pro for speed/cost
- For creative transformations: start with Kontext Pro, fall back to gpt-image-1 for complex tasks
- FLUX.2 supports up to 8 reference images for multi-reference consistency
- Nano Banana Pro handles up to 14 reference images
- Break complex character changes into steps: change outfit first, then change scene

### Image editing (object removal, background swap, perspective changes)

**Object removal:**
Best: SeedEdit 3.0, Qwen Image Edit
Worst: FLUX Kontext (leaves artifacts like towers/structures)

**Background editing:**
Best: SeedEdit 3.0, Seedream 4 (natural integration, consistent lighting)
Worst: Nano Banana Pro (tends to cut subjects poorly)

**Perspective/angle changes:**
Best: GPT-image-1, Qwen Image Edit
Note: ByteDance models (SeedEdit, Seedream) struggle with turning characters or changing viewing angles

**Text editing in images:**
Best: FLUX Kontext, Nano Banana Pro (preserve typography and texture)
Worst: GPT-image-1 (changes the whole note/surface), Seedream 4 (produces artifacts)

### Inpainting and outpainting

Best models: FLUX Fill (pro/dev), Ideogram v2

- FLUX Fill excels at text inpainting: mask text, prompt with new text, matches original style
- For Ideogram v2 inpainting:
  - Magic Prompt off: describe the whole scene, not just the edited region
  - Magic Prompt on: focus on just the edit, model infers the rest
  - Describing only the inpainting region makes the model emphasize the prompt more
- Use ControlNet Canny for structure-preserving edits (turns sketches into detailed images)
- Use ControlNet Depth for realistic perspective in generated images

### Product photography and commercial work

Best models: Recraft V4, FLUX.2, Ideogram 3

- Specify materials precisely: "brushed steel," "matte aluminum," "kraft paper"
- Describe lighting setup: "soft diffused studio lighting, crisp highlights and gentle shadows"
- For brand assets: Recraft V4 SVG produces editable vector files (logos, icons, illustrations)
- FLUX.2 handles product mockups with consistent identity across variants
- Ideogram 3 handles layouts with branding and copy placement

### Multi-image and storyboard generation

Best models: Seedream 5, Nano Banana Pro

- Ask for "a series," "a set," or specify a grid layout (e.g., "2x2 storyboard grid")
- Describe each panel individually with consistent character descriptions
- Seedream 5: maintains consistent style and character continuity across panels
- Nano Banana Pro: create grids of editorial images with consistent style and color palette


## Model selection guide

### Quick reference by task

| Task                     | First choice             | Runner-up                   | Avoid           |
| ------------------------ | ------------------------ | --------------------------- | --------------- |
| Text rendering           | Nano Banana Pro          | Recraft V4                  |                 |
| Style transfer           | FLUX Kontext Pro         | Nano Banana Pro             | Runway Gen-4    |
| Character consistency    | Runway Gen-4 (photos)    | FLUX Kontext Pro            |                 |
| Object removal           | SeedEdit 3.0             | Qwen Image Edit             | FLUX Kontext    |
| Background editing       | SeedEdit 3.0             | Seedream 4                  | Nano Banana Pro |
| Text editing in images   | FLUX Kontext Pro         | Nano Banana Pro             | GPT-image-1     |
| Inpainting               | FLUX Fill Pro            | Ideogram v2                 |                 |
| Product photography      | Recraft V4               | FLUX.2                      |                 |
| SVG / vector output      | Recraft V4 SVG           | (no alternatives)           |                 |
| Design / typography      | Ideogram 3 Quality       | Recraft V4                  |                 |
| Infographics / diagrams  | Nano Banana Pro          | Seedream 5                  |                 |
| Multi-image / storyboard | Seedream 5               | Nano Banana Pro             |                 |
| Complex reasoning edits  | Seedream 5               | Nano Banana Pro             |                 |
| Fastest generation       | FLUX Kontext Dev (1.7s)  | FLUX.2 Dev (2.5s)           |                 |
| Cheapest per image       | GPT-image-1 ($0.01)     | FLUX Kontext Dev ($0.025)   |                 |

### Cost and speed reference

| Model              | Price per image   | Speed  |
| ------------------ | ----------------- | ------ |
| FLUX Kontext Dev   | $0.025            | 1.7s   |
| FLUX Kontext Pro   | $0.04             | 4.4s   |
| FLUX Kontext Max   | $0.08             | 4.9s   |
| FLUX.2 Pro         | $0.015+/megapixel | 6s     |
| FLUX.2 Flex        | $0.06/megapixel   | 22s    |
| FLUX.2 Dev         | $0.012/megapixel  | 2.5s   |
| Seedream 5         | ~$0.03            | ~14s   |
| SeedEdit 3.0       | $0.03             | 13s    |
| Nano Banana Pro    | $0.039            | 10s    |
| Qwen Image Edit    | $0.03             | 2.9s   |
| Ideogram 3 Turbo   | varies            | fast   |
| Ideogram 3 Quality | varies            | slower |
| Recraft V4         | $0.04             | ~10s   |
| Recraft V4 SVG     | $0.08             | ~15s   |
| Imagen 4           | varies            | varies |
| GPT-image-1        | $0.01-$0.25       | ~40s   |
| Runway Gen-4 Image | $0.05-$0.08       | 20-27s |


## Common pitfalls

1. **Negative prompts in SD3**: SD3 was not trained with negative prompts. Using them introduces noise and varies your output randomly rather than removing unwanted elements.

2. **Keyword-stuffed prompts on modern models**: FLUX, Seedream 5, Nano Banana Pro, and Imagen 4 all respond better to natural language sentences than comma-separated keyword lists.

3. **Using "transform" when you want a small edit**: In FLUX Kontext, "transform the person into a Viking" may swap the entire identity. Use targeted language: "change her outfit to Viking armor, keeping her face and expression unchanged."

4. **Not specifying what to keep**: When editing, always say what should stay the same. Without explicit instructions, models may change anything.

5. **Using Runway Gen-4 for style transfer**: Gen-4 is great for photographic character consistency but cannot restyle scenes. Use FLUX Kontext or Nano Banana Pro instead.

6. **Using FLUX Kontext for object removal**: Kontext often leaves structural artifacts (e.g., bridge towers remain after removing a bridge). Use SeedEdit 3.0 or Qwen Image Edit.

7. **Using Nano Banana Pro for background replacement**: It tends to cut out a piece of the subject and paste it onto a generic background. Use SeedEdit 3.0 or Seedream 4.

8. **Too-high CFG in SD3**: If images look "burnt" with excessive contrast, lower the guidance scale. Recommended range is 3.5-4.5.

9. **Expecting real-time knowledge**: No image model has internet access. Nano Banana Pro has strong world knowledge baked in, but it's from training data, not live.

10. **Short prompts for complex scenes**: Modern models accept thousands of tokens. For complex compositions with many specific requirements, use that capacity. A prompt with 12+ specific requirements (text on objects, labeled diagrams, color-coded elements, specific materials) can work if each requirement is stated clearly.

11. **GPT-image-1 yellow tint**: gpt-image-1 adds a distinctive yellow tint to outputs even with high quality settings. If color accuracy matters, use other models.

12. **Ignoring aspect ratio**: Most models have specific resolutions they work best at (~1 megapixel for SD3, common ratios for FLUX). Going too large produces edge artifacts. Going too small produces harsh crops. Use the model's recommended aspect ratios.


## Sources

All techniques in this skill are sourced from Replicate's blog:

- [How to prompt Seedream 5.0](https://replicate.com/blog/how-to-prompt-seedream-5) (Feb 2026)
- [Recraft V4](https://replicate.com/blog/recraft-v4) (Feb 2026)
- [Run FLUX.2 on Replicate](https://replicate.com/blog/run-flux-2-on-replicate) (Nov 2025)
- [How to prompt Nano Banana Pro](https://replicate.com/blog/how-to-prompt-nano-banana-pro) (Nov 2025)
- [Which image editing model should I use?](https://replicate.com/blog/compare-image-editing-models) (Sep 2025)
- [Generate consistent characters](https://replicate.com/blog/generate-consistent-characters) (Jul 2025)
- [Use FLUX.1 Kontext to edit images with words](https://replicate.com/blog/flux-kontext) (May 2025)
- [Imagen 4](https://replicate.com/blog/google-imagen-4) (May 2025)
- [Ideogram 3.0 on Replicate](https://replicate.com/blog/ideogram-v3) (May 2025)
- [FLUX.1 Tools](https://replicate.com/blog/flux-tools) (Nov 2024)
- [Ideogram v2 inpainting](https://replicate.com/blog/ideogram-v2-inpainting) (Oct 2024)
- [Using synthetic data to improve Flux finetunes](https://replicate.com/blog/using-synthetic-data-to-improve-flux-finetunes) (Sep 2024)
- [FLUX.1: First Impressions](https://replicate.com/blog/flux-first-impressions) (Aug 2024)
- [How to get the best results from Stable Diffusion 3](https://replicate.com/blog/get-the-best-from-stable-diffusion-3) (Jun 2024)
