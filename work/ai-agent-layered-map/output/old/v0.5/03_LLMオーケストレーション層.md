# 第3層「LLMオーケストレーション層」詳細解説

> **5層モデルでの位置**: 第3層（LLMオーケストレーション層）
> **概要**: **直接LLMとやり取りする層**。一般的なアプリケーション層ではなく、LLMを操作するための専門層。

---

## LLMオーケストレーション層の定義

**重要**: この層は一般的なアプリケーション層とは異なり、**直接LLMとやり取りする**ことを目的とした専門層である。

### 第3層に含まれるもの

| コンポーネント | 説明 |
|--------------|------|
| **ワークフロー** | LLMを操作するための処理フロー |
| **Hooks** | LLM操作にフィードバックを与える仕組み（PreToolUse等） |
| **MCPクライアント** | MCPサーバーからtools[]を取得しLLMへ渡す |
| **エージェントランタイム** | LLM呼び出しループを管理する実行環境 |

### 第3層に含まれないもの

| コンポーネント | 属する層 | 備考 |
|--------------|---------|------|
| ユーザ認証・認可（基本部分） | 第5層 | ただし、権限制御でLLM操作に波及する場合は第3層 |
| UI（チャット画面、設定画面等） | 第5層 | - |
| 監視・ログ分析 | 第5層 | - |
| ユーザ管理 | 第5層 | - |

---

## LLMオーケストレーション層の役割

**概要**: LLMオーケストレーション層は**直接LLMを操作するランタイム**であり、第2層（通信層）を通じてLLMとやりとりし、第4層（外部ツール層）を実行する責任を持つ。

```
第5層：UI・運用層            ユーザーとの接点（UI）+ LLMに直接関係しない周辺機能
                              例: VSCode UI / Claude.ai / 監視 / ログ分析 / ユーザー管理

第4層：外部ツール層          ベクトルDB、MCPサーバー、サブエージェント

第3層：LLMオーケストレーション層  ← 今回の解説対象
                              ワークフロー / Hooks / MCPクライアント / エージェントランタイム

第2層：通信層                A: クライアント起点 / B: LLM起点

第1層：LLM層                 モデル、パラメータ
```

**LLMオーケストレーション層の責務**:
- ワークフローの定義と実行（LLMを操作）
- Hooksによる決定論的制御（LLM操作にフィードバック）
- MCPクライアントによるツール定義の取得・変換
- エージェントループの管理（複数のLLM呼び出しのオーケストレーション）

---

## 系列ごとの整理

### プログラム系：LLMフレームワーク

プログラムコードでLLMアプリケーションを構築するためのライブラリ・フレームワーク。

| ツール | 特徴 | 主な用途 |
|--|--|--|
| **LangChain** | Python/JSの汎用フレームワーク | RAG、Agent、Chain構築 |
| **LlamaIndex** | RAG特化、データ接続に強み | 文書検索、ナレッジベース |
| **LangGraph** | グラフベースのワークフロー定義 | 複雑な分岐・ループ処理 |
| **Strands Agents (AWS)** | AWS Bedrock連携 | エンタープライズAgent |

**層の位置づけ**: LLMオーケストレーション層（第3層）として構築し、内部でLLM APIを直接呼び出す。

