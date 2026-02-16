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
- **CSS** (lines ~10-1390): Inline styles with CSS custom properties for theming
- **HTML** (lines ~1391-1655): Component palette sidebar, canvas, properties panel, modals
- **JavaScript** (lines ~1657-end): All application logic

JavaScript is organized into sections:
- State variables and initialization (lines ~1658-1700)
- Document Management System (lines ~1700-2300)
- Grid/Ruler/Snap functions (lines ~2339-2500)
- Component rendering and manipulation (lines ~2696-3600)
- Event handlers (drag, resize, selection) (lines ~3600-4280)
- Save/Load/Export functions (lines ~4280-4730)
- AI image import (Claude API integration) (lines ~4730-end)

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
- `Document` class holds all state for each design file (id, filePath, fileHandle, displayName, isDirty, content, history)
- Uses File System Access API (`fileHandle`) for direct file saves
- `newDocument()`, `switchToDocument()`, `closeDocument()` manage tabs
- Each document has its own undo/redo history and component ID counter
- Unsaved changes prompt before closing/switching (isDirty flag)
- Tab titles show asterisk (*) when document has unsaved changes

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
- Drag images: Visual clones appear during drag operations via `createDragImage()`

### Grid and Ruler System
```javascript
gridSettings = {
    showGrid: false,      // Toggle grid overlay
    showRulers: true,     // Toggle rulers (30px ruler bars on top/left)
    snapToGrid: false,    // Snap component positions
    gridSize: 'small',    // 'small' (10px), 'medium' (50px), 'large' (100px)
    snapSize: 10          // Pixels for snap-to-grid
}

// Grid size constants
gridSizes = { small: 10, medium: 50, large: 100 }
```

Functions: `toggleGrid()`, `toggleRulers()`, `toggleSnapToGrid()`, `setGridSize()`, `drawRulers()`, `snapToGrid()`

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

**IMPORTANT - Sync AI Prompt with Component Properties:**
When modifying component properties (in `getDefaultProperties()` or adding new components), you MUST also update `getAIPrompt()` function to keep the AI prompt in sync:
- Update `propDescriptions` object with the new/modified property definitions
- Add new component types to `componentTypes` array if adding new components
- Update CRITICAL RULES if the new component has special handling requirements

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
| `showProperties(comp)` | Shows component-specific properties panel |
| `saveHistory()` | Saves current state for undo/redo |
| `docSave()` / `docSaveAs()` | Document save operations |
| `exportImage()` | PNG export using html2canvas |
| `changeTheme(theme)` | Applies theme CSS variables |
| `analyzeImage()` | AI-powered image to component conversion |
| `getAIPrompt()` | Generates prompt for AI image analysis (sync with getDefaultProperties) |
| `snapToGrid(value)` | Snaps position value to grid |
| `createDragImage(comp, el, x, y)` | Creates visual clone during drag operations |
| `toggleGrid()` / `toggleRulers()` | Toggle grid/ruler display |
| `saveDocumentState()` / `loadDocumentState()` | Save/restore document state when switching tabs |

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
- `Ctrl+Shift+S`: Save As
- `Ctrl+O`: Open
- `Ctrl+W`: Close current tab
- `Ctrl+Z`: Undo
- `Ctrl+Y`: Redo
- `Ctrl+D`: Duplicate selected
- `Delete`: Delete selected
- `Shift+Click`: Multi-select
- `Ctrl/Cmd+Click`: Add to selection
