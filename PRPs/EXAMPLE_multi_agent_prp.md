name: "Multi-Agent System: Research Agent with Email Draft Sub-Agent"
description: |

## 目的
プライマリリサーチエージェントがBrave Search APIを使用し、ツールとしてメールドラフトエージェント（Gmail API使用）を持つPydantic AIマルチエージェントシステムを構築。これは外部API統合を伴うエージェント・アズ・ツールパターンを実証します。

## 核となる原則
1. **コンテキストが王**: 必要なすべてのドキュメント、例、注意点を含める
2. **検証ループ**: AIが実行して修正できる実行可能なテスト/リントを提供
3. **情報密度**: コードベースからのキーワードとパターンを使用
4. **段階的成功**: シンプルに開始し、検証し、その後強化

---

## 目標
ユーザーがCLI経由でトピックをリサーチでき、リサーチエージェントがメールドラフトタスクをメールドラフトエージェントに委譲できる本番準備完了のマルチエージェントシステムを作成。システムは複数のLLMプロバイダーをサポートし、API認証を安全に処理する必要があります。

## 理由
- **ビジネス価値**: リサーチとメールドラフトワークフローを自動化
- **統合**: 高度なPydantic AIマルチエージェントパターンを実証
- **解決する問題**: リサーチベースのメールコミュニケーションの手動作業を削減

## 何を
以下のCLIベースアプリケーション：
- ユーザーがリサーチクエリを入力
- リサーチエージェントがBrave APIを使用して検索
- リサーチエージェントがGmailドラフトを作成するためにメールドラフトエージェントを呼び出し可能
- 結果がリアルタイムでユーザーにストリーミング

### 成功基準
- [ ] リサーチエージェントがBrave API経由で正常に検索
- [ ] メールエージェントが適切な認証でGmailドラフトを作成
- [ ] リサーチエージェントがツールとしてメールエージェントを呼び出し可能
- [ ] CLIがツール可視性を伴うストリーミングレスポンスを提供
- [ ] すべてのテストが合格し、コードが品質基準を満たす

## 必要なすべてのコンテキスト

### ドキュメントと参照
```yaml
# 必ず読む - これらをコンテキストウィンドウに含める
- url: https://ai.pydantic.dev/agents/
  why: コアエージェント作成パターン
  
- url: https://ai.pydantic.dev/multi-agent-applications/
  why: マルチエージェントシステムパターン、特にエージェント・アズ・ツール
  
- url: https://developers.google.com/gmail/api/guides/sending
  why: Gmail API認証とドラフト作成
  
- url: https://api-dashboard.search.brave.com/app/documentation
  why: Brave Search API RESTエンドポイント
  
- file: examples/agent/agent.py
  why: エージェント作成、ツール登録、依存関係のパターン
  
- file: examples/agent/providers.py
  why: マルチプロバイダーLLM設定パターン
  
- file: examples/cli.py
  why: ストリーミングレスポンスとツール可視性を伴うCLI構造

- url: https://github.com/googleworkspace/python-samples/blob/main/gmail/snippet/send%20mail/create_draft.py
  why: 公式Gmailドラフト作成例
```

### 現在のコードベースツリー
```bash
.
├── examples/
│   ├── agent/
│   │   ├── agent.py
│   │   ├── providers.py
│   │   └── ...
│   └── cli.py
├── PRPs/
│   └── templates/
│       └── prp_base.md
├── INITIAL.md
├── CLAUDE.md
└── requirements.txt
```

### 追加されるファイルを含む希望するコードベースツリー
```bash
.
├── agents/
│   ├── __init__.py               # パッケージ初期化
│   ├── research_agent.py         # Brave Searchを持つプライマリエージェント
│   ├── email_agent.py           # Gmail機能を持つサブエージェント
│   ├── providers.py             # LLMプロバイダー設定
│   └── models.py                # データ検証用のPydanticモデル
├── tools/
│   ├── __init__.py              # パッケージ初期化
│   ├── brave_search.py          # Brave Search API統合
│   └── gmail_tool.py            # Gmail API統合
├── config/
│   ├── __init__.py              # パッケージ初期化
│   └── settings.py              # 環境と設定管理
├── tests/
│   ├── __init__.py              # パッケージ初期化
│   ├── test_research_agent.py   # リサーチエージェントテスト
│   ├── test_email_agent.py      # メールエージェントテスト
│   ├── test_brave_search.py     # Brave検索ツールテスト
│   ├── test_gmail_tool.py       # Gmailツールテスト
│   └── test_cli.py              # CLIテスト
├── cli.py                       # CLIインターフェース
├── .env.example                 # 環境変数テンプレート
├── requirements.txt             # 更新された依存関係
├── README.md                    # 包括的なドキュメント
└── credentials/.gitkeep         # Gmail認証情報用ディレクトリ
```

