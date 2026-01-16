---
name: stripe-setup
description: Stripe payment integration guide. Implements subscription plans (3-tier), one-time payments, Customer Portal, and webhooks. Use when adding payment functionality to web applications.
---

# Stripe Setup

Complete guide for implementing Stripe payments including subscriptions and one-time purchases.

## When to Use

- Adding payment functionality to a web app
- Implementing SaaS subscription tiers
- Setting up one-time product purchases
- Configuring Customer Portal for self-service
- Setting up webhook handlers

## Decision Tree

```
Payment Type?
├─ Subscription → Go to "Subscription Setup"
├─ One-time    → Go to "One-time Payment Setup"
└─ Both        → Implement both patterns
```

---

## Step 1: Initial Setup (Common)

### 1.1 Install Dependencies

```bash
npm install stripe @stripe/stripe-js
```

### 1.2 Environment Variables

```env
# .env.local
STRIPE_SECRET_KEY=sk_test_xxx
NEXT_PUBLIC_STRIPE_PUBLISHABLE_KEY=pk_test_xxx
STRIPE_WEBHOOK_SECRET=whsec_xxx
```

### 1.3 Stripe Client Setup

**Server-side (lib/stripe.ts):**

```typescript
import Stripe from 'stripe';

export const stripe = new Stripe(process.env.STRIPE_SECRET_KEY!, {
  apiVersion: '2024-12-18.acacia',
});
```

**Client-side (lib/stripe-client.ts):**

```typescript
import { loadStripe } from '@stripe/stripe-js';

export const stripePromise = loadStripe(
  process.env.NEXT_PUBLIC_STRIPE_PUBLISHABLE_KEY!
);
```

---

## Subscription Setup

### Step S1: Create Products in Stripe Dashboard

Create 3-tier pricing in Stripe Dashboard:

| Tier | Price ID | Monthly | Features |
|------|----------|---------|----------|
| Basic | price_basic_xxx | ¥980 | Core features |
| Pro | price_pro_xxx | ¥1,980 | + Advanced features |
| Enterprise | price_enterprise_xxx | ¥4,980 | + Priority support |

### Step S2: Pricing Page Component

```typescript
// components/pricing.tsx
'use client';

import { useState } from 'react';

const plans = [
  {
    name: 'Basic',
    price: '¥980',
    priceId: 'price_basic_xxx',
    features: ['Feature 1', 'Feature 2'],
  },
  {
    name: 'Pro',
    price: '¥1,980',
    priceId: 'price_pro_xxx',
    features: ['Everything in Basic', 'Feature 3', 'Feature 4'],
    popular: true,
  },
  {
    name: 'Enterprise',
    price: '¥4,980',
    priceId: 'price_enterprise_xxx',
    features: ['Everything in Pro', 'Priority Support', 'Custom Integration'],
  },
];

export function PricingPage() {
  const [loading, setLoading] = useState<string | null>(null);

  const handleSubscribe = async (priceId: string) => {
    setLoading(priceId);
    try {
      const res = await fetch('/api/stripe/checkout', {
        method: 'POST',
        headers: { 'Content-Type': 'application/json' },
        body: JSON.stringify({ priceId, mode: 'subscription' }),
      });
      const { url } = await res.json();
      window.location.href = url;
    } catch (error) {
      console.error(error);
    } finally {
      setLoading(null);
    }
  };

  return (
    <div className="grid md:grid-cols-3 gap-6">
      {plans.map((plan) => (
        <div
          key={plan.name}
          className={`p-6 rounded-lg border ${
            plan.popular ? 'border-blue-500 ring-2 ring-blue-500' : ''
          }`}
        >
          {plan.popular && (
            <span className="bg-blue-500 text-white px-2 py-1 rounded text-sm">
              Popular
            </span>
          )}
          <h3 className="text-xl font-bold mt-2">{plan.name}</h3>
          <p className="text-3xl font-bold mt-2">
            {plan.price}<span className="text-sm">/月</span>
          </p>
          <ul className="mt-4 space-y-2">
            {plan.features.map((feature) => (
              <li key={feature}>✓ {feature}</li>
            ))}
          </ul>
          <button
            onClick={() => handleSubscribe(plan.priceId)}
            disabled={loading === plan.priceId}
            className="w-full mt-6 py-2 bg-blue-600 text-white rounded hover:bg-blue-700 disabled:opacity-50"
          >
            {loading === plan.priceId ? 'Loading...' : 'Subscribe'}
          </button>
        </div>
      ))}
    </div>
  );
}
```

