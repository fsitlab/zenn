# AIエージェント概念マップ完全版：JSONからAgentsまで一気通貫で理解する
## ～「相性問題」はなぜ起きるのか、プロトコルと責任境界から読み解く～

> **この記事の目的**
> LLMへの入出力JSONから出発し、RAG・Tool Use・MCP・CLAUDE.md・Skill・Hooks・Agents・Agent Teamsまで、
> 各概念が**どの層に属し**、**何がLLMの仕事で何がプログラムの仕事か**を包含関係として整理する。
> 「相性問題」として片付けられがちな現象が、なぜプロトコル・責任境界の話に帰着するかを明らかにする。
> 
> **方針**: 良質な公式ドキュメント・既存記事へのリンクを最大活用。本文は「つなぎ」と「なぜ」に集中。
> **主軸**: Claude / Claude Code（2026年2月時点）。他実装は対比・補足として言及。

---

## 目次（章立て）

```
第0章  この記事の読み方と3層モデル（全体地図）
第1章  LLMへの入出力：JSON通信プロトコルの実態
第2章  Tool Use：LLMが「実行しない」という大原則
第3章  RAG：ツールではなくプロンプト操作だった
第4章  MCP：ツール定義を標準化・外部化するプロトコル
第5章  CLAUDE.md / AGENTS.md：プロンプト注入の設定ファイル
第6章  Skill：動的プロンプト拡張と「コンテキスト節約」
第7章  Hooks：プログラムが担保する決定論的制御
第8章  Subagent / Agents：Tool Useの応用として別セッションを起動する
第9章  Agent Teams：エージェント間通信プロトコルの話
第10章  プラグイン：上記すべての配布単位
第11章  権限管理：②層の設計比較（Claude/Codex/AgentCore）
第12章  全体俯瞰：「相性問題」はどこで起きるか
```

---

## 第0章　この記事の読み方と3層モデル（全体地図）

### 解説すべき項目

- **3層モデルの導入**（本記事の軸となる分類）
  ```
  ① LLM入出力層   JSONで入出力するだけ。実行しない。
  ② ホスト制御層  ClaudeCode等のプログラムが担保。Hooks/Permissions。
  ③ プロンプト層  system promptへの注入。Skills/CLAUDE.md/RAG。
  ```
- 各章がどの層の話かを冒頭に常に示す
- 「相性問題」の正体：プロトコルのフォーマット差 or 責任境界の認識違い
- 図：全概念の包含関係マップ（本文中に1枚）

### リンク先
- この記事自身が「地図」になる。外部リンク不要。

---

## 第1章　LLMへの入出力：JSON通信プロトコルの実態

> **対象層**: ① LLM入出力層

### 解説すべき項目

#### 1-1. 共通構造：messagesの配列が全て
- `system` / `user` / `assistant` ロールの意味
- マルチターン会話 = 全履歴を毎回送るステートレス設計
- **なぜステートレスか**：LLMはセッションを覚えていない → これが「記憶」問題の根本

