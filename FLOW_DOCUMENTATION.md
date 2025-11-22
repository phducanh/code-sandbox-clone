# DevTools Frontend Demo - Flow Documentation

Tài liệu này mô tả chi tiết luồng hoạt động của ứng dụng trong 2 scenarios chính.

## Kiến trúc tổng quan

```
┌─────────────────────────────────────────────────────────┐
│                    Parent Window (App.tsx)               │
│                     Message Router                       │
└───────────┬─────────────────────────────┬───────────────┘
            │                             │
            ↓                             ↓
┌─────────────────────┐         ┌─────────────────────┐
│   Preview iframe    │         │  DevTools iframe    │
│   (User Code)       │         │     (Chii)          │
│                     │         │                     │
│  ┌───────────────┐  │         │  - Elements tab     │
│  │   Chobitsu    │  │         │  - Console tab      │
│  │   (Bridge)    │  │         │  - Network tab      │
│  └───────────────┘  │         │  - Sources tab      │
└─────────────────────┘         └─────────────────────┘
```

---

## Scenario 1: User Inspect Element trong Preview

### Mô tả

User click vào một element trong Preview iframe để inspect nó trong DevTools.

### Flow chi tiết

#### Bước 1: User Interaction

```
User clicks element trong Preview iframe
└─> Browser triggers element selection
```

#### Bước 2: Chobitsu Detection & Serialization

```javascript
// Trong Preview iframe
// Chobitsu tự động detect element selection
// và serialize element data:

{
  method: 'DOM.inspectNodeRequested',
  params: {
    nodeId: 123,
    nodeName: 'DIV',
    nodeType: 1,
    attributes: ['class', 'container', 'id', 'app'],
    childNodeCount: 3
  }
}
```

**Data được collect:**

- Node ID (internal identifier)
- Node name (tag name)
- Attributes (class, id, data-\*, etc.)
- Computed styles
- Box model (margin, padding, border, dimensions)

#### Bước 3: Chobitsu → Parent Window

```javascript
// Trong Preview iframe
chobitsu.setOnMessage((message) => {
  if (message.includes('"id":"tmp')) return; // Skip internal
  window.parent.postMessage(message, "*"); // Send to parent
});
```

**Message format:**

```json
{
  "method": "DOM.inspectNodeRequested",
  "params": {
    "nodeId": 123,
    "nodeName": "DIV",
    "attributes": ["class", "container"]
  }
}
```

#### Bước 4: Parent Window Routing

```javascript
// Trong App.tsx
window.addEventListener("message", ({ data }) => {
  try {
    const parsed = typeof data === "string" ? JSON.parse(data) : data;

    // Forward to DevTools
    if (devtoolsIframeRef.current?.contentWindow) {
      devtoolsIframeRef.current.contentWindow.postMessage(parsed, "*");
    }
  } catch (e) {
    console.error("Message routing error:", e);
  }
});
```

#### Bước 5: DevTools UI Update

```
DevTools iframe nhận message
└─> Parse element data
└─> Update Elements tab:
    ├─> Highlight element trong DOM tree
    ├─> Expand parent nodes nếu cần
    ├─> Show HTML structure
    └─> Display trong right panel:
        ├─> Styles panel (CSS rules)
        ├─> Computed tab (computed styles)
        ├─> Layout tab (box model)
        └─> Event Listeners tab
```

#### Bước 6: Request Additional Data (nếu cần)

```javascript
// DevTools request thêm CSS data
{
  method: 'CSS.getMatchedStylesForNode',
  params: { nodeId: 123 }
}

// Flow: DevTools → Parent → Preview → Chobitsu
```

#### Bước 7: Chobitsu Execute & Return

```javascript
// Chobitsu collect CSS data
const styles = window.getComputedStyle(element);
const matchedRules = getMatchedCSSRules(element);

// Return to DevTools
{
  id: 'request_id',
  result: {
    inlineStyle: {...},
    matchedCSSRules: [...],
    inherited: [...]
  }
}
```

#### Bước 8: DevTools Render Styles

