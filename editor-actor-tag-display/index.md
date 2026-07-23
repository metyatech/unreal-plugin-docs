---
---

# Actor Metadata Overlay

[Back to plugin list](../)

## Overview

Actor Metadata Overlay is an editor-only Unreal Engine viewport tool that keeps the metadata needed for level layout and debugging visible above matching actors. It displays Actor Label, Actor Name, Actor Class, Actor Tags, Gameplay Tags, Folder, Data Layers, and selected direct property values through configurable templates.

## Requirements

- Unreal Engine 5.6, 5.7, or 5.8
- Windows Win64
- Editor only
- A normal Level Editor Viewport

Use the ZIP built for the project's Unreal Engine version. Runtime packaged games are not supported.

## Installation

### From Fab

1. Download the ZIP for the project's Unreal Engine version.
2. Place the `EditorActorTagDisplay` folder in the project's `Plugins` directory.
3. Enable **Actor Metadata Overlay** from **Edit > Plugins**.
4. Restart the editor when requested.

### From source

Place the repository in `<Project>/Plugins/EditorActorTagDisplay`, regenerate project files if needed, and enable the plugin from **Edit > Plugins**.

## Quick Start

1. Open **Project Settings > Plugins > Actor Metadata Overlay**.
2. Confirm the default rule is enabled, or add a rule for the actor class you want to inspect.
3. Choose a template such as `{ActorLabel}\nClass: {ActorClass}\nTags: {ActorTags}`.
4. Select an actor in a normal Level Editor Viewport.
5. Open the viewport **Show** menu and choose **Actor Metadata Overlay > Selected Actors**.
6. Switch to **All Matching Actors** to inspect every matching actor, or **Off** to hide the overlay.

Turn **Game View** on to hide the overlay. Turn it off to show it again. PIE and SIE game screens, Static Mesh Editor previews, and other Asset Preview Viewports never display the overlay.

## Display Modes

- **Selected Actors** displays matching selected actors.
- **All Matching Actors** displays every cached actor that matches a rule.
- **Off** hides all overlay text and bounding boxes.

These modes are available from the Level Editor Viewport **Show** menu as radio buttons.

## Features

- Actor Label, Name, and Class tokens.
- Actor Tags and Gameplay Tags sorted case-insensitively.
- Folder and Data Layers tokens.
- Direct Property Tokens for safe, public, non-transient properties.
- Selected, All, and Off display modes.
- Required and excluded Actor Tag filters.
- Per-rule and global distance filtering.
- Optional actor bounding boxes.
- Separate shared Project Settings and per-user Editor Preferences.
- Event-driven cache updates and per-viewport Canvas rendering.

## Rule Evaluation

Rules are evaluated from top to bottom. The first enabled rule whose Actor Class, inheritance setting, required Actor Tags, and excluded Actor Tags match owns the actor. Later matching rules are ignored. Empty or unresolved Actor Classes never match.

## Rule Fields

| Field | Description |
| --- | --- |
| Rule Name | Identifies the rule and unresolved-class warnings. |
| Enabled | Disables a rule without removing it. |
| Actor Class | Required class filter. |
| Include Derived Classes | Uses class inheritance when enabled. |
| Required Actor Tags | Every listed Actor Tag must be present. |
| Excluded Actor Tags | Any listed Actor Tag rejects the rule. |
| Display Template | Overrides the project default when non-empty. |
| Display Color | Text and bounding-box color. |
| World Offset | Offset added to the actor anchor. |
| Max Draw Distance | Overrides the global distance when greater than zero. |
| Selected Only | Requires the actor to be selected. |
| Draw Bounding Box | Enables the rule's box when the user preference also allows boxes. |

## Template Tokens

| Token | Value |
| --- | --- |
| `{ActorLabel}` | Actor label shown in the editor. |
| `{ActorName}` | UObject actor name. |
| `{ActorClass}` | Actor class name with a display-only `_C` suffix removed. |
| `{ActorTags}` | Actor Tags sorted and joined by `, `. |
| `{GameplayTags}` | Owned Gameplay Tags sorted and joined by `, `. |
| `{Folder}` | Actor folder path. |
| `{DataLayers}` | Actor data layer names sorted and joined by `, `. |
| `{Property:PropertyName}` | A supported direct property value. |

Unknown tokens become `<unknown:TokenName>`. Missing properties become `<missing:PropertyName>`. Unsupported properties become `<unsupported:PropertyName>`.

## Property Tokens

Property tokens accept direct top-level public properties with `CPF_Edit` or `CPF_BlueprintVisible`. They reject dotted paths, array indexes, functions, transient properties, deprecated properties, and private properties. Supported values include booleans, signed and unsigned integers, floats, doubles, enums, `FName`, `FString`, `FText`, UObject references, soft object references, and structs. Line breaks and tabs become spaces, and long values are shortened with `...`.

## Project Settings

Open **Project Settings > Plugins > Actor Metadata Overlay** to configure shared project data:

- Rules and rule order.
- Default Display Template.
- Outline Color.
- Max Property Value Length.

## Editor Preferences

Open **Editor Preferences > Plugins > Actor Metadata Overlay** to configure per-user display preferences:

- Display Mode.
- Global Max Draw Distance.
- Text Scale.
- Outlined.
- Draw Bounding Boxes.

## Viewport Behavior

The plugin draws only into the matching normal Level Editor Viewport scene. It displays in Perspective, Top, Front, Side, and split Level Editor Viewports, and remains visible when **Game View** is off. It is hidden when **Game View** is on, during PIE or SIE game screens, in Static Mesh Editor previews, and in Scene Capture, Thumbnail, or Asset Preview Viewports.

## Performance

The overlay uses an event-driven cache. Level changes, actor movement, folder changes, property changes, component changes, map changes, and settings changes refresh the affected cache or rebuild it once. Drawing traverses cached matching actors per viewport rather than scanning the world every frame.

## Limitations

- Editor only; no runtime module or packaged-game feature.
- Win64 only.
- Actor Tags are displayed but not edited.
- Data Layers are empty in a non-World Partition map; verify non-empty output in a World Partition map.
- Property Tokens support direct properties only.
- Asset Preview Viewports are intentionally excluded.

## Troubleshooting

### Nothing appears

Confirm the plugin is enabled, the editor was restarted, the viewport mode is **Selected Actors** or **All Matching Actors**, the actor matches the first rule, and the actor is visible and within the applicable distance limit.

### The overlay disappears

Turn **Game View** off. The overlay is intentionally hidden in Game View, PIE, SIE, and preview viewports.

### Data Layers are empty

Use a World Partition map and assign the actor to data layers. New non-World Partition maps have no data layers to display.

### The wrong rule is used

Rules are first-match-wins. Move the intended rule above later matching rules and verify required and excluded Actor Tags.

## Support

Open an issue in the [EditorActorTagDisplayPlugin repository](https://github.com/metyatech/EditorActorTagDisplayPlugin/issues).

## Version History

### 2.0.0

- Renamed the product to Actor Metadata Overlay.
- Replaced transient TextRender actors with per-viewport canvas rendering.
- Added rule-based metadata templates.
- Added Actor Tags, Gameplay Tags, folder, data layer, and property tokens.
- Added Selected, All, and Off modes.
- Added distance filtering and optional bounding boxes.
- Split project and per-user settings.
- Added event-driven cache updates.
- Restricted drawing to matching Level Editor viewport scenes.
- Added Automation Tests.

### 1.0.0

- Initial actor tag viewport display.
