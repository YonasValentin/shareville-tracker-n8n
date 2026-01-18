# Codebase Structure

**Analysis Date:** 2026-01-18

## Directory Layout

```
shareville-tracker-n8n/
├── .planning/              # Planning and analysis documents
│   └── codebase/           # Architecture and convention docs
├── compose.yaml            # Docker Compose for n8n deployment
├── portfolio-news-workflow.json  # Portfolio news alert workflow
├── shareville-workflow.json      # Trader tracking workflow
└── README.md               # Project documentation
```

## Directory Purposes

**Root Directory:**
- Purpose: Contains all project files (flat structure)
- Contains: Workflow JSON files, Docker config, documentation
- Key files: `shareville-workflow.json`, `portfolio-news-workflow.json`, `compose.yaml`

**.planning/:**
- Purpose: Project planning and codebase analysis
- Contains: Architecture documentation, conventions
- Key files: `codebase/ARCHITECTURE.md`, `codebase/STRUCTURE.md`

## Key File Locations

**Entry Points:**
- `shareville-workflow.json`: Main trader tracking workflow (import into n8n)
- `portfolio-news-workflow.json`: Portfolio news monitoring workflow (import into n8n)

**Configuration:**
- `compose.yaml`: Docker Compose configuration for n8n deployment
- Environment variables defined inline in compose.yaml

**Core Logic:**
- `shareville-workflow.json`: Contains all trader tracking logic in Code nodes
- `portfolio-news-workflow.json`: Contains all news monitoring logic in Code nodes

**Testing:**
- No automated tests - testing done via n8n's "Test Workflow" button
- Manual execution in n8n UI validates workflow behavior

## Naming Conventions

**Files:**
- Workflow JSON: `{feature}-workflow.json` (e.g., `shareville-workflow.json`)
- Docker config: `compose.yaml` (standard Docker Compose name)
- Documentation: `README.md`, uppercase for planning docs (`ARCHITECTURE.md`)

**Directories:**
- Lowercase with hyphens: `.planning/codebase/`

**n8n Node IDs:**
- Kebab-case descriptive names: `schedule-trigger`, `get-user-id`, `filter-trades`

**n8n Node Display Names:**
- Title case descriptive: `Every 5 Minutes`, `Get User ID`, `Filter Trades`

## Where to Add New Code

**New Workflow:**
- Create: `{feature}-workflow.json` in project root
- Follow existing node structure and connection patterns
- Use same Discord webhook URL for consistent channel delivery
- Document in README.md

**New Trader to Track:**
- Location: `shareville-workflow.json` > "Trader List" node > jsCode
- Edit the `traders` array:
```javascript
const traders = [
  { slug: 'existing', name: 'Existing', focus: 'Strategy' },
  { slug: 'new-slug', name: 'New Name', focus: 'New Strategy' }
];
```

**New Data Enrichment:**
- Add HTTP Request node after existing enrichment nodes
- Add Code node to process response and merge with existing data
- Use spread operator pattern: `{ ...prevData, newField: value }`

**New Discord Field:**
- Location: "Format Discord Message" Code node
- Add to `fields` array following existing pattern:
```javascript
if (data.newField) {
  fields.push({ name: 'Icon Name', value: data.newField, inline: true });
}
```

**Infrastructure Changes:**
- Location: `compose.yaml`
- Add new environment variables under `environment:`
- Add new volumes under `volumes:` if persistent storage needed

## Special Directories

**.planning/:**
- Purpose: Houses GSD planning and codebase analysis documents
- Generated: By GSD commands
- Committed: Yes

**n8n_data (Docker volume):**
- Purpose: Persistent n8n data (workflows, credentials, execution history)
- Generated: By n8n container at runtime
- Committed: No (Docker managed volume)
- Location: Docker volume, not in project directory

## File Relationships

```
compose.yaml
    └── Deploys n8n container
            └── Imports workflow JSON files manually via n8n UI
                    ├── shareville-workflow.json
                    │       └── Uses Discord webhook (hardcoded)
                    │       └── Uses Shareville API (public)
                    │       └── Uses Yahoo Finance API (cookie auth)
                    │
                    └── portfolio-news-workflow.json
                            └── Uses Discord webhook (same)
                            └── Uses Shareville API (same)
                            └── Uses Yahoo Finance API (same)
```

## Workflow Import Process

1. Deploy n8n via `compose.yaml` in Dockge
2. Access n8n UI at `http://192.168.0.25:5678`
3. Import each `*-workflow.json` file via Workflows > Import
4. Activate workflows via toggle switch

---

*Structure analysis: 2026-01-18*
