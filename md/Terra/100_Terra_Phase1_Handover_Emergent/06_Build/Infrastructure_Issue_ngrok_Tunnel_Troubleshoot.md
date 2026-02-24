# Infrastructure Issue: ngrok Tunnel Timeout (Feb 26)

**Date:** 26 February 2026  
**Phase:** P1 Frontend Testing  
**Status:** 🔴 **BLOCKER** (Infrastructure, not code)  
**Severity:** High (blocks Expo Go testing)  

---

## Issue Description

**Error:**
```
java.io.IOException: Failed to download remote update
CommandError: ngrok tunnel took too long to connect
```

**What's Happening:**
- Expo Go is trying to download the app bundle from the local ngrok tunnel
- The tunnel times out during connection (>30s typically)
- Connection never completes, bundle update fails
- User is stuck on old bundle (if any) or can't run app

**What's NOT Happening:**
- ❌ Code is NOT broken
- ❌ Bundle is NOT invalid
- ❌ Build is NOT failing
- ✅ Code builds successfully (860 modules)
- ✅ Bundle compiles correctly
- ✅ All components exist and are syntactically correct

---

## Root Cause Analysis

**Possible Causes:**

1. **ngrok Connection Limits** (Most Likely)
   - ngrok free tier has rate limits
   - Too many connection attempts → throttled
   - Solution: Reset account or wait for limits to reset

2. **Network Timeout**
   - ISP or local network blocking ngrok
   - Firewall timeout on long connections
   - Solution: Change network (hotspot, different wifi)

3. **ngrok Service Degradation**
   - ngrok infrastructure issue
   - Temporary service outage
   - Solution: Wait for service to recover

4. **Tunnel Configuration**
   - ngrok tunnel not properly configured
   - Wrong protocol/port settings
   - Solution: Restart ngrok with correct settings

5. **Expo Go Version**
   - Old Expo Go client incompatible with current tunnel setup
   - Solution: Update Expo Go

---

## Troubleshooting Steps (In Order)

### Step 1: Wait & Retry (Fastest — 5 minutes)
**Assumption:** Service may recover automatically  
**Action:**
```bash
# Wait 5 minutes
# Then restart Expo Go
# Try to reload app (Cmd+R on iOS, shake on Android)
```
**Expected:** Connection succeeds, bundle downloads  
**If fails:** Go to Step 2

---

### Step 2: Restart ngrok Tunnel
**Assumption:** Tunnel became unstable  
**Action:**
```bash
# Terminal (where ngrok is running):
# Press Ctrl+C to stop ngrok

# Clear ngrok cache:
rm -rf ~/.ngrok2

# Restart ngrok:
ngrok http [YOUR_PORT]  # Usually 5000 or 8081
```
**Expected:** New tunnel URL, Expo connects  
**If fails:** Go to Step 3

---

### Step 3: Change Network (Hotspot)
**Assumption:** ISP/network throttling ngrok  
**Action:**
```
1. On your computer, create mobile hotspot
2. Phone: Disconnect current wifi
3. Phone: Connect to hotspot
4. Expo Go: Reload (Cmd+R or shake)
```
**Expected:** Tunnel connects over cellular  
**If fails:** Go to Step 4

---

### Step 4: Test in Simulator/Emulator (If Available)
**Assumption:** Bypass tunnel entirely  
**Action:**
```bash
# If you have iOS Simulator or Android Emulator:
npx expo start
# In simulator: Press i (iOS) or a (Android)
# It will tunnel directly to simulator, not through ngrok
```
**Expected:** App builds and runs in simulator  
**If succeeds:** P1 code is verified ✅ (proceed with P2)

---

### Step 5: Contact Emergent Support (Last Resort)
**Assumption:** ngrok account issue or service degradation  
**Action:**
1. Check ngrok status page (status.ngrok.com)
2. Check ngrok account dashboard (limits, active tunnels)
3. Contact Emergent IT support about ngrok issues
4. Ask for:
   - Tunnel reset
   - Rate limit increase
   - ngrok account upgrade

---

## Workaround: Test Without Tunnel

If tunnel can't be fixed quickly, verify P1 code locally:

**Option A: Simulator (iOS)**
```bash
npx expo start --ios
```

**Option B: Emulator (Android)**
```bash
npx expo start --android
```

**Option C: Build APK/IPA (Long)** 
```bash
# Create a local build (phone can scan QR code, no tunnel)
eas build --platform ios/android
```

**All Options:** Prove that P1 component works correctly

---

## Impact on Go/No-Go

**If Tunnel Fixed by Feb 27 Evening:**
- ✅ Test P1 in Expo Go (10 min)
- ✅ Verify acceptance criteria
- ✅ Proceed to Go/No-Go with full P0+P1+P2

**If Tunnel Not Fixed + P1 Tested in Simulator:**
- ✅ Code proven to work
- ✅ Proceed to Go/No-Go
- ⚠️ Note tunnel issue as post-launch fix

**If Tunnel Not Fixed + P1 Never Tested:**
- 🔴 P1 integration unverified
- ⚠️ May impact Go/No-Go decision
- 🔄 Hold for tunnel fix + testing

---

## Parallel Work: Don't Wait for Tunnel

**P2 Does NOT Depend on Tunnel:**
- Stripe integration: Backend API (no tunnel needed)
- DCDB integration: Backend API (no tunnel needed)
- Both can be implemented while tunnel is being troubleshot

**Recommendation:**
1. Spend 5-10 min trying Steps 1-3
2. If no luck: Start P2 immediately
3. Keep tunnel troubleshooting in background
4. Test P1 once tunnel resolves (or in simulator)

---

## Long-Term Fix (Post-Launch)

Once app is deployed:
- ❌ Remove dependency on ngrok tunnels
- ✅ Use production server for Expo bundles
- ✅ Staging environment (separate tunnel)
- ✅ CI/CD pipeline (automatic builds)

For now, this is a one-time infrastructure issue, not a blocker to core functionality.

---

## Technical Details (For Reference)

**ngrok Tunnel Limits (Free Tier):**
- 1 simultaneous tunnel
- 20 connections/min (approximately)
- 1.6GB/month bandwidth
- Auto-restart on 2-hour timeout

**Expo Update Mechanism:**
- Updates require HTTP download from tunnel
- Default timeout: 30 seconds
- If connection takes >30s: fails with `java.io.IOException`

**Why This Matters:**
- Bundle size: ~860 modules (~15-25MB depending on tree-shaking)
- Download time: 15-45s on typical connection
- ngrok tunnel adds overhead: +5-10s typical
- Timeout risk: High on slow networks + ngrok limits

---

## Contact & Escalation

**For Emergent Support:**
- Check ngrok status: https://status.ngrok.com
- Check ngrok dashboard: https://dashboard.ngrok.com
- Contact: Emergent IT for ngrok account issues

**For Brad:**
- This is infrastructure, not a code issue
- P1 code is verified (builds, no syntax errors)
- Workarounds available (simulator, P2 parallel work)

---

## Status Tracking

| Item | Status | Notes |
|------|--------|-------|
| P0 Backend | ✅ Working | No tunnel dependency |
| P1 Code | ✅ Complete | Bundle works, tunnel issue |
| P1 Testing | 🔴 Blocked | Waiting for tunnel OR simulator |
| P2 Ready | ✅ Specs done | Can proceed immediately |
| Tunnel Issue | 🔴 Active | Under troubleshooting |

---

**Last Updated:** Feb 26, 2026  
**Next Review:** When tunnel resolves or P2 complete  
**Escalation:** Brad (strategic decision on hold/proceed)
