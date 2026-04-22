# Skill: harness-engineering
## Requirements → Harness Engineering Pipeline

任意の要件定義マークダウンを Input として受け取り、VSDD フェーズゲートに整合した
ハーネス設計・テスト実装の全成果物を生成するパイプライン。
**フレームワーク・言語・DB に依存しない汎用スキル。**

---

## 前提: 本スキルと VSDD フェーズの関係

```
VSDD Pipeline
─────────────────────────────────────────────────────────
Phase 1a: EARS形式仕様書              ← spec-crystallization スキル
Phase 1b: 検証アーキテクチャ定義      ← 本スキル (Step B)
Phase 1c: Adversary 仕様レビュー      ← adversarial-review スキル
    ↓ [CEGゲート: coherence-engine スキル]
Phase 2a: 失敗するテスト生成 (Red)    ← 本スキル (Step C)
Phase 2b: 実装 (Green)                ← Builder
Phase 2c: リファクタリング
    ↓
Phase 3: Adversary 実装レビュー       ← adversarial-review スキル
Phase 4: フィードバック統合           ← feedback-routing スキル
Phase 5: 形式検証                     ← formal-hardening スキル
Phase 6: 収束判定                     ← convergence-detection スキル
─────────────────────────────────────────────────────────

本スキルが担う範囲:
  Step A: Input 解析        (Phase 1a の補助)
  Step B: ハーネス設計      (Phase 1b)
  Step C: テスト実装        (Phase 2a)
```

---

## Step A: Input 解析

### A-1: 要件フォーマットの認識

| フォーマット | 識別シグナル | Feature の切り出し単位 |
|---|---|---|
| EARS形式 | `WHEN/WHILE/IF ... shall` | SHALL句の主語 |
| ユーザーストーリー | `As a ... I want ... So that` | ストーリー単位 |
| 機能一覧 | 番号付きリスト / 見出し階層 | 大見出し単位 |
| 画面仕様 | 画面名 + 操作定義 | 画面グループ単位 |
| API仕様 | エンドポイント定義 | リソース単位 |
| 混在 | 上記を組み合わせ識別 | ドメイン概念単位 |

### A-2: Feature 切り出し基準

```
✓ 同一アクターが担う操作群
✓ 同一エンティティのライフサイクル (CRUD + 状態遷移)
✓ 独立してテスト可能な機能境界
✗ 1エンドポイント単位 (細かすぎる)
✗ システム全体を 1 Feature (粗すぎる)
```

### A-3: 境界の識別と H番号マッピング

システム境界を特定し、以下のマッピングでハーネス番号を決定する。
H番号は `coherence/deps.json` の `coherence.depends_on` に必ず登録する。

| 境界の種類 | ハーネス | リアリズム | 実装パターン |
|---|---|---|---|
| SQL DB (RDBMS) | H-01 | Lv 2 | SQLite in-memory / 同一スキーマ再現 |
| NoSQL / KV Store | H-01 | Lv 1 | dict / list による Fake |
| 認証・認可 (session) | H-02 | Lv 1 | ロール別固定ユーザーオブジェクト |
| 認証・認可 (JWT) | H-02 | Lv 1 | デコード済みペイロードのモック |
| CSRF / セキュリティ | H-03 | Lv 0 | AsyncMock bypass |
| 外部 HTTP API | H-04 | Lv 0 | `responses` / `httpretty` / `nock` |
| メール送信 | H-05 | Lv 0 | 送信リストを記録するスパイ |
| ファイルシステム | H-06 | Lv 1 | `tmp_path` / `MemoryFS` |
| 現在時刻 | H-07 | Lv 0 | `freeze_time` / `jest.useFakeTimers` |
| 外部決済 | H-08 | Lv 0 | 固定レスポンス返却 |
| 乱数 / UUID | H-09 | Lv 0 | seed 固定 / mock |

> **ルール**: 新たなハーネスを定義したら `templates/harness/harness-spec.template.md` を
> `tests/harness/{name}.md` としてコピーし記入する。

---

## Step B: Phase 1b — ハーネス設計 (verification-arch.md)

Phase 1a の EARS仕様書が完成したら以下を生成する。

### B-1: verification-arch.md の生成

`templates/harness/verification-arch.template.md` を使い、Feature ごとに生成する:

```
.vcsdd/features/{feature}/specs/verification-arch.md
```

frontmatter の `coherence.depends_on` を必ず記入すること:

```yaml
---
id: verification:{feature}
linked-spec: spec:{feature}
harnesses: [H-01, H-02]
coherence:
  depends_on:
    - spec:{feature}
    - design:{schema}
---
```

### B-2: harness-spec.md の生成

使用するハーネスごとに `templates/harness/harness-spec.template.md` を記入する:

```
tests/harness/{H-NN}-{name}.md
```

### B-3: CEG 依存グラフの更新

`coherence/deps.json` を更新し、各ハーネスと Feature の依存を宣言する:

```json
{
  "nodes": {
    "H-01": { "file": "tests/harness/db.py", "realism": 2 },
    "feature:{name}": { "tests": ["tests/integration/test_{name}.py"] }
  },
  "edges": [
    { "from": "H-01", "to": "feature:{name}", "reason": "..." }
  ],
  "propagationRules": {
    "H-01_changed": "Re-run: feature:{name}, feature:{other}"
  }
}
```

### B-4: Phase 1b ゲートチェック (Adversary に渡す前)

```
[ ] verification-arch.md が全 Feature に存在するか
[ ] harness-spec.md が全 H番号に存在するか
[ ] coherence.depends_on に spec:{feature} が宣言されているか
[ ] 循環依存が deps.json に存在しないか
```

---

## Step C: Phase 2a — テスト実装 (Red)

### C-1: conftest.py の設計原則

```python
# 環境変数は全 import より前に設定する (E402 lint 例外対象)
import os
os.environ.setdefault("SECRET_KEY", "test-only")

# 役割別クライアントフィクスチャの命名規則
# anon_client        → 未認証 (H-03)
# {role}_client      → 認証済み (H-02{role} + H-03)
# mem_db             → H-01 DB ハーネスのインスタンス

# テストデータ投入ヘルパーの命名規則
# seed_{entity}()    → エンティティごとのデータ挿入関数
```

### C-2: ユニットテスト生成方針

```
対象: ビジネスロジック関数 (DB・HTTP に依存しない純粋関数)
ハーネス: MagicMock / FakeObject のみ
命名: test_{関数名}_{シナリオ}
     例: test_verify_password_valid
         test_verify_password_invalid_hash
         test_get_current_user_no_session

必須ケース:
  [ ] 正常ケース (ゴールデンパス)
  [ ] 境界値 (最小 / 最大 / 空 / null)
  [ ] エラーケース (認証失敗 / 権限不足 / 入力不正)
  [ ] ガード節の全分岐 (early return 条件を網羅)
```

### C-3: 統合テスト生成方針

```
対象: ルート / コントローラー (HTTP レベルの I/O)
ハーネス: H-01 + H-02 + H-03 (+ 必要な H-04 以降)
命名: test_{エンドポイントの意味}_{シナリオ}
     例: test_create_reservation_valid
         test_create_reservation_duplicate
         test_create_reservation_unauthorized

検証内容:
  [ ] HTTP ステータスコード
  [ ] リダイレクト先 URL
  [ ] DB への副作用 (INSERT / UPDATE / DELETE)
  [ ] エラーレスポンスのメッセージ
  [ ] 認証ガードのリダイレクト
  [ ] ロール別アクセス制御 (viewer が user ルートにアクセス等)
```

### C-4: Red 確認ゲート

Phase 2a の終了条件:

```bash
# 全テストが Red (失敗) であることを確認する
pytest tests/ --tb=no -q
# → 全件 FAILED、PASSED が 0 件であること
```

---

## Step D: Phase 2b — 実装後チェック

```bash
# 全テストが Green になること
pytest tests/ --tb=short -q
# → 全件 PASSED

# カバレッジ基準 (VSDD デフォルト)
pytest tests/unit/       --cov --cov-fail-under=90
pytest tests/integration/ --cov --cov-fail-under=80
```

---

## AI への実行指示フォーマット

本スキルを AI に実行させる際の推奨プロンプト:

```
Step A (Input 解析):
  「.vcsdd/skills/harness-engineering/SKILL.md の Step A を実行し、
   {要件ファイルパス} から Feature 一覧と境界マップを抽出してください。」

Step B (ハーネス設計):
  「Step A で確定した Feature について Phase 1b の全成果物を生成してください。
   templates/harness/ 配下のテンプレートを使用してください。」

Step C (テスト実装):
  「Phase 2a に進み、tests/ 配下にハーネスとテストを実装してください。
   全テストが Red であることを確認してから報告してください。」

Step D (実装 → Green):
  「Phase 2b を実施し、全テストが Green になるまで実装してください。
   完了後 pytest の結果を報告してください。」
```

---

## ハーネスリアリズム段階

| Level | 説明 | 典型ハーネス |
|---|---|---|
| 0 | 固定値返却 / bypass のみ | H-03, H-07, H-08 |
| 1 | 入力に応じた動的応答 | H-02, H-04, H-05 |
| 2 | 本番仕様との乖離ゼロ保証 | H-01 (SQL DB) |
| 3 | 本番同等のレイテンシ・エラー率 | E2E テスト層のみ |

> Lv 0/1 のハーネスは必ず `coherence/deps.json` に登録し、
> Adversary が「ハーネスが本番乖離を生んでいないか」を Phase 3 で検証する。

---

## 関連スキル

| スキル | 役割 | フェーズ |
|---|---|---|
| `spec-crystallization` | EARS形式仕様書生成 | Phase 1a |
| `coherence-engine` | CEG依存グラフ管理・ゲートチェック | Phase 1b / 全体 |
| `adversarial-review` | 仕様・実装の敵対的レビュー | Phase 1c / 3 |
| `feedback-routing` | FAIL差し戻しルーティング | Phase 4 |
| `formal-hardening` | プロパティテスト・ファジング | Phase 5 |
| `convergence-detection` | 4条件収束判定 | Phase 6 |
