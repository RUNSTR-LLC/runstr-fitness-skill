---
name: runstr-fitness
description: Give your AI agent access to your health and fitness data from RUNSTR. Fetches workouts from Supabase and competition leaderboards from Nostr using only your public key (npub). No private key required for core features.
metadata: {"openclaw":{"emoji":"\ud83c\udfc3","requires":{"bins":["curl","jq"]}}}
---

# RUNSTR Fitness Skill

Give your AI agent access to your real health and fitness data. RUNSTR is a free fitness app that tracks workouts and syncs them automatically. This skill lets your agent read that data for fitness coaching, competition tracking, and health insights.

## Security & Privacy

**This skill is designed to be safe:**

| Feature | What's needed | Risk level |
|---------|---------------|------------|
| Workouts & stats | npub (public key) | ✅ Safe to share |
| Competition rankings | npub or nothing | ✅ Safe |
| Leaderboards | Nothing | ✅ Public data |
| PPQ.AI credits | nsec (private key) | ⚠️ Optional, see below |

**Your npub (public key)** is safe to share — it's like a username. Anyone can look you up on Nostr with it.

**Your nsec (private key)** is sensitive — it controls your Nostr identity. This skill only needs nsec for ONE optional feature: extracting your PPQ.AI credentials from your encrypted backup. If you don't use PPQ.AI rewards, you never need to share your nsec.

---

## What Your Agent Gets Access To

**With just your npub (recommended):**
- ✅ Workout history (runs, walks, cycles, etc.)
- ✅ Competition rankings and your position
- ✅ Leaderboards and daily records
- ✅ Charity impact stats

**With your nsec (optional):**
- ✅ Everything above, plus:
- ✅ PPQ.AI API key (to use workout-earned AI credits)
- ✅ Habits and journal entries (from encrypted backup)

---

## Setup: Connect Your RUNSTR Data

### Quick Start (npub only — recommended)

1. **Find your npub** in RUNSTR: Settings > Keys
2. **Tell your agent:** "My RUNSTR npub is npub1..."
3. **Done!** Your agent can now see your workouts and competition stats.

### Full Access (includes PPQ.AI)

If you use PPQ.AI as your reward destination and want your agent to use those credits:

1. **Create a backup** in RUNSTR: Settings > Backup > Backup to Nostr
2. **Find your nsec** in RUNSTR: Settings > Keys
3. **Tell your agent:** "My RUNSTR nsec is nsec1..."

⚠️ **nsec safety tips:**
- Consider creating a dedicated Nostr identity just for fitness
- Your nsec is only used locally to decrypt your backup
- Never share your nsec in public channels

---

## For the Agent: Fetching Data

### Method 1: Supabase (Recommended — Real-time Workouts)

RUNSTR syncs workouts to Supabase automatically. This is the freshest data source.

**Supabase connection:**
```
Project URL: [Get from RUNSTR app config or ask user]
Anon Key: [Public anon key - safe to expose, in app bundle]
```

> **Note for skill maintainers:** The Supabase URL and anon key need to be filled in from the RUNSTR app's configuration. These are safe to include — the anon key is already public in the app bundle.

**Fetch workouts by npub:**
```bash
# Convert npub to hex first
HEX_PK=$(nak decode npub1...)

# Replace SUPABASE_URL and ANON_KEY with actual values
curl -s "$SUPABASE_URL/rest/v1/workouts?user_pubkey=eq.$HEX_PK&order=created_at.desc&limit=50" \
  -H "apikey: $ANON_KEY" \
  -H "Authorization: Bearer $ANON_KEY"
```

**Workout fields:**
```json
{
  "id": "uuid",
  "user_pubkey": "hex pubkey",
  "activity_type": "running",
  "distance_meters": 5200,
  "duration_seconds": 2100,
  "calories": 312,
  "created_at": "2025-01-15T07:35:00Z"
}
```

### Method 2: Nostr Leaderboards (Public — No Auth)

Competition leaderboards are published publicly on Nostr. No npub or nsec needed.

```bash
# RUNSTR aggregator pubkey
RUNSTR_AGG="611021eaaa2692741b1236bbcea54c6aa9f20ba30cace316c3a93d45089a7d0f"

# Requires nak: go install github.com/fiatjaf/nak@latest
nak req -k 30150 -a $RUNSTR_AGG -t d=runstr-leaderboards -l 1 \
  wss://relay.damus.io wss://nos.lol | jq -r '.content' | jq .
```

