# AIエージェント概念マップ完全版：JSONからAgentsまで一気通貫で理解する
## ～「相性問題」はなぜ起きるのか、プロトコルと責任境界から読み解く～

> **この記事の目的**
> LLMへの入出力JSONから出発し、RAG・Tool Use・MCP・CLAUDE.md・Skill・Hooks・Agents・Agent Teamsまで、
> 各概念が**どの層に属し**、**何がLLMの仕事で何がプログラムの仕事か**を包含関係として整理する地図。

*骨子バージョン: 2026-02-21 rev2 / 主軸: Claude / Claude Code*

---

## 目次

```
第0章  全体地図：3層モデル
第1章  LLMへの入出力：JSON通信プロトコルの実態
第2章  Tool Use：LLMが「実行しない」という大原則
第3章  MCP：ツール定義を標準化・外部化するプロトコル
第4章  RAG：ツールではなくプロンプト操作だった
第5章  CLAUDE.md / AGENTS.md：プロンプト注入の設定ファイル
第6章  Skill：動的プロンプト拡張と「コンテキスト節約」
第7章  Hooks：プログラムが担保する決定論的制御
第8章  Subagent / Agents：Tool Useの応用として別セッションを起動する
第9章  Agent Teams：エージェント間通信プロトコルの話
第10章  プラグイン：上記すべての配布単位
第11章  権限管理：②層の設計比較
第12章  全体俯瞰：「相性問題」はどこで起きるか
```

---

## 第0章　全体地図：3層モデル

「相性問題」の正体：**プロトコルのフォーマット差（①層）** または **責任境界の認識違い（①②③の混同）**

```
① LLM入出力層   JSONで入出力するだけ。実行しない。
② ホスト制御層  プログラムが担保。Hooks/Permissions/ライブラリ。
③ プロンプト層  system promptへの注入。Skills/CLAUDE.md/RAG。
```

---

### ① LLM入出力層

**一言**: LLMへのリクエストJSONに含まれる推論パラメータ。モデルの「確率的振る舞い」をここで制御する。

| パラメータ | 概要 |
|--|--|
| `temperature` | 出力のランダム性（0=決定的、1=多様） |
| `top_p` | 上位p%の確率トークンのみサンプリング |
| `top_k` | 上位k個のトークンのみサンプリング（Gemini等） |
| `max_tokens` | 最大出力トークン数 |
| `stop` | 生成停止シーケンス |