```
DevTools nhận CSS data
└─> Render trong Styles panel:
    ├─> Inline styles (highest priority)
    ├─> CSS rules (by specificity)
    ├─> Inherited styles (grayed out)
    └─> Computed values tab
```

### Visual Flow Diagram

```
┌─────────────┐
│   Preview   │
│   (iframe)  │ ← 1. User clicks element
└──────┬──────┘
       │ 2. Chobitsu detects & serializes
       │    { method: 'DOM.inspectNode', params: {...} }
       ↓
┌──────────────────────────────────┐
│      window.parent.postMessage   │
└──────────────┬───────────────────┘
               │ 3. Message to parent
               ↓
┌─────────────────────┐
│   Parent Window     │
│     (App.tsx)       │ ← 4. Route message
└──────────┬──────────┘
           │ 5. Forward to DevTools
           ↓
┌─────────────────────┐
│   DevTools iframe   │
│      (Chii)         │ ← 6. Update UI
└─────────────────────┘
       │ Elements tab:
       ├─> DOM tree (highlighted)
       ├─> Styles panel
       ├─> Computed tab
       └─> Box model
```

### Code Example

```javascript
// Preview iframe - Chobitsu setup
chobitsu.setOnMessage((message) => {
  // Filter internal messages
  if (message.includes('"id":"tmp')) return;

  // Forward to parent
  window.parent.postMessage(message, "*");
});

// Parent window - Message router
window.addEventListener("message", ({ data }) => {
  // Forward to DevTools
  devtoolsIframeRef.current?.contentWindow?.postMessage(data, "*");
});

// DevTools receives and displays
// (handled by Chii library)
```

---

## Scenario 2: User Sửa Content trong DevTools

### Mô tả

User edit text content của một element trong DevTools Elements tab, và thay đổi được reflect trong Preview iframe.

### Flow chi tiết

#### Bước 1: User Edit trong DevTools

```
User trong DevTools Elements tab:
1. Double-click vào text node
2. Edit: "Hello" → "Hello World"
3. Press Enter để confirm
```

#### Bước 2: DevTools Generate Command

```javascript
// DevTools (Chii) tạo Chrome DevTools Protocol command
{
  id: 'msg_12345',
  method: 'DOM.setNodeValue',
  params: {
    nodeId: 123,
    value: 'Hello World'
  }
}
```

**Các loại commands khác:**

- `DOM.setOuterHTML` - Edit HTML structure
- `CSS.setStyleTexts` - Edit CSS styles
- `DOM.setAttributeValue` - Edit attributes
- `DOM.removeNode` - Delete element

#### Bước 3: DevTools → Parent Window

```javascript
// DevTools iframe gửi command
window.parent.postMessage(
  {
    event: "DEV",
    data: JSON.stringify({
      id: "msg_12345",
      method: "DOM.setNodeValue",
      params: { nodeId: 123, value: "Hello World" },
    }),
  },
  "*"
);
```

**Message structure:**

```json
{
  "event": "DEV",
  "data": "{\"id\":\"msg_12345\",\"method\":\"DOM.setNodeValue\",\"params\":{...}}"
}
```

#### Bước 4: Parent Window Routing

```javascript
// Trong App.tsx
window.addEventListener("message", ({ data }) => {
  const { event } = data;

  if (event === "DEV") {
    // Forward to Preview iframe
    if (previewIframeRef.current?.contentWindow) {
      previewIframeRef.current.contentWindow.postMessage(
        {
          event: "DEV",
          data: data.data, // Raw CDP command string
        },
        "*"
      );
    }
  }
});
```

#### Bước 5: Preview iframe Receives

```javascript
// Trong Preview iframe (generateHTML)
window.addEventListener("message", ({ data }) => {
  try {
    const { event, value } = data;

    if (event === "DEV") {
      // Forward to Chobitsu
      chobitsu.sendRawMessage(data.data);
    }
  } catch (e) {
    console.error("Preview message error:", e);
  }
});
```

#### Bước 6: Chobitsu Execute Command

