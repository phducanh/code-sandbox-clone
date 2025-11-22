# Test Page Documentation

## Overview

This document provides comprehensive information about the Code Playground test page - a browser-based JavaScript/React development environment with integrated Chrome DevTools debugging capabilities.

**Live Demo:** [https://phducanh.github.io/code-sandbox-clone/](https://phducanh.github.io/code-sandbox-clone/)

## Features

### 🎨 **Interactive Code Editor**

- Monaco Editor integration with syntax highlighting
- Real-time code editing with TypeScript/JavaScript support
- Automatic JSX transpilation using Babel Standalone
- Dark/Light theme support

### 🌐 **Live Preview**

- Instant code execution in isolated iframe
- Hot reload capability
- Support for React components
- Import maps for external dependencies (React, React-DOM)

### 🔧 **Transpiled Code View**

- View Babel-transpiled JSX code
- Real-time transpilation feedback
- Error handling and display

### 🛠️ **Chrome DevTools Integration**

- Full Chrome DevTools powered by [Chii](https://github.com/liriliri/chii)
- Debug protocol communication via [Chobitsu](https://github.com/liriliri/chobitsu)
- Inspect DOM elements
- Console logging
- Network monitoring
- JavaScript debugging
- Local storage inspection

### 📐 **Resizable Layout**

- Drag-to-resize panels (horizontal and vertical)
- Persistent layout with CSS custom properties
- Responsive design

## Architecture

### Component Structure

```
App.tsx
├── Header (Controls)
│   ├── Logo
│   ├── Theme Toggle
│   ├── DevTools Toggle
│   └── Reload Button
├── Left Panel
│   └── CodeEditor (Monaco)
├── Right Panel
│   ├── Tabs (Preview / Transpiled)
│   │   ├── Preview (iframe)
│   │   └── Transpiled Code (Monaco)
│   └── DevTools Panel (iframe)
└── Grid Resizers
```

### Communication Flow

```
┌─────────────────┐
│   Parent App    │
│   (App.tsx)     │
└────────┬────────┘
         │
    ┌────┴────┐
    │         │
┌───▼──────┐  │  ┌──────────┐
│ Preview  │  └─▶│ DevTools │
│  iframe  │◀────│  iframe  │
└──────────┘     └──────────┘
    │
    │ Chobitsu
    │ Protocol
    │
    ▼
Chrome DevTools
```

### Message Passing

The application uses `postMessage` API for inter-frame communication:

1. **CODE_UPDATE**: Parent → Preview iframe

   - Triggers code execution in preview
   - Initiates JSX transpilation

2. **TRANSPILED_CODE**: Preview iframe → Parent

   - Sends transpiled code back to parent
   - Updates transpiled code view

3. **DEV**: Parent ↔ DevTools iframe

   - Forwards Chrome DevTools Protocol messages
   - Enables debugging capabilities

4. **LOADED**: Preview iframe → Parent
   - Signals iframe ready state
   - Initializes DevTools connection

## Technical Details

### JSX Transpilation

The preview iframe uses Babel Standalone to transpile JSX:

```javascript
const transformedCode = window.Babel.transform(value, {
  presets: ["react"],
  plugins: [],
}).code;
```

- Automatic JSX detection using regex: `/<[A-Z]|<[a-z]/`
- Fallback to plain JavaScript if no JSX detected
- Error handling with user feedback

### Import Maps

External dependencies are loaded via import maps:

```json
{
  "imports": {
    "react": "https://esm.sh/react@18",
    "react-dom": "https://esm.sh/react-dom@18",
    "react-dom/client": "https://esm.sh/react-dom@18/client"
  }
}
```

### DevTools Integration

- **Chii**: Provides the Chrome DevTools frontend
- **Chobitsu**: Implements Chrome DevTools Protocol in the preview iframe
- Communication via `sendRawMessage` and message event listeners

### Layout System

Uses CSS custom properties for dynamic resizing:

```css
--left-panel-width: 50%
--right-top-height: 60%
```

Benefits:

- No React re-renders during resize
- Smooth drag performance
- Persistent layout state

## Usage Guide

### Basic Usage

1. **Write Code**: Edit JavaScript/React code in the left panel
2. **View Preview**: See live output in the Preview tab
3. **Debug**: Toggle DevTools to inspect and debug
4. **Check Transpilation**: Switch to Transpiled tab to see compiled code

### Sample Code

The playground comes with a default React counter example:

```jsx
import React from "react";
import ReactDOM from "react-dom/client";

function HelloWorld() {
  const [count, setCount] = React.useState(0);

  return (
    <div style={{ padding: "20px" }}>
      <h1>Hello, Playground! 🚀</h1>
      <p>Count: {count}</p>
      <button onClick={() => setCount(count + 1)}>Click me!</button>
    </div>
  );
}

const root = ReactDOM.createRoot(document.getElementById("app"));
root.render(<HelloWorld />);
```

### Keyboard Shortcuts

- **Cmd/Ctrl + =**: Zoom in (works in preview iframe)
- **Cmd/Ctrl + -**: Zoom out (works in preview iframe)

### Theme Toggle

- Click "☀️ Light" / "🌙 Dark" button to switch themes
- Theme persists in localStorage
- Applies to all panels including DevTools

## Development

### Local Setup

```bash
# Install dependencies
npm install

# Start development server
npm run dev

# Build for production
npm run build

# Preview production build
npm run preview
```

### Key Dependencies

- **React 19**: UI framework
- **Monaco Editor**: Code editor component
- **Vite**: Build tool and dev server
- **TailwindCSS**: Styling
- **Radix UI**: Accessible UI components
- **Babel Standalone**: JSX transpilation
- **Chii**: DevTools frontend
- **Chobitsu**: DevTools protocol implementation

### File Structure

```
src/
├── App.tsx                 # Main application component
├── components/
│   ├── CodeEditor.tsx      # Monaco editor wrapper
│   ├── GridResizer.tsx     # Resizable panel divider
│   ├── Logo.tsx            # Application logo
│   └── ui/                 # Radix UI components
│       ├── button.tsx
│       └── tabs.tsx
├── lib/
│   └── utils.ts            # Utility functions
└── main.tsx                # Application entry point
```

## Deployment

### GitHub Pages

The project is configured for automatic deployment to GitHub Pages via GitHub Actions.

**Workflow**: `.github/workflows/demo-page.yml`

```yaml
- Build with Bun
- Archive dist folder
- Deploy to GitHub Pages
```

**Important**: The `vite.config.ts` must include the correct base path:

```typescript
export default defineConfig({
  base: "/code-sandbox-clone/",
  // ...
});
```

### Manual Deployment

```bash
# Build the project
npm run build

# The dist/ folder contains the static files
# Deploy dist/ to any static hosting service
```

## Browser Compatibility

- **Chrome/Edge**: Full support ✅
- **Firefox**: Full support ✅
- **Safari**: Full support ✅
- **Mobile browsers**: Limited (DevTools may not work optimally)

## Known Limitations

1. **No File System**: Single-file editing only
2. **Limited Imports**: Only ESM modules from CDN
3. **No TypeScript Compilation**: JavaScript/JSX only
4. **DevTools Limitations**: Some advanced features may not work
5. **Mobile Support**: Best experienced on desktop

## Troubleshooting

### Preview Not Loading

- Check browser console for errors
- Ensure code has no syntax errors
- Try clicking the Reload button

### DevTools Not Working

- Ensure preview iframe is loaded first
- Check that Chobitsu script loaded successfully
- Refresh the page

### Transpilation Errors

- Verify JSX syntax is correct
- Check that React is imported
- Look for error messages in transpiled code view

## Performance Optimization

### Implemented Optimizations

1. **React.memo**: Preview and DevTools components memoized
2. **useCallback**: Resize handlers optimized
3. **useMemo**: Iframe URLs cached
4. **CSS Custom Properties**: No re-renders during resize
5. **Blob URLs**: Efficient iframe content delivery

### Best Practices

- Keep code size reasonable (< 10KB recommended)
- Avoid infinite loops in preview code
- Use DevTools sparingly for better performance

## Security Considerations

### Iframe Sandbox

The preview iframe uses restrictive sandbox attributes:

```html
sandbox="allow-popups-to-escape-sandbox allow-scripts allow-popups allow-forms
allow-pointer-lock allow-top-navigation allow-modals allow-same-origin"
```

### Content Security

- Code executes in isolated iframe context
- No access to parent window directly
- Communication only via postMessage

## Future Enhancements

- [ ] Multi-file support
- [ ] TypeScript compilation
- [ ] Code sharing via URL
- [ ] Save/Load from localStorage
- [ ] More import map presets
- [ ] Mobile-optimized layout
- [ ] Code templates library
- [ ] Collaborative editing

## Credits

- **Chrome DevTools**: [Chii](https://github.com/liriliri/chii) by liriliri
- **DevTools Protocol**: [Chobitsu](https://github.com/liriliri/chobitsu) by liriliri
- **Code Editor**: [Monaco Editor](https://microsoft.github.io/monaco-editor/) by Microsoft
- **UI Components**: [Radix UI](https://www.radix-ui.com/)

## License

This project is for educational and demonstration purposes.

## Support

For issues or questions:

- Check existing documentation
- Review browser console for errors
- Inspect network requests in browser DevTools
- Verify all external scripts loaded successfully

---

**Last Updated**: November 22, 2025
