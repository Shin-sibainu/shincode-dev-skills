---
name: growth-advisor
description: Data-driven revenue growth advisor using MCP servers (Supabase, Stripe, GA4, Search Console) to analyze metrics and suggest scaling strategies for profitable apps. Use when you want to increase revenue of an existing profitable app.
---

# Growth Advisor

Data-driven revenue scaling advisor for profitable apps. Uses MCP servers to collect real metrics and suggest actionable growth strategies.

## When to Use

- When you have a profitable app and want to scale revenue
- When you want data-driven growth strategies
- When user says "help me grow my app revenue" or "how can I scale my SaaS"

## Required MCP Servers

This skill works best with the following MCP servers connected:

| MCP Server | Purpose | Key Data |
|------------|---------|----------|
| **Supabase** | User & subscription data | ユーザー数、アクティブ率、登録推移 |
| **Stripe** | Revenue metrics | MRR、チャーン率、LTV、プラン別収益 |
| **GA4** | Traffic & conversion | PV、セッション、CV率、離脱ページ |
| **Search Console** | SEO performance | 検索流入、キーワード順位、CTR |

> **Note**: The skill adapts based on available MCP servers. Missing servers will be skipped.

---

## Workflow Overview

```
Step 1: ヒアリング (Interactive)
   ↓
Step 2: MCP接続確認 & データ収集
   ↓
Step 3: 現状分析 (Metrics Dashboard)
   ↓
Step 4: ボトルネック特定
   ↓
Step 5: 施策提案 (Prioritized Actions)
   ↓
Step 6: 実装サポート
```

---

## Step 1: ヒアリング

Use AskUserQuestion to gather context:

### 1a. アプリ概要

```
アプリ/サービスの概要を教えてください。
- サービス名
- 何を解決するサービスか
- ターゲットユーザー
```

### 1b. 現在の状況

```
現在の収益状況を教えてください。
- 月間売上（MRR）: 概算でOK
- 有料ユーザー数: 概算でOK
- 主な収益源: サブスク / 買い切り / 広告
```

### 1c. 目標

```
目標を教えてください。
- 目標MRR: いくらまで伸ばしたいか
- 期間: いつまでに達成したいか
- 優先事項: 新規獲得 / 解約防止 / 単価UP / その他
```

### 1d. 接続済みMCPサーバー

```
接続可能なMCPサーバーを選択してください（複数選択可）
- Supabase / Turso（BaaS）
- Stripe（決済）
- Google Analytics 4
- Google Search Console
- なし（手動でデータ入力）
```

---

## Step 2: MCP接続確認 & データ収集

### 2a. 利用可能なMCPサーバーを確認

Check which MCP servers are available in the current session.

### 2b. データ収集クエリ

#### Supabase MCP

```sql
-- ユーザー数推移（過去30日）
SELECT
  DATE(created_at) as date,
  COUNT(*) as new_users
FROM users
WHERE created_at >= NOW() - INTERVAL '30 days'
GROUP BY DATE(created_at)
ORDER BY date;

-- プラン別ユーザー数
SELECT
  subscription_plan,
  COUNT(*) as user_count
FROM users
WHERE subscription_status = 'active'
GROUP BY subscription_plan;

-- 解約ユーザー（過去30日）
SELECT COUNT(*) as churned_users
FROM users
WHERE subscription_status = 'cancelled'
AND updated_at >= NOW() - INTERVAL '30 days';
```

#### Stripe MCP

Use Stripe MCP tools to retrieve:

```
1. MRR (Monthly Recurring Revenue)
   - stripe_list_subscriptions で有効なサブスク取得
   - プラン別の収益を集計

2. Churn Rate
   - stripe_list_subscriptions(status: "canceled")
   - 過去30日のキャンセル数 / 月初のアクティブ数

3. LTV (Lifetime Value)
   - stripe_list_customers でcustomer取得
   - 各customerの総支払額を計算

4. Plan Distribution
   - プラン別の契約数と収益
```

#### GA4 MCP