```javascript
// Chobitsu internally:
// 1. Parse CDP command
const command = JSON.parse(message);
// { method: 'DOM.setNodeValue', params: {...} }

// 2. Get node reference
const node = getNodeByNodeId(command.params.nodeId);
// node = <div>Hello</div> text node

// 3. Execute mutation
node.textContent = command.params.value;
// DOM updated: <div>Hello World</div>

// 4. Prepare response
const response = {
  id: command.id,
  result: { success: true },
};
```

**Chobitsu execution examples:**

```javascript
// Set attribute
case 'DOM.setAttributeValue':
  element.setAttribute(params.name, params.value);
  break;

// Set outer HTML
case 'DOM.setOuterHTML':
  element.outerHTML = params.outerHTML;
  break;

// Set style
case 'CSS.setStyleTexts':
  element.style.cssText = params.styleText;
  break;
```

#### Bước 7: DOM Update Visible

```
Preview iframe DOM được update
└─> User thấy thay đổi ngay lập tức:
    "Hello" → "Hello World"
```

#### Bước 8: Confirmation Response

```javascript
// Chobitsu gửi confirmation về DevTools
chobitsu.setOnMessage((message) => {
  // Response: { id: 'msg_12345', result: { success: true } }
  window.parent.postMessage(message, "*");
});

// Parent forwards to DevTools
// DevTools nhận confirmation và update UI state
```

### Visual Flow Diagram

```
┌─────────────────────┐
│   DevTools iframe   │
│      (Chii)         │ ← 1. User edits: "Hello" → "Hello World"
└──────────┬──────────┘
           │ 2. Generate CDP command
           │    { method: 'DOM.setNodeValue', params: {...} }
           ↓
┌──────────────────────────────────┐
│  { event: 'DEV', data: '...' }   │
└──────────┬───────────────────────┘
           │ 3. postMessage to parent
           ↓
┌─────────────────────┐
│   Parent Window     │
│     (App.tsx)       │ ← 4. Route message
└──────────┬──────────┘
           │ 5. Forward with event: 'DEV'
           ↓
┌─────────────────────┐
│   Preview iframe    │ ← 6. Receive command
└──────────┬──────────┘
           │ 7. chobitsu.sendRawMessage()
           ↓
┌─────────────────────┐
│     Chobitsu        │
│   Execute command   │ ← 8. Update DOM
│   node.textContent  │    "Hello World" appears!
└─────────────────────┘
           │ 9. Send confirmation
           ↓
     (back to DevTools via parent)
```

### Code Example

```javascript
// DevTools generates command (handled by Chii)
const command = {
  id: generateId(),
  method: "DOM.setNodeValue",
  params: { nodeId: 123, value: "Hello World" },
};
window.parent.postMessage({ event: "DEV", data: JSON.stringify(command) }, "*");

// Parent routes message
window.addEventListener("message", ({ data }) => {
  if (data.event === "DEV") {
    previewIframeRef.current.contentWindow.postMessage(data, "*");
  }
});

// Preview forwards to Chobitsu
window.addEventListener("message", ({ data }) => {
  if (data.event === "DEV") {
    chobitsu.sendRawMessage(data.data);
  }
});

// Chobitsu executes (internal)
// node.textContent = 'Hello World';

// Confirmation flows back
// DevTools → Parent → Preview → Chobitsu → Preview → Parent → DevTools
```

---

## Event Types Summary

### Trong Parent Window (App.tsx)

```javascript
window.addEventListener("message", ({ data }) => {
  const { event } = data;

  switch (event) {
    case "DEV":
      // DevTools ↔ Preview communication
      // Chrome DevTools Protocol commands
      // Bidirectional: commands & responses
      break;

    case "CODE_UPDATE":
      // Editor → Preview
      // User types code in CodeEditor
      // Preview executes new code
      break;

    case "TRANSPILED_CODE":
      // Preview → Parent
      // Babel transpiled JSX → JS
      // Display in "Transpiled" tab
      break;

    case "LOADED":
      // Preview → Parent
      // Iframe finished loading
      // Enable DevTools protocols
      break;
  }
});
```

### Message Flow Summary

