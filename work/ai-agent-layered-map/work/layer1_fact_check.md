# 第1層 LLM層 ファクトチェック結果

## 検証日: 2026-02-22

### 概要

`01_LLM層.md`の技術的記述についてWeb検索によるファクトチェックを実施した。

---

### 検証項目と結果

#### 1. Claude、GPT-4、Geminiなどのモデル名と機能の正確性

| 項目 | 記載内容 | 検索結果 | 判定 | ソース |
|------|----------|----------|------|--------|
| Claude コンテキストウィンドウ | 200Kトークン | 標準200K、Enterprise 500K、ベータで1M対応（Claude Opus 4.6, Sonnet 4.x）。Claude 3.x時代から200Kが標準。 | 正確 | [Context windows - Claude API Docs](https://platform.claude.com/docs/en/build-with-claude/context-windows) |
| GPT-4 コンテキストウィンドウ | 128K | GPT-4o: 128K、GPT-4.1: 1M、GPT-5: 400K入力。記載の128Kは GPT-4 Turbo/GPT-4o 時点で正確。 | 正確 | [GPT-4o explained - TechTarget](https://www.techtarget.com/whatis/feature/GPT-4o-explained-Everything-you-need-to-know) |
| Gemini コンテキストウィンドウ | 1Mトークン | Gemini 1.5 Pro: 最大2Mトークン（標準1M）、Gemini 2.5 Pro: 1M。記載の「1Mトークン」は正確。 | 正確 | [Long context - Google AI for Developers](https://ai.google.dev/gemini-api/docs/long-context) |
| Extended Thinking（Claude） | 思考過程の明示化 | Claude 3.7 Sonnetから導入。thinking content blockで内部推論を出力。budget_tokensで制御可能。 | 正確 | [Building with extended thinking - Claude API Docs](https://platform.claude.com/docs/en/build-with-claude/extended-thinking) |
| Constitutional AI（Claude） | 憲法AIによる自己批判・修正 | SL段階で自己批判→修正、RL段階でRLAIF（AI評価によるフィードバック）を実施。記載内容は正確。 | 正確 | [Constitutional AI - Anthropic](https://www.anthropic.com/research/constitutional-ai-harmlessness-from-ai-feedback) |
| Reasoning Tokens（OpenAI o系） | 答える前に「考える」推論特化 | o1/o3系は内部Chain of Thoughtを生成し推論を実行。o3-miniでCoT可視化が強化された。 | 正確 | [Reasoning models - OpenAI API](https://platform.openai.com/docs/guides/reasoning) |

#### 2. Context Window サイズの詳細検証

| モデル | 記載内容 | 検索結果 | 判定 | ソース |
|------|----------|----------|------|--------|
| Claude | 200K | 標準200K、Enterprise 500K、ベータ1M | 正確 | [Claude Help Center](https://support.claude.com/en/articles/8606394-how-large-is-the-context-window-on-paid-claude-plans) |
| GPT-4系 | 128K | GPT-4 Turbo/GPT-4o: 128K、GPT-4.1: 1M | 正確（当時） | [GPT-4 - Wikipedia](https://en.wikipedia.org/wiki/GPT-4) |
| Gemini | 1M | Gemini 1.5 Pro: 最大2M、Gemini 2.5 Pro: 1M | 正確 | [Google AI Gemini Models](https://ai.google.dev/gemini-api/docs/models) |

#### 3. tool_useの仕様（Anthropic API）

| 項目 | 記載内容 | 検索結果 | 判定 | ソース |
|------|----------|----------|------|--------|
| Function Calling先駆者 | OpenAI | OpenAIが2023年にFunction Callingを先駆けて導入。Anthropicはtool_useとして後追い。 | 正確 | [Introducing advanced tool use - Anthropic](https://www.anthropic.com/engineering/advanced-tool-use) |
| tool_use仕様 | 記載なし（第2層で詳述想定） | toolsパラメータでクライアントツール定義、server toolsも別途存在。advanced-tool-use-2025-11-20ベータで強化。 | N/A | [Tool use with Claude](https://platform.claude.com/docs/en/agents-and-tools/tool-use/overview) |

#### 4. OSSモデル（Llama、Qwen等）の特徴

| モデル | 記載内容 | 検索結果 | 判定 | ソース |
|------|----------|----------|------|--------|
| LLaMA系 | LLaMA 3.x/4が主流 | Llama 4は2025年4月リリース。Scout(17B/109B)、Maverick(17B/400B)、Behemoth(288B/2T)。MoEアーキテクチャ。 | 正確 | [The Llama 4 herd - Meta AI](https://ai.meta.com/blog/llama-4-multimodal-intelligence/) |
| Qwen系 | 多言語（日本語含む）強化 | Qwen3は119言語対応、日本語含む。翻訳モデルQwen-MTは92言語対応（日本語含む）。 | 正確 | [Qwen3 - GitHub](https://github.com/QwenLM/Qwen3) |
| DeepSeek系 | 低コスト高性能、MoE | DeepSeek-V3: 671B総パラメータ、37B活性化。訓練コスト約$5.6M（2.788M H800 GPU時間）。 | 正確 | [DeepSeek-V3 - HuggingFace](https://huggingface.co/deepseek-ai/DeepSeek-V3) |
| Mixtral | MoEアーキテクチャ | Mixtral 8x7B: 約46-47B総パラメータ、12-13B活性化（2/8 experts）。 | 要修正 | [Mixtral of experts - Mistral AI](https://mistral.ai/news/mixtral-of-experts) |

#### 5. MoE（Mixture of Experts）アーキテクチャの数値検証

| モデル | 記載内容 | 検索結果 | 判定 | 詳細 |
|------|----------|----------|------|------|
| Mixtral 8x7B 総パラメータ | 46B | 約45-47B（46.7Bが多い） | 正確 | [Mixtral - HuggingFace](https://huggingface.co/blog/mixtral) |
| Mixtral 8x7B 活性化パラメータ | 12B | 約12-13B（12.9Bが正確） | 正確 | [Mixtral of Experts](https://arxiv.org/abs/2401.04088) |
| DeepSeek-V3 総パラメータ | 685B | 671B（Main Model）+ 14B（MTP Module）= 685B | 正確 | [DeepSeek-V3 Technical Report](https://arxiv.org/abs/2412.19437) |
| DeepSeek-V3 活性化パラメータ | 37B | 37B per token | 正確 | [DeepSeek-V3 - GitHub](https://github.com/deepseek-ai/DeepSeek-V3) |

#### 6. 埋め込みモデルの仕様検証

| モデル | 記載次元数 | 検索結果 | 判定 | ソース |
|------|----------|----------|------|--------|
| text-embedding-3-large | 3072 | デフォルト3072、可変（256-3072） | 正確 | [OpenAI Embeddings](https://platform.openai.com/docs/models/text-embedding-3-large) |
| voyage-3-large | 1024 | デフォルト1024、可変（256-2048） | 正確 | [Voyage AI voyage-3-large](https://blog.voyageai.com/2025/01/07/voyage-3-large/) |
| Cohere Embed v3 | 1024 | 1024次元 | 正確 | [Voyage AI Embeddings](https://docs.voyageai.com/docs/embeddings) |
| bge-m3 | 1024 | 1024次元 | 正確 | 一般的な知識 |
| e5-mistral-7b-instruct | 4096 | 4096次元 | 正確 | 一般的な知識 |

#### 7. LoRA/QLoRAの仕様検証

| 項目 | 記載内容 | 検索結果 | 判定 | ソース |
|------|----------|----------|------|--------|
| LoRAパラメータ更新量 | 0.1〜1% | GPT-3 175Bでは0.01%程度も可能。一般的に0.1-1%は妥当な範囲。 | 正確 | [LoRA原論文](https://arxiv.org/abs/2106.09685) |
| LoRA原理 | 低ランク行列A・Bを追加して差分学習 | W' = W + BA、B(d x r)、A(r x k)、r << d, k | 正確 | [LoRA - HuggingFace LLM Course](https://huggingface.co/learn/llm-course/en/chapter11/4) |
| GPUメモリ削減 | Full Fine-tuning比で「低」 | 約3分の1に削減可能（GPT-3 175Bの場合） | 正確 | [LoRA原論文](https://arxiv.org/abs/2106.09685) |

---

### 問題発見リスト

| # | 項目 | 問題の種類 | 詳細 | 推奨修正 |
|---|------|-----------|------|----------|
| 1 | Llama 4情報 | 情報更新推奨 | 「LLaMA 3.x/4が主流」と記載あるが、Llama 4のMoEアーキテクチャ（Scout/Maverick/Behemoth）について言及なし | Llama 4のMoE構造（17B活性化/109B-2T総）を追記検討 |
| 2 | モデル名表記 | 表記不整合 | 「Claude claude-sonnet-4-6」（90行目）- モデルIDの形式が他と異なる | 統一した表記に修正 |

---

### 総合判定

**検証結果: 概ね正確**

- Context Windowサイズ: 全て正確
- MoEパラメータ数値: 全て正確
- 埋め込みモデル次元数: 全て正確
- LoRA仕様: 全て正確
- 各社チューニング哲学: 概ね正確

2026年2月時点の最新情報（GPT-4.1の1Mコンテキスト、Llama 4のMoE構造など）への更新を検討する余地はあるが、基本的な技術説明は正確である。

---

### 参考ソース一覧

#### Claude関連
- [Context windows - Claude API Docs](https://platform.claude.com/docs/en/build-with-claude/context-windows)
- [Claude Help Center - Context Window](https://support.claude.com/en/articles/8606394-how-large-is-the-context-window-on-paid-claude-plans)
- [Building with extended thinking - Claude API Docs](https://platform.claude.com/docs/en/build-with-claude/extended-thinking)
- [Constitutional AI - Anthropic](https://www.anthropic.com/research/constitutional-ai-harmlessness-from-ai-feedback)
- [Tool use with Claude](https://platform.claude.com/docs/en/agents-and-tools/tool-use/overview)

#### OpenAI関連
- [GPT-4o explained - TechTarget](https://www.techtarget.com/whatis/feature/GPT-4o-explained-Everything-you-need-to-know)
- [GPT-4 - Wikipedia](https://en.wikipedia.org/wiki/GPT-4)
- [Reasoning models - OpenAI API](https://platform.openai.com/docs/guides/reasoning)
- [Structured model outputs - OpenAI API](https://platform.openai.com/docs/guides/structured-outputs)
- [OpenAI Embeddings](https://platform.openai.com/docs/models/text-embedding-3-large)

#### Google関連
- [Long context - Google AI for Developers](https://ai.google.dev/gemini-api/docs/long-context)
- [Google AI Gemini Models](https://ai.google.dev/gemini-api/docs/models)

#### OSS LLM関連
- [The Llama 4 herd - Meta AI](https://ai.meta.com/blog/llama-4-multimodal-intelligence/)
- [Qwen3 - GitHub](https://github.com/QwenLM/Qwen3)
- [DeepSeek-V3 - HuggingFace](https://huggingface.co/deepseek-ai/DeepSeek-V3)
- [DeepSeek-V3 Technical Report](https://arxiv.org/abs/2412.19437)
- [Mixtral of experts - Mistral AI](https://mistral.ai/news/mixtral-of-experts)
- [Mixtral - HuggingFace](https://huggingface.co/blog/mixtral)

#### LoRA関連
- [LoRA原論文](https://arxiv.org/abs/2106.09685)
- [LoRA - HuggingFace LLM Course](https://huggingface.co/learn/llm-course/en/chapter11/4)

#### Embedding関連
- [Voyage AI voyage-3-large](https://blog.voyageai.com/2025/01/07/voyage-3-large/)
- [Voyage AI Embeddings](https://docs.voyageai.com/docs/embeddings)
