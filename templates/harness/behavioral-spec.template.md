---
id: spec:{FEATURE}
version: 1.0.0
status: draft
coherence:
  depends_on:
    - design:{SCHEMA}
---

# 行動仕様: {FEATURE 名}

<!-- harness-engineering/SKILL.md の Step A-2 で Feature を確定後、
     spec-crystallization/SKILL.md の EARS 形式に従って記述する -->

## 概要

{Feature の目的・スコープ・対象エンティティ}

---

## 要件定義

### REQ-{XXX}-001: {要件名}

<!-- EARS 形式: WHEN/WHILE/IF ... the system shall ... -->

**EARS**:
```
WHEN {トリガー}
  the system shall {動作}
```

**アクター**: {誰が}
**事前条件**: {満たされている必要がある状態}

**受け入れ条件**:
- [ ] {検証可能・自動化可能な条件1}
- [ ] {HTTP ステータスコード / リダイレクト先}
- [ ] {DB への副作用}

**エラー条件**:

| 条件 | 期待動作 | HTTP |
|---|---|---|
| {不正入力} | {エラーメッセージ表示} | 200/400 |
| {未認証} | /login へリダイレクト | 302 |
| {権限不足} | Forbidden / リダイレクト | 403/302 |

**ハーネス依存**: [H-01, H-02, H-03]

---

### REQ-{XXX}-002: {要件名}

<!-- 同形式で続ける -->

---

## 検証不可能な領域と理由

| 領域 | 理由 |
|---|---|
| {フロントエンド JS} | ブラウザ依存 |
| {外部サービス} | H-0x で代替 |