```
1. Traffic Overview
   - 過去30日のセッション数
   - ユーザー数（新規/リピーター）
   - 流入元別（Organic, Direct, Referral, Social）

2. Conversion Funnel
   - LP → 登録ページ → 登録完了 → 課金
   - 各ステップの離脱率

3. Top Pages
   - PV上位ページ
   - 離脱率の高いページ

4. User Behavior
   - 平均セッション時間
   - ページ/セッション
```

#### Search Console MCP

```
1. Search Performance
   - 過去28日のクリック数、表示回数
   - 平均CTR、平均順位

2. Top Queries
   - クリック数上位キーワード
   - 表示されているが順位が低いキーワード（改善機会）

3. Top Pages
   - 検索流入の多いページ
   - CTRの低いページ（タイトル改善機会）
```

---

## Step 3: 現状分析

Compile collected data into a metrics dashboard:

### 出力フォーマット

```markdown
## 現状メトリクスダッシュボード

### 収益指標 (Stripe)
| 指標 | 値 | 前月比 |
|------|-----|--------|
| MRR | ¥XXX,XXX | +XX% |
| 有料ユーザー数 | XXX人 | +XX人 |
| ARPU | ¥X,XXX | - |
| チャーン率 | X.X% | - |
| LTV | ¥XX,XXX | - |

### プラン別内訳
| プラン | ユーザー数 | 収益 | 割合 |
|--------|-----------|------|------|
| Basic | XX人 | ¥XX,XXX | XX% |
| Pro | XX人 | ¥XX,XXX | XX% |
| Enterprise | XX人 | ¥XX,XXX | XX% |

### トラフィック指標 (GA4)
| 指標 | 値 | 前月比 |
|------|-----|--------|
| セッション数 | X,XXX | +XX% |
| ユーザー数 | X,XXX | +XX% |
| 新規ユーザー率 | XX% | - |
| 直帰率 | XX% | - |

### 流入元内訳
| チャネル | セッション | 割合 |
|----------|-----------|------|
| Organic Search | X,XXX | XX% |
| Direct | X,XXX | XX% |
| Referral | X,XXX | XX% |
| Social | X,XXX | XX% |

### SEO指標 (Search Console)
| 指標 | 値 | 前月比 |
|------|-----|--------|
| 検索クリック | X,XXX | +XX% |
| 表示回数 | XX,XXX | +XX% |
| 平均CTR | X.X% | - |
| 平均順位 | XX.X | - |

### コンバージョンファネル
| ステップ | 数 | 転換率 |
|----------|-----|--------|
| LP訪問 | X,XXX | 100% |
| 登録ページ | XXX | XX% |
| 登録完了 | XXX | XX% |
| 有料転換 | XX | XX% |
```

---

## Step 4: ボトルネック特定

Analyze the data to identify bottlenecks:

### チェックリスト

| カテゴリ | チェック項目 | 判定基準 |
|----------|-------------|----------|
| **集客** | オーガニック流入 | < 50%なら改善余地 |
| **集客** | 直帰率 | > 60%なら改善必要 |
| **転換** | LP→登録率 | < 5%なら改善必要 |
| **転換** | 登録→課金率 | < 10%なら改善必要 |
| **収益** | チャーン率 | > 5%なら改善必要 |
| **収益** | ARPU | 業界平均と比較 |
| **SEO** | 平均CTR | < 3%なら改善余地 |
| **SEO** | 上位表示キーワード数 | 増加傾向か確認 |

### ボトルネック判定ロジック

```
IF チャーン率 > 5% THEN
  → 解約防止が最優先（穴の空いたバケツ問題）

ELSE IF LP→登録率 < 5% THEN
  → LP改善が優先（せっかくのトラフィックを逃している）

ELSE IF 登録→課金率 < 10% THEN
  → オンボーディング改善が優先

ELSE IF オーガニック < 30% THEN
  → SEO/コンテンツ強化が優先（広告依存リスク）

ELSE
  → 単価UP or 新規チャネル開拓
```

### 出力フォーマット

```markdown
## ボトルネック分析

### 最大のボトルネック
**[ボトルネック名]**: [説明]

例: 「登録→課金の転換率が3%と低い。業界平均10%と比較して大きな機会損失。」

### 機会損失の試算
- 現状: 月XX人が登録 → XX人が課金
- 改善後（転換率10%想定）: 月XX人が課金
- **逸失収益: 約¥XXX,XXX/月**

### その他の改善ポイント
1. [ポイント1]
2. [ポイント2]
3. [ポイント3]
```

