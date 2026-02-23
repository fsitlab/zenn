# 新規性分析レポート

## 調査日: 2026-02-22

---

## 類似記事・書籍

### 層モデル・アーキテクチャを扱う記事

| # | タイトル | URL/出版情報 | 概要 | 本記事との差異 |
|---|----------|--------------|------|----------------|
| 1 | Boomi: Breaking Down the Agentic Layers of AI | https://boomi.com/blog/agentic-layers-of-ai-integration/ | 6層モデル（Application/Orchestration/Agent/Context/Data/Model）でAIエージェントを整理 | **Data層とContext層を分離**している点が異なる。本記事は「通信層」でLLMとのやり取りに焦点を当てており、データ管理は外部ツール層に統合 |
| 2 | Pylar: 5 Layers of Safely Connecting AI Agents to Your Data Stack | https://www.pylar.ai/blog/5-layer-architecture-connecting-agents-databases | データアクセスに特化した5層モデル（Raw Data/SQL Views/MCP Tools/Application/End User） | **データガバナンス特化**。本記事は「LLM操作全般」を対象とし、より包括的 |
| 3 | Deloitte: Agent Layer Architecture | https://www.deloitte.com/us/en/insights/industry/technology/technology-media-and-telecom-predictions/2026/ai-agent-orchestration.html | Agent Layer/Experience Layerの2層構造 | 運用・ビジネス視点。本記事は**技術実装視点**でより詳細 |
| 4 | AI Shift: AIエージェント設計の勘所 | https://www.ai-shift.co.jp/techblog/5252 | Orchestrator-Workers等の設計パターン解説 | パターン解説が主。本記事は**全体の構造化**を提供 |
| 5 | Qiita: AIエージェント入門 - 6つの設計アーキテクチャ | https://qiita.com/syukan3/items/153409d7f387ea8c3065 | ReAct, Plan-and-Execute等の6パターン解説 | **アーキテクチャパターン**の解説。本記事は層モデルによる**位置づけ**を提供 |
| 6 | Zenn: AI Agent認可の3層アーキテクチャ入門 | https://zenn.dev/exwzd/articles/20251224_aiagent_authz_architecture | 認可に特化した3層モデル | **セキュリティ特化**。本記事は技術全般をカバー |

### プロトコル比較を扱う記事

| # | タイトル | URL/出版情報 | 概要 | 本記事との差異 |
|---|----------|--------------|------|----------------|
| 7 | OneReach.ai: MCP vs A2A: Protocols for Multi-Agent Collaboration | https://onereach.ai/blog/guide-choosing-mcp-vs-a2a-protocols/ | MCP/A2Aの機能比較・選択ガイド | 比較が主。本記事は**層モデル内での位置づけ**を明確化 |
| 8 | Cisco Blogs: MCP and A2A - A Network Engineer's Mental Model | https://blogs.cisco.com/ai/mcp-and-a2a-a-network-engineers-mental-model | ネットワーク技術者向けのメンタルモデル | 既存知識からの類推。本記事は**独自の層分類**を提供 |
| 9 | Auth0: MCP vs A2A - AI Agent Communication Protocols | https://auth0.com/blog/mcp-vs-a2a/ | 認証・認可観点でのプロトコル比較 | **認証特化**。本記事は全体構造を提供 |
| 10 | InfoQ: Architecting Agentic MLOps with A2A and MCP | https://www.infoq.com/articles/architecting-agentic-mlops-a2a-mcp/ | MLOpsにおけるA2A+MCP活用 | MLOps特化。本記事は**汎用の整理フレームワーク** |
| 11 | Niklas Heidloff: Comparison of Agent Protocols MCP, ACP and A2A | https://heidloff.net/article/mcp-acp-a2a-agent-protocols/ | 3プロトコルの技術比較 | 機能比較が主。本記事は**層間関係**を可視化 |

### フレームワーク比較を扱う記事

| # | タイトル | URL/出版情報 | 概要 | 本記事との差異 |
|---|----------|--------------|------|----------------|
| 12 | DataCamp: CrewAI vs LangGraph vs AutoGen | https://www.datacamp.com/tutorial/crewai-vs-langgraph-vs-autogen | 3フレームワークの機能・設計比較 | フレームワーク比較が主。本記事は**層モデルで統一的に整理** |
| 13 | AQUA: AIエージェント開発フレームワーク徹底比較 | https://www.aquallc.jp/2025/12/18/ai-agent-frameworks/ | LangChain/CrewAI/AutoGen/LlamaIndex比較 | 選定ガイド的。本記事は**概念的位置づけ**を重視 |
| 14 | GMO: フレームワーク選定ガイド | https://recruit.group.gmo/engineer/jisedai/blog/agents-sdk/ | 各SDKの特徴と選定基準 | 実践的選定ガイド。本記事は**構造理解**を重視 |

### 製品分析を扱う記事

