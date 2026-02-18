# RUNSTR Fitness Skill 🏃

**Give your AI agent access to your health and fitness data — and earn AI credits just by working out.**

This skill connects your AI agent to your [RUNSTR](https://runstr.app) fitness data via Nostr's encrypted backup system. Your agent gets full context on your workouts, habits, mood, and competition standings — enabling personalized coaching, accountability, and health insights.

## What This Skill Does

### 1. 🏋️ Health & Fitness Context for Your Agent

RUNSTR aggregates fitness data from multiple sources:
- **Apple Health** (iOS)
- **Health Connect** (Android)
- **Garmin, Nike Run Club, Strava** (via Health integrations)
- **Manual tracking** in the app

Your agent can access:
- Workout history (running, walking, cycling, strength, yoga, etc.)
- Daily habits and streaks (quit smoking, daily meditation, etc.)
- Journal entries with mood and energy levels
- Daily step counts
- Personal records

### 2. 🏆 Virtual Fitness Competitions

RUNSTR runs virtual fitness challenges (Season II, Einundzwanzig, etc.). Your agent can:
- Tell you who's winning any active competition
- Show your current rank and gap to the leader
- Track daily records (fastest 5K, most steps)
- Monitor charity impact for team-based events

**Leaderboards are public** — your agent can answer "who's winning?" without needing your private key.

### 3. 🤖 Earn AI Credits by Working Out

This is the killer feature. RUNSTR lets users earn:
- **Bitcoin (sats)** for daily workouts
- **PPQ.AI credits** as an alternative reward

By choosing PPQ.AI as your reward destination, your workouts fund your agent's LLM access. Work out → earn credits → agent gets smarter.

Your PPQ.AI key is stored in your encrypted RUNSTR backup on Nostr. This skill retrieves it (with your permission) and connects your agent to PPQ.AI's multi-model API.

## How It Works

```
┌─────────────┐     ┌─────────────┐     ┌─────────────┐
│   RUNSTR    │────▶│    Nostr    │────▶│  Your Agent │
│  (Mobile)   │     │   Relays    │     │  (OpenClaw) │
└─────────────┘     └─────────────┘     └─────────────┘
       │                   │                   │
       │  Encrypted        │  Fetch &          │  Coaching,
       │  Backup           │  Decrypt          │  Insights,
       │  (NIP-44)         │                   │  PPQ.AI Access
       ▼                   ▼                   ▼
```

1. **You track workouts** in RUNSTR (automatic from Apple Health/Health Connect)
2. **You backup to Nostr** (encrypted with your private key)
3. **Your agent fetches the backup** and decrypts it with your nsec
4. **Your agent now has context** for personalized coaching

## Quick Start

### For Users

1. **Download RUNSTR** — [App Store](https://apps.apple.com/app/runstr) | [Zapstore](https://zapstore.dev) | [GitHub](https://github.com/RUNSTR-LLC/RUNSTR)
2. **Track some workouts** (or let it sync from Apple Health)
3. **Create a backup** — Settings > Backup > Backup to Nostr
4. **Tell your agent your nsec** — "Here's my RUNSTR nsec: nsec1..."

Your agent can now access your fitness data and help with coaching.

### For Agents (Technical)

See [SKILL.md](./SKILL.md) for full implementation details:
- How to fetch and decrypt backups
- Payload structure and field reference
- Leaderboard queries (no nsec needed)
- Troubleshooting guide

## Privacy & Security

- **Your data is encrypted** — NIP-44 self-encryption means only you can read it
- **nsec never leaves your session** — used only for decryption, not stored
- **Optional dedicated identity** — create a separate Nostr key just for fitness
- **Open source** — verify exactly what data is collected

## Roadmap

- [ ] Cron job notifications for competition standings
- [ ] In-person race discovery (events near you)
- [ ] PPQ.AI key in encrypted backup ([RUNSTR#31](https://github.com/RUNSTR-LLC/RUNSTR/issues/31))
- [ ] Multi-model routing based on AI credit balance

## Links

- **RUNSTR App**: https://runstr.app
- **RUNSTR GitHub**: https://github.com/RUNSTR-LLC/RUNSTR
- **PPQ.AI**: https://ppq.ai
- **Nostr Protocol**: https://nostr.com

## License

MIT

---

*Work out. Earn AI credits. Let your agent help you get fitter.* 🦾
