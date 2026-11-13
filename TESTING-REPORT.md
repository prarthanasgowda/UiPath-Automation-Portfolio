# UiPath Portfolio Testing Report

**Date:** 2026-11-13  
**Scope:** Local inspection and attempted runtime validation of all five existing projects  
**Overall status:** NOT TESTED — runtime execution is blocked by Studio licensing/Robot IPC startup errors.

## Follow-up environment diagnosis

### Licensing and connection follow-up

UiPath Assistant was launched through the installed application and visibly reported `Status: Connected, Licensed`. UiPath Studio visibly reported `Community License` and showed the signed-in user. Robot logs showed successful Orchestrator connection, `StartService Status:OK`, heartbeat responses, and SignalR negotiation. Authentication was completed by the existing local UiPath session; no credentials were entered or stored.

The actual Studio installation is present and complete enough to include the supported command-line application:

- `C:\Program Files\UiPathPlatform\Studio\26.10.0-cloud.25710\UiPath.Studio.exe`
- `C:\Program Files\UiPathPlatform\Studio\26.10.0-cloud.25710\UiPath.Studio.CommandLine.exe`
- Matching binaries also exist under `26.10.0-cloud.25715`.

The earlier Robot packaging error was caused by Robot looking for the CLI under its own directory (`...Robot\\26.10.0-cloud.25715\\UiPath.Studio.CommandLine.exe`) instead of the actual Studio directory. The Studio CLI was executed directly and responded normally, so the CLI is not absent from the installation.

Studio startup logs then exposed the current blocker:

```text
Error: No license exist for this installation
Acquire orchestrator license error: ... GetRobotInfoCore timed out.
```

`StudiomodeSvc`, `UiPath RobotJS Service`, and `UiPath.Connected.Updater.Service` are running. No destructive repair was attempted. The safe next procedure is to verify the signed-in UiPath account/license in Studio or Assistant, confirm the Robot service’s local IPC health, and use the official UiPath repair/update mechanism only if the license/account and service checks do not resolve the timeout. Reinstalling, copying executables between folders, deleting licensing state, or changing service configuration requires explicit approval and is not justified yet.

## Environment

| Component | Result |
|---|---|
| UiPath Studio | Installed: `26.10.0-cloud.25710` and `26.10.0-cloud.25715` binaries detected |
| UiRobot | Installed: `26.10.0-cloud.25715`; help command succeeded |
| UiPath Executor | Installed under the Robot installation |
| UiPath Studio CLI | Not available at the Robot-expected path; Robot `pack` failed with `Could not find Studio Command-Line application!` |
| UiPath Connected Installer CLI | Available: `26.11.92` |
| Browser bridge | UiPath browser bridge/native messaging processes detected |
| Chrome | Chrome app registered; executable was not returned by PATH lookup |
| Excel/Word/PowerPoint | Office app registrations were visible to the desktop inventory; standard executable/registry path lookup did not return executable paths |

## Cached activity packages

Detected package families and local versions:

- `UiPath.System.Activities`: `25.10.5`, `26.8.2`, `26.10.0`
- `UiPath.Excel.Activities`: `3.5.1`, `3.6.0-preview`
- `UiPath.Word.Activities`: `2.5.1`, `2.6.0-preview`
- `UiPath.Presentations.Activities`: `2.6.0-preview`
- `UiPath.UIAutomation.Activities`: `26.10.3`, `26.10.5`

The five portfolio `project.json` files were updated to use these locally available package families rather than the older 2023-era versions.

## Activity-resolution investigation

The local NuGet restore is present: each project has a `.local` restore folder and successful NuGet resolution metadata. The failure is not simply “packages absent”; the XAML was serialized with guessed or obsolete activity mappings.

Verified package evidence:

| Existing XAML type | Finding in installed 26.10 packages | Repair status |
|---|---|---|
| `http://schemas.uipath.com/workflow/activities/web:OpenBrowser` | Runtime assembly exposes classic `UiPath.Core.Activities.OpenBrowser`; the `/web` mapping is invalid | Browser opening namespace corrected to the generic UiPath activity namespace |
| `GetBrowserData` | No matching runtime activity type was found in the installed UI Automation runtime package | Requires Studio-authored modern extraction activity; not guessed |
| `ExcelProcessScope` | Excel 3.5.1 package contains modern `ExcelProcessScopeX` metadata | Excel scope tag corrected to `ExcelProcessScopeX` |
| `WordApplicationScope` | Word package documents the supported Word Application Scope / Use Word File family | Existing type still requires Studio package load validation |
| `CreatePresentation` | Presentations package exposes `PptDocumentInsertSlide` and `PptDocumentAddTextToSlide`; `CreatePresentation` is not present | Requires Studio-authored replacement; not guessed |
| `LogMessage` | System activity is valid only when the System package assembly context is loaded | Integrated workflow still requires packaged Studio execution |

