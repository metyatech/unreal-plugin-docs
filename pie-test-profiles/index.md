# PIE Test Profiles

**Save reusable PIE multiplayer and network test setups, then run them without overwriting your normal Play settings.**

PIE Test Profiles is an Unreal Editor plugin for saving named Play In Editor configurations and launching PIE with those settings applied transiently for that session.

## Requirements

- Unreal Engine 5.6, 5.7, or 5.8
- Win64
- Unreal Editor

## Installation

1. Install the plugin for your Unreal Engine project.
2. Enable **PIE Test Profiles** and restart the Editor if requested.
3. Open **Tools > PIE Test Profiles**.

## Quick Start

1. Open **Tools > PIE Test Profiles**.
2. Select one of the included profiles or choose **New from Current**.
3. Edit the profile in the Details panel.
4. Click **Run Profile**.
5. Stop PIE normally when the test is complete.

## Included Profiles

### Solo

- Standalone
- 1 Player
- Run Under One Process
- No separate server
- Network Emulation off

### 2P Listen

- Listen Server
- 2 Players
- Run Under One Process
- No separate server
- Network Emulation off

### 2P Bad Network

- Listen Server
- 2 Players
- Run Under One Process
- Network Emulation on
- Target: Everyone
- Incoming latency: 80–120 ms
- Incoming packet loss: 2%
- Outgoing latency: 80–120 ms
- Outgoing packet loss: 2%

## Profile Settings

- Launch Mode
  - Selected Viewport
  - New Window
- Net Mode
  - Standalone
  - Listen Server
  - Play as Client
- Players: 1–10
- Run Under One Process
- Launch Separate Server
- Network Emulation
- Network Target
  - Server Only
  - Clients Only
  - Everyone
- Incoming Min / Max Latency
- Incoming Packet Loss
- Outgoing Min / Max Latency
- Outgoing Packet Loss

## Profile Management

- New from Current
- Duplicate
- Delete
- Rename/edit profile fields
- Run Profile

Profiles are stored per-project and per-user.

Deleting every profile must not cause the built-in default profiles to reappear automatically.

## How Running a Profile Works

1. PIE Test Profiles reads Unreal Editor's current Play settings.
2. It creates a transient copy.
3. Only the settings owned by the selected profile are changed on that transient copy.
4. The transient settings are supplied to Unreal Engine's normal PIE request.
5. Unreal Editor starts PIE through its standard Play In Editor flow.

## Persistent Settings Safety

- Running a profile does not overwrite the persistent `ULevelEditorPlaySettings`.
- It does not modify the user's normal Play settings and then restore them afterward.
- PIE Test Profiles stores only its own profile data persistently.
- Profile data uses per-project/per-user Editor settings.

## Network Emulation

The plugin uses Unreal Engine's existing PIE Network Emulation settings.

It does not implement:

- its own network stack
- custom sockets
- custom packet simulation

## Validation

- Profile name is required.
- Profile names must be unique, case-insensitively after trimming whitespace.
- Player count must be 1–10.
- When Network Emulation is enabled:
  - latency must be 0–5000 ms
  - Min must be <= Max
  - packet loss must be 0–100%

## Scope

Version 1.0.0 intentionally does not provide:

- runtime functionality
- map selection
- PIE window arrangement
- audio settings profiles
- gamepad routing
- HMD / VR settings
- packaging/build launching
- profile import/export
- profile cloud sync
- Online Session management
- Steam integration
- custom networking
- automatic modification of private Unreal Engine settings
- telemetry
- cloud services

## Troubleshooting

### PIE Test Profiles is not visible in the Tools menu

Confirm the plugin is enabled and restart the Editor if needed.

### Run Profile is disabled

A profile must be valid, and another PIE session must not already be active or queued.

### Selected Viewport cannot run

Selected Viewport requires an active Level Editor viewport. Use New Window if no viewport is available.

### My normal Play settings changed

PIE Test Profiles should not persistently modify the global Play settings during a profile run. The profile itself is stored separately in per-project/per-user settings.

## Privacy

- No telemetry
- No cloud service
- No network calls performed by the plugin itself
- No third-party runtime dependency

PIE itself may perform networking according to the user's Unreal project/test configuration; this privacy statement refers to PIE Test Profiles itself.

## Version

1.0.0
