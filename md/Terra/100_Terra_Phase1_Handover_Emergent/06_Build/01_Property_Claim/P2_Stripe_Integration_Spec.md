# P2: Stripe Integration Specification

## Overview
Implement full Stripe integration for **paid monitoring subscriptions**. Users get a **30-day free trial** automatically when they claim a property. At day 30, they either:
1. **Subscribe** ($20/month) → monitoring continues (monitoring_state = subscribed)
2. **Do nothing** → monitoring pauses (monitoring_state = paused)

This ties the entire trust loop together: real parcel → visible on map → satellite icon shows status → payment keeps it active.

## Dependency
**Blocked by:** P1_Satellite_Icon_State_Logic_Spec (must have real parcel + icon working first)  
**Unblocks:** None (this is the final P2 feature)

## Critical Note
**Do NOT implement this against mock parcel data.** Wait until P1 is complete (DCDB integration, real cadastral boundaries, real grid overlap). The subscription only matters when users see their actual land being monitored.

---

## Requirements

### 1. Trial Lifecycle

#### 1.1 Trial Starts on Claim Creation
When user claims a property:
```
POST /api/claims with parcel_id + boundary
  ↓
Create property_claim record:
  - user_id = authenticated user
  - parcel_id = claimed property
  - monitoring_state = "trial_active"
  - trial_begins = NOW
  - trial_ends = NOW + 30 days
  - stripe_customer_id = null (not yet created)
  - stripe_subscription_id = null
  ↓
Return response with monitoring_state = trial_active
```

#### 1.2 Trial Duration
- **Free period:** 30 calendar days from claim creation
- **No upfront payment required** → Low friction entry
- **No card required during trial** (unless user wants early notification)
- User can subscribe anytime during trial to prevent pause

#### 1.3 Day-30 Automation
At day 30:
```
Scheduled job runs daily:
  FOR each claim WHERE monitoring_state = "trial_active" AND trial_ends <= TODAY
    IF claim has NO subscription (stripe_subscription_id = null)
      → Set monitoring_state = "paused"
      → Set paused_date = TODAY
      → Send email: "Your trial ended. Subscribe to continue."
    ELSE IF claim has active subscription
      → Keep monitoring_state = "subscribed" (no change)
```

---

### 2. Stripe Setup

#### 2.1 Stripe Products & Pricing
**Create in Stripe Dashboard:**

**Product:** "Land for Wildlife Monitoring"
- **Price:** $20/month (recurring)
- **Billing cycle:** Monthly
- **Trial period:** 0 days (trial is handled by our day-30 logic, not Stripe)
- **Payment method:** Card required at subscription time

**Example Stripe IDs:**
```
Price ID: price_1234567890ABCDEF
Product ID: prod_1234567890ABCDEF
```

#### 2.2 Stripe API Keys
**Environment variables to set:**
```
STRIPE_SECRET_KEY=sk_live_...
STRIPE_PUBLISHABLE_KEY=pk_live_...
STRIPE_WEBHOOK_SECRET=whsec_...
```

---

### 3. Backend Implementation

#### 3.1 Stripe Service
**File:** `backend/services/stripe_service.py` (new)

