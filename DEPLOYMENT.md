# Deployment Info

## Live Dashboard
**URL:** https://openclaw-dashboard.vercel.app

## Project Repos
- **Frontend:** https://github.com/dakirkley/openclaw-dashboard
- **API Server:** https://github.com/dakirkley/openclaw-dashboard-api
- **Documentation:** https://github.com/dakirkley/dashboard-memory-bank (this repo)

## Architecture
```
┌─────────────────┐     ┌─────────────────┐     ┌─────────────────┐
│  OpenClaw       │────▶│  Dashboard API  │────▶│  Supabase       │
│  Instances      │◀────│  Server         │◀────│  (Database)     │
│  (Sync Skill)   │     │                 │     │                 │
└─────────────────┘     └─────────────────┘     └─────────────────┘
                                │
                                ▼
                       ┌─────────────────┐
                       │  Dashboard UI   │
                       │  (React/Vite)   │
                       │  Deployed on    │
                       │  Vercel         │
                       └─────────────────┘
```

## Current Status
- ✅ Dashboard deployed and accessible
- ✅ Supabase connection established
- ✅ Sync skill operational
- ✅ Multi-install synchronization working
- ✅ 9 skills detected and inventoried
- ✅ 10 APIs tracked
