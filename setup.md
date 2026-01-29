# Ombori Hackathon Bootstrap (Windows + Electron React)

Execute this setup completely. Ask me for a project name first, then proceed autonomously.

## REQUIREMENTS
- You must be a member of the `ombori-hackathon` GitHub organization
- Windows 10/11 (this bootstrap is optimized for PC)
- Internet connection

## RULES
1. Use exact versions specified below - DO NOT research or guess
2. Execute commands sequentially in the order given
3. Create ALL files with the EXACT content shown
4. Ask for project name FIRST, then proceed without stopping
5. NEVER ask the user to run commands manually - execute everything yourself
6. Install missing dependencies automatically without asking for confirmation
7. BEFORE deleting any folder, always `cd` to workspace root first (prevents shell from breaking)

---

## STEP 1: Get Project Name

Ask me for a project name (lowercase, hyphens ok, no spaces). Example: `team-awesome`

This creates repos:
- `{name}-ws` (workspace)
- `{name}-desktop` (Electron React client)
- `{name}-api` (Python backend)

---

## STEP 2: Install & Verify Prerequisites

Check each tool and install if missing. Run these sequentially:

### 2.1 Install winget (Windows Package Manager)
winget comes pre-installed on Windows 11 and recent Windows 10 builds. Verify it exists:
```powershell
winget --version
```

If not available, install it from the Microsoft Store (App Installer).

### 2.2 Install Git (if missing)
```powershell
where git
if ($LASTEXITCODE -ne 0) {
    Write-Host "Installing Git..."
    winget install Git.Git -e --silent
    $env:Path = [System.Environment]::GetEnvironmentVariable("Path","Machine") + ";" + [System.Environment]::GetEnvironmentVariable("Path","User")
}
git --version
```

### 2.3 Install GitHub CLI (if missing)
```powershell
where gh
if ($LASTEXITCODE -ne 0) {
    Write-Host "Installing GitHub CLI..."
    winget install GitHub.cli -e --silent
    $env:Path = [System.Environment]::GetEnvironmentVariable("Path","Machine") + ";" + [System.Environment]::GetEnvironmentVariable("Path","User")
}
gh --version
```

### 2.4 Authenticate GitHub CLI
```powershell
# Check if authenticated
gh auth status
if ($LASTEXITCODE -ne 0) {
    Write-Host "Please authenticate with GitHub..."
    gh auth login
}

# Configure git to use gh credentials (prevents 403 errors on org repos)
gh auth setup-git

# Verify access to ombori-hackathon org
$result = gh api orgs/ombori-hackathon 2>$null
if ($LASTEXITCODE -eq 0) {
    Write-Host "Access to ombori-hackathon org confirmed"
} else {
    Write-Host "No access to ombori-hackathon org - contact organizer"
}
```

### 2.5 Install Docker Desktop (if missing) and ensure it's running
```powershell
where docker
if ($LASTEXITCODE -ne 0) {
    Write-Host "Installing Docker Desktop..."
    winget install Docker.DockerDesktop -e --silent
    Write-Host "Please restart your terminal after Docker installation completes"
}

# Check if Docker daemon is running
docker info 2>$null
if ($LASTEXITCODE -ne 0) {
    Write-Host "Starting Docker Desktop..."
    Start-Process "C:\Program Files\Docker\Docker\Docker Desktop.exe"
    Write-Host "Waiting for Docker to start..."
    while ($true) {
        docker info 2>$null
        if ($LASTEXITCODE -eq 0) { break }
        Start-Sleep -Seconds 2
    }
    Write-Host "Docker is running"
}

docker --version
docker compose version
```

### 2.6 Install Node.js (if missing)
```powershell
where node
if ($LASTEXITCODE -ne 0) {
    Write-Host "Installing Node.js LTS..."
    winget install OpenJS.NodeJS.LTS -e --silent
    $env:Path = [System.Environment]::GetEnvironmentVariable("Path","Machine") + ";" + [System.Environment]::GetEnvironmentVariable("Path","User")
}
node --version
npm --version
```

### 2.7 Install Python & uv (if missing)
```powershell
# Install Python 3.12+ if needed
where python
if ($LASTEXITCODE -ne 0) {
    Write-Host "Installing Python 3.12..."
    winget install Python.Python.3.12 -e --silent
    $env:Path = [System.Environment]::GetEnvironmentVariable("Path","Machine") + ";" + [System.Environment]::GetEnvironmentVariable("Path","User")
}
python --version

# Install uv
where uv
if ($LASTEXITCODE -ne 0) {
    Write-Host "Installing uv..."
    powershell -ExecutionPolicy ByPass -c "irm https://astral.sh/uv/install.ps1 | iex"
    $env:Path = [System.Environment]::GetEnvironmentVariable("Path","Machine") + ";" + [System.Environment]::GetEnvironmentVariable("Path","User")
}
uv --version
```