```python
import stripe
from datetime import datetime, timedelta
import json

stripe.api_key = os.getenv('STRIPE_SECRET_KEY')

class StripeService:
    """
    Handles all Stripe customer, subscription, and webhook logic.
    """
    
    # Create Stripe customer for user
    async def create_customer(self, user_id: str, email: str, name: str) -> str:
        """
        Input: User info
        Output: Stripe customer ID
        
        Stores customer_id in users table for future reference.
        """
        try:
            customer = stripe.Customer.create(
                email=email,
                name=name,
                metadata={'user_id': user_id}
            )
            return customer.id  # e.g., "cus_12345678"
        except stripe.error.StripeError as e:
            logger.error(f"Failed to create Stripe customer: {e}")
            raise
    
    # Create subscription
    async def create_subscription(
        self, 
        customer_id: str, 
        price_id: str = 'price_1234567890ABCDEF'
    ) -> Dict:
        """
        Input: Stripe customer ID, price ID
        Output: Stripe subscription object
        
        User pays immediately; subscription starts today.
        """
        try:
            subscription = stripe.Subscription.create(
                customer=customer_id,
                items=[{'price': price_id}],
                payment_behavior='error_if_incomplete',
            )
            
            return {
                'subscription_id': subscription.id,
                'status': subscription.status,
                'current_period_end': datetime.fromtimestamp(
                    subscription.current_period_end
                ),
            }
        except stripe.error.CardError as e:
            logger.error(f"Card declined: {e}")
            raise
        except stripe.error.StripeError as e:
            logger.error(f"Subscription creation failed: {e}")
            raise
    
    # Cancel subscription
    async def cancel_subscription(self, subscription_id: str) -> None:
        """
        Input: Stripe subscription ID
        Process: Cancel immediately
        Output: None
        
        Used when user explicitly cancels monitoring.
        """
        try:
            stripe.Subscription.delete(subscription_id)
            logger.info(f"Subscription {subscription_id} cancelled")
        except stripe.error.StripeError as e:
            logger.error(f"Failed to cancel subscription: {e}")
            raise
    
    # Verify webhook signature
    @staticmethod
    def verify_webhook_signature(
        payload: bytes,
        signature: str,
        secret: str
    ) -> Dict:
        """
        Input: Raw HTTP body, signature header, webhook secret
        Output: Verified event data
        Process: Cryptographic verification using HMAC-SHA256
        
        Prevents forged webhook requests.
        """
        try:
            event = stripe.Webhook.construct_event(
                payload, signature, secret
            )
            return event
        except ValueError as e:
            logger.error(f"Invalid webhook payload: {e}")
            raise
        except stripe.error.SignatureVerificationError as e:
            logger.error(f"Invalid webhook signature: {e}")
            raise

stripe_service = StripeService()
```

#### 3.2 Claim Creation Enhancement
**File:** `backend/server.py`

Update `POST /api/claims` to handle trial activation:

```python
@app.post('/api/claims')
async def create_claim(claim_data: ClaimData):
    """
    Create a new property claim with automatic trial activation.
    """
    
    # Validate user is authenticated
    user = await get_current_user()
    
    # Create claim with trial dates
    trial_begins = datetime.utcnow()
    trial_ends = trial_begins + timedelta(days=30)
    
    claim = {
        'user_id': user.id,
        'parcel_id': claim_data.parcel_id,
        'boundary': claim_data.boundary,
        'monitoring_state': 'trial_active',  # Auto-activated
        'trial_begins': trial_begins,
        'trial_ends': trial_ends,
        'stripe_customer_id': None,
        'stripe_subscription_id': None,
        'created_at': datetime.utcnow(),
    }
    
    # Persist to database
    result = await db.property_claims.insert_one(claim)
    
    return {
        'claim_id': str(result.inserted_id),
        'parcel_id': claim_data.parcel_id,
        'monitoring_state': 'trial_active',
        'trial_ends': trial_ends.isoformat(),
        'message': f'Claim created! You have 30 days free monitoring.'
    }
```

#### 3.3 Subscription Endpoint
**Endpoint:** `POST /api/subscription/create-checkout`

User clicks "Subscribe" during their trial (or after pausing):

