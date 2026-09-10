# MRQ Preflight

**Catch broken Movie Render Queue structure before starting a long render.**

MRQ Preflight is a read-only Unreal Editor plugin for validating the current Movie Render Queue and the Level Sequences referenced by enabled jobs.

## Requirements

- Unreal Engine 5.6, 5.7, or 5.8
- Win64
- Unreal Editor
- Movie Render Pipeline

## Installation

1. Install the plugin for your Unreal Engine project.
2. Enable **MRQ Preflight** and restart the Editor when requested.
3. Configure Movie Render Queue normally.
4. Open **Tools > MRQ Preflight**.

## Usage

1. Open the Movie Render Queue you want to validate.
2. Open **Tools > MRQ Preflight**.
3. Click **Scan Current Queue**.
4. Review the Error, Warning, and Info findings.
5. Double-click a supported finding to open its existing Level Sequence asset.
6. Correct the project manually and scan again.

MRQ Preflight never starts a render and provides no automatic fix action.

## Validation Rules

MRQ Preflight checks enabled jobs for:

- Missing Level Sequence references
- Invalid Level Sequence references
- Missing Map references
- Invalid Map references
- No MRQ-generated renderable shots
- All generated shots disabled
- Camera Cuts with invalid binding IDs
- Camera Cuts with empty ranges
- Active Subsequence sections with missing sequence references
- Active Subsequence sections with empty outer ranges

Disabled jobs remain counted in the queue summary but are not diagnosed.

## Read-Only Behavior

Before Unreal Engine's MRQ shot-list generation is called, the current queue is copied to a transient in-memory queue.

The source queue, jobs, shots, Level Sequences, Maps, and project assets are not saved or intentionally modified by a scan.

## Scope

Version 1.0.0 intentionally does not:

- Start or test renders
- Automatically fix findings
- Open Maps automatically
- Predict visual rendering quality
- Diagnose World Partition behavior
- Predict GameMode or camera takeover
- Predict Temporal Sampling artifacts
- Diagnose GPU or VRAM requirements
- Validate Lumen, Nanite, or rendering CVars
- Estimate disk usage or rendering time
- Generate HTML, JSON, or CSV reports
- Provide telemetry, AI, cloud services, or network communication

## Troubleshooting

### MRQ Preflight is not visible in the Tools menu

Verify that the plugin is enabled and restart Unreal Editor.

### No jobs are scanned

MRQ Preflight validates the current Movie Render Queue. Add or load MRQ jobs first. Disabled jobs are not diagnosed.

### A Camera Cut binding is reported as invalid

The rule reports an invalid or empty binding ID in a Camera Cut used by an MRQ-generated shot. Open the referenced Level Sequence and repair the Camera Cut binding manually.

### A missing asset does not open when double-clicked

Missing assets cannot be opened. Double-click navigation is only performed for existing supported Level Sequence assets.

### Does scanning modify my Movie Render Queue?

No. MRQ shot analysis is executed against a transient queue copy rather than the source queue.

## Version

1.0.0