### 2.8 Final Check - All Prerequisites
```powershell
Write-Host "=== Prerequisites Check ==="
Write-Host "Git: $(git --version)"
Write-Host "GitHub CLI: $(gh --version | Select-Object -First 1)"
Write-Host "Docker: $(docker --version)"
Write-Host "Node.js: $(node --version)"
Write-Host "npm: $(npm --version)"
Write-Host "Python: $(python --version)"
Write-Host "uv: $(uv --version)"
Write-Host "=========================="
```

---

## STEP 3: Create Workspace Structure

The current folder becomes your workspace. Create the project structure:

```powershell
New-Item -ItemType Directory -Force -Path apps/desktop-client
New-Item -ItemType Directory -Force -Path services/api
```

---

## STEP 4: Create Electron React App

### 4.1 Initialize Electron React App

```powershell
cd apps/desktop-client
npm init -y
```

### 4.2 Install Dependencies

```powershell
npm install react react-dom electron
npm install -D @types/react @types/react-dom typescript electron-builder concurrently wait-on vite @vitejs/plugin-react
```

### 4.3 Create package.json

Replace the generated package.json with:

```json
{
  "name": "{name}-client",
  "version": "1.0.0",
  "description": "{name} - Electron React Desktop Client",
  "main": "dist-electron/main.js",
  "scripts": {
    "dev": "concurrently \"npm run dev:vite\" \"npm run dev:electron\"",
    "dev:vite": "vite",
    "dev:electron": "wait-on tcp:5173 && electron .",
    "build": "tsc && vite build && electron-builder",
    "build:vite": "vite build",
    "preview": "vite preview"
  },
  "dependencies": {
    "react": "^18.2.0",
    "react-dom": "^18.2.0"
  },
  "devDependencies": {
    "@types/react": "^18.2.0",
    "@types/react-dom": "^18.2.0",
    "@vitejs/plugin-react": "^4.2.0",
    "concurrently": "^8.2.0",
    "electron": "^28.0.0",
    "electron-builder": "^24.9.0",
    "typescript": "^5.3.0",
    "vite": "^5.0.0",
    "wait-on": "^7.2.0"
  },
  "build": {
    "appId": "com.ombori.{name}",
    "productName": "{Name}Client",
    "directories": {
      "output": "release"
    },
    "win": {
      "target": "nsis"
    }
  }
}
```

Note: `{Name}` is the project name in PascalCase (e.g., `team-awesome` → `TeamAwesome`)

### 4.4 Create tsconfig.json

```json
{
  "compilerOptions": {
    "target": "ES2020",
    "useDefineForClassFields": true,
    "lib": ["ES2020", "DOM", "DOM.Iterable"],
    "module": "ESNext",
    "skipLibCheck": true,
    "moduleResolution": "bundler",
    "allowImportingTsExtensions": true,
    "resolveJsonModule": true,
    "isolatedModules": true,
    "noEmit": true,
    "jsx": "react-jsx",
    "strict": true,
    "noUnusedLocals": true,
    "noUnusedParameters": true,
    "noFallthroughCasesInSwitch": true
  },
  "include": ["src"],
  "references": [{ "path": "./tsconfig.node.json" }]
}
```

### 4.5 Create tsconfig.node.json

```json
{
  "compilerOptions": {
    "composite": true,
    "skipLibCheck": true,
    "module": "ESNext",
    "moduleResolution": "bundler",
    "allowSyntheticDefaultImports": true,
    "outDir": "dist-electron"
  },
  "include": ["electron"]
}
```

### 4.6 Create vite.config.ts

```typescript
import { defineConfig } from 'vite'
import react from '@vitejs/plugin-react'

export default defineConfig({
  plugins: [react()],
  base: './',
  build: {
    outDir: 'dist'
  },
  server: {
    port: 5173
  }
})
```

### 4.7 Create Electron Main Process

Create `electron/main.ts`:

```typescript
import { app, BrowserWindow } from 'electron'
import path from 'path'

function createWindow() {
  const win = new BrowserWindow({
    width: 800,
    height: 600,
    webPreferences: {
      nodeIntegration: false,
      contextIsolation: true
    },
    titleBarStyle: 'hidden',
    titleBarOverlay: {
      color: '#1a1a1a',
      symbolColor: '#ffffff'
    }
  })

  // In development, load from Vite dev server
  if (process.env.NODE_ENV === 'development' || !app.isPackaged) {
    win.loadURL('http://localhost:5173')
    win.webContents.openDevTools()
  } else {
    // In production, load from built files
    win.loadFile(path.join(__dirname, '../dist/index.html'))
  }
}

app.whenReady().then(createWindow)

app.on('window-all-closed', () => {
  if (process.platform !== 'darwin') {
    app.quit()
  }
})

app.on('activate', () => {
  if (BrowserWindow.getAllWindows().length === 0) {
    createWindow()
  }
})
```

### 4.8 Create index.html

Create `index.html` in root:

