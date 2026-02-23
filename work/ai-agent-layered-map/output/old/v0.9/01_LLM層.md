# 第1層「LLM層」詳細解説：モデルの本質から各社チューニング方針まで

> **5層モデルでの位置**: 第1層（LLM層）
> **一言**: モデルの能力そのもの。ファインチューニング、LoRA、モデル系列、各社のチューニング方針を解説する。

*作成日: 2026-02-22*

---

## 目次

```
1. ファインチューニングとLoRA
2. モデル系列の整理
3. 読めること（理解）と書けること（生成）の違い
4. 各社のチューニング方針
```

---

## 1. ファインチューニングとLoRA

### 1-1. ベースモデルとファインチューニングの関係

**一言**: ベースモデルは「何でも知っているが何も専門ではない」状態。ファインチューニングで特定ドメインに最適化する。

```
ベースモデル（Pre-trained Model）
  │
  ├── Full Fine-tuning: 全パラメータを再学習（コスト高）
  │
  └── PEFT（Parameter-Efficient Fine-Tuning）: 一部のみ学習
        ├── LoRA: 低ランク行列を追加
        ├── QLoRA: 量子化 + LoRA
        └── Adapter: 小さなネットワークを挿入
```

| 手法 | パラメータ更新量 | メモリ使用量 | 用途 |
|--|--|--|--|
| Full Fine-tuning | 100% | 高 | 大規模な方向転換 |
| LoRA | 0.1〜1% | 低 | ドメイン適応 |
| QLoRA | 0.1〜1% | 極低 | 消費者GPUでのファインチューニング |

### 1-2. LoRA（Low-Rank Adaptation）の仕組み

**一言**: 重み行列Wを直接変更する代わりに、低ランク行列A・Bを追加して「差分」だけを学習する。学習済みの知識を壊さずに新しい振る舞いを追加できる。

```
元の重み: W (d × k)
LoRAの追加: W' = W + BA
  - B: (d × r)
  - A: (r × k)
  - r << d, k （低ランク）

例: d=4096, k=4096 → 16M パラメータ
    r=8 の場合 → 65K パラメータ（0.4%）
```

**業界ごとの最適化例**:

| 業界 | ベースモデル | LoRAアダプタで追加する知識 |
|--|--|--|
| 医療 | LLaMA 3.2 | 医学論文・診療ガイドライン |
| 法律 | Mistral | 判例・法令テキスト |
| 金融 | Qwen | 財務報告書・市場用語 |
| コーディング | DeepSeek-Coder | 社内コードベース・コーディング規約 |

**アダプタの切り替え**: 同一ベースモデルに複数のLoRAアダプタをロードし、リクエストごとに切り替えることが可能。マルチテナントSaaSでの活用パターン。

### 1-3. 公式リファレンス・解説