### Step S3: Checkout API Route

```typescript
// app/api/stripe/checkout/route.ts
import { NextRequest, NextResponse } from 'next/server';
import { stripe } from '@/lib/stripe';
import { auth } from '@/lib/auth'; // Your auth solution

export async function POST(req: NextRequest) {
  try {
    const session = await auth();
    if (!session?.user) {
      return NextResponse.json({ error: 'Unauthorized' }, { status: 401 });
    }

    const { priceId, mode } = await req.json();

    const checkoutSession = await stripe.checkout.sessions.create({
      customer_email: session.user.email!,
      mode: mode, // 'subscription' or 'payment'
      payment_method_types: ['card'],
      line_items: [{ price: priceId, quantity: 1 }],
      success_url: `${process.env.NEXT_PUBLIC_APP_URL}/dashboard?success=true`,
      cancel_url: `${process.env.NEXT_PUBLIC_APP_URL}/pricing?canceled=true`,
      metadata: {
        userId: session.user.id,
      },
    });

    return NextResponse.json({ url: checkoutSession.url });
  } catch (error) {
    console.error('Checkout error:', error);
    return NextResponse.json({ error: 'Checkout failed' }, { status: 500 });
  }
}
```

### Step S4: Customer Portal

```typescript
// app/api/stripe/portal/route.ts
import { NextRequest, NextResponse } from 'next/server';
import { stripe } from '@/lib/stripe';
import { auth } from '@/lib/auth';
import { db } from '@/lib/db';

export async function POST(req: NextRequest) {
  try {
    const session = await auth();
    if (!session?.user) {
      return NextResponse.json({ error: 'Unauthorized' }, { status: 401 });
    }

    // Get customer ID from your database
    const user = await db.user.findUnique({
      where: { id: session.user.id },
      select: { stripeCustomerId: true },
    });

    if (!user?.stripeCustomerId) {
      return NextResponse.json({ error: 'No subscription' }, { status: 400 });
    }

    const portalSession = await stripe.billingPortal.sessions.create({
      customer: user.stripeCustomerId,
      return_url: `${process.env.NEXT_PUBLIC_APP_URL}/dashboard`,
    });

    return NextResponse.json({ url: portalSession.url });
  } catch (error) {
    console.error('Portal error:', error);
    return NextResponse.json({ error: 'Portal failed' }, { status: 500 });
  }
}
```

### Step S5: Plan Change API (Upgrade/Downgrade)

```typescript
// app/api/stripe/change-plan/route.ts
import { NextRequest, NextResponse } from 'next/server';
import { stripe } from '@/lib/stripe';
import { auth } from '@/lib/auth';
import { db } from '@/lib/db';

export async function POST(req: NextRequest) {
  try {
    const session = await auth();
    if (!session?.user) {
      return NextResponse.json({ error: 'Unauthorized' }, { status: 401 });
    }

    const { newPriceId } = await req.json();

    const user = await db.user.findUnique({
      where: { id: session.user.id },
      select: { stripeSubscriptionId: true },
    });

    if (!user?.stripeSubscriptionId) {
      return NextResponse.json({ error: 'No subscription' }, { status: 400 });
    }

    // Get current subscription
    const subscription = await stripe.subscriptions.retrieve(
      user.stripeSubscriptionId
    );

    // Update subscription with new price (proration enabled by default)
    const updatedSubscription = await stripe.subscriptions.update(
      user.stripeSubscriptionId,
      {
        items: [
          {
            id: subscription.items.data[0].id,
            price: newPriceId,
          },
        ],
        proration_behavior: 'create_prorations', // Charge/credit difference immediately
        // Or use 'none' to apply at next billing cycle
      }
    );

    return NextResponse.json({
      success: true,
      subscription: {
        status: updatedSubscription.status,
        currentPeriodEnd: updatedSubscription.current_period_end,
      },
    });
  } catch (error) {
    console.error('Plan change error:', error);
    return NextResponse.json({ error: 'Plan change failed' }, { status: 500 });
  }
}
```

### Step S6: Plan Change Component

