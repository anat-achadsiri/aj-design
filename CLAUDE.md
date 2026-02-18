# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

SA Screen Designer is a browser-based UI specification/wireframe tool for designing screen mockups. It is a single-page HTML application (`index.html`) with no build system - open directly in a browser to use.

## Running the Application

Open `index.html` in a web browser. No server or build step required.

For PNG export functionality, the application loads html2canvas from CDN.

## Architecture

### Single-File Application
The entire application is contained in `index.html` (~5350 lines):
- **CSS** (lines 9-1497): Inline styles with CSS custom properties for theming
- **HTML** (lines 1499-1766): Component palette sidebar, canvas, properties panel, modals
- **JavaScript** (lines 1781-5350): All application logic

JavaScript is organized into sections:
- State variables and initialization (~1782-1827)
- Document class and management system (~1828-2460)
- Grid/Ruler/Snap functions (~2463-2620)
- Component functions: `addComponent` (~2820), `renderComponent` (~2915), `getComponentHTML` (~3098), `getDefaultProperties` (~2883)
- Event handlers (drag, resize, selection) (~3600-4280)
- Save/Load/Export functions: `docSave` (~2160), `exportImage` (~4856)
- AI image import: `analyzeImage` (~5190), `getAIPrompt` (~5039)

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
- `Document` class (line 1828) holds all state for each design file (id, filePath, fileHandle, displayName, isDirty, content, history)
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
```

Functions: `toggleGrid()` (2463), `toggleRulers()` (2474), `snapToGrid()` (2611)

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
When modifying component properties (in `getDefaultProperties()` at line 2883 or adding new components), you MUST also update `getAIPrompt()` at line 5039 to keep the AI prompt in sync:
- Update `propDescriptions` object with the new/modified property definitions
- Add new component types to `componentTypes` array if adding new components
- Update CRITICAL RULES if the new component has special handling requirements

### Data Persistence
- **Save/Save As**: Exports JSON file with screen metadata, theme, canvas size, and components
- **Load**: Imports JSON file and creates new document tab
- **Export PNG**: Uses html2canvas library (loaded from CDN at line 5353)

## Key Functions

| Function | Line | Purpose |
|----------|------|---------|
| `addComponent(type, x, y)` | 2820 | Creates new component on canvas |
| `renderComponent(comp)` | 2915 | Creates DOM element for component |
| `getComponentHTML(comp)` | 3098 | Returns inner HTML for component type |
| `getDefaultProperties(type)` | 2883 | Returns default properties for component type |
| `saveHistory()` | - | Saves current state for undo/redo |
| `docSave()` / `docSaveAs()` | 2160/2177 | Document save operations |
| `exportImage()` | 4856 | PNG export using html2canvas |
| `changeTheme(theme)` | - | Applies theme CSS variables |
| `analyzeImage()` | 5190 | AI-powered image to component conversion |
| `getAIPrompt()` | 5039 | Generates prompt for AI image analysis |
| `snapToGrid(value)` | 2611 | Snaps position value to grid |
| `toggleGrid()` / `toggleRulers()` | 2463/2474 | Toggle grid/ruler display |

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