```python
@app.post('/api/subscription/create-checkout')
async def create_checkout_session(request_data: Dict):
    """
    Create Stripe Checkout Session for subscription.
    
    Request:
    {
        "claim_id": "claim_123",
        "parcel_id": "1234-567",
        "success_url": "https://app.terraknowledge.com/success",
        "cancel_url": "https://app.terraknowledge.com/cancel"
    }
    """
    
    user = await get_current_user()
    claim_id = request_data['claim_id']
    
    # Get claim
    claim = await db.property_claims.find_one({
        '_id': ObjectId(claim_id),
        'user_id': user.id
    })
    
    if not claim:
        raise HTTPException(status_code=404, detail='Claim not found')
    
    # Create or get Stripe customer
    if not claim.get('stripe_customer_id'):
        customer_id = await stripe_service.create_customer(
            user_id=user.id,
            email=user.email,
            name=user.name
        )
        # Store customer ID
        await db.property_claims.update_one(
            {'_id': ObjectId(claim_id)},
            {'$set': {'stripe_customer_id': customer_id}}
        )
    else:
        customer_id = claim['stripe_customer_id']
    
    # Create Checkout Session
    session = stripe.checkout.Session.create(
        customer=customer_id,
        payment_method_types=['card'],
        line_items=[{
            'price': 'price_1234567890ABCDEF',  # $20/month
            'quantity': 1,
        }],
        mode='subscription',
        success_url=request_data['success_url'] + '?session_id={CHECKOUT_SESSION_ID}',
        cancel_url=request_data['cancel_url'],
        metadata={
            'claim_id': claim_id,
            'parcel_id': claim['parcel_id'],
            'user_id': user.id
        }
    )
    
    return {
        'checkout_url': session.url,
        'session_id': session.id
    }
```

#### 3.4 Subscription Success Handler
**Endpoint:** `POST /api/subscription/success`

After user completes Stripe Checkout:

```python
@app.post('/api/subscription/success')
async def subscription_success(request_data: Dict):
    """
    Verify Checkout Session was successful and activate subscription.
    
    Request:
    {
        "session_id": "cs_live_12345..."
    }
    """
    
    user = await get_current_user()
    session_id = request_data['session_id']
    
    # Retrieve Checkout Session from Stripe
    session = stripe.checkout.Session.retrieve(session_id)
    
    if session.payment_status != 'paid':
        raise HTTPException(status_code=400, detail='Payment not completed')
    
    # Get subscription ID
    subscription_id = session.subscription
    
    # Find claim by metadata
    claim_id = session.metadata['claim_id']
    parcel_id = session.metadata['parcel_id']
    
    # Update claim in database
    await db.property_claims.update_one(
        {'_id': ObjectId(claim_id)},
        {'$set': {
            'stripe_subscription_id': subscription_id,
            'monitoring_state': 'subscribed',
            'subscription_started': datetime.utcnow(),
        }}
    )
    
    return {
        'status': 'success',
        'monitoring_state': 'subscribed',
        'subscription_id': subscription_id,
        'message': f'Subscription active! Monitoring continues.'
    }
```

#### 3.5 Day-30 Automation Job
**File:** `backend/jobs/trial_expiration_job.py` (new)

Runs daily to transition expired trials to paused:

```python
import asyncio
from datetime import datetime, timedelta
from motor.motor_asyncio import AsyncIOMotorClient
import logging

logger = logging.getLogger(__name__)

async def check_expired_trials(db):
    """
    Daily scheduled job: Find all trials that have expired and pause them.
    
    Called daily at 00:00 UTC by APScheduler or similar.
    """
    
    now = datetime.utcnow()
    
    # Find claims where trial has ended and no subscription exists
    expired_trials = await db.property_claims.find({
        'monitoring_state': 'trial_active',
        'trial_ends': {'$lte': now},
        'stripe_subscription_id': None,
    }).to_list(length=None)
    
    logger.info(f"Found {len(expired_trials)} expired trials")
    
    for claim in expired_trials:
        # Transition to paused
        await db.property_claims.update_one(
            {'_id': claim['_id']},
            {'$set': {
                'monitoring_state': 'paused',
                'paused_date': now,
            }}
        )
        
        # Send email notification
        user = await db.users.find_one({'_id': claim['user_id']})
        await send_trial_expired_email(
            user_email=user['email'],
            parcel_address=claim['parcel_address'],
            subscription_url='https://app.terraknowledge.com/subscribe'
        )
        
        logger.info(f"Claim {claim['_id']} transitioned to paused")

# Schedule with APScheduler
scheduler.add_job(
    check_expired_trials,
    'cron',
    hour=0, minute=0,  # Daily at midnight UTC
    args=[db],
    id='check_expired_trials'
)
```

