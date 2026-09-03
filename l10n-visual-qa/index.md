# L10N Visual QA

**Capture every explicitly selected UMG widget across every language and
resolution, then review only what changed.** L10N Visual QA is a Win64 Unreal
Engine editor plugin for localization screenshot regression checks.

> **Remember:** Use **Editor mode** for everyday localization checks and
> **Runtime mode** for final verification or runtime-specific behavior.

## Overview

The plugin evaluates the combinations you put in a profile:

```text
UMG Widget × Culture × Resolution
```

It writes local PNG captures, metadata, JSON summaries, an offline HTML report,
and image differences produced by Unreal's `ScreenShotComparisonTools`.

## Requirements

- Unreal Engine **5.5, 5.6, 5.7, or 5.8**
- **Win64** development environment
- A real rendering RHI for capture; `-nullrhi` is not supported

The plugin has no third-party plugin dependency and ships no `Content/` assets.

## Installation

1. Copy the plugin folder into `<Project>/Plugins/L10NVisualQA`.
2. Regenerate project files if your IDE needs it.
3. Enable **L10N Visual QA** in the project plugin settings.
4. Add a profile to `<Project>/Config/L10NVisualQA/Profiles/`.
5. Open **Tools > L10N Visual QA**.

## Editor check

Editor mode runs inside the current Unreal Editor process. It uses the current
Editor World, Game Localization Preview, Slate, and `FWidgetRenderer` for
quick offscreen captures. Internally this remains the existing Fast mode and
uses the existing `fast` profile and CLI names. It restores the localization
preview enabled state, configured preview language, and current culture after
success, failure, Stop, or module shutdown. Editor mode does not execute
fixtures.

Use Editor mode for everyday localization UI iteration.

## Runtime check

Runtime Matrix starts a separate sibling `UnrealEditor.exe -game` process for
each culture, one culture at a time. Each process changes its requested
resolution sequentially, waits for the viewport to reach that size, warms up,
and captures each widget. The process uses `-RenderOffscreen` and never uses a
packaged executable or `-nullrhi`.

Each case has a `targets[].timeoutSeconds` timeout covering widget creation
through capture completion. A target timeout records `CaptureFailed`, invokes
fixture cleanup once with failure, and continues to the next case. The
separate `runtime.cultureProcessTimeoutSeconds` limit covers the complete
culture worker process. On user cancellation, the child process tree is
terminated, waited on, and its handles and pipes are closed; unfinished cases
in the current culture and all cases in later cultures are recorded as
`Cancelled`. A culture process timeout or abnormal worker exit records all
unexecuted cases as `RunnerError` with the infrastructure cause. Messages for
`CaptureFailed`, `Cancelled`, and `RunnerError` cases are retained in the
summary and report, and cancellation or culture timeout saves the started
culture's `logs/runtime-<culture>.log`.

## Creating a Profile

Save a file such as `ReleaseUI.json`. The basename and `name` must match
exactly. This example is a complete profile:

```json
{
  "schemaVersion": 1,
  "name": "ReleaseUI",
  "map": "/Game/Maps/L10NQA",
  "targets": [
    {
      "id": "MainMenu",
      "widgetClass": "/Game/UI/WBP_MainMenu.WBP_MainMenu_C",
      "fixtureClass": null,
      "enabled": true,
      "warmupFrames": 3,
      "timeoutSeconds": 15
    }
  ],
  "cultures": ["en", "ja", "de"],
  "resolutions": [
    { "width": 1280, "height": 720 },
    { "width": 1920, "height": 1080 }
  ],
  "comparison": { "tolerance": "IgnoreAntiAliasing" },
  "fast": { "enabled": true },
  "runtime": {
    "enabled": true,
    "cultureProcessTimeoutSeconds": 120,
    "resolutionSettleFrames": 3
  }
}
```

## Profile Reference

**Strict validation happens before a run.** Unknown fields at every level are
errors, so a misspelled setting cannot silently change a QA run.

- `schemaVersion`: exactly `1`.
- `name`: `[A-Za-z0-9_]{1,64}` and equal to the filename basename.
- `targets`: 1–100 entries; enabled combinations must remain at or below 4096.
- `id`: unique case-insensitively and `[A-Za-z0-9_]{1,64}`.
- `widgetClass`: a concrete `UUserWidget` subclass, checked before capture.
- `fixtureClass`: `null` or a concrete `UL10NVisualQAFixture` subclass.
- `warmupFrames`: 0–600; `timeoutSeconds`: 1–300.
- `cultures`: 1–64 valid `FInternationalization` cultures, without duplicate
  names or surrounding whitespace.
