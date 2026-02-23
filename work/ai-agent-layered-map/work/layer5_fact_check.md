# 第5層・製品解説 ファクトチェック結果

**検証日**: 2026-02-22
**検証対象ファイル**:
- `output/05_UI運用層.md`
- `output/07_a_ClaudeCode.md`
- `output/07_f_その他製品.md`

---

## 1. Claude Code の最新機能

### 記載内容の確認

| 項目 | 記載内容 | 検証結果 | 判定 |
|------|---------|----------|------|
| 提供元 | Anthropic | 正確 | OK |
| 形態 | CLI（コマンドラインインターフェース） | 正確 | OK |
| 主要コマンド | `claude` | 正確 | OK |
| 対象LLM | Claude Sonnet / Opus | 正確（Sonnet 4.6、Opus 4.6が最新） | OK |
| Sub-agents | サブエージェント機能 | 正確 | OK |
| Agent Teams | チーム協働機能 | 正確（`/teams`コマンド） | OK |
| MCP対応 | MCPクライアント機能 | 正確 | OK |
| Hooks | PreToolUse / PostToolUse | 正確 | OK |
| Skills | Progressive Disclosure | 正確 | OK |

### 最新機能の追加情報（2026年時点）

Web検索で確認された最新機能（記載されていないもの）:

| 機能 | 説明 | 対応要否 |
|------|------|----------|
| **Claude in Chrome (Beta)** | Chrome拡張と連携してブラウザを直接制御 | 追記推奨 |
| **Claude Agent SDK** | 旧Claude Code SDK。サブエージェント・Hooks対応 | 追記推奨 |
| **Claude Code Analytics API** | 組織向け利用状況API | 任意 |
| **Claude Code Security** | コードベースの脆弱性スキャン | 追記推奨 |
| **checkpointing機能** | 長時間タスクの進捗管理 | 任意 |
| **npm非推奨化** | ネイティブインストール推奨 | 追記推奨 |
| **Claude Sonnet 4.6 / Opus 4.6** | 最新モデル対応 | モデル名更新推奨 |

