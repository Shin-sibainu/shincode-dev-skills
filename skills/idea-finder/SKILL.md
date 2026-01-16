---
name: idea-finder
description: Niche business idea discovery tool. Helps find profitable app/service ideas through interactive questions, trend analysis, competitor research, and market validation. Use before starting a new project to find ideas worth building.
---

# Idea Finder

Find profitable niche ideas for web apps and services through research and analysis.

## When to Use

- Before starting a new project
- When looking for side project ideas
- When pivoting to a new market
- When user says "help me find an app idea" or "what should I build?"

## Workflow Overview

```
Step 1: ヒアリング (Interactive)
   ↓
Step 2: トレンド分析 (Web Search)
   ↓
Step 3: 競合リサーチ (Web Search)
   ↓
Step 4: アイデア生成
   ↓
Step 5: 市場検証
   ↓
Step 6: ペルソナ作成
   ↓
Step 7: 最終レポート
```

---

## Step 1: ヒアリング

Use AskUserQuestion to gather information:

### 1a. スキル・興味

```
あなたの得意分野や興味のある領域を教えてください。
例: プログラミング、デザイン、マーケティング、教育、ヘルスケアなど
```

### 1b. ターゲット層

```
ターゲットにしたいユーザー層は？
- 開発者・エンジニア
- 中小企業・スタートアップ
- フリーランス・個人事業主
- 学生・教育関係
- 主婦・一般消費者
- クリエイター（デザイナー、動画制作者など）
- その他（自由記述）
```

### 1c. 解決したい課題

```
日常で感じる不満や「こんなツールがあれば」と思うことはありますか？
または、特定の業界・分野の課題があれば教えてください。
```

### 1d. 収益モデルの希望

```
希望する収益モデルは？
- サブスクリプション（月額課金）
- 買い切り（一度の購入）
- フリーミアム（基本無料+有料機能）
- 広告モデル
- こだわりなし
```

### 1e. 開発規模

```
想定する開発規模は？
- 小規模（1-2週間で作れるもの）
- 中規模（1-2ヶ月で作れるもの）
- 大規模（3ヶ月以上）
```

---

## Step 2: トレンド分析

Use WebSearch to research current trends based on user's interests.

### 検索クエリ例

Based on user's answers, search for:

```
"{ユーザーの興味分野} SaaS trends 2025"
"{ターゲット層} software tools trending"
"micro SaaS ideas {分野}"
"indie hacker successful products {分野}"
```

### 分析ポイント

| 観点 | 確認内容 |
|------|----------|
| 成長市場 | 伸びている分野・縮小している分野 |
| 新技術 | AI、自動化など活用できる技術 |
| 規制変化 | 新しい法律・規制による機会 |
| 行動変化 | リモートワークなど生活様式の変化 |

### 出力フォーマット

```markdown
## トレンド分析結果

### 成長している分野
1. [分野名] - [理由]
2. [分野名] - [理由]

### 注目の技術トレンド
- [技術] - [活用方法]

### 市場機会
- [機会の説明]
```

---

## Step 3: 競合リサーチ

Use WebSearch to find existing solutions and gaps.

### 検索クエリ例

```
"{アイデアのキーワード} alternatives"
"{アイデアのキーワード} competitors"
"best {カテゴリ} tools for {ターゲット}"
"{カテゴリ} software market size"
```

### 競合分析マトリックス

For each potential competitor found:

| 項目 | 確認内容 |
|------|----------|
| 名前 | サービス名 |
| URL | サイトURL |
| 価格帯 | 無料/有料、月額いくらか |
| 強み | 何が優れているか |
| 弱み | レビューで不満な点 |
| ターゲット | 誰向けか |
| 差別化機会 | 攻められる隙間 |

### ギャップ分析

```markdown
## 競合分析結果

### 主要競合 (3-5社)

#### 1. [競合名]
- URL: [URL]
- 価格: [価格]
- 強み: [強み]
- 弱み/不満点: [レビューから抽出]

### 市場のギャップ（参入機会）
1. [ギャップ1] - 競合が対応していない領域
2. [ギャップ2] - 価格帯の隙間
3. [ギャップ3] - 特定ニッチへの特化
```

---

## Step 4: アイデア生成

Based on Steps 1-3, generate 3-5 niche ideas.

### アイデアテンプレート

For each idea:

