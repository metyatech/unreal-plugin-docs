---
---

# Server Manage Tool

[Back to plugin list](../)

## Overview

Server Manage Tool provides map-specific server address settings and an Unreal Editor workflow for launching local dedicated-server processes during Play In Editor.

The plugin contains three modules:

- `ServerModePlayMenu`: editor menu and local-server process management
- `ServerInfoSettingsModule`: map and server-address configuration
- `ServerManageLibrary`: runtime Blueprint functions

## Requirements

- An Unreal Engine project using a plugin package compatible with the project's engine version
- A saved `.uproject` file
- Saved maps referenced by their full Unreal package names
- The Unreal Editor executable must be available when using Local Launch
- Dedicated-server behavior must already be implemented by the project

The plugin does not package, deploy, configure, or host a production dedicated server.

## Installation

### From Fab

1. Acquire **Server Manage Tool** from Fab.
2. Install the plugin for the appropriate Unreal Engine version.
3. Open the project.
4. Open **Edit > Plugins**.
5. Enable **Server Manage Tool**.
6. Restart the editor when requested.

### From Source

Place the repository at:

```text
<Project>/Plugins/ServerManageTool
```

Then regenerate project files or open the project and accept the rebuild prompt.

## Quick Start

1. Open **Project Settings**.
2. Open **Project > Servers**.
3. Add one entry to `Server List` for each server map.
4. Set `Map Name` to the map's full package name, for example:

```text
/Game/Maps/Lobby
```

5. Set `Server Address` to the address the client should use outside Local Launch, for example:

```text
example.com:7777
```

6. Open the Play menu in the Unreal Editor.
7. Open **Server Mode**.
8. Select one of the following:

   * **Project Setting**: use the configured `Server Address`.
   * **Local Launch**: launch local editor server processes and use loopback addresses.

9. Call **Get Server Address** with the map asset.
10. Pass the returned string to the project's own connection or client-travel logic.

The plugin resolves an address. It does not automatically connect the client.

## Features

* Per-map server-address configuration
* Project Settings integration
* Editor Play menu integration
* Project Setting mode
* Local Launch mode
* Automatic local server-process startup at PIE begin
* Automatic local server-process termination at PIE end
* Blueprint-accessible server-address lookup
* Blueprint-accessible server-exit request

## Settings / Blueprint Nodes

### Server List

`Project Settings > Project > Servers` contains `Server List`.

Each entry contains:

| Field            | Purpose                                                                                             |
| ---------------- | --------------------------------------------------------------------------------------------------- |
| `Map Name`       | Full Unreal package name of the map, such as `/Game/Maps/Lobby`.                                    |
| `Server Address` | Address returned outside Local Launch. The plugin stores and returns this as an unvalidated string. |

### Server Mode

The editor Play menu contains **Server Mode**.

#### Project Setting

`Get Server Address` returns the `Server Address` configured for the requested map.

#### Local Launch

At PIE begin, the plugin starts one Unreal Editor server process for every entry in `Server List`.

Ports are assigned from the list order:

```text
First entry:  7777
Second entry: 7778
Third entry:  7779
```

In Local Launch mode, `Get Server Address` returns:

```text
127.0.0.1:<assigned port>
```

The local processes are terminated when PIE ends.

### Get Server Address

Blueprint function:

```text
GetServerAddress(Map)
```

Behavior:

* Uses the map's full package name.
* Finds an exact matching `Map Name` entry.
* Returns the configured address in Project Setting mode.
* Returns the loopback address and assigned local port in Local Launch mode.
* Returns an empty string and logs an error when the map is null or no matching entry exists.

### Request Server Exit

Blueprint function:

```text
RequestServerExit()
```

Requests the current process to exit its main loop.

## Limitations

* Local Launch is an editor and PIE feature.
* Local Launch starts a server process for every configured server entry, not only the currently selected map.
* The base port is fixed at `7777`.
* Additional ports are assigned from `Server List` order.
* Reordering entries changes their Local Launch ports.
* No port-availability check is performed.
* No server-readiness or health check is performed.
* No retry or automatic restart is performed.
* No automatic client travel or connection is performed.
* No production-server deployment is performed.
* No remote-server lifecycle management is performed.
* `Server Address` is not parsed or validated.
* A missing map configuration returns an empty string.
* Local server processes are terminated when PIE ends.

## Troubleshooting

### Get Server Address returns an empty string

Verify that:

* The map input is not null.
* `Map Name` uses the full package name.
* The configured value exactly matches `Map.GetLongPackageName()`.
* The entry exists in `Project Settings > Project > Servers`.

Use a value such as:

```text
/Game/Maps/Lobby
```

Do not use only:

```text
Lobby
```

### A local server does not start

Verify that:

* The `.uproject` file exists.
* The current Unreal Editor executable exists.
* The configured map package name is valid.
* The assigned port is not already in use.
* The project supports running with `-server`.
* The Output Log does not contain a process-launch error.

### The client does not connect automatically

This is expected. Call `GetServerAddress`, then pass its result to the project's own client-travel or connection implementation.

### Local ports changed

Local ports are derived from `Server List` order. Restore the original order or update the project logic that depends on the assigned ports.

For reproducible problems, open an issue in the [ServerManageToolPlugin repository](https://github.com/metyatech/ServerManageToolPlugin/issues).

## Version History

### 1.0

* Initial map-specific server settings
* Project Setting and Local Launch modes
* Local dedicated-server process startup during PIE
* `GetServerAddress`
* `RequestServerExit`
