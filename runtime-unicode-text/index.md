---
---

# Runtime Unicode Text

[Back to plugin list](../)

Version: 0.1.0

## Overview

Runtime Unicode Text is an Unreal Engine Code Plugin for rendering runtime
Unicode text in world space. It supports two workflows:

- Place a `RuntimeUnicodeTextActor` directly in the level.
- Add a `RuntimeUnicodeTextComponent` to an existing Actor.

Slate performs shaping and rasterization, and the plugin uses a shared,
plugin-owned GPU glyph atlas. It does not require a Widget Component, UMG, or
Text3D. While the application is running, `SetText` can change the component
to text that contains previously unused characters.

Version 0.1.0 has been validated on **Unreal Engine 5.8 / Win64**. Real-font
integration testing covers Japanese text, Latin text, mixed
Latin/Japanese/numeric text, `龍鬱鶴`, and multiline text. This release does not
provide broad script certification.

## Requirements

- Unreal Engine 5.8
- Win64
- Recommended: a project-owned, cooked, Runtime Cached `UFont`
- Offline font pages are not a prerequisite.
- The plugin package does not require glyph pre-enumeration or pre-baking.

UE 5.7 and earlier, and platforms other than Win64, are not certified by this
release.

## Installation

### From Fab

1. Acquire **Runtime Unicode Text** from Fab.
2. Install it for the supported Engine version.
3. Open the project.
4. Open **Edit > Plugins**.
5. Enable **Runtime Unicode Text**.
6. Restart the editor if requested.

### Project Plugin / Source Checkout

For development workflows where you already have repository access, the
plugin root is:

```text
<Project>/Plugins/RuntimeUnicodeText/
```

Place the plugin checkout at that path, then use the project's normal source
build workflow.

## Quick Start — Actor

1. From **Place Actors**, place the **Runtime Unicode Text** Actor.
2. Select its root **Runtime Unicode Text Component**.
3. Set `Font` to a Runtime Cached `UFont`.
4. Set `Text`, for example, to `こんにちは世界`.
5. Adjust `WorldSize`, `Color`, and alignment.
6. Check the result in PIE.

The Actor owns a `RuntimeUnicodeTextComponent` as its root component. The
component is the source of truth; the Actor has no separate rendering state.

The `Aあ` Billboard/Class icon shown in the editor is editor-only and does not
affect packaged rendering.

## Quick Start — Component

Add **Runtime Unicode Text Component** to an existing custom Actor and
configure that component directly. The component is the source of truth for:

- `Text`
- `Font`
- `WorldSize`
- `Color`
- `HorizontalAlignment`
- `VerticalAlignment`

Use the Component workflow when the text should be part of an existing
custom Actor rather than a separately placed text Actor.

## Properties

| Property | Contract |
| --- | --- |
| `Text` | Unicode text. Explicit newline is supported. |
| `Font` | Runtime Cached `UFont`. |
| `WorldSize` | Font em height in world units. Default `80.0`; minimum `0.01`. |
| `Color` | Rendered vertex/material color. |
| `HorizontalAlignment` | `Left` / `Center` / `Right`. |
| `VerticalAlignment` | `Top` / `Center` / `Bottom`. |

## Blueprint API

The component's public Blueprint-callable functions are:

```text
SetText
SetFont
SetColor
SetWorldSize
```

The Actor also provides this BlueprintPure getter:

```text
Get Runtime Unicode Text Component
```

Example:

```text
Event BeginPlay
  → Get Runtime Unicode Text Component
  → SetText (New Text: こんにちは世界)
```

## Font Behavior

The formal recommendation is to set an explicit project-owned Runtime Cached
`UFont`.

