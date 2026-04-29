# MinnalDB Electron Example

A task manager desktop app built with [minnaldb](https://github.com/sanishchirayath1/minnaldb), Electron, and React. Demonstrates reactive SQLite queries over Electron IPC with full CRUD, transactions, and live-updating UI.

## Prerequisites

- **Node.js 18–22** (Node 23+ is not supported — `better-sqlite3` lacks prebuilt binaries for newer versions)
- npm

If you use nvm:

```bash
nvm install 22
nvm use 22
```

## Setup

```bash
git clone <repo-url>
cd minnaldb-electron-example
npm install
```

`npm install` automatically runs `electron-rebuild` via the `postinstall` script to rebuild the `better-sqlite3` native addon against Electron's Node headers. No manual step needed.

## Development

```bash
npm run dev
```

This starts electron-vite in dev mode with hot reload for the renderer process. The database is stored at:

- **macOS:** `~/Library/Application Support/minnaldb-electron-example/minnaldb-tasks.db`
- **Windows:** `%APPDATA%/minnaldb-electron-example/minnaldb-tasks.db`
- **Linux:** `~/.config/minnaldb-electron-example/minnaldb-tasks.db`

## Building Distributables

```bash
npm run dist          # Build for current platform
npm run dist:mac      # macOS .dmg (arm64 + x64)
npm run dist:win      # Windows .exe (NSIS installer)
npm run dist:linux    # Linux .AppImage
```

Output goes to the `release/` directory.

> **Note:** macOS builds will show a "code signing skipped" warning unless you have an Apple Developer ID certificate. The app still runs — right-click > Open to bypass Gatekeeper.

## Project Structure

```
src/
├── main/index.ts           # Electron main process — creates DB, exposes via IPC
├── preload/index.ts        # Preload script — installs the IPC bridge
├── shared/schema.ts        # Table definitions (projects, tasks) — shared by main & renderer
└── renderer/
    ├── index.html
    └── src/
        ├── main.tsx        # React entry point
        ├── db.ts           # connectDB() — renderer-side remote DB client
        └── App.tsx         # Task manager UI with useQuery/useMutation
```

## How It Works

| Layer | Package | Role |
|-------|---------|------|
| Main process | `minnaldb` | Creates the SQLite database, runs queries |
| IPC bridge | `minnaldb-electron` | Exposes DB to renderer via Electron IPC |
| Renderer | `minnaldb-react` | `useQuery()` and `useMutation()` React hooks |

1. **Main process** opens the database with `createDB()` and calls `exposeDB(db, ipcMain)` to register IPC handlers.
2. **Preload script** calls `exposeMinnaldbBridge()` to set up `contextBridge` between main and renderer.
3. **Renderer** calls `connectDB(schema)` to get a remote DB client that mirrors the in-process API. Queries are compiled locally, only the SQL + params cross IPC.
4. **React hooks** (`useQuery`, `useMutation`) subscribe to queries. When a mutation fires, the main process re-runs affected subscriptions and pushes updated results to the renderer — but only if the data actually changed (deep equality check).

## Other Scripts

```bash
npm run build         # Build without packaging (electron-vite only)
npm run start         # Preview the production build locally
npm run typecheck     # Run TypeScript type checking
```

## License

MIT
