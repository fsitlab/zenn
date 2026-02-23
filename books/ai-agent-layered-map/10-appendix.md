---
title: "付録"
---

# 付録

---

## 付録A ディレクトリ構造比較表

### A-1. 主要AIコーディングエージェントのディレクトリ構造

#### Claude Code

```
プロジェクトルート/
├── CLAUDE.md                      # プロジェクトレベルのプロンプト注入
├── .claude/
│   ├── settings.json              # 権限・モデル・hooks設定
│   ├── settings.local.json        # ローカル環境固有の設定（.gitignore推奨）
│   ├── agents/                    # Subagent定義
│   │   └── <agent-name>.md        # 各エージェントの定義ファイル
│   ├── commands/                  # カスタムスラッシュコマンド
│   │   └── <command-name>.md
│   ├── skills/                    # Skill定義
│   │   └── <skill-name>/
│   │       └── SKILL.md
│   ├── rules/                     # ルールファイル
│   │   └── *.md
│   └── hooks/                     # Hookスクリプト
│       ├── pre-tool-use.sh
│       ├── post-tool-use.sh
│       └── ...
├── .mcp.json                      # MCPサーバー設定
└── .claudeignore                  # 除外パターン

グローバル設定:
~/.claude/
├── CLAUDE.md                      # グローバルプロンプト
├── settings.json                  # グローバル設定
├── rules/
├── agents/
└── skills/
```

#### OpenAI Codex

```
プロジェクトルート/
├── AGENTS.md                      # プロジェクトレベルのプロンプト注入（AGENTS.md標準）
├── AGENTS.override.md             # ローカル上書き設定
├── .agents/
│   └── skills/
│       └── <skill-name>/
│           └── SKILL.md           # Skill定義
└── codex.json                     # プロジェクト設定（推定）

グローバル設定:
~/.codex/
├── AGENTS.md                      # グローバルプロンプト
├── config.json                    # グローバル設定
└── skills/
```

#### Cursor

```
プロジェクトルート/
├── .cursor/
│   ├── rules/                     # プロンプトルール
│   │   └── *.md
│   └── mcp.json                   # MCP設定
├── .cursorignore                  # 除外パターン
└── .cursorrules                   # レガシー形式（非推奨）

グローバル設定:
~/.cursor/
└── mcp.json                       # グローバルMCP設定
```

#### GitHub Copilot

```
プロジェクトルート/
├── .github/
│   └── copilot-instructions.md    # プロジェクトレベルの指示
└── .copilotignore                 # 除外パターン

グローバル設定:
VS Code / JetBrains の設定内で管理
```

### A-2. 機能対応表

| 機能 | Claude Code | OpenAI Codex | Cursor | GitHub Copilot |
|------|-------------|--------------|--------|----------------|
| プロンプト注入 | CLAUDE.md | AGENTS.md | .cursor/rules/ | copilot-instructions.md |
| Skill | .claude/skills/ | .agents/skills/ | - | - |
| Agent/Subagent | .claude/agents/ | - | - | - |
| Hooks | .claude/hooks/ | - | - | - |
| MCP設定 | .mcp.json | - | .cursor/mcp.json | - |
| 権限設定 | settings.json | CLI引数 | - | - |
| 除外パターン | .claudeignore | - | .cursorignore | .copilotignore |
| グローバル設定 | ~/.claude/ | ~/.codex/ | ~/.cursor/ | エディタ設定 |

### A-3. 設定ファイルの優先順位

#### Claude Code

1. `~/.claude/settings.json` (グローバル)
2. `.claude/settings.json` (プロジェクト)
3. `.claude/settings.local.json` (ローカル上書き)
4. CLI引数 (最高優先度)

#### CLAUDE.md / AGENTS.md の読み込み順

```
Claude Code:
1. ~/.claude/CLAUDE.md (グローバル)
2. ~/work/CLAUDE.md (親ディレクトリ、再帰的に探索)
3. ./CLAUDE.md (カレントディレクトリ)
4. ./.claude/rules/*.md (追加ルール)

OpenAI Codex:
1. ~/.codex/AGENTS.md (グローバル)
2. ./AGENTS.md (プロジェクト)
3. ./AGENTS.override.md (ローカル上書き)
```

---

## 付録B 公式ドキュメント一覧（2026年2月時点）

### B-1. AIコーディングエージェント