### 既知の落とし穴とライブラリの癖
```python
# 重要: Pydantic AIは全体で非同期を必要とする - 非同期コンテキストで同期関数を使用しない
# 重要: Gmail APIは初回実行時にOAuth2フローを必要とする - credentials.jsonが必要
# 重要: Brave APIにはレート制限がある - 無料枠で月2000リクエスト
# 重要: エージェント・アズ・ツールパターンはトークン追跡のためにctx.usageを渡す必要がある
# 重要: Gmailドラフトは適切なMIMEフォーマットでbase64エンコーディングが必要
# 重要: よりクリーンなコードのために常に絶対インポートを使用
# 重要: 機密認証情報は.envに保存し、コミットしない
```

## 実装青写真

### データモデルと構造

```python
# models.py - コアデータ構造
from pydantic import BaseModel, Field
from typing import List, Optional
from datetime import datetime

class ResearchQuery(BaseModel):
    query: str = Field(..., description="調査するリサーチトピック")
    max_results: int = Field(10, ge=1, le=50)
    include_summary: bool = Field(True)

class BraveSearchResult(BaseModel):
    title: str
    url: str
    description: str
    score: float = Field(0.0, ge=0.0, le=1.0)

class EmailDraft(BaseModel):
    to: List[str] = Field(..., min_items=1)
    subject: str = Field(..., min_length=1)
    body: str = Field(..., min_length=1)
    cc: Optional[List[str]] = None
    bcc: Optional[List[str]] = None

class ResearchEmailRequest(BaseModel):
    research_query: str
    email_context: str = Field(..., description="メール生成のためのコンテキスト")
    recipient_email: str
```

### 完了すべきタスクのリスト

```yaml
タスク1: 設定と環境のセットアップ
CREATE config/settings.py:
  - パターン: 例がos.getenvを使用するようにpydantic-settingsを使用
  - デフォルトで環境変数を読み込み
  - 必要なAPIキーの存在を検証

CREATE .env.example:
  - 説明付きのすべての必要な環境変数を含める
  - examples/README.mdのパターンに従う

タスク2: Brave Searchツールの実装
CREATE tools/brave_search.py:
  - パターン: examples/agent/tools.pyのような非同期関数
  - httpxを使用したシンプルなRESTクライアント（既にrequirementsに含まれている）
  - レート制限とエラーを適切に処理
  - 構造化されたBraveSearchResultモデルを返す

タスク3: Gmailツールの実装
CREATE tools/gmail_tool.py:
  - パターン: GmailクイックスタートからOAuth2フローに従う
  - credentials/ディレクトリにtoken.jsonを保存
  - 適切なMIMEエンコーディングでドラフトを作成
  - 認証更新を自動的に処理

タスク4: メールドラフトエージェントの作成
CREATE agents/email_agent.py:
  - パターン: examples/agent/agent.py構造に従う
  - deps_typeパターンでAgentを使用
  - gmail_toolを@agent.toolとして登録
  - EmailDraftモデルを返す

タスク5: リサーチエージェントの作成
CREATE agents/research_agent.py:
  - パターン: Pydantic AIドキュメントからのマルチエージェントパターン
  - brave_searchをツールとして登録
  - email_agent.run()をツールとして登録
  - 依存性注入のためにRunContextを使用

タスク6: CLIインターフェースの実装
CREATE cli.py:
  - パターン: examples/cli.pyストリーミングパターンに従う
  - ツール可視性を伴うカラーコード出力
  - asyncio.run()で非同期を適切に処理
  - 会話コンテキストのためのセッション管理

タスク7: 包括的なテストの追加
CREATE tests/:
  - パターン: 例のテスト構造を反映
  - 外部API呼び出しをモック
  - ハッピーパス、エッジケース、エラーをテスト
  - 80%以上のカバレッジを確保

タスク8: ドキュメントの作成
CREATE README.md:
  - パターン: examples/README.md構造に従う
  - セットアップ、インストール、使用方法を含める
  - APIキー設定手順
  - アーキテクチャ図
```

### タスクごとの疑似コード

```python
# タスク2: Brave Searchツール
async def search_brave(query: str, api_key: str, count: int = 10) -> List[BraveSearchResult]:
    # パターン: 例がaiohttpを使用するようにhttpxを使用
    async with httpx.AsyncClient() as client:
        headers = {"X-Subscription-Token": api_key}
        params = {"q": query, "count": count}
        
        # 落とし穴: Brave APIはAPIキーが無効な場合401を返す
        response = await client.get(
            "https://api.search.brave.com/res/v1/web/search",
            headers=headers,
            params=params,
            timeout=30.0  # 重要: ハングを避けるためにタイムアウトを設定
        )
        
        # パターン: 構造化されたエラーハンドリング
        if response.status_code != 200:
            raise BraveAPIError(f"API returned {response.status_code}")
        
        # Pydanticで解析と検証
        data = response.json()
        return [BraveSearchResult(**result) for result in data.get("web", {}).get("results", [])]

# タスク5: メールエージェントをツールとして持つリサーチエージェント
@research_agent.tool
async def create_email_draft(
    ctx: RunContext[AgentDependencies],
    recipient: str,
    subject: str,
    context: str
) -> str:
    """リサーチコンテキストに基づいてメールドラフトを作成。"""
    # 重要: トークン追跡のためにusageを渡す
    result = await email_agent.run(
        f"Create an email to {recipient} about: {context}",
        deps=EmailAgentDeps(subject=subject),
        usage=ctx.usage  # マルチエージェントドキュメントからのパターン
    )
    
    return f"Draft created with ID: {result.data}"
```