```markdown
### アイデア [番号]: [アイデア名]

**一言説明**: [1文で説明]

**解決する課題**:
[ターゲット]が[課題]で困っている。
このツールは[解決方法]を提供する。

**主な機能**:
1. [機能1]
2. [機能2]
3. [機能3]

**収益モデル**: [サブスク/買い切り/フリーミアム]
**想定価格**: [価格帯]

**なぜニッチか**:
- [差別化ポイント1]
- [差別化ポイント2]

**技術スタック案**:
- Frontend: Next.js
- Backend: Supabase
- 追加: [必要な技術]

**開発難易度**: ★☆☆☆☆ 〜 ★★★★★
**市場ポテンシャル**: ★☆☆☆☆ 〜 ★★★★★
```

---

## Step 5: 市場検証

For the most promising idea(s), validate with additional research.

### 検証項目

Use WebSearch for:

```
"{アイデア関連キーワード} market size"
"{アイデア関連キーワード} user reviews complaints"
"reddit {アイデア関連キーワード} recommendations"
"twitter {アイデア関連キーワード} looking for"
```

### 検証チェックリスト

| 項目 | 確認方法 | 結果 |
|------|----------|------|
| 検索ボリューム | キーワードの検索数 | 高/中/低 |
| 既存の不満 | レビューサイト、Reddit | あり/なし |
| お金を払う意思 | 類似有料サービスの存在 | あり/なし |
| 技術的実現性 | 必要な技術・API | 可能/困難 |
| 参入障壁 | 競合の強さ | 低/中/高 |

### 検証結果フォーマット

```markdown
## 市場検証結果: [アイデア名]

### 市場規模
- 推定市場規模: [金額または規模感]
- 成長率: [成長/横ばい/縮小]

### 需要の証拠
- [証拠1: Redditでの質問、レビューの不満など]
- [証拠2]

### リスク
- [リスク1]
- [リスク2]

### Go/No-Go判定: [GO / 要検討 / NO-GO]
```

---

## Step 6: ペルソナ作成

Create a detailed user persona for the selected idea.

### ペルソナテンプレート

```markdown
## ターゲットペルソナ

### 基本情報
- **名前**: [架空の名前]
- **年齢**: [年齢層]
- **職業**: [職業]
- **年収**: [収入レンジ]
- **居住地**: [都市部/地方など]

### 日常
- **典型的な1日**: [朝〜夜の流れ]
- **使用ツール**: [普段使っているアプリ・サービス]
- **情報収集**: [Twitter, YouTube, ブログなど]

### 課題・ペイン
1. [課題1]: [詳細]
2. [課題2]: [詳細]
3. [課題3]: [詳細]

### ゴール
- **短期**: [すぐに達成したいこと]
- **長期**: [将来的に達成したいこと]

### 購買行動
- **予算感**: [ツールに払える金額]
- **決め手**: [購入を決める要因]
- **障壁**: [購入をためらう要因]

### このペルソナに響くメッセージ
「[キャッチコピー案]」
```

---

## Step 7: 最終レポート

Compile all findings into a final report.

### レポートフォーマット

```markdown
# アイデア発見レポート

## エグゼクティブサマリー
[3-5文で全体をまとめる]

## 推奨アイデア: [アイデア名]

### なぜこのアイデアか
1. [理由1]
2. [理由2]
3. [理由3]

### 市場機会
- 市場規模: [規模]
- 競合状況: [状況]
- 差別化ポイント: [ポイント]

### ターゲットユーザー
[ペルソナの要約]

### 収益予測
| プラン | 価格 | 想定ユーザー数 | 月間収益 |
|--------|------|---------------|----------|
| Basic | ¥XX | XX人 | ¥XX |
| Pro | ¥XX | XX人 | ¥XX |

### 次のステップ
1. `/new-webapp` でプロジェクトセットアップ
2. `/dev-workflow` で開発開始
3. MVP完成後、[検証方法]で市場テスト

## 他の候補アイデア
[検討した他のアイデアの簡易リスト]

---
レポート生成日: [日付]
```

---

## Quick Mode

If user wants faster results, skip detailed research:

```
ユーザー: 「簡単にアイデアだけ出して」

→ Step 1 (ヒアリング) のみ実施
→ Step 4 (アイデア生成) に直接ジャンプ
→ 簡易版アイデアリスト3-5個を出力
```

---

## Tips for Good Ideas

### ニッチの見つけ方

1. **既存ツールの特化版**: 「Notion for [特定職種]」
2. **2つの分野の掛け合わせ**: 「AI × [業界]」
3. **面倒な作業の自動化**: 「[手作業]を自動化」
4. **特定地域・言語特化**: 「日本向けの[サービス]」
5. **価格帯の隙間**: 「[高額ツール]の廉価版」

### 避けるべきアイデア

- 大企業が既に支配している市場
- 技術的に実現困難なもの
- 法的リスクが高いもの
- 自分が使わない/理解できないもの
