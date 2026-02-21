# Supabase Schema – Claims Management

**Database:** Terra Phase 1  
**Version:** 1.0  
**Status:** Active  

---

## Table: `claims`

Persistent storage for all property claims, monitoring states, and Stripe linkage.

```sql
CREATE TABLE claims (
  -- Primary Key
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  
  -- User Reference (links to Supabase auth)
  user_id UUID NOT NULL REFERENCES auth.users(id) ON DELETE CASCADE,
  
  -- Property Data (immutable after creation)
  parcel_id TEXT NOT NULL,
  lot_number TEXT,
  property_name TEXT,
  owner_name TEXT NOT NULL,
  email TEXT NOT NULL,
  boundary JSONB NOT NULL, -- GeoJSON Polygon
  area_m2 INTEGER,
  address TEXT,
  
  -- Monitoring State (mutable, drives icon + behavior)
  monitoring_state TEXT NOT NULL DEFAULT 'inactive',
  -- Allowed values: 'inactive' | 'trial_active' | 'subscribed' | 'paused'
  
  -- Trial Period (set when trial starts)
  trial_start_date TIMESTAMP WITH TIME ZONE,
  trial_end_date TIMESTAMP WITH TIME ZONE,
  
  -- Stripe Integration (nullable, populated after subscription)
  stripe_customer_id TEXT UNIQUE,
  stripe_subscription_id TEXT UNIQUE,
  
  -- Audit Timestamps
  created_at TIMESTAMP WITH TIME ZONE DEFAULT NOW(),
  updated_at TIMESTAMP WITH TIME ZONE DEFAULT NOW(),
  deleted_at TIMESTAMP WITH TIME ZONE, -- Soft delete (optional)
  
  -- Constraints
  UNIQUE(user_id, parcel_id, lot_number), -- One claim per property/lot per user
  CHECK (monitoring_state IN ('inactive', 'trial_active', 'subscribed', 'paused'))
);

-- Indexes for performance
CREATE INDEX idx_claims_user_id ON claims(user_id);
CREATE INDEX idx_claims_monitoring_state ON claims(monitoring_state);
CREATE INDEX idx_claims_trial_end_date ON claims(trial_end_date) WHERE monitor_state = 'trial_active';
CREATE INDEX idx_claims_stripe_customer_id ON claims(stripe_customer_id);
CREATE INDEX idx_claims_created_at ON claims(created_at DESC);

-- Row-Level Security (RLS)
ALTER TABLE claims ENABLE ROW LEVEL SECURITY;

-- Users can only see their own claims
CREATE POLICY "Users can view own claims"
  ON claims FOR SELECT
  USING (auth.uid() = user_id);

CREATE POLICY "Users can create own claims"
  ON claims FOR INSERT
  WITH CHECK (auth.uid() = user_id);

CREATE POLICY "Users can update own claims"
  ON claims FOR UPDATE
  USING (auth.uid() = user_id);

CREATE POLICY "Users can delete own claims"
  ON claims FOR DELETE
  USING (auth.uid() = user_id);
```

---

## Field Definitions

| Field | Type | Purpose | Example |
|-------|------|---------|---------|
| `id` | UUID | Primary key, auto-generated | `550e8400-e29b-41d4-a716-446655440000` |
| `user_id` | UUID | Links to authenticated user | `auth.uid()` value |
| `parcel_id` | TEXT | DCDB unique parcel ID | `dcdb_001` |
| `lot_number` | TEXT | Lot number (if multi-lot property) | `1`, `2`, etc. |
| `property_name` | TEXT | User-friendly name | `"My Brookfield Property"` |
| `owner_name` | TEXT | Owner name | `"John Doe"` |
| `email` | TEXT | Owner email | `"john@example.com"` |
| `boundary` | JSONB | GeoJSON Polygon | `{"type": "Polygon", "coordinates": [...]}` |
| `area_m2` | INTEGER | Property area in square meters | `52500` |
| `address` | TEXT | Full address string | `"123 Sample Road, Brookfield QLD"` |
| `monitoring_state` | TEXT | Current monitoring status | `"trial_active"`, `"subscribed"`, etc. |
| `trial_start_date` | TIMESTAMP | Trial period start | `2026-02-21T10:00:00Z` |
| `trial_end_date` | TIMESTAMP | Trial period end (Day 30) | `2026-03-23T10:00:00Z` |
| `stripe_customer_id` | TEXT | Stripe customer reference | `"cus_K8q4N5xvZ..."` |
| `stripe_subscription_id` | TEXT | Stripe subscription reference | `"sub_K8q4N5xvZ..."` |
| `created_at` | TIMESTAMP | Record creation time | `NOW()` |
| `updated_at` | TIMESTAMP | Last update time | `NOW()` on each UPDATE |

---

## Monitoring State Flow

```
inactive
  ├─ User hasn't started trial yet
  ├─ Icon: Not shown
  └─ Next: User taps "Start Free Trial"
     ↓
trial_active
  ├─ Day 0-30 of free trial
  ├─ Icon: 🟢 Green/lit
  ├─ trial_end_date set to NOW() + 30 days
  └─ Next: Day 30 arrives
     ├─ If user subscribes: → subscribed
     ├─ If user does nothing: → paused
     └─ If payment fails: → paused
        ↓
subscribed
  ├─ Active paid subscription
  ├─ Icon: 🟢 Green/lit
  ├─ stripe_subscription_id populated
  └─ Next: Payment succeeds or fails
     ├─ Success: Stay subscribed
     └─ Failure: → paused
        ↓
paused
  ├─ Trial expired or payment failed
  ├─ Icon: ⚫ Grey/faded
  ├─ User can reactivate (add payment)
  └─ Next: User reactivates
     └─ → subscribed
```