- 📘 [Anthropic API パラメータ一覧](https://docs.anthropic.com/en/api/messages)
- 📘 [OpenAI API パラメータ一覧](https://platform.openai.com/docs/api-reference/chat/create)
- 📘 [Google Gemini generationConfig](https://ai.google.dev/api/generate-content#v1beta.GenerationConfig)

---

### ② ホスト制御層

**一言**: LLMを呼び出すプログラム側。フレームワーク・エージェントランタイム・ライブラリがここに属する。

| カテゴリ | 代表例 |
|--|--|
| AIコーディングエージェント | Claude Code、OpenAI Codex、Cursor、Goose |
| LLMフレームワーク | LangChain、LlamaIndex、Strands Agents（AWS） |
| ノーコード/ローコード | Dify、n8n、Flowise |
| クラウドエージェント基盤 | Amazon AgentCore、Google ADK |

- 📘 [LangChain ドキュメント](https://python.langchain.com/docs/)
- 📘 [Strands Agents（AWS）](https://strandsagents.com/)
- 📘 [Dify ドキュメント](https://docs.dify.ai/)

---

### ③ プロンプト層

**一言**: LLMへの入力テキストを構成・操作する層。コードではなく「言葉」で動作を制御する。

| 技術 | 概要 |
|--|--|
| プロンプトエンジニアリング | system prompt・few-shot・Chain of Thought等でLLMの出力を誘導 |
| RAG | 外部知識をpromptに注入してLLMの知識を拡張 |
| CLAUDE.md / AGENTS.md | セッション起動時にsystem promptへ自動注入するMarkdown |
| Skill | タスク発生時にのみロードされるプロンプトモジュール |

- 📘 [Anthropic Prompt Engineering ガイド](https://docs.anthropic.com/en/docs/build-with-claude/prompt-engineering/overview)
- 🔗 [Chip Huyen "Building LLM applications for production"](https://huyenchip.com/2023/04/11/llm-engineering.html) — プロンプトの非決定論的性質との向き合い方
- 🔗 [Phil Schmid "Context Engineering" (図解)](https://www.philschmid.de/context-engineering)

---

## 第1章　LLMへの入出力：JSON通信プロトコルの実態

> **対象層**: ① LLM入出力層

**一言**: LLMはステートレスなHTTPサービス。`messages[]`配列を毎回全部送るだけ。ベンダー間でフォーマットが微妙に異なることが「相性問題①」の根本原因。

---

### 1-1. 主要3社のAPIフォーマット（本記事掲載）

#### Anthropic Messages API

```json
POST https://api.anthropic.com/v1/messages
{
  "model": "claude-sonnet-4-6",
  "max_tokens": 1024,
  "system": "You are a helpful assistant.",
  "messages": [
    {"role": "user", "content": "Hello"},
    {"role": "assistant", "content": "Hi!"},
    {"role": "user", "content": "What is 2+2?"}
  ]
}
```

**設計思想**: `system`が独立フィールド。`content[]`はブロック配列（text/image/tool_use/tool_result）。ツール結果は`role:user`の`content[]`内に`type:"tool_result"`で返す。**型安全・マルチモーダル統一**を優先。

- 📘 [Anthropic Messages API リファレンス](https://docs.anthropic.com/en/api/messages)

#### OpenAI Chat Completions API

```json
POST https://api.openai.com/v1/chat/completions
{
  "model": "gpt-4o",
  "messages": [
    {"role": "system", "content": "You are a helpful assistant."},
    {"role": "user", "content": "Hello"},
    {"role": "assistant", "content": "Hi!"},
    {"role": "tool", "tool_call_id": "call_abc", "content": "result..."}
  ]
}
```

**設計思想**: `system`は`messages[]`の1要素。ツール結果は`role:"tool"`という専用ロール。ツール引数は**文字列化JSON**（`"arguments": "{\"query\":\"...\"}"`）。**汎用性・後方互換**を優先。

- 📘 [OpenAI Chat Completions API リファレンス](https://platform.openai.com/docs/api-reference/chat)

#### Google Gemini API

```json
POST https://generativelanguage.googleapis.com/v1beta/models/gemini-2.0-flash:generateContent
{
  "systemInstruction": {"parts": [{"text": "You are a helpful assistant."}]},
  "contents": [
    {"role": "user", "parts": [{"text": "Hello"}]},
    {"role": "model", "parts": [{"text": "Hi!"}]}
  ],
  "generationConfig": {"temperature": 0.7, "maxOutputTokens": 1024}
}
```

**設計思想**: `contents`に`parts[]`配列でマルチモーダルを統一。アシスタントロールが`"assistant"`でなく`"model"`。ツール定義は`functionDeclarations`。`generationConfig`が独立オブジェクト。**Google Cloud ecosystem統合・マルチモーダル**を優先。

- 📘 [Google Gemini API - generateContent](https://ai.google.dev/api/generate-content)
- 📘 [Gemini API - Function Calling](https://ai.google.dev/gemini-api/docs/function-calling)
- 🔗 [Gemini API クイックスタート（日本語）](https://ai.google.dev/gemini-api/docs/quickstart?hl=ja)

---

### 1-2. フォーマット差分比較

| 項目 | Anthropic | OpenAI | Gemini |
|--|--|--|--|
| systemの場所 | 独立フィールド | messages[]の要素 | systemInstruction独立 |
| アシスタントロール | `"assistant"` | `"assistant"` | `"model"` |
| ツール引数 | オブジェクト（`input: {...}`） | **文字列化JSON**（`arguments: "..."` ） | オブジェクト |
| ツール結果のrole | `"user"` + `type:"tool_result"` | `"tool"` | `"user"` + `functionResponse` |
| ツール定義キー | `input_schema` | `function.parameters` | `functionDeclarations` |

- 🔗 [Structured Output Comparison: OpenAI / Gemini / Anthropic / Mistral / Bedrock](https://medium.com/@rosgluk/structured-output-comparison-across-popular-llm-providers-openai-gemini-anthropic-mistral-and-1a5d42fa612a) — コード例付き詳細比較
- 🔗 [AI API Comparison 2024: Anthropic vs Google vs OpenAI (big-AGI)](https://big-agi.com/blog/ai-api-comparison-2024-anthropic-vs-google-vs-openai) — 機能横断の比較表
- 🔗 [LLM API Tool Comparison: OpenAI vs Claude vs Grok vs Gemini](https://apimagic.ai/blog/llm-tool-comparison) — tool定義JSONの差分詳細

---

### 1-3. マルチモーダルAPIの形式

**一言**: テキスト以外の入力（画像・音声・動画）は`content[]`または`parts[]`内にブロックとして埋め込む。

```json
// Anthropic: content[]内にimageブロック
{"role": "user", "content": [
  {"type": "image", "source": {"type": "base64", "media_type": "image/jpeg", "data": "..."}},
  {"type": "text", "text": "この画像は何ですか？"}
]}

// OpenAI: content配列にimage_url
{"role": "user", "content": [
  {"type": "image_url", "image_url": {"url": "https://..."}},
  {"type": "text", "text": "この画像は何ですか？"}
]}

// Gemini: parts[]にinlineDataまたはfileData（動画・音声も同形式）
{"role": "user", "parts": [
  {"inlineData": {"mimeType": "image/jpeg", "data": "..."}},
  {"text": "この画像は何ですか？"}
]}
```

**Geminiの独自点**: YouTube URLを直接`fileData`に渡せる（動画理解）。音声ファイルも`parts`で統一。

- 📘 [Anthropic Vision ガイド](https://docs.anthropic.com/en/docs/build-with-claude/vision)
- 📘 [OpenAI Vision ガイド](https://platform.openai.com/docs/guides/vision)
- 📘 [Gemini Multimodal ガイド](https://ai.google.dev/gemini-api/docs/vision)

---

### 1-4. ファイル転送の方法

**一言**: LLMへのファイル送信は3方式。Base64は小さいファイル向き、URL参照は大きいファイル向き、Files APIはセッション跨ぎ用。

| 方式 | 概要 | 対応 |
|--|--|--|
| **Base64埋め込み** | JSONにエンコードして直接渡す | Anthropic・OpenAI・Gemini全対応 |
| **URL参照** | 画像URLを文字列で渡す | OpenAI・Gemini対応、Anthropicは非対応 |
| **Files API** | 事前アップロード→IDで参照 | OpenAI（`/v1/files`）・Anthropic（Beta）・Gemini（`/upload`） |

- 📘 [Anthropic Files API（Beta）](https://docs.anthropic.com/en/docs/build-with-claude/files)
- 📘 [OpenAI Files API](https://platform.openai.com/docs/api-reference/files)
- 📘 [Gemini File API](https://ai.google.dev/gemini-api/docs/files)

---

### 1-5. ストリーミングの形式

**一言**: LLMのレスポンスを生成と同時に受け取る仕組み。UXと遅延削減に直結する。プロトコルは3種。

| プロトコル | 概要 | 用途 |
|--|--|--|
| **ストリーミングなし** | 生成完了後に全レスポンスを一括返却 | バッチ処理、シンプルな実装 |
| **SSE（Server-Sent Events）** | HTTP上の一方向テキストストリーム。`data: {...}\n\n`形式 | Anthropic・OpenAI Chat・Gemini全対応 |
| **WebSocket** | 双方向通信。低遅延・音声対応 | OpenAI Realtime API（音声会話） |

```
// SSEのイベント形式（共通パターン）
data: {"type":"content_block_delta","delta":{"type":"text_delta","text":"Hello"}}
data: {"type":"message_stop"}
data: [DONE]   ← OpenAI形式の終端
```

**各社の対応**: Anthropic=SSEのみ。OpenAI=SSE（Chat Completions）＋WebSocket（Realtime API）。Gemini=SSE。

- 🔗 [Comparing streaming response structure for different LLM APIs](https://medium.com/percolation-labs/comparing-the-streaming-response-structure-for-different-llm-apis-2b8645028b41) — delta構造の差分詳細
- 📘 [Anthropic Streaming ガイド](https://docs.anthropic.com/en/api/messages-streaming)
- 📘 [OpenAI Streaming ガイド](https://platform.openai.com/docs/api-reference/streaming)
- 📘 [OpenAI Realtime API（WebSocket）](https://platform.openai.com/docs/api-reference/realtime)

---

### 1-6. OSSモデルとHuggingFace エコシステム

**一言**: HuggingFace Hubがモデルの中央リポジトリ。サービング層はOpenAI互換APIを公開することが多く、内部でモデル固有フォーマットに変換している。

#### HuggingFace の役割
- **HuggingFace Hub**: モデル・データセット・Spaceの公開プラットフォーム。OSSモデルのGitHub。
- **Transformers**: モデルをPythonから使うための標準ライブラリ
- **Inference API**: Hub上のモデルをそのままAPIで呼べるサービス

- 📘 [HuggingFace Hub](https://huggingface.co/docs/hub/)
- 📘 [HuggingFace Transformers](https://huggingface.co/docs/transformers/)

#### 主要モデル系列（2025〜2026年）

| 系列 | 提供元 | 特徴 |
|--|--|--|
| **LLaMA系** | Meta | OSSの代名詞。LLaMA 3以降が主流 |
| **Mistral/Mixtral系** | Mistral AI | 軽量・高性能。欧州発 |
| **Qwen系** | Alibaba | 多言語強化版 |
| **Gemma系** | Google | Geminiの軽量OSS版 |
| **DeepSeek系** | DeepSeek | 低コスト高性能。MoEアーキテクチャ |
| **Phi系** | Microsoft | 小型高性能 |

#### サービング系ライブラリの包含関係

```
HuggingFace Hub（モデル置き場）
  └── Transformers（Python API）
        ├── TGI（Text Generation Inference）― HF公式サービング。OpenAI互換エンドポイント付き
        ├── vLLM ― PagedAttentionで高スループット。OpenAI互換
        ├── Ollama ― ローカル実行向け。GGUFフォーマット。OpenAI互換
        └── llama.cpp ― C++実装の軽量サービング。Ollamaのバックエンド
```

**全てOpenAI互換`/v1/chat/completions`を提供** → LiteLLMなどのラッパーで各社API形式に変換可能

- 📘 [TGI（Text Generation Inference）](https://huggingface.co/docs/text-generation-inference/)
- 📘 [vLLM ドキュメント](https://docs.vllm.ai/)
- 📘 [Ollama ドキュメント](https://ollama.com/docs)
- 📘 [LiteLLM ドキュメント](https://docs.litellm.ai/) — 100以上のLLM APIを統一インターフェースで呼ぶ

---

### 1-7. 構造化出力 / JSON Schema

- 🔗 [How JSON Schema Works for LLM Tools & Structured Outputs (PromptLayer)](https://blog.promptlayer.com/how-json-schema-works-for-structured-outputs-and-tool-integration/) — input_schemaの役割
- 🔗 [Awesome LLM JSON (GitHub)](https://github.com/imaurer/awesome-llm-json) — structured output関連リソース集
- 🔗 [Instructor - Mode Comparison Guide](https://python.useinstructor.com/modes-comparison/) — `ANTHROPIC_TOOLS` vs `ANTHROPIC_JSON` 等のモード整理

---

### ❌ 相性問題①
- **`arguments`（文字列）vs `input`（オブジェクト）**: OpenAIはtool引数をJSON文字列化して渡す。Anthropicはオブジェクトのまま。ライブラリがパース処理を忘れると壊れる
- **`role:"tool"` vs `role:"user"+type:"tool_result"`**: ツール結果の返し方がOpenAIとAnthropicで異なる
- **OSSモデルのtool use未学習**: OpenAI互換エンドポイントを提供しても、モデルがtool useをfine-tuneされていないと正しく呼ばない

---

## 第2章　Tool Use：LLMが「実行しない」という大原則

> **対象層**: ① LLM入出力層 + ② ホスト制御層の境界

**一言**: LLMは`tool_use`ブロック（JSON）を出力するだけ。実行は常にホスト側。この「LLMとホストの往復プロトコル」がProvider間・ライブラリ間で差異を生む。

---

### 2-1. 各社のTool Use実装

#### Anthropic Tool Use

```json
// リクエスト: tools[]定義
{"tools": [{"name": "get_weather", "description": "天気を取得",
            "input_schema": {"type": "object", "properties": {"city": {"type": "string"}}}}]}

// レスポンス: tool_useブロック
{"stop_reason": "tool_use",
 "content": [{"type": "tool_use", "id": "toolu_01", "name": "get_weather", "input": {"city": "Tokyo"}}]}

// 次リクエスト: tool_result（role:userのcontent内）
{"role": "user", "content": [{"type": "tool_result", "tool_use_id": "toolu_01", "content": "晴れ 25℃"}]}
```

- 📘 [Anthropic Tool Use ガイド](https://docs.anthropic.com/en/docs/build-with-claude/tool-use) — `tools[]`定義、`tool_result`の返し方
- 📘 [Anthropic Advanced Tool Use (Programmatic Tool Calling)](https://www.anthropic.com/engineering/advanced-tool-use) — 2026年GA、トークン150,000→2,000削減事例

#### OpenAI Function Calling

```json
// レスポンス: tool_callsブロック（argumentsは文字列化JSON）
{"finish_reason": "tool_calls",
 "message": {"tool_calls": [{"id": "call_abc", "function": {
   "name": "get_weather", "arguments": "{\"city\":\"Tokyo\"}"}}]}}

// 次リクエスト: role:"tool"（専用ロール）
{"role": "tool", "tool_call_id": "call_abc", "content": "晴れ 25℃"}
```

- 📘 [OpenAI Function Calling ガイド](https://platform.openai.com/docs/guides/function-calling)
- `parallel_tool_calls: true`で複数ツール同時呼び出し可能

#### Google Gemini Function Calling

```json
// レスポンス: functionCallパーツ
{"candidates": [{"content": {"parts": [
  {"functionCall": {"name": "get_weather", "args": {"city": "Tokyo"}}}]}}]}

// 次リクエスト: functionResponseパーツ（role:userのparts内）
{"role": "user", "parts": [
  {"functionResponse": {"name": "get_weather", "response": {"result": "晴れ 25℃"}}}]}
```

- 📘 [Gemini Function Calling ガイド](https://ai.google.dev/gemini-api/docs/function-calling)
- `tool_config`で`mode: "ANY"`（強制呼び出し）や`allowed_function_names`の絞り込みが可能

#### 画像生成モデルのツール呼び出し

**一言**: DALL-E・Imagen等はLLMのtool_use呼び出し対象として使われる。モデル自体がtool_useを返すのではなく、「画像生成APIを呼ぶツール」として②層で実装する。

- 📘 [OpenAI Images API](https://platform.openai.com/docs/api-reference/images) — `tool_use`からDALL-Eを呼ぶパターン
- 📘 [Anthropic Computer Use](https://docs.anthropic.com/en/docs/build-with-claude/computer-use) — スクリーンショット（画像）をtool_resultとして返すパターン

---

### 2-2. LLMフレームワークによる抽象化

**一言**: LangChain・Strands等のライブラリは各社APIのtool_use形式差を吸収し、統一インターフェースを提供する。抽象化層が多いほどデバッグが難しくなる。

#### LangChain

**一言**: `@tool`デコレータで定義→内部でAnthropicなら`input_schema`形式、OpenAIなら`parameters`形式に自動変換。`ToolMessage`で結果を統一管理。

- 📘 [LangChain Tool Use ドキュメント](https://python.langchain.com/docs/how_to/tool_calling/)
- 📘 [LangChain Anthropic統合](https://python.langchain.com/docs/integrations/chat/anthropic/)

#### AWS Strands Agents

**一言**: `@tool`デコレータ＋BedrockのConverseAPI経由。Anthropic/Amazon/Cohere等のモデルを統一的に扱える。

- 📘 [Strands Agents Tool ドキュメント](https://strandsagents.com/latest/user-guide/concepts/tools/python-tools/)
- 📘 [Amazon Bedrock Converse API](https://docs.aws.amazon.com/bedrock/latest/userguide/conversation-inference.html) — 各社モデルのtool_use形式を吸収

#### Dify

**一言**: ノーコードでTool Useを設定。内部でOpenAI/Anthropic/Geminiそれぞれのtool定義形式に変換。GUIで「ツールノード」として視覚化。

- 📘 [Dify Tool ドキュメント](https://docs.dify.ai/guides/tools)

---

### 解説ブログ
- 🔗 [Cloudflare "Code Mode: the better way to use MCP"](https://blog.cloudflare.com/code-mode/) — 「LLMはスペシャルトークンとJSONを出力するだけ、実行はハーネス側」の図解が秀逸
- 🔗 [Akamai AI Explainer: What Is Model Context Protocol?](https://www.akamai.com/blog/trends/2025/nov/ai-explainer-what-is-model-context-protocol) — LLMがJSON出力→クライアントが受け取り実行の流れ図
- 🔗 [OpenAI Function Calling vs Anthropic MCP (Medium)](https://evgeniisaurov.medium.com/demystifying-openai-function-calling-vs-anthropics-model-context-protocol-mcp-b5e4c7b59ac2)

---

### ❌ 相性問題②
- **tool descriptionが貧弱**: LLMはdescriptionを読んで「いつ使うか」を判断する。曖昧な説明→ツールを呼ばない・誤った呼び方をする
  - 🔗 [Evaluating AI agents: Amazon実践知見](https://aws.amazon.com/blogs/machine-learning/evaluating-ai-agents-real-world-lessons-from-building-agentic-systems-at-amazon/) — 「ツールスキーマと意味記述の不備が誤ツール選択の主因」
- **ライブラリのバージョンとAPIのズレ**: LangChainのAPIラッパーが古いとAnthropicの新パラメータ（`tool_choice: {type:"any"}`等）が使えない
- **フレームワークの隠れたsystem prompt**: LangChain等が裏で追加するpromptが意図しない動作を引き起こす。「APIを直接叩けばわかる」
  - 🔗 [FU, Show Me The Prompt. (Hamel Husain)](https://hamel.dev/blog/posts/prompt/) — mitmproxyでAPIコールを傍受してフレームワーク内部のpromptを見る方法

---

## 第3章　MCP：ツール定義を標準化・外部化するプロトコル

> **対象層**: ① / ② 境界（サーバーは②、クライアントも②、LLMが受け取るのは①）

**一言**: `tools[]`の定義をAPIリクエストに毎回書く代わりに、MCPサーバーが自動公開する仕組み。**LLMが受け取るJSONは変わらない**——MCPはあくまで「どうやってtools[]を用意するか」の話。

---

### 3-1. 層の対応関係

```
① LLM入出力層:   LLMが受け取る tools[] は普通のJSON（MCPを知らない）
                  ↑ MCPクライアントが tools/list の結果を tools[] に変換して渡す
② ホスト制御層:  MCPクライアント（ClaudeCodeやCursorなど宿主アプリの一部）
                  MCPサーバー（ツールを実際に実行するプログラム）
③ プロンプト層:  .mcp.json → どのMCPサーバーを起動するかの設定（起動は②層）
```

| 概念 | 属する層 | 役割 |
|--|--|--|
| MCPサーバー | ② ホスト制御層 | ツールを実装・公開するプログラム |
| MCPクライアント | ② ホスト制御層 | サーバーからtools[]を取得し、LLMへ渡す橋渡し |
| `.mcp.json` | ②層の設定 | どのサーバーを起動するか記述。ClaudeCodeが読んでサーバーを起動 |
| LLMが受け取るもの | ① | 通常の `tools[]` JSON（MCPを意識しない） |

---

### 3-2. ローカルMCP vs リモートMCP

| | ローカルMCP | リモートMCP |
|--|--|--|
| トランスポート | stdio（プロセス間通信） | HTTP/SSE（Streamable HTTP） |
| 起動方法 | `.mcp.json`に`command`を記述 | `.mcp.json`に`url`を記述 |
| 認証 | 不要（同一マシン） | OAuth 2.0 必須 |
| 用途 | 個人・ローカル開発 | Enterprise・クラウド |

```json
// .mcp.json の例
{
  "mcpServers": {
    "filesystem": {
      "command": "npx", "args": ["-y", "@modelcontextprotocol/server-filesystem", "/tmp"]
    },
    "remote-crm": {
      "url": "https://crm.example.com/mcp", "transport": "streamable-http"
    }
  }
}
```

---

### 3-3. OpenAI / Gemini での MCP

- **OpenAI**: 2025年3月にMCP採用発表。Agents SDK（Python）がMCPクライアントとして動作。内部でMCPのtools→OpenAI形式の`function.parameters`に変換。
  - 📘 [OpenAI Agents SDK + MCP](https://openai.github.io/openai-agents-python/mcp/)
- **Gemini / Google ADK**: ADK（Agent Development Kit）がMCPクライアントをサポート。`functionDeclarations`形式に変換。
  - 📘 [Google ADK + MCP](https://google.github.io/adk-docs/tools/mcp-tools/)

**結論**: どのプロバイダーでも「MCPサーバー → クライアントが変換 → 各社のtool定義形式」という経路は同じ。変換先フォーマットが違うだけ。

---

### 公式リファレンス
- 📘 [MCP 公式サイト](https://modelcontextprotocol.io/) — 仕様・SDK・サーバー一覧
- 📘 [Anthropic MCP発表ブログ](https://www.anthropic.com/news/model-context-protocol)
- 📘 [MCP仕様 (GitHub)](https://github.com/modelcontextprotocol/modelcontextprotocol) — JSON-RPC 2.0の詳細

### 解説ブログ（英語）
- 🔗 [Cloudflare "Code Mode"](https://blog.cloudflare.com/code-mode/) — 「LLMはコードを書いてMCPを呼ぶ方が、MCPを直接呼ぶより得意」
- 🔗 [Tools for Your LLM: a Deep Dive into MCP (Towards Data Science)](https://towardsdatascience.com/tools-for-your-llm-a-deep-dive-into-mcp/) — `tools/list → tools/call`の往復、fastmcpコード例
- 🔗 [Akamai MCP Explainer](https://www.akamai.com/blog/trends/2025/nov/ai-explainer-what-is-model-context-protocol) — 「最終的にLLMに渡るのはテキストとJSONのみ」の明快な説明
- 🔗 [Plugging AI Into Everything (Bertelsmann Tech)](https://tech.bertelsmann.com/en/blog/articles/plugging-ai-into-everything-how-the-model-context-protocol-is-changing-the-game-apis-for-llms) — USB-C比喩、N+N問題の解説
- 🔗 [MCP vs. Function Calling (Descope)](https://www.descope.com/blog/post/mcp-vs-function-calling) — ローカルMCP vs リモートMCPのトレードオフ
- 🔗 [Anthropic MCP vs OpenAI Agents SDK 比較](https://rabot.medium.com/winning-in-the-autonomous-ai-agents-race-a0c03d52acad)

### 解説ブログ（日本語）
- 🔗 [AI Shift Advent Calendar 2024「AIエージェントの設計とその勘所」](https://www.ai-shift.co.jp/techblog/5252) — 「MCPの価値は関心の集約」、パターン1〜3の整理
- 🔗 [Elastic「モデルコンテキストプロトコル（MCP）とは？」日本語版](https://www.elastic.co/jp/what-is/mcp)

### 各ツールのMCP設定ファイル
| ツール | 設定ファイル | ドキュメント |
|--|--|--|
| Claude Code | `.mcp.json` (プロジェクト) / `~/.claude/` | [Claude Code MCP設定](https://code.claude.com/docs/en/mcp) |
| Cursor | `.cursor/mcp.json` | [Cursor MCP](https://docs.cursor.com/context/model-context-protocol) |
| Amazon Q CLI | `~/.aws/amazonq/mcp.json` | [Amazon Q MCP](https://docs.aws.amazon.com/amazonq/latest/qdeveloper-ug/mcp.html) |

### ❌ 相性問題③：MCPサーバーのスキーマ不正 → LLMが正しく呼べない。また「MCPサーバーを入れればどのLLMでも動く」は誤り——クライアント側の変換実装が必要。

---

## 第4章　RAG：ツールではなくプロンプト操作だった

> **対象層**: ③ プロンプト層（従来RAG）/ ① + ② （Tool型RAG）

**一言（対比）**:
- **従来RAG（③層）**: ホストが機械的にベクトル検索→結果をpromptに埋め込む。LLMは検索を知らない。クエリはユーザー入力のコピー。1回しか検索しない。
- **Tool型RAG（①②層）**: LLMが自分でクエリを生成し、検索ツールを`tool_use`で呼ぶ。結果を見て再検索できる。複数クエリ・反復可能。「A社とB社を比較して」のような複合クエリに強い。

**設計の転換点**: 「検索の主体がホストかLLMか」が分岐点。

---

### 4-1. 主要ライブラリ・フレームワークの分類

| ツール/OSS | 従来RAG | Tool型RAG | 備考 |
|--|--|--|--|
| **LangChain** | ✅（RAG chain） | ✅（Agent + retriever tool） | 両対応の代表格 |
| **LlamaIndex** | ✅（SimpleRAG） | ✅（AgentRAG / QueryEngineTool） | 両対応、RAG特化で詳細 |
| **Dify** | ✅（Knowledgebase） | ✅（Tool nodes） | ノーコードで両対応 |
| **LangGraph** | ✅ | ✅（グラフで制御フロー明示） | 反復・条件分岐が得意 |
| **AutoGen** | — | ✅ | Tool型RAG中心 |
| **Strands Agents** | — | ✅（@toolで検索ツール定義） | AWS Bedrock連携 |
| **Anthropic RAGガイド** | ✅（サンプルあり） | ✅（推奨） | Tool型を推奨 |

---

### 公式リファレンス
- 📘 [Anthropic RAG ガイド](https://docs.anthropic.com/en/docs/build-with-claude/retrieval-augmented-generation)
- 📘 [LlamaIndex ドキュメント](https://docs.llamaindex.ai/) — SimpleRAG〜AgentRAGまで
- 📘 [LangChain RAG ドキュメント](https://python.langchain.com/docs/tutorials/rag/)

### 解説ブログ
- 🔗 [Chip Huyen "Agents" (2025/01)](https://huyenchip.com/2025/01/07/agents.html) — 「RAGシステムもエージェント。テキスト検索器・SQLエグゼキュータがツール」
- 🔗 [Anthropic "Building Effective Agents"](https://www.anthropic.com/research/building-effective-agents) — augmented LLMの基本構成（Retrieval + Tools + Memory）
- 🔗 [Context Engineering Overview (Elastic)](https://www.elastic.co/search-labs/blog/context-engineering-overview) — RAGをコンテキスト工学の一部として位置づけ。コンテキスト汚染の問題も
- 🔗 [IDCF「プロンプトエンジニアリング完全ガイド」](https://blog.idcf.jp/entry/PromptEngineering) — RAG vs MCPの日本語解説

### 書籍
- 📗 Chip Huyen "AI Engineering" (O'Reilly 2025) — Chapter: Agents → RAG as tool use

### ❌ 相性問題④：従来RAGをTool型RAGと混同すると、LLMが「なぜ文書が来たか知らない」問題が起きる。LLMの視点では③層の文書注入と①層のtool_resultは全く別物。

---

## 第5章　CLAUDE.md / AGENTS.md：プロンプト注入の設定ファイル

> **対象層**: ③ プロンプト層

**一言**: セッション開始時にsystem promptへ自動注入されるMarkdown。APIレベルでは「長いsystem prompt」でしかない。

### 公式リファレンス
- 📘 [Claude Code Memory管理 公式ドキュメント](https://code.claude.com/docs/en/memory) — ディレクトリ階層・優先度・`@import`構文
- 📘 [Claude Code Settings 公式ドキュメント](https://code.claude.com/docs/en/settings)
- 📘 [AGENTS.md 標準 (Agentic AI Foundation)](https://agents.md/) — Linux Foundation傘下、Cursor/Amp/Google Jules/Factory/OpenAI Codexが採用

### 解説ブログ
- 🔗 [IDCF ブログ](https://blog.idcf.jp/entry/PromptEngineering) — system promptとプロンプト注入の基礎

### 対応表
| ツール | コンテキストファイル | グローバル設定 |
|--|--|--|
| Claude Code | `CLAUDE.md` / `.claude/` | `~/.claude/CLAUDE.md` |
| OpenAI Codex | `AGENTS.md` | `~/.codex/AGENTS.md` |
| Cursor | `.cursor/rules/` | — |
| GitHub Copilot | `.github/copilot-instructions.md` | — |

---

## 第6章　Skill：動的プロンプト拡張と「コンテキスト節約」

> **対象層**: ③ プロンプト層（`context: fork`でSubagentに移行可能）

**一言**: 使用時だけロードされるプロンプトのモジュール。起動時は`name`と`description`だけ（~100トークン）読む。タスクが来たとき初めてSKILL.md全体をロード（Progressive Disclosure）。

### 公式リファレンス
- 📘 [Claude Code Skills ドキュメント](https://code.claude.com/docs/en/skills)

### SkillとToolの根本的な違い
| | Tool (tool_use) | Skill |
|--|--|--|
| LLMからの視点 | 「このツールを呼ぶ」宣言 | 「こういう手順で解く」知識 |
| プロトコル | ① JSON往復 | ③ プロンプト注入 |
| 実行主体 | ホスト側プログラム | LLM自身 |

### ディレクトリ比較
```
Claude Code:  .claude/skills/<skill-name>/SKILL.md
OpenAI Codex: .agents/skills/<skill-name>/SKILL.md
```

---

## 第7章　Hooks：プログラムが担保する決定論的制御

> **対象層**: ② ホスト制御層（最重要）

**一言**: LLMへの指示（③層）は確率的で「必ず実行」できない。Hooksはシェルスクリプトを確実に実行する確率ゼロの制御。

### 公式リファレンス
- 📘 [Claude Code Hooks 公式ドキュメント](https://code.claude.com/docs/en/hooks) — 14種のイベント一覧、exit code 2でのブロック方法
- 📘 [Claude Code Settings](https://code.claude.com/docs/en/settings) — `settings.json`の`hooks`ブロック構造

### Hooksイベント（主要）
```
PreToolUse     ← ★最高優先。ブロック可能。tool_use受け取り直後
PostToolUse    ← 実行後のログ・コンテキスト注入
SessionStart / SessionEnd
UserPromptSubmit
SubagentStart / SubagentStop
```

### 他実装との比較
| | Claude Code Hooks | OpenAI Codex | Amazon AgentCore |
|--|--|--|--|
| ブロック機能 | PreToolUse exit 2 | なし（現状） | Cedar policy |
| 実装方式 | シェルスクリプト | — | Cedar DSL |

### 参考
- 🔗 [Descope "What Is MCP?"](https://www.descope.com/learn/post/mcp) — Replit 事例：「2025年7月、本番DBの1,200件以上のレコード削除。外部OAuthスコープで権限管理していれば防げた」→ ②層での安全担保の重要性

### ❌ 相性問題⑤：「CLAUDE.mdに書いたからHooksより弱い」という認識違い（PreToolUseは③層の上位）

---

## 第8章　Subagent / Agents：Tool Useの応用として別セッションを起動する

> **対象層**: ① ② ③ 全層の集大成

**一言**: SubagentはTaskツールの`tool_use`から始まる。ClaudeCodeがAPIリクエストを新規立ち上げ、完了したら`tool_result`で返る。

### 公式リファレンス
- 📘 [Claude Code Sub-agents 公式ドキュメント](https://code.claude.com/docs/en/sub-agents)
- 📘 [Amazon Bedrock AgentCore](https://aws.amazon.com/bedrock/agentcore/) — クラウドスケール版

### 解説ブログ
- 🔗 [Anthropic "Building Effective Agents"](https://www.anthropic.com/research/building-effective-agents) — Orchestrator-Workers パターンの公式解説
- 🔗 [Chip Huyen "Agents"](https://huyenchip.com/2025/01/07/agents.html) — Tool Use・Planning・Failure Modesまで
- 🔗 [Building AI Agents: Anthropic's 6 Composable Patterns (AIMpultiple)](https://aimultiple.com/building-ai-agents) — prompt chaining / routing / parallelization等パターン実装例

### Subagentの1ファイル = 3層全てを含む
```yaml
# .claude/agents/code-reviewer.md
---
tools: Read, Glob, Grep          # ① 層の制限
disallowedTools: Write, Bash     # ① 層の制限
model: claude-haiku-4-5          # コスト最適化
permissionMode: default          # ② 層の設定
hooks:                           # ② 層（Subagent固有）
  PreToolUse: ...
skills:                          # ③ 層（Subagent固有）
  - code-review-patterns
---
# ③層: system prompt本文
You are a code reviewer...
```

### ClaudeCode Subagent vs AgentCore Runtime
| 観点 | ClaudeCode | AgentCore |
|--|--|--|
| 実行場所 | ローカル | クラウド |
| 永続性 | セッション終了で消える | 最大8時間・永続対応 |
| フレームワーク | ClaudeCode専用 | LangGraph/CrewAI/Strands等 |

---

## 第9章　Agent Teams：エージェント間通信プロトコルの話

> **対象層**: ① プロトコル（エージェント間）

**一言**: ClaudeCode Teamsはファイル共有（ホワイトボードパターン）。AgentCoreはA2A JSON-RPC。通信方式の違いが「相性問題」になる。

### 公式リファレンス
- 📘 [Amazon Bedrock AgentCore A2A対応](https://aws.amazon.com/bedrock/agentcore/) — Agent Card / JSON-RPC 2.0 / SSE
- 📘 [Google A2A プロトコル仕様](https://google.github.io/A2A/)

### 解説ブログ
- 🔗 [Anthropic "Building Effective Agents" - Orchestrator-Workers](https://www.anthropic.com/research/building-effective-agents)
- 🔗 [Zenn「マルチLLMエージェントシステム（MLAS）のアーキテクチャ4分類」](https://zenn.dev/r_kaga/articles/ebeba0bd1385e1) — Star/Ring/Mesh/Hierarchical の分類（日本語）
- 🔗 [Zenn「2025年の年始に読み直したいAIエージェント設計原則」](https://zenn.dev/r_kaga/articles/e0c096d03b5781)（日本語）

### ClaudeCode Teams vs AgentCore Teams
| 観点 | ClaudeCode Teams | AgentCore Teams |
|--|--|--|
| 通信方式 | tool_result（ファイル共有） | A2A JSON-RPC 2.0 |
| 状態共有 | ファイルシステム（同一マシン） | HTTP / ステートレス |
| スケール | 1マシン | クラウドスケール |
| 永続性 | セッション終了で消える | 最大8時間 |

### ❌ 相性問題⑥：フレームワーク間のAgent通信 → A2Aプロトコル未対応

---

## 第10章　プラグイン：上記すべての配布単位

> **対象層**: ③を中心に全層を梱包する

**一言**: PluginはSkill/Agent/Hook/MCPサーバーをまとめた配布パッケージ。

### ディレクトリ構造
```
my-plugin/
├── .claude-plugin/plugin.json  # メタデータ
├── commands/                   # スラッシュコマンド（③層）
├── agents/                     # Subagent定義（①②③層）
├── skills/                     # Skill定義（③層）
├── hooks/                      # Hookスクリプト（②層）
└── .mcp.json                   # MCPサーバー（①層の拡張）
```

### 参考
- 📘 [Claude Code カスタムコマンド](https://code.claude.com/docs/en/slash-commands)
- 📘 [OpenAI Codex Skills公式](https://developers.openai.com/codex/skills/) — `allow_implicit_invocation`、`dependencies.tools`

---

## 第11章　権限管理：②層の設計比較

> **対象層**: ② ホスト制御層

**一言**: 「何を誰が信頼するか」の哲学の違い。LLMを信頼→人間レビュー（ClaudeCode）/ 人間承認を頻繁に挟む（Codex）/ Policyが全て判定（AgentCore）。

### 公式リファレンス
- 📘 [Claude Code Settings - permissions](https://code.claude.com/docs/en/settings) — `permissionMode` / `allowedTools` / `disallowedTools`
- 📘 [Amazon Bedrock AgentCore - Authorization](https://aws.amazon.com/bedrock/agentcore/) — Cedar policy / IAM+OAuth2.0 / VPC connectivity

### 解説ブログ
- 🔗 [Descope "What Is MCP?" — セキュリティと認証](https://www.descope.com/learn/post/mcp) — Knostic調査：2,000 MCPサーバーのうち認証なし100%の衝撃
- 🔗 [Anthropic "Building Effective Agents"](https://www.anthropic.com/research/building-effective-agents) — 「human-in-the-loop設計の重要性」

### 設計哲学の比較
```
ClaudeCode：  LLMを信頼。人間がレビュー。Hooksで例外処理。
Codex：       人間承認を頻繁に挟む。サンドボックスで封じ込め。
AgentCore：   Policyが全てを判定。LLMの判断を通さない。
```

---

## 第12章　全体俯瞰：「相性問題」はどこで起きるか

### 相性問題の発生箇所マッピング

| 問題の種類 | 発生層 | 原因 | 参照 |
|--|--|--|--|
| APIフォーマット不一致（arguments文字列化等） | ① | ベンダー間のtool定義・結果形式の差 | 第1・2章 |
| tool descriptionが貧弱 | ① | LLMがツールを使わない | 第2章 |
| モデルがtool use未学習 | ① | OSSモデルの選択ミス | 第1章 |
| ライブラリのバージョンとAPIのズレ | ①② | LangChain等のラッパー層の更新遅れ | 第2章 |
| HookがPermissionより弱い認識 | ② | PreToolUseの設計理解不足 | 第7章 |
| MCPスキーマ不正 | ①② | サーバー実装のJSONスキーマ誤り | 第3章 |
| CLAUDE.mdが巨大すぎる | ③ | コンテキスト汚染・優先度喪失 | 第5章 |
| Skillのdescriptionが曖昧 | ③ | 自動マッチ失敗 | 第6章 |
| 従来RAGとTool型RAGの混同 | ①③ | LLMが文書の出所を知らない | 第4章 |
| Subagentのtools制限が甘い | ①② | セキュリティリスク | 第8章 |
| フレームワーク間Agent通信 | ① | A2Aプロトコル未対応 | 第9章 |

### AgentCore vs ClaudeCode：何を選ぶべきか
- ローカル・個人開発・Claude専用 → **ClaudeCode**
- エンタープライズ・マルチフレームワーク・クラウドスケール → **AgentCore**
- 両方連携（ClaudeCodeでAgentCoreのMCPを叩く）も可能

---

## 付録A　ディレクトリ構造比較表

```
Claude Code                    OpenAI Codex
─────────────────────          ─────────────────────
.claude/                       .agents/  or ~/.codex/
├── settings.json              ├── AGENTS.md
├── settings.local.json        ├── AGENTS.override.md
├── agents/*.md                └── skills/<n>/SKILL.md
├── commands/*.md
├── skills/<n>/SKILL.md    Cursor
├── rules/*.md                 .cursor/
└── hooks/                     ├── rules/
                               └── mcp.json
~/.claude/
├── CLAUDE.md                  GitHub Copilot
├── rules/                     .github/
├── agents/                    └── copilot-instructions.md
└── skills/
```

---

## 付録B　公式ドキュメント一覧（2026年2月時点）

| サービス | ドキュメント |
|--|--|
| Claude Code | https://code.claude.com/docs/en/ |
| Anthropic API | https://docs.anthropic.com/ |
| OpenAI Codex | https://developers.openai.com/codex |
| OpenAI API | https://platform.openai.com/docs |
| Google Gemini API | https://ai.google.dev/gemini-api/docs |
| Google ADK | https://google.github.io/adk-docs/ |
| Amazon AgentCore | https://aws.amazon.com/bedrock/agentcore/ |
| Amazon Bedrock Converse API | https://docs.aws.amazon.com/bedrock/latest/userguide/conversation-inference.html |
| MCP公式 | https://modelcontextprotocol.io/ |
| AGENTS.md標準 | https://agents.md/ |
| HuggingFace Hub | https://huggingface.co/docs/hub/ |
| LangChain | https://python.langchain.com/docs/ |
| LangGraph | https://langchain-ai.github.io/langgraph/ |
| LlamaIndex | https://docs.llamaindex.ai/ |
| Dify | https://docs.dify.ai/ |
| Strands Agents (AWS) | https://strandsagents.com/ |
| Microsoft AutoGen | https://microsoft.github.io/autogen/ |
| Goose (Block Inc) | https://block.github.io/goose/ |
| vLLM | https://docs.vllm.ai/ |
| Ollama | https://ollama.com/docs |
| LiteLLM | https://docs.litellm.ai/ |

---

## 付録C　書籍リスト

| 書籍 | 著者 | 出版 | 特に参考になる章 |
|--|--|--|--|
| **AI Engineering: Building Applications with Foundation Models** | Chip Huyen | O'Reilly 2025 | Agents / RAG / Tool Use / Planning |
| **LLM Engineer's Handbook** | Iusztin & Labonne | Packt 2024 | LLMOps / プロダクション化 |
| **Building LLM Powered Applications** | Valentina Alto | Packt 2024 | LangChain / メモリ / ツール統合 |
| **Prompt Engineering for LLMs** | Beurer-Kellner et al. | O'Reilly 2024 | プロンプト設計原則 |

---

## 付録D　この記事では解説しない（リンク集）

### フレームワーク比較
- 🔗 [Qiita「AIエージェント入門③ 設計アーキテクチャ」](https://qiita.com/syukan3/items/153409d7f387ea8c3065)（日本語）
- 🔗 [Zenn「2025年の年始に読み直したいAIエージェント設計原則」](https://zenn.dev/r_kaga/articles/e0c096d03b5781)（日本語）

### プロンプトエンジニアリング基礎
- 📘 [Anthropic Prompt Engineering ガイド](https://docs.anthropic.com/en/docs/build-with-claude/prompt-engineering/overview)
- 🔗 [Chip Huyen "Building LLM applications for production"](https://huyenchip.com/2023/04/11/llm-engineering.html)

### コンテキスト工学
- 🔗 [Phil Schmid "Context Engineering" (図解)](https://www.philschmid.de/context-engineering)
- 🔗 [Elastic "Context Engineering Overview"](https://www.elastic.co/search-labs/blog/context-engineering-overview)

---

## 未解決・要調査事項

- [ ] Gemini API の tool_choice に相当するフォーマット詳細（`tool_config`の`mode`の全オプション）
- [ ] OpenCode（OSSのClaudeCode代替）のディレクトリ構造
- [ ] ClaudeCode SubagentとAgentCoreのRuntimeを連携させた場合の制限
- [ ] Dify / LangChain でのTool Use実装が各APIをどう抽象化しているか
- [ ] Gooseのpermission設計（ClaudeCode Hooksとの対比）

---

*骨子バージョン: 2026-02-21 rev2*
*調査日: 2026-02-21*
*主な参照: Claude Code公式docs / OpenAI Codex公式 / Amazon AgentCore公式 / Chip Huyen "AI Engineering" / Anthropic "Building Effective Agents"*
