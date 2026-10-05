# Recently Saved Assets

Find project assets you saved or imported recently. **Last Saved is the asset package file's filesystem modification time**, and the list refreshes only when you open the tab or choose Refresh.

## Overview

Recently Saved Assets is an Editor-only Unreal Engine plugin. It lists assets under `/Game` newest first, with search, time-window filters, sorting, and standard Unreal Editor navigation.

## Requirements

- Unreal Engine **5.6, 5.7, or 5.8**
- **Win64**
- Unreal Editor
- Editor-only; no runtime component

## Installation

1. Install or add **Recently Saved Assets** to your Unreal project through Fab.
2. Enable **Recently Saved Assets** in **Edit > Plugins** if needed.
3. Restart the Editor if requested.
4. Open **Tools > Recently Saved Assets**.

## Quick Start

1. Open **Tools > Recently Saved Assets**.
2. Search for an asset name, type, or package path, or choose a time window.
3. Review the list, which defaults to **Newest First**.
4. Choose **Refresh** when you want to scan the project again.

## Search

Search matches a case-insensitive substring in the asset name, asset type, or package path. Leading and trailing spaces are ignored. An empty search shows all assets allowed by the other filters.

## Time Window

Choose **All**, **Last Hour**, **Last 24 Hours**, **Last 7 Days**, or **Last 30 Days**. Time windows use the package timestamp and include assets exactly on the selected boundary.

## Sorting

Choose **Newest First** or **Oldest First**. Equal timestamps are ordered by a stable asset identifier.

## Open / Sync navigation

Double-click an asset row to open it with its standard Unreal Editor asset editor. Right-click a row and choose **Sync in Content Browser** to select that asset in the standard Content Browser.

## Last Saved semantics

**Last Saved means the filesystem modification time of the asset package file.** It is not a source-control commit time. Git or Perforce syncs, file copies, and similar file operations can change the timestamp. Unsaved Editor changes do not appear until the package is saved. The time shown in the list is local time.

## Refresh behavior

The plugin scans `/Game` when its tab is created and when you press **Refresh**. Search, time-window, and sort changes reuse the cached scan. It does not run a background filesystem watcher or polling loop.

## Safety and Privacy

Recently Saved Assets is read-only: it does not save, modify, rename, move, or delete project assets. It has no network or cloud service, telemetry, or third-party dependency.

## Limitations

- It lists project assets under `/Game` only.
- World Partition external actor and external object assets under `/Game/__ExternalActors__/` and `/Game/__ExternalObjects__/` are not shown.
- It uses the package file's filesystem modification time, which can change during sync or copy operations.
- It is Editor-only and supports Win64 with Unreal Engine 5.6, 5.7, and 5.8.
- It does not monitor the filesystem in the background.

## Troubleshooting

### Recently Saved Assets is not in the Tools menu

Confirm the plugin is enabled in **Edit > Plugins**. Restart the Editor if it requested a restart.

### The list is empty

Confirm the project has saved assets under `/Game`, then choose **Refresh**. Assets without a resolvable package file timestamp are omitted.

### An asset does not appear in a time window

The filter uses the package file's filesystem modification time. Choose **All** to inspect the complete timestamped list, then check whether a source-control sync or file copy changed the timestamp.

### A recent unsaved edit is missing

Save the package first. Unsaved Editor changes do not change the package file's Last Saved timestamp.

## Version History

### 1.0.0

Initial release with `/Game` package filesystem timestamp listing, search, time-window filters, sorting, standard asset-editor navigation, Content Browser synchronization, and manual refresh. Supports Unreal Engine 5.6, 5.7, and 5.8 on Win64.

## Support

- [Support and troubleshooting](https://metyatech.github.io/unreal-plugin-docs/recently-saved-assets/#troubleshooting)