The official Studio CLI now reaches workflow analysis and reports the exact remaining browser issue: `Cannot create unknown type ... GetBrowserData`. This confirms the namespace repair was effective and that the next failure is a genuinely unsupported activity type.

## Static inspection

| Project | JSON | XAML/XML | Dependency declaration | Runtime |
|---|---:|---:|---:|---:|
| 01 Browser Automation | PASS | PASS | PASS | NOT TESTED |
| 02 Excel Automation | PASS | PASS | PASS | NOT TESTED |
| 03 Word Report Generator | PASS | PASS | PASS | NOT TESTED |
| 04 PowerPoint Automation | PASS | PASS | PASS | NOT TESTED |
| 05 Integrated Office Automation | PASS | PASS | PASS | NOT TESTED |

All 13 XAML files parsed as XML. All five `project.json` files parsed as JSON. The integrated project’s child workflow paths point to its `Workflows` folder, and its default output folder is `Output`.

## Runtime attempts

After licensing became available, Robot execution was attempted directly with `UiRobot execute --file <project.json>`. These were genuine execution attempts, not static parsing. The first error was corrected across the portfolio by defining the missing `sap2010` XAML namespace. The next execution pass reached activity loading and exposed these implementation/package-loading errors:

| Project | Runtime result | First observed error |
|---|---|---|
| 01 Browser Automation | FAIL | `Cannot create unknown type ... OpenBrowser` |
| 02 Excel Automation | FAIL | `Cannot create unknown type ... ExcelProcessScope` |
| 03 Word Report Generator | FAIL | `Cannot create unknown type ... WordApplicationScope` |
| 04 PowerPoint Automation | FAIL | `Cannot create unknown type ... CreatePresentation` |
| 05 Integrated Office Automation | FAIL | `Cannot create unknown type ... LogMessage` |

These errors indicate that direct Robot execution of the source project metadata is not loading the designer package assemblies. A supported Studio CLI publish was attempted using the actual Studio CLI, but it stalled during project loading and produced no package artifact; that CLI process was stopped without changing the projects.

Earlier, UiRobot `pack` was attempted for every project. Those attempts failed before workflow compilation because the Robot installation could not locate `UiPath.Studio.CommandLine.exe`:

```text
Could not find Studio Command-Line application!
Path: C:\Program Files\UiPathPlatform\Robot\26.10.0-cloud.25715\UiPath.Studio.CommandLine.exe
```

UiPath Studio was launched from the actual installation directory. Desktop inventory reported multiple Studio windows, and a usable Community Studio window was later observed after Assistant authentication. No workflow completed successfully and no output file is accepted as runtime proof. No success screenshots were fabricated; captured screenshots show only UiPath Assistant/Studio license state.

## Repairs implemented

1. Updated all five project dependency declarations to match packages present in the local NuGet cache.
2. Preserved the original portfolio and workflow files; no existing project was deleted or recreated.
3. Kept the integrated child workflows under `05-Integrated-Office-Automation/Workflows` and aligned the main workflow’s default output folder with `Output`.
4. Added `_packages/` to `.gitignore` so local packaging attempts are not committed.

## Remaining issues

- Studio runtime testing remains blocked until UiPath Studio can start normally and the matching Studio Command-Line component is available, or the workflows can be opened and run manually in Studio.
- Activity compatibility must be confirmed in Studio after package restore, especially the Presentations and Word activity property schemas.
- Office executable availability should be confirmed by opening Excel, Word, and PowerPoint manually before running Office-dependent workflows.
- Browser runtime testing still requires Chrome and the UiPath browser extension to be active.

## Required next validation

Open each `project.json` in UiPath Studio, restore packages, resolve any designer migration prompts, and run each main XAML. Record the observed output files and screenshots here only after the execution is visibly successful.

## Designer-session blocker (2026-11-13)

