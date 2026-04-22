# QUITR - Supabase Setup Guide

## What Changed
Complete refactor of the Supabase sync system to make it reliable and automatic across multiple devices.

### Major Fixes
1. **Auto-sync on every state change** - No more manual "Sync" button needed (though button still available)
2. **Complete state storage** - Now stores full JSON state, not just counts
3. **Device identification** - Each device gets a unique ID to track separate streaks
4. **Conflict resolution** - Uses timestamps to merge newer data automatically
5. **Removed broken code** - Deleted non-functional Google Forms integration
6. **Removed page reload** - No more forced reload after starting journey
7. **Better error handling** - Proper error messages and retry logic

## Supabase Table Schema

You need a table named `tracker` with the following columns:

```sql
CREATE TABLE tracker (
  id BIGINT PRIMARY KEY GENERATED ALWAYS AS IDENTITY,
  device_id TEXT NOT NULL UNIQUE,
  state_data TEXT NOT NULL,
  start_time BIGINT,
  created_at TIMESTAMP WITH TIME ZONE DEFAULT NOW(),
  updated_at TIMESTAMP WITH TIME ZONE DEFAULT NOW()
);

CREATE INDEX idx_device_id ON tracker(device_id);
```

### Column Descriptions
- **id**: Auto-incrementing primary key
- **device_id**: Unique identifier for each device (auto-generated on first use)
- **state_data**: Complete JSON state as string - this is the full app state
- **start_time**: Timestamp of current streak start (for quick queries)
- **created_at**: When record was first created
- **updated_at**: When record was last updated (used for conflict resolution)

## How It Works

### On App Start
1. Load state from localStorage
2. Try to fetch latest state from Supabase for this device
3. If cloud state is newer than local, use it instead
4. Otherwise keep local state

### On Every Action
- State is immediately saved to localStorage
- After 1 second of inactivity, state is synced to Supabase
- If it's your first time on this device, a new record is created
- If you've synced before, the existing record is updated

### Manual Sync
Two buttons on Analytics page:
- **Sync Now**: Download latest state from cloud (pull)
- **Push Data**: Upload current state to cloud (push)

## Row Limit Rights
⚠️ **Important**: Supabase free tier has a 1MB row size limit. The full JSON state should be well under this for normal usage.

## Testing Multi-Device Sync

1. On Device A:
   - Start journey
   - Log a few urges
   - Go to Analytics → "Push Data"

2. On Device B:
   - Open the app (auto-loads on init if newer)
   - Or go to Analytics → "Sync Now" to pull

3. Both devices should now show the same streak and activity

## Troubleshooting

**Q: Sync button shows "No cloud data found"**
- First device? That's normal. Just keep using the app; it will sync on next action.

**Q: Different data on different devices**
- Go to Analytics → "Sync Now" on the device with outdated data
- Or "Push Data" from the device with current data

**Q: Conflict between devices**
- The app automatically uses whichever state is newer (based on timestamp)
- Manual sync buttons let you override if needed

## API Credentials
The Supabase keys are embedded in the HTML (anon/public key is safe). For production:
- Enable RLS (Row Level Security) on the `tracker` table
- Restrict to authorized users only
- Use a real authentication system instead of device IDs

## JSON State Structure

What gets saved to `state_data`:
```json
{
  "startTime": 1713792000000,
  "streaks": [
    {
      "start": 1713700000000,
      "end": 1713792000000,
      "duration": 92000000,
      "urgesResisted": 3
    }
  ],
  "currentStreak": { "start": null, "urgesResisted": 0 },
  "relapses": [],
  "urges": [
    { "time": 1713750000000, "level": 5 },
    { "time": 1713760000000, "level": 3 }
  ],
  "naturals": [
    { "time": 1713770000000 }
  ],
  "journals": [],
  "milestones": [
    { "type": "Journey started", "time": 1713792000000, "category": "start" },
    { "type": "Urge level 5", "time": 1713750000000, "category": "urge" }
  ],
  "lastModified": 1713792000000
}
```

All of this gets stored as a string in the `state_data` column and automatically synced across your devices.
