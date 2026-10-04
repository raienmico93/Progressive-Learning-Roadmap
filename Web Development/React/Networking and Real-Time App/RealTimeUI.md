# Real-Time UI Patterns & High-Frequency Data: A Comprehensive Programming Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition:** Real-Time UI Patterns & High-Frequency Data is the discipline of designing React interfaces that remain responsive and consistent while receiving continuous streams of data from multiple users or high-velocity sources, using presence tracking, conflict resolution, throttled state synchronisation, and optimistic merging.

**Technical Definition:** Real-time UI patterns address the unique challenges of interfaces that must render updates arriving at 10–100+ events per second from WebSocket connections, collaborative editing sessions, or live data feeds. These challenges include: **presence tracking** (knowing who else is online and where their cursors are), **high-frequency state synchronisation** (preventing UI thread lockups from rapid `setState` calls), **distributed conflict resolution** (merging concurrent edits without data loss using CRDTs or OT), and **optimistic merging** (reconciling instant client-side predictions with delayed server acknowledgments). React 19's `useOptimistic` hook and `useSyncExternalStore` provide modern primitives for these patterns, while libraries like Yjs, Liveblocks, and CloudSignal abstract away the most complex parts. The core architectural principle is **decoupling data ingestion from rendering**: incoming events are buffered outside React state (in refs or external stores) and flushed to the UI at a controlled cadence (typically once per animation frame).

**Beginner-Friendly Explanation:** Imagine you're building a live dashboard that shows stock prices updating 50 times per second, and you also want multiple people to collaborate on it with live cursors. If you naively call `setState` on every update, React will choke and the page will freeze. Real-time UI patterns solve this by buffering updates and only rendering what the user can actually perceive. For collaboration, you need to handle the fact that two people might edit the same thing at the same time — that's where conflict resolution comes in. And for actions like sending a chat message, you want it to appear instantly even though the server hasn't confirmed it yet — that's optimistic merging.

### Key Characteristics

- **Perception-Limited Rendering:** The human eye can perceive roughly 60 frames per second. Rendering more frequently than that wastes CPU and causes jank. Throttling to `requestAnimationFrame` aligns updates with the browser's refresh cycle.
- **Decoupled Ingestion and Rendering:** High-frequency events are captured in mutable refs or external stores, then flushed to React state at a controlled rate. This prevents React's reconciler from being overwhelmed.
- **Conflict-Free by Design:** CRDTs (Conflict-free Replicated Data Types) such as Yjs guarantee that concurrent edits merge deterministically without a central coordinator, enabling offline-first and peer-to-peer collaboration.
- **Presence is Ephemeral:** Presence data (who is online, cursor positions) is inherently temporary. It should be stored separately from persistent document state and expire via heartbeats when users disconnect without a clean close.
- **Optimistic Merging Over Replacement:** When the server's canonical version of a row arrives, it should merge field-by-field with the optimistic ghost (same ID) rather than replace it wholesale, avoiding visual flashes.

### Prerequisites

- Solid understanding of React Hooks (`useState`, `useRef`, `useEffect`, `useSyncExternalStore`, `useOptimistic`).
- Familiarity with WebSocket APIs and real-time communication patterns.
- Basic understanding of distributed systems concepts (eventual consistency, conflict resolution).
- Knowledge of `requestAnimationFrame` and browser rendering cycles.
- Awareness of React 18/19 concurrent features (`useTransition`, `startTransition`).

### Related Programming Areas

- **CRDTs and Operational Transformation:** Mathematical models for conflict-free concurrent editing.
- **WebSocket Lifecycles:** Connection management, reconnection, and heartbeat mechanisms.
- **Optimistic UI:** Instant feedback patterns for mutations.
- **State Management:** External stores, selectors, and subscription patterns.
- **Presence Systems:** Ephemeral state tracking with heartbeats and expiration.
- **Performance Engineering:** Frame budgeting, throttling, and debouncing.

### Core Concepts / Features

1. Collaborative & Presence Interfaces
2. High-Frequency State Synchronization
3. Conflict Resolution & Distributed State
4. Optimistic Data Merging

---

## Core Concept 1: Collaborative & Presence Interfaces

### Definitions

**Core Definition:** Collaborative and presence interfaces are UI patterns that show other users' live cursors, selections, and online status in a shared workspace, enabling real-time multi-user interaction.

