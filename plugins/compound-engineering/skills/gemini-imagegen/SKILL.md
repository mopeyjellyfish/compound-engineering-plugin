---
name: gemini-imagegen
description: Generate and edit images using Gemini API. Use for text-to-image, image editing, style transfers, logos, stickers, and product mockups. Requires GEMINI_API_KEY.
---

# Gemini Image Generation

Generate and edit images using Google's Gemini API. Requires `GEMINI_API_KEY` environment variable.

## Quick Reference

| Setting | Default | Options |
|---------|---------|---------|
| Model | `gemini-3-pro-image-preview` | Always use Pro |
| Resolution | 1K | 1K, 2K, 4K |
| Aspect Ratio | 1:1 | 1:1, 16:9, 9:16, 4:3, 3:2, 21:9 |

## Core Pattern

```python
import os
from google import genai
from google.genai import types

client = genai.Client(api_key=os.environ["GEMINI_API_KEY"])

response = client.models.generate_content(
    model="gemini-3-pro-image-preview",
    contents=["Your prompt here"],
    config=types.GenerateContentConfig(
        response_modalities=['TEXT', 'IMAGE'],
    ),
)

for part in response.parts:
    if part.inline_data:
        image = part.as_image()
        image.save("output.jpg")  # MUST use .jpg (Gemini returns JPEG)
```

## Custom Resolution & Aspect Ratio

```python
config=types.GenerateContentConfig(
    response_modalities=['TEXT', 'IMAGE'],
    image_config=types.ImageConfig(
        aspect_ratio="16:9",
        image_size="2K"
    ),
)
```

## CRITICAL: File Format

**Gemini returns JPEG by default.** Always use `.jpg`:

```python
# ✅ CORRECT
image.save("output.jpg")

# ❌ WRONG - causes errors
image.save("output.png")
```

**Full API patterns:** `references/api-patterns.md`

## Quick Prompting Tips

| Type | Key Elements |
|------|-------------|
| Photorealistic | Lens, lighting, depth of field |
| Stylized art | Style name, outlines, shading |
| Text in images | Exact text in quotes, font style |
| Product mockups | Surface, lighting setup, angle |

**Full prompting guide:** `references/prompting.md`

## Notes

- All images include SynthID watermarks
- Default to 1K for speed; 2K/4K when quality matters
- Model understands semantic editing—describe changes conversationally
- Supports up to 14 reference images for composition

## Reference Files

| File | Content |
|------|---------|
| `references/api-patterns.md` | Editing, multi-turn, composition, formats |
| `references/prompting.md` | Best practices by image type |