```typescript
// components/change-plan.tsx
'use client';

import { useState } from 'react';

interface ChangePlanProps {
  currentPriceId: string;
}

const plans = [
  { name: 'Basic', priceId: 'price_basic_xxx', price: 980 },
  { name: 'Pro', priceId: 'price_pro_xxx', price: 1980 },
  { name: 'Enterprise', priceId: 'price_enterprise_xxx', price: 4980 },
];

export function ChangePlan({ currentPriceId }: ChangePlanProps) {
  const [loading, setLoading] = useState(false);

  const handleChangePlan = async (newPriceId: string) => {
    if (newPriceId === currentPriceId) return;

    setLoading(true);
    try {
      const res = await fetch('/api/stripe/change-plan', {
        method: 'POST',
        headers: { 'Content-Type': 'application/json' },
        body: JSON.stringify({ newPriceId }),
      });

      if (res.ok) {
        // Refresh page or update state
        window.location.reload();
      }
    } catch (error) {
      console.error(error);
    } finally {
      setLoading(false);
    }
  };

  const currentPlan = plans.find((p) => p.priceId === currentPriceId);

  return (
    <div className="space-y-4">
      <h3 className="font-bold">Change Plan</h3>
      <p>Current: {currentPlan?.name}</p>
      <div className="flex gap-2">
        {plans.map((plan) => (
          <button
            key={plan.priceId}
            onClick={() => handleChangePlan(plan.priceId)}
            disabled={loading || plan.priceId === currentPriceId}
            className={`px-4 py-2 rounded ${
              plan.priceId === currentPriceId
                ? 'bg-gray-300 cursor-not-allowed'
                : 'bg-blue-600 text-white hover:bg-blue-700'
            }`}
          >
            {plan.name} (¥{plan.price}/月)
          </button>
        ))}
      </div>
    </div>
  );
}
```

---

## One-time Payment Setup

### Step O1: Create Products

Create one-time products in Stripe Dashboard:

| Product | Price ID | Price | Description |
|---------|----------|-------|-------------|
| Basic Package | price_once_basic | ¥5,000 | One-time purchase |
| Pro Package | price_once_pro | ¥15,000 | Premium features |

### Step O2: Product Page Component

```typescript
// components/products.tsx
'use client';

import { useState } from 'react';

const products = [
  {
    id: 'basic-package',
    name: 'Basic Package',
    price: '¥5,000',
    priceId: 'price_once_basic',
    description: 'Perfect for getting started',
    features: ['Feature A', 'Feature B'],
  },
  {
    id: 'pro-package',
    name: 'Pro Package',
    price: '¥15,000',
    priceId: 'price_once_pro',
    description: 'For power users',
    features: ['Everything in Basic', 'Feature C', 'Feature D'],
  },
];

export function ProductsPage() {
  const [loading, setLoading] = useState<string | null>(null);

  const handlePurchase = async (priceId: string) => {
    setLoading(priceId);
    try {
      const res = await fetch('/api/stripe/checkout', {
        method: 'POST',
        headers: { 'Content-Type': 'application/json' },
        body: JSON.stringify({ priceId, mode: 'payment' }),
      });
      const { url } = await res.json();
      window.location.href = url;
    } catch (error) {
      console.error(error);
    } finally {
      setLoading(null);
    }
  };

  return (
    <div className="grid md:grid-cols-2 gap-6">
      {products.map((product) => (
        <div key={product.id} className="p-6 rounded-lg border">
          <h3 className="text-xl font-bold">{product.name}</h3>
          <p className="text-gray-600 mt-1">{product.description}</p>
          <p className="text-3xl font-bold mt-4">{product.price}</p>
          <ul className="mt-4 space-y-2">
            {product.features.map((feature) => (
              <li key={feature}>✓ {feature}</li>
            ))}
          </ul>
          <button
            onClick={() => handlePurchase(product.priceId)}
            disabled={loading === product.priceId}
            className="w-full mt-6 py-2 bg-green-600 text-white rounded hover:bg-green-700 disabled:opacity-50"
          >
            {loading === product.priceId ? 'Loading...' : 'Purchase'}
          </button>
        </div>
      ))}
    </div>
  );
}
```

---

## Webhook Handler (Common)

### Step W1: Webhook API Route

