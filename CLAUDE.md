# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

SA Screen Designer is a browser-based UI specification/wireframe tool for designing screen mockups. It is a single-page HTML application (`designer.html`) with no build system - open directly in a browser to use.

## Running the Application

Open `designer.html` in a web browser. No server or build step required.

For PNG export functionality, the application loads html2canvas from CDN.

## Architecture

### Single-File Application
The entire application is contained in `designer.html`:
- **CSS**: Inline styles with CSS custom properties for theming (lines 9-830)
- **HTML**: Component palette sidebar, canvas, and properties panel (lines 832-1001)
- **JavaScript**: All logic including drag-drop, component rendering, undo/redo, and save/load (lines 1003-2168)

### Core State Variables
- `components[]` - Array of all dropped components with their properties
- `selectedComponent` - Currently selected component reference
- `history[]` / `historyIndex` - Undo/redo state stack
- `currentTheme` - Active theme name

### Component System
Components are defined by type and rendered via `getComponentHTML()`. Each component has:
- Unique ID (`comp_N`)
- Position (x, y) and dimensions (width, height)
- Type-specific properties in `properties` object

Available component types: label, title, textbox, textarea, datepicker, dropdown, checkbox, radio, button, iconbutton, table, tabs, panel, annotation, notebox

### Theming
Six themes available via CSS custom properties: default, aj, metronic, blue, purple, gray. Theme affects table headers, buttons, tabs, inputs, and labels.

### Icon System
Icon buttons and table action columns support two icon types:
- **emoji**: Unicode emoji characters
- **keenicons**: KeenThemes icon font (loaded from `assets/plugins/global/plugins.bundle.css`)

### Data Persistence
- **Save**: Exports JSON file containing screen metadata, theme, and component array
- **Load**: Imports JSON file and restores complete design state
- **Export PNG**: Uses html2canvas library for image capture

## Directory Structure

- `designer.html` - Main application
- `assets/plugins/global/` - KeenIcons and global plugin bundles
- `assets/js/` - Additional JavaScript bundles (scripts, widgets)
- `AJDesign/`, `Spec/`, `Bug/` - Screenshot/reference image directories

## Key Functions

- `addComponent(type, x, y)` - Creates new component on canvas
- `renderComponent(comp)` - Creates DOM element for component
- `getComponentHTML(comp)` - Returns inner HTML for component type
- `updateComponentElement(comp)` - Re-renders component after property change
- `saveDesign()` / `loadDesign()` - JSON persistence
- `exportImage()` - PNG export using html2canvas
- `changeTheme(theme)` - Applies theme CSS variables
