---
name: autotrader-backtest
description: Use this skill when developing new backtest features, running comparisons, or creating analysis reports for AutoTraderV4. Provides standard workflow for worktree-based feature development.
---

# AutoTraderV4 バックテスト開発ワークフロー

新機能開発・比較検証・レポート作成の標準手順。

---

## 1. Worktree セットアップ

```bash
BRANCH="feat/<機能名>"  # 例: feat/dynamic-pip-value
WORKTREE="/d/Projects/AutoTraderV4/tmp/${BRANCH//\//_}"

# main を最新化
git -C /d/Projects/AutoTraderV4 pull origin main

# ブランチ作成 + worktree 作成
git -C /d/Projects/AutoTraderV4 branch "$BRANCH"
git -C /d/Projects/AutoTraderV4 worktree add "$WORKTREE" "$BRANCH"
```

> **重要**: ファイル編集は `$WORKTREE/` 配下のパスで行う

---

## 2. 開発フロー

### ファイル編集の場所
```
$WORKTREE/src/autotrader/backtest/simulator.py  # シミュレータ本体
$WORKTREE/src/autotrader/backtest/runner.py     # バックテスト実行
$WORKTREE/src/autotrader/backtest/service.py    # 並列処理サービス
```

### テスト実行（worktree内）
```bash
cd "$WORKTREE"
python -m pytest tests/ -x -q --tb=short

# カバレッジ確認
python -m pytest tests/ --cov=src/autotrader --cov-report=term-missing -q
```

---

## 3. 比較スクリプト作成

比較スクリプトは `$WORKTREE/reports/` に配置:

```python
# reports/<feature>_comparison.py
"""<機能名> 比較スクリプト。

使い方:
    python reports/<feature>_comparison.py [--no-m1] [--year YEAR]

オプション:
    --no-m1   M1データをスキップして高速実行
    --year    対象年（デフォルト: 2023）
"""
from __future__ import annotations

import argparse
import sys
from pathlib import Path

# data_dir の解決（worktree対応）
PROJECT_ROOT = Path(__file__).parent.parent
DATA_DIR = PROJECT_ROOT / "data"
if not (DATA_DIR / "USDJPY").exists():
    # worktreeの場合はメインプロジェクトのdataを使用
    MAIN_PROJECT = Path("/d/Projects/AutoTraderV4")
    DATA_DIR = MAIN_PROJECT / "data"

sys.path.insert(0, str(PROJECT_ROOT / "src"))
```

### レポート出力形式
```python
# 結果はreports/フォルダに出力
REPORT_PATH = PROJECT_ROOT / "reports" / f"<feature>_result.txt"

with open(REPORT_PATH, "w", encoding="utf-8") as f:
    f.write("# 比較結果\n\n")
    f.write("## パラメータ\n")
    f.write(f"- 対象通貨ペア: {symbols}\n")
    f.write(f"- 対象期間: {years}\n\n")
    f.write("## 結果\n\n")
    # テーブル形式で出力
```

---

## 4. クイック検証（Phase 1）

2023年データのみで素早く確認:

```bash
# worktree内で実行
python reports/<feature>_comparison.py --year 2023 --no-m1
```

確認ポイント:
- [ ] ゼロトレードでないか
- [ ] 基準ケースより改善しているか
- [ ] エラーが出ていないか

---

## 5. PR 作成

```bash
# worktree内でコミット
git -C "$WORKTREE" add src/ tests/ reports/
git -C "$WORKTREE" commit -m "feat: <変更内容を日本語で>"
git -C "$WORKTREE" push -u origin "$BRANCH"

# PR 作成
"C:/Program Files/GitHub CLI/gh.exe" pr create \
  --repo KaePen/AutoTraderV4 --base main \
  --title "feat: <変更内容>" \
  --body "$(cat <<'EOF'
## 変更内容
-

## 検証結果
| 設定 | 利益 | トレード数 | Sharpe |
|------|------|----------|--------|
| ベースライン | | | |
| 新機能 | | | |

## テスト
- [ ] pytest 全PASS確認
- [ ] 比較レポート確認
EOF
)"
```

---

## 6. Worktree クリーンアップ

```bash
git -C /d/Projects/AutoTraderV4 worktree remove "$WORKTREE" --force
git -C /d/Projects/AutoTraderV4 branch -d "$BRANCH"
```

---

## よくあるトラブル

### ゼロトレード問題
```python
# worktreeではUSDJPYデータが存在しない
# → data_dir の fallback ロジックを確認
DATA_DIR = PROJECT_ROOT / "data"
if not (DATA_DIR / "USDJPY").exists():
    DATA_DIR = Path("/d/Projects/AutoTraderV4") / "data"
```

### import エラー（sys.path 問題）
```python
# worktree内スクリプトの先頭に必ず追加
import sys
from pathlib import Path
PROJECT_ROOT = Path(__file__).parent.parent
sys.path.insert(0, str(PROJECT_ROOT / "src"))
```

### 並列処理がハングする
```bash
# シリアル実行で確認
python scripts/run_backtest.py --symbol USDJPY --year 2023 --no-parallel
```

---

## 主要ファイル構成

```
src/autotrader/backtest/
├── simulator.py      # TradeSimulator, SimulatorConfig
├── runner.py         # BacktestRunner, BacktestConfig
├── service.py        # 並列バックテスト実行
├── file_listener.py  # イベントログ
└── ...

tests/
├── unit/backtest/    # バックテストユニットテスト
└── ...

reports/              # 比較スクリプト・結果ファイル
scripts/
└── run_backtest.py   # バックテスト実行エントリポイント
```