```html
<!DOCTYPE html>
<html lang="en">
  <head>
    <meta charset="UTF-8" />
    <meta name="viewport" content="width=device-width, initial-scale=1.0" />
    <meta http-equiv="Content-Security-Policy" content="default-src 'self'; script-src 'self'; connect-src 'self' http://localhost:8000">
    <title>{Name} Client</title>
  </head>
  <body>
    <div id="root"></div>
    <script type="module" src="/src/main.tsx"></script>
  </body>
</html>
```

### 4.9 Create React Source Files

Create `src/main.tsx`:

```tsx
import React from 'react'
import ReactDOM from 'react-dom/client'
import App from './App'
import './index.css'

ReactDOM.createRoot(document.getElementById('root')!).render(
  <React.StrictMode>
    <App />
  </React.StrictMode>,
)
```

Create `src/App.tsx`:

```tsx
import { useState, useEffect } from 'react'
import ItemsTable from './components/ItemsTable'
import { Item, HealthResponse } from './types'
import './App.css'

const BASE_URL = 'http://localhost:8000'

function App() {
  const [items, setItems] = useState<Item[]>([])
  const [isLoading, setIsLoading] = useState(false)
  const [apiStatus, setApiStatus] = useState('Checking...')
  const [errorMessage, setErrorMessage] = useState<string | null>(null)

  useEffect(() => {
    loadData()
  }, [])

  const loadData = async () => {
    setIsLoading(true)
    setErrorMessage(null)

    // Check health
    try {
      const healthRes = await fetch(`${BASE_URL}/health`)
      const health: HealthResponse = await healthRes.json()
      setApiStatus(health.status)
    } catch {
      setApiStatus('offline')
      setErrorMessage('API not running')
      setIsLoading(false)
      return
    }

    // Fetch items
    try {
      const itemsRes = await fetch(`${BASE_URL}/items`)
      const data: Item[] = await itemsRes.json()
      setItems(data)
    } catch {
      setErrorMessage('Failed to load items')
    }

    setIsLoading(false)
  }

  return (
    <div className="app">
      <header className="header">
        <h1>{name}</h1>
        <div className="status">
          <span className={`status-dot ${apiStatus === 'healthy' ? 'healthy' : 'offline'}`} />
          <span className="status-text">{apiStatus}</span>
        </div>
      </header>

      <main className="content">
        {errorMessage ? (
          <div className="error-container">
            <span className="error-icon">⚠️</span>
            <p className="error-message">{errorMessage}</p>
            <p className="error-hint">Start API: cd services/api && uv run fastapi dev</p>
          </div>
        ) : isLoading ? (
          <div className="loading">Loading items...</div>
        ) : (
          <ItemsTable items={items} />
        )}
      </main>
    </div>
  )
}

export default App
```

Create `src/components/ItemsTable.tsx`:

```tsx
import { Item } from '../types'

interface ItemsTableProps {
  items: Item[]
}

function ItemsTable({ items }: ItemsTableProps) {
  return (
    <table className="items-table">
      <thead>
        <tr>
          <th style={{ width: '50px' }}>ID</th>
          <th style={{ width: '150px' }}>Name</th>
          <th>Description</th>
          <th style={{ width: '80px' }}>Price</th>
        </tr>
      </thead>
      <tbody>
        {items.map((item) => (
          <tr key={item.id}>
            <td className="mono">{item.id}</td>
            <td>{item.name}</td>
            <td>{item.description}</td>
            <td className="mono">${item.price.toFixed(2)}</td>
          </tr>
        ))}
      </tbody>
    </table>
  )
}

export default ItemsTable
```

Create `src/types.ts`:

```typescript
export interface Item {
  id: number
  name: string
  description: string
  price: number
}

export interface HealthResponse {
  status: string
}
```

Create `src/index.css`:

```css
* {
  margin: 0;
  padding: 0;
  box-sizing: border-box;
}

body {
  font-family: -apple-system, BlinkMacSystemFont, 'Segoe UI', Roboto, Oxygen, Ubuntu, sans-serif;
  background-color: #1a1a1a;
  color: #ffffff;
}

#root {
  min-height: 100vh;
}
```

Create `src/App.css`:

```css
.app {
  display: flex;
  flex-direction: column;
  height: 100vh;
}

.header {
  display: flex;
  justify-content: space-between;
  align-items: center;
  padding: 16px 24px;
  background-color: #2a2a2a;
  border-bottom: 1px solid #3a3a3a;
  -webkit-app-region: drag;
}

.header h1 {
  font-size: 1.5rem;
  font-weight: bold;
}

.status {
  display: flex;
  align-items: center;
  gap: 8px;
  -webkit-app-region: no-drag;
}

.status-dot {
  width: 12px;
  height: 12px;
  border-radius: 50%;
}

.status-dot.healthy {
  background-color: #22c55e;
}

.status-dot.offline {
  background-color: #ef4444;
}

.status-text {
  color: #888;
}

.content {
  flex: 1;
  padding: 24px;
  overflow: auto;
}

.error-container {
  display: flex;
  flex-direction: column;
  align-items: center;
  justify-content: center;
  height: 100%;
  gap: 12px;
}

.error-icon {
  font-size: 3rem;
}

.error-message {
  color: #888;
}

.error-hint {
  color: #666;
  font-size: 0.875rem;
}

.loading {
  display: flex;
  align-items: center;
  justify-content: center;
  height: 100%;
  color: #888;
}

.items-table {
  width: 100%;
  border-collapse: collapse;
}

.items-table th,
.items-table td {
  padding: 12px;
  text-align: left;
  border-bottom: 1px solid #3a3a3a;
}

.items-table th {
  background-color: #2a2a2a;
  font-weight: 600;
}

.items-table tr:hover {
  background-color: #2a2a2a;
}

.mono {
  font-family: 'Consolas', 'Monaco', monospace;
}
```

### 4.10 Create .gitignore for Electron/React

```
node_modules/
dist/
dist-electron/
release/
.env
*.log
```

### 4.11 Create CLAUDE.md for Electron submodule

```markdown
# {Name}Client - Electron React App

Windows desktop app that communicates with the FastAPI backend.

## Commands
- Dev: `npm run dev` (runs Vite + Electron concurrently)
- Build: `npm run build` (creates distributable)
- Vite only: `npm run dev:vite` (for web testing)

## Architecture
- Electron main process: electron/main.ts
- React app: src/
- Entry point: src/main.tsx
- Main component: src/App.tsx
- Data models: src/types.ts
- Uses fetch for API calls
- TypeScript throughout

## API Integration
- Backend runs at http://localhost:8000
- Health check: GET /health
- Sample data: GET /items (returns list of items)

## Adding Features
1. Create new React components in src/components/
2. Add new types in src/types.ts
3. Use fetch() for API calls
4. Add new routes if using react-router
```

### 4.12 Test Build

```powershell
npm install
npm run build:vite
```

Go back to workspace root:
```powershell
cd ..\..
```

---

## STEP 5: Create FastAPI Backend

### 5.1 Initialize with uv

```powershell
cd services/api
uv init --app
```

### 5.2 Replace pyproject.toml

```toml
[project]
name = "hackathon-api"
version = "0.1.0"
description = "Hackathon FastAPI Backend"
readme = "README.md"
requires-python = ">=3.12"
dependencies = [
    "fastapi[standard]>=0.115.0",
    "sqlalchemy>=2.0.0",
    "alembic>=1.13.0",
    "psycopg2-binary>=2.9.0",
    "pydantic-settings>=2.0.0",
]

[dependency-groups]
dev = [
    "pytest>=8.0.0",
    "pytest-asyncio>=0.23.0",
    "httpx>=0.27.0",
]
```

### 5.3 Create app directory structure

```powershell
New-Item -ItemType Directory -Force -Path app/routers
New-Item -ItemType Directory -Force -Path app/models
New-Item -ItemType Directory -Force -Path app/schemas
New-Item -ItemType File -Force -Path app/__init__.py
New-Item -ItemType File -Force -Path app/routers/__init__.py
New-Item -ItemType File -Force -Path app/models/__init__.py
New-Item -ItemType File -Force -Path app/schemas/__init__.py
```

### 5.4 Create app/config.py

```python
from pydantic_settings import BaseSettings


class Settings(BaseSettings):
    database_url: str = "postgresql://postgres:postgres@localhost:5432/hackathon"
    debug: bool = True

    class Config:
        env_file = ".env"


settings = Settings()
```

### 5.5 Create app/db.py

```python
from sqlalchemy import create_engine
from sqlalchemy.orm import sessionmaker, DeclarativeBase

from app.config import settings

engine = create_engine(settings.database_url)
SessionLocal = sessionmaker(autocommit=False, autoflush=False, bind=engine)


class Base(DeclarativeBase):
    pass


def get_db():
    db = SessionLocal()
    try:
        yield db
    finally:
        db.close()
```

### 5.6 Create app/models/item.py

```python
from sqlalchemy import Column, Integer, String, Float

from app.db import Base


class Item(Base):
    __tablename__ = "items"

    id = Column(Integer, primary_key=True, index=True)
    name = Column(String, nullable=False)
    description = Column(String)
    price = Column(Float, nullable=False)
```

### 5.7 Create app/schemas/item.py

```python
from pydantic import BaseModel


class ItemBase(BaseModel):
    name: str
    description: str
    price: float


class ItemCreate(ItemBase):
    pass


class Item(ItemBase):
    id: int

    class Config:
        from_attributes = True
```

### 5.8 Create app/main.py