**Sources**:
- [Claude Code CHANGELOG](https://github.com/anthropics/claude-code/blob/main/CHANGELOG.md)
- [Claude Code Overview](https://code.claude.com/docs/en/overview)
- [Claude Code Releases](https://github.com/anthropics/claude-code/releases)
- [Enabling Claude Code to work more autonomously](https://www.anthropic.com/news/enabling-claude-code-to-work-more-autonomously)

---

## 2. Cursor の機能と特徴

### 記載内容の確認

| 項目 | 記載内容 | 検証結果 | 判定 |
|------|---------|----------|------|
| 概要 | AI機能をネイティブ統合したVSCodeフォーク | 正確 | OK |
| 形態 | IDE (VS Code fork) | 正確 | OK |
| 主要LLM | 複数LLM対応 | 正確（Claude 4.5 Sonnet, GPT-5.2等） | OK |

### 最新機能（2026年時点）

| 機能 | 説明 | 記載状況 |
|------|------|----------|
| **Composer** | 自然言語でリポジトリ全体の編集を実行 | 05_UI運用層.mdには未記載 |
| **Parallel Subagents** | 複数ワーカーの並列実行 | 未記載 |
| **Background Agents** | 別ブランチでの非同期作業、PR作成 | 未記載 |
| **Agent Mode** | ターミナル・ブラウザアクセス付き自律エージェント | 未記載 |
| **Plan Mode & Rules** | `.cursor/rules/`フォルダでルール管理 | 未記載 |

**備考**: 05_UI運用層.mdでは概要レベルの記載のみ。詳細解説が必要な場合は07_b_IDE統合AI.mdで対応。

**Sources**:
- [Cursor AI Review 2026](https://www.nxcode.io/resources/news/cursor-review-2026)
- [Cursor IDE in 2026](https://techjacksolutions.com/ai/ai-development/cursor-ide-what-it-is/)
- [Best Cursor AI Settings 2026](https://mindevix.com/ai-usage-strategy/best-cursor-ai-settings-2026/)

---

## 3. Dify の機能と特徴

### 記載内容の確認

| 項目 | 記載内容 | 検証結果 | 判定 |
|------|---------|----------|------|
| 概要 | ローコードでAIエージェントを構築できるプラットフォーム | 正確 | OK |
| WebUI | ダッシュボード、ユーザー管理 | 正確 | OK |
| ワークフローエンジン | GUIでノードを接続 | 正確 | OK |
| ノードタイプ | LLM、条件分岐、ツール、コード、ナレッジベース | 正確 | OK |
| 層構成 | 第5層（UI）+ 第3層（ワークフロー）+ 第4層（ツール） | 正確 | OK |

### 最新機能（2026年時点）

| 機能 | 説明 | 記載状況 |
|------|------|----------|
| **Trigger Automation** | ファイル変更等をトリガーに自動実行 | 未記載 |
| **Plugin System** | マルチモーダル対応、OCR等のプラグイン | 未記載 |
| **DSL (Domain Specific Language)** | ワークフローのエクスポート・インポート | 未記載 |
| **RBAC** | ロールベースアクセス制御（Enterprise） | 未記載 |

**判定**: 基本的な記載内容は正確。2026年の最新機能（Trigger、Plugin）は追記推奨。

**Sources**:
- [Dify公式](https://dify.ai/)
- [Dify GitHub](https://github.com/langgenius/dify)
- [Dify AI Review 2026](https://www.gptbots.ai/blog/dify-ai)
- [Introducing Dify Plugins](https://dify.ai/blog/introducing-dify-plugins)

---

## 4. Kiro の機能（Specs駆動開発）

### 記載内容の確認

| 項目 | 記載内容 | 検証結果 | 判定 |
|------|---------|----------|------|
| 提供元 | AWS | 正確 | OK |
| 概要 | AI IDE、Specs駆動開発 | 正確 | OK |
| 発表時期 | 2025年 | 正確（2025年7月14日プレビュー開始） | OK |
| VS Code互換 | VS Code互換環境 | 正確（Code OSSフォーク） | OK |
| Specs構成 | Requirements, Design, Tasks, Tests | **部分的に不正確** | 要修正 |

### 問題点

**Specsの構成要素について修正が必要**:

| 記載内容 | 正確な情報 |
|---------|-----------|
| Requirements, Design, Tasks, Tests | **requirements.md, design.md, tasks.md** の3ファイル |

公式ドキュメントによると、Kiroのスペックは3つの主要ファイルで構成される:
1. `requirements.md` - ユーザーストーリーと受け入れ基準（EARS形式）
2. `design.md` - 技術アーキテクチャ（コンポーネント、データモデル、インターフェース）
3. `tasks.md` - 実装タスクのチェックリスト

「Tests」は別ファイルではなく、`requirements.md`の受け入れ基準に含まれる。

### 追加情報

| 項目 | 情報 |
|------|------|
| **LLM** | Claude Sonnet 4 モデル（Bedrock経由） |
| **価格** | Free（50回/月）、Pro $19（1,000回/月）、Pro+ $39（3,000回/月） |
| **Hooks機能** | ファイル変更やコミットをトリガーに自動実行 |
| **AWS非依存** | 任意のクラウド・技術スタックで使用可能 |

**Sources**:
- [Kiro公式](https://kiro.dev/)
- [Kiro Docs - Specs](https://kiro.dev/docs/specs/)
- [AWS Kiro: Testing an AI IDE](https://thenewstack.io/aws-kiro-testing-an-ai-ide-with-a-spec-driven-approach/)
- [InfoQ - AWS Kiro](https://www.infoq.com/news/2025/08/aws-kiro-spec-driven-agent/)

---

## 5. Claude Cowork（Computer Use）の仕様

### 記載内容の確認

| 項目 | 記載内容 | 検証結果 | 判定 |
|------|---------|----------|------|
| 概要 | Claude Desktop上の機能、Computer Use活用 | 正確 | OK |
| 操作方法 | Visionでスクリーンショット解析、座標指定 | 正確 | OK |
| 操作タイプ | mouse_move, click, type, key, screenshot等 | 正確 | OK |
| ツール | computer, bash, text_editor | 正確 | OK |

### 最新情報（2026年時点）

| 項目 | 情報 | 記載状況 |
|------|------|----------|
| **Windows対応** | 2026年2月11日にWindows版リリース | 未記載 |
| **リリース時期** | 2026年1月（Mac）、2026年2月（Windows） | 未記載 |
| **Pro/Team/Enterprise** | Team/Enterprise: 2026/1/23、Pro: 2026/1/16 | 未記載 |
| **開発期間** | Claude Codeを使って約2週間で開発 | 任意 |
| **MCP対応** | MCPコネクタで外部サービス統合 | 未記載 |
| **Sub-agent coordination** | 複雑なタスクを分割して並列実行 | 未記載 |

### 操作タイプの追加情報

Web検索で確認された操作タイプ:

| 操作タイプ | 記載状況 | バージョン |
|-----------|----------|------------|
| zoom | 未記載 | computer_20251124（Claude 4.6系） |
| scroll（方向・量指定） | 記載あり | computer_20250124 |

**Sources**:
- [Claude Cowork Getting Started](https://support.claude.com/en/articles/13345190-getting-started-with-cowork)
- [Introducing Cowork](https://claude.com/blog/cowork-research-preview)
- [VentureBeat - Claude Cowork Windows](https://venturebeat.com/technology/anthropics-claude-cowork-finally-lands-on-windows-and-it-wants-to-automate)
- [Computer Use Tool Docs](https://platform.claude.com/docs/en/agents-and-tools/tool-use/computer-use-tool)

---

## 6. Perplexity の検索方式（従来型RAGかTool型か）

### 記載内容の確認

| 項目 | 記載内容 | 検証結果 | 判定 |
|------|---------|----------|------|
| 分類 | 従来型RAGアプリ | **要検証** | 検証中 |
| 検索判断 | アプリ層が決定 | **部分的に不正確** | 要修正 |
| 検索タイミング | 固定フロー | **不正確** | 要修正 |

### 検証結果

Web検索の結果、Perplexityのアーキテクチャについて以下が判明:

**1. 常時検索アーキテクチャ**
- Perplexity AI は**常にライブ検索を実行**する設計
- ChatGPTの「必要な時だけブラウジング」とは異なるアプローチ
- 「従来型RAG」というより「検索ファースト設計」

**2. LLMの役割**
- クエリの理解・書き換え（Query Understanding）はLLMが実行
- モデルルーティング（どのLLMを使うか）はシステムが判断
- 検索実行自体はアプリ層が常に実行

**3. Sonar API のagentic機能**
- Sonar Pro Search は「agentic search system」と公式に記載
- 複数ステップの推論・計画・ツール実行が可能
- `search_classifier`パラメータで検索要否を動的判断可能

### 結論

| 観点 | 判定 |
|------|------|
| 基本動作 | 従来型RAG（常時検索）に近い |
| API機能 | Tool型RAGの要素あり（search_classifier、agentic features） |
| 記載の正確性 | 「従来型RAG」は基本動作として正確だが、API側のagentic機能について補足が必要 |

**推奨修正**:
- 「従来型RAGアプリ」の記載は維持
- ただし「Sonar Pro API ではagentic機能（search_classifier）も提供」と補足追加

**Sources**:
- [Perplexity Architecture - Frugal Testing](https://www.frugaltesting.com/blog/behind-perplexitys-architecture-how-ai-search-handles-real-time-web-data)
- [How Perplexity Built an AI Google](https://blog.bytebytego.com/p/how-perplexity-built-an-ai-google)
- [Perplexity Vespa.ai](https://vespa.ai/perplexity/)
- [Sonar API Docs](https://docs.perplexity.ai/docs/sonar/quickstart)
- [Introducing Sonar Pro API](https://www.perplexity.ai/hub/blog/introducing-the-sonar-pro-api)

---

## 7. 源内の構成（デジタル庁ブログを参照）

### 記載内容の確認

| 項目 | 記載内容 | 検証結果 | 判定 |
|------|---------|----------|------|
| 概要 | 政府職員向け生成AIプラットフォーム | 正確 | OK |
| 名称由来 | 平賀源内 | 正確（+ "Gen AI" の略称） | OK |
| 源内Web | 認証基盤（第5層） | **確認必要** | 要追加調査 |
| 源内アプリ群 | UIレスの業務特化アプリ | **確認必要** | 要追加調査 |
| デフォルトチャット | 第3層+第1層、第4層連携なし | **確認必要** | 要追加調査 |

### Web検索で確認された情報

| 項目 | 確認内容 |
|------|----------|
| **提供開始** | 2025年5月（デジタル庁職員向け） |
| **アプリ数** | 2025年8月時点で20種類以上の行政実務特化アプリ |
| **利用モデル** | AWS Nova Lite、Claude 4.5 Haiku、Claude 4.5 Sonnet（2025年11月時点） |
| **国産LLM** | PFN「PLaMo翻訳」（2025年12月導入開始） |
| **展開予定** | 2026年1月以降に一部省庁導入、2026年度以降に希望省庁へ本格展開 |
| **セキュリティ** | ガバメントクラウド内運用、日本国内での推論に限定 |

### 記載内容との差異

デジタル庁の公式ブログ（note）へのアクセスが制限されており、「源内Web」「源内アプリ群」の詳細な構造についての公式情報は確認できませんでした。

ただし、検索結果から推測される構造:
- **源内**: 生成AI利用環境全体の名称
- **複数のアプリ**: 20種類以上の行政実務特化アプリが存在
- **認証基盤**: 職員認証・アクセス制御機能が存在

**記載内容の「源内Web（認証基盤）+ 源内アプリ群（UIレス）」という構造は、検索結果と矛盾しないが、公式ソースでの確認が取れていない状態。**

**Sources**:
- [デジタル庁 源内紹介記事](https://digital-gov.note.jp/n/ndc07326b7491)（アクセス制限あり）
- [デジタル庁ニュース - ガバメントAI](https://digital-agency-news.digital.go.jp/articles/2025-12-11)
- [デジタル庁AI利用実績](https://www.digital.go.jp/assets/contents/node/information/field_ref_resources/08ded405-ca03-48c7-9b92-6b8878854a74/5147384f/20250829_news_ai_usage_report_01.pdf)
- [デジタル庁 PLaMo翻訳導入](https://aismiley.co.jp/ai_news/digital-pfn-plamo-ai/)

---

## 8. GenU（Generative AI Use Cases JP）の構成

### 記載内容の確認

| 項目 | 記載内容 | 検証結果 | 判定 |
|------|---------|----------|------|
| 概要 | AWSが提供する生成AIアプリケーションの基盤 | 正確 | OK |
| GenUの役割 | 第5層のみ（認証+ユースケース登録） | **部分的に不正確** | 要修正 |
| 認証 | Cognito | 正確 | OK |
| フロントエンド | React（CloudFront + S3） | 正確 | OK |
| 第3層の担当 | AWSマネージドサービス（Bedrock等） | 正確 | OK |

### 問題点

**GenUの機能について追記が必要**:

Web検索で確認されたGenUの機能:

| 機能 | 説明 | 第3層該当 |
|------|------|----------|
| **RAG Chat** | Amazon Kendra / Knowledge Base連携 | 第3層にも関与 |
| **Agent機能** | Web Search Agent、Code Interpreter Agent | 第3層にも関与 |
| **Use Case Builder** | プロンプトテンプレートからカスタムユースケース作成 | 第3層にも関与 |
| **Voice Chat** | 双方向音声チャット | 第3層にも関与 |
| **MCP Chat** | 外部MCP サーバー連携 | 第3層にも関与 |

**結論**: GenUは「第5層のみ」ではなく、第3層の一部機能（Agent、RAG、ワークフロー）も持っている。

**推奨修正**:
- 「第5層のみ」→「第5層中心（認証・ユースケース登録）+ 第3層の一部機能（Agent、RAG）」

**Sources**:
- [GenU GitHub](https://github.com/aws-samples/generative-ai-use-cases)
- [GenU クイックスタート](https://aws-samples.github.io/generative-ai-use-cases-jp/ABOUT.html)
- [GenU ドキュメント](https://aws-samples.github.io/generative-ai-use-cases/en/index.html)
- [AWS Summit 2025 GenU資料](https://pages.awscloud.com/rs/112-TZM-766/images/AWS_Summit_2025_O-01A_AWS_Japan_SAS%E7%99%BA%E3%81%AE%E7%94%9F%E6%88%90AI%E3%83%A6%E3%83%BC%E3%82%B9%E3%82%B1%E3%83%BC%E3%82%B9%E5%AE%9F%E8%A3%85.pdf)

---

## 検証結果サマリ

### 問題あり（要修正）

| # | 項目 | ファイル | 問題内容 | 推奨アクション |
|---|------|----------|----------|----------------|
| 1 | Kiro Specs構成 | 07_f_その他製品.md | 「Tests」は別ファイルではない | 3ファイル構成（requirements.md, design.md, tasks.md）に修正 |
| 2 | GenU層構成 | 07_f_その他製品.md | 「第5層のみ」は不正確 | 「第5層中心 + 第3層の一部（Agent、RAG）」に修正 |
| 3 | Perplexity説明 | 07_f_その他製品.md | Sonar API のagentic機能について補足が必要 | agentic機能の存在を追記 |

### 推奨追記（最新情報）

| # | 項目 | ファイル | 追記内容 |
|---|------|----------|----------|
| 4 | Claude Code最新機能 | 07_a_ClaudeCode.md | Claude in Chrome、SDK、Security機能 |
| 5 | Claude Code モデル名 | 07_a_ClaudeCode.md | Sonnet 4.6 / Opus 4.6 への更新 |
| 6 | Claude Cowork Windows | 07_f_その他製品.md | 2026年2月のWindows版リリース |
| 7 | 源内モデル更新 | 07_f_その他製品.md | 利用可能モデル（Nova Lite, Claude 4.5系）を追記 |

### 確認できず（追加調査推奨）

| # | 項目 | 理由 |
|---|------|------|
| 8 | 源内の詳細構造 | デジタル庁noteへのアクセス制限 |

---

## 参照した情報源

### Claude Code
- https://github.com/anthropics/claude-code/blob/main/CHANGELOG.md
- https://code.claude.com/docs/en/overview
- https://claude.com/product/claude-code
- https://www.anthropic.com/news/enabling-claude-code-to-work-more-autonomously

### Cursor
- https://www.nxcode.io/resources/news/cursor-review-2026
- https://techjacksolutions.com/ai/ai-development/cursor-ide-what-it-is/
- https://mindevix.com/ai-usage-strategy/best-cursor-ai-settings-2026/

### Dify
- https://dify.ai/
- https://github.com/langgenius/dify
- https://dify.ai/blog/introducing-dify-plugins

### Kiro
- https://kiro.dev/
- https://kiro.dev/docs/specs/
- https://thenewstack.io/aws-kiro-testing-an-ai-ide-with-a-spec-driven-approach/
- https://www.infoq.com/news/2025/08/aws-kiro-spec-driven-agent/

### Claude Cowork
- https://support.claude.com/en/articles/13345190-getting-started-with-cowork
- https://claude.com/blog/cowork-research-preview
- https://platform.claude.com/docs/en/agents-and-tools/tool-use/computer-use-tool

### Perplexity
- https://www.frugaltesting.com/blog/behind-perplexitys-architecture-how-ai-search-handles-real-time-web-data
- https://blog.bytebytego.com/p/how-perplexity-built-an-ai-google
- https://vespa.ai/perplexity/
- https://docs.perplexity.ai/docs/sonar/quickstart

### 源内
- https://digital-agency-news.digital.go.jp/articles/2025-12-11
- https://aismiley.co.jp/ai_news/digital-pfn-plamo-ai/

### GenU
- https://github.com/aws-samples/generative-ai-use-cases
- https://aws-samples.github.io/generative-ai-use-cases-jp/ABOUT.html

---

*ファクトチェック実施日: 2026-02-22*
