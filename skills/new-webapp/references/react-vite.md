# React + Vite Stack Patterns

## Table of Contents
1. [Base Setup](#base-setup)
2. [With Supabase](#with-supabase)
3. [With Turso](#with-turso)
4. [Routing](#routing)
5. [Authentication](#authentication)
6. [Payment Integration](#payment-integration)

## Base Setup

### Dependencies
```json
{
  "dependencies": {
    "react": "^19.0.0",
    "react-dom": "^19.0.0",
    "react-router-dom": "^7.0.0"
  },
  "devDependencies": {
    "@types/react": "^19.0.0",
    "@types/react-dom": "^19.0.0",
    "@vitejs/plugin-react": "^4.3.0",
    "typescript": "^5.7.0",
    "vite": "^6.0.0",
    "tailwindcss": "^4.0.0",
    "autoprefixer": "^10.4.0",
    "postcss": "^8.4.0"
  }
}
```

### Directory Structure
```
src/
├── main.tsx
├── App.tsx
├── index.css
├── components/
│   └── ui/
├── pages/
├── lib/
│   └── utils.ts
└── types/
    └── index.ts
docs/
├── requirements.md
├── architecture.md
├── database.md
├── api.md
└── tickets/
index.html
vite.config.ts
```

### Essential Files

**vite.config.ts**
```ts
import { defineConfig } from "vite";
import react from "@vitejs/plugin-react";
import path from "path";

export default defineConfig({
  plugins: [react()],
  resolve: {
    alias: {
      "@": path.resolve(__dirname, "./src"),
    },
  },
});
```

**src/main.tsx**
```tsx
import React from "react";
import ReactDOM from "react-dom/client";
import { BrowserRouter } from "react-router-dom";
import App from "./App";
import "./index.css";

ReactDOM.createRoot(document.getElementById("root")!).render(
  <React.StrictMode>
    <BrowserRouter>
      <App />
    </BrowserRouter>
  </React.StrictMode>
);
```

**src/App.tsx**
```tsx
import { Routes, Route } from "react-router-dom";

function App() {
  return (
    <Routes>
      <Route path="/" element={<div>Home</div>} />
    </Routes>
  );
}

export default App;
```

## With Supabase

### Additional Dependencies
```json
{
  "dependencies": {
    "@supabase/supabase-js": "^2.47.0"
  }
}
```

### Supabase Client Setup

**src/lib/supabase.ts**
```ts
import { createClient } from "@supabase/supabase-js";

const supabaseUrl = import.meta.env.VITE_SUPABASE_URL;
const supabaseAnonKey = import.meta.env.VITE_SUPABASE_ANON_KEY;

export const supabase = createClient(supabaseUrl, supabaseAnonKey);
```

### Environment Variables
```env
VITE_SUPABASE_URL=your-project-url
VITE_SUPABASE_ANON_KEY=your-anon-key
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

**src/lib/db/index.ts**
```ts
import { drizzle } from "drizzle-orm/libsql";
import { createClient } from "@libsql/client";
import * as schema from "./schema";

const client = createClient({
  url: import.meta.env.VITE_TURSO_DATABASE_URL,
  authToken: import.meta.env.VITE_TURSO_AUTH_TOKEN,
});

export const db = drizzle(client, { schema });
```

**src/lib/db/schema.ts**
```ts
import { sqliteTable, text, integer } from "drizzle-orm/sqlite-core";

export const users = sqliteTable("users", {
  id: integer("id").primaryKey({ autoIncrement: true }),
  email: text("email").notNull().unique(),
  name: text("name"),
  createdAt: integer("created_at", { mode: "timestamp" }).$defaultFn(() => new Date()),
});
```

### Environment Variables
```env
VITE_TURSO_DATABASE_URL=libsql://your-db-name-your-org.turso.io
VITE_TURSO_AUTH_TOKEN=your-auth-token
```

Note: For client-side apps, consider using Turso with a backend API to protect auth tokens.

## Routing

### React Router Setup

**src/pages/Home.tsx**
```tsx
export default function Home() {
  return <div>Home Page</div>;
}
```

**src/App.tsx with routes**
```tsx
import { Routes, Route } from "react-router-dom";
import Home from "./pages/Home";
import About from "./pages/About";
import Layout from "./components/Layout";

function App() {
  return (
    <Routes>
      <Route path="/" element={<Layout />}>
        <Route index element={<Home />} />
        <Route path="about" element={<About />} />
      </Route>
    </Routes>
  );
}

export default App;
```

## Authentication

### Supabase Auth

**src/lib/supabase.ts** (already set up in With Supabase section)

**src/components/Auth.tsx**
```tsx
import { useState } from "react";
import { supabase } from "@/lib/supabase";

export function Auth() {
  const [loading, setLoading] = useState(false);

  const handleLogin = async () => {
    setLoading(true);
    await supabase.auth.signInWithOAuth({
      provider: "github",
      options: { redirectTo: window.location.origin },
    });
    setLoading(false);
  };

  return (
    <button onClick={handleLogin} disabled={loading}>
      {loading ? "Loading..." : "Sign in with GitHub"}
    </button>
  );
}
```

**src/hooks/useAuth.ts**
```ts
import { useEffect, useState } from "react";
import { supabase } from "@/lib/supabase";
import type { User } from "@supabase/supabase-js";

export function useAuth() {
  const [user, setUser] = useState<User | null>(null);
  const [loading, setLoading] = useState(true);

  useEffect(() => {
    supabase.auth.getUser().then(({ data: { user } }) => {
      setUser(user);
      setLoading(false);
    });

    const { data: { subscription } } = supabase.auth.onAuthStateChange(
      (_event, session) => setUser(session?.user ?? null)
    );

    return () => subscription.unsubscribe();
  }, []);

  const signOut = () => supabase.auth.signOut();

  return { user, loading, signOut };
}
```

### better-auth

Note: better-auth requires a backend. For React + Vite, use with Express or similar.

```json
{
  "dependencies": {
    "better-auth": "^1.0.0"
  }
}
```

**src/lib/auth-client.ts**
```ts
import { createAuthClient } from "better-auth/react";

export const authClient = createAuthClient({
  baseURL: import.meta.env.VITE_API_URL,
});

export const { signIn, signOut, useSession } = authClient;
```

**src/components/AuthButton.tsx**
```tsx
import { useSession, signIn, signOut } from "@/lib/auth-client";

export function AuthButton() {
  const { data: session, isPending } = useSession();

  if (isPending) return <div>Loading...</div>;

  if (session) {
    return <button onClick={() => signOut()}>Sign out</button>;
  }

  return (
    <button onClick={() => signIn.social({ provider: "github" })}>
      Sign in with GitHub
    </button>
  );
}
```

## Payment Integration (Stripe Subscription)

React + Vite requires a backend for Stripe operations. Use Supabase Edge Functions.

### Dependencies
```json
{
  "dependencies": {
    "@stripe/stripe-js": "^4.0.0"
  }
}
```

### Environment Variables
```env
VITE_STRIPE_PUBLISHABLE_KEY=pk_test_...
VITE_SUPABASE_URL=your-project-url
VITE_SUPABASE_ANON_KEY=your-anon-key
```

### Stripe Client

**src/lib/stripe.ts**
```ts
import { loadStripe } from "@stripe/stripe-js";

export const stripePromise = loadStripe(
  import.meta.env.VITE_STRIPE_PUBLISHABLE_KEY
);

export const PLANS = {
  basic: { name: "Basic", price: 980, features: ["Feature 1", "Feature 2"] },
  pro: { name: "Pro", price: 2980, features: ["All Basic", "Feature 3", "Feature 4"] },
  enterprise: { name: "Enterprise", price: 9800, features: ["All Pro", "Feature 5"] },
} as const;

export type PlanKey = keyof typeof PLANS;
```

### Supabase Edge Functions

Create edge functions for Stripe operations:

**supabase/functions/stripe-checkout/index.ts**
```ts
import { serve } from "https://deno.land/std@0.168.0/http/server.ts";
import Stripe from "https://esm.sh/stripe@17.0.0?target=deno";

const stripe = new Stripe(Deno.env.get("STRIPE_SECRET_KEY")!, {
  apiVersion: "2024-12-18.acacia",
});

const PRICE_IDS: Record<string, string> = {
  basic: Deno.env.get("STRIPE_PRICE_BASIC")!,
  pro: Deno.env.get("STRIPE_PRICE_PRO")!,
  enterprise: Deno.env.get("STRIPE_PRICE_ENTERPRISE")!,
};

serve(async (req) => {
  const { plan, email, userId } = await req.json();

  const customers = await stripe.customers.list({ email, limit: 1 });
  let customerId = customers.data[0]?.id;

  if (!customerId) {
    const customer = await stripe.customers.create({ email, metadata: { userId } });
    customerId = customer.id;
  }

  const session = await stripe.checkout.sessions.create({
    customer: customerId,
    mode: "subscription",
    line_items: [{ price: PRICE_IDS[plan], quantity: 1 }],
    success_url: `${req.headers.get("origin")}/dashboard?success=true`,
    cancel_url: `${req.headers.get("origin")}/pricing?canceled=true`,
  });

  return new Response(JSON.stringify({ url: session.url }), {
    headers: { "Content-Type": "application/json" },
  });
});
```

**supabase/functions/stripe-portal/index.ts**
```ts
import { serve } from "https://deno.land/std@0.168.0/http/server.ts";
import Stripe from "https://esm.sh/stripe@17.0.0?target=deno";

const stripe = new Stripe(Deno.env.get("STRIPE_SECRET_KEY")!, {
  apiVersion: "2024-12-18.acacia",
});

serve(async (req) => {
  const { email } = await req.json();

  const customers = await stripe.customers.list({ email, limit: 1 });
  const customerId = customers.data[0]?.id;

  if (!customerId) {
    return new Response(JSON.stringify({ error: "No customer found" }), { status: 404 });
  }

  const session = await stripe.billingPortal.sessions.create({
    customer: customerId,
    return_url: `${req.headers.get("origin")}/dashboard`,
  });

  return new Response(JSON.stringify({ url: session.url }), {
    headers: { "Content-Type": "application/json" },
  });
});
```

### Pricing Component

**src/components/PricingCard.tsx**
```tsx
import { useState } from "react";
import { PLANS, PlanKey } from "@/lib/stripe";
import { supabase } from "@/lib/supabase";
import { useAuth } from "@/hooks/useAuth";

export function PricingCard({ plan, recommended }: { plan: PlanKey; recommended?: boolean }) {
  const [loading, setLoading] = useState(false);
  const { user } = useAuth();
  const data = PLANS[plan];

  const handleSubscribe = async () => {
    if (!user) return;
    setLoading(true);

    const { data: result } = await supabase.functions.invoke("stripe-checkout", {
      body: { plan, email: user.email, userId: user.id },
    });

    if (result?.url) {
      window.location.href = result.url;
    }
    setLoading(false);
  };

  return (
    <div className={`p-6 rounded-lg border ${recommended ? "border-blue-500 ring-2 ring-blue-500" : ""}`}>
      {recommended && <span className="text-sm text-blue-500">Recommended</span>}
      <h3 className="text-xl font-bold mt-2">{data.name}</h3>
      <p className="text-3xl font-bold mt-4">
        ¥{data.price.toLocaleString()}<span className="text-sm">/month</span>
      </p>
      <ul className="mt-6 space-y-2">
        {data.features.map((f) => <li key={f}>✓ {f}</li>)}
      </ul>
      <button
        onClick={handleSubscribe}
        disabled={loading || !user}
        className="mt-6 w-full py-2 bg-blue-500 text-white rounded disabled:opacity-50"
      >
        {loading ? "Loading..." : "Subscribe"}
      </button>
    </div>
  );
}
```

### Manage Subscription

**src/components/ManageSubscription.tsx**
```tsx
import { useState } from "react";
import { supabase } from "@/lib/supabase";
import { useAuth } from "@/hooks/useAuth";

export function ManageSubscription() {
  const [loading, setLoading] = useState(false);
  const { user } = useAuth();

  const handleManage = async () => {
    if (!user) return;
    setLoading(true);

    const { data } = await supabase.functions.invoke("stripe-portal", {
      body: { email: user.email },
    });

    if (data?.url) {
      window.location.href = data.url;
    }
    setLoading(false);
  };

  return (
    <button onClick={handleManage} disabled={loading} className="text-blue-500 underline">
      {loading ? "Loading..." : "Manage Subscription"}
    </button>
  );
}
```

### Deploy Edge Functions
```bash
# Set secrets
supabase secrets set STRIPE_SECRET_KEY=sk_test_...
supabase secrets set STRIPE_PRICE_BASIC=price_...
supabase secrets set STRIPE_PRICE_PRO=price_...
supabase secrets set STRIPE_PRICE_ENTERPRISE=price_...

# Deploy
supabase functions deploy stripe-checkout
supabase functions deploy stripe-portal
```

## File Storage

### Supabase Storage

**src/lib/storage.ts**
```ts
import { supabase } from "./supabase";

export async function uploadFile(file: File, bucket: string, path: string) {
  const { data, error } = await supabase.storage
    .from(bucket)
    .upload(path, file, { upsert: true });

  if (error) throw error;
  return data;
}

export function getPublicUrl(bucket: string, path: string) {
  const { data } = supabase.storage.from(bucket).getPublicUrl(path);
  return data.publicUrl;
}
```

**src/components/FileUpload.tsx**
```tsx
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
      className={`border-2 border-dashed p-8 text-center rounded-lg cursor-pointer ${dragOver ? "border-blue-500 bg-blue-50" : ""}`}
    >
      {uploading ? "Uploading..." : "Drag & drop or click to upload"}
    </div>
  );
}
```

### Cloudflare R2 (via Supabase Edge Function)

**supabase/functions/r2-upload/index.ts**
```ts
import { serve } from "https://deno.land/std@0.168.0/http/server.ts";
import { S3Client, PutObjectCommand } from "npm:@aws-sdk/client-s3";
import { getSignedUrl } from "npm:@aws-sdk/s3-request-presigner";

const client = new S3Client({
  region: "auto",
  endpoint: `https://${Deno.env.get("R2_ACCOUNT_ID")}.r2.cloudflarestorage.com`,
  credentials: {
    accessKeyId: Deno.env.get("R2_ACCESS_KEY_ID")!,
    secretAccessKey: Deno.env.get("R2_SECRET_ACCESS_KEY")!,
  },
});

serve(async (req) => {
  const { filename, contentType } = await req.json();
  const key = `uploads/${Date.now()}-${filename}`;

  const command = new PutObjectCommand({
    Bucket: Deno.env.get("R2_BUCKET_NAME"),
    Key: key,
    ContentType: contentType,
  });

  const uploadUrl = await getSignedUrl(client, command, { expiresIn: 3600 });
  const publicUrl = `${Deno.env.get("R2_PUBLIC_URL")}/${key}`;

  return new Response(JSON.stringify({ uploadUrl, publicUrl }), {
    headers: { "Content-Type": "application/json" },
  });
});
```

## Email (Resend via Supabase Edge Function)

**supabase/functions/send-email/index.ts**
```ts
import { serve } from "https://deno.land/std@0.168.0/http/server.ts";
import { Resend } from "npm:resend";

const resend = new Resend(Deno.env.get("RESEND_API_KEY"));

serve(async (req) => {
  const { to, subject, template, data } = await req.json();

  let html = "";
  if (template === "welcome") {
    html = `
      <div style="font-family: sans-serif; padding: 40px; background: #fff;">
        <h1>Welcome, ${data.name}!</h1>
        <p>Thank you for signing up.</p>
        <a href="${data.loginUrl}" style="background: #3b82f6; color: #fff; padding: 12px 24px; border-radius: 6px; text-decoration: none;">
          Get Started
        </a>
      </div>
    `;
  } else if (template === "password-reset") {
    html = `
      <div style="font-family: sans-serif; padding: 40px; background: #fff;">
        <h1>Reset Your Password</h1>
        <p>Click the button below to reset your password.</p>
        <a href="${data.resetUrl}" style="background: #3b82f6; color: #fff; padding: 12px 24px; border-radius: 6px; text-decoration: none;">
          Reset Password
        </a>
      </div>
    `;
  }

  const { data: result, error } = await resend.emails.send({
    from: Deno.env.get("EMAIL_FROM")!,
    to,
    subject,
    html,
  });

  if (error) {
    return new Response(JSON.stringify({ error }), { status: 500 });
  }

  return new Response(JSON.stringify({ id: result?.id }), {
    headers: { "Content-Type": "application/json" },
  });
});
```

**Usage in React**
```ts
import { supabase } from "@/lib/supabase";

async function sendWelcomeEmail(email: string, name: string) {
  await supabase.functions.invoke("send-email", {
    body: {
      to: email,
      subject: "Welcome to Our App!",
      template: "welcome",
      data: { name, loginUrl: window.location.origin + "/login" },
    },
  });
}
```

### Deploy Edge Functions
```bash
supabase secrets set RESEND_API_KEY=re_...
supabase secrets set EMAIL_FROM=noreply@yourdomain.com
supabase functions deploy send-email
```
