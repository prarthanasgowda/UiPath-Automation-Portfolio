# Integrated Office Automation

## Purpose
`Main.xaml` invokes browser, Excel, PowerPoint, and Word child workflows in sequence. Arguments and relative folders provide the data handoff and output location.

## Run
Open `project.json`, restore all dependencies, confirm Chrome and the UiPath browser extension are installed, and run `Main.xaml`. The default browser URL is `https://example.com`; outputs are written to `Output`. The invoked child workflows are in `Workflows`.

## Error handling
Browser access is wrapped in Try Catch and diagnostic Log Message activities. Review the UiPath Output panel for activity-level errors before accepting a runtime test as passed.

## Status
Static file validation only; complete runtime execution still required in UiPath Studio.
