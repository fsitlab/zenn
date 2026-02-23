# 第2層（通信層）ファクトチェック結果

**検証日**: 2026-02-22
**対象ファイル**: `output/02_通信層.md`

---

## 1. OpenAI API形式とAnthropic API形式の違い（arguments文字列化等）

### 記載内容（02_通信層.md）

> **OpenAI**: `arguments`は**文字列化JSON**（パース必要）
> **Anthropic**: `input`はオブジェクト（パース不要）
> **Gemini**: `args`はオブジェクト

### 検証結果: **正確**

#### OpenAI API
- 公式ドキュメント確認: `function.arguments` は "A JSON string of the arguments to pass to the function" と明記
- ストリーミング時も文字列としてチャンク化されて送信される
- Source: [Function calling | OpenAI API](https://developers.openai.com/api/docs/guides/function-calling/)

#### Anthropic API
- `input` フィールドは「A JSON object containing the parameters Claude generated based on the input_schema」
- オブジェクト形式で直接返却される
- Source: [How to implement tool use - Claude API Docs](https://platform.claude.com/docs/en/agents-and-tools/tool-use/implement-tool-use)

#### Gemini API
- `args` はオブジェクト形式（`parts[].functionCall.args`）
- Source: [Using Tools & Agents with Gemini API](https://ai.google.dev/gemini-api/docs/tools)

---

## 2. system/user/assistantロールの仕様

### 記載内容（02_通信層.md）

> **Anthropic**: `system`が独立フィールド
> **OpenAI**: `system`は`messages[]`の1要素
> **Gemini**: `systemInstruction`が独立フィールド、アシスタントロールが`"model"`

### 検証結果: **正確**

#### Anthropic API
- `system` パラメータは `messages` 配列の外に独立して配置
- "You can assign a system role to Claude models by using the 'system' parameter outside of the messages array"
- Source: [Anthropic Claude Messages API - Amazon Bedrock](https://docs.aws.amazon.com/bedrock/latest/userguide/model-parameters-anthropic-claude-messages.html)

#### OpenAI API
- "Each message in the list has two properties: role and content. The 'role' can take one of three values: 'system', 'user' or the 'assistant'"
- system は messages 配列内の1要素として配置
- Source: [Moving from Completions to Chat Completions | OpenAI Help Center](https://help.openai.com/en/articles/7042661-moving-from-completions-to-chat-completions-in-the-openai-api)

#### Gemini API
- `systemInstruction` は独立フィールド
- アシスタントロールは `"model"` を使用
- Source: [Generate content with the Gemini API](https://docs.cloud.google.com/vertex-ai/generative-ai/docs/model-reference/inference)

---

## 3. tool_choiceパラメータの仕様

### 記載内容（02_通信層.md）

| 値 | 動作 | Anthropic | OpenAI | Gemini |
|--|--|--|--|--|
| auto | LLMが判断 | `{"type": "auto"}` | `"auto"` | `mode: "AUTO"` |
| required | 必ず呼ぶ | `{"type": "any"}` | `"required"` | `mode: "ANY"` |
| 特定ツール | 指定ツールを呼ぶ | `{"type": "tool", "name": "..."}` | `{"type": "function", "function": {"name": "..."}}` | `allowed_function_names: [...]` |
| none | 呼ばない | `{"type": "none"}` | `"none"` | `mode: "NONE"` |

### 検証結果: **正確**

#### OpenAI API
- `auto`: モデルが判断（デフォルト）
- `required`: "tool_choice: 'required'" で強制
- `none`: ツール呼び出しを禁止
- 特定ツール指定: `{"type": "function", "function": {"name": "..."}}`
- Source: [New API feature: forcing function calling via `tool_choice: "required"`](https://community.openai.com/t/new-api-feature-forcing-function-calling-via-tool-choice-required/731488)

#### Anthropic API
- `auto`: Claude が判断（デフォルト）
- `any`: いずれかのツールを必ず呼ぶ
- `tool`: 特定ツールを強制
- `none`: ツール呼び出しを禁止（2025年2月追加）
- **注意**: Extended Thinking 使用時は `any` と `tool` は使用不可
- Source: [How to implement tool use - Claude API Docs](https://docs.anthropic.com/en/docs/build-with-claude/tool-use/implement-tool-use)

#### Gemini API
- `AUTO`: モデルが判断（デフォルト）
- `ANY`: 関数呼び出しを強制
- `NONE`: 関数呼び出しを禁止
- `allowed_function_names`: 特定関数に限定
- Source: [Using Tools & Agents with Gemini API](https://ai.google.dev/gemini-api/docs/tools)

---

## 4. マルチモーダルAPI（画像、PDF、音声）の形式

### 4-1. 画像の送信形式

### 記載内容（02_通信層.md）

> **Anthropic**: Base64埋め込み（対応）、URL参照（非対応）
> **OpenAI**: Base64埋め込み（対応）、URL参照（対応）
> **Gemini**: Base64埋め込み（対応）、URL参照（対応）

### 検証結果: **部分的に古い情報**

#### 発見した問題

| # | 項目 | 問題 | 優先度 |
|---|------|------|--------|
| 31 | Anthropic URL参照 | **現在は対応済み**（2025年2〜3月頃追加）。`"source": {"type": "url", "url": "..."}` 形式でURL参照可能 | **高** |

#### Anthropic API（更新が必要）
- **Base64**: 対応（`"type": "base64"`, `media_type`, `data`）
- **URL参照**: **対応**（2025年2〜3月頃追加）
  - `"source": {"type": "url", "url": "https://example.com/image.jpg"}`
- **Files API**: Beta対応（`files-api-2025-04-14` ヘッダー必要）
- Source: [llm-anthropic #24: Use new URL parameter to send attachments](https://simonwillison.net/2025/Mar/1/llm-anthropic/)

#### OpenAI API（正確）
- **Base64**: 対応（`data:image/jpeg;base64,${base64Image}` 形式）
- **URL参照**: 対応
- **Files API**: 対応
- Source: [Images and vision | OpenAI API](https://developers.openai.com/api/docs/guides/images-vision/)

#### Gemini API（正確）
- **Base64**: 対応（`inlineData` フィールド）
- **URL参照**: 対応
- **Files API**: 対応
- Source: [Video understanding | Gemini API](https://ai.google.dev/gemini-api/docs/video-understanding)

### 4-2. PDF対応

### 記載内容（02_通信層.md）

> **Anthropic**: Beta対応
> **OpenAI**: 対応
> **Gemini**: 対応

### 検証結果: **正確**

- OpenAI: 直接PDF入力対応。テキストと各ページの画像を抽出してモデルに渡す
- Anthropic: Beta対応
- Gemini: 対応
- Source: [File inputs | OpenAI API](https://developers.openai.com/api/docs/guides/pdf-files/)

### 4-3. 動画・音声対応

### 記載内容（02_通信層.md）

> **Anthropic**: 音声（非対応）、動画（非対応）
> **OpenAI**: 音声（対応・Realtime API）、動画（非対応）
> **Gemini**: 音声（対応）、動画（対応・YouTube URL可）

### 検証結果: **概ね正確だが補足推奨**

#### 発見した問題

| # | 項目 | 問題 | 優先度 |
|---|------|------|--------|
| 32 | Anthropic音声API | 補足推奨: Claude Voice機能（`/v1/audio/stream` API、WebSocket対応）の記載がない。ただし、これはMessages APIではなく別APIなので、現在の記載でも誤りではない | 低 |
| 33 | Gemini WebSocket | 補足推奨: Gemini Live API（WebSocket）の記載がない | 低 |

#### 補足情報

**Anthropic Voice API**（別途追記推奨）:
- `/v1/audio/stream` エンドポイントでWebSocket対応
- PCM/Opus音声ストリーミング可能
- claude.aiアプリでは音声対話可能

**Gemini Live API**（別途追記推奨）:
- WebSocket対応の双方向リアルタイムAPI
- エンドポイント: `wss://generativelanguage.googleapis.com/ws/google.ai.generativelanguage.v1beta.GenerativeService.BidiGenerateContent`
- 音声・動画のリアルタイム処理可能

---

## 5. ストリーミングレスポンスの仕様

### 記載内容（02_通信層.md）

#### SSEイベント形式
**Anthropic**:
```
event: content_block_delta
data: {"type":"content_block_delta","index":0,"delta":{"type":"text_delta","text":"Hello"}}
```

**OpenAI**:
```
data: {"choices":[{"delta":{"content":"Hello"}}]}
data: [DONE]
```

**Gemini**:
```
data: {"candidates":[{"content":{"parts":[{"text":"Hello"}]}}]}
```

### 検証結果: **正確**

#### Anthropic SSE
- `event:` 行でイベントタイプを指定
- `content_block_delta` イベントで増分テキストを送信
- `message_stop` イベントで終了
- Delta types: `text_delta`, `thinking_delta`, `input_json_delta`, `signature_delta`
- Source: [Streaming Messages - Claude API Docs](https://platform.claude.com/docs/en/build-with-claude/streaming)

#### OpenAI SSE
- `data:` プレフィックスのみ（`event:` 行なし）
- `data: [DONE]` で終了シグナル
- Source: [Streaming events | OpenAI API Reference](https://platform.openai.com/docs/api-reference/responses-streaming)

#### Gemini SSE
- `alt=sse` パラメータでSSEモード有効化
- `candidates[0].content.parts[0].text` にテキスト
- Source: [Gemini API: Streaming Quickstart with REST](https://github.com/google-gemini/cookbook/blob/main/quickstarts/rest/Streaming_REST.ipynb)

### WebSocket対応

### 記載内容（02_通信層.md）

| プロバイダー | SSE | WebSocket |
|--|--|--|
| Anthropic | 対応 | 非対応 |
| OpenAI | 対応（Chat Completions） | 対応（Realtime API） |
| Gemini | 対応 | 非対応 |

### 検証結果: **部分的に古い情報**

| # | 項目 | 問題 | 優先度 |
|---|------|------|--------|
| 34 | Anthropic WebSocket | `/v1/audio/stream` でWebSocket対応あり。ただしMessages APIとは別API | 低 |
| 35 | Gemini WebSocket | **Gemini Live APIでWebSocket対応**。双方向リアルタイム通信可能 | **中** |

#### 更新推奨

| プロバイダー | SSE | WebSocket |
|--|--|--|
| Anthropic | 対応 | 対応（Voice API） |
| OpenAI | 対応（Chat Completions） | 対応（Realtime API） |
| Gemini | 対応 | **対応（Live API）** |

---

## 6. CLAUDE.mdの仕様（Claude Code）

### 記載内容

02_通信層.md には CLAUDE.md の直接的な記載なし（これは第3層・第4層の話題）

### 検証結果: **N/A（該当なし）**

CLAUDE.md は通信層ではなく、Claude Code固有の設定ファイル。以下の情報は参考として記載:

- ファイル名は大文字小文字を区別: `CLAUDE.md`（大文字）
- Markdown形式
- プロジェクトルートまたは親ディレクトリに配置
- 内容はシステムプロンプトに自動注入される
- Source: [Using CLAUDE.MD files: Customizing Claude Code for your codebase](https://claude.com/blog/using-claude-md-files)

---

## 発見した問題のまとめ

| # | 行番号 | 問題の種類 | 詳細 | 優先度 |
|---|--------|-----------|------|--------|
| 31 | 543-544 | **事実誤り** | Anthropic URL参照「非対応」→ **対応済み**（2025年2-3月追加） | **高** |
| 32 | 556-561 | 情報不足 | Anthropic音声APIの補足記載がない | 低 |
| 33 | 374-375 | 情報不足 | Gemini Live API（WebSocket）の記載がない | 低 |
| 34 | 374 | 表記更新推奨 | WebSocket対応表: Anthropic「非対応」→「対応（Voice API）」 | 低 |
| 35 | 375 | **事実誤り** | WebSocket対応表: Gemini「非対応」→ **「対応（Live API）」** | **中** |

---

## 推奨アクション

### 高優先度

1. **Anthropic URL参照の修正** (#31)
   ```
   修正箇所: 543-544行目のファイル転送方式表
   修正前: URL参照 | 非対応 | 対応 | 対応
   修正後: URL参照 | 対応 | 対応 | 対応
   ```
   補足として「2025年2-3月頃追加」「`source.type: "url"` 形式」を追記推奨

### 中優先度

2. **Gemini WebSocket対応の修正** (#35)
   ```
   修正箇所: 374-375行目の各社の対応表
   修正前: Gemini | 対応 | 非対応
   修正後: Gemini | 対応 | 対応（Live API）
   ```

3. **ストリーミングセクションの補足**
   - Gemini Live API（WebSocket）の説明を追加
   - エンドポイント: `wss://generativelanguage.googleapis.com/ws/...`

### 低優先度

4. Anthropic Voice API（WebSocket）の補足記載 (#32, #34)
5. 音声・動画対応表の精緻化 (#33)

---

## ソース一覧

### 公式ドキュメント

- [Function calling | OpenAI API](https://developers.openai.com/api/docs/guides/function-calling/)
- [How to implement tool use - Claude API Docs](https://platform.claude.com/docs/en/agents-and-tools/tool-use/implement-tool-use)
- [Using Tools & Agents with Gemini API](https://ai.google.dev/gemini-api/docs/tools)
- [Images and vision | OpenAI API](https://developers.openai.com/api/docs/guides/images-vision/)
- [Vision - Claude API Docs](https://platform.claude.com/docs/en/build-with-claude/vision)
- [File inputs | OpenAI API](https://developers.openai.com/api/docs/guides/pdf-files/)
- [Video understanding | Gemini API](https://ai.google.dev/gemini-api/docs/video-understanding)
- [Streaming Messages - Claude API Docs](https://platform.claude.com/docs/en/build-with-claude/streaming)
- [Streaming events | OpenAI API Reference](https://platform.openai.com/docs/api-reference/responses-streaming)
- [Realtime API with WebSocket | OpenAI API](https://platform.openai.com/docs/guides/realtime-websocket)
- [Live API - WebSockets API reference | Gemini API](https://ai.google.dev/api/live)
- [Using CLAUDE.MD files | Claude](https://claude.com/blog/using-claude-md-files)

### 解説記事

- [llm-anthropic #24: Use new URL parameter to send attachments](https://simonwillison.net/2025/Mar/1/llm-anthropic/)
- [How streaming LLM APIs work | Simon Willison](https://til.simonwillison.net/llms/streaming-llm-apis)
- [Gemini API: Streaming Quickstart with REST](https://github.com/google-gemini/cookbook/blob/main/quickstarts/rest/Streaming_REST.ipynb)

---

*検証日: 2026-02-22*
*検証者: Claude Code*