```python
from contextlib import asynccontextmanager

from fastapi import FastAPI, Depends, HTTPException
from fastapi.middleware.cors import CORSMiddleware
from sqlalchemy.orm import Session

from app.db import Base, engine, get_db
from app.models.item import Item as ItemModel
from app.schemas.item import Item as ItemSchema


def seed_database(db: Session):
    """Seed the database with sample items if empty"""
    if db.query(ItemModel).count() == 0:
        sample_items = [
            ItemModel(name="Widget", description="A useful widget for your desk", price=9.99),
            ItemModel(name="Gadget", description="A fancy gadget with buttons", price=19.99),
            ItemModel(name="Gizmo", description="An amazing gizmo that does things", price=29.99),
        ]
        db.add_all(sample_items)
        db.commit()
        print("Database seeded with sample items")


@asynccontextmanager
async def lifespan(app: FastAPI):
    # Startup: create tables and seed data
    Base.metadata.create_all(bind=engine)
    db = next(get_db())
    seed_database(db)
    db.close()
    yield
    # Shutdown: cleanup if needed


app = FastAPI(
    title="Hackathon API",
    description="Backend API for Ombori Hackathon",
    version="0.1.0",
    lifespan=lifespan,
)

app.add_middleware(
    CORSMiddleware,
    allow_origins=["*"],
    allow_credentials=True,
    allow_methods=["*"],
    allow_headers=["*"],
)


@app.get("/")
async def root():
    return {"message": "Hackathon API is running!", "docs": "/docs"}


@app.get("/health")
async def health():
    return {"status": "healthy"}


@app.get("/items", response_model=list[ItemSchema])
async def get_items(db: Session = Depends(get_db)):
    """Get all items from the database"""
    return db.query(ItemModel).all()


@app.get("/items/{item_id}", response_model=ItemSchema)
async def get_item(item_id: int, db: Session = Depends(get_db)):
    """Get a specific item by ID"""
    item = db.query(ItemModel).filter(ItemModel.id == item_id).first()
    if not item:
        raise HTTPException(status_code=404, detail="Item not found")
    return item
```

### 5.9 Create .gitignore for Python

```
__pycache__/
*.py[cod]
.venv/
.env
*.egg-info/
dist/
build/
.pytest_cache/
.ruff_cache/
uv.lock
```

### 5.10 Create .env.example

```
DATABASE_URL=postgresql://postgres:postgres@localhost:5432/hackathon
DEBUG=true
```

### 5.11 Create CLAUDE.md for Python submodule

```markdown
# Hackathon API - FastAPI Backend

Python FastAPI backend with PostgreSQL database.

## Commands
- Run dev server: `uv run fastapi dev`
- Run tests: `uv run pytest`
- Sync dependencies: `uv sync`
- Add dependency: `uv add <package>`

## Project Structure
```
app/
├── main.py          # FastAPI app entry point
├── config.py        # Pydantic settings
├── db.py            # SQLAlchemy database setup
├── models/          # SQLAlchemy ORM models
├── schemas/         # Pydantic request/response schemas
└── routers/         # API route handlers
```

## Database
- PostgreSQL via Docker Compose
- SQLAlchemy 2.0 ORM
- Connection: postgresql://postgres:postgres@localhost:5432/hackathon

## API Docs
- Swagger UI: http://localhost:8000/docs
- ReDoc: http://localhost:8000/redoc

## Adding Features
1. Create model in app/models/
2. Create schemas in app/schemas/
3. Create router in app/routers/
4. Register router in app/main.py
```

### 5.12 Install dependencies and test

```powershell
uv sync
Start-Process -NoNewWindow powershell -ArgumentList "uv run fastapi dev --host 0.0.0.0 --port 8000"
Start-Sleep -Seconds 5
Invoke-RestMethod http://localhost:8000/health
# Kill the test server
Stop-Process -Name "python" -ErrorAction SilentlyContinue
```

Go back to workspace root:
```powershell
cd ..\..
```

---

## STEP 6: Create GitHub Repos

```powershell
gh repo create ombori-hackathon/{name}-ws --public --description "{name} - Terminal Velocity workspace"
gh repo create ombori-hackathon/{name}-desktop --public --description "{name} - Electron React desktop client"
gh repo create ombori-hackathon/{name}-api --public --description "{name} - FastAPI Python backend"
```

---

## STEP 7: Push Electron Project and Convert to Submodule

### 7.1 Initialize and push Electron repo

```powershell
cd apps/desktop-client
git init
git add .
git commit -m "Initial Electron React app setup"
git branch -M main
git remote add origin https://github.com/ombori-hackathon/{name}-desktop.git
git push -u origin main
cd ..\..
```

### 7.2 Remove folder and add as submodule

**IMPORTANT: Must be in workspace root before deleting!**

```powershell
# Ensure we're in workspace root
cd C:\Users\fed\Documents\Hackathon  # Use actual workspace path!
Remove-Item -Recurse -Force apps/desktop-client
git submodule add https://github.com/ombori-hackathon/{name}-desktop.git apps/desktop-client
```

---

## STEP 8: Push Python Project and Convert to Submodule