- 📘 [LangChain ドキュメント](https://python.langchain.com/docs/)
- 📘 [LlamaIndex ドキュメント](https://docs.llamaindex.ai/)
- 📘 [LangGraph ドキュメント](https://langchain-ai.github.io/langgraph/)
- 📘 [Strands Agents（AWS）](https://strandsagents.com/)

---

### ローコード系：ノーコード/ローコードプラットフォーム

GUIでワークフローを構築し、LLMアプリケーションを開発するプラットフォーム。

| ツール | 特徴 | 主な用途 |
|--|--|--|
| **Dify** | オープンソース、RAG・Agent対応 | 社内チャットボット、RAGアプリ |
| **n8n** | ワークフロー自動化、LLM統合 | 業務自動化、API連携 |
| **Flowise** | LangChainベースのGUI | プロトタイピング |

**層の位置づけ**: 第3層（LLMオーケストレーション層）と第5層（UI・運用層）の両方を持つ。GUIは第5層、ワークフローエンジンは第3層。

- 📘 [Dify ドキュメント](https://docs.dify.ai/)
- 📘 [n8n ドキュメント](https://docs.n8n.io/)
- 📘 [Flowise ドキュメント](https://docs.flowiseai.com/)

---

### コンシューマアプリ：エンドユーザー向けサービス

一般ユーザーが直接利用するLLMアプリケーション。

| ツール | 提供元 | 特徴 |
|--|--|--|
| **ChatGPT** | OpenAI | GPTシリーズのWebインターフェース |
| **Claude.ai** | Anthropic | Claudeのコンシューマ向けUI |
| **Gemini** | Google | Geminiモデルのコンシューマ向けUI |

**層の位置づけ**: 第3層（LLMオーケストレーション層）と第5層（UI・運用層）を統合。UIは第5層、内部のLLM操作は第3層。

- 📘 [ChatGPT](https://chat.openai.com/)
- 📘 [Claude.ai](https://claude.ai/)
- 📘 [Gemini](https://gemini.google.com/)

---

### コーディングエージェント：開発者向けAIアシスタント

ソフトウェア開発を支援するAIエージェント。IDEやCLIで動作。

| ツール | 提供元 | 特徴 |
|--|--|--|
| **Claude Code** | Anthropic | CLI/IDE統合、Hooks・Skill対応 |
| **Cursor** | Cursor | VS Code fork、AIネイティブIDE |
| **Goose** | Block Inc | オープンソース、プラグイン拡張 |
| **GitHub Copilot** | GitHub/Microsoft | IDE統合、コード補完特化 |
| **OpenAI Codex** | OpenAI | CLI、AGENTS.md対応 |

**層の位置づけ**: 第5層（ターミナルUI）+ 第3層（LLMオーケストレーション層：Hooks、Permissions、MCPクライアント）+ 第2層A（CLAUDE.md、Skill）を統合。

- 📘 [Claude Code ドキュメント](https://code.claude.com/docs/en/)
- 📘 [Cursor ドキュメント](https://docs.cursor.com/)
- 📘 [Goose ドキュメント](https://block.github.io/goose/)
- 📘 [GitHub Copilot](https://docs.github.com/en/copilot)
- 📘 [OpenAI Codex](https://developers.openai.com/codex)

---

### クラウド基盤：エンタープライズ向けAgent基盤

クラウドスケールでエージェントを実行・管理するプラットフォーム。

| サービス | 提供元 | 特徴 |
|--|--|--|
| **AWS Strands Agents** | AWS | オープンソースフレームワーク |
| **Amazon AgentCore** | AWS | マネージドAgent実行環境、Cedar policy |
| **Google ADK** | Google | Agent Development Kit、A2A対応 |
| **Azure AI Agent Service** | Microsoft | Azure統合のAgent基盤 |

**層の位置づけ**: 第3層（LLMオーケストレーション層）のプラットフォームとして、LLM操作のランタイム（権限管理、実行環境）を提供。

- 📘 [Amazon Bedrock AgentCore](https://aws.amazon.com/bedrock/agentcore/)
- 📘 [Strands Agents](https://strandsagents.com/)
- 📘 [Google ADK](https://google.github.io/adk-docs/)

---

### 検索系：AI検索エンジン

LLMを活用した検索サービス。内部でRAG的な処理を行い、検索結果を生成。

| サービス | 特徴 |
|--|--|
| **Perplexity** | リアルタイムWeb検索+LLM回答生成 |
| **You.com** | AI検索、ソース引用付き回答 |
| **Phind** | 開発者向けAI検索 |

**層の位置づけ**: 第3層（LLMオーケストレーション層）でLLMを操作し、第5層（UI・運用層）でUIを提供。内部でベクトル検索・Web検索を実行し、結果をLLMに渡して回答を生成。

- 📘 [Perplexity](https://www.perplexity.ai/)

---

## 従来型RAGの位置づけ

**概要**: 従来型RAGは第3層（LLMオーケストレーション層）が第4層（ベクトルDB）を使い、結果を第2層A（プロンプト注入）に渡す構造。LLMは検索の存在を知らない。

```
従来型RAG の処理フロー:

[第3層: LLMオーケストレーション層]
    ↓ ユーザー入力を受け取る
[第4層: ベクトルDB検索実行]
    ↓ Embedding Modelでベクトル化
    ↓ ベクトルDBでANN検索
    ↓ 上位k件のテキストを取得
[第2層A: プロンプト注入]
    ↓ 検索結果をsystem promptに挿入
[第1層: LLMへ送信]
    ↓ LLMは「文書が来た理由」を知らない
```

**Tool型RAGとの違い**:

| | 従来型RAG | Tool型RAG |
|--|--|--|
| 検索の主体 | 第3層（ホストプログラム） | LLM（第1層でtool_use宣言） |
| LLMの認識 | 文書が来た理由を知らない | 自分で検索ツールを呼んだと認識 |
| クエリ生成 | ユーザー入力のコピー | LLMが最適なクエリを生成 |
| 反復 | 1回のみ | 結果を見て再検索可能 |

**設計の転換点**: 「検索の主体がホストかLLMか」が分岐点。複合クエリや反復検索が必要な場合はTool型RAGが適切。

- 📘 [Anthropic RAG ガイド](https://docs.anthropic.com/en/docs/build-with-claude/retrieval-augmented-generation)
- 📘 [LlamaIndex RAG ドキュメント](https://docs.llamaindex.ai/)
- 📘 [LangChain RAG ドキュメント](https://python.langchain.com/docs/tutorials/rag/)

---

## ワークフローの役割

**概要**: ワークフローは第3層（LLMオーケストレーション層）で定義される「LLM呼び出しの順序と条件分岐」。単一のLLM呼び出しではなく、複数の処理を組み合わせて複雑なタスクを実行する。

```
ワークフローの構成要素:

[入力] → [前処理] → [LLM呼び出し1] → [条件分岐]
                                          ↓
                     [LLM呼び出し2a] または [LLM呼び出し2b]
                                          ↓
                                      [後処理] → [出力]
```

**ワークフローの実装パターン**:

| パターン | 説明 | 実装例 |
|--|--|--|
| **Prompt Chaining** | 出力を次の入力に渡す | 要約→翻訳→校正 |
| **Routing** | 入力に応じて分岐 | 質問分類→専門Agent |
| **Parallelization** | 複数処理を並列実行 | 複数ソース同時検索 |
| **Orchestrator-Workers** | 親が子を呼び出し | メインAgent→SubAgent |

- 🔗 [Anthropic "Building Effective Agents"](https://www.anthropic.com/research/building-effective-agents) — パターンの公式解説
- 📘 [LangGraph ドキュメント](https://langchain-ai.github.io/langgraph/) — グラフベースのワークフロー定義

---

## CLAUDE.md、Skill、Hooksの位置づけ

**概要**: CLAUDE.md/Skillは第2層A（プロンプト注入）、Hooksは第3層（決定論的制御）。LLMオーケストレーション層（第3層）はこれらを読み込み・実行する責任を持つ。

```
第3層（LLMオーケストレーション層: Claude Code等）
  ├── 読み込み: CLAUDE.md（第2層A）→ system promptに注入
  ├── 読み込み: Skill（第2層A）→ 必要時にロード
  └── 実行: Hooks（第3層）→ PreToolUse等のイベントでシェルスクリプト実行
```

### CLAUDE.md / AGENTS.md（第2層A）

セッション開始時にsystem promptへ自動注入されるMarkdown。

| ツール | コンテキストファイル |
|--|--|
| Claude Code | `CLAUDE.md` / `.claude/` |
| OpenAI Codex | `AGENTS.md` |
| Cursor | `.cursor/rules/` |
| GitHub Copilot | `.github/copilot-instructions.md` |

- 📘 [Claude Code Memory管理](https://code.claude.com/docs/en/memory)
- 📘 [AGENTS.md 標準](https://agents.md/)

### Skill（第2層A）

使用時だけロードされるプロンプトのモジュール。起動時は`name`と`description`だけ読み、タスクが来たとき初めて全体をロード（Progressive Disclosure）。

```
.claude/skills/<skill-name>/SKILL.md
```

- 📘 [Claude Code Skills ドキュメント](https://code.claude.com/docs/en/skills)

### Hooks（第3層）

LLMの確率的判断を介さず、シェルスクリプトを確実に実行する決定論的制御。

| イベント | タイミング | ブロック可否 |
|--|--|--|
| `PreToolUse` | tool_use受け取り直後、実行前 | exit 2でブロック可能 |
| `PostToolUse` | ツール実行後 | ブロック不可 |
| `PostToolUseFailure` | ツール実行失敗後 | ブロック不可 |
| `SessionStart` | セッション開始時 | ブロック不可 |
| `SessionEnd` | セッション終了時 | ブロック不可 |
| `SubagentStart` | Subagent起動時 | ブロック不可（コンテキスト注入は可能） |
| `SubagentStop` | Subagent終了時 | ブロック不可 |
| `Stop` | エージェント停止時 | ブロック不可 |
| `Notification` | 通知発生時 | ブロック不可 |
| `UserPromptSubmit` | ユーザー入力送信時 | exit 2でブロック可能 |
| `PermissionRequest` | 権限リクエスト時 | ブロック不可 |
| `PreCompact` | コンテキスト圧縮前 | ブロック不可 |

**PreToolUseの追加機能**:
- v2.0.10以降: ツール入力の修正が可能
- v2.1.9以降: `additionalContext`でモデルに追加コンテキストを提供可能

- 📘 [Claude Code Hooks ドキュメント](https://code.claude.com/docs/en/hooks)

---

## ツール利用の中継機能

**概要**: LLMオーケストレーション層（第3層）は、ユーザーの要求を受けてツール（第4層）を呼び出し、その結果をLLM（第1層）に渡す中継役を担う。MCPサーバーへの接続もこの中継機能の一部。

```
[ユーザー] → [第3層: LLMオーケストレーション] → [第3層: MCPクライアント] → [第4層: MCPサーバー]
                                                        ↓
                                               tools[]に変換してLLMへ
                                                        ↓
                                               [第1層: tool_use/tool_result往復]
```

**中継機能の構成要素**:

| コンポーネント | 層 | 役割 |
|--|--|--|
| MCPクライアント | 第3層 | サーバーからtools[]を取得し、LLMへ渡す |
| MCPサーバー | 第4層 | ツールを実装・公開 |
| `.mcp.json` | 第3層設定 | どのサーバーを起動するか記述 |
| エージェントランタイム（Claude Code等） | 第3層 | MCPクライアントを内包し、LLM操作を統合 |

**各ツールのMCP設定ファイル**:

| ツール | 設定ファイル |
|--|--|
| Claude Code | `.mcp.json` (プロジェクト) / `~/.claude/` |
| Cursor | `.cursor/mcp.json` |
| Amazon Q CLI | `~/.aws/amazonq/mcp.json` |

- 📘 [MCP 公式サイト](https://modelcontextprotocol.io/)
- 📘 [Claude Code MCP設定](https://code.claude.com/docs/en/mcp)

---

## まとめ：LLMオーケストレーション層の判定基準

新しい技術・ツールが第3層（LLMオーケストレーション層）に属するかどうかは、以下の基準で判定できる。

```
Q1. 直接LLMとやり取りする機能か？
Q2. LLMの呼び出しをオーケストレート（ワークフロー、Hooks等）するか？
    → YES: 第3層（LLMオーケストレーション層）

Q3. UIのみ、またはLLMに直接関係しない周辺機能か？
    → YES: 第5層（UI・運用層）
```

**LLMオーケストレーション層（第3層）に含まれるもの**:
- ワークフロー（LLMを操作）
- Hooks（LLM操作にフィードバック）
- MCPクライアント
- エージェントランタイム

**UI・運用層（第5層）に含まれるもの**:
- UI（チャット画面、設定画面等）
- ユーザ認証・認可（基本部分）※権限制御でLLM操作に波及する場合は第3層
- 監視・ログ分析
- ユーザ管理

---

## 関連リンク

### 公式ドキュメント
- 📘 [Claude Code](https://code.claude.com/docs/en/)
- 📘 [LangChain](https://python.langchain.com/docs/)
- 📘 [LlamaIndex](https://docs.llamaindex.ai/)
- 📘 [Dify](https://docs.dify.ai/)
- 📘 [Strands Agents](https://strandsagents.com/)
- 📘 [Amazon AgentCore](https://aws.amazon.com/bedrock/agentcore/)

### 解説ブログ
- 🔗 [Anthropic "Building Effective Agents"](https://www.anthropic.com/research/building-effective-agents)
- 🔗 [Chip Huyen "Agents"](https://huyenchip.com/2025/01/07/agents.html)
- 🔗 [Phil Schmid "Context Engineering"](https://www.philschmid.de/context-engineering)

---

*作成日: 2026-02-22*
*シリーズ: AIエージェント概念マップ 詳細解説*
*対象: 第3層（LLMオーケストレーション層）*