| サービス | 公式ドキュメント | 備考 |
|----------|------------------|------|
| Claude Code | https://code.claude.com/docs/en/ | Anthropic公式CLI |
| OpenAI Codex | https://developers.openai.com/codex | 2025年リリース |
| Cursor | https://docs.cursor.com/ | VS Code fork |
| GitHub Copilot | https://docs.github.com/copilot | Enterprise対応 |
| Amazon Q Developer | https://docs.aws.amazon.com/amazonq/latest/qdeveloper-ug/ | AWS統合 |
| Goose (Block Inc) | https://block.github.io/goose/ | OSS、MCP対応 |

### B-2. LLM API

| サービス | 公式ドキュメント | 主要エンドポイント |
|----------|------------------|-------------------|
| Anthropic API | https://docs.anthropic.com/ | /v1/messages |
| OpenAI API | https://platform.openai.com/docs | /v1/chat/completions |
| Google Gemini API | https://ai.google.dev/gemini-api/docs | /v1beta/models/:generateContent |
| Amazon Bedrock | https://docs.aws.amazon.com/bedrock/ | Converse API |
| Azure OpenAI | https://learn.microsoft.com/azure/ai-services/openai/ | OpenAI互換 |
| Mistral AI | https://docs.mistral.ai/ | /v1/chat/completions |

### B-3. エージェントフレームワーク・プラットフォーム

| サービス | 公式ドキュメント | カテゴリ |
|----------|------------------|---------|
| Google ADK | https://google.github.io/adk-docs/ | Agent Development Kit |
| Amazon AgentCore | https://aws.amazon.com/bedrock/agentcore/ | クラウドエージェント基盤 |
| LangChain | https://python.langchain.com/docs/ | LLMフレームワーク |
| LangGraph | https://langchain-ai.github.io/langgraph/ | ステートフルワークフロー |
| LlamaIndex | https://docs.llamaindex.ai/ | RAG特化フレームワーク |
| Strands Agents (AWS) | https://strandsagents.com/ | AWS公式エージェントSDK |
| Microsoft AutoGen | https://microsoft.github.io/autogen/ | マルチエージェント |
| CrewAI | https://docs.crewai.com/ | マルチエージェント |
| Dify | https://docs.dify.ai/ | ノーコード/ローコード |
| Flowise | https://docs.flowiseai.com/ | ノーコード |
| n8n | https://docs.n8n.io/ | ワークフロー自動化 |

### B-4. プロトコル・標準

| 標準 | 公式サイト | 管轄 |
|------|-----------|------|
| MCP (Model Context Protocol) | https://modelcontextprotocol.io/ | Anthropic |
| AGENTS.md 標準 | https://agents.md/ | Agentic AI Foundation (Linux Foundation) |
| A2A (Agent-to-Agent) | https://google.github.io/A2A/ | Google |
| OpenAPI | https://www.openapis.org/ | Linux Foundation |

### B-5. OSSエコシステム

| ツール | 公式ドキュメント | 用途 |
|--------|------------------|------|
| HuggingFace Hub | https://huggingface.co/docs/hub/ | モデルリポジトリ |
| HuggingFace Transformers | https://huggingface.co/docs/transformers/ | モデル利用ライブラリ |
| TGI (Text Generation Inference) | https://huggingface.co/docs/text-generation-inference/ | HF公式サービング |
| vLLM | https://docs.vllm.ai/ | 高スループットサービング |
| Ollama | https://ollama.com/docs | ローカル実行 |
| llama.cpp | https://github.com/ggerganov/llama.cpp | C++実装 |
| LiteLLM | https://docs.litellm.ai/ | API統一ラッパー |

### B-6. 評価・モニタリング

| ツール | 公式ドキュメント | 用途 |
|--------|------------------|------|
| LangSmith | https://docs.smith.langchain.com/ | LLMアプリ観測性 |
| Weights & Biases | https://docs.wandb.ai/ | 実験管理 |
| Phoenix (Arize) | https://docs.arize.com/phoenix | LLM観測性 |
| PromptLayer | https://docs.promptlayer.com/ | プロンプト管理 |

---

## 付録C 書籍リスト

### C-1. 主要書籍