### 8.1 Initialize and push Python repo

```powershell
cd services/api
git init
git add .
git commit -m "Initial FastAPI backend setup"
git branch -M main
git remote add origin https://github.com/ombori-hackathon/{name}-api.git
git push -u origin main
cd ..\..
```

### 8.2 Remove folder and add as submodule

**IMPORTANT: Must be in workspace root before deleting!**

```powershell
# Ensure we're in workspace root
cd C:\Users\fed\Documents\Hackathon  # Use actual workspace path!
Remove-Item -Recurse -Force services/api
git submodule add https://github.com/ombori-hackathon/{name}-api.git services/api
```

---

## STEP 9: Create Docker Compose

Create `docker-compose.yml`:

```yaml
services:
  postgres:
    image: postgres:17
    container_name: hackathon-db
    environment:
      POSTGRES_USER: postgres
      POSTGRES_PASSWORD: postgres
      POSTGRES_DB: hackathon
    ports:
      - "5432:5432"
    volumes:
      - postgres_data:/var/lib/postgresql/data
    healthcheck:
      test: ["CMD-SHELL", "pg_isready -U postgres"]
      interval: 10s
      timeout: 5s
      retries: 5

volumes:
  postgres_data:
```

Create `.env`:

```
DATABASE_URL=postgresql://postgres:postgres@localhost:5432/hackathon
```

---

## STEP 10: Create Claude Code Agents

Create `.claude/agents/` directory and add these files:

### .claude/agents/architect.md

```yaml
---
name: architect
description: System design and cross-repo planning. Use for API contracts, database schemas, architectural decisions.
tools: Read, Grep, Glob, WebSearch, WebFetch
model: opus
---

# Architect Agent

Senior software architect for hackathon project.

## Your Codebase
- Electron React client: apps/desktop-client/
- FastAPI backend: services/api/
- PostgreSQL database via Docker
- Specs: specs/

## Responsibilities
1. Design API contracts (OpenAPI spec)
2. Plan database schemas (SQLAlchemy models)
3. Ensure client-server compatibility
4. Document architectural decisions
5. **Update CLAUDE.md and agents** with patterns and learnings

## Output Format
Always output GitHub-compatible markdown specs to `specs/` folder with:
- Clear requirements
- API endpoint definitions
- Data models
- Implementation steps

## Continuous Improvement
After completing features, suggest updates to:
- `CLAUDE.md` - New patterns, conventions discovered
- Agent files - Better instructions based on what worked
- Submodule CLAUDE.md files - Tech-specific learnings
```

### .claude/agents/react-coder.md

```yaml
---
name: react-coder
description: Electron React client development. Use for implementing features in apps/desktop-client/.
model: sonnet
---

# React Coder Agent

React/TypeScript developer for the Electron desktop app.

## Your Codebase
- Location: apps/desktop-client/
- Electron main: electron/main.ts
- React entry: src/main.tsx
- Main component: src/App.tsx
- Types: src/types.ts
- Dev: npm run dev
- Build: npm run build

## Patterns
- Use React hooks (useState, useEffect, etc.)
- TypeScript for all files
- fetch() for HTTP requests
- Functional components only
- CSS modules or plain CSS

## When Adding Features
1. Create new components in src/components/
2. Add types to src/types.ts
3. Use async/await with fetch for API calls
4. Test with npm run dev
```

### .claude/agents/python-coder.md

```yaml
---
name: python-coder
description: FastAPI backend development. Use for implementing features in services/api/.
model: sonnet
---

# Python Coder Agent

FastAPI developer for the backend API.

## Your Codebase
- Location: services/api/
- Entry point: app/main.py
- Run: uv run fastapi dev

## Patterns
- Pydantic schemas for request/response
- SQLAlchemy models for database
- Dependency injection with Depends()
- Async endpoints where beneficial

## When Adding Features
1. Create model in app/models/
2. Create schemas in app/schemas/
3. Create router in app/routers/
4. Register router in main.py
5. Test at /docs
```

### .claude/agents/reviewer.md

```yaml
---
name: reviewer
description: Code review across all repos. Use before committing significant changes.
tools: Read, Grep, Glob
model: sonnet
---

# Code Reviewer Agent

Reviews code for quality, security, and consistency.

## Review Checklist
- [ ] No hardcoded secrets
- [ ] Error handling present
- [ ] Types/schemas defined
- [ ] API contracts match between client/server
- [ ] Database queries are safe (no SQL injection)

## Output Format
Provide structured feedback:
1. **Issues** - Must fix before merge
2. **Suggestions** - Recommended improvements
3. **Approval** - Ready to commit or not
```

### .claude/agents/debugger.md