| Event Type        | Direction          | Purpose                  |
| ----------------- | ------------------ | ------------------------ |
| `DEV`             | DevTools ↔ Preview | CDP commands & responses |
| `CODE_UPDATE`     | Parent → Preview   | Execute user code        |
| `TRANSPILED_CODE` | Preview → Parent   | Show transpiled output   |
| `LOADED`          | Preview → Parent   | Iframe ready signal      |

---

## Key Components

### 1. Parent Window (App.tsx)

**Role:** Message router và state manager

**Responsibilities:**

- Route messages giữa DevTools và Preview
- Manage UI state (dark mode, panels, tabs)
- Handle code editor changes
- Coordinate iframe lifecycle

### 2. Preview Iframe

**Role:** Code execution environment

**Responsibilities:**

- Execute user code trong isolated context
- Host Chobitsu bridge
- Transpile JSX với Babel
- Render user UI

### 3. Chobitsu Library

**Role:** DevTools Protocol bridge

**Responsibilities:**

- Implement Chrome DevTools Protocol APIs
- Execute CDP commands trong iframe context
- Collect inspection data (DOM, CSS, Network)
- Serialize data về DevTools format

### 4. DevTools Iframe (Chii)

**Role:** DevTools UI

**Responsibilities:**

- Render DevTools interface
- Generate CDP commands từ user actions
- Display inspection data
- Provide debugging tools

---

## Communication Protocol

### Message Format

#### From DevTools to Preview

```json
{
  "event": "DEV",
  "data": "{\"id\":\"123\",\"method\":\"DOM.setNodeValue\",\"params\":{...}}"
}
```

#### From Preview to DevTools

```json
{
  "method": "DOM.inspectNodeRequested",
  "params": {
    "nodeId": 123,
    "nodeName": "DIV"
  }
}
```

#### From Parent to Preview

```json
{
  "event": "CODE_UPDATE",
  "value": "import React from 'react';\n..."
}
```

### Security Considerations

- Tất cả iframes sử dụng blob URLs (different origins)
- Communication chỉ qua `postMessage` API
- Preview iframe có sandbox restrictions
- No direct DOM access giữa các contexts

---

## Debugging Tips

### Enable Console Logging

```javascript
// Trong App.tsx - log all messages
window.addEventListener("message", ({ data }) => {
  console.log("[Parent] Received:", data);
});

// Trong Preview iframe - log Chobitsu messages
chobitsu.setOnMessage((message) => {
  console.log("[Chobitsu]:", message);
  window.parent.postMessage(message, "*");
});
```

### Common Issues

1. **Message not reaching DevTools**

   - Check if `devtoolsIframeRef.current` exists
   - Verify `contentWindow` is accessible
   - Check for CORS/sandbox restrictions

2. **DOM changes not reflecting**

   - Verify Chobitsu received command
   - Check nodeId is valid
   - Ensure element still exists in DOM

3. **Styles not updating**
   - Check CSS specificity
   - Verify style command format
   - Look for `!important` overrides

---

## Performance Considerations

### Message Throttling

```javascript
// Throttle frequent updates
let updateTimeout;
const throttledUpdate = (data) => {
  clearTimeout(updateTimeout);
  updateTimeout = setTimeout(() => {
    window.parent.postMessage(data, "*");
  }, 16); // ~60fps
};
```

### Memory Management

```javascript
// Cleanup blob URLs
useEffect(() => {
  return () => {
    if (iframeSrcUrl) {
      URL.revokeObjectURL(iframeSrcUrl);
    }
  };
}, [iframeSrcUrl]);
```

---

## Future Enhancements

- [ ] Add breakpoint support
- [ ] Network request interception
- [ ] Performance profiling
- [ ] Memory heap snapshots
- [ ] React DevTools integration
- [ ] Source map support
- [ ] Hot module replacement

---

## References

- [Chrome DevTools Protocol](https://chromedevtools.github.io/devtools-protocol/)
- [Chobitsu Library](https://github.com/liriliri/chobitsu)
- [Chii DevTools](https://github.com/liriliri/chii)
- [postMessage API](https://developer.mozilla.org/en-US/docs/Web/API/Window/postMessage)