The installed UiPath Studio 26.10 Community application launches and is licensed. The current computer-use session can inspect Studio and its native project-open dialog, but the dialog remains stuck in an unrelated `matplotlib\\mpl-data\\sample_data` directory. Its filename and address controls are exposed in the accessibility tree but do not accept input through the available UI bridge. Because the requested repair must be performed by the actual Studio designer, no additional hand-authored XAML changes were made during this attempt, and all five projects remain NOT TESTED/FAIL pending a usable Studio project-opening interaction.

## Supported opening-route investigation (2026-11-13)

- `UiPath.Studio.CommandLine.exe` is installed and exposes `analyze`, `analyze-file`, `publish`, `manage`, `list`, and `options`. UiPath's documentation defines `analyze` and `analyze-file` as Workflow Analyzer commands; it does not document a command-line project-opening or designer-editing command.
- `UiPath.Studio.exe --help` exposes only `--profile`, `--open-project-origin`, `--help`, `--version`, and one positional value. Passing the portfolio `project.json` to the installed executable was tested. It started/targeted Studio processes but did not open the project into a usable designer window.
- Launching the same `project.json` through its Windows file association was tested. It did not produce a usable project designer window and did not enable workflow editing or execution.
- The available computer-use bridge can inspect Studio's accessibility tree, but cannot enter a path into the native Open dialog. No supported, reliable alternative was found in this environment.

These tests do not constitute runtime testing. The five projects remain NOT TESTED/FAIL, and no output files or screenshots are treated as execution evidence.

## Project-creation and package inventory (2026-11-13)

Read-only inspection of the installed Studio 26.10.0-cloud.25710 files and CLI was completed. The Studio CLI exposes `analyze`, `analyze-file`, `publish`, `manage`, `list`, and `options`; it does not expose a documented project-creation or template-instantiation command. UiPath's 2026.10 documentation describes project creation and built-in/custom templates as Studio UI operations. The only locally cached project-template package found was `UiPath.Template.SAPTestingProject` 24.10.1; no general-purpose CLI project-template factory was found.

Installed/cached activity packages relevant to this portfolio:

| Area | Package/version verified | Supported design direction |
|---|---|---|
| Browser | `UiPath.UIAutomation.Activities` 26.10.5 (also 26.10.3 cached) | Modern UI Automation activities, including Use Application/Browser and Get Text, must be placed through Studio's Activities panel/designer. Legacy `OpenBrowser`/`GetBrowserData` mappings in the existing files are not accepted as a repair basis. |
| Excel | `UiPath.Excel.Activities` 3.5.1 (3.6.0-preview also cached) | Modern Excel activities, including Use Excel File; the package API metadata exposes `UseExcelFile` and `ExcelProcessScope`, but workflow serialization must be generated by Studio. |
| Word | `UiPath.Word.Activities` 2.5.1 (2.6.0-preview also cached) | Use Word File/Word document activities; package API metadata exposes `UseWordDocument`. Existing legacy mappings require designer confirmation. |
| PowerPoint | `UiPath.Presentations.Activities` 2.6.0-preview | Presentation document activities must be selected and serialized by Studio; this is a preview package and needs designer validation before use. |
| Logging/integration | `UiPath.System.Activities` 26.10.0 | System activities package is installed; Log Message and Invoke Workflow File must be inserted/serialized by Studio rather than guessed from XML. |

No new project was created because the supported creation flow requires the Studio Start/New Project or Templates UI, and the native Studio project-opening/creation interaction is currently inaccessible through the available UI bridge. Existing projects and the preserved backup were not overwritten. No package inventory result is treated as runtime validation.

## Isolated designer test attempt (2026-11-13)

The approved attempt to create `UiPathPortfolio-Test` was stopped before any file creation. The computer-use session's window identifiers no longer resolve to the UiPath Studio windows listed by desktop inventory; direct Studio access returned `getApp requires { windowId } from listApps() or listWindows()`, and the available window identifier resolved to the Codex window rather than Studio. Since the Studio designer could not be targeted, no Process template was selected, no workflow was created, and no execution result exists. The required next action is to reinitialize the computer-use connection so it exposes a current UiPath Studio window identifier; no manual project-file or XML workaround is appropriate.

## Final runtime-test attempt (2026-11-13)

The computer-use session was reset once and the native window inventory refreshed. UiPath Studio windows were listed, but binding those identifiers returned no Studio accessibility tree and visually displayed the Codex window instead. Studio could not be operated to create or execute the isolated test workflow. No test project, workflow files, runtime outputs, or success screenshots were created.

Final result: **FAIL — execution could not be verified.** The portfolio should be treated as an untested draft, not as a working UiPath project.