- 📘 [HuggingFace PEFT ドキュメント](https://huggingface.co/docs/peft/) --- LoRA、QLoRA、Adapter等の実装
- 📘 [HuggingFace LoRA概念ガイド](https://huggingface.co/docs/peft/main/en/conceptual_guides/lora)
- 🔗 [LoRA: Low-Rank Adaptation of Large Language Models（原論文）](https://arxiv.org/abs/2106.09685)
- 🔗 [QLoRA: Efficient Finetuning of Quantized LLMs（原論文）](https://arxiv.org/abs/2305.14314)
- 📘 [AWS SageMaker LoRAによるファインチューニング](https://docs.aws.amazon.com/sagemaker/latest/dg/jumpstart-foundation-models-fine-tuning-lora.html)
- 📘 [Google Vertex AI Model tuning](https://cloud.google.com/vertex-ai/docs/generative-ai/models/tune-models)

---

## 2. モデル系列の整理

### 2-1. 対応モダリティによる分類

**一言**: 「テキストのみ」から「画像・動画・音声」まで、モデルによって入出力できるモダリティが異なる。APIフォーマット（第2層）の`content[]`や`parts[]`がこの差を吸収する。

| モダリティ | 説明 | 代表モデル |
|--|--|--|
| **Text-only** | テキスト入力→テキスト出力 | 初期GPT、多くのOSSモデル |
| **Vision（画像理解）** | 画像+テキスト入力→テキスト出力 | GPT-4V、claude-sonnet-4-6、Gemini |
| **画像生成** | テキスト→画像出力 | DALL-E 3、Stable Diffusion、Midjourney |
| **音声理解** | 音声入力→テキスト出力 | Whisper、Gemini（音声入力対応） |
| **音声生成（TTS）** | テキスト→音声出力 | OpenAI TTS、ElevenLabs |
| **動画理解** | 動画入力→テキスト出力 | Gemini 2.0（YouTube URL直接対応） |
| **動画生成** | テキスト→動画出力 | Sora、Runway |
| **マルチモーダル統合** | 上記を複合的に処理 | Gemini 2.0、GPT-4o |

### 2-2. 主要モデル系列（2025〜2026年）

#### クローズドソース（API提供）

| 系列 | 提供元 | 特徴 | 公式ドキュメント |
|--|--|--|--|
| **Claude** | Anthropic | 憲法AI、長文脈（200K）、Extended Thinking | [Anthropic Docs](https://docs.anthropic.com/) |
| **GPT-4系** | OpenAI | 汎用性、Function Calling先駆者 | [OpenAI Docs](https://platform.openai.com/docs/) |
| **o1/o3系** | OpenAI | 推論特化（Reasoning Tokens） | [OpenAI Reasoning](https://platform.openai.com/docs/guides/reasoning) |
| **Gemini** | Google | マルチモーダル統合、1Mコンテキスト | [Google AI Docs](https://ai.google.dev/gemini-api/docs) |

#### オープンソース / オープンウェイト

| 系列 | 提供元 | 特徴 | HuggingFace |
|--|--|--|--|
| **LLaMA系** | Meta | OSSの代名詞。LLaMA 3.x/4が主流 | [meta-llama](https://huggingface.co/meta-llama) |
| **Mistral/Mixtral** | Mistral AI | 軽量・高性能。MoEアーキテクチャ | [mistralai](https://huggingface.co/mistralai) |
| **Qwen系** | Alibaba | 多言語（日本語含む）強化 | [Qwen](https://huggingface.co/Qwen) |
| **Gemma系** | Google | Geminiの軽量OSS版 | [google/gemma](https://huggingface.co/google/gemma-2-9b) |
| **DeepSeek系** | DeepSeek | 低コスト高性能、MoE、コーディング特化版あり | [deepseek-ai](https://huggingface.co/deepseek-ai) |
| **Phi系** | Microsoft | 小型高性能（SLM） | [microsoft/phi](https://huggingface.co/microsoft/phi-4) |
| **Command R系** | Cohere | RAG特化、多言語 | [CohereForAI](https://huggingface.co/CohereForAI) |

### 2-3. MoE（Mixture of Experts）アーキテクチャ

**一言**: 全パラメータを毎回使わず、入力に応じて一部の「専門家」のみを活性化する。パラメータ数は大きいが推論コストを抑えられる。

```
入力トークン
    │
    ▼
ルーターネットワーク
    │
    ├──▶ Expert 1 （活性化）
    ├──▶ Expert 2 （活性化）
    ├──   Expert 3 （スキップ）
    └──   Expert 4 （スキップ）
    │
    ▼
出力を統合
```

| モデル | 総パラメータ | 活性化パラメータ |
|--|--|--|
| Mixtral 8x7B | 46B | 12B（2/8 experts活性化） |
| DeepSeek-V3 | 685B | 37B |

- 🔗 [Mixtral of Experts（Mistral AI公式）](https://mistral.ai/news/mixtral-of-experts/)
- 🔗 [DeepSeek-V3 Technical Report](https://arxiv.org/abs/2412.19437)

### 2-4. サービング環境とOpenAI互換

```
HuggingFace Hub（モデル置き場）
  └── Transformers（Python API）
        ├── TGI（Text Generation Inference）--- HF公式サービング
        ├── vLLM --- PagedAttentionで高スループット
        ├── Ollama --- ローカル実行向け、GGUFフォーマット
        └── llama.cpp --- C++実装の軽量サービング

全てOpenAI互換 /v1/chat/completions を提供可能
```

- 📘 [vLLM ドキュメント](https://docs.vllm.ai/)
- 📘 [Ollama ドキュメント](https://ollama.com/)
- 📘 [LiteLLM](https://docs.litellm.ai/) --- 100以上のLLM APIを統一インターフェースで呼ぶ

---

## 3. 読めること（理解）と書けること（生成）の違い

### 3-1. エンコーダーとデコーダー

**一言**: Transformerアーキテクチャには「理解」に強いエンコーダーと「生成」に強いデコーダーがある。現代のLLMはデコーダーのみ（decoder-only）が主流だが、理解タスクも生成として解くことで両方をこなす。

| アーキテクチャ | 得意タスク | 代表モデル |
|--|--|--|
| **Encoder-only** | 分類、埋め込み、理解 | BERT、RoBERTa |
| **Decoder-only** | テキスト生成、会話 | GPT系、Claude、LLaMA、Mistral |
| **Encoder-Decoder** | 翻訳、要約 | T5、BART |

### 3-2. 理解タスクと生成タスクの違い

```
理解（Understanding）:
  - 入力テキストの意味を把握
  - 分類、感情分析、情報抽出、埋め込み生成
  - 正解が「決まっている」ことが多い

生成（Generation）:
  - 新しいテキストを作り出す
  - 会話、文章作成、コード生成、翻訳
  - 正解が「複数ありうる」（確率的）
```

### 3-3. 埋め込みモデル（Embedding Models）

**一言**: 「読む」に特化したモデル。テキストを固定長ベクトルに変換し、意味的類似度を数値化する。RAG（第4章）のベクトル検索で使われる。

| モデル | 提供元 | 次元数 | 用途 |
|--|--|--|--|
| text-embedding-3-large | OpenAI | 3072 | 汎用埋め込み |
| voyage-3-large | Anthropic/Voyage AI | 1024 | Claude推奨 |
| Cohere Embed v3 | Cohere | 1024 | 多言語対応 |
| bge-m3 | BAAI（OSS） | 1024 | 多言語OSS |
| e5-mistral-7b-instruct | Microsoft（OSS） | 4096 | 高精度OSS |

- 📘 [OpenAI Embeddings ガイド](https://platform.openai.com/docs/guides/embeddings)
- 📘 [Voyage AI Embeddings](https://docs.voyageai.com/docs/embeddings)
- 🔗 [MTEB Leaderboard](https://huggingface.co/spaces/mteb/leaderboard) --- 埋め込みモデルのベンチマーク

### 3-4. 生成モデルでも「理解」はできる

**一言**: decoder-onlyモデル（GPT、Claude等）は生成タスクを通じて理解タスクも解く。「この文章の感情は？」と聞けば「ポジティブ」と生成する形で分類を行う。

```
BERT（encoder-only）の分類:
  入力 → エンコーダー → [CLS]トークンの埋め込み → 分類ヘッド → ラベル

GPT（decoder-only）の分類:
  入力 + "この文章の感情は？" → デコーダー → "ポジティブ" を生成
```

**実務上の使い分け**:
- 大量の分類タスク → 専用の分類モデルや埋め込み+分類器が効率的
- 柔軟な理解・説明が必要 → 生成モデルに自然言語で指示

---

## 4. 各社のチューニング方針

### 4-1. Anthropic：憲法AIと信頼性重視

**チューニング哲学**: 「有用で無害なAI」を目指す。憲法AI（Constitutional AI）により、人間のフィードバックだけでなく、明文化された原則に基づいて自己批判・修正を行う。

```
Constitutional AIのプロセス:
1. モデルが回答を生成
2. 「この回答は有害ではないか？」と自己批判（Critique）
3. 原則に基づいて回答を修正（Revision）
4. 修正後の回答でRLHF
```

**主な特徴**:
- 長文脈（200Kトークン）での一貫性
- Extended Thinking（思考過程の明示化）
- 「わからない」と言う傾向（過信を避ける）
- プロンプトエンジニアリングへの応答性が高い

- 📘 [Anthropic Constitutional AI](https://www.anthropic.com/research/constitutional-ai-harmlessness-from-ai-feedback)
- 📘 [Anthropic Core Views on AI Safety](https://www.anthropic.com/core-views-on-ai-safety)
- 📘 [Claude Model Card](https://docs.anthropic.com/en/docs/resources/model-card)

### 4-2. Google：マルチモーダル重視

**チューニング哲学**: テキスト・画像・音声・動画を統合的に扱うネイティブマルチモーダルモデル。Google検索・YouTube・Google Cloudとのエコシステム統合を重視。

```
Geminiのマルチモーダル統合:
- テキスト、画像、音声、動画を同一モデルで処理
- YouTube URL直接入力による動画理解
- Google Search Groundingによるリアルタイム情報取得
- 1Mトークンの超長文脈
```

**主な特徴**:
- ネイティブマルチモーダル（後付けでない）
- 長文脈（Gemini 1.5 Pro: 1Mトークン）
- コード実行（Code Interpreter内蔵）
- Google Cloud/Workspaceとの統合

- 📘 [Gemini モデル概要](https://ai.google.dev/gemini-api/docs/models/gemini)
- 📘 [Google DeepMind Gemini Technical Report](https://storage.googleapis.com/deepmind-media/gemini/gemini_1_report.pdf)

### 4-3. OpenAI：汎用性と推論能力

**チューニング哲学**: 「汎用人工知能（AGI）への段階的アプローチ」。GPT系は汎用性、o1/o3系は推論能力に特化。Function Calling、構造化出力など開発者向け機能を先駆けて提供。

```
OpenAIの2系統:
GPT系（GPT-4o等）: 汎用・高速・マルチモーダル
o系（o1, o3等）:   推論特化・Reasoning Tokens・遅いが正確
```

**主な特徴**:
- Reasoning（o1/o3系）: 答える前に「考える」
- Function Calling / Tool Useの先駆者
- 構造化出力（Structured Outputs）のAPI保証
- Realtime API（音声会話）

- 📘 [OpenAI GPT-4 System Card](https://cdn.openai.com/papers/gpt-4-system-card.pdf)
- 📘 [OpenAI Reasoning Models](https://platform.openai.com/docs/guides/reasoning)

### 4-4. 各社の思想比較

| 観点 | Anthropic (Claude) | Google (Gemini) | OpenAI (GPT/o系) |
|--|--|--|--|
| **安全性アプローチ** | 憲法AI、原則ベース | Google AI Principles | RLHF、システムカード |
| **マルチモーダル** | Vision対応、生成は非対応 | ネイティブ統合 | 段階的追加 |
| **長文脈** | 200K | 1M | 128K |
| **推論特化** | Extended Thinking | Thinking Config | Reasoning Tokens (o系) |
| **開発者UX** | シンプルAPI | Google Cloud統合 | 機能豊富、先駆的 |
| **OSS/オープン化** | なし | Gemma（軽量版） | なし（ただしGPT-2まで公開） |

### 4-5. AGIへの各社のスタンス

**一言**: 各社とも「AGI（汎用人工知能）」を意識しているが、アプローチと時間軸の認識が異なる。

| 企業 | AGIへのスタンス | 公式発言・文書 |
|--|--|--|
| **Anthropic** | AGIのリスクを重視。安全なAGI開発が使命 | [Core Views on AI Safety](https://www.anthropic.com/core-views-on-ai-safety) |
| **OpenAI** | AGI開発を明確な目標として掲げる | [Planning for AGI and beyond](https://openai.com/blog/planning-for-agi-and-beyond) |
| **Google DeepMind** | AGI研究を継続、Geminiで統合アプローチ | [About Google DeepMind](https://deepmind.google/about/) |

---

## まとめ：LLM層を理解するためのチェックリスト

```
[ ] ベースモデルとファインチューニングの関係を理解しているか
[ ] LoRAが「差分学習」であることを説明できるか
[ ] 使いたいモデルの対応モダリティ（テキスト/画像/音声）を把握しているか
[ ] Encoder-only / Decoder-only / Encoder-Decoderの違いを説明できるか
[ ] 埋め込みモデルと生成モデルの使い分けを理解しているか
[ ] 各社のチューニング思想の違いを説明できるか
```

---

## 公式ドキュメント一覧

| カテゴリ | リソース |
|--|--|
| **Anthropic** | [docs.anthropic.com](https://docs.anthropic.com/) |
| **OpenAI** | [platform.openai.com/docs](https://platform.openai.com/docs/) |
| **Google Gemini** | [ai.google.dev/gemini-api/docs](https://ai.google.dev/gemini-api/docs) |
| **HuggingFace** | [huggingface.co/docs](https://huggingface.co/docs/) |
| **HuggingFace PEFT** | [huggingface.co/docs/peft](https://huggingface.co/docs/peft/) |
| **vLLM** | [docs.vllm.ai](https://docs.vllm.ai/) |
| **Ollama** | [ollama.com](https://ollama.com/) |
| **LiteLLM** | [docs.litellm.ai](https://docs.litellm.ai/) |

---

## 参考文献・解説ブログ

- 🔗 [LoRA: Low-Rank Adaptation of Large Language Models（原論文）](https://arxiv.org/abs/2106.09685)
- 🔗 [QLoRA: Efficient Finetuning of Quantized LLMs（原論文）](https://arxiv.org/abs/2305.14314)
- 🔗 [Anthropic Constitutional AI](https://www.anthropic.com/research/constitutional-ai-harmlessness-from-ai-feedback)
- 🔗 [Chip Huyen "Building LLM applications for production"](https://huyenchip.com/2023/04/11/llm-engineering.html)
- 🔗 [MTEB Leaderboard（埋め込みモデルベンチマーク）](https://huggingface.co/spaces/mteb/leaderboard)
- 📗 Chip Huyen "AI Engineering: Building Applications with Foundation Models" (O'Reilly 2025)

---

*作成日: 2026-02-22*
*対象: AIエージェント概念マップ 第1層「LLM層」の詳細解説*