```typescript
// app/api/stripe/webhook/route.ts
import { NextRequest, NextResponse } from 'next/server';
import { stripe } from '@/lib/stripe';
import { db } from '@/lib/db';
import Stripe from 'stripe';

export async function POST(req: NextRequest) {
  const body = await req.text();
  const signature = req.headers.get('stripe-signature')!;

  let event: Stripe.Event;

  try {
    event = stripe.webhooks.constructEvent(
      body,
      signature,
      process.env.STRIPE_WEBHOOK_SECRET!
    );
  } catch (error) {
    console.error('Webhook signature verification failed');
    return NextResponse.json({ error: 'Invalid signature' }, { status: 400 });
  }

  try {
    switch (event.type) {
      // Subscription events
      case 'checkout.session.completed': {
        const session = event.data.object as Stripe.Checkout.Session;
        await handleCheckoutCompleted(session);
        break;
      }

      case 'customer.subscription.updated': {
        const subscription = event.data.object as Stripe.Subscription;
        await handleSubscriptionUpdated(subscription);
        break;
      }

      case 'customer.subscription.deleted': {
        const subscription = event.data.object as Stripe.Subscription;
        await handleSubscriptionDeleted(subscription);
        break;
      }

      case 'invoice.payment_succeeded': {
        const invoice = event.data.object as Stripe.Invoice;
        await handleInvoicePaymentSucceeded(invoice);
        break;
      }

      case 'invoice.payment_failed': {
        const invoice = event.data.object as Stripe.Invoice;
        await handleInvoicePaymentFailed(invoice);
        break;
      }

      default:
        console.log(`Unhandled event type: ${event.type}`);
    }

    return NextResponse.json({ received: true });
  } catch (error) {
    console.error('Webhook handler error:', error);
    return NextResponse.json({ error: 'Webhook failed' }, { status: 500 });
  }
}

async function handleCheckoutCompleted(session: Stripe.Checkout.Session) {
  const userId = session.metadata?.userId;
  if (!userId) return;

  if (session.mode === 'subscription') {
    // Subscription purchase - get subscription details for price ID
    const subscription = await stripe.subscriptions.retrieve(
      session.subscription as string
    );
    const priceId = subscription.items.data[0]?.price.id;

    await db.user.update({
      where: { id: userId },
      data: {
        stripeCustomerId: session.customer as string,
        stripeSubscriptionId: session.subscription as string,
        stripePriceId: priceId,
        subscriptionStatus: 'active',
        currentPeriodEnd: new Date(subscription.current_period_end * 1000),
      },
    });
  } else {
    // One-time purchase
    await db.purchase.create({
      data: {
        userId,
        stripePaymentId: session.payment_intent as string,
        amount: session.amount_total!,
        status: 'completed',
      },
    });
  }
}

async function handleSubscriptionUpdated(subscription: Stripe.Subscription) {
  const priceId = subscription.items.data[0]?.price.id;

  await db.user.updateMany({
    where: { stripeSubscriptionId: subscription.id },
    data: {
      subscriptionStatus: subscription.status,
      stripePriceId: priceId, // Track current plan
      currentPeriodEnd: new Date(subscription.current_period_end * 1000),
    },
  });

  // Log plan changes for debugging
  console.log(`Subscription ${subscription.id} updated:`, {
    status: subscription.status,
    priceId,
    cancelAtPeriodEnd: subscription.cancel_at_period_end,
  });
}

async function handleSubscriptionDeleted(subscription: Stripe.Subscription) {
  await db.user.updateMany({
    where: { stripeSubscriptionId: subscription.id },
    data: {
      subscriptionStatus: 'canceled',
      stripeSubscriptionId: null,
      stripePriceId: null,
    },
  });
}

async function handleInvoicePaymentSucceeded(invoice: Stripe.Invoice) {
  // Monthly renewal - update period end date
  if (invoice.subscription) {
    const subscription = await stripe.subscriptions.retrieve(
      invoice.subscription as string
    );

    await db.user.updateMany({
      where: { stripeSubscriptionId: subscription.id },
      data: {
        subscriptionStatus: 'active',
        currentPeriodEnd: new Date(subscription.current_period_end * 1000),
      },
    });

    // Optional: Send renewal confirmation email
    // await sendRenewalEmail(invoice.customer_email, invoice.amount_paid);
  }

  console.log('Payment succeeded:', {
    invoiceId: invoice.id,
    amount: invoice.amount_paid,
    customerEmail: invoice.customer_email,
  });
}

async function handleInvoicePaymentFailed(invoice: Stripe.Invoice) {
  // Update status to past_due
  if (invoice.subscription) {
    await db.user.updateMany({
      where: { stripeSubscriptionId: invoice.subscription as string },
      data: {
        subscriptionStatus: 'past_due',
      },
    });

    // Optional: Send payment failed notification
    // await sendPaymentFailedEmail(invoice.customer_email);
  }

  console.log('Payment failed:', {
    invoiceId: invoice.id,
    customerEmail: invoice.customer_email,
    attemptCount: invoice.attempt_count,
  });
}
```

### Step W2: Configure Webhook in Stripe Dashboard