```yaml
---
name: debugger
description: Debug issues across repos. Use when something isn't working.
model: sonnet
---

# Debugger Agent

Investigates and fixes issues.

## Debugging Steps
1. Reproduce the issue
2. Check logs (docker compose logs, uvicorn output, npm run dev errors)
3. Trace the code path
4. Identify root cause
5. Propose fix

## Common Issues
- API not running: `uv run fastapi dev`
- Database not running: `docker compose up -d`
- Electron build fails: Check package.json dependencies
- Connection refused: Check ports (8000 for API, 5432 for DB, 5173 for Vite)
```

### .claude/agents/tester.md

```yaml
---
name: tester
description: Test-driven development. Use to write and run tests.
model: sonnet
---

# Tester Agent

Writes and runs tests for both repos.

## Python Tests
- Location: services/api/tests/
- Run: `uv run pytest`
- Use httpx for API testing
- Use pytest-asyncio for async tests

## React Tests
- Location: apps/desktop-client/src/__tests__/
- Run: `npm test`
- Use React Testing Library
- Use Vitest or Jest

## TDD Workflow
1. Write failing test first
2. Implement feature
3. Run tests until passing
4. Refactor if needed
```

---

## STEP 11: Create Skills

Create `.claude/skills/` directory for workflow commands:

```powershell
New-Item -ItemType Directory -Force -Path .claude/skills/feature
```

### .claude/skills/feature/SKILL.md

```yaml
---
name: feature
description: Build a new feature end-to-end. Asks clarifying questions, creates a spec/PRD, then implements using TDD (tests first). Use this to start any new feature.
user-invocable: true
allowed-tools: Read, Write, Edit, Bash, Grep, Glob
---

# /feature - New Feature Workflow

Build features using test-driven development with a proper spec.

## Usage

```
/feature <brief description of what you want to build>
```

## Workflow

### Step 1: Understand the Feature

Ask the user these questions (wait for answers before proceeding):

1. **What should this feature do?** (one sentence)
2. **Who/what triggers it?** (user action, API call, scheduled, etc.)
3. **What's the expected output/result?**
4. **Any edge cases to handle?**

### Step 2: Identify Affected Repos

Based on the feature, determine scope:
- **API only** → `services/api/`
- **Electron client only** → `apps/desktop-client/`
- **Full stack** → Both repos

### Step 3: Enter Plan Mode

**IMPORTANT: Enter plan mode now using the EnterPlanMode tool.**

In plan mode, create the spec file `specs/YYYY-MM-DD-feature-name.md` with this structure:

```markdown
# Feature: [Name]

## Summary
[One sentence from Step 1]

## Trigger
[From Step 1 question 2]

## Expected Result
[From Step 1 question 3]

## Edge Cases
[From Step 1 question 4]

## Technical Design

### API Changes (if applicable)
- Endpoint: `METHOD /path`
- Request: `{ field: type }`
- Response: `{ field: type }`

### React Client Changes (if applicable)
- New component: `ComponentName`
- New types: `TypeName`
- Display: [how it shows to user]

### Database Changes (if applicable)
- New table/columns: [describe]

## Implementation Plan
1. [ ] Write API tests (Red)
2. [ ] Implement API endpoint (Green)
3. [ ] Write React tests (Red)
4. [ ] Implement React code (Green)
5. [ ] Integration test
6. [ ] Commit and push
```

After writing the spec, use ExitPlanMode to present it for user approval.

**Do not proceed to implementation until the user approves the plan.**

### Step 4: TDD Red Phase - Write Failing Tests

**For API (if applicable):**
1. Create `services/api/tests/test_<feature>.py`
2. Write test for the new endpoint
3. Run `uv run pytest` - confirm it FAILS (Red)

**For React (if applicable):**
1. Add test in `apps/desktop-client/src/__tests__/`
2. Write test for new functionality
3. Run `npm test` - confirm it FAILS (Red)

### Step 5: TDD Green Phase - Implement

Implement minimum code to make tests pass:

**API:**
1. Add endpoint to `services/api/app/main.py` or create router
2. Run `uv run pytest` - should PASS (Green)

**React:**
1. Add code to `apps/desktop-client/src/`
2. Run `npm test` - should PASS (Green)

### Step 6: Refactor (Optional)

Clean up code while keeping tests green.

### Step 7: Create PR

Ask user: "All tests pass. Ready to create a PR?"

If yes, use `gh` CLI to create PRs (never use GitHub web interface):

```powershell
# Create feature branch, commit, and open PR for each repo
cd apps/desktop-client
git checkout -b feature/<feature-name>
git add .
git commit -m "feat: <description>"
git push -u origin feature/<feature-name>
gh pr create --title "feat: <description>" --body "Implements <feature>"

cd ../../services/api
git checkout -b feature/<feature-name>
git add .
git commit -m "feat: <description>"
git push -u origin feature/<feature-name>
gh pr create --title "feat: <description>" --body "Implements <feature>"
```

Update spec with PR links and completion status.
```

---

## STEP 12: Create Specs Directory

Create a `specs/` folder for centralized feature specifications:

```powershell
New-Item -ItemType Directory -Force -Path specs
```

