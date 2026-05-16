# OpenClaw Dashboard - Technical Notes

## Technology Stack

### Frontend
- **Framework:** React 18+
- **Build Tool:** Vite
- **Language:** TypeScript
- **Styling:** Tailwind CSS
- **Components:** shadcn/ui
- **State Management:** React Query / Zustand
- **Routing:** React Router

### Backend
- **Database:** Supabase (PostgreSQL)
- **Authentication:** Supabase Auth
- **Realtime:** Supabase Realtime
- **API:** RESTful + WebSocket

### Infrastructure
- **Hosting:** Vercel
- **Database:** Supabase Cloud
- **CDN:** Vercel Edge Network

---

## Database Schema

### Core Tables

```sql
-- Businesses
CREATE TABLE businesses (
  id UUID PRIMARY KEY DEFAULT uuid_generate_v4(),
  name TEXT NOT NULL,
  description TEXT,
  created_at TIMESTAMP DEFAULT NOW(),
  updated_at TIMESTAMP DEFAULT NOW()
);

-- Bots
CREATE TABLE bots (
  id UUID PRIMARY KEY DEFAULT uuid_generate_v4(),
  business_id UUID REFERENCES businesses(id),
  name TEXT NOT NULL,
  type TEXT,
  status TEXT DEFAULT 'active',
  install_id TEXT,
  created_at TIMESTAMP DEFAULT NOW()
);

-- Agents
CREATE TABLE agents (
  id UUID PRIMARY KEY DEFAULT uuid_generate_v4(),
  bot_id UUID REFERENCES bots(id),
  name TEXT NOT NULL,
  role TEXT,
  parent_id UUID REFERENCES agents(id),
  created_at TIMESTAMP DEFAULT NOW()
);

-- Skills
CREATE TABLE skills (
  id UUID PRIMARY KEY DEFAULT uuid_generate_v4(),
  name TEXT NOT NULL,
  version TEXT,
  description TEXT,
  install_id TEXT,
  detected_at TIMESTAMP DEFAULT NOW()
);

-- APIs
CREATE TABLE apis (
  id UUID PRIMARY KEY DEFAULT uuid_generate_v4(),
  name TEXT NOT NULL,
  endpoint TEXT,
  method TEXT,
  install_id TEXT,
  detected_at TIMESTAMP DEFAULT NOW()
);

-- Installs (for multi-install tracking)
CREATE TABLE installs (
  id TEXT PRIMARY KEY,
  name TEXT,
  location TEXT,
  last_sync TIMESTAMP,
  status TEXT DEFAULT 'active'
);
```

---

## Sync Skill Architecture

### Detection Flow

```
OpenClaw Instance
       │
       ▼
┌──────────────┐
│ Scan Skills  │──▶ Parse SKILL.md files
│ Directory    │    Extract metadata
└──────────────┘
       │
       ▼
┌──────────────┐
│ Detect APIs  │──▶ Scan API configurations
│ & Endpoints  │    Catalog endpoints
└──────────────┘
       │
       ▼
┌──────────────┐
│ Inventory    │──▶ List bots/agents
│ Local State  │    Read config files
└──────────────┘
       │
       ▼
┌──────────────┐
│ Sync to      │──▶ POST to dashboard API
│ Dashboard    │    Update remote state
└──────────────┘
```

### Sync API Endpoints

```
POST /api/sync/skills
  Body: { install_id, skills: [...] }

POST /api/sync/apis
  Body: { install_id, apis: [...] }

POST /api/sync/bots
  Body: { install_id, bots: [...] }

POST /api/sync/agents
  Body: { install_id, agents: [...] }

POST /api/sync/full
  Body: { install_id, data: { skills, apis, bots, agents } }
```

---

## Environment Variables

### Dashboard
```bash
# Supabase
VITE_SUPABASE_URL=https://xxxx.supabase.co
VITE_SUPABASE_ANON_KEY=eyJ...

# API
VITE_API_URL=https://api.dashboard.example.com
```

### Sync Skill
```bash
# Dashboard Connection
DASHBOARD_API_URL=https://api.dashboard.example.com
DASHBOARD_API_KEY=sk_...

# Local OpenClaw
OPENCLAW_INSTALL_ID=install_001
OPENCLAW_SKILLS_DIR=/path/to/skills
```

---

## Known Technical Issues

### Skill Detection Gaps
**Problem:** Some skills not being detected despite being installed

**Possible Causes:**
- Skills in non-standard directories
- Missing SKILL.md files
- Permission issues reading skill directories
- Case sensitivity in file matching

**Investigation Steps:**
1. Check skill directory structure
2. Verify SKILL.md exists for all skills
3. Review file permissions
4. Add debug logging to sync skill

### Real-Time Updates
**Current:** Polling-based updates
**Target:** WebSocket/Realtime subscriptions

**Implementation Plan:**
1. Enable Supabase Realtime for relevant tables
2. Subscribe to changes in React components
3. Update local cache on events
4. Handle reconnection logic

---

## Performance Considerations

### Database
- Indexes on `install_id` for fast filtering
- Indexes on `business_id` for relationship queries
- Consider partitioning by install for scale

### API
- Rate limiting on sync endpoints
- Batch updates for large inventories
- Pagination for list endpoints

### Frontend
- Virtualization for long lists
- Debounced search
- Lazy loading for detail views

---

## Security Notes

- RLS policies enforced on all tables
- API key authentication for sync endpoints
- Install ID validation required
- No sensitive data in client-side code

---

## Development Commands

```bash
# Dashboard
cd dashboard
npm install
npm run dev
npm run build

# Sync Skill
# (Installed as OpenClaw skill)
openclaw skill run sync
```
