# UiPath Automation Portfolio

![UiPath](https://img.shields.io/badge/UiPath-Windows%20Projects-blue)
![Status](https://img.shields.io/badge/status-local%20draft-orange)
![License](https://img.shields.io/badge/license-MIT-green)

## About

A practical portfolio of five independent UiPath Windows automation projects. The collection demonstrates browser interaction, structured Excel processing, document generation, presentation automation, and cross-application orchestration using reusable workflows and portable relative paths.

> Runtime status is documented in `TESTING-REPORT.md`. The workflows are preserved as a local development draft until each one has been executed successfully in UiPath Studio.

## Projects

| Project | Focus | Entry point |
|---|---|---|
| [01 Browser Automation](01-Browser-Automation) | Open a browser, extract page metadata, and log results | `BrowserAutomation.xaml` |
| [02 Excel Automation](02-Excel-Automation) | Create/process office data with DataTables and Excel activities | `ExcelAutomation.xaml` |
| [03 Word Report Generator](03-Word-Report-Generator) | Generate a formatted Word report | `WordReportGenerator.xaml` |
| [04 PowerPoint Automation](04-PowerPoint-Automation) | Create a presentation with title/content slides | `PowerPointAutomation.xaml` |
| [05 Integrated Office Automation](05-Integrated-Office-Automation) | Orchestrate browser, Excel, PowerPoint, and Word workflows | `Main.xaml` |

## Technologies and skills

UiPath Studio Windows projects, UI Automation, Excel/Word/Presentations activities, Invoke Workflow File, variables and arguments, DataTables, Try Catch, Log Message, relative paths, package restoration, and reusable workflow design.

## Repository structure

```text
UiPath-Automation-Portfolio/
├── 01-Browser-Automation/
├── 02-Excel-Automation/
├── 03-Word-Report-Generator/
├── 04-PowerPoint-Automation/
├── 05-Integrated-Office-Automation/
├── README.md
├── TESTING-REPORT.md
└── .gitignore
```

Each project is self-contained with its own `project.json`, workflow files, README, and input/output folders where applicable.

## Key concepts demonstrated

- Independent UiPath Windows processes with portable project-relative paths
- Browser data extraction and configurable workflow arguments
- DataTable preparation, filtering, calculations, and Excel output
- Word report generation and PowerPoint slide creation
- Workflow reuse through child workflows and argument-based data transfer
- Try Catch, Log Message, package dependencies, and structured output folders

## Learning objectives

This portfolio is designed to practice UiPath project organization, package management, variables and arguments, office automation, browser automation, error handling, and integration patterns in a GitHub-friendly structure.

## Local execution

Open any project’s `project.json` in UiPath Studio, restore its dependencies, and run the project’s entry-point XAML. Each project contains a README and uses `Input`/`Output` folders where applicable. Review the project-specific instructions before execution.

## Verification policy

XAML/XML and JSON have been inspected locally. Runtime status is intentionally documented in `TESTING-REPORT.md` and is not inferred from static parsing or sample output files. No GitHub publishing has been performed.

## Original work preserved

The initial project remains available at `../Smart Office Automation System Using UiPath` and is the source of the reusable office workflows used here.
