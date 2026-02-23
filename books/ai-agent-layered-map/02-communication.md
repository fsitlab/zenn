---
title: "第2層：通信層（LLM入出力層）"
---

# 第2層：通信層（LLM入出力層）

> **5層モデルでの位置**: 第2層
> **一言**: LLMはステートレスなHTTPサービス。JSONを受け取り、JSONを返すだけ。実行しない。

---

## 目次

1. [通信層の概要](#1-通信層の概要)
2. [2層IN：クライアント起点の通信](#2-2層INクライアント起点の通信)
3. [2層OUT：LLM起点の通信](#3-2層OUTllm起点の通信)
4. [ストリーミング](#4-ストリーミング)
5. [フォーマット差分比較表](#5-フォーマット差分比較表)

---

## 1. 通信層の概要

通信層は、クライアント（ホスト）とLLMの間でやり取りされるJSONの構造を定義する層。

### 基本原則

- LLMはステートレス：`messages[]`配列を毎回全部送る
- LLMは実行しない：`tool_use`を出力するだけ、実行はホスト側
- ベンダー間でフォーマットが微妙に異なる（相性問題の根本原因）

### 主要3社のAPIエンドポイント

| プロバイダー | エンドポイント |
|--|--|
| Anthropic | `POST https://api.anthropic.com/v1/messages` |
| OpenAI | `POST https://api.openai.com/v1/chat/completions` |
| Gemini | `POST https://generativelanguage.googleapis.com/v1beta/models/{model}:generateContent` |

### 公式リファレンス

- [Anthropic Messages API](https://docs.anthropic.com/en/api/messages)
- [OpenAI Chat Completions API](https://platform.openai.com/docs/api-reference/chat)
- [Google Gemini generateContent](https://ai.google.dev/api/generate-content)

---

## 2. 2層IN：クライアント起点の通信

クライアント（ホスト）からLLMへ送るリクエストの構造。

### 2-1. 基本リクエスト構造

#### Anthropic Messages API

```json
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

**特徴**: `system`が独立フィールド。`content`はブロック配列対応。

#### OpenAI Chat Completions API

```json
{
  "model": "gpt-4o",
  "max_tokens": 1024,
  "messages": [
    {"role": "system", "content": "You are a helpful assistant."},
    {"role": "user", "content": "Hello"},
    {"role": "assistant", "content": "Hi!"},
    {"role": "user", "content": "What is 2+2?"}
  ]
}
```

**特徴**: `system`は`messages[]`の1要素。

#### Gemini API

```json
{
  "systemInstruction": {"parts": [{"text": "You are a helpful assistant."}]},
  "contents": [
    {"role": "user", "parts": [{"text": "Hello"}]},
    {"role": "model", "parts": [{"text": "Hi!"}]},
    {"role": "user", "parts": [{"text": "What is 2+2?"}]}
  ],
  "generationConfig": {"temperature": 0.7, "maxOutputTokens": 1024}
}
```

**特徴**: `contents`に`parts[]`配列。アシスタントロールが`"model"`。

---

### 2-2. プロンプトチューニング（リクエストパラメータ）

モデルの「確率的振る舞い」を制御するパラメータ。

| パラメータ | 概要 | Anthropic | OpenAI | Gemini |
|--|--|--|--|--|
| `temperature` | 出力のランダム性（0=決定的、1=多様） | 0-1 | 0-2 | 0-2 |
| `top_p` | 上位p%の確率トークンのみサンプリング | 対応 | 対応 | 対応 |
| `top_k` | 上位k個のトークンのみサンプリング | 対応 | 非対応 | 対応 |
| `max_tokens` | 最大出力トークン数 | `max_tokens` | `max_tokens` | `maxOutputTokens` |
| `stop` | 生成停止シーケンス | `stop_sequences` | `stop` | `stopSequences` |

**設計指針**:
- 決定論的な出力が必要 → `temperature: 0`
- 創造的な出力が必要 → `temperature: 0.7-1.0`
- JSONなど構造化出力 → `temperature: 0` + JSON Schema

#### 公式リファレンス

- [Anthropic API パラメータ一覧](https://docs.anthropic.com/en/api/messages)
- [OpenAI API パラメータ一覧](https://platform.openai.com/docs/api-reference/chat/create)
- [Google Gemini generationConfig](https://ai.google.dev/api/generate-content#v1beta.GenerationConfig)

---

### 2-3. ツール一覧の宣言（tools[]）

LLMに「使用可能なツール」を知らせる配列。LLMはこの宣言を見て、どのツールをいつ呼ぶか判断する。

#### Anthropic形式

```json
{
  "tools": [
    {
      "name": "get_weather",
      "description": "指定した都市の天気を取得する",
      "input_schema": {
        "type": "object",
        "properties": {
          "city": {"type": "string", "description": "都市名"}
        },
        "required": ["city"]
      }
    }
  ]
}
```

#### OpenAI形式

```json
{
  "tools": [
    {
      "type": "function",
      "function": {
        "name": "get_weather",
        "description": "指定した都市の天気を取得する",
        "parameters": {
          "type": "object",
          "properties": {
            "city": {"type": "string", "description": "都市名"}
          },
          "required": ["city"]
        }
      }
    }
  ]
}
```

#### Gemini形式

```json
{
  "tools": [{
    "functionDeclarations": [
      {
        "name": "get_weather",
        "description": "指定した都市の天気を取得する",
        "parameters": {
          "type": "object",
          "properties": {
            "city": {"type": "string", "description": "都市名"}
          },
          "required": ["city"]
        }
      }
    ]
  }]
}
```

### ツール宣言の重要ポイント

- **description**が最重要：LLMはdescriptionを読んで「いつ使うか」を判断する
- 曖昧なdescription → ツールを呼ばない・誤った呼び方をする
- JSON Schemaでパラメータを厳密に定義 → 正確な引数生成

#### 公式リファレンス

- [Anthropic Tool Use ガイド](https://docs.anthropic.com/en/docs/build-with-claude/tool-use)
- [OpenAI Function Calling ガイド](https://platform.openai.com/docs/guides/function-calling)
- [Gemini Function Calling ガイド](https://ai.google.dev/gemini-api/docs/function-calling)

---

## 3. 2層OUT：LLM起点の通信

LLMがホストに対して「ツールを呼んでほしい」と要求する仕組み。

### 3-1. ツール利用の判断（tool_use）

LLMは`tool_use`ブロック（JSON）を出力するだけ。**実行は常にホスト側**。

#### Anthropicのtool_useレスポンス

```json
{
  "stop_reason": "tool_use",
  "content": [
    {
      "type": "tool_use",
      "id": "toolu_01ABC",
      "name": "get_weather",
      "input": {"city": "Tokyo"}
    }
  ]
}
```

**特徴**: `input`はオブジェクト（パース不要）。

#### OpenAIのtool_callsレスポンス

```json
{
  "finish_reason": "tool_calls",
  "message": {
    "tool_calls": [
      {
        "id": "call_abc123",
        "type": "function",
        "function": {
          "name": "get_weather",
          "arguments": "{\"city\":\"Tokyo\"}"
        }
      }
    ]
  }
}
```

**特徴**: `arguments`は**文字列化JSON**（パース必要）。

#### GeminiのfunctionCallレスポンス

```json
{
  "candidates": [{
    "content": {
      "parts": [{
        "functionCall": {
          "name": "get_weather",
          "args": {"city": "Tokyo"}
        }
      }]
    }
  }]
}
```

**特徴**: `args`はオブジェクト。`parts[]`内に配置。

---

### 3-2. LLMが主体的にツールを呼び出す仕組み

#### 往復プロトコルの流れ

```
1. クライアント → LLM: tools[]宣言 + ユーザーメッセージ
2. LLM → クライアント: tool_use（「このツールを呼んで」）
3. クライアント: ツールを実際に実行
4. クライアント → LLM: tool_result（実行結果）
5. LLM → クライアント: 最終回答
```

#### tool_resultの返し方

##### Anthropic

```json
{
  "role": "user",
  "content": [
    {
      "type": "tool_result",
      "tool_use_id": "toolu_01ABC",
      "content": "東京の天気は晴れ、気温25度です"
    }
  ]
}
```

##### OpenAI

```json
{
  "role": "tool",
  "tool_call_id": "call_abc123",
  "content": "東京の天気は晴れ、気温25度です"
}
```

##### Gemini

```json
{
  "role": "user",
  "parts": [{
    "functionResponse": {
      "name": "get_weather",
      "response": {"result": "東京の天気は晴れ、気温25度です"}
    }
  }]
}
```

### tool_choiceによる呼び出し制御

LLMにツール呼び出しを強制または禁止できる。

| 値 | 動作 | Anthropic | OpenAI | Gemini |
|--|--|--|--|--|
| auto | LLMが判断 | `{"type": "auto"}` | `"auto"` | `mode: "AUTO"` |
| required | 必ず呼ぶ | `{"type": "any"}` | `"required"` | `mode: "ANY"` |
| 特定ツール | 指定ツールを呼ぶ | `{"type": "tool", "name": "..."}` | `{"type": "function", "function": {"name": "..."}}` | `allowed_function_names: [...]` |
| none | 呼ばない | `{"type": "none"}` | `"none"` | `mode: "NONE"` |

---

## 4. ストリーミング

LLMのレスポンスを生成と同時に受け取る仕組み。UXと遅延削減に直結する。

### 4-1. プロトコル比較

| プロトコル | 概要 | 特徴 |
|--|--|--|
| **ストリーミングなし** | 生成完了後に全レスポンスを一括返却 | シンプル、バッチ処理向き |
| **SSE (Server-Sent Events)** | HTTP上の一方向テキストストリーム | 標準的、全社対応 |
| **WebSocket** | 双方向通信 | 低遅延、音声対話向き |

### 4-2. SSE（Server-Sent Events）

HTTP上で`text/event-stream`形式のレスポンスを返す。

```
data: {"type":"content_block_delta","delta":{"type":"text_delta","text":"Hello"}}

data: {"type":"content_block_delta","delta":{"type":"text_delta","text":" World"}}

data: {"type":"message_stop"}
```

#### 各社の対応

| プロバイダー | SSE | WebSocket |
|--|--|--|
| Anthropic | 対応 | 非対応 |
| OpenAI | 対応（Chat Completions） | 対応（Realtime API） |
| Gemini | 対応 | 対応（Live API） |

#### SSEのイベント形式比較

##### Anthropic

```
event: content_block_delta
data: {"type":"content_block_delta","index":0,"delta":{"type":"text_delta","text":"Hello"}}

event: message_stop
data: {"type":"message_stop"}
```

##### OpenAI

```
data: {"choices":[{"delta":{"content":"Hello"}}]}

data: [DONE]
```

##### Gemini

```
data: {"candidates":[{"content":{"parts":[{"text":"Hello"}]}}]}
```

### 4-3. WebSocket（OpenAI Realtime API）

双方向通信で音声会話に対応。

```javascript
// WebSocket接続
const ws = new WebSocket("wss://api.openai.com/v1/realtime?model=gpt-4o-realtime-preview");

// イベント送信
ws.send(JSON.stringify({
  type: "input_audio_buffer.append",
  audio: base64AudioData
}));

// イベント受信
ws.onmessage = (event) => {
  const data = JSON.parse(event.data);
  // type: "response.audio.delta" など
};
```

#### 公式リファレンス

- [Anthropic Streaming ガイド](https://docs.anthropic.com/en/api/messages-streaming)
- [OpenAI Streaming ガイド](https://platform.openai.com/docs/api-reference/streaming)
- [OpenAI Realtime API（WebSocket）](https://platform.openai.com/docs/api-reference/realtime)

#### 解説ブログ

- [Comparing streaming response structure for different LLM APIs](https://medium.com/percolation-labs/comparing-the-streaming-response-structure-for-different-llm-apis-2b8645028b41)

---

## 5. フォーマット差分比較表

### 5-1. 基本構造の差分

| 項目 | Anthropic | OpenAI | Gemini |
|--|--|--|--|
| systemの場所 | 独立フィールド`system` | `messages[]`の要素 | 独立`systemInstruction` |
| アシスタントロール | `"assistant"` | `"assistant"` | `"model"` |
| メッセージ配列 | `messages[]` | `messages[]` | `contents[]` |
| コンテンツ構造 | `content[]`（ブロック配列） | `content`（文字列 or 配列） | `parts[]` |

### 5-2. Tool Use関連の差分

| 項目 | Anthropic | OpenAI | Gemini |
|--|--|--|--|
| ツール定義キー | `input_schema` | `function.parameters` | `functionDeclarations` |
| ツール引数形式 | オブジェクト（`input: {...}`） | **文字列化JSON**（`arguments: "..."`） | オブジェクト（`args: {...}`） |
| ツール結果のrole | `"user"` + `type:"tool_result"` | `"tool"` | `"user"` + `functionResponse` |
| 停止理由 | `stop_reason: "tool_use"` | `finish_reason: "tool_calls"` | `finishReason` |

### 5-3. パラメータ名の差分

| 概念 | Anthropic | OpenAI | Gemini |
|--|--|--|--|
| 最大トークン | `max_tokens` | `max_tokens` | `maxOutputTokens` |
| 停止シーケンス | `stop_sequences` | `stop` | `stopSequences` |
| ツール選択 | `tool_choice` | `tool_choice` | `tool_config` |

### 5-4. 相性問題のポイント

| 問題 | 原因 | 対策 |
|--|--|--|
| `arguments`のパース失敗 | OpenAIは文字列化JSON、Anthropicはオブジェクト | ライブラリで吸収 or 条件分岐 |
| `role:"tool"`が未定義 | AnthropicにはないロールをOpenAI形式で送る | API形式ごとに変換 |
| `"model"` vs `"assistant"` | Geminiのロール名が異なる | マッピング処理を追加 |

---

## 付録：API統一化ライブラリ

フォーマット差分を吸収するライブラリ。

| ライブラリ | 概要 |
|--|--|
| [LiteLLM](https://docs.litellm.ai/) | 100以上のLLM APIを統一インターフェースで呼ぶ |
| [LangChain](https://python.langchain.com/docs/) | `@tool`デコレータで各社形式に自動変換 |
| [Amazon Bedrock Converse API](https://docs.aws.amazon.com/bedrock/latest/userguide/conversation-inference.html) | 各社モデルのtool_use形式を吸収 |

---

## 6. マルチモーダルAPI形式

テキスト以外の入力（画像・音声・動画）を送る方法。

### 6-1. 画像の送信形式

#### Anthropic

```json
{
  "role": "user",
  "content": [
    {
      "type": "image",
      "source": {
        "type": "base64",
        "media_type": "image/jpeg",
        "data": "/9j/4AAQSkZJRg..."
      }
    },
    {"type": "text", "text": "この画像は何ですか？"}
  ]
}
```

#### OpenAI

```json
{
  "role": "user",
  "content": [
    {
      "type": "image_url",
      "image_url": {"url": "https://example.com/image.jpg"}
    },
    {"type": "text", "text": "この画像は何ですか？"}
  ]
}
```

#### Gemini

```json
{
  "role": "user",
  "parts": [
    {"inlineData": {"mimeType": "image/jpeg", "data": "/9j/4AAQSkZJRg..."}},
    {"text": "この画像は何ですか？"}
  ]
}
```

### 6-2. ファイル転送方式

| 方式 | 概要 | Anthropic | OpenAI | Gemini |
|--|--|--|--|--|
| **Base64埋め込み** | JSONにエンコードして直接渡す | 対応 | 対応 | 対応 |
| **URL参照** | 画像URLを文字列で渡す | 対応 | 対応 | 対応 |
| **Files API** | 事前アップロード→IDで参照 | Beta対応 | 対応 | 対応 |

#### Files API の流れ

```
1. POST /files でファイルをアップロード → file_id を取得
2. messages[] でfile_idを参照
3. セッション終了後も再利用可能
```

### 6-3. 動画・音声の対応状況

| メディア | Anthropic | OpenAI | Gemini |
|--|--|--|--|
| 画像 | 対応 | 対応 | 対応 |
| 音声 | 非対応 | 対応（Realtime API） | 対応 |
| 動画 | 非対応 | 非対応 | 対応（YouTube URL可） |
| PDF | Beta対応 | 対応 | 対応 |

**Geminiの独自点**: YouTube URLを直接`fileData`に渡して動画を理解できる。

#### 公式リファレンス

- [Anthropic Vision ガイド](https://docs.anthropic.com/en/docs/build-with-claude/vision)
- [Anthropic Files API（Beta）](https://docs.anthropic.com/en/docs/build-with-claude/files)
- [OpenAI Vision ガイド](https://platform.openai.com/docs/guides/vision)
- [OpenAI Files API](https://platform.openai.com/docs/api-reference/files)
- [Gemini Multimodal ガイド](https://ai.google.dev/gemini-api/docs/vision)
- [Gemini File API](https://ai.google.dev/gemini-api/docs/files)

---

## 7. 完全版JSON（全部のせリクエスト例）

ツール宣言、マルチモーダル、Extended Thinking、Prompt Cachingを全て含むリクエスト例。

### Anthropic完全版

```json
{
  "model": "claude-sonnet-4-6",
  "max_tokens": 8000,
  "system": [
    {
      "type": "text",
      "text": "You are a helpful assistant.",
      "cache_control": {"type": "ephemeral"}
    }
  ],
  "thinking": {
    "type": "enabled",
    "budget_tokens": 5000
  },
  "tools": [
    {
      "name": "get_weather",
      "description": "指定した都市の天気を取得する",
      "input_schema": {
        "type": "object",
        "properties": {
          "city": {"type": "string", "description": "都市名"}
        },
        "required": ["city"]
      }
    }
  ],
  "tool_choice": {"type": "auto"},
  "messages": [
    {
      "role": "user",
      "content": [
        {
          "type": "image",
          "source": {
            "type": "base64",
            "media_type": "image/jpeg",
            "data": "..."
          }
        },
        {
          "type": "text",
          "text": "この画像の場所の天気を調べて"
        }
      ]
    }
  ]
}
```

### OpenAI完全版

```json
{
  "model": "gpt-4o",
  "max_tokens": 8000,
  "messages": [
    {"role": "system", "content": "You are a helpful assistant."},
    {
      "role": "user",
      "content": [
        {
          "type": "image_url",
          "image_url": {"url": "https://example.com/image.jpg"}
        },
        {
          "type": "text",
          "text": "この画像の場所の天気を調べて"
        }
      ]
    }
  ],
  "tools": [
    {
      "type": "function",
      "function": {
        "name": "get_weather",
        "description": "指定した都市の天気を取得する",
        "parameters": {
          "type": "object",
          "properties": {
            "city": {"type": "string", "description": "都市名"}
          },
          "required": ["city"]
        }
      }
    }
  ],
  "tool_choice": "auto",
  "stream": true
}
```

### 各フィールドの位置（層対応）

| フィールド | 位置 | 層 |
|--|--|--|
| `model` | リクエストルート | 第1層 |
| `system` / `messages` | リクエストルート | 第1層（構造）+ 第2層IN（内容） |
| `tools[]` | リクエストルート | 第1層（構造）+ 第2層IN（宣言） |
| `tool_choice` | リクエストルート | 第2層IN |
| `thinking` | リクエストルート | 第1層 |
| `cache_control` | system/content内 | 第1層 |
| `max_tokens` | リクエストルート | 第1層 |
| `stream` | リクエストルート | 第1層 |

---

## 参考リンク

### 公式ドキュメント

- [Anthropic Messages API](https://docs.anthropic.com/en/api/messages)
- [OpenAI Chat Completions API](https://platform.openai.com/docs/api-reference/chat)
- [Google Gemini API](https://ai.google.dev/gemini-api/docs)

### 解説ブログ

- [Structured Output Comparison: OpenAI / Gemini / Anthropic / Mistral / Bedrock](https://medium.com/@rosgluk/structured-output-comparison-across-popular-llm-providers-openai-gemini-anthropic-mistral-and-1a5d42fa612a)
- [AI API Comparison 2024: Anthropic vs Google vs OpenAI](https://big-agi.com/blog/ai-api-comparison-2024-anthropic-vs-google-vs-openai)
- [LLM API Tool Comparison: OpenAI vs Claude vs Grok vs Gemini](https://apimagic.ai/blog/llm-tool-comparison)
- [Cloudflare "Code Mode: the better way to use MCP"](https://blog.cloudflare.com/code-mode/)

---

*作成日: 2026-02-22*
*対象: 5層モデル 第2層（通信層）*
