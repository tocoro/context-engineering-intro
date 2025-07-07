name: "Base PRP Template v2 - Context-Rich with Validation Loops"
description: |

## 目的
AIエージェントが十分なコンテキストと自己検証機能で機能を実装し、反復的な改善を通じて動作するコードを達成するために最適化されたテンプレート。

## 核となる原則
1. **コンテキストが王**: 必要なすべてのドキュメント、例、注意点を含める
2. **検証ループ**: AIが実行して修正できる実行可能なテスト/リントを提供
3. **情報密度**: コードベースからのキーワードとパターンを使用
4. **段階的成功**: シンプルに開始し、検証し、その後強化
5. **グローバルルール**: CLAUDE.mdのすべてのルールに従うことを必ず確認

---

## 目標
[構築する必要があるもの - 最終状態と希望について具体的に]

## 理由
- [ビジネス価値とユーザーへの影響]
- [既存機能との統合]
- [これが解決する問題と対象者]

## 何を
[ユーザーが見える動作と技術要件]

### 成功基準
- [ ] [具体的で測定可能な結果]

## 必要なすべてのコンテキスト

### ドキュメントと参照（機能を実装するために必要なすべてのコンテキストをリストアップ）
```yaml
# 必ず読む - これらをコンテキストウィンドウに含める
- url: [公式APIドキュメントURL]
  why: [必要な特定のセクション/メソッド]
  
- file: [path/to/example.py]
  why: [従うべきパターン、避けるべき落とし穴]
  
- doc: [ライブラリドキュメントURL] 
  section: [一般的な落とし穴に関する特定のセクション]
  critical: [一般的なエラーを防ぐ重要な洞察]

- docfile: [PRPs/ai_docs/file.md]
  why: [ユーザーがプロジェクトに貼り付けたドキュメント]

```

### 現在のコードベースツリー（プロジェクトのルートで`tree`を実行）してコードベースの概要を取得
```bash

```

### 追加されるファイルとファイルの責任を含む希望するコードベースツリー
```bash

```

### コードベースとライブラリの癖の既知の落とし穴
```python
# 重要: [ライブラリ名]は[特定のセットアップ]を必要とする
# 例: FastAPIはエンドポイントに非同期関数を必要とする
# 例: このORMは1000レコードを超えるバッチ挿入をサポートしない
# 例: 我々はpydantic v2を使用し、  
```

## 実装青写真

### データモデルと構造

コアデータモデルを作成し、型安全性と一貫性を確保します。
```python
例: 
 - ormモデル
 - pydanticモデル
 - pydanticスキーマ
 - pydanticバリデーター

```

### PRPを満たすために完了すべきタスクのリスト（完了すべき順序で）

```yaml
タスク1:
MODIFY src/existing_module.py:
  - FIND pattern: "class OldImplementation"
  - INJECT after line containing "def __init__"
  - PRESERVE existing method signatures

CREATE src/new_feature.py:
  - MIRROR pattern from: src/similar_feature.py
  - MODIFY class name and core logic
  - KEEP error handling pattern identical

...(...)

タスクN:
...

```


### 各タスクに必要に応じて追加される疑似コード
```python

# タスク1
# 重要な詳細を含む疑似コード、完全なコードは書かない
async def new_feature(param: str) -> Result:
    # パターン: 常に最初に入力を検証（src/validators.pyを参照）
    validated = validate_input(param)  # ValidationErrorを発生させる
    
    # 落とし穴: このライブラリは接続プールを必要とする
    async with get_connection() as conn:  # src/db/pool.pyを参照
        # パターン: 既存のリトライデコレーターを使用
        @retry(attempts=3, backoff=exponential)
        async def _inner():
            # 重要: APIは10 req/secを超えると429を返す
            await rate_limiter.acquire()
            return await external_api.call(validated)
        
        result = await _inner()
    
    # パターン: 標準化されたレスポンス形式
    return format_response(result)  # src/utils/responses.pyを参照
```

### 統合ポイント
```yaml
データベース:
  - migration: "usersテーブルに'feature_enabled'カラムを追加"
  - index: "CREATE INDEX idx_feature_lookup ON users(feature_id)"
  
設定:
  - add to: config/settings.py
  - pattern: "FEATURE_TIMEOUT = int(os.getenv('FEATURE_TIMEOUT', '30'))"
  
ルート:
  - add to: src/api/routes.py  
  - pattern: "router.include_router(feature_router, prefix='/feature')"
```

## 検証ループ

### レベル1: 構文とスタイル
```bash
# これらを最初に実行 - 続行前にエラーを修正
ruff check src/new_feature.py --fix  # 可能なものを自動修正
mypy src/new_feature.py              # 型チェック

# 期待: エラーなし。エラーがある場合、エラーを読み取って修正。
```

### レベル2: 各新機能/ファイル/関数のユニットテスト（既存のテストパターンを使用）
```python
# これらのテストケースでtest_new_feature.pyを作成:
def test_happy_path():
    """基本的な機能が動作する"""
    result = new_feature("valid_input")
    assert result.status == "success"

def test_validation_error():
    """無効な入力はValidationErrorを発生させる"""
    with pytest.raises(ValidationError):
        new_feature("")

def test_external_api_timeout():
    """タイムアウトを適切に処理する"""
    with mock.patch('external_api.call', side_effect=TimeoutError):
        result = new_feature("valid")
        assert result.status == "error"
        assert "timeout" in result.message
```

```bash
# 合格するまで実行して反復:
uv run pytest test_new_feature.py -v
# 失敗する場合: エラーを読み取り、根本原因を理解し、コードを修正し、再実行（合格するためにモックしない）
```

### レベル3: 統合テスト
```bash
# サービスを開始
uv run python -m src.main --dev

# エンドポイントをテスト
curl -X POST http://localhost:8000/feature \
  -H "Content-Type: application/json" \
  -d '{"param": "test_value"}'

# 期待: {"status": "success", "data": {...}}
# エラーの場合: logs/app.logでスタックトレースを確認
```

## 最終検証チェックリスト
- [ ] すべてのテストが合格: `uv run pytest tests/ -v`
- [ ] リンティングエラーなし: `uv run ruff check src/`
- [ ] 型エラーなし: `uv run mypy src/`
- [ ] 手動テスト成功: [特定のcurl/コマンド]
- [ ] エラーケースが適切に処理される
- [ ] ログは有益だが冗長ではない
- [ ] 必要に応じてドキュメントが更新される

---

## 避けるべきアンチパターン
- ❌ 既存のものが機能する場合に新しいパターンを作成しない
- ❌ 「動作するはず」だからといって検証をスキップしない  
- ❌ 失敗するテストを無視しない - 修正する
- ❌ 非同期コンテキストで同期関数を使用しない
- ❌ 設定すべき値をハードコードしない
- ❌ すべての例外をキャッチしない - 具体的に