### 統合ポイント
```yaml
環境:
  - add to: .env
  - vars: |
      # LLM設定
      LLM_PROVIDER=openai
      LLM_API_KEY=sk-...
      LLM_MODEL=gpt-4
      
      # Brave Search
      BRAVE_API_KEY=BSA...
      
      # Gmail（credentials.jsonへのパス）
      GMAIL_CREDENTIALS_PATH=./credentials/credentials.json
      
設定:
  - Gmail OAuth: 初回実行時にブラウザが開いて認証
  - トークン保存: ./credentials/token.json（自動作成）
  
依存関係:
  - requirements.txtを以下で更新:
    - google-api-python-client
    - google-auth-httplib2
    - google-auth-oauthlib
```

## 検証ループ

### レベル1: 構文とスタイル
```bash
# これらを最初に実行 - 続行前にエラーを修正
ruff check . --fix              # スタイル問題を自動修正
mypy .                          # 型チェック

# 期待: エラーなし。エラーがある場合、読み取って修正。
```

### レベル2: ユニットテスト
```python
# test_research_agent.py
async def test_research_with_brave():
    """リサーチエージェントが正しく検索することをテスト"""
    agent = create_research_agent()
    result = await agent.run("AI safety research")
    assert result.data
    assert len(result.data) > 0

async def test_research_creates_email():
    """リサーチエージェントがメールエージェントを呼び出せることをテスト"""
    agent = create_research_agent()
    result = await agent.run(
        "Research AI safety and draft email to john@example.com"
    )
    assert "draft_id" in result.data

# test_email_agent.py  
def test_gmail_authentication(monkeypatch):
    """Gmail OAuthフロー処理をテスト"""
    monkeypatch.setenv("GMAIL_CREDENTIALS_PATH", "test_creds.json")
    tool = GmailTool()
    assert tool.service is not None

async def test_create_draft():
    """適切なエンコーディングでドラフト作成をテスト"""
    agent = create_email_agent()
    result = await agent.run(
        "Create email to test@example.com about AI research"
    )
    assert result.data.get("draft_id")
```

```bash
# 合格するまでテストを反復的に実行:
pytest tests/ -v --cov=agents --cov=tools --cov-report=term-missing

# 失敗する場合: 特定のテストをデバッグし、コードを修正し、再実行
```

### レベル3: 統合テスト
```bash
# CLI対話をテスト
python cli.py

# 期待される対話:
# You: Research latest AI safety developments
# 🤖 Assistant: [リサーチ結果をストリーミング]
# 🛠 Tools Used:
#   1. brave_search (query='AI safety developments', limit=10)
#
# You: Create an email draft about this to john@example.com  
# 🤖 Assistant: [ドラフトを作成]
# 🛠 Tools Used:
#   1. create_email_draft (recipient='john@example.com', ...)

# 作成されたドラフトについてGmailドラフトフォルダを確認
```

## 最終検証チェックリスト
- [ ] すべてのテストが合格: `pytest tests/ -v`
- [ ] リンティングエラーなし: `ruff check .`
- [ ] 型エラーなし: `mypy .`
- [ ] Gmail OAuthフローが動作（ブラウザが開き、トークンが保存される）
- [ ] Brave Searchが結果を返す
- [ ] リサーチエージェントがメールエージェントを正常に呼び出す
- [ ] CLIがツール可視性を伴うレスポンスをストリーミング
- [ ] エラーケースが適切に処理される
- [ ] READMEに明確なセットアップ手順が含まれる
- [ ] .env.exampleにすべての必要な変数がある

---

## 避けるべきアンチパターン
- ❌ APIキーをハードコードしない - 環境変数を使用
- ❌ 非同期エージェントコンテキストで同期関数を使用しない
- ❌ GmailのOAuthフローセットアップをスキップしない
- ❌ APIのレート制限を無視しない
- ❌ マルチエージェント呼び出しでctx.usageを渡すことを忘れない
- ❌ credentials.jsonやtoken.jsonファイルをコミットしない

## 信頼度スコア: 9/10

高い信頼度の理由：
- コードベースからの明確な例に従える
- 外部APIが十分にドキュメント化されている
- マルチエージェントシステムの確立されたパターン
- 包括的な検証ゲート

Gmail OAuth初回セットアップUXについては軽微な不確実性があるが、ドキュメントが明確なガイダンスを提供している。