#### 3.6 Stripe Webhook Handler
**Endpoint:** `POST /webhooks/stripe`

Receives events from Stripe when subscriptions change:

```python
@app.post('/webhooks/stripe')
async def stripe_webhook(request):
    """
    Webhook handler for Stripe events.
    
    Listens for:
    - customer.subscription.created
    - customer.subscription.updated
    - invoice.payment_succeeded
    - invoice.payment_failed
    - customer.subscription.deleted
    
    Idempotent: Can safely process the same event multiple times.
    """
    
    payload = await request.body()
    signature = request.headers.get('stripe-signature')
    
    try:
        # Verify webhook signature
        event = stripe_service.verify_webhook_signature(
            payload=payload,
            signature=signature,
            secret=os.getenv('STRIPE_WEBHOOK_SECRET')
        )
    except stripe.error.SignatureVerificationError as e:
        logger.error(f"Webhook signature verification failed: {e}")
        return {'error': 'Signature verification failed'}, 403
    
    event_type = event['type']
    event_data = event['data']['object']
    
    # Idempotency: Check if we've already processed this event
    idempotency_key = event['id']
    existing = await db.stripe_webhooks.find_one({
        'stripe_event_id': idempotency_key
    })
    
    if existing:
        logger.info(f"Event {idempotency_key} already processed, skipping")
        return {'status': 'ok'}, 200
    
    # Process events
    if event_type == 'customer.subscription.updated':
        subscription = event_data
        
        # Find claim by subscription ID
        claim = await db.property_claims.find_one({
            'stripe_subscription_id': subscription.id
        })
        
        if not claim:
            logger.warning(f"Claim not found for subscription {subscription.id}")
            return {'status': 'ok'}, 200
        
        # Determine new monitoring state based on subscription status
        if subscription.status == 'active':
            new_state = 'subscribed'
        elif subscription.status == 'past_due':
            new_state = 'paused'  # Payment overdue, pause monitoring
        elif subscription.status == 'canceled':
            new_state = 'paused'
        else:
            new_state = claim['monitoring_state']  # No change
        
        # Update claim
        await db.property_claims.update_one(
            {'_id': claim['_id']},
            {'$set': {'monitoring_state': new_state}}
        )
        
        logger.info(f"Claim {claim['_id']} updated to {new_state}")
    
    elif event_type == 'invoice.payment_failed':
        subscription = event_data['subscription']
        
        # Find claim
        claim = await db.property_claims.find_one({
            'stripe_subscription_id': subscription
        })
        
        if claim:
            # Mark as paused (payment failed)
            await db.property_claims.update_one(
                {'_id': claim['_id']},
                {'$set': {'monitoring_state': 'paused'}}
            )
            
            # Send email to user
            user = await db.users.find_one({'_id': claim['user_id']})
            await send_payment_failed_email(user['email'])
            
            logger.info(f"Claim {claim['_id']} paused due to payment failure")
    
    # Record successful webhook processing (idempotency)
    await db.stripe_webhooks.insert_one({
        'stripe_event_id': idempotency_key,
        'event_type': event_type,
        'processed_at': datetime.utcnow(),
    })
    
    return {'status': 'ok'}, 200
```

---

### 4. Frontend Implementation

#### 4.1 Subscribe Button in UI
**File:** `frontend/src/flows/PropertyClaim/NDVICard.tsx`

When trial is active, show "Subscribe" button:

```tsx
export function NDVICard({ claim, ndvi }) {
  const [showSubscribeModal, setShowSubscribeModal] = useState(false);
  
  const daysRemaining = calculateDaysRemaining(claim.trial_ends);
  const isTrialActive = claim.monitoring_state === 'trial_active';
  const isPaused = claim.monitoring_state === 'paused';
  
  return (
    <View style={styles.container}>
      {/* NDVI display */}
      <Text style={styles.title}>Vegetation health (from satellite)</Text>
      <Text style={styles.ndvi}>{ndvi?.current?.toFixed(2) || '--'}</Text>
      
      {/* Trial status */}
      {isTrialActive && (
        <View style={styles.trialBanner}>
          <Text style={styles.trialText}>
            Free trial · {daysRemaining} days left
          </Text>
          <Button
            title="Activate Subscription"
            onPress={() => setShowSubscribeModal(true)}
          />
        </View>
      )}
      
      {/* Paused status */}
      {isPaused && (
        <View style={styles.pausedBanner}>
          <Text style={styles.pausedText}>Trial ended · Monitoring paused</Text>
          <Button
            title="Upgrade to Resume"
            onPress={() => setShowSubscribeModal(true)}
          />
        </View>
      )}
      
      {/* Subscription modal */}
      {showSubscribeModal && (
        <SubscriptionModal 
          claim={claim}
          onClose={() => setShowSubscribeModal(false)}
          onSuccess={() => {
            // Update claim state
            refreshClaimData(claim.id);
            setShowSubscribeModal(false);
          }}
        />
      )}
    </View>
  );
}
```

#### 4.2 Subscription Modal
**File:** `frontend/src/components/SubscriptionModal.tsx`

```tsx
export function SubscriptionModal({ claim, onClose, onSuccess }) {
  const [loading, setLoading] = useState(false);
  
  const handleSubscribe = async () => {
    setLoading(true);
    
    try {
      // Create checkout session
      const response = await fetch('/api/subscription/create-checkout', {
        method: 'POST',
        body: JSON.stringify({
          claim_id: claim.id,
          parcel_id: claim.parcel_id,
          success_url: 'https://app.terraknowledge.com/subscription/success',
          cancel_url: 'https://app.terraknowledge.com/subscription/cancel',
        })
      });
      
      const { checkout_url } = await response.json();
      
      // Redirect to Stripe Checkout
      window.location.href = checkout_url;
      
    } catch (error) {
      console.error('Failed to start subscription:', error);
    } finally {
      setLoading(false);
    }
  };
  
  return (
    <Modal visible={true} animationType="slide">
      <SafeAreaView>
        <Text style={styles.title}>Upgrade to Paid Monitoring</Text>
        
        <Text style={styles.price}>$20/month</Text>
        
        <View style={styles.features}>
          <Text>✓ Continuous NDVI monitoring</Text>
          <Text>✓ Email alerts on major changes</Text>
          <Text>✓ Extended data history</Text>
          <Text>✓ Export reports</Text>
        </View>
        
        <Button
          title="Start Subscription"
          onPress={handleSubscribe}
          disabled={loading}
        />
        
        <Button
          title="Cancel"
          onPress={onClose}
        />
      </SafeAreaView>
    </Modal>
  );
}
```

#### 4.3 Post-Subscription Success
**File:** `frontend/src/screens/SubscriptionSuccessScreen.tsx`

After user completes Stripe Checkout:

```tsx
export function SubscriptionSuccessScreen({ route }) {
  const [loading, setLoading] = useState(true);
  const [status, setStatus] = useState('processing');
  
  useEffect(() => {
    const verifySubscription = async () => {
      const { session_id } = route.params;
      
      try {
        const response = await fetch('/api/subscription/success', {
          method: 'POST',
          body: JSON.stringify({ session_id })
        });
        
        const data = await response.json();
        
        if (data.status === 'success') {
          setStatus('success');
          // Redirect to property screen after delay
          setTimeout(() => {
            navigation.navigate('PropertyClaim');
          }, 2000);
        } else {
          setStatus('error');
        }
      } catch (error) {
        setStatus('error');
      } finally {
        setLoading(false);
      }
    };
    
    verifySubscription();
  }, []);
  
  return (
    <View style={styles.container}>
      {status === 'processing' && <ActivityIndicator />}
      {status === 'success' && (
        <View>
          <Text style={styles.title}>✓ Subscription Active!</Text>
          <Text style={styles.message}>
            Your property is now on continuous monitoring.
          </Text>
          <Text style={styles.subtext}>Redirecting...</Text>
        </View>
      )}
      {status === 'error' && (
        <View>
          <Text style={styles.title}>Subscription Failed</Text>
          <Text style={styles.message}>
            Please try again or contact support.
          </Text>
        </View>
      )}
    </View>
  );
}
```