When `Font == nullptr`, the current Win64 implementation checks `%WINDIR%\Fonts\`
for an installed font in this order:

```text
NotoSansJP-VF.ttf
YuGothM.ttc
meiryo.ttc
```

This fallback depends on the PC environment. **Do not rely on it for a
portable or reproducible packaged result**; use an explicit Runtime Cached
`UFont` instead.

The plugin package does not include the Noto Sans JP font itself. The Noto
Sans JP font used by the Sample is a Sample-side third-party dependency and
is not included in the Plugin product.

## Runtime Rendering Behavior

- Shaping and rasterization use Slate.
- Glyphs are copied into a shared plugin-owned BGRA8 atlas.
- A stable destination glyph cache is used.
- Arbitrary Actor and world rotation are supported.
- Text is not billboard text; the scene proxy uses the normal world transform.
- First-use glyphs can be added at runtime.

## Packaging

- UE 5.8 Win64 packaged Development and Shipping builds have been validated.
- The material in Plugin Content is normally cooked together with the plugin.
- If you use a project-owned `UFont`, configure the project so that the font
  asset is included in the packaged build.
- For portable results, use an explicit Runtime Cached `UFont` instead of the
  Windows system-font fallback.

No other platform is implied by this packaging validation.

## Performance / Glyph Cache

- Atlas eviction is not implemented.
- Atlas memory increases as unique glyphs are added.
- A stress test reached 16 `1024x1024 BGRA8` atlas pages, approximately 64 MiB
  of atlas memory.
- Large simultaneous `SetText` updates, such as 100 components, may hitch.
- Speculative size quantization, eviction, raster buckets, and similar
  changes are not implemented in v0.1.0.

## Limitations

- UE 5.8 / Win64 only is certified.
- Atlas eviction is not implemented.
- Rich text is not supported.
- Outline is not supported.
- Shadow is not supported.
- Word wrapping is not supported.
- Vertical writing is not supported.
- Emoji support is not guaranteed.
- Automatic font downloading is not provided.
- Broad script certification is not provided.
- Very large simultaneous `SetText` workloads may hitch.
- The plugin itself does not bundle a font product.
- Automatic network replication is not provided.

## Troubleshooting

### Text does not appear

Check the following:

- The plugin is enabled.
- The Component or Actor is registered.
- An explicit Runtime Cached `UFont` is set.
- The font contains the target glyphs.
- The project is running on Win64 with UE 5.8.
- The project font is cooked into the packaged build.
- The Output Log contains a `RUNTIME_UNICODE_TEXT` warning that identifies
  the failure.

If `Font` is unset and no installed fallback font is found, the
implementation logs:

```text
RUNTIME_UNICODE_TEXT no Runtime Cached UFont or installed default font
```

### Packaged output uses a different or missing font

Do not depend on the system fallback. Set an explicit project-owned Runtime
Cached `UFont` and ensure that the font asset is included in the packaged
build.

### First appearance of new text hitches

The first use of new text may perform Slate shaping and rasterization and
populate the glyph atlas. Avoid updating many components with new text at the
same time when a hitch is unacceptable.

### Missing characters

Confirm that the selected font contains the missing glyphs. The plugin does
not provide font downloading or a font fallback package.

## Sample / Verification

The companion `RuntimeUnicodeTextSample` project is used for integration
verification. The Sample owns the Noto Sans JP Runtime Cached font asset.

Its real-font test is:

```text
RuntimeUnicodeTextSample.Layout.RealNotoSansJP
```

The test verifies Japanese text, mixed Latin/Japanese/numeric text,
`龍鬱鶴`, and multiline text. The Sample font is not included in the Plugin
package.

## Version History

### 0.1.0

- Initial Runtime Unicode Text release.
- Runtime world-space Unicode text through Actor and Component workflows.
- Runtime `SetText`, `SetFont`, `SetColor`, and `SetWorldSize`.
- Runtime Cached `UFont` support.
- Horizontal and vertical alignment.
- Multiline text.
- Runtime first-use glyph generation through Slate and the shared stable glyph atlas.
- UE 5.8 Win64 validation.
