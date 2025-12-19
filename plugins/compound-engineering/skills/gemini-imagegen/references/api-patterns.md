<overview>
Advanced API patterns for Gemini image generation including editing, multi-turn refinement, and composition.
</overview>

<pattern name="editing">
## Editing Images

Pass existing images with text prompts:

```python
from PIL import Image

img = Image.open("input.png")
response = client.models.generate_content(
    model="gemini-3-pro-image-preview",
    contents=["Add a sunset to this scene", img],
    config=types.GenerateContentConfig(
        response_modalities=['TEXT', 'IMAGE'],
    ),
)
```

**Note:** Model understands semantic masking—describe changes conversationally.
</pattern>

<pattern name="multi-turn">
## Multi-Turn Refinement

Use chat for iterative editing:

```python
from google.genai import types

chat = client.chats.create(
    model="gemini-3-pro-image-preview",
    config=types.GenerateContentConfig(response_modalities=['TEXT', 'IMAGE'])
)

response = chat.send_message("Create a logo for 'Acme Corp'")
# Save first image...

response = chat.send_message("Make the text bolder and add a blue gradient")
# Save refined image...

response = chat.send_message("Now add a subtle shadow behind the text")
# Save final image...
```
</pattern>

<pattern name="multiple-images">
## Multiple Reference Images (Up to 14)

Combine elements from multiple sources:

```python
response = client.models.generate_content(
    model="gemini-3-pro-image-preview",
    contents=[
        "Create a group photo of these people in an office",
        Image.open("person1.png"),
        Image.open("person2.png"),
        Image.open("person3.png"),
    ],
    config=types.GenerateContentConfig(
        response_modalities=['TEXT', 'IMAGE'],
    ),
)
```
</pattern>

<pattern name="search-grounding">
## Google Search Grounding

Generate images based on real-time data:

```python
response = client.models.generate_content(
    model="gemini-3-pro-image-preview",
    contents=["Visualize today's weather in Tokyo as an infographic"],
    config=types.GenerateContentConfig(
        response_modalities=['TEXT', 'IMAGE'],
        tools=[{"google_search": {}}]
    )
)
```

**Note:** Image-only mode won't work with Google Search grounding.
</pattern>

<pattern name="resolution">
## Resolution Options

```python
# 1K (default) - Fast, good for previews
image_config=types.ImageConfig(image_size="1K")

# 2K - Balanced quality/speed
image_config=types.ImageConfig(image_size="2K")

# 4K - Maximum quality, slower
image_config=types.ImageConfig(image_size="4K")
```

**Recommendation:** Default to 1K; use 2K/4K when quality is critical.
</pattern>

<pattern name="aspect-ratios">
## Aspect Ratio Examples

```python
# Square (default)
image_config=types.ImageConfig(aspect_ratio="1:1")

# Landscape wide (videos, headers)
image_config=types.ImageConfig(aspect_ratio="16:9")

# Ultra-wide panoramic
image_config=types.ImageConfig(aspect_ratio="21:9")

# Portrait (stories, mobile)
image_config=types.ImageConfig(aspect_ratio="9:16")

# Photo standard
image_config=types.ImageConfig(aspect_ratio="4:3")
```
</pattern>

<pattern name="file-format">
## File Format (CRITICAL)

**Gemini returns JPEG by default.** Always use `.jpg` extension:

```python
# CORRECT
image.save("output.jpg")

# WRONG - causes "Image does not match media type" errors
image.save("output.png")  # Creates JPEG with PNG extension!
```

**Converting to PNG:**

```python
from PIL import Image

for part in response.parts:
    if part.inline_data:
        img = part.as_image()
        img.save("output.png", format="PNG")  # Explicit format
```

**Verify format:**
```bash
file image.png
# If output shows "JPEG image data" - rename to .jpg!
```
</pattern>