---

### 5. Data Flow (Complete Lifecycle)

```
USER CLAIMS PROPERTY
  ↓
Create claim with monitoring_state = "trial_active"
  ↓ (trial_begins = TODAY, trial_ends = TODAY + 30 days)
  ↓
Satellite icon appears (green)
NDVI monitoring starts immediately
  ↓
SCENARIO A: User subscribes during trial
  ↓
  User clicks "Subscribe" button
    ↓
  POST /api/subscription/create-checkout
    ↓
  Stripe Checkout Session created
    ↓
  User enters card details + completes payment
    ↓
  Stripe calls POST /webhooks/stripe (customer.subscription.updated)
    ↓
  Webhook updates claim: monitoring_state = "subscribed"
  stripe_subscription_id stored
    ↓
  Satellite icon remains green
  NDVI continues (no interruption)

SCENARIO B: User does nothing (no subscription)
  ↓
  Day 30 arrives
    ↓
  Daily job runs: check_expired_trials()
    ↓
  Finds claim with trial_ends <= TODAY and NO subscription
    ↓
  Updates claim: monitoring_state = "paused"
  Sets paused_date = TODAY
    ↓
  Satellite icon fades (gray, 50% opacity)
  NDVI stops updating
  Email sent: "Trial ended. Upgrade to continue."
    ↓
  User can click "Upgrade to Resume" anytime
    ↓
  POST /api/subscription/create-checkout
    ↓
  (Same workflow as Scenario A)
    ↓
  monitoring_state = "subscribed"
  Icon turns green again
```

---

### 6. Testing Checklist

#### Unit Tests
- [ ] `test_trial_created_on_claim()` - monitoring_state = trial_active when claim created
- [ ] `test_trial_duration_30_days()` - trial_ends = created_at + 30 days
- [ ] `test_stripe_customer_created()` - Customer created in Stripe
- [ ] `test_subscription_created()` - Subscription created for $20/month
- [ ] `test_expired_trial_transitioned()` - Expired trial becomes paused
- [ ] `test_webhook_signature_verified()` - Invalid signatures rejected
- [ ] `test_webhook_idempotent()` - Duplicate events processed once

#### Integration Tests
- [ ] Create claim → trial_active set with 30-day duration
- [ ] Trial → subscription endpoint redirects to Stripe Checkout
- [ ] Stripe Checkout completed → subscription_id stored
- [ ] Webhook received → monitoring_state updated to subscribed
- [ ] Day 30 job → expired trial transitioned to paused
- [ ] Paused claim → can subscribe again

#### E2E Tests
- [ ] User claims property → trial_active, satellite icon green
- [ ] Icon shows "Trial ends in 30 days"
- [ ] User clicks "Subscribe" → Stripe Checkout appears
- [ ] User enters card → payment succeeds
- [ ] Stripe webhook received → claim updates to subscribed
- [ ] Icon remains green, uninterrupted monitoring
- [ ] Day 30 without subscription → trial expires, monitoring pauses
- [ ] Paused claim shows "Upgrade to Resume"
- [ ] User upgrades paused claim → subscription restarted, icon green

#### Payment Scenarios
- [ ] Successful card payment → subscription active
- [ ] Card declined → user sees error, trial continues
- [ ] Payment failed webhook → monitoring paused, email sent
- [ ] Subscription canceled by user → monitoring paused
- [ ] Subscription resumption → monitoring resumes