| # | タイトル | URL/出版情報 | 概要 | 本記事との差異 |
|---|----------|--------------|------|----------------|
| 15 | Qodo: Claude Code vs Cursor Deep Comparison | https://www.qodo.ai/blog/claude-code-vs-cursor/ | 2製品の機能・ユースケース比較 | 機能比較が主。本記事は**層モデルでの位置づけ**を提供 |
| 16 | Skywork: Dify Review 2025 | https://skywork.ai/blog/dify-review-2025-workflows-agents-rag-ai-apps/ | Difyの機能レビュー | 単一製品レビュー。本記事は**複数製品を統一フレームワークで分析** |

### 書籍

| # | タイトル | URL/出版情報 | 概要 | 本記事との差異 |
|---|----------|--------------|------|----------------|
| 17 | AI Engineering | Chip Huyen, O'Reilly, 2025 | エージェント/RAG/評価の包括的解説 | 書籍形式で詳細。本記事は**層モデルによるクイックリファレンス** |
| 18 | AIエージェント開発／運用入門 | 御田稔他, SBクリエイティブ, 2025 | ReAct/Function Calling/フレームワーク解説 | 開発入門書。本記事は**俯瞰的な知識マップ** |
| 19 | AIエージェント革命 | シグマクシス, 日経BP, 2025 | ビジネス×技術の体系的解説 | ビジネス視点込み。本記事は**技術特化の層モデル** |
| 20 | AIエージェント白書2025 | CMCリサーチ, 2025 | エージェントタイプ分類・市場動向 | 調査レポート形式。本記事は**実装者向け整理** |

### 公式ドキュメント・ガイド

| # | タイトル | URL/出版情報 | 概要 | 本記事との差異 |
|---|----------|--------------|------|----------------|
| 21 | Anthropic: Building Effective Agents | https://www.anthropic.com/research/building-effective-agents | エージェント設計の公式ベストプラクティス | パターン解説が主。本記事は**層構造での整理**を提供 |
| 22 | Microsoft: AI Agent Orchestration Patterns | https://learn.microsoft.com/en-us/azure/architecture/ai-ml/guide/ai-agent-design-patterns | Azure向けオーケストレーションパターン | Azure特化。本記事は**ベンダー中立**の整理 |
| 23 | IBM: What is AI Agent Orchestration? | https://www.ibm.com/think/topics/ai-agent-orchestration | オーケストレーション概念の解説 | 概念説明が主。本記事は**実装との対応**を明確化 |

---

## 本記事の新規性

### 独自のアプローチ

1. **5層モデルによる体系的整理**
   - 既存の記事は3層、4層、6層など様々だが、本記事は「LLM層/通信層/LLMオーケストレーション層/外部ツール層/UI・運用層」の5層で一貫して整理
   - 特に「通信層」を2層A（クライアント起点）と2層B（LLM起点）に分けている点は、他に類を見ない詳細な分類

2. **層の判定フローの提供**
   - 新技術が登場したときに「どの層に属するか」を判定するためのフローチャートを提供
   - 他の記事は分類を示すのみで、判定ロジックを提供していない

3. **「定義」と「影響」の区別**
   - 技術が「定義される層」と「影響を及ぼす層」を明確に区別
   - 例: MCPは第3層↔第4層間の定義だが、第2層A/Bにも影響
   - 他の記事はこの区別を行っていない

4. **具体製品の層分解**
   - Claude Code、Cursor、Dify等を5層モデルで分解
   - 製品比較記事は多いが、統一的なフレームワークで分解した記事は少ない

5. **日本語での包括的な知識マップ**
   - MCP、A2A、Agent Teams等の最新技術を日本語で体系的に整理
   - 英語記事は多いが、日本語での包括的整理は希少

### 類似記事にない観点

1. **OSI参照モデル的な発想の応用**
   - ネットワークのOSI参照モデルのように、AIエージェント技術を層で整理する発想
   - 検索した限り、AIエージェントにOSI参照モデル的な層モデルを明示的に適用した日本語記事は存在しない

2. **MCP/A2Aの層間での役割の明確化**
   - 他の記事は「MCPとA2Aの違い」を説明するが、本記事は「どの層間で機能するか」を明確化
   - MCP: 第3層（MCPクライアント）↔ 第4層（MCPサーバー）
   - A2A: 第4層（サブエージェント間）

3. **従来RAGとツール型RAGの層的区別**
   - 従来RAG: 第2層A（プロンプトへの注入）
   - ツール型RAG: 第4層（MCPサーバー等）
   - この区別を層モデルで明示した記事は見当たらない

4. **Hooks/Skillsの位置づけ**
   - Claude CodeのHooks（第3層）とSkills（第2層A）の違いを層モデルで説明
   - 製品ドキュメント以外でこの区別を体系的に説明した記事は少ない