**Technical Definition:** Presence interfaces track ephemeral per-user state — cursor position, text selection, active component focus, typing indicators, and online/offline status — and broadcast it to all participants in a shared room. Presence data is **ephemeral**: it is not persisted and is discarded when the user disconnects. The architecture typically involves a WebSocket server that maintains a room roster, a presence protocol (often using awareness APIs from CRDT libraries like Yjs's `awareness`), and heartbeat mechanisms to detect ghost connections. Presence is stored separately from document state: Yjs's `Awareness` protocol keeps presence outside the CRDT document, so it does not grow the document or affect conflict resolution. The React client renders cursors as SVG overlays positioned absolutely over the shared canvas or content area.

**Beginner-Friendly Explanation:** Presence is the "who's here and what are they doing" layer of collaboration. When you see someone else's cursor moving in a Figma or Google Docs document, that's presence. It's not part of the document itself — it's a separate stream of temporary information. If someone closes their laptop without logging out, their cursor should disappear after a few seconds, which is why heartbeats matter. Presence is always ephemeral: you never want to store it permanently.

### Purposes

- To show users who else is currently active in a shared workspace.
- To render live cursors and selections so collaborators can see each other's focus.
- To display typing indicators and component-level presence (who is editing which field).
- To track online/offline status with heartbeat-based expiration for ghost connection detection.
- To provide an avatar stack showing the count and identity of active participants.
- To enable contextual presence (e.g., "Alice is editing the title field").

### Syntax Rules and Structure

**General Syntax with Yjs Awareness:**

```tsx
import { useEffect, useState } from 'react';
import * as Y from 'yjs';
import { WebsocketProvider } from 'y-websocket';

function usePresence(roomId: string, user: { name: string; color: string }) {
  const [awareness, setAwareness] = useState(null);

  useEffect(() => {
    const doc = new Y.Doc();
    const provider = new WebsocketProvider(
      'wss://demos.yjs.dev',
      roomId,
      doc
    );

    // Set local presence state
    provider.awareness.setLocalStateField('user', user);
    setAwareness(provider.awareness);

    return () => {
      provider.awareness.setLocalState(null);
      provider.destroy();
    };
  }, [roomId, user]);

  return awareness;
}
```

**Component Breakdown:**
- `new Y.Doc()`: Creates a CRDT document.
- `WebsocketProvider`: Connects the document to a WebSocket server and manages awareness.
- `provider.awareness.setLocalStateField('user', user)`: Sets the local user's presence data.
- `provider.awareness.setLocalState(null)`: Clears presence on cleanup, broadcasting the user's departure.
- `provider.destroy()`: Closes the WebSocket connection and cleans up resources.

**General Syntax for Rendering Remote Cursors:**

```tsx
function CursorOverlay({ awareness }) {
  const [cursors, setCursors] = useState([]);

  useEffect(() => {
    if (!awareness) return;

    const updateCursors = () => {
      const states = Array.from(awareness.getStates().entries());
      const remote = states
        .filter(([clientId]) => clientId !== awareness.clientID)
        .map(([clientId, state]) => ({
          clientId,
          user: state.user,
          cursor: state.cursor,
        }));
      setCursors(remote);
    };

    awareness.on('change', updateCursors);
    updateCursors();

    return () => awareness.off('change', updateCursors);
  }, [awareness]);

  return (
    <div className="cursor-overlay">
      {cursors.map(({ clientId, user, cursor }) =>
        cursor ? (
          <div
            key={clientId}
            className="remote-cursor"
            style={{
              transform: `translate(${cursor.x}px, ${cursor.y}px)`,
              '--cursor-color': user.color,
            }}
          >
            <svg viewBox="0 0 24 24" width="20" height="20">
              <path d="M5 3l14 9-6 1-4 7z" fill="currentColor" />
            </svg>
            <span className="cursor-label">{user.name}</span>
          </div>
        ) : null
      )}
    </div>
  );
}
```

**Component Breakdown:**
- `awareness.getStates()`: Returns a Map of client IDs to their presence states.
- `awareness.clientID`: The local client's ID, filtered out to avoid rendering your own cursor.
- `awareness.on('change', ...)`: Subscribes to presence changes.
- The cursor is rendered as an SVG arrow with a label, positioned using `transform: translate()`.

**Heartbeat-Based Presence Expiration (Server-Side Concept):**

```javascript
// Server-side presence with Redis sorted set
// Heartbeat every 5 seconds, prune entries older than 30 seconds
const PRESENCE_KEY = `presence:room:${roomId}`;

async function heartbeat(clientId) {
  await redis.zadd(PRESENCE_KEY, Date.now(), clientId);
}

async function pruneStale() {
  const cutoff = Date.now() - 30000; // 30 seconds
  await redis.zremrangebyscore(PRESENCE_KEY, 0, cutoff);
}

// Run pruneStale every 5 seconds
setInterval(pruneStale, 5000);
```

**Component Breakdown:**
- `redis.zadd(key, score, member)`: Adds or updates a client's heartbeat timestamp in a sorted set.
- `redis.zremrangebyscore(key, 0, cutoff)`: Removes entries with timestamps older than 30 seconds.
- The heartbeat timer refreshes each active client's timestamp; stale entries are pruned automatically.

**Syntax Rules:**
- Store presence data separately from document state — presence is ephemeral and should not grow the document.
- Use heartbeat intervals of 5–10 seconds with a TTL of 2.5–3× the interval for expiration.
- Filter out the local client ID when rendering remote cursors.
- Clean up presence on unmount by setting local state to `null` and destroying the provider.
- Use `transform: translate()` for cursor positioning rather than `top`/`left` for better performance.
- Throttle cursor updates to 30–60 FPS to avoid overwhelming the network.

**Constraints and Limitations:**
- Presence data is not persisted; it is lost when the user disconnects.
- Heartbeats consume network bandwidth and server resources; balance frequency against detection latency.
- Cursor rendering can be expensive with many users; throttle updates and consider hiding cursors when the count exceeds a threshold.
- Browser `navigator.onLine` is not reliable for presence detection; use heartbeat-based expiration.
- Yjs awareness protocol requires a compatible server (e.g., Hocuspocus, y-websocket).

### Annotated Code Examples

**Example 1: Complete Presence Hook with Yjs Awareness**

```tsx
import { useEffect, useState, useCallback } from 'react';
import * as Y from 'yjs';
import { WebsocketProvider } from 'y-websocket';

function useCollaborativePresence(roomId, user) {
  const [awareness, setAwareness] = useState(null);
  const [remoteUsers, setRemoteUsers] = useState([]);

  useEffect(() => {
    const doc = new Y.Doc();
    const provider = new WebsocketProvider('wss://demos.yjs.dev', roomId, doc);

    provider.awareness.setLocalStateField('user', user);
    setAwareness(provider.awareness);

    const update = () => {
      const states = Array.from(provider.awareness.getStates().entries());
      const others = states
        .filter(([id]) => id !== provider.awareness.clientID)
        .map(([id, state]) => ({ id, ...state }));
      setRemoteUsers(others);
    };

    provider.awareness.on('change', update);
    update();

    return () => {
      provider.awareness.off('change', update);
      provider.awareness.setLocalState(null);
      provider.destroy();
    };
  }, [roomId, user.name, user.color]);

  const updateCursor = useCallback((x, y) => {
    if (awareness) {
      awareness.setLocalStateField('cursor', { x, y });
    }
  }, [awareness]);

  return { remoteUsers, updateCursor };
}
```

**Expected Output:** The hook returns a list of remote users (each with `id`, `user`, and `cursor`) and an `updateCursor` function. When the local user moves their mouse and calls `updateCursor`, other clients see the cursor move in real time.

**Why This Output Occurs:** Yjs awareness broadcasts the local state to all connected clients via the WebSocket provider. The `change` event fires whenever any client's awareness state changes. The hook filters out the local client ID and exposes remote users for rendering.

### Real-World Cases

- **Figma-like design tools:** Live cursors, selection highlights, and avatar stacks.
- **Google Docs-style editors:** Typing indicators, cursor presence, and user color coding.
- **Whiteboard applications:** Multi-user cursors with name labels and presence borders.
- **Project management tools:** "Who's viewing this task" indicators and avatar stacks.
- **Live support dashboards:** Agent presence and customer queue status.

---

## Core Concept 2: High-Frequency State Synchronization

### Definitions

**Core Definition:** High-frequency state synchronisation is the practice of buffering, throttling, and batching rapid data updates to prevent UI thread lockups while keeping the interface responsive and current.

**Technical Definition:** WebSocket feeds in fintech, gaming, and IoT dashboards can fire 50–100+ updates per second. Calling `setState` on every message causes React to re-render excessively, blocking the main thread and causing input lag. The solution is a **buffer-flush architecture**: incoming messages are stored in a mutable `useRef` buffer (outside React state), and a controlled flush mechanism (`requestAnimationFrame` or `setInterval`) moves the buffered data into React state at a limited rate (typically 10–60 FPS). For key-value data (e.g., stock tickers), a `Map` keyed by symbol keeps only the latest value per key, collapsing multiple updates into one render. `useSyncExternalStore` provides a tear-free way to subscribe to external high-frequency stores, with selector-based subscriptions ensuring that only components whose selected slice changed re-render.

**Beginner-Friendly Explanation:** If you have a stock ticker updating 100 times per second, rendering all 100 updates is pointless — the human eye can't see them. Instead, you collect all the updates in a buffer (a plain JavaScript object, not React state), and then every 16 milliseconds (once per frame) you take the latest value for each stock and update React state once. This way React only renders 60 times per second instead of 100, and the browser stays smooth. For key-value data, you keep a `Map` where each stock symbol maps to its latest price, so 10 updates to Apple stock in one frame become just one render.

### Purposes

- To prevent UI thread lockups caused by excessive `setState` calls from high-frequency feeds.
- To align data updates with the browser's refresh rate using `requestAnimationFrame`.
- To deduplicate rapid updates by key, keeping only the latest value per entity.
- To provide a tear-free subscription mechanism for external stores using `useSyncExternalStore`.
- To enable selector-based re-renders so only components whose data actually changed update.
- To maintain smooth animations and input responsiveness during heavy data streaming.

### Syntax Rules and Structure

**General Syntax with `use-latest-batch`:**

```tsx
import { useLatestBatch } from 'use-latest-batch';
import { useWebSocket } from 'react-use-websocket';

function PriceGrid() {
  const { lastJsonMessage } = useWebSocket('wss://feed.example.com/ticks');

  // Buffer updates, deduplicate by symbol, flush at 10 FPS
  const rows = useLatestBatch(lastJsonMessage, {
    keyField: 'symbol',
    fps: 10,
  });

  return (
    <table>
      <tbody>
        {rows.map((row) => (
          <tr key={row.symbol}>
            <td>{row.symbol}</td>
            <td>{row.price}</td>
          </tr>
        ))}
      </tbody>
    </table>
  );
}
```

**Component Breakdown:**
- `lastJsonMessage`: The latest WebSocket message (fires on every message).
- `useLatestBatch(lastJsonMessage, { keyField: 'symbol', fps: 10 })`: Buffers messages in a ref, keeps only the latest value per `symbol`, and flushes to React state at 10 FPS.
- `rows`: The current state, always ≤ 10 renders per second.

**General Syntax with `requestAnimationFrame` Batching:**

```tsx
function useRafBatch(initialValue = []) {
  const [state, setState] = useState(initialValue);
  const bufferRef = useRef([]);
  const rafRef = useRef(null);

  const push = useCallback((item) => {
    bufferRef.current.push(item);

    if (rafRef.current === null) {
      rafRef.current = requestAnimationFrame(() => {
        setState(bufferRef.current);
        bufferRef.current = [];
        rafRef.current = null;
      });
    }
  }, []);

  useEffect(() => {
    return () => {
      if (rafRef.current !== null) cancelAnimationFrame(rafRef.current);
    };
  }, []);

  return { state, push };
}
```

**Component Breakdown:**
- `bufferRef`: A mutable ref that holds incoming items outside React state.
- `rafRef`: Tracks the pending `requestAnimationFrame` ID.
- `push(item)`: Adds an item to the buffer and schedules a flush if none is pending.
- The flush callback moves the buffered items into React state and clears the buffer.

**General Syntax with `useSyncExternalStore` for External Stores:**

```tsx
import { useSyncExternalStore } from 'react';

// External store (outside React)
const priceStore = {
  prices: new Map(),
  listeners: new Set(),
  subscribe(listener) {
    this.listeners.add(listener);
    return () => this.listeners.delete(listener);
  },
  getSnapshot() {
    return this.prices;
  },
  update(symbol, price) {
    this.prices.set(symbol, price);
    this.listeners.forEach((l) => l());
  },
};

// Component with selector-based subscription
function usePrice(symbol) {
  const prices = useSyncExternalStore(
    priceStore.subscribe.bind(priceStore),
    priceStore.getSnapshot.bind(priceStore)
  );
  return prices.get(symbol);
}
```

**Component Breakdown:**
- `priceStore`: A plain JavaScript store outside React.
- `subscribe(listener)`: Registers a listener and returns an unsubscribe function.
- `getSnapshot()`: Returns the current snapshot (the `Map` of prices).
- `useSyncExternalStore(subscribe, getSnapshot)`: Subscribes the component to the store and re-renders when the snapshot changes.
- `prices.get(symbol)`: Selects only the specific price the component needs.

**Syntax Rules:**
- Never call `setState` on every WebSocket message; buffer messages in a `useRef`.
- Flush the buffer at a controlled rate: `requestAnimationFrame` (60 FPS), `setInterval` (e.g., 100ms), or a library like `use-latest-batch`.
- Deduplicate by key: use a `Map` to keep only the latest value per entity.
- Use `useSyncExternalStore` for external stores that update outside React.
- With `useSyncExternalStore`, `getSnapshot` must return a cached immutable value — returning a new object on every call causes infinite loops.
- Use selector-based subscriptions to ensure only components whose selected slice changed re-render.
- Set a minimum flush interval (16ms) to prevent runaway timers.

**Constraints and Limitations:**
- `useSyncExternalStore` cannot be used with Suspense in a way that suspends based on store values; mutations to external stores trigger blocking updates.
- Over-throttling (e.g., 1 FPS) makes the UI feel sluggish; balance freshness against performance.
- `requestAnimationFrame` does not run when the tab is backgrounded; use `setInterval` for background-safe updates.
- Selector equality checks must be shallow; deeply nested comparisons are expensive.
- External stores bypass React's batching; use `startTransition` for non-urgent updates if needed.

### Annotated Code Examples

**Example 1: High-Frequency Stock Ticker with `use-latest-batch`**

```tsx
import { useWebSocket } from 'react-use-websocket';
import { useLatestBatch } from 'use-latest-batch';

function StockTicker() {
  const { lastJsonMessage } = useWebSocket('wss://feed.example.com/stocks');

  const rows = useLatestBatch(lastJsonMessage, {
    keyField: 'symbol',
    fps: 10,
  });

  return (
    <table>
      <thead>
        <tr><th>Symbol</th><th>Price</th><th>Change</th></tr>
      </thead>
      <tbody>
        {rows.map((row) => (
          <tr key={row.symbol}>
            <td>{row.symbol}</td>
            <td>{row.price.toFixed(2)}</td>
            <td style={{ color: row.change >= 0 ? 'green' : 'red' }}>
              {row.change >= 0 ? '+' : ''}{row.change.toFixed(2)}
            </td>
          </tr>
        ))}
      </tbody>
    </table>
  );
}
```

**Expected Output:** The table displays stock symbols, prices, and changes. The WebSocket fires at 50+ messages per second, but the table only re-renders 10 times per second (10 FPS), with each render showing the latest price per symbol.

**Why This Output Occurs:** `useLatestBatch` buffers incoming messages in a `ref`, keeps only the latest value per `symbol` in a `Map`, and flushes to React state at a controlled FPS. Multiple updates to the same symbol within a frame are collapsed into a single render.

### Real-World Cases

- **Fintech dashboards:** Stock tickers, crypto prices, and order book updates.
- **Gaming:** Real-time score updates, player positions, and leaderboards.
- **IoT monitoring:** Sensor data streams from thousands of devices.
- **Analytics dashboards:** Live user counts, revenue metrics, and conversion rates.
- **Sports betting:** Live odds updates and event feeds.

---

## Core Concept 3: Conflict Resolution & Distributed State

### Definitions

**Core Definition:** Conflict resolution and distributed state is the practice of enabling multiple users to edit shared data concurrently without data loss, using mathematical models (CRDTs or OT) that guarantee eventual consistency.

**Technical Definition:** Two primary techniques exist for conflict resolution. **Operational Transformation (OT)** transforms operations against concurrent operations to maintain document consistency, assuming a central server to order and transform operations. **Conflict-free Replicated Data Types (CRDTs)** use mathematically structured data types (e.g., grow-only sets, last-writer-wins registers, sequence CRDTs) that merge deterministically without a central coordinator, enabling peer-to-peer and offline-first collaboration. Yjs is the dominant CRDT implementation for React, providing `Y.Doc` (the shared document), `Y.Text`, `Y.Array`, and `Y.Map` (shared types), and `Awareness` (ephemeral presence). OT was invented in the late 1980s and remains the choice for the vast majority of production co-editors (e.g., Google Docs), while CRDTs are preferred for offline-first, peer-to-peer, and local-first applications.

**Beginner-Friendly Explanation:** Imagine two people typing in the same sentence at the same time. One types "hello" and the other types "world" — what should the final sentence be? OT and CRDT are two mathematical approaches to solving this. OT asks a central server to "fix up" the operations so they work together. CRDTs are designed so that no matter what order the edits arrive in, everyone ends up with the same result. CRDTs are better for offline work and peer-to-peer, but OT is still used by most big products because it's been battle-tested for decades.

### Purposes

- To enable multiple users to edit the same document concurrently without data loss.
- To guarantee eventual consistency across all replicas (every client ends up with the same state).
- To support offline editing with automatic merging on reconnection.
- To eliminate the need for a central server to order operations (CRDTs).
- To provide a mathematical foundation for collaborative applications (shared text, arrays, maps).
- To enable peer-to-peer collaboration without a coordinating server.

### Syntax Rules and Structure

**General Syntax with Yjs (CRDT):**

```tsx
import { useEffect, useState } from 'react';
import * as Y from 'yjs';
import { WebsocketProvider } from 'y-websocket';

function useYText(roomId, fieldName) {
  const [text, setText] = useState('');
  const [doc, setDoc] = useState(null);

  useEffect(() => {
    const ydoc = new Y.Doc();
    const provider = new WebsocketProvider('wss://demos.yjs.dev', roomId, ydoc);

    const ytext = ydoc.getText(fieldName);
    setDoc(ydoc);

    const update = () => setText(ytext.toString());
    ytext.observe(update);
    update();

    return () => {
      ytext.unobserve(update);
      provider.destroy();
    };
  }, [roomId, fieldName]);

  const insert = (index, content) => {
    if (doc) doc.getText(fieldName).insert(index, content);
  };

  return { text, insert };
}
```

**Component Breakdown:**
- `new Y.Doc()`: Creates a CRDT document.
- `ydoc.getText(fieldName)`: Gets or creates a shared `Y.Text` type.
- `ytext.observe(update)`: Subscribes to changes on the shared text.
- `ytext.insert(index, content)`: Inserts content at a specific position.
- `provider.destroy()`: Cleans up the WebSocket connection.

**General Syntax for a Collaborative React Editor (with Yjs + Lexical):**

```tsx
import { LexicalComposer } from '@lexical/react/LexicalComposer';
import { RichTextPlugin } from '@lexical/react/LexicalRichTextPlugin';
import { CollaborationPlugin } from '@lexical/react/LexicalCollaborationPlugin';
import * as Y from 'yjs';

function CollaborativeEditor({ roomId, user }) {
  return (
    <LexicalComposer initialConfig={{ namespace: 'MyEditor' }}>
      <RichTextPlugin contentEditable={null} />
      <CollaborationPlugin
        id={roomId}
        providerFactory={(id, yjsDocMap) => {
          const doc = new Y.Doc();
          yjsDocMap.set(id, doc);
          return new WebsocketProvider('wss://demos.yjs.dev', id, doc);
        }}
        shouldBootstrap={true}
      />
    </LexicalComposer>
  );
}
```

**Component Breakdown:**
- `CollaborationPlugin`: Integrates Yjs with Lexical for collaborative editing.
- `providerFactory`: Creates a Yjs document and WebSocket provider for the given room.
- `shouldBootstrap`: Whether the editor should bootstrap the Yjs document if it is empty.

**OT vs CRDT Comparison:**

| Aspect | OT (Operational Transformation) | CRDT (Conflict-free Replicated Data Type) |
|--------|--------------------------------|-------------------------------------------|
| **Coordination** | Requires central server | No central coordinator required |
| **Offline support** | Limited | Excellent (merges on reconnect) |
| **Peer-to-peer** | Not supported | Supported |
| **Complexity** | High (transformation functions) | High (mathematical structure) |
| **Maturity** | Decades of production use (Google Docs) | Newer, but maturing rapidly |
| **Use case** | Centralised co-editing | Local-first, offline, P2P |

**Syntax Rules:**
- Use CRDTs (Yjs) for offline-first, peer-to-peer, or local-first applications.
- Use OT for centralised co-editing with a server you control.
- Always observe shared types (e.g., `ytext.observe(callback)`) and unobserve in cleanup.
- Never mutate Yjs types directly; use the provided methods (`insert`, `delete`, `set`).
- Use `Awareness` for presence, not the document itself — presence is ephemeral.
- Wrap Yjs mutations in `doc.transact(() => { ... })` for atomic operations.
- Destroy the provider and unobserve listeners on unmount.

**Constraints and Limitations:**
- Yjs and CRDTs add bundle size and memory overhead.
- OT is complex to implement correctly; use an existing library or service.
- CRDTs can grow document size over time; garbage collection may be needed.
- Yjs requires a compatible server (Hocuspocus, y-websocket) for real-time sync.
- OT assumes a central server; it is not suitable for peer-to-peer applications.

### Annotated Code Examples

**Example 1: Collaborative Text Editor with Yjs**

```tsx
import { useEffect, useState, useRef } from 'react';
import * as Y from 'yjs';
import { WebsocketProvider } from 'y-websocket';

function CollaborativeTextEditor({ roomId, user }) {
  const [text, setText] = useState('');
  const docRef = useRef(null);

  useEffect(() => {
    const doc = new Y.Doc();
    docRef.current = doc;
    const provider = new WebsocketProvider('wss://demos.yjs.dev', roomId, doc);
    provider.awareness.setLocalStateField('user', user);

    const ytext = doc.getText('content');
    const update = () => setText(ytext.toString());
    ytext.observe(update);
    update();

    return () => {
      ytext.unobserve(update);
      provider.destroy();
      doc.destroy();
    };
  }, [roomId, user.name]);

  function handleChange(e) {
    const ytext = docRef.current.getText('content');
    ytext.delete(0, ytext.length);
    ytext.insert(0, e.target.value);
  }

  return (
    <textarea value={text} onChange={handleChange} />
  );
}
```

**Expected Output:** Two users in the same room can type in the textarea simultaneously. Both see the same final text, with conflicts resolved automatically by Yjs.

**Why This Output Occurs:** Yjs's `Y.Text` is a sequence CRDT that merges concurrent insertions and deletions deterministically. When two users type at the same time, Yjs applies both operations and converges to the same result on both clients. The `observe` callback fires on every change, updating the React state.

### Real-World Cases

- **Google Docs-style editors:** Collaborative text editing with conflict-free merging.
- **Code editors:** Multi-user code editing with cursor presence (e.g., VS Code Live Share).
- **Note-taking apps:** Offline-first note editing that syncs when connectivity returns.
- **Whiteboards:** Collaborative diagramming with shared shapes and text.
- **Project management:** Concurrent task editing with field-level conflict resolution.

---

## Core Concept 4: Optimistic Data Merging

### Definitions

**Core Definition:** Optimistic data merging is the practice of rendering the predicted result of a mutation instantly while reconciling the server's canonical response field-by-field when it arrives, avoiding visual flashes and duplicate rows.

**Technical Definition:** Optimistic UI has two phases: **prediction** (the client renders an optimistic ghost of the result before the server confirms) and **reconciliation** (the server's canonical row arrives over the WebSocket and merges with the ghost). The naive approach — replacing the ghost with the server row — causes a flash because the row is deleted and re-inserted. The correct approach is **field-level merging**: the optimistic ghost and the canonical row share the same ID, so the WebSocket broadcast is treated as an idempotent merge (same `row_id`, fields refreshed) rather than a delete-then-replace. React 19's `useOptimistic` hook provides the client-side primitive for this pattern; frameworks like Pylon provide server-side support for accepting client-minted optimistic IDs and treating the resulting broadcast as a merge. On rejection (e.g., a 409 conflict), the optimistic ghost is rolled back without leaving a tombstone, so retrying the mutation works.

**Beginner-Friendly Explanation:** When you send a chat message, it should appear instantly in the conversation. But the server also needs to create the message in the database and broadcast it to other users. If you just replace your local message with the server's version, there's a brief flash where the message disappears and reappears. Optimistic merging fixes this by giving the optimistic message the same ID as the server's message, so when the server's version arrives, it merges field-by-field instead of replacing. The user never sees a flash.

### Purposes

- To provide instant visual feedback for mutations without waiting for server acknowledgment.
- To avoid visual flashes caused by delete-then-replace reconciliation.
- To merge server-authoritative fields (e.g., timestamps, server-generated IDs) with client predictions.
- To roll back optimistic updates cleanly on server rejection (409 conflict).
- To enable retry logic after failed mutations without leaving orphaned ghost rows.
- To integrate with React 19's `useOptimistic` for declarative optimistic UI.

### Syntax Rules and Structure

**General Syntax with React 19's `useOptimistic`:**

```tsx
'use client';
import { useOptimistic } from 'react';

function Chat({ messages, sendMessage }) {
  const [optimisticMessages, addOptimisticMessage] = useOptimistic(
    messages,
    (currentMessages, newMessage) => [
      ...currentMessages,
      { id: `temp-${Date.now()}`, text: newMessage, sending: true },
    ]
  );

  async function handleSend(formData) {
    const text = formData.get('message');
    addOptimisticMessage(text);
    await sendMessage(text);
  }

  return (
    <div>
      <ul>
        {optimisticMessages.map((msg) => (
          <li key={msg.id} style={{ opacity: msg.sending ? 0.6 : 1 }}>
            {msg.text} {msg.sending && '(sending...)'}
          </li>
        ))}
      </ul>
      <form action={handleSend}>
        <input name="message" />
        <button type="submit">Send</button>
      </form>
    </div>
  );
}
```

**Component Breakdown:**
- `useOptimistic(messages, updateFn)`: Returns the optimistic message list and a dispatch function.
- `addOptimisticMessage(text)`: Appends a temporary message with `sending: true`.
- `sendMessage(text)`: The server action; when it resolves, the real messages array replaces the optimistic one.
- The optimistic message has `opacity: 0.6` while sending.

**General Syntax for Field-Level Merging (Server-Side Concept):**

```javascript
// Server: accept client-minted optimistic ID
async function sendMessage(args) {
  const message = await db.insert('Message', {
    id: args._optimisticId, // Use the client-minted ID
    body: args.body,
    authorId: ctx.userId,
    createdAt: new Date().toISOString(),
  });

  // Broadcast to all clients — the client treats this as a merge (same id)
  broadcast('message:new', message);
  return message;
}
```

**Component Breakdown:**
- `args._optimisticId`: The client-minted temporary ID.
- `db.insert('Message', { id: args._optimisticId, ... })`: Uses the client's ID as the canonical ID.
- `broadcast('message:new', message)`: Sends the canonical message to all clients.
- Clients treat the broadcast as an idempotent merge because the `id` matches the optimistic ghost.

**General Syntax for Rolling Back on Conflict:**

```javascript
// Server returns 409 Conflict with version token
if (currentVersion !== expectedVersion) {
  return { error: 'CONFLICT', serverVersion: currentVersion };
}

// Client rolls back optimistic ghost on rejection
try {
  await sendMessage(text);
} catch (error) {
  if (error.code === 'CONFLICT') {
    // The optimistic ghost is automatically discarded by useOptimistic
    // Show a conflict resolution UI
  }
}
```

**Component Breakdown:**
- `currentVersion !== expectedVersion`: Detects a version conflict.
- `return { error: 'CONFLICT', serverVersion: currentVersion }`: Returns a conflict response.
- `useOptimistic` automatically discards the optimistic state when the real state updates or the action fails.

**Syntax Rules:**
- Use `useOptimistic` for client-side optimistic predictions.
- Mint a client-side temporary ID and pass it to the server as `_optimisticId`.
- The server should use the client's ID as the canonical ID to enable field-level merging.
- Broadcast the canonical row over WebSocket; clients treat it as an idempotent merge (same ID).
- On 409 conflict, roll back the optimistic ghost and present a resolution UI.
- Never replace the optimistic ghost with the server row — merge field-by-field.
- Use `opacity` or a subtle visual indicator (e.g., "(sending...)") to distinguish optimistic from confirmed rows.

**Constraints and Limitations:**
- `useOptimistic` does not persist across navigation or page reloads.
- Client-minted IDs must be validated by the server (e.g., 40-char hex) to prevent collisions.
- If two clients mint the same ID, the second insert fails with a conflict error.
- Field-level merging requires the server and client to agree on the ID format and merge semantics.
- Optimistic updates can cause layout shift if the server's fields differ significantly from the prediction.
- Race conditions can occur if multiple optimistic updates target the same row; use `startTransition` or a mutation queue.

### Annotated Code Examples

**Example 1: Optimistic Chat Message with Field-Level Merging**

```tsx
'use client';
import { useOptimistic, useRef } from 'react';
import { sendMessage } from './actions';

function ChatRoom({ messages }) {
  const formRef = useRef(null);

  const [optimisticMessages, addOptimisticMessage] = useOptimistic(
    messages,
    (current, newText) => [
      ...current,
      {
        id: `temp-${crypto.randomUUID()}`,
        text: newText,
        sending: true,
        createdAt: new Date().toISOString(),
      },
    ]
  );

  async function handleSend(formData) {
    const text = formData.get('message');
    formRef.current?.reset();
    addOptimisticMessage(text);
    await sendMessage(text);
  }

  return (
    <div>
      <ul>
        {optimisticMessages.map((msg) => (
          <li key={msg.id} style={{ opacity: msg.sending ? 0.6 : 1 }}>
            <strong>{msg.text}</strong>
            {msg.sending && <span> (sending...)</span>}
          </li>
        ))}
      </ul>
      <form ref={formRef} action={handleSend}>
        <input name="message" placeholder="Type a message..." />
        <button type="submit">Send</button>
      </form>
    </div>
  );
}
```

**Expected Output:** When the user sends a message, it appears instantly in the list with reduced opacity and "(sending...)". When the server confirms and broadcasts the canonical message (with the same ID), the message becomes fully opaque and the "(sending...)" label disappears — without a flash.

**Why This Output Occurs:** `useOptimistic` appends a temporary message with a client-minted ID. When the server action completes, the real messages array (from the server) replaces the optimistic one. Because the server uses the same ID for the canonical message, the reconciliation is a field-level merge, not a delete-then-replace.

### Real-World Cases

- **Chat applications:** Instant message sending with "sending..." indicators and no flash on confirmation.
- **Social media:** Optimistic likes, comments, and shares with rollback on failure.
- **E-commerce:** Optimistic "Add to Cart" with immediate cart count updates.
- **Collaborative editing:** Optimistic cursor and selection updates with server reconciliation.
- **Notification panes:** Instant notification dismissal with optimistic removal and rollback on error.

---

## References

- @cloudsignal/collaborate – npm: https://www.npmjs.com/package/@cloudsignal/collaborate
- Realtime Cursor – Supabase Docs: https://supabase.com/docs/guides/realtime/realtime-cursor
- New React components for realtime presence – Liveblocks Blog: https://liveblocks.io/blog/new-react-components-for-realtime-presence
- Build Notion-style real-time presence with WebSockets on Vercel – Vercel KB: https://vercel.com/kb/guide/real-time-presence-hono-react
- Real Differences between OT and CRDT – arXiv: https://arxiv.org/pdf/1905.01517v1
- useOptimistic – React: https://react.dev/reference/react/useOptimistic
- Optimistic updates – Pylon Docs: https://docs.pylonsync.com/concepts/optimistic-updates
- use-latest-batch – Socket.dev: https://socket.dev/npm/package/use-latest-batch
- Streaming Backends & React: Controlling Re-render Chaos – SitePoint: https://www.sitepoint.com/streaming-backends-react-high-frequency-data/
- useSyncExternalStore – React: https://react.dev/reference/react/useSyncExternalStore
- react-frame-throttle – npm: https://www.npmjs.com/package/react-frame-throttle
- Real-Time Collaboration in React Block Editor – Syncfusion: https://ej2.syncfusion.com/react/documentation/block-editor/collaborative-editing
- Collaborative Editing in React: CRDTs, Yjs and Architecture – Makers' Den: https://makersden.io/blog/collaborative-editing-react-crdt-yjs
- React 19 useOptimistic – StackBlitz: https://stackblitz.com/edit/react-19-useoptimistic
- Yjs Documentation: https://docs.yjs.dev/
- Hocuspocus – Tiptap Collaboration Docs: https://tiptap.dev/docs/collaboration/getting-started/overview