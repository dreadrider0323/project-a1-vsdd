---
id: verification:{FEATURE}
linked-spec: spec:{FEATURE}
harnesses: [H-01, H-02, H-03]
coherence:
  depends_on:
    - spec:{FEATURE}
    - design:{SCHEMA}
---

# 検証アーキテクチャ: {FEATURE 名}

<!-- harness-engineering/SKILL.md の Step B-1 に従って生成する -->
<!-- linked-spec が Phase 1a で Adversary PASS を得てから本ファイルを作成すること -->

## テスト戦略

### ユニットテスト (`tests/unit/test_{module}.py`)
- **対象**: {ビジネスロジック関数・バリデーション・純粋関数}
- **ハーネス**: MagicMock / FakeObject のみ

### 統合テスト (`tests/integration/test_{feature}.py`)
- **対象**: `/{prefix}/*` エンドポイント全件
- **ハーネス**: H-01 + H-02({role}) + H-03

---

## テストケース一覧

### ユニットテスト

| テスト名 | 検証内容 | REQ 参照 |
|---|---|---|
| `test_{function}_valid` | 正常ケース | REQ-{XXX}-001 |
| `test_{function}_invalid` | 異常ケース | REQ-{XXX}-001 |
| `test_{function}_boundary` | 境界値 | REQ-{XXX}-001 |
| `test_{function}_no_session` | 未認証ガード | REQ-{XXX}-001 |

### 統合テスト

| テスト名 | 検証内容 | REQ 参照 |
|---|---|---|
| `test_{action}_valid` | 正常フロー → 302 + DB 変化 | REQ-{XXX}-001 |
| `test_{action}_duplicate` | 重複操作 → エラーフォーム | REQ-{XXX}-002 |
| `test_{action}_unauthorized` | 未認証 → 302 /login | REQ-{XXX}-001 |
| `test_{action}_forbidden` | 権限不足 → 403/302 | REQ-{XXX}-001 |
| `test_{action}_invalid_input` | バリデーション違反 → エラー | REQ-{XXX}-001 |
| `test_{action}_state_transition` | 状態遷移の逆順防止 | REQ-{XXX}-003 |

---

## 検証不可能な領域と理由

| 領域 | 理由 |
|---|---|
| {フロントエンド JS の動作} | ブラウザ依存 / E2E テスト層で対応 |
| {外部サービスの応答} | H-0x ハーネスで代替済み |
| {パフォーマンス特性} | 負荷テストで別途対応 |