| 書籍 | 著者 | 出版社 | 出版年 | 特に参考になる章 |
|------|------|--------|--------|-----------------|
| **AI Engineering: Building Applications with Foundation Models** | Chip Huyen | O'Reilly | 2025 | Agents / RAG / Tool Use / Planning / Evaluation |
| **LLM Engineer's Handbook** | Paul Iusztin, Maxime Labonne | Packt | 2024 | LLMOps / プロダクション化 / デプロイメント |
| **Building LLM Powered Applications** | Valentina Alto | Packt | 2024 | LangChain / メモリ / ツール統合 |
| **Prompt Engineering for LLMs** | John Beurer-Kellner et al. | O'Reilly | 2024 | プロンプト設計原則 / few-shot / CoT |
| **Natural Language Processing with Transformers** | Lewis Tunstall et al. | O'Reilly | 2022 (Revised 2024) | Transformer基礎 / HuggingFace入門 |
| **Designing Machine Learning Systems** | Chip Huyen | O'Reilly | 2022 | MLシステム設計全般（LLM以前の基礎） |

### C-2. Chip Huyen "AI Engineering" の章構成

本記事で特に参照した章:

| 章 | タイトル | 主な内容 |
|----|---------|---------|
| Chapter 1 | Introduction to Foundation Models | FM概論、能力と限界 |
| Chapter 4 | Evaluate AI Systems | 評価指標、ベンチマーク、人間評価 |
| Chapter 5 | Prompt Engineering | プロンプト設計、RAG、構造化出力 |
| Chapter 6 | RAG and Agents | RAGアーキテクチャ、Tool Use、エージェント設計 |
| Chapter 7 | Finetuning | ファインチューニング手法 |
| Chapter 8 | Dataset Engineering | データセット作成、品質管理 |
| Chapter 9 | Inference Optimization | 推論最適化、量子化 |
| Chapter 10 | AI Engineering Architecture and User Feedback | システムアーキテクチャ |

### C-3. オンラインリソース

| リソース | URL | 内容 |
|----------|-----|------|
| Chip Huyen Blog | https://huyenchip.com/ | LLMエンジニアリング解説 |
| Chip Huyen "Agents" (2025/01) | https://huyenchip.com/2025/01/07/agents.html | エージェント設計の詳細解説 |
| Anthropic "Building Effective Agents" | https://www.anthropic.com/research/building-effective-agents | 公式エージェント設計ガイド |
| Phil Schmid "Context Engineering" | https://www.philschmid.de/context-engineering | コンテキスト工学の図解 |
| Simon Willison's Blog | https://simonwillison.net/ | LLM・AI関連の技術解説 |
| Hamel Husain Blog | https://hamel.dev/blog/ | LLMアプリ開発実践 |

### C-4. 日本語リソース

| リソース | URL | 内容 |
|----------|-----|------|
| AI Shift Advent Calendar 2024 | https://www.ai-shift.co.jp/techblog/5252 | AIエージェント設計の勘所 |
| Zenn「マルチLLMエージェントシステム」 | https://zenn.dev/r_kaga/articles/ebeba0bd1385e1 | MLAS 4分類 |
| IDCF「プロンプトエンジニアリング完全ガイド」 | https://blog.idcf.jp/entry/PromptEngineering | 基礎から応用まで |
| Qiita「AIエージェント入門」シリーズ | https://qiita.com/syukan3 | アーキテクチャ解説 |
| AQUA「AIエージェント開発フレームワーク徹底比較」 | https://www.aquallc.jp/2025/12/18/ai-agent-frameworks/ | LangChain/CrewAI/AutoGen/LlamaIndex比較 |
| GMO「フレームワーク選定ガイド」 | https://recruit.group.gmo/engineer/jisedai/blog/agents-sdk/ | 各SDKの特徴と選定基準 |
| Zenn「AI Agent認可の3層アーキテクチャ入門」 | https://zenn.dev/exwzd/articles/20251224_aiagent_authz_architecture | 認可に特化した3層モデル |

### C-5. 層モデル・アーキテクチャ解説（英語）

| リソース | URL | 内容 |
|----------|-----|------|
| Boomi: Breaking Down the Agentic Layers of AI | https://boomi.com/blog/agentic-layers-of-ai-integration/ | 6層モデルによるAIエージェント整理 |
| Pylar: 5 Layers Architecture | https://www.pylar.ai/blog/5-layer-architecture-connecting-agents-databases | データアクセス5層モデル |
| Deloitte: AI Agent Orchestration | https://www.deloitte.com/us/en/insights/industry/technology/technology-media-and-telecom-predictions/2026/ai-agent-orchestration.html | エンタープライズ向けオーケストレーション |
| Microsoft: AI Agent Orchestration Patterns | https://learn.microsoft.com/en-us/azure/architecture/ai-ml/guide/ai-agent-design-patterns | Azure向けパターン集 |
| IBM: AI Agent Orchestration | https://www.ibm.com/think/topics/ai-agent-orchestration | オーケストレーション概念解説 |
| Algolia: Agentic Architecture for Enterprise | https://www.algolia.com/blog/ai/agentic-architecture | エンタープライズ向けエージェント設計 |