#### Webhook Security
- [ ] Valid signature passes verification
- [ ] Invalid signature rejected (403)
- [ ] Tampered payload rejected
- [ ] Duplicate event (idempotency key) processed once
- [ ] Event processed within expected timeframe

---

### 7. Success Criteria

- [ ] Claim automatically starts trial (30-day, no upfront payment)
- [ ] Stripe customer created when user subscribes
- [ ] Subscription created for $20/month
- [ ] Day-30 automation transitions expired trials to paused
- [ ] Payment success updates monitoring_state to subscribed
- [ ] Payment failure updates monitoring_state to paused
- [ ] Webhooks are idempotent (safe to process duplicates)
- [ ] Satellite icon reflects current monitoring_state
- [ ] All E2E test cases pass
- [ ] Sensitive data (API keys, webhook secrets) not exposed

---

### 8. Files to Create/Modify

| File | Type | Purpose |
|------|------|---------|
| `backend/services/stripe_service.py` | NEW | Stripe API integration (customer, subscription, webhooks) |
| `backend/jobs/trial_expiration_job.py` | NEW | Daily job to expire trials at day 30 |
| `backend/server.py` | MODIFY | Add /api/subscription/* and /webhooks/stripe endpoints |
| `frontend/src/components/SubscriptionModal.tsx` | NEW | UI for subscription flow |
| `frontend/src/screens/SubscriptionSuccessScreen.tsx` | NEW | Post-payment confirmation |
| `frontend/src/flows/PropertyClaim/NDVICard.tsx` | MODIFY | Add "Subscribe" and "Upgrade" buttons |
| Database schema | MODIFY | Add stripe_customer_id, stripe_subscription_id, subscription_started columns |
| `.env` | MODIFY | Add STRIPE_SECRET_KEY, STRIPE_PUBLISHABLE_KEY, STRIPE_WEBHOOK_SECRET |

---

### 9. Timeline Estimate
- Stripe API integration (stripe_service.py): **2-3 hours**
- Backend endpoints (checkout, success, webhook): **3-4 hours**
- Day-30 automation job: **1-2 hours**
- Frontend subscription UI: **2-3 hours**
- Testing + webhook simulation: **2-3 hours**
- **Total: 11-15 hours**

---

### 10. Security Checklist

- [ ] API keys stored in environment variables (not hardcoded)
- [ ] Webhook signature verified before processing
- [ ] Webhook idempotency prevents duplicate charges
- [ ] User auth required on all subscription endpoints
- [ ] Stripe customer ID verified before charge
- [ ] PCI compliance: Never store raw card data
- [ ] HTTPS enforced on all payment flows
- [ ] Sensitive errors logged but not exposed to client

---

## Why This Approach

**Trial First, Payment Second:**
- Users see satellite data + parcel boundary working perfectly
- At day 30, they understand the value and are willing to pay
- Payment is the *retention mechanism*, not the entry barrier

**$20/Month:**
- Affordable for small-property owners
- Simple pricing (no tiers, no complexity)
- Sustainable for ongoing NDVI updates

**Idempotent Webhooks:**
- If Stripe resends an event, it's processed safely once
- Prevents double-charging or duplicate state changes
- Critical for production reliability

**Auto-Pause at Day 30:**
- No surprise charges
- User explicitly subscribes (opt-in)
- Clear expectation management

---

## Next Steps (After Stripe Complete)

Once this spec is implemented, Phase 1 is complete. The trust loop is closed:
- ✅ Real DCDB cadastral data
- ✅ Parcel boundary visible on map
- ✅ Grid overlay for landscape context
- ✅ Satellite icon shows monitoring status
- ✅ Payment keeps monitoring active

Ready for:
1. **Real user testing** with Land for Wildlife coordinators
2. **Screen recording** for demo/marketing
3. **Phase 2 expansion** (multi-region, advanced features)