---

## Step 5: 施策提案

Based on bottlenecks, propose prioritized actions:

### 施策カテゴリ

#### A. 解約防止（チャーン対策）

| 施策 | 期待効果 | 難易度 | 優先度 |
|------|----------|--------|--------|
| 解約理由アンケート設置 | チャーン原因の特定 | 低 | 高 |
| 解約前リテンションオファー | チャーン率-1-2% | 中 | 高 |
| オンボーディングメール改善 | 初月チャーン減少 | 低 | 中 |
| 利用状況アラート | 休眠ユーザー復活 | 中 | 中 |
| カスタマーサクセス導入 | LTV向上 | 高 | 低 |

#### B. 転換率改善

| 施策 | 期待効果 | 難易度 | 優先度 |
|------|----------|--------|--------|
| LP A/Bテスト | CV率+20-50% | 低 | 高 |
| 社会的証明追加（実績、レビュー） | CV率+10-30% | 低 | 高 |
| 無料トライアル導入/延長 | 登録率UP | 中 | 中 |
| オンボーディングチェックリスト | 課金率UP | 中 | 中 |
| ライブチャットサポート | CV率+10% | 中 | 低 |

#### C. 単価UP（ARPU向上）

| 施策 | 期待効果 | 難易度 | 優先度 |
|------|----------|--------|--------|
| 価格改定（値上げ） | MRR+20-50% | 低 | 高 |
| 上位プラン訴求強化 | アップセル+10% | 低 | 中 |
| アドオン機能追加 | ARPU+10-20% | 高 | 中 |
| 年間プラン割引 | キャッシュフロー改善 | 低 | 中 |

#### D. 集客強化

| 施策 | 期待効果 | 難易度 | 優先度 |
|------|----------|--------|--------|
| SEOコンテンツ強化 | オーガニック+50-100% | 中 | 高 |
| Product Hunt再ローンチ | 一時的トラフィック急増 | 低 | 中 |
| アフィリエイトプログラム | 持続的な新規獲得 | 中 | 中 |
| SNS運用強化 | ブランド認知向上 | 中 | 低 |

### 出力フォーマット

```markdown
## 施策提案（優先度順）

### 今すぐやるべき（This Week）

#### 1. [施策名]
- **期待効果**: [具体的な数値]
- **難易度**: 低/中/高
- **実装方法**: [概要]

#### 2. [施策名]
...

### 今月中にやるべき（This Month）

#### 3. [施策名]
...

### 四半期で検討（This Quarter）

#### 5. [施策名]
...

---

### 期待される成果

| 施策 | 現状 | 目標 | MRRインパクト |
|------|------|------|--------------|
| [施策1] | X% | Y% | +¥XX,XXX |
| [施策2] | X% | Y% | +¥XX,XXX |
| **合計** | - | - | **+¥XXX,XXX** |
```

---

## Step 6: 実装サポート

For selected actions, provide implementation guidance:

### 実装テンプレート

#### 解約理由アンケート

```typescript
// app/api/cancel-survey/route.ts
import { createClient } from '@supabase/supabase-js';

const CANCEL_REASONS = [
  { id: 'too_expensive', label: '料金が高い' },
  { id: 'not_using', label: '使わなくなった' },
  { id: 'missing_features', label: '必要な機能がない' },
  { id: 'competitor', label: '他のサービスに乗り換え' },
  { id: 'other', label: 'その他' },
];

export async function POST(req: Request) {
  const { userId, reason, feedback } = await req.json();

  const supabase = createClient(
    process.env.SUPABASE_URL!,
    process.env.SUPABASE_SERVICE_KEY!
  );

  await supabase.from('cancel_surveys').insert({
    user_id: userId,
    reason,
    feedback,
    created_at: new Date().toISOString(),
  });

  return Response.json({ success: true });
}
```

#### リテンションオファー（解約前）

