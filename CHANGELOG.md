# QUITR - Changelog: Supabase & Bug Fixes

## 🔧 Major Refactoring

### Supabase Integration - Complete Overhaul
**Problem**: Sync was unreliable, only stored counts (not full data), and couldn't properly merge updates from multiple devices.

**Solution**: Complete rewrite with:
- ✅ Automatic sync after every state change (1-second debounce)
- ✅ Full JSON state storage instead of just counts
- ✅ Device ID system to identify each device
- ✅ Timestamp-based conflict resolution (newer data wins)
- ✅ Proper async/await error handling
- ✅ Status tracking to prevent concurrent syncs

**Before**: 
```js
// Only stored counts, data was reconstructed & lost
saveToSupabase('action') -> stores only urges, naturals, journals count
loadFromSupabase() -> tries to reconstruct data from counts (creates fake data!)
```

**After**:
```js
// Stores complete state automatically
saveState() -> updates localStorage immediately
syncToSupabase() -> uploads full state JSON (debounced)
loadFromSupabase() -> downloads complete state with timestamp check
```

### Device Identification
- Each device gets a unique ID on first use
- Stored in localStorage as `quitr-device-id`
- Format: `device-{timestamp}-{randomId}`
- Allows tracking separate streaks per device
- Enables proper multi-device sync

### Broken Code Removed
- ❌ Deleted broken `syncToGoogleForm()` function (referenced undefined FORM_URL, ENTRY_TYPE, ENTRY_DATA)
- ❌ Removed Google Forms sync attempts in `startJourney()`
- ❌ Removed `loadFromSheets()` (non-functional)
- ❌ Deleted forced `location.reload()` timeout that caused sync issues

### State Management Improvements
- ✅ Added `lastModified` timestamp to state
- ✅ Server uses `updated_at` for comparison
- ✅ Proper "newest wins" conflict resolution
- ✅ Error messages on sync failures

---

## 🐛 Bug Fixes

### 1. Data Loss on loadFromSupabase()
**Issue**: Function tried to find "MAX" values from last 10 records and reconstruct fake data
```js
// BROKEN:
let maxUrges = 0, maxNaturals = 0;
for (let i = 0; i < data.length; i++) {
  if (row.urges > maxUrges) maxUrges = row.urges; // Just a number!
}
// Creates fake urges array with wrong timestamps
for (let i = 0; i < maxUrges; i++) state.urges.push({time: Date.now()});
```

**Fix**: Now loads complete JSON state from `state_data` field
```js
// FIXED:
if (data[0].state_data) {
  const remoteState = JSON.parse(data[0].state_data);
  state = { ...state, ...remoteState }; // Full merge
}
```

### 2. Race Conditions in Sync
**Issue**: Multiple rapid changes could cause lost updates due to overlapping Supabase calls

**Fix**: Added sync status tracking
```js
let syncInProgress = false;
if (syncInProgress) return; // Skip if already syncing
syncInProgress = true;
// ... do sync ...
syncInProgress = false;
```

### 3. Inefficient Saves
**Issue**: Multiple `localStorage.setItem()` calls on same state change

**Fix**: Consolidated to single save point in `saveState()`
```js
saveState() {
  state.lastModified = Date.now();
  localStorage.setItem(STORAGE_KEY, JSON.stringify(state)); // Once here
  // Debounce Supabase sync 1 second
}
```

### 4. Page Reload After Journey Start
**Issue**: Forced `location.reload()` after 2 seconds caused sync to fail and lose data
```js
// BROKEN - removed:
setTimeout(() => {
  if (state.startTime) {
    location.reload(); // Nukes pending sync!
  }
}, 2000);
```

**Fix**: Direct UI update without reload
```js
updateUI();
startCounter();
showToast('Journey started!');
syncToSupabase(); // Explicit sync instead of reload
```

### 5. Manual Sync Only
**Issue**: User had to manually click "Sync" button for cloud updates

**Fix**: Now auto-syncs after every action
```js
// Every state change calls saveState() which debounces syncToSupabase()
// When you log urge, load natural, etc. -> auto-synced within 1 second
```

### 6. Button Label Confusion
**Changed**: 
- "Test Save" → "Push Data" (more intuitive)
- "Sync" → "Sync Now" (clarifies it's downloading)

### 7. Error Handling
**Added**:
- Toast notifications on sync errors
- Console logging for debugging
- Try/catch blocks around all network calls
- Proper async/await with error feedback

---

## 📊 Performance Improvements
- Debounced Supabase syncs (1 second) prevents API hammering
- Checked if device record exists before INSERT (prevents duplicates)
- PATCH updates reuse existing record (no orphaned rows)
- Removed redundant localStorage calls

---

## ✨ New Features
- Device identification system (auto-generated)
- Multi-device detection and sync on app load
- Manual push/pull buttons for explicit control
- Better toast notifications for sync status
- Full state history (not just counts)

---

## 🚀 How to Use (Updated)

### Basic Usage (No Manual Sync Needed)
Just use the app normally:
1. Set your last relapse time
2. Log urges, naturals, journal entries
3. Everything auto-syncs to Supabase automatically

### Multi-Device Setup
**Device A**:
- Use app normally (auto-syncs)

**Device B**:  
- Opens app → auto-loads latest state from Supabase
- If you want to pull latest: Analytics → "Sync Now"
- If you want to push your version: Analytics → "Push Data"

### Offline
- App works offline automatically (uses localStorage)
- Next online sync catches it up
- Newer data always wins (timestamp-based)

---

## 🔒 Security Notes
- Device IDs are unique but not cryptographically secure (localStorage is local)
- Supabase keys are embedded (anon key is safe but visible in HTML)
- For production: implement proper authentication and RLS policies

---

## Testing Checklist
- [x] Auto-sync on each state change
- [x] Multi-device data merge works
- [x] No page reloads
- [x] Error messages appear on sync failure
- [x] localStorage always up-to-date first
- [x] No more data reconstruction bugs
- [x] Removed broken Google Forms code
- [x] Device ID persists across page loads
- [x] First device gets new Supabase record
- [x] Second device updates existing record
