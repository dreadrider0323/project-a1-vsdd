# VSDD — Verified Spec-Driven Development
## ハーネスエンジニアリング基盤

VSDDとCoDDを統合した**VCSDD（Verified Coherence Spec-Driven Development）**の
Claude Skill・オーケストレーション実装です。

## ディレクトリ構成

```
project-a1-vsdd/
├── README.md
├── CLAUDE.md                         # Claude Code 向けコンテキスト定義
├── skills/                           # Claude Skills（各フェーズの専門知識）
│   ├── spec-crystallization/         # Phase 1a: EARS形式仕様書生成
│   ├── harness-engineering/          # Phase 1b+2a: 要件→ハーネス変換パイプライン ★
│   ├── adversarial-review/           # Phase 1c/3: 敵対的レビュー（Adversary）
│   ├── feedback-routing/             # Phase 4: FAIL差し戻しルーティング
│   ├── formal-hardening/             # Phase 5: プロパティテスト・ファジング
│   ├── convergence-detection/        # Phase 6: 収束判定
│   ├── traceability/                 # Chainlink Bead トレーサビリティ
│   ├── coherence-engine/             # CoDD: CEG依存グラフ管理
│   └── sprint-contracts/             # Strictモード: スプリント契約
├── orchestration/                    # エージェント間オーケストレーション
├── hooks/                            # フェーズゲート自動チェック
├── scripts/                          # コアスクリプト
│   └── lib/                          # coherence.js / bead.js / gate.js
├── templates/                        # 成果物テンプレート
│   ├── ears/                         # EARS形式テンプレート
│   ├── coherence/                    # CEG依存グラフテンプレート
│   ├── findings/                     # Adversary FINDING テンプレート
│   └── harness/                      # ハーネス関連テンプレート ★
│       ├── behavioral-spec.template.md
│       ├── verification-arch.template.md
│       └── harness-spec.template.md
└── docs/                             # 固有設定
```

## ハーネスエンジニアリング クイックスタート

要件定義マークダウンを Input としてハーネス設計とテスト実装を一括生成:

```bash
# Step A: Input 解析 (Feature 一覧と境界マップを抽出)
# AI への指示:
# 「skills/harness-engineering/SKILL.md の Step A を実行し、
#   {要件ファイルパス} から Feature 一覧と境界マップを抽出してください。」

# Step B: Phase 1b — ハーネス設計
# AI への指示:
# 「Step A の結果をもとに Phase 1b の全成果物を生成してください。
#   templates/harness/ 配下のテンプレートを使用してください。」

# Step C: Phase 2a — テスト実装 (Red)
# AI への指示:
# 「Phase 2a を実施し、全テストが Red の状態で完了報告してください。」

# Step D: Phase 2b — 実装 (Green)
# AI への指示:
# 「Phase 2b を実施し、全テストが Green になるまで実装してください。」
```

---

## VSDD フルパイプライン クイックスタート

```bash
# 1. フィーチャーパイプライン初期化
./scripts/vcsdd-init.sh <feature-name> [--mode lean|strict]

# 2. 仕様結晶化
./scripts/vcsdd-spec.sh

# 3. Adversaryレビュー（必ず新規ターミナルで）
./scripts/vcsdd-adversary.sh <feature-name> --phase 1c

# 4. TDDゲートチェック
./hooks/pre-tdd-gate.sh <feature-name>

# 5. 実装後Adversaryレビュー（必ず新規ターミナルで）
./scripts/vcsdd-adversary.sh <feature-name>

# 6. フィードバック（FAILの場合）
./scripts/vcsdd-feedback.sh <feature-name>

# 7. 形式検証
./scripts/vcsdd-harden.sh <feature-name> --lang python

# 8. 収束判定
./scripts/vcsdd-converge.sh <feature-name>

# ステータス確認
./scripts/vcsdd-status.sh
```

## 4エージェント構成

| エージェント | モデル | 役割 |
|---|---|---|
| Orchestrator | claude-sonnet | パイプライン調整・ゲートチェック |
| Builder | claude-sonnet | 仕様執筆・テスト生成・実装 |
| Adversary | claude-opus（**毎回フレッシュコンテキスト**） | 敵対的レビュー・PASS/FAIL判定 |
| Verifier | claude-sonnet | 形式検証調整 |

## 収束条件（4条件すべてが必須）

1. ✅ 仕様 Adversary PASS
2. ✅ テスト全GREEN
3. ✅ 実装 Adversary PASS
4. ✅ CEG整合性 PASS