Create `specs/README.md`:

```markdown
# Feature Specifications

All feature specs live here. Created via plan mode before implementation.

## Naming Convention
`YYYY-MM-DD-feature-name.md`

## Template
See `specs/_template.md`
```

Create `specs/_template.md`:

```markdown
# Feature: [Name]

## Summary
Brief description of the feature.

## API Changes
- `POST /endpoint` - Description

## Database Changes
- New table: `table_name`
- New columns: `column_name`

## React Client Changes
- New component: `ComponentName`

## Implementation Steps
1. [ ] Step 1
2. [ ] Step 2

## Tests
- [ ] API test: description
- [ ] Client test: description
```

---

## STEP 13: Create Workspace CLAUDE.md

Create `CLAUDE.md` in workspace root:

```markdown
# Hackathon Workspace

Multi-repo workspace for Ombori hackathon.

## Key Rules

1. **Plan-mode-first**: ALL features start with spec creation in `specs/` folder
2. **TDD where applicable**: Write tests before implementation
3. **Specs in workspace**: All specs centralized in `specs/` folder
4. **Evolve the config**: Update CLAUDE.md and agents with learnings as you build

## Continuous Improvement

As you build, update these files with learnings:
- `CLAUDE.md` - Add new patterns, gotchas, project-specific conventions
- `.claude/agents/*.md` - Refine agent instructions based on what works
- `apps/desktop-client/CLAUDE.md` - React/Electron-specific learnings
- `services/api/CLAUDE.md` - Python/FastAPI-specific learnings

Examples of things to capture:
- "Always use X pattern for Y"
- "Don't forget to run Z after changing W"
- "API endpoint naming follows this convention..."
- "Database migrations require this step..."

## Quick Start

```powershell
# Start database
docker compose up -d

# Run API (in new terminal)
cd services/api
uv run fastapi dev

# Run Electron client (in new terminal)
cd apps/desktop-client
npm run dev
```

## Structure
- `apps/desktop-client/` - Electron React desktop app (submodule)
- `services/api/` - FastAPI Python backend (submodule)
- `specs/` - Feature specifications (plan-mode output)
- `docker-compose.yml` - PostgreSQL database

## Skills (Commands)
Available in `.claude/skills/`:
- `/feature` - **Start here!** Asks questions → creates spec → TDD implementation

## Agents
Available in `.claude/agents/`:
- `/architect` - System design, API contracts → outputs to `specs/`
- `/react-coder` - Electron React client development
- `/python-coder` - FastAPI backend development
- `/reviewer` - Code review across all repos
- `/debugger` - Issue investigation
- `/tester` - Test-driven development

## Development Workflow

### New Features (MANDATORY)
1. **Plan mode first** - Create spec in `specs/YYYY-MM-DD-feature-name.md`
2. **Write tests** - TDD: tests before implementation
3. **Implement** - Use coder agents (can run in parallel)
4. **Review** - Use reviewer agent
5. **Commit** - Submodules first, then workspace

### Git Workflow (use `gh` CLI, not GitHub web)
Always use feature branches and `gh pr create`:
```powershell
# In each submodule
git checkout -b feature/<name>
git add .
git commit -m "feat: ..."
git push -u origin feature/<name>
gh pr create --title "feat: ..." --body "Description"

# After PRs merged, update workspace
cd ..\..
git add .
git commit -m "Update submodules"
git push
```

## API Reference
- Swagger: http://localhost:8000/docs
- Health: http://localhost:8000/health
- Items: http://localhost:8000/items (from database)
```

---

## STEP 14: Initialize and Push Workspace

```powershell
git init
git add .
git commit -m "Initial workspace setup with submodules"
git branch -M main
git remote add origin https://github.com/ombori-hackathon/{name}-ws.git
git push -u origin main
```

---

## STEP 15: Verify Everything Works

Run these verification steps:

```powershell
# 1. Check submodules
git submodule status

# 2. Start database
docker compose up -d

# 3. Wait for postgres
Start-Sleep -Seconds 5

# 4. Check database is healthy
docker compose ps

# 5. Start API
cd services/api
uv sync
Start-Process -NoNewWindow powershell -ArgumentList "uv run fastapi dev"
Start-Sleep -Seconds 3

# 6. Test API
Invoke-RestMethod http://localhost:8000/health

# 7. Test Electron client
cd ../apps/desktop-client
npm install
npm run dev

# 8. Check agents exist
Get-ChildItem ..\..\..\.claude\agents\

# 9. Stop processes when done testing
Stop-Process -Name "python" -ErrorAction SilentlyContinue
```

---

## DONE!

Your hackathon workspace is ready:
- 3 GitHub repos created and linked
- Electron React app builds and runs
- FastAPI backend runs with health endpoint
- PostgreSQL database via Docker
- Claude Code agents and skills configured

**Next: Restart Claude Code, then use `/feature` to start building!**

Happy hacking!
