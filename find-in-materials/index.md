# Find in Materials

Search Materials, Material Functions, and Material Instances across open assets and project-owned content, then open matching assets and navigate directly to matching graph expressions.

## Overview

Find in Materials is an editor plugin for searching Materials, Material Functions, and Material Instances without opening assets one by one. Unreal's built-in Material Editor search is useful within the Material already open; Find in Materials searches across project-owned Materials, Material Functions, and Material Instances, or just currently open supported assets.

Search expression types and names, parameter names and values, texture and Material Function references, comments, and asset paths. Results are grouped by asset and show the matching expressions or overrides.

## Requirements

- Unreal Engine **5.6, 5.7, or 5.8**
- **Win64**
- **Unreal Editor**
- Editor-only plugin; it does not add runtime functionality

## Installation

1. In Fab, install or add Find in Materials to the target Unreal project and engine version.
2. If needed, enable **Find in Materials** from **Edit > Plugins**.
3. Restart the Editor if requested.
4. Open **Tools > Find in Materials**.

For a source installation, copy the `FindInMaterials` plugin folder from the [source repository](https://github.com/metyatech/FindInMaterials) into the target project's `Plugins` folder, then enable it in the Editor.

## Quick Start

1. Open **Tools > Find in Materials**.
2. Choose a search scope.
3. Enter a query.
4. Press **Enter** or click **Search**.
5. Double-click a matching child row.
6. For a Material or Material Function, the editor opens and focuses/selects the matching expression.
7. For a Material Instance, its editor opens.

## Search Scopes

### Open Materials

Searches currently open Materials, Material Functions, and Material Instances in the Editor.

### Entire Project

Searches project-owned content under `/Game` and content in enabled plugins loaded from the project that contain content. It intentionally excludes `/Engine` and Engine plugin content. “Entire Project” means project-owned content, not engine content.

### Folder

Searches a folder under `/Game` or `/Game/...`. Enter a path or use **Browse...** to choose a Content Browser folder. Folder scope does not accept `/Engine` paths.

## Query Syntax

Plain-text terms are case-insensitive substring matches. Multiple terms are combined with **AND**: every term must match. Use quotation marks to keep a phrase containing spaces together.

| Filter | Searches |
| --- | --- |
| `type:` | Expression or match type |
| `name:` | Node, display, or object name |
| `parameter:` | Parameter name |
| `value:` | Searchable, default, or reference value |
| `texture:` | Texture references |
| `function:` | Material Function references |
| `comment:` | Node comments |
| `path:` | Asset package path |

Examples:

```text
Roughness
texture:T_SearchGrid
parameter:Roughness
function:MF_ChannelTint
comment:"temporary surface"
parameter:Roughness texture:T_SearchGrid
type:TextureSample
name:SurfaceTexture
value:0.5
path:/Game/Materials
```

The combined example `parameter:Roughness texture:T_SearchGrid` returns matches that satisfy both filters.

## What Is Searched

- **Materials:** graph expressions and comments, expression type and name, parameter names, searchable/default values, texture and Material Function references, and asset/path context.
- **Material Functions:** graph expressions and comments with the same relevant graph metadata.
- **Material Instances:** scalar, vector, and texture parameter overrides, including parameter names, values, and texture references. Parent asset context is included when relevant to searchable values.

## Results and Navigation

Results are grouped by asset; child rows represent individual matches. The footer reports the number of assets scanned, matching assets, and matches. Results and progress update as assets are processed.

Double-click a Material match to open the Material Editor and focus/select the exact matching expression. A Material Function match opens its editor and focuses/selects the matching expression. A Material Instance match opens the Material Instance Editor; it does not jump to a graph node because a Material Instance does not own an editable material graph expression.

If the matching expression cannot be resolved or focused, the asset may still open and the panel reports **“Could not focus the matching node. The asset was opened.”**

## Content Browser Usage Search

Select a Texture or Material Function in the Content Browser, right-click it, and choose **Find Usages in Materials**. For a Texture, the search uses its texture reference; for a Material Function, it uses its function reference. Find in Materials opens, switches to **Entire Project**, and immediately runs the corresponding `texture:` or `function:` query.

## Performance and Search Behavior

Assets load asynchronously in bounded batches. Results and progress update incrementally, and you can cancel a search. Extracted metadata is cached for the current Editor session to speed up repeated searches; modifying relevant assets invalidates cached metadata. The first large Entire Project search may take longer while assets are loaded and metadata is collected.

Searching is read-only: it does not modify, dirty, or save assets. The session cache is not a persistent search index.

## Safety and Privacy

Find in Materials performs a local, read-only search in the Unreal Editor. The plugin includes no telemetry, analytics, cloud service, custom networking, or AI API or functionality.

## Limitations

Version **1.0.0** has these limits:

- Editor-only; no runtime functionality.
- Win64 only.
- No find/replace, graph editing, shader optimization, or shader analysis.
- No persistent search index or database, export or reporting, or cloud sync.
- No telemetry.
- Entire Project excludes Engine content and Engine plugin content.
- Material Instance results open the MI Editor and do not navigate to graph nodes.

## Troubleshooting

### Find in Materials is not in the Tools menu

Check that the plugin is enabled under **Edit > Plugins**. Restart the Editor if it requested a restart.

### Search returns no results

Verify the selected scope and query or filters. Entire Project searches project-owned content only. For Folder scope, check that the path is under `/Game`.

### Folder scope shows an error

Folder scope accepts only `/Game` or `/Game/...` paths. Use **Browse...** to choose a project content folder.

### A result opens the asset but does not focus a node

Exact graph navigation applies to Material and Material Function expressions, and the matching expression must still resolve in the editor. The asset may open with a status message if it cannot be focused. Material Instance results behave differently by design; see [Material Instance does not jump to a graph node](#material-instance-does-not-jump-to-a-graph-node).

### Material Instance does not jump to a graph node

This is expected. A Material Instance opens in the MI Editor. Its overrides are searchable, but it does not own editable graph expressions.

### Entire Project does not find an Engine Material

This is expected. Engine content and Engine plugin content are intentionally excluded.

### The first project-wide search is slower

The first search loads assets and extracts metadata. Repeated searches may use the Editor-session cache. Progress is shown and the search can be cancelled.

## Version History

### 1.0.0

Initial release with:

- Project-wide search for Materials and Material Functions, plus Material Instance override search.
- Material and Material Function graph-expression navigation.
- Texture and Material Function usage actions in the Content Browser.
- Open Materials, Entire Project, and Folder scopes.
- Asynchronous progress, cancellation, and an Editor-session metadata cache.
- Unreal Engine 5.6, 5.7, and 5.8 support on Win64.

## Links

- [Source repository](https://github.com/metyatech/FindInMaterials)
- [Support and troubleshooting](https://metyatech.github.io/unreal-plugin-docs/find-in-materials/#troubleshooting)
