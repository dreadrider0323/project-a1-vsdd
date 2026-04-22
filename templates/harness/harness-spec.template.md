---
harness: H-{NN}
name: {HarnessName}
realism: 0
boundary: {DB | Auth | CSRF | HTTP | Mail | FS | Clock | Payment | Rand}
replaces: {本番コードのどのモジュール / クラス / 関数を差し替えるか}
file: tests/harness/{name_lower}.py
coherence:
  depends_on: []
---

# H-{NN}: {HarnessName}

<!-- harness-engineering/SKILL.md の Step B-2 に従って生成する -->

## 境界定義

{このハーネスが何を代替するか。本番コードのどの接続点をモックするか。}

例:
- H-01: psycopg2 DBConnection を SQLite in-memory で代替する
- H-02: セッション検証関数を固定ユーザーオブジェクトで代替する
- H-03: CSRF トークン検証を AsyncMock で無効化する

---

## インターフェース互換性

本番コードが呼ぶメソッド / 関数と同名・同シグネチャを提供すること:

| 本番コード | ハーネス実装 | 戻り値の型 |
|---|---|---|
| `{OriginalClass.method(args)}` | `{HarnessClass.method(args)}` | `{Type}` |

---

## リアリズムレベル根拠

採用レベル: **{0 | 1 | 2 | 3}**

理由: {なぜこのレベルで十分か / なぜそれ以上が不要か}

---

## 実装スケルトン (Python)

```python
# tests/harness/{name_lower}.py

class {HarnessName}:
    """H-{NN}: {境界の説明} (realism={N})"""

    def __init__(self):
        pass  # 初期化処理

    def {method}(self, *args, **kwargs):
        # 本番コードと同じシグネチャで実装
        pass
```

## pytest フィクスチャ例

```python
# tests/conftest.py
import pytest
from tests.harness.{name_lower} import {HarnessName}

@pytest.fixture
def {fixture_name}():
    return {HarnessName}()
```

---

## このハーネスが担保すべきテストケース

- [ ] {正常系}
- [ ] {異常系}
- [ ] {境界値}

---

## 既知の差異と注意事項

| 項目 | 本番の動作 | ハーネスの動作 | テストへの影響 |
|---|---|---|---|
| {差異1} | {本番} | {ハーネス} | {影響} |

---

## CoDD 登録 (coherence/deps.json)

```json
{
  "from": "H-{NN}",
  "to": "feature:{name}",
  "reason": "{なぜ依存するか}"
}
```
