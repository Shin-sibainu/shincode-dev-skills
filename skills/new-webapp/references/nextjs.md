# Next.js Stack Patterns

## Table of Contents
1. [Base Setup](#base-setup)
2. [With Supabase](#with-supabase)
3. [With Turso](#with-turso)
4. [Authentication Options](#authentication-options)
5. [Payment Integration](#payment-integration)

## Base Setup

### Dependencies
```json
{
  "dependencies": {
    "next": "^15.0.0",
    "react": "^19.0.0",
    "react-dom": "^19.0.0"
  },
  "devDependencies": {
    "@types/node": "^22.0.0",
    "@types/react": "^19.0.0",
    "typescript": "^5.7.0",
    "tailwindcss": "^4.0.0"
  }
}
```

### Directory Structure
```
app/
├── layout.tsx
├── page.tsx
├── globals.css
└── api/
components/
├── ui/
lib/
├── utils.ts
types/
├── index.ts
docs/
├── requirements.md
├── architecture.md
├── database.md
├── api.md
└── tickets/
```

### Essential Files

**app/layout.tsx**
```tsx
import type { Metadata } from "next";
import "./globals.css";

export const metadata: Metadata = {
  title: "App Name",
  description: "App description",
};

export default function RootLayout({
  children,
}: {
  children: React.ReactNode;
}) {
  return (
    <html lang="en">
      <body>{children}</body>
    </html>
  );
}
```

**tailwind.config.ts**
```ts
import type { Config } from "tailwindcss";

const config: Config = {
  content: [
    "./app/**/*.{js,ts,jsx,tsx,mdx}",
    "./components/**/*.{js,ts,jsx,tsx,mdx}",
  ],
  theme: {
    extend: {},
  },
  plugins: [],
};
export default config;
```

## With Supabase

### Additional Dependencies
```json
{
  "dependencies": {
    "@supabase/supabase-js": "^2.47.0",
    "@supabase/ssr": "^0.5.0"
  }
}
```

### Supabase Client Setup

**lib/supabase/client.ts** (Client Component)
```ts
import { createBrowserClient } from "@supabase/ssr";

export function createClient() {
  return createBrowserClient(
    process.env.NEXT_PUBLIC_SUPABASE_URL!,
    process.env.NEXT_PUBLIC_SUPABASE_ANON_KEY!
  );
}
```

**lib/supabase/server.ts** (Server Component)
```ts
import { createServerClient } from "@supabase/ssr";
import { cookies } from "next/headers";

export async function createClient() {
  const cookieStore = await cookies();

  return createServerClient(
    process.env.NEXT_PUBLIC_SUPABASE_URL!,
    process.env.NEXT_PUBLIC_SUPABASE_ANON_KEY!,
    {
      cookies: {
        getAll() {
          return cookieStore.getAll();
        },
        setAll(cookiesToSet) {
          cookiesToSet.forEach(({ name, value, options }) =>
            cookieStore.set(name, value, options)
          );
        },
      },
    }
  );
}
```

### Environment Variables
```env
NEXT_PUBLIC_SUPABASE_URL=your-project-url
NEXT_PUBLIC_SUPABASE_ANON_KEY=your-anon-key
```

## With Turso

### Additional Dependencies
```json
{
  "dependencies": {
    "@libsql/client": "^0.14.0",
    "drizzle-orm": "^0.38.0"
  },
  "devDependencies": {
    "drizzle-kit": "^0.30.0"
  }
}
```

### Turso Client Setup

**lib/db/index.ts**
```ts
import { drizzle } from "drizzle-orm/libsql";
import { createClient } from "@libsql/client";
import * as schema from "./schema";

const client = createClient({
  url: process.env.TURSO_DATABASE_URL!,
  authToken: process.env.TURSO_AUTH_TOKEN,
});

export const db = drizzle(client, { schema });
```

**lib/db/schema.ts**
```ts
import { sqliteTable, text, integer } from "drizzle-orm/sqlite-core";

export const users = sqliteTable("users", {
  id: integer("id").primaryKey({ autoIncrement: true }),
  email: text("email").notNull().unique(),
  name: text("name"),
  createdAt: integer("created_at", { mode: "timestamp" }).$defaultFn(() => new Date()),
});
```

**drizzle.config.ts**
```ts
import type { Config } from "drizzle-kit";

export default {
  schema: "./lib/db/schema.ts",
  out: "./drizzle",
  dialect: "turso",
  dbCredentials: {
    url: process.env.TURSO_DATABASE_URL!,
    authToken: process.env.TURSO_AUTH_TOKEN,
  },
} satisfies Config;
```

### Environment Variables
```env
TURSO_DATABASE_URL=libsql://your-db-name-your-org.turso.io
TURSO_AUTH_TOKEN=your-auth-token
```

### Turso CLI Commands
```bash
# Install Turso CLI
curl -sSfL https://get.tur.so/install.sh | bash

# Login
turso auth login

# Create database
turso db create my-app-db

# Get database URL
turso db show my-app-db --url

# Create auth token
turso db tokens create my-app-db
```

## Authentication Options

### Supabase Auth
Already included with Supabase setup. See [With Supabase](#with-supabase) section.

**Usage in Server Components**
```ts
import { createClient } from "@/lib/supabase/server";

export default async function Page() {
  const supabase = await createClient();
  const { data: { user } } = await supabase.auth.getUser();

  if (!user) {
    redirect("/login");
  }

  return <div>Hello {user.email}</div>;
}
```

**Login Page**
```tsx
"use client";
import { createClient } from "@/lib/supabase/client";

export default function Login() {
  const supabase = createClient();

  const handleLogin = async () => {
    await supabase.auth.signInWithOAuth({
      provider: "github",
      options: { redirectTo: `${location.origin}/auth/callback` },
    });
  };

  return <button onClick={handleLogin}>Sign in with GitHub</button>;
}
```

### better-auth
```json
{
  "dependencies": {
    "better-auth": "^1.0.0"
  }
}
```

**lib/auth.ts**
```ts
import { betterAuth } from "better-auth";
import { drizzleAdapter } from "better-auth/adapters/drizzle";
import { db } from "./db";

export const auth = betterAuth({
  database: drizzleAdapter(db, { provider: "sqlite" }),
  emailAndPassword: { enabled: true },
  socialProviders: {
    github: {
      clientId: process.env.GITHUB_CLIENT_ID!,
      clientSecret: process.env.GITHUB_CLIENT_SECRET!,
    },
    google: {
      clientId: process.env.GOOGLE_CLIENT_ID!,
      clientSecret: process.env.GOOGLE_CLIENT_SECRET!,
    },
  },
});
```

**lib/auth-client.ts**
```ts
import { createAuthClient } from "better-auth/react";

export const authClient = createAuthClient();
export const { signIn, signOut, useSession } = authClient;
```

**app/api/auth/[...all]/route.ts**
```ts
import { auth } from "@/lib/auth";
import { toNextJsHandler } from "better-auth/next-js";

export const { GET, POST } = toNextJsHandler(auth.handler);
```

**Usage in Components**
```tsx
"use client";
import { useSession, signIn, signOut } from "@/lib/auth-client";

export function AuthButton() {
  const { data: session } = useSession();

  if (session) {
    return <button onClick={() => signOut()}>Sign out</button>;
  }
  return <button onClick={() => signIn.social({ provider: "github" })}>Sign in</button>;
}
```

### Clerk
```json
{
  "dependencies": {
    "@clerk/nextjs": "^6.0.0"
  }
}
```

**middleware.ts**
```ts
import { clerkMiddleware } from "@clerk/nextjs/server";

export default clerkMiddleware();

export const config = {
  matcher: ["/((?!.*\\..*|_next).*)", "/", "/(api|trpc)(.*)"],
};
```

## Payment Integration (Stripe Subscription)

### Dependencies
```json
{
  "dependencies": {
    "stripe": "^17.0.0",
    "@stripe/stripe-js": "^4.0.0"
  }
}
```

### Environment Variables
```env
STRIPE_SECRET_KEY=sk_test_...
STRIPE_WEBHOOK_SECRET=whsec_...
NEXT_PUBLIC_STRIPE_PUBLISHABLE_KEY=pk_test_...
NEXT_PUBLIC_APP_URL=http://localhost:3000

# Price IDs from Stripe Dashboard
STRIPE_PRICE_BASIC=price_...
STRIPE_PRICE_PRO=price_...
STRIPE_PRICE_ENTERPRISE=price_...
```

### Stripe Client

**lib/stripe.ts**
```ts
import Stripe from "stripe";

export const stripe = new Stripe(process.env.STRIPE_SECRET_KEY!, {
  apiVersion: "2024-12-18.acacia",
});

export const PLANS = {
  basic: {
    name: "Basic",
    price: 980,
    priceId: process.env.STRIPE_PRICE_BASIC!,
    features: ["Feature 1", "Feature 2", "Email support"],
  },
  pro: {
    name: "Pro",
    price: 2980,
    priceId: process.env.STRIPE_PRICE_PRO!,
    features: ["All Basic features", "Feature 3", "Feature 4", "Priority support"],
  },
  enterprise: {
    name: "Enterprise",
    price: 9800,
    priceId: process.env.STRIPE_PRICE_ENTERPRISE!,
    features: ["All Pro features", "Feature 5", "Custom integrations", "Dedicated support"],
  },
} as const;

export type PlanKey = keyof typeof PLANS;
```

### Checkout API

**app/api/stripe/checkout/route.ts**
```ts
import { NextResponse } from "next/server";
import { stripe, PLANS, PlanKey } from "@/lib/stripe";
import { auth } from "@/lib/auth"; // or your auth solution

export async function POST(request: Request) {
  const session = await auth();
  if (!session?.user?.email) {
    return NextResponse.json({ error: "Unauthorized" }, { status: 401 });
  }

  const { plan } = (await request.json()) as { plan: PlanKey };
  const planData = PLANS[plan];

  if (!planData) {
    return NextResponse.json({ error: "Invalid plan" }, { status: 400 });
  }

  // Get or create Stripe customer
  const customers = await stripe.customers.list({ email: session.user.email, limit: 1 });
  let customerId = customers.data[0]?.id;

  if (!customerId) {
    const customer = await stripe.customers.create({
      email: session.user.email,
      metadata: { userId: session.user.id },
    });
    customerId = customer.id;
  }

  const checkoutSession = await stripe.checkout.sessions.create({
    customer: customerId,
    mode: "subscription",
    payment_method_types: ["card"],
    line_items: [{ price: planData.priceId, quantity: 1 }],
    success_url: `${process.env.NEXT_PUBLIC_APP_URL}/dashboard?success=true`,
    cancel_url: `${process.env.NEXT_PUBLIC_APP_URL}/pricing?canceled=true`,
    subscription_data: {
      metadata: { userId: session.user.id, plan },
    },
  });

  return NextResponse.json({ url: checkoutSession.url });
}
```

### Customer Portal API

**app/api/stripe/portal/route.ts**
```ts
import { NextResponse } from "next/server";
import { stripe } from "@/lib/stripe";
import { auth } from "@/lib/auth";

export async function POST() {
  const session = await auth();
  if (!session?.user?.email) {
    return NextResponse.json({ error: "Unauthorized" }, { status: 401 });
  }

  const customers = await stripe.customers.list({ email: session.user.email, limit: 1 });
  const customerId = customers.data[0]?.id;

  if (!customerId) {
    return NextResponse.json({ error: "No subscription found" }, { status: 404 });
  }

  const portalSession = await stripe.billingPortal.sessions.create({
    customer: customerId,
    return_url: `${process.env.NEXT_PUBLIC_APP_URL}/dashboard`,
  });

  return NextResponse.json({ url: portalSession.url });
}
```

### Webhook Handler

**app/api/stripe/webhook/route.ts**
```ts
import { NextResponse } from "next/server";
import { headers } from "next/headers";
import { stripe } from "@/lib/stripe";
import Stripe from "stripe";

export async function POST(request: Request) {
  const body = await request.text();
  const headersList = await headers();
  const signature = headersList.get("stripe-signature")!;

  let event: Stripe.Event;

  try {
    event = stripe.webhooks.constructEvent(
      body,
      signature,
      process.env.STRIPE_WEBHOOK_SECRET!
    );
  } catch (err) {
    console.error("Webhook signature verification failed");
    return NextResponse.json({ error: "Invalid signature" }, { status: 400 });
  }

  switch (event.type) {
    case "checkout.session.completed": {
      const session = event.data.object as Stripe.Checkout.Session;
      // Update user subscription status in database
      console.log("Checkout completed:", session.subscription);
      break;
    }

    case "customer.subscription.updated": {
      const subscription = event.data.object as Stripe.Subscription;
      // Handle plan changes, renewals
      console.log("Subscription updated:", subscription.id, subscription.status);
      break;
    }

    case "customer.subscription.deleted": {
      const subscription = event.data.object as Stripe.Subscription;
      // Handle cancellation
      console.log("Subscription canceled:", subscription.id);
      break;
    }

    case "invoice.payment_succeeded": {
      const invoice = event.data.object as Stripe.Invoice;
      // Monthly renewal successful
      console.log("Payment succeeded:", invoice.id);
      break;
    }

    case "invoice.payment_failed": {
      const invoice = event.data.object as Stripe.Invoice;
      // Payment failed - notify user
      console.log("Payment failed:", invoice.id);
      break;
    }
  }

  return NextResponse.json({ received: true });
}
```

### Subscription Status Check

**lib/subscription.ts**
```ts
import { stripe } from "./stripe";

export async function getSubscription(email: string) {
  const customers = await stripe.customers.list({ email, limit: 1 });
  const customerId = customers.data[0]?.id;

  if (!customerId) return null;

  const subscriptions = await stripe.subscriptions.list({
    customer: customerId,
    status: "active",
    limit: 1,
  });

  return subscriptions.data[0] || null;
}

export async function getSubscriptionStatus(email: string) {
  const subscription = await getSubscription(email);

  if (!subscription) {
    return { isActive: false, plan: null, currentPeriodEnd: null };
  }

  return {
    isActive: subscription.status === "active",
    plan: subscription.metadata.plan,
    currentPeriodEnd: new Date(subscription.current_period_end * 1000),
    cancelAtPeriodEnd: subscription.cancel_at_period_end,
  };
}
```

### Pricing Page Component

**components/PricingCard.tsx**
```tsx
"use client";

import { useState } from "react";
import { PLANS, PlanKey } from "@/lib/stripe";

export function PricingCard({ plan, recommended }: { plan: PlanKey; recommended?: boolean }) {
  const [loading, setLoading] = useState(false);
  const data = PLANS[plan];

  const handleSubscribe = async () => {
    setLoading(true);
    const res = await fetch("/api/stripe/checkout", {
      method: "POST",
      headers: { "Content-Type": "application/json" },
      body: JSON.stringify({ plan }),
    });
    const { url } = await res.json();
    window.location.href = url;
  };

  return (
    <div className={`p-6 rounded-lg border ${recommended ? "border-blue-500 ring-2 ring-blue-500" : ""}`}>
      {recommended && <span className="text-sm text-blue-500 font-medium">Recommended</span>}
      <h3 className="text-xl font-bold mt-2">{data.name}</h3>
      <p className="text-3xl font-bold mt-4">
        ¥{data.price.toLocaleString()}<span className="text-sm font-normal">/month</span>
      </p>
      <ul className="mt-6 space-y-2">
        {data.features.map((feature) => (
          <li key={feature} className="flex items-center gap-2">
            <span>✓</span> {feature}
          </li>
        ))}
      </ul>
      <button
        onClick={handleSubscribe}
        disabled={loading}
        className="mt-6 w-full py-2 px-4 bg-blue-500 text-white rounded-lg disabled:opacity-50"
      >
        {loading ? "Loading..." : "Subscribe"}
      </button>
    </div>
  );
}
```

**app/pricing/page.tsx**
```tsx
import { PricingCard } from "@/components/PricingCard";

export default function PricingPage() {
  return (
    <div className="max-w-5xl mx-auto py-12 px-4">
      <h1 className="text-3xl font-bold text-center">Pricing Plans</h1>
      <p className="text-center text-gray-600 mt-2">Choose the plan that fits your needs</p>

      <div className="grid md:grid-cols-3 gap-6 mt-12">
        <PricingCard plan="basic" />
        <PricingCard plan="pro" recommended />
        <PricingCard plan="enterprise" />
      </div>
    </div>
  );
}
```

### Customer Portal Button

**components/ManageSubscription.tsx**
```tsx
"use client";

import { useState } from "react";

export function ManageSubscription() {
  const [loading, setLoading] = useState(false);

  const handleManage = async () => {
    setLoading(true);
    const res = await fetch("/api/stripe/portal", { method: "POST" });
    const { url } = await res.json();
    window.location.href = url;
  };

  return (
    <button onClick={handleManage} disabled={loading} className="text-blue-500 underline">
      {loading ? "Loading..." : "Manage Subscription"}
    </button>
  );
}
```

### Stripe CLI for Local Webhook Testing
```bash
# Install Stripe CLI
# https://stripe.com/docs/stripe-cli

# Login
stripe login

# Forward webhooks to local server
stripe listen --forward-to localhost:3000/api/stripe/webhook

# Copy the webhook signing secret to .env.local
```

## File Storage

### Supabase Storage

Already included with Supabase. Configure bucket in Supabase Dashboard.

**lib/storage.ts**
```ts
import { createClient } from "@/lib/supabase/client";

export async function uploadFile(file: File, bucket: string, path: string) {
  const supabase = createClient();
  const { data, error } = await supabase.storage
    .from(bucket)
    .upload(path, file, { upsert: true });

  if (error) throw error;
  return data;
}

export function getPublicUrl(bucket: string, path: string) {
  const supabase = createClient();
  const { data } = supabase.storage.from(bucket).getPublicUrl(path);
  return data.publicUrl;
}
```

**components/FileUpload.tsx**
```tsx
"use client";

import { useState, useCallback } from "react";
import { uploadFile, getPublicUrl } from "@/lib/storage";

export function FileUpload({ bucket, onUpload }: { bucket: string; onUpload: (url: string) => void }) {
  const [uploading, setUploading] = useState(false);
  const [dragOver, setDragOver] = useState(false);

  const handleUpload = useCallback(async (file: File) => {
    setUploading(true);
    const path = `${Date.now()}-${file.name}`;
    await uploadFile(file, bucket, path);
    const url = getPublicUrl(bucket, path);
    onUpload(url);
    setUploading(false);
  }, [bucket, onUpload]);

  return (
    <div
      onDragOver={(e) => { e.preventDefault(); setDragOver(true); }}
      onDragLeave={() => setDragOver(false)}
      onDrop={(e) => {
        e.preventDefault();
        setDragOver(false);
        const file = e.dataTransfer.files[0];
        if (file) handleUpload(file);
      }}
      className={`border-2 border-dashed p-8 text-center rounded-lg ${dragOver ? "border-blue-500 bg-blue-50" : ""}`}
    >
      {uploading ? "Uploading..." : "Drag & drop or click to upload"}
      <input
        type="file"
        className="hidden"
        onChange={(e) => e.target.files?.[0] && handleUpload(e.target.files[0])}
      />
    </div>
  );
}
```

### Cloudflare R2

```json
{
  "dependencies": {
    "@aws-sdk/client-s3": "^3.700.0",
    "@aws-sdk/s3-request-presigner": "^3.700.0"
  }
}
```

**Environment Variables**
```env
R2_ACCOUNT_ID=your-account-id
R2_ACCESS_KEY_ID=your-access-key
R2_SECRET_ACCESS_KEY=your-secret-key
R2_BUCKET_NAME=your-bucket
R2_PUBLIC_URL=https://your-bucket.your-account.r2.cloudflarestorage.com
```

**lib/r2.ts**
```ts
import { S3Client, PutObjectCommand, GetObjectCommand } from "@aws-sdk/client-s3";
import { getSignedUrl } from "@aws-sdk/s3-request-presigner";

const client = new S3Client({
  region: "auto",
  endpoint: `https://${process.env.R2_ACCOUNT_ID}.r2.cloudflarestorage.com`,
  credentials: {
    accessKeyId: process.env.R2_ACCESS_KEY_ID!,
    secretAccessKey: process.env.R2_SECRET_ACCESS_KEY!,
  },
});

export async function getUploadUrl(key: string, contentType: string) {
  const command = new PutObjectCommand({
    Bucket: process.env.R2_BUCKET_NAME,
    Key: key,
    ContentType: contentType,
  });
  return getSignedUrl(client, command, { expiresIn: 3600 });
}

export function getPublicUrl(key: string) {
  return `${process.env.R2_PUBLIC_URL}/${key}`;
}
```

**app/api/upload/route.ts**
```ts
import { NextResponse } from "next/server";
import { getUploadUrl, getPublicUrl } from "@/lib/r2";

export async function POST(request: Request) {
  const { filename, contentType } = await request.json();
  const key = `uploads/${Date.now()}-${filename}`;

  const uploadUrl = await getUploadUrl(key, contentType);
  const publicUrl = getPublicUrl(key);

  return NextResponse.json({ uploadUrl, publicUrl, key });
}
```

**Usage (Client)**
```ts
async function uploadToR2(file: File) {
  // Get presigned URL
  const res = await fetch("/api/upload", {
    method: "POST",
    body: JSON.stringify({ filename: file.name, contentType: file.type }),
  });
  const { uploadUrl, publicUrl } = await res.json();

  // Upload directly to R2
  await fetch(uploadUrl, {
    method: "PUT",
    body: file,
    headers: { "Content-Type": file.type },
  });

  return publicUrl;
}
```

## Email (Resend)

### Dependencies
```json
{
  "dependencies": {
    "resend": "^4.0.0",
    "@react-email/components": "^0.0.30"
  }
}
```

### Environment Variables
```env
RESEND_API_KEY=re_...
EMAIL_FROM=noreply@yourdomain.com
```

### Email Client

**lib/email.ts**
```ts
import { Resend } from "resend";

export const resend = new Resend(process.env.RESEND_API_KEY);
```

### Email Templates

**emails/welcome.tsx**
```tsx
import { Html, Head, Body, Container, Text, Button, Hr } from "@react-email/components";

interface WelcomeEmailProps {
  name: string;
  loginUrl: string;
}

export function WelcomeEmail({ name, loginUrl }: WelcomeEmailProps) {
  return (
    <Html>
      <Head />
      <Body style={{ fontFamily: "sans-serif", backgroundColor: "#f6f9fc" }}>
        <Container style={{ padding: "40px", backgroundColor: "#fff", borderRadius: "8px" }}>
          <Text style={{ fontSize: "24px", fontWeight: "bold" }}>Welcome, {name}!</Text>
          <Text>Thank you for signing up. We're excited to have you on board.</Text>
          <Hr />
          <Button
            href={loginUrl}
            style={{ backgroundColor: "#3b82f6", color: "#fff", padding: "12px 24px", borderRadius: "6px" }}
          >
            Get Started
          </Button>
        </Container>
      </Body>
    </Html>
  );
}
```

**emails/password-reset.tsx**
```tsx
import { Html, Head, Body, Container, Text, Button } from "@react-email/components";

interface PasswordResetEmailProps {
  resetUrl: string;
}

export function PasswordResetEmail({ resetUrl }: PasswordResetEmailProps) {
  return (
    <Html>
      <Head />
      <Body style={{ fontFamily: "sans-serif", backgroundColor: "#f6f9fc" }}>
        <Container style={{ padding: "40px", backgroundColor: "#fff", borderRadius: "8px" }}>
          <Text style={{ fontSize: "24px", fontWeight: "bold" }}>Reset Your Password</Text>
          <Text>Click the button below to reset your password. This link expires in 1 hour.</Text>
          <Button
            href={resetUrl}
            style={{ backgroundColor: "#3b82f6", color: "#fff", padding: "12px 24px", borderRadius: "6px" }}
          >
            Reset Password
          </Button>
          <Text style={{ color: "#666", fontSize: "12px" }}>
            If you didn't request this, you can safely ignore this email.
          </Text>
        </Container>
      </Body>
    </Html>
  );
}
```

### Send Email API

**app/api/email/send/route.ts**
```ts
import { NextResponse } from "next/server";
import { resend } from "@/lib/email";
import { WelcomeEmail } from "@/emails/welcome";

export async function POST(request: Request) {
  const { to, name } = await request.json();

  const { data, error } = await resend.emails.send({
    from: process.env.EMAIL_FROM!,
    to,
    subject: "Welcome to Our App!",
    react: WelcomeEmail({ name, loginUrl: `${process.env.NEXT_PUBLIC_APP_URL}/login` }),
  });

  if (error) {
    return NextResponse.json({ error }, { status: 500 });
  }

  return NextResponse.json({ id: data?.id });
}
```

### Usage Example

```ts
// Send welcome email after user signs up
await fetch("/api/email/send", {
  method: "POST",
  headers: { "Content-Type": "application/json" },
  body: JSON.stringify({ to: user.email, name: user.name }),
});
```
