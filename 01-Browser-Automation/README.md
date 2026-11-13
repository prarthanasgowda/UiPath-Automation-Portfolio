# Browser Automation

## Purpose
Opens a configurable URL in Chrome, reads the page title, and logs the result using UiPath browser activities.

## Run
Open `project.json` in Studio, restore packages, set `in_Url` in the Arguments panel, and run `BrowserAutomation.xaml`. Chrome and the UiPath browser extension are required.

## Inputs and outputs
`Input` is reserved for local fixtures. Runtime logs are written by UiPath; generated files belong in `Output`.

## Status
Static file validation only; runtime execution still required in UiPath Studio.