```typescript
// components/CancelFlow.tsx
'use client';

import { useState } from 'react';

export function CancelFlow({ onCancel }: { onCancel: () => void }) {
  const [step, setStep] = useState<'reason' | 'offer' | 'confirm'>('reason');
  const [reason, setReason] = useState('');

  const handleReasonSubmit = () => {
    // 料金が理由なら割引オファー
    if (reason === 'too_expensive') {
      setStep('offer');
    } else {
      setStep('confirm');
    }
  };

  if (step === 'offer') {
    return (
      <div className="p-6 bg-blue-50 rounded-lg">
        <h3 className="text-lg font-bold">特別オファー</h3>
        <p>次の3ヶ月間、50%オフでご利用いただけます。</p>
        <button
          onClick={() => applyDiscount()}
          className="btn btn-primary"
        >
          オファーを受ける
        </button>
        <button
          onClick={() => setStep('confirm')}
          className="btn btn-ghost"
        >
          解約を続ける
        </button>
      </div>
    );
  }
  // ... rest of the flow
}
```

#### LP A/Bテスト設定

```typescript
// lib/ab-test.ts
import { cookies } from 'next/headers';

export function getVariant(testName: string): 'A' | 'B' {
  const cookieStore = cookies();
  const existing = cookieStore.get(`ab_${testName}`);

  if (existing) {
    return existing.value as 'A' | 'B';
  }

  // 50/50 split
  const variant = Math.random() < 0.5 ? 'A' : 'B';

  // Track in analytics
  trackEvent('ab_test_assigned', {
    test: testName,
    variant,
  });

  return variant;
}

// Usage in LP
export default function LandingPage() {
  const heroVariant = getVariant('hero_headline');

  return (
    <Hero
      headline={heroVariant === 'A'
        ? '売上を3倍にする'
        : '成長を加速させる'
      }
    />
  );
}
```

---

## Quick Analysis Mode

If user wants faster results without full MCP data:

```
ユーザー: 「簡単に分析して」

→ Step 1 (ヒアリング) で手動で数値入力
→ Step 4-5 (ボトルネック分析 + 施策提案) に直接ジャンプ
→ MCP連携なしで一般的なベストプラクティスを提案
```

---

## Growth Metrics Benchmarks

Reference benchmarks for SaaS:

| 指標 | 悪い | 普通 | 良い | 優秀 |
|------|------|------|------|------|
| チャーン率（月次） | >8% | 5-8% | 3-5% | <3% |
| LP→登録率 | <2% | 2-5% | 5-10% | >10% |
| 登録→課金率 | <5% | 5-10% | 10-20% | >20% |
| NPS | <0 | 0-30 | 30-50 | >50 |
| CAC回収期間 | >18ヶ月 | 12-18ヶ月 | 6-12ヶ月 | <6ヶ月 |

---

## MCP Server Setup References

### Supabase MCP

```json
{
  "mcpServers": {
    "supabase": {
      "command": "npx",
      "args": ["-y", "@supabase/mcp-server"],
      "env": {
        "SUPABASE_URL": "https://xxx.supabase.co",
        "SUPABASE_SERVICE_KEY": "your-service-key"
      }
    }
  }
}
```

### Stripe MCP

```json
{
  "mcpServers": {
    "stripe": {
      "command": "npx",
      "args": ["-y", "@stripe/mcp"],
      "env": {
        "STRIPE_SECRET_KEY": "sk_live_xxx"
      }
    }
  }
}
```

### GA4 MCP

```json
{
  "mcpServers": {
    "ga4": {
      "command": "npx",
      "args": ["-y", "@anthropic/ga4-mcp"],
      "env": {
        "GA4_PROPERTY_ID": "properties/XXXXXXXXX",
        "GOOGLE_APPLICATION_CREDENTIALS": "/path/to/credentials.json"
      }
    }
  }
}
```

### Search Console MCP

```json
{
  "mcpServers": {
    "search-console": {
      "command": "npx",
      "args": ["-y", "@anthropic/search-console-mcp"],
      "env": {
        "SITE_URL": "https://your-site.com",
        "GOOGLE_APPLICATION_CREDENTIALS": "/path/to/credentials.json"
      }
    }
  }
}
```