**Leaderboard structure:**
```json
{
  "v": 1,
  "updatedAt": "2026-02-06T12:05:00Z",
  "competitions": [
    {
      "id": "season-ii",
      "name": "Season II",
      "activityType": "running",
      "scoringMethod": "total_distance",
      "status": "active",
      "entries": [
        {"r": 1, "p": "npub1abc...", "n": "Runner1", "s": 142.5, "w": 28}
      ]
    }
  ],
  "daily": {
    "fastest5k": [{"r": 1, "p": "npub1...", "n": "SpeedKing", "s": 1320}],
    "mostSteps": [{"r": 1, "p": "npub1...", "n": "StepQueen", "s": 18432}]
  },
  "charityRankings": [
    {"id": "als-foundation", "name": "ALS Foundation", "km": 342.5, "sats": 342500}
  ]
}
```

**Entry fields:** `r`=rank, `p`=npub, `n`=name, `s`=score, `w`=workout count

**Finding user's position:** Search `entries` array for matching `p` field.

### Method 3: Encrypted Backup (Optional — Requires nsec)

Only use this if the user wants PPQ.AI access or habits/journal data.

**Prerequisites:** `nak` (Nostr Army Knife)
```bash
go install github.com/fiatjaf/nak@latest
```

**Fetch and decrypt:**
```bash
# Decode keys
hex_sk=$(nak decode nsec1...)
hex_pk=$(nak key public $hex_sk)

# Fetch backup
content=$(nak req -k 30078 -a $hex_pk -t d=runstr-workout-backup -l 1 \
  wss://relay.damus.io wss://nos.lol | jq -r '.content')

# Decrypt and decompress
echo "$content" | nak encrypt --sec $hex_sk $hex_pk --decrypt | base64 -d | gunzip | jq .
```

**Backup payload:**
```json
{
  "version": 1,
  "exportedAt": "2025-01-15T10:30:00Z",
  "workouts": [...],
  "habits": [
    {"name": "No Smoking", "currentStreak": 45, "longestStreak": 45}
  ],
  "journal": [
    {"date": "2025-01-15", "content": "Great run!", "mood": "great", "energy": 4}
  ],
  "preferences": {
    "unitSystem": "imperial",
    "selectedCharity": "hrf",
    "ppqApiKey": "ppq_abc123...",
    "ppqCreditId": "credit_xyz789..."
  }
}
```

---

## PPQ.AI Integration (Optional)

If the user earns PPQ.AI credits instead of sats, their backup contains API credentials.

**Check balance:**
```bash
curl -X POST https://api.ppq.ai/credits/balance \
  -H "Content-Type: application/json" \
  -d '{"credit_id": "CREDIT_ID_HERE"}'
```

**Make LLM request:**
```bash
curl -X POST https://api.ppq.ai/v1/chat/completions \
  -H "Authorization: Bearer PPQ_API_KEY_HERE" \
  -H "Content-Type: application/json" \
  -d '{"model": "claude-sonnet-4-20250514", "messages": [{"role": "user", "content": "Hello!"}]}'
```

**For full model access:** Guide user to https://ppq.ai — the browser interface has more models than the API.

---

## Coaching Mode

Once you have data, you can:
- Track competition position: "You're #7, 12km behind #3"
- Analyze trends: "Your pace improved 8% this month"
- Encourage streaks: "14-day streak — keep it going!"
- Set goals: "You need 3 more runs to hit 100km this month"
- Report impact: "Your workouts donated 500 sats to ALS Foundation"

**Example prompts users can try:**
- "How's my running this month?"
- "What's my position in Season II?"
- "Who's winning?"
- "What did I earn this week?"

---

## Troubleshooting

| Problem | Solution |
|---------|----------|
| No workouts found | User may not have synced yet — workouts appear after app sync |
| Leaderboard empty | Check if competitions are active; aggregator updates every 5 min |
| PPQ.AI key missing | User needs to backup after setting up PPQ.AI rewards |
| Decryption fails | Verify nsec is correct; try Node.js method if nak fails |

---

## About RUNSTR

Free, open-source fitness app for the Bitcoin/Nostr community. Track workouts, earn sats or AI credits, support charities.

- Website: https://runstr.app
- GitHub: https://github.com/RUNSTR-LLC/RUNSTR
- PPQ.AI: https://ppq.ai