#### 1-2. Anthropic API のJSON形式
```json
// リクエスト・レスポンスの最小構造（コード例として掲載）
```
- 公式リファレンス：[Anthropic Messages API](https://docs.anthropic.com/en/api/messages)
- `content[]` がブロック配列になっている設計思想

#### 1-3. OpenAI API のJSON形式との差分
```json
// 比較表として: tool定義の入れ子・argumentsの文字列化問題
```
- `parameters` vs `input_schema`
- `arguments`（文字列） vs `input`（オブジェクト）→ **これが相性問題の1つ目**
- `role: "tool"` vs `role: "user" + type: "tool_result"`
- 公式：[OpenAI Chat Completions API](https://platform.openai.com/docs/api-reference/chat)

#### 1-4. Gemini API との差分
- `functionDeclarations` という独自命名
- マルチモーダル（画像・音声）の扱い方の違い
- 公式：[Google Gemini API](https://ai.google.dev/api/generate-content)

#### 1-5. OSSモデル（vLLM / TGI / Ollama）
- OpenAI互換 `/v1/chat/completions` で吸収
- 内部では「モデル固有のプロンプトフォーマット」に変換している
  → **相性問題の2つ目**：モデルがtool use学習しているかどうか
- LiteLLM等のラッパーが吸収する層

#### 1-6. マルチモーダル・拡張フォーマット（補足）
- 画像・PDFの `content[]` ブロック
- Anthropic Structured Outputs（`output_config.format`）
- Extended Thinking（`thinking` ブロック）

### ❌ この章では解説しないこと
- Tool Useの詳細（→第2章）
- RAG（→第3章）

---

## 第2章　Tool Use：LLMが「実行しない」という大原則

> **対象層**: ① LLM入出力層 + ② ホスト制御層の境界

### 解説すべき項目

#### 2-1. 最重要原則の宣言
- LLMは `tool_use` ブロック（JSON）を出力するだけ
- 実行するのは**常にホスト側アプリ**
- ホスト ⇄ LLM の往復プロトコル全体像（シーケンス図）

#### 2-2. リクエスト：tools[] 定義
```json
// tools[] の完全な構造
// name / description / input_schema の役割
// tool_choice: auto / any / none
```
- `description` の重要性：LLMはここを読んで「いつ使うか」を判断する
  → **descriptionが貧弱だと正しく呼ばれない（相性問題の3つ目）**

#### 2-3. レスポンス：tool_use ブロック
```json
// stop_reason: "tool_use" のとき
// content[].type: "tool_use" の構造
// id の役割（後のtool_resultとの対応付け）
```

#### 2-4. ホスト側での実行と tool_result の返し方
```json
// tool_result ブロックの完全構造
// is_error: true のケース
// 全会話履歴を引き継ぐ必要性
```

#### 2-5. 複数ツール・並列呼び出し
- 1ターンで複数の `tool_use` を返せる
- Anthropicの `tool_choice: {type: "any"}` 強制モード

#### 2-6. Programmatic Tool Calling（2026年2月GA）
- 従来方式の問題：ループごとにコンテキストが積まれる
- コードを書いてサンドボックス実行 → 最終結果だけ返ってくる
- トークン削減の実績（150,000→2,000の事例）
- 参考：[Anthropic Advanced Tool Use発表](https://www.anthropic.com/engineering/advanced-tool-use)

#### 2-7. 他実装との比較（簡略）
| | Anthropic | OpenAI | Gemini |
|--|--|--|--|
| ツール定義 | `input_schema` | `function.parameters` | `functionDeclarations` |
| 引数 | オブジェクト | 文字列化JSON | オブジェクト |
| 結果のrole | user | tool | user |

---

## 第3章　RAG：ツールではなくプロンプト操作だった

> **対象層**: ③ プロンプト層（従来RAG）/ ① + ② （Tool型RAG）

### 解説すべき項目

#### 3-1. 従来RAGの正体
- ユーザー入力を機械的にベクトル検索 → 結果をプロンプトに埋め込む
- LLMは「なぜ文書が来たか」を知らない
- 検索クエリ = ユーザー入力のコピー（LLMは関与しない）
- 1回しか検索しない制約

#### 3-2. 従来RAGの限界が「Tool型RAG」を生んだ
- 複数クエリ・反復検索の必要性
- 「A社とB社を比較して」問題
- LLMが検索戦略を自律的に立てられるようになる転換点

#### 3-3. Tool型RAGの実装
```json
// search_documents ツール定義の例
```
- LLMがクエリを自分で生成する
- 検索結果を見て再検索できる（Agentic RAG / ReAct パターン）

#### 3-4. RAGは「ツールか否か」の整理
| | 従来RAG | Tool型RAG |
|--|--|--|
| 検索の主体 | ホスト（機械的） | LLM（自律的） |
| LLMの認識 | 知らない | 自分が呼んだ |
| プロトコル | ただのprompt操作 | tool_use/tool_result |

#### 3-5. 参考リンク
- 良質なRAG解説記事へのリンク（外部）
- Agentic RAG / Langchain / LlamaIndex の実装例へのリンク

---

## 第4章　MCP：ツール定義を標準化・外部化するプロトコル

> **対象層**: ① LLM入出力層 の拡張（標準化レイヤー）

### 解説すべき項目

#### 4-1. MCPの概念（本章では解説しない、リンクのみ）
- [MCP公式サイト](https://modelcontextprotocol.io/)
- [Anthropic MCP紹介記事](https://www.anthropic.com/news/model-context-protocol)
- ★良質な日本語解説記事へのリンク（外部）

#### 4-2. MCPが解決した問題
- ツール定義を毎回APIリクエストに書かなくていい
- サーバーが仕様を公開 → クライアントが自動取得
- N×N の統合問題 → N+N（USB-C比喩）

#### 4-3. ローカルMCP vs リモートMCP
```
ローカル: stdio transport（プロセス間通信）
  Claude Desktopなど。コマンドとして起動
  
リモート: HTTP/SSE transport（Streamable HTTP）
  Enterprise用。ネットワーク越し
  認証（OAuth 2.0）が必要
```
- `.mcp.json` の構造（プロジェクト単位の設定）

#### 4-4. MCPがJSON通信でどう動くか（内部）
```json
// JSON-RPC 2.0 メッセージの例
// tools/list → tools/call の往復
// LLMへの渡し方は変わらない（最終的にはtools[]に展開される）
```
- MCPはあくまでトランスポート層。LLMが受け取るJSONは同じ
- **相性問題の4つ目**：MCPサーバーのスキーマが不正だとLLMが正しく呼べない

#### 4-5. 各ツールでのMCP設定ファイルの場所
| ツール | 設定ファイル |
|--|--|
| Claude Code | `.mcp.json` (プロジェクト) / `~/.claude/` (グローバル) |
| OpenAI Codex | 未対応（MCP ServerとしてCodexを公開する形） |
| Cursor | `.cursor/mcp.json` |
| Amazon Q CLI | `~/.aws/amazonq/mcp.json` |
| Amazon AgentCore | Gateway経由でMCP互換ツールとして公開 |

---

## 第5章　CLAUDE.md / AGENTS.md：プロンプト注入の設定ファイル

> **対象層**: ③ プロンプト層

### 解説すべき項目

#### 5-1. CLAUDE.md の本質
- セッション開始時に **system prompt に自動注入**されるMarkdown
- LLMはAPIJSONレベルでは「長いsystem promptが来た」だけ
- → ③層であって①②層ではない

#### 5-2. ディレクトリ階層と優先度
```
優先度（具体的なものが勝つ）:
  ./src/auth/CLAUDE.md   ← 最高（モジュール層）
  ./CLAUDE.md            ← 中（プロジェクト層）
  ~/.claude/CLAUDE.md    ← 最低（ユーザー層）

ロードのタイミングの違い:
  作業ディレクトリ以上 → 起動時に全部ロード
  子ディレクトリ       → そのディレクトリのファイルを読んだときにオンデマンドロード
```
- `.claude/rules/` ：パス条件付きルールファイル（paths: frontmatter）
- `CLAUDE.local.md` ：.gitignore推奨の個人設定
- 公式：[Claude Code Memory管理](https://code.claude.com/docs/en/memory)

#### 5-3. @import 構文
```markdown
@~/.claude/CLAUDE.md          # 個人設定のインポート
@docs/api-guidelines.md       # 外部ファイルのインポート
```

#### 5-4. OpenAI Codex の AGENTS.md との比較
| 項目 | Claude Code / CLAUDE.md | OpenAI Codex / AGENTS.md |
|--|--|--|
| ファイル名 | CLAUDE.md（またはAGENTS.md） | AGENTS.md（またはAGENTS.override.md） |
| グローバル | `~/.claude/CLAUDE.md` | `~/.codex/AGENTS.md` |
| 優先度 | 深い階層が勝つ | 近い階層が勝つ（closest wins） |
| スキル | `.claude/skills/` | `.agents/skills/` |

- AGENTS.md はLinux Foundation傘下の[Agentic AI Foundationが標準化](https://agents.md/)
- Cursor / Amp / Google Jules / Factory も採用

#### 5-5. 他ツールの対応表
| ツール | コンテキストファイル | 場所 |
|--|--|--|
| Claude Code | CLAUDE.md | プロジェクトルート / .claude/ |
| OpenAI Codex | AGENTS.md | プロジェクトルート / ~/.codex/ |
| Cursor | .cursor/rules/ | プロジェクト |
| GitHub Copilot | .github/copilot-instructions.md | プロジェクト |
| Amazon Q | 未定（MCP経由が主） | - |

---

## 第6章　Skill：動的プロンプト拡張と「コンテキスト節約」

> **対象層**: ③ プロンプト層（ただし `context: fork` でSubagentに移行可能）

### 解説すべき項目

#### 6-1. Skillとは何か（本質）
- 使用時にだけロードされるプロンプトのモジュール
- 起動時は `name` と `description` だけ読み込む（~100トークン/Skill）
- タスクが来たとき `description` でマッチしたら `SKILL.md` 全体をロード
- → **Progressive Disclosure**（文脈節約の設計思想）

#### 6-2. SKILL.md の構造
```yaml
---
name: pdf-handler
description: PDFを扱うとき（作成・読み込み・変換）。Use when...
allowed-tools: "Bash, Read, Write"
context: fork          # これがあるとSubagentとして実行（③→①②へ移行）
agent: code-reviewer   # どのSubagentに渡すか
version: 1.0.0
---
# ここからがプロンプト本文
## 手順
1. まずライブラリを確認する...
```

#### 6-3. SkillとToolの根本的な違い
| | Tool (tool_use) | Skill |
|--|--|--|
| LLMからの視点 | 「このツールを呼ぶ」宣言 | 「こういう手順で解く」知識 |
| プロトコル | ① JSON往復 | ③ プロンプト注入 |
| 実行主体 | ホスト側プログラム | LLM自身 |
| 副作用 | ファイル操作・API呼び出し | なし（思考・手順のみ） |

#### 6-4. ClaudeとOpenAI Codexでのディレクトリ比較
```
Claude Code:
  .claude/skills/<skill-name>/SKILL.md
  ~/.claude/skills/<skill-name>/SKILL.md  （ユーザーグローバル）

OpenAI Codex:
  .agents/skills/<skill-name>/SKILL.md
  ~/.codex/skills/<skill-name>/SKILL.md
```
- 起動方法：`/skill-name`（明示）or 自動マッチ（暗黙）

#### 6-5. `context: fork` で①②層に接続する（設計の橋渡し）
- この1行でSkillはSubagentの起動になる
- → 第8章（Subagent）との連続性

---

## 第7章　Hooks：プログラムが担保する決定論的制御

> **対象層**: ② ホスト制御層（最重要）

### 解説すべき項目

#### 7-1. なぜHooksが必要か
- LLMへの指示（③層）は確率的 → 「必ず実行」できない
- **Hooksは確率ゼロの制御**（シェルスクリプトを確実に実行）
- 「lintを必ず走らせたい」「mainブランチへの書き込みを禁止したい」→ promptでなくHooks

#### 7-2. Hooksのイベント一覧（14種）
```
PreToolUse     ← ★最高優先。ブロック可能。tool_use受け取り直後
PostToolUse    ← 実行後のログ・コンテキスト注入
SessionStart / SessionEnd
UserPromptSubmit
SubagentStart / SubagentStop
Notification / Stop / TaskCompleted
PreCompact / TeammateIdle
```

#### 7-3. PreToolUse が「最高優先」な理由
- `settings.json` の permissions設定よりも優先
- exit code 2 でブロック、メッセージをフィードバック
- → **これが②層のアーキテクチャ的な意味**

#### 7-4. Hooks の設定場所と書き方
```json
// .claude/settings.json の構造例
{
  "hooks": {
    "PreToolUse": [{
      "matcher": "Bash",
      "hooks": [{
        "type": "command",
        "command": "./scripts/validate-command.sh"
      }]
    }]
  }
}
```

#### 7-5. 他実装との比較
| | Claude Code Hooks | OpenAI Codex | Amazon AgentCore |
|--|--|--|--|
| 実行タイミング制御 | 14種のイベント | なし（現状） | Policy（Cedar policy）|
| ブロック機能 | PreToolUseのexit 2 | なし | Policy enforcement |
| 実装方式 | シェルスクリプト | - | Cedar DSL |

---

## 第8章　Subagent / Agents：Tool Useの応用として別セッションを起動する

> **対象層**: ① ② ③ 全層の集大成

### 解説すべき項目

#### 8-1. SubagentはTaskツールのtool_useから始まる
```json
// LLMが返すtool_useの例
{
  "type": "tool_use",
  "name": "Task",
  "input": {
    "description": "コードレビュー",
    "prompt": "...",
    "subagent_type": "code-reviewer"
  }
}
```
- ClaudeCodeがこれを受け取り **新しいAPIリクエストを立ち上げる**
- `.claude/agents/code-reviewer.md` の内容がsystem promptになる
- 完了したら `tool_result` として元セッションに返る

#### 8-2. `.claude/agents/*.md` の構造
```yaml
---
name: code-reviewer
description: コードレビュー。reivew/lint/securityを含むタスクで使用
tools: Read, Glob, Grep          # ① 層の制限
disallowedTools: Write, Bash     # 書き込み禁止
model: claude-haiku-4-5          # コスト最適化
permissionMode: default          # ② 層の設定
hooks:                           # ② 層（Subagent固有）
  PreToolUse: ...
skills:                          # ③ 層（Subagent固有）
  - code-review-patterns
memory: true
maxTurns: 20
---
# ここがSubagentのsystem prompt（③層）
You are a code reviewer...
```
- **1ファイルの中に①②③層が全て入っている**（この包含関係が重要）

#### 8-3. Subagentの並列実行
- 複数の `Task` tool_useを1ターンで返す → 並列起動
- Ctrl+B でバックグラウンド送り

#### 8-4. OpenAI Codex の場合
- `AGENTS.md` がSystem Prompt相当
- `.agents/skills/` でSkillを管理
- [Codex as MCPサーバーとして他SDKから呼ぶ](https://developers.openai.com/codex/guides/agents-sdk/)

#### 8-5. Amazon AgentCore の場合
- フレームワーク非依存（LangGraph / CrewAI / Strands / OpenAI SDK）
- Runtime = Subagentのホスティング基盤
- Gateway = ツール定義をMCP互換で公開
- **Claude CodeのSubagentとの本質的な違い**：インフラ管理の有無
  - ClaudeCode：ローカル実行、セッション終了で消える
  - AgentCore Runtime：スケール・8時間実行・VPC対応

#### 8-6. 参考リンク
- [Claude Code 公式 Sub-agents ドキュメント](https://code.claude.com/docs/en/sub-agents)
- [Amazon Bedrock AgentCore](https://aws.amazon.com/bedrock/agentcore/)

---

## 第9章　Agent Teams：エージェント間通信プロトコルの話

> **対象層**: ① プロトコル（エージェント間）

### 解説すべき項目

#### 9-1. 単一Agentの限界とTeamsの必然性
- コンテキスト窓の制約
- 専門化による品質向上
- 並列実行によるスピードアップ

#### 9-2. ClaudeCodeでの実装パターン
- Orchestrator + Specialist Agents 構成
- **ホワイトボード（共有ファイル）パターン**
  - 共有の `PLAN.md` / `TASKS.md` / `RESULTS.md` に書き込む
  - → 各SubagentがTaskを読んで実行し結果を書く → 協働が成立
  - 参考：[Agent Teamsにホワイトボードを置いたら分業が協働に変わった記事](リンク)

#### 9-3. A2A（Agent-to-Agent）プロトコル
- AgentCoreが採用している標準プロトコル
- Agent Card（JSON）でエージェントの能力を公告
- JSON-RPC 2.0 over HTTP/S または SSE
- LangGraph / CrewAI / Strands / OpenAI SDK をまたいで通信できる
- 参考：[Amazon Bedrock AgentCore A2A対応発表](https://www.infoq.com/news/2025/11/a2a-amazon-bedrock-agentcore/)

#### 9-4. 「相性問題」の本質：ClaudeCode Teams vs AgentCore Teams
| 観点 | ClaudeCode Teams | AgentCore Teams |
|--|--|--|
| 通信方式 | tool_result（ファイル・メモリ共有） | A2A JSON-RPC 2.0 |
| 状態共有 | ファイルシステム（同一マシン） | HTTP/ステートレス |
| スケール | 1マシン | クラウドスケール |
| フレームワーク | ClaudeCode専用 | フレームワーク非依存 |
| 永続性 | セッション終了で消える | 最大8時間・永続対応 |

---

## 第10章　プラグイン：上記すべての配布単位

> **対象層**: ③ プロンプト層を中心に全層を梱包する

### 解説すべき項目

#### 10-1. プラグインの構造（Claudeの場合）
```
my-plugin/
├── .claude-plugin/
│   └── plugin.json         # メタデータ
├── commands/               # スラッシュコマンド（③層）
├── agents/                 # Subagent定義（①②③層）
├── skills/                 # Skill定義（③層）
├── hooks/                  # Hookスクリプト（②層）
└── .mcp.json               # MCPサーバー（①層の拡張）
```

#### 10-2. プラグインとSkill/Agentの包含関係
```
Plugin（配布単位）
  ├── Skills（③プロンプト層）
  ├── Agents（①②③全層）
  │     ├── tools定義（①）
  │     ├── Hooks（②）
  │     └── system prompt（③）
  ├── Commands（③プロンプト層）
  ├── Hooks（②ホスト制御層）
  └── MCP Servers（①の拡張）
```

#### 10-3. OpenAI Codex のスキルシステム
- `.agents/skills/` + `agents/openai.yaml`（UIメタデータ）
- `allow_implicit_invocation: false` で明示呼び出し専用にできる
- Skillがツール依存を宣言できる（`dependencies.tools`）
- 参考：[Codex Skills公式](https://developers.openai.com/codex/skills/)

---

## 第11章　権限管理：②層の設計比較

> **対象層**: ② ホスト制御層

### 解説すべき項目

#### 11-1. Claude Code の権限設計
- `settings.json` の `permissions` ブロック
- `permissionMode`: default / acceptEdits / bypassPermissions / plan
- `allowedTools` / `disallowedTools` の優先度
- PreToolUse Hookが全ての上位に立つ
- Sandboxing（macOS seatbelt / Linux bubblewrap）
  - 内部Anthropicテストで1日200件→5〜10件に削減
- 公式：[Claude Code Settings](https://code.claude.com/docs/en/settings)

#### 11-2. OpenAI Codex の権限設計
- Approval prompt（exec / patch）による人間承認
- Seatbeltサンドボックス（`CODEX_SANDBOX=seatbelt`）
- ネットワークアクセス禁止がデフォルト（`CODEX_SANDBOX_NETWORK_DISABLED=1`）
- HooksのようなプログラマブルなPre/Post制御は現状なし

#### 11-3. Amazon AgentCore の権限設計
- Cedar policy による宣言的アクセス制御
- IAM + OAuth 2.0 のダブル認証
- VPC connectivity でネットワーク分離
- Fine-grained authorization（LLMではなくポリシーが判定）
- **ClaudeCodeとの本質的違い**：LLMへの不信任を前提とした設計

#### 11-4. 「何を誰が信頼するか」の設計思想の違い
```
ClaudeCode：  LLMを信頼。人間がレビュー。Hooksで例外処理。
Codex：       人間承認を頻繁に挟む。サンドボックスで封じ込め。
AgentCore：   Policyが全てを判定。LLMの判断を通さない。
```
→ これが「相性問題」の根本：どの層で安全を担保するかの哲学の違い

---

## 第12章　全体俯瞰：「相性問題」はどこで起きるか

### 解説すべき項目

#### 12-1. 相性問題の発生箇所マッピング
```
問題の種類                      発生層      原因
─────────────────────────────────────────────────
APIフォーマット不一致            ①          arguments文字列化 / roleの違い
tool descriptionが貧弱          ①          LLMがツールを使わない
モデルがtool use未学習          ①          OSSモデルの選択ミス
HookがPermissionより弱い認識    ②          PreToolUseの設計理解不足
CLAUDE.mdが巨大すぎる           ③          コンテキスト汚染・優先度喪失
Skillの description が曖昧      ③          自動マッチ失敗
Subagentのtools制限が甘い       ①②        セキュリティリスク
フレームワーク間のAgent通信     ①          A2Aプロトコル未対応
```

#### 12-2. AgentCore vs ClaudeCode：何を選ぶべきか
- ローカル・個人開発 → ClaudeCode
- エンタープライズ・マルチフレームワーク・クラウドスケール → AgentCore
- 両方を連携させる（ClaudeCodeでAgentCoreのMCPを叩く）ことも可能

#### 12-3. LangChain / Dify などフレームワーク比較へのリンク集
- ★比較記事へのリンク（外部）
- [LangChain公式](https://www.langchain.com/)
- [Dify公式](https://dify.ai/)
- [Microsoft AutoGen](https://microsoft.github.io/autogen/)
- [Google ADK](https://google.github.io/adk-docs/)
- [Strands Agents（AWS）](https://strandsagents.com/)

#### 12-4. Goose（Block Inc）について
- ClaudeCode/Codexに近いターミナル型エージェント
- MCPクライアントとして使える
- [Goose公式](https://block.github.io/goose/)

#### 12-5. 「公式ドキュメントへの誘導」の姿勢
- 本記事はリンク集。公式ドキュメントが更新されても**本記事の価値は落ちない**
- でぐれるリンクには日付を付記する（例：2026年2月時点）
- でぐれないのは「なぜこの設計か」「どの層の問題か」という概念の整理

---

## 付録A　ディレクトリ構造比較表

```
Claude Code                    OpenAI Codex
─────────────────────          ─────────────────────
.claude/                       .agents/  （または ~/.codex/）
├── settings.json              ├── AGENTS.md
├── settings.local.json        ├── AGENTS.override.md
├── agents/*.md                └── skills/<name>/SKILL.md
├── commands/*.md
├── skills/<name>/SKILL.md    Cursor
├── rules/*.md                 .cursor/
└── hooks/                     ├── rules/
                               └── mcp.json
~/.claude/
├── CLAUDE.md                  GitHub Copilot
├── rules/                     .github/
├── agents/                    └── copilot-instructions.md
└── skills/
```

## 付録B　公式ドキュメント一覧（2026年2月時点）

| サービス | ドキュメント |
|--|--|
| Claude Code | https://code.claude.com/docs/en/ |
| Anthropic API | https://docs.anthropic.com/ |
| OpenAI Codex | https://developers.openai.com/codex |
| OpenAI API | https://platform.openai.com/docs |
| Amazon AgentCore | https://aws.amazon.com/bedrock/agentcore/ |
| MCP公式 | https://modelcontextprotocol.io/ |
| AGENTS.md標準 | https://agents.md/ |
| Google ADK | https://google.github.io/adk-docs/ |
| Goose | https://block.github.io/goose/ |

---

## 未解決・要調査事項

- [ ] Gemini API の tool_choice に相当するフォーマット詳細
- [ ] OpenCode（OSSのClaudeCode代替）のディレクトリ構造
- [ ] `./codex` フォルダ vs `.agents/` フォルダの実際の動作検証
- [ ] ClaudeCode の SubagentとAgentCoreのRuntimeを連携させた場合の制限
- [ ] Dify / LangChain でのTool Use実装が各APIをどう抽象化しているか
- [ ] Gooseのpermission設計（ClaudeCode Hooksとの対比）

---

*骨子バージョン: 2026-02-21*
*主な参照: Claude Code公式docs / OpenAI Codex公式 / Amazon AgentCore公式 / 会話セッション内容*