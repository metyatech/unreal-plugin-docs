---
---

# Editor Actor Tag Display

[Back to plugin list](../)

## Overview

Editor Actor Tag Display shows an actor's `Actor Tags` above the actor in the Unreal Editor viewport.

Display behavior is configured per actor class. Each configured class can have its own text color and position offset.

The plugin is editor-only and does not add a runtime feature to packaged games.

## Requirements

- Unreal Engine 5.6
- Windows Win64
- Actors with at least one value in the actor-level `Tags` array
- At least one configured actor class
- Actor Tag Display enabled in settings or through the viewport Show menu

Use only plugin packages explicitly marked compatible with the project's Unreal Engine version.

## Installation

### From Fab

1. Acquire **Editor Actor Tag Display** from Fab.
2. Install the plugin for the appropriate Unreal Engine version.
3. Open the project.
4. Open **Edit > Plugins**.
5. Enable **Editor Actor Tag Display**.
6. Restart the editor when requested.

### From Source

Place the repository at:

```text
<Project>/Plugins/EditorActorTagDisplay
```

Regenerate project files or open the project and accept the rebuild prompt.

## Quick Start

1. Open **Editor Preferences**.
2. Open **Plugins > Actor Tag Display**.
3. Add an entry to **Class Configurations**.
4. Select the actor class whose tags should be displayed.
5. Choose the text color.
6. Set the optional position offset.
7. Enable **Enable Tag Display**.
8. Select an actor of the configured class.
9. Add one or more values to the actor's `Tags` array.
10. Return to the level viewport.

The tags appear above the actor.

The display can also be toggled from:

```text
Level Viewport > Show > Actor Tags
```

## Features

* Displays actor-level tags in the editor viewport
* Filters actors by configured actor class
* Matches subclasses through Unreal's class hierarchy
* Per-class text color
* Per-class position offset
* Global text-size setting
* Global outline-width setting
* Viewport Show menu toggle
* Multi-tag display with one tag per line
* Automatic placement above actor bounds
* Automatic camera-facing text
* Automatic cleanup when actors or configurations no longer match
* Transient display actors hidden from the Scene Outliner

## Settings / Blueprint Nodes

This plugin does not expose Blueprint nodes. Configuration is performed through editor settings.

### Class Configurations

Each entry contains:

| Field             | Purpose                                                                 |
| ----------------- | ----------------------------------------------------------------------- |
| `Actor Class`     | Actor class and subclasses whose actor-level tags are displayed.        |
| `Display Color`   | Text color for the matching class.                                      |
| `Position Offset` | World-space offset added after calculating the top of the actor bounds. |

When more than one configuration could match an actor, the first matching configuration is used.

### Enable Tag Display

Enables or disables all generated tag labels.

The same value is controlled by the viewport **Show > Actor Tags** toggle.

### Text Size

Sets the world size of all tag text.

Default:

```text
30
```

Existing labels are updated when the value changes.

### Outline Width

Sets the outline-width parameter used by the bundled text material.

Default:

```text
10
```

Existing dynamic material instances are updated when the value changes.

### Display Position

The plugin calculates the top of the actor's component bounds and then adds the configured position offset.

When valid component bounds are unavailable, the actor origin is used before applying the offset.

### Stored Settings

The settings use `EditorPerProjectUserSettings`. They are stored per user and per project rather than as shared project-wide defaults.

## Limitations

* Editor-only
* Win64 only in the current plugin descriptor
* Not available as a packaged-game runtime feature
* Displays actor-level tags only
* Component tags are not displayed
* Actors without tags are ignored
* Actors without a matching configured class are ignored
* The first matching class configuration wins
* Tags are displayed one per line
* No per-tag color or position configuration
* No tag-name include or exclude filter
* No distance culling
* No occlusion testing
* No custom font setting
* No label background
* Settings are per user and per project
* The editor world is scanned continuously while display is enabled, which may add editor overhead in very large levels

## Troubleshooting

### No tag text appears

Verify that:

* **Enable Tag Display** is enabled.
* **Show > Actor Tags** is checked.
* The actor has at least one actor-level tag.
* A Class Configuration exists.
* The configured class matches the actor or one of its parent classes.
* The plugin is enabled.
* The current platform and Unreal Engine version are supported.

### The wrong color or offset is used

An actor may match more than one configured parent class. The first matching Class Configuration is used. Reorder the configurations so the intended entry appears first.

### The text is inside the actor

Increase the class-specific `Position Offset`, especially its Z value.

Actors without valid component bounds use the actor origin as the base position.

### Another user does not see the same configuration

The settings are stored as `EditorPerProjectUserSettings`. Each user must configure or receive the appropriate editor-user settings separately.

### Performance decreases in a very large level

Disable **Actor Tags** when labels are not required, or reduce the number of configured classes and tagged actors.

For reproducible problems, open an issue in the [EditorActorTagDisplayPlugin repository](https://github.com/metyatech/EditorActorTagDisplayPlugin/issues).

## Version History

### 1.0.0

* Initial editor viewport tag display
* Per-class color and position configuration
* Text size and outline width settings
* Viewport Show menu integration