---

## API Interactions

### Create Claim

```
POST /api/claims
Authorization: Bearer {user_token}

{
  "parcel_id": "dcdb_001",
  "lot_number": "1",
  "property_name": "My Property",
  "owner_name": "John Doe",
  "email": "john@example.com",
  "boundary": { /* GeoJSON */ },
  "area_m2": 52500,
  "address": "123 Sample Road, Brookfield"
}

Response:
{
  "id": "claim_uuid",
  "user_id": "user_uuid",
  "monitoring_state": "inactive",
  "trial_start_date": null,
  "trial_end_date": null,
  "stripe_customer_id": null,
  "stripe_subscription_id": null,
  "created_at": "2026-02-21T10:00:00Z"
}
```

### Start Trial

```
POST /api/claims/{claim_id}/start-trial
Authorization: Bearer {user_token}

Response:
{
  "id": "claim_uuid",
  "monitoring_state": "trial_active",
  "trial_start_date": "2026-02-21T10:00:00Z",
  "trial_end_date": "2026-03-23T10:00:00Z",
  "stripe_customer_id": null,
  "stripe_subscription_id": null,
  "updated_at": "2026-02-21T10:00:00Z"
}
```

### Get Claim

```
GET /api/claims/{claim_id}
Authorization: Bearer {user_token}

Response:
{
  "id": "claim_uuid",
  "user_id": "user_uuid",
  "parcel_id": "dcdb_001",
  "property_name": "My Property",
  "monitoring_state": "trial_active",
  "trial_end_date": "2026-03-23T10:00:00Z",
  "boundary": { /* GeoJSON */ },
  "created_at": "2026-02-21T10:00:00Z"
}
```

### List User Claims

```
GET /api/claims
Authorization: Bearer {user_token}

Response:
[
  { /* claim 1 */ },
  { /* claim 2 */ },
  ...
]
```

### Delete Claim

```
DELETE /api/claims/{claim_id}
Authorization: Bearer {user_token}

Response:
{ "deleted": true }
```

---

## Backend Implementation Notes

### Node.js / Express + Supabase Client

```typescript
// Example: Create claim
import { createClient } from '@supabase/supabase-js';

const supabase = createClient(
  process.env.SUPABASE_URL,
  process.env.SUPABASE_SERVICE_ROLE_KEY
);

app.post('/api/claims', async (req, res) => {
  const { user_id } = req.user; // From JWT
  const { parcel_id, lot_number, boundary, ...rest } = req.body;

  const { data, error } = await supabase
    .from('claims')
    .insert([
      {
        user_id,
        parcel_id,
        lot_number,
        boundary,
        monitoring_state: 'inactive',
        ...rest
      }
    ])
    .select()
    .single();

  if (error) return res.status(400).json({ error: error.message });
  res.json(data);
});

// Example: Start trial
app.post('/api/claims/:claim_id/start-trial', async (req, res) => {
  const { claim_id } = req.params;
  const { user_id } = req.user;

  const trialEndDate = new Date();
  trialEndDate.setDate(trialEndDate.getDate() + 30);

  const { data, error } = await supabase
    .from('claims')
    .update({
      monitoring_state: 'trial_active',
      trial_start_date: new Date().toISOString(),
      trial_end_date: trialEndDate.toISOString()
    })
    .eq('id', claim_id)
    .eq('user_id', user_id)
    .select()
    .single();

  if (error) return res.status(400).json({ error: error.message });
  res.json(data);
});
```

---

## Frontend Integration

### React Hook

```typescript
import { useSupabaseClient, useSession } from '@supabase/auth-helpers-react';

export function useClaims() {
  const supabase = useSupabaseClient();
  const session = useSession();

  const fetchClaims = async () => {
    const { data, error } = await supabase
      .from('claims')
      .select('*')
      .order('created_at', { ascending: false });

    return data;
  };

  const createClaim = async (claimData) => {
    const { data, error } = await supabase
      .from('claims')
      .insert([claimData])
      .select()
      .single();

    return data;
  };

  const startTrial = async (claimId) => {
    const { data, error } = await supabase
      .from('claims')
      .update({
        monitoring_state: 'trial_active',
        trial_start_date: new Date().toISOString(),
        trial_end_date: new Date(Date.now() + 30 * 24 * 60 * 60 * 1000).toISOString()
      })
      .eq('id', claimId)
      .select()
      .single();

    return data;
  };

  return {
    fetchClaims,
    createClaim,
    startTrial
  };
}
```

---

## Monitoring & Maintenance

### Check for Expired Trials (Cron Job)

Run daily at midnight:

```sql
UPDATE claims
SET monitoring_state = 'paused'
WHERE monitoring_state = 'trial_active'
  AND trial_end_date < NOW();
```

### Verify RLS is Working

```sql
-- Non-owner should get empty result
SELECT * FROM claims WHERE user_id != auth.uid();
-- → Should return empty, not error
```

---

## Future Extensions

- **Soft delete:** Use `deleted_at` timestamp for recovery
- **Audit log:** Separate table tracking all state transitions
- **Webhooks:** Supabase webhooks on claim creation / state change
- **Real-time:** Supabase realtime subscriptions for icon state changes
