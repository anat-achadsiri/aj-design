# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

SA Screen Designer is a browser-based UI specification/wireframe tool for designing screen mockups. It is a single-page HTML application (`index.html`) with no build system - open directly in a browser to use.

## Running the Application

Open `index.html` in a web browser. No server or build step required.

For PNG export functionality, the application loads html2canvas from CDN.

## Architecture

### Single-File Application
The entire application is contained in `index.html` (~5000+ lines):
- **CSS** (lines ~10-1358): Inline styles with CSS custom properties for theming
- **HTML** (lines ~1360-1620): Component palette sidebar, canvas, properties panel, modals
- **JavaScript** (lines ~1625-end): All application logic

### Core State Variables
```javascript
components[]           // Array of all dropped components with their properties
selectedComponent      // Currently selected component reference
selectedComponents[]   // Multi-selection array for group operations
history[] / historyIndex  // Undo/redo state stack
currentTheme           // Active theme name ('default', 'aj', 'metronic', 'blue', 'purple', 'gray')
documents[]            // Multi-document management (tabs)
activeDocumentId       // Currently active document
gridSettings           // Grid/ruler/snap settings object
```

### Document Management System
The app supports multiple open documents via a tab interface:
- `Document` class holds all state for each design file
- `newDocument()`, `switchToDocument()`, `closeDocument()` manage tabs
- Each document has its own undo/redo history
- Unsaved changes prompt before closing/switching

### Component System
Components are defined by type and rendered via `getComponentHTML()`. Each component has:
- Unique ID (`comp_N`)
- Position (x, y) and dimensions (width, height)
- Type-specific properties in `properties` object

Available component types: label, title, textbox, textarea, datepicker, dropdown, checkbox, radio, button, iconbutton, table, tabs, panel, annotation, notebox, image

### Multi-Selection
- Marquee selection: Click and drag on canvas background
- Shift+click: Add/remove from selection
- `selectedComponents[]` stores multi-selection
- Group operations: move, duplicate (Ctrl+D), delete

### Grid and Ruler System
```javascript
gridSettings = {
    showGrid: false,      // Toggle grid overlay
    showRulers: true,     // Toggle rulers
    snapToGrid: false,    // Snap component positions
    gridSize: 'small',    // 'small' (10px), 'medium' (50px), 'large' (100px)
    snapSize: 10          // Pixels for snap-to-grid
}
```

### Theming
Six themes available via CSS custom properties. Theme CSS variables are defined at the top of `<style>` using `[data-theme="themename"]` selectors. Theme affects table headers, buttons, tabs, inputs, and labels.

### Icon System
Icon buttons and table action columns support two icon types:
- **emoji**: Unicode emoji characters
- **keenicons**: KeenThemes icon font (loaded from `assets/plugins/global/plugins.bundle.css`)

### AI Image Import
The application can import UI mockups from images using Claude's Vision API:
- Requires Anthropic API key (stored in localStorage)
- Analyzes images to extract component structure
- Converts to designer components automatically
- See `API Keys/info.txt` for setup instructions

### Data Persistence
- **Save/Save As**: Exports JSON file with screen metadata, theme, canvas size, and components
- **Load**: Imports JSON file and creates new document tab
- **Export PNG**: Uses html2canvas library (loaded from CDN)

## Key Functions

| Function | Purpose |
|----------|---------|
| `addComponent(type, x, y)` | Creates new component on canvas |
| `renderComponent(comp)` | Creates DOM element for component |
| `getComponentHTML(comp)` | Returns inner HTML for component type |
| `updateComponentElement(comp)` | Re-renders component after property change |
| `showPropertiesPanel(comp)` | Shows component-specific properties panel |
| `saveHistory()` | Saves current state for undo/redo |
| `docSave()` / `docSaveAs()` | Document save operations |
| `exportImage()` | PNG export using html2canvas |
| `changeTheme(theme)` | Applies theme CSS variables |
| `analyzeImageWithClaude()` | AI-powered image to component conversion |
| `snapToGrid(value)` | Snaps position value to grid |

## Project Structure

```
/index.html          # Main application (single-file)
/assets/             # Metronic theme assets (plugins, icons, fonts)
/Spec/               # Reference screenshots for UI design
/Bug/                # Bug report screenshots
/API Keys/           # Instructions for Claude API setup
/*.json              # Example saved design files
```

## Keyboard Shortcuts

- `Ctrl+N`: New document
- `Ctrl+S`: Save
- `Ctrl+O`: Open
- `Ctrl+Z`: Undo
- `Ctrl+Y`: Redo
- `Ctrl+D`: Duplicate selected
- `Delete`: Delete selected
- `Shift+Click`: Multi-select