1. Go to Stripe Dashboard → Developers → Webhooks
2. Add endpoint: `https://your-domain.com/api/stripe/webhook`
3. Select events:
   - `checkout.session.completed`
   - `customer.subscription.updated`
   - `customer.subscription.deleted`
   - `invoice.payment_succeeded`
   - `invoice.payment_failed`
4. Copy webhook signing secret to `.env.local`

---

## Database Schema

### Required Fields (Prisma example)

```prisma
model User {
  id                   String    @id @default(cuid())
  email                String    @unique
  stripeCustomerId     String?   @unique
  stripeSubscriptionId String?   @unique
  stripePriceId        String?   // Current plan price ID (e.g., 'price_pro_xxx')
  subscriptionStatus   String?   // 'active', 'canceled', 'past_due', 'trialing'
  currentPeriodEnd     DateTime?
  purchases            Purchase[]
}

model Purchase {
  id              String   @id @default(cuid())
  userId          String
  user            User     @relation(fields: [userId], references: [id])
  stripePaymentId String   @unique
  amount          Int
  status          String   // 'completed', 'refunded'
  createdAt       DateTime @default(now())
}
```

---

## Utility Functions

### Check Subscription Status

```typescript
// lib/subscription.ts
import { db } from '@/lib/db';

export async function hasActiveSubscription(userId: string): Promise<boolean> {
  const user = await db.user.findUnique({
    where: { id: userId },
    select: { subscriptionStatus: true, currentPeriodEnd: true },
  });

  if (!user) return false;

  return (
    user.subscriptionStatus === 'active' &&
    user.currentPeriodEnd &&
    user.currentPeriodEnd > new Date()
  );
}

export async function getSubscriptionTier(userId: string): Promise<string | null> {
  const user = await db.user.findUnique({
    where: { id: userId },
    select: { stripePriceId: true, subscriptionStatus: true },
  });

  if (!user?.stripePriceId || user.subscriptionStatus !== 'active') {
    return null;
  }

  // Map price ID to tier name
  const tierMap: Record<string, string> = {
    'price_basic_xxx': 'basic',
    'price_pro_xxx': 'pro',
    'price_enterprise_xxx': 'enterprise',
  };

  return tierMap[user.stripePriceId] || null;
}

// Get full subscription info from DB (no Stripe API call)
export async function getSubscriptionInfo(userId: string) {
  const user = await db.user.findUnique({
    where: { id: userId },
    select: {
      stripePriceId: true,
      subscriptionStatus: true,
      currentPeriodEnd: true,
    },
  });

  if (!user) return null;

  const tierMap: Record<string, string> = {
    'price_basic_xxx': 'basic',
    'price_pro_xxx': 'pro',
    'price_enterprise_xxx': 'enterprise',
  };

  return {
    tier: user.stripePriceId ? tierMap[user.stripePriceId] : null,
    priceId: user.stripePriceId,
    status: user.subscriptionStatus,
    currentPeriodEnd: user.currentPeriodEnd,
    isActive: user.subscriptionStatus === 'active' &&
              user.currentPeriodEnd &&
              user.currentPeriodEnd > new Date(),
  };
}
```

### Check Purchase Status

```typescript
// lib/purchase.ts
import { db } from '@/lib/db';

export async function hasPurchased(
  userId: string,
  productId?: string
): Promise<boolean> {
  const purchases = await db.purchase.findMany({
    where: {
      userId,
      status: 'completed',
    },
  });

  return purchases.length > 0;
}
```

---

## Testing

### Test Cards

| Card Number | Scenario |
|-------------|----------|
| 4242424242424242 | Success |
| 4000000000000341 | Attach fails |
| 4000000000009995 | Insufficient funds |

### Test Webhooks Locally

```bash
# Install Stripe CLI
stripe listen --forward-to localhost:3000/api/stripe/webhook

# Trigger test events
stripe trigger checkout.session.completed
stripe trigger customer.subscription.updated
```

---

## Checklist

### Setup
- [ ] Stripe account created
- [ ] API keys configured
- [ ] Products/prices created in Dashboard

### Implementation
- [ ] Checkout API route
- [ ] Webhook handler
- [ ] Pricing/product page
- [ ] Customer portal (for subscriptions)

### Database
- [ ] User model with Stripe fields
- [ ] Purchase model (for one-time)
- [ ] Migration applied

### Testing
- [ ] Test mode purchases
- [ ] Webhook events verified
- [ ] Customer portal tested

### Production
- [ ] Live API keys
- [ ] Webhook endpoint registered
- [ ] HTTPS enabled