### C-6. プロトコル比較（MCP/A2A/ACP）

| リソース | URL | 内容 |
|----------|-----|------|
| OneReach.ai: MCP vs A2A | https://onereach.ai/blog/guide-choosing-mcp-vs-a2a-protocols/ | プロトコル選択ガイド |
| Cisco: MCP and A2A Mental Model | https://blogs.cisco.com/ai/mcp-and-a2a-a-network-engineers-mental-model | ネットワーク観点でのメンタルモデル |
| Auth0: MCP vs A2A | https://auth0.com/blog/mcp-vs-a2a/ | 認証観点でのプロトコル比較 |
| InfoQ: Agentic MLOps with A2A and MCP | https://www.infoq.com/articles/architecting-agentic-mlops-a2a-mcp/ | MLOpsでのプロトコル活用 |
| Niklas Heidloff: MCP, ACP, A2A Comparison | https://heidloff.net/article/mcp-acp-a2a-agent-protocols/ | 3プロトコル技術比較 |
| A2A Protocol: A2A and MCP | https://a2a-protocol.org/latest/topics/a2a-and-mcp/ | 公式のA2A/MCP関係説明 |
| Clarifai: MCP vs A2A Clearly Explained | https://www.clarifai.com/blog/mcp-vs-a2a-clearly-explained | 明快なプロトコル比較 |
| Stream: Top AI Agent Protocols 2026 | https://getstream.io/blog/ai-agent-protocols/ | MCP/A2A/ACP等の包括的解説 |

### C-7. フレームワーク比較

| リソース | URL | 内容 |
|----------|-----|------|
| DataCamp: CrewAI vs LangGraph vs AutoGen | https://www.datacamp.com/tutorial/crewai-vs-langgraph-vs-autogen | 主要フレームワーク比較 |
| Turing: Top AI Agent Frameworks 2026 | https://www.turing.com/resources/ai-agent-frameworks | 6フレームワーク詳細比較 |
| Langflow: Guide to AI Agent Frameworks 2025 | https://www.langflow.org/blog/the-complete-guide-to-choosing-an-ai-agent-framework-in-2025 | 選定ガイド |
| Shakudo: Top 9 AI Agent Frameworks | https://www.shakudo.io/blog/top-9-ai-agent-frameworks | フレームワーク一覧 |
| Chatbase: LLM Agent Framework Guide | https://www.chatbase.co/blog/llm-agent-framework-guide | 2026年版ガイド |

### C-8. 製品比較（Claude Code/Cursor/Dify等）

| リソース | URL | 内容 |
|----------|-----|------|
| Qodo: Claude Code vs Cursor Deep Comparison | https://www.qodo.ai/blog/claude-code-vs-cursor/ | 開発チーム向け詳細比較 |
| Builder.io: Cursor vs Claude Code | https://www.builder.io/blog/cursor-vs-claude-code | 2026年版比較 |
| DoltHub: Comparing Cursor Agent to Claude Code | https://www.dolthub.com/blog/2025-08-15-cursor-agent-vs-claude-code/ | 実践的比較 |
| Skywork: Dify Review 2025 | https://skywork.ai/blog/dify-review-2025-workflows-agents-rag-ai-apps/ | Dify機能レビュー |
| GPTBots: Dify AI Review 2026 | https://www.gptbots.ai/blog/dify-ai | 機能・代替製品解説 |
| Render: Testing AI Coding Agents 2025 | https://render.com/blog/ai-coding-agents-benchmark | コーディングエージェントベンチマーク |

### C-9. 日本語書籍（2024-2025年）

| 書籍 | 著者 | 出版社 | 出版年 | 内容 |
|------|------|--------|--------|------|
| AIエージェント開発／運用入門 | 御田稔, 大坪悠, 塚田真規 | SBクリエイティブ | 2025 | ReAct/Function Calling/フレームワーク解説 |
| AIエージェント革命 | シグマクシス | 日経BP | 2025 | ビジネス×技術の体系的解説 |
| AIエージェント白書2025 | CMCリサーチ | CMC出版 | 2025 | エージェントタイプ分類・市場動向 |
| やさしく学ぶLLMエージェント | 井上顧基, 下垣内隆太, 松山純大 | オーム社 | 2025 | 基礎からマルチエージェント構築まで |
| LangChainとLangGraphによるRAG・AIエージェント入門 | 西見公宏, 吉田真吾, 大嶋勇樹 | 技術評論社 | 2024 | LangChain/LangGraph実践 |
| Azure OpenAIエージェント・RAG構築実践ガイド | 菅原允他 | 日経BP | 2025 | Azure向け実装ガイド |
| 実践AIエージェントの教科書 | 日立製作所 AI CoE | リックテレコム | 2025 | 産業別導入事例 |
| AIエージェント | EY Japan | 東洋経済新報社 | 2025 | 入門書 |