5. **相性問題の層間分析**
   - 「どの層間で問題が発生するか」という観点での分析
   - トラブルシューティングの際に層を意識することの重要性を示す
   - 他の記事にはない実践的視点

### 潜在的な弱点・改善点

1. **実装例・コードが少ない**
   - 概念整理が主で、具体的なコード例は少なめ
   - 「AIエージェント開発／運用入門」等の書籍のほうが実装詳細は充実

2. **評価・ベンチマークの観点が弱い**
   - Chip Huyen "AI Engineering"のような評価手法の詳細は含まれていない

3. **ビジネス・ROI観点が薄い**
   - 「AIエージェント革命」のようなビジネス視点は含まれていない

---

## 結論

本記事シリーズ「AIエージェント知識マップ」の最大の新規性は、**5層モデルによるAIエージェント技術の体系的整理**である。

既存の記事・書籍は以下のいずれかに該当することが多い：
- 特定領域（プロトコル、フレームワーク、製品）に特化
- パターン・ベストプラクティスの列挙
- ビジネス観点との融合

これに対し、本記事は：
- **ベンダー中立**の層モデルを提供
- **判定フロー**により新技術の位置づけが可能
- **定義と影響の区別**により複雑な関係性を整理
- **日本語**で包括的に整理

という独自の価値を提供している。特に「通信層の2層A/B分割」と「層の判定フロー」は、他の記事には見られない本記事独自の貢献である。

---

## 参考文献リスト（09_付録.mdへの追加候補）

### 層モデル・アーキテクチャ

| リソース | URL | 内容 |
|----------|-----|------|
| Boomi: Breaking Down the Agentic Layers of AI | https://boomi.com/blog/agentic-layers-of-ai-integration/ | 6層モデルによるAIエージェント整理 |
| Pylar: 5 Layers Architecture | https://www.pylar.ai/blog/5-layer-architecture-connecting-agents-databases | データアクセス5層モデル |
| Deloitte: AI Agent Orchestration | https://www.deloitte.com/us/en/insights/industry/technology/technology-media-and-telecom-predictions/2026/ai-agent-orchestration.html | エンタープライズ向けオーケストレーション |
| Microsoft: AI Agent Orchestration Patterns | https://learn.microsoft.com/en-us/azure/architecture/ai-ml/guide/ai-agent-design-patterns | Azure向けパターン集 |
| IBM: AI Agent Orchestration | https://www.ibm.com/think/topics/ai-agent-orchestration | オーケストレーション概念解説 |

### プロトコル比較

| リソース | URL | 内容 |
|----------|-----|------|
| OneReach.ai: MCP vs A2A | https://onereach.ai/blog/guide-choosing-mcp-vs-a2a-protocols/ | プロトコル選択ガイド |
| Cisco: MCP and A2A Mental Model | https://blogs.cisco.com/ai/mcp-and-a2a-a-network-engineers-mental-model | ネットワーク観点でのメンタルモデル |
| Auth0: MCP vs A2A | https://auth0.com/blog/mcp-vs-a2a/ | 認証観点でのプロトコル比較 |
| InfoQ: Agentic MLOps with A2A and MCP | https://www.infoq.com/articles/architecting-agentic-mlops-a2a-mcp/ | MLOpsでのプロトコル活用 |
| Niklas Heidloff: MCP, ACP, A2A Comparison | https://heidloff.net/article/mcp-acp-a2a-agent-protocols/ | 3プロトコル技術比較 |
| A2A Protocol: A2A and MCP | https://a2a-protocol.org/latest/topics/a2a-and-mcp/ | 公式のA2A/MCP関係説明 |

### フレームワーク比較

| リソース | URL | 内容 |
|----------|-----|------|
| DataCamp: CrewAI vs LangGraph vs AutoGen | https://www.datacamp.com/tutorial/crewai-vs-langgraph-vs-autogen | 主要フレームワーク比較 |
| Turing: Top AI Agent Frameworks 2026 | https://www.turing.com/resources/ai-agent-frameworks | 6フレームワーク詳細比較 |
| Langflow: Guide to AI Agent Frameworks 2025 | https://www.langflow.org/blog/the-complete-guide-to-choosing-an-ai-agent-framework-in-2025 | 選定ガイド |

### 書籍（追加）

| リソース | 出版情報 | 内容 |
|----------|----------|------|
| AIエージェント開発／運用入門 | 御田稔他, SBクリエイティブ, 2025 | ReAct/Function Calling/フレームワーク解説 |
| AIエージェント革命 | シグマクシス, 日経BP, 2025 | ビジネス×技術の体系的解説 |
| AIエージェント白書2025 | CMCリサーチ, 2025 | エージェントタイプ分類・市場動向 |
| やさしく学ぶLLMエージェント | 井上顧基他, オーム社, 2025 | 基礎からマルチエージェント構築まで |

---

*分析日: 2026-02-22*
*分析者: Claude Opus 4.5*