- `resolutions`: 1–32 unique entries, 64–8192 per axis, at most 33,177,600
  pixels.
- `comparison.tolerance`: `IgnoreAntiAliasing` or `Strict`.
- `runtime.map`: required and must be a valid long package name when Runtime
  Matrix is enabled.
- `runtime.cultureProcessTimeoutSeconds`: 10–1800.
- `runtime.resolutionSettleFrames`: 1–60.

## Runtime Fixture

`UL10NVisualQAFixture` is the single customer-facing Blueprint extension point.
Runtime order is:

```text
create fixture → set World context → BeforeWidgetCreated
→ create widget → AfterWidgetCreated → add to viewport
→ warmup → capture → remove widget → AfterCapture → cleanup
```

`AfterCapture(..., false)` is called when capture fails. Fixtures are ignored
by Fast Matrix. A fixture is a **QA-only asset**: exclude it from Shipping
cook/package rules because it is not gameplay content.

## Baseline Workflow

The first successful capture of a combination is `NoBaseline`. No baseline
exists yet: review the Current image, then use **Approve Selected** or the
confirmed **Approve All** action in the editor dashboard to establish it. Only
`Changed`, `NoBaseline`, and `InvalidBaseline` cases are
eligible. Approval preflights and stages the current PNG and `.meta.json`,
backs up any existing pair, commits both in the same destination directory,
and restores the old pair on failure. It never leaves one file from the old
pair and one from the new pair. If rollback itself fails, recoverable
`.backup-*` files remain and their PNG and metadata paths are reported in the
error. After success, cases are compared again and `summary.json` and
`report.html` are regenerated.

Fast and Runtime baselines are intentionally separate, and UE minor versions
are separate too:

```text
<Project>/Test/L10NVisualQA/Baselines/UE<major>.<minor>/<Mode>/<Target>/<Culture>/<Width>x<Height>.png
```

## Editor Dashboard

Open **Tools > L10N Visual QA**. The dashboard provides:

- profile selection and validation errors;
- **Reload Profiles** and **Open Profiles Folder**;
- an **Editor / Runtime** mode selector, **Run Check**, and **Stop**;
- Editor mode selected by default for everyday checks;
- a `Results: Editor` or `Results: Runtime` label that identifies the
  displayed result independently from the next selected mode; a valid profile
  without a matching summary shows `Results: Not run for this profile`;
- culture × resolution result status using both text and symbols;
- enabled Target selection list on the left; drag the divider to resize the
  Target list while keeping the Matrix and previews usable;
- clickable Culture × Resolution cells on the right with explicit `[PASS]`,
  `[CHANGED]`, `[NO BASELINE]`, `[CAPTURE FAILED]`, `[INVALID BASELINE]`,
  `[CANCELLED]`, `[RUNNER ERROR]`, and `[NOT RUN]` labels;
- simultaneous Baseline, Current, and Difference Image previews with image paths; a No
  Baseline case shows `No baseline yet` and `No difference image — a baseline is
  required first`; click an available image to open the Image Viewer with Fit
  Window, Actual Size (100%), zoom, pan, view selection, Changed Areas, raw
  Difference Image, and Alternate Baseline / Current; the Alternate
  Baseline / Current interval is adjustable from 0.10 to 5.00 seconds using
  **Switch every**;
- selected Target, Culture, Resolution, status, global difference, maximum
  local difference, and message;
- **Approve Selected** and confirmation-protected **Approve All**.

Missing preview files show an empty image and explain why they are unavailable.
No baseline means: `No baseline exists yet. Review Current and approve it to
establish the baseline.` While a run is
active, profile changes, reload, run, and approval controls are disabled;
**Stop** remains available. The dashboard refreshes approximately every 0.1
seconds so intermediate matrix results are visible without rebuilding the
matrix structure. The dashboard does not edit profile JSON; edit the file and
reload it instead.

## Command Line / CI

Use the commandlet for Runtime Matrix from the same project and plugin
installation:

```powershell
UnrealEditor-Cmd.exe <Project>.uproject `
  -run=L10NVisualQA `
  -Profile=ReleaseUI `
  -Mode=Runtime `
  -AllowNoBaseline `
  -unattended `
  -nop4 `
  -RenderOffscreen
```

The internal Fast mode must run in `UnrealEditor.exe`, because it uses Slate
and `FWidgetRenderer`. Use the editor bootstrap flag:

```powershell
UnrealEditor.exe <Project>.uproject `
  -L10NVisualQAFast `
  -Profile=ReleaseUI `
  -Mode=Fast `
  -AllowNoBaseline `
  -unattended `
  -nop4 `
  -RenderOffscreen
```

Parameters:

- `-Profile=<profile basename>` — selects a basename only; separators and
  traversal are rejected.
- `-Mode=Runtime` — selects Runtime Matrix for the commandlet.
- `-L10NVisualQAFast` with `-Mode=Fast` — selects Fast Matrix for the editor
  bootstrap.
- `-AllowNoBaseline` — allows `NoBaseline` cases to return success.

Exit codes:

- `0`: every case is `Pass`, or `NoBaseline` with `-AllowNoBaseline`.
- `1`: `Changed`, `NoBaseline`, `CaptureFailed`, or `InvalidBaseline` remains.
- `2`: invalid profile, startup, worker, orchestration, rendering, or internal
  failure. `-nullrhi` always returns `2`.

## Output Files

Runs are written below:

```text
<Project>/Saved/L10NVisualQA/Runs/<RunId>/
├── summary.json
├── report.html
├── captures/Fast/ or captures/Runtime/
├── diffs/
├── metadata/
└── logs/
```

`RunId` is `yyyyMMddTHHmmssfffZ-<8 hex chars>`. `summary.json` contains run
metadata, totals, cases, warnings, and errors. `report.html` is a standalone
offline file with escaped values, filtering, and relative current/baseline/diff
links.

## Limitations

Version 0.1.0 does not translate text, call AI APIs, edit PO/CSV, scan missing
or hardcoded translations, scan glyphs, discover every project widget, judge
overlap or visual beauty, automate gameplay/CommonUI, test packaged EXEs,
provide pseudo-localization, support macOS/Linux, use cloud/telemetry/
analytics, modify user assets, operate source control, auto-approve baselines,
or run Runtime workers in parallel.

## Privacy

There is **no telemetry, analytics, cloud service, or AI API**. Screenshots,
metadata, reports, and logs stay local to the project unless you copy them
elsewhere. The plugin does not upload captures or inspect a remote service.

## Troubleshooting

### Profile is not listed

Confirm the file is under `Config/L10NVisualQA/Profiles/`, ends in `.json`,
uses a safe basename, and has the same `name`. Click **Reload Profiles**.
Unknown fields, invalid cultures, or a disabled Fast and Runtime pair also
make a profile invalid.

### Widget class load failure

Check that the soft class path names the generated class, for example
`/Game/UI/WBP_MainMenu.WBP_MainMenu_C`, and that the class is a concrete
`UUserWidget` subclass available to the selected project/map.

### Fixture class load failure

Check that the class is a concrete subclass of `UL10NVisualQAFixture`. Fixtures
run only in Runtime Matrix; they are not needed for Fast Matrix.

### Runtime worker starts but capture fails

Open the run's `logs/runtime-<culture>.log`, then check the PNG and metadata
paths in the summary. Confirm a real RHI, a valid map, a viewport, and a valid
widget class. `-nullrhi` cannot capture widgets.

### Runtime worker timeout

Confirm the map loads without prompts, lower warmup/settle settings only when
the project can safely do so, and inspect the culture log. The runner
terminates the child tree on culture process timeout and records unfinished
cases as `RunnerError`. A `targets[].timeoutSeconds` timeout is a case-level
`CaptureFailed` result and still advances to the next case.

### Unexpected `Changed` results

Check culture, resolution, UE minor baseline folder, Fast versus Runtime mode,
font availability, and the selected comparison tolerance. Review the current,
baseline, and diff links before approving; baseline approval is intentionally
manual.

### RHI warning

An RHI difference is reported as a warning and does not alone make a baseline
invalid. For stable pixel output, compare on the same RHI and machine class.

### `-nullrhi` cannot be used

This is intentional. Widget rendering and pixel comparison require a real
rendering RHI. Remove `-nullrhi` and use `-RenderOffscreen` instead.

### A worker remains after abnormal OS termination

Close the owning editor process, verify no `UnrealEditor.exe` process still has
the project command line, and remove only the stale run directory after
preserving any evidence needed for diagnosis. Normal Stop/timeout paths wait
for the child and its tree before returning.

### Fixture Blueprint packaging/cook warning

Blueprint fixtures are development QA assets. Exclude them from Shipping cook
and package rules; do not add them to a production content dependency merely
to make Runtime Matrix profiles work.

## Version History

### 0.1.0

Initial release for Unreal Engine 5.5–5.8 on Win64 with Fast Matrix, Runtime
Matrix, strict profiles, Epic image comparison, manual baseline approval,
offline JSON/HTML reports, and CI commandlet support.

## Links

- [Source repository](https://github.com/metyatech/L10NVisualQAPlugin)
- [Support](https://metyatech.github.io/unreal-plugin-docs/l10n-visual-qa/#troubleshooting)