---

## 付録D 未解決・要調査事項

### D-1. API・プロトコル関連

- [ ] **Gemini API の tool_choice 詳細**: `tool_config`の`mode`オプションの全一覧と挙動の違い
- [ ] **OpenAI Responses API**: 2025年リリースのResponses APIとChat Completions APIの違い
- [ ] **A2A プロトコルの採用状況**: Google以外のベンダーでの実装状況
- [ ] **MCP Streamable HTTP**: SSE vs WebSocket vs Streamable HTTPの実装差

### D-2. ツール・フレームワーク関連

- [ ] **OpenCode**: OSSのClaudeCode代替のディレクトリ構造と機能比較
- [ ] **Aider**: Claude Code / Codex との機能比較
- [ ] **Claude Code SubagentとAgentCore Runtime連携**: 実際の制限と構成パターン
- [ ] **Dify / LangChain のTool Use抽象化**: 各APIをどう内部変換しているか

### D-3. 権限・セキュリティ関連

- [ ] **Gooseのpermission設計**: Claude Code Hooksとの詳細比較
- [ ] **MCP Remote の OAuth実装状況**: 主要MCPサーバーでの認証対応
- [ ] **AgentCore Cedar policy**: サンプルポリシーと設計パターン

### D-4. アーキテクチャ関連

- [ ] **Orchestrator-Workers vs Mesh**: 実際のユースケースでのパフォーマンス比較
- [ ] **長時間実行エージェント**: セッション管理とチェックポイントの設計パターン
- [ ] **マルチモデルエージェント**: 同一タスク内で複数モデルを使い分ける設計

### D-5. 評価・運用関連

- [ ] **エージェント評価基準**: 信頼性・安全性の定量評価手法
- [ ] **コスト最適化**: Subagent分割によるトークン削減の実測値
- [ ] **本番運用でのHooksパターン**: 実際のPreToolUseスクリプト例

### D-6. 将来動向

- [ ] **Claude 4 / GPT-5 世代のTool Use**: 新しいAPIフォーマットの可能性
- [ ] **Computer Use / Browser Use**: GUIエージェントの標準化
- [ ] **エージェント間相互運用性**: 異なるフレームワーク間でのエージェント呼び出し

---

## 付録E 用語集（クイックリファレンス）

| 用語 | 定義 | 属する層 |
|------|------|---------|
| **tool_use** | LLMが出力するツール呼び出し宣言（JSON） | 第2層OUT（LLM起点） |
| **tool_result** | ホストがツール実行後に返す結果 | 第2層IN（クライアント起点） |
| **MCP** | ツール定義を標準化するプロトコル | 第3層↔第4層 |
| **MCPサーバー** | ツールを実装・公開するプログラム | 第4層（外部ツール層） |
| **MCPクライアント** | サーバーからtools[]を取得しLLMへ渡す | 第3層（LLMオーケストレーション層） |
| **Hooks** | ツール実行前後に確実に実行されるスクリプト | 第3層（LLMオーケストレーション層） |
| **RAG** | 外部知識をプロンプトに注入する技術 | 第2層IN / 第4層 |
| **CLAUDE.md** | セッション開始時に自動注入されるMarkdown | 第2層IN（クライアント起点） |
| **Skill** | 使用時のみロードされるプロンプトモジュール | 第2層IN（クライアント起点） |
| **Subagent** | 親エージェントから起動される子エージェント | 第4層（外部ツール層） |
| **A2A** | エージェント間通信プロトコル（Google） | 第4層（外部ツール層） |

---

## 付録F 変更履歴

| 日付 | バージョン | 変更内容 |
|------|-----------|---------|
| 2026-02-22 | 1.0 | 初版作成 |

---

*調査日: 2026-02-22*
*主な参照: Claude Code公式docs / OpenAI Codex公式 / Amazon AgentCore公式 / Chip Huyen "AI Engineering" / Anthropic "Building Effective Agents"*
