---
---

# Server Manage Tool

[Back to plugin list](../)

**Server Manage Tool configures map-specific addresses and launches the complete configured local server group during PIE only after every required UDP port passes preflight.** It does not build, deploy, or host a production dedicated server.

Version: **1.1.0**

## Overview

Server Manage Tool provides map-specific server address settings, a Server Mode menu in the Unreal Editor Play menu, and runtime Blueprint functions for address lookup and server exit requests.

The plugin contains three modules:

- `ServerModePlayMenu`: editor menu, Local Launch, and managed-process cleanup
- `ServerInfoSettingsModule`: map and server-address configuration
- `ServerManageLibrary`: runtime Blueprint functions

## Requirements

- An Unreal Engine project using a compatible plugin package.
- A saved `.uproject` and maps referenced by full package names.
- An Unreal Editor executable available for Local Launch.
- Project-side dedicated-server behavior that already supports `-server`.

The package was verified with a Blueprint-only UE 5.8 Win64 host. Real game-project Development/Shipping integration and packaged executable behavior remain project-specific and unverified here.

## Installation

### From Fab

1. Acquire **Server Manage Tool** from Fab.
2. Install it for the project's Unreal Engine version.
3. Open the project.
4. Open `Edit > Plugins`.
5. Enable **Server Manage Tool**.
6. Restart the editor when requested.

### From source

Place the plugin at:

```text
<Project>/Plugins/ServerManageTool
```

Regenerate project files or use the project's normal source build workflow, then restart the editor after module changes.

## Quick Start

1. Open `Project Settings > Project > Servers`.
2. Add one `Server List` entry per server map.
3. Use each map's full package name, such as `/Game/Maps/Lobby`.
4. Enter the `Server Address` used by Project Setting mode.
5. Open the editor Play menu and choose **Server Mode**.
6. Select **Project Setting** or **Local Launch**.
7. Call **Get Server Address** from the project's own connection flow.
8. Call **Request Server Exit** only from the dedicated server process when it should stop.

The plugin resolves an address; it does not connect the client automatically.

## Features

- Map-specific server-address configuration.
- Project Settings integration.
- Project Setting and Local Launch modes.
- Local dedicated-server process launch during PIE.
- All-port UDP preflight before Local Launch.
- All-or-nothing launch when a configured port is unavailable.
- Shifted-port runtime detection through expected-port validation.
- PIE-end and module-shutdown managed-server cleanup.
- Blueprint functions `GetServerAddress(Map)` and `RequestServerExit()`.

## Configuration

`Project Settings > Project > Servers` contains `Server List`.

| Field | Meaning |
| --- | --- |
| `Map Name` | Exact full Unreal package name, such as `/Game/Maps/Lobby`. |
| `Server Address` | Address returned in Project Setting mode. |

In Local Launch mode, every configured entry is launched. Ports begin at fixed base port `7777` and follow list order:

```text
Entry 0: 7777
Entry 1: 7778
Entry 2: 7779
```

Reordering entries changes their Local Launch ports. Before any process starts, all assigned UDP ports are checked. If one is unavailable, the group is not partially launched.

## Blueprint API

### Get Server Address

`GetServerAddress(Map)` finds an exact configured map entry. Project Setting mode returns the configured address. Local Launch returns `127.0.0.1:<assigned port>`. A null map or missing exact entry returns an empty string and logs an error.

### Request Server Exit

`RequestServerExit()` requests the current process to exit its main loop. Use it only on the dedicated-server side; do not call it casually from a client. The plugin does not guarantee a particular operating-system exit code.

## Local Launch lifecycle

At PIE begin, the plugin reads all entries, assigns ports, performs UDP preflight, launches the full group when preflight passes, and validates each server's actual bound port. At PIE end, managed server processes are cleaned up. Module shutdown also clears managed server ownership.

The Output Log markers are:

```text
SMT_PORT_PREFLIGHT_PASSED
SMT_PORT_PREFLIGHT_FAILED
SMT_SERVER_PROCESS_LAUNCH_FAILED
SMT_PORT_VALIDATION_PASSED
SMT_PORT_VALIDATION_FAILED
```

There is no health check, retry, or automatic restart. Do not rely on a specific operating-system exit code after shifted-port detection.

## Troubleshooting

### Empty address

Check that the map is not null and that `Map Name` exactly matches its full package name. Use `/Game/Maps/Lobby`, not only `Lobby`.

### No local servers

Check Local Launch mode, the project executable, map package names, project `-server` support, and the Output Log. A preflight failure prevents partial launch and reports the unavailable port.

### Port conflict

Resolve the unavailable port named by `SMT_PORT_PREFLIGHT_FAILED`, then retry PIE. The plugin does not promise an automatic port shift for the managed group.

### Shifted port

Use `SMT_PORT_VALIDATION_FAILED` to compare expected and actual ports. The plugin requests clean server exit; the exact OS exit status is not a contract.

### Client connection

The plugin does not connect the client automatically. Pass `GetServerAddress(Map)` to the project's own connection or travel flow.

For reproducible problems, open an issue in the [ServerManageToolPlugin repository](https://github.com/metyatech/ServerManageToolPlugin/issues) with sanitized logs, engine version, mode, list order, and the relevant markers.

## Limitations

- Local Launch is Editor/PIE only.
- The base port is fixed at `7777` and assignment follows Server List order.
- Local Launch launches all configured entries.
- The project must already support `-server`.
- The plugin does not build, package, deploy, or host a production server.
- The plugin does not connect clients automatically.
- There is no health check, retry, or automatic restart.
- Real game-project Development/Shipping integration and packaged executable behavior remain unverified.

## Version history

### 1.1.0

- UDP preflight.
- Atomic all-or-nothing launch.
- Shifted-port validation.
- Editor delegate cleanup.
- Module-shutdown server cleanup.
- Fab Config/Content structure and packaged documentation.

### 1.0.0

- Initial settings.
- Project Setting and Local Launch modes.
- Initial Blueprint API.
