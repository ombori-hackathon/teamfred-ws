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
- `/feature` - **Start here!** Asks questions -> creates spec -> TDD implementation

## Agents
Available in `.claude/agents/`:
- `/architect` - System design, API contracts -> outputs to `specs/`
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
