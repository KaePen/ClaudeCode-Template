---
name: python-tdd
description: Use this skill when writing new Python features, fixing bugs, or refactoring. Enforces pytest-based TDD with 80%+ coverage. Replaces the JS/TS-focused tdd-workflow for Python projects.
---

# Python TDD ワークフロー（pytest）

このスキルはPython/pytestを使ったTDD（テスト駆動開発）を強制します。

## 基本原則

1. **テストを先に書く（RED）**
2. **最小実装でテストを通す（GREEN）**
3. **リファクタリング（IMPROVE）**
4. **カバレッジ80%以上を確認**

---

## Step 1: テストファイルを先に作成

```python
# tests/unit/test_new_feature.py
from __future__ import annotations

import pytest
from autotrader.module import new_function


class TestNewFunction:
    """new_function のテストスイート。"""

    def test_正常系_基本動作(self) -> None:
        """正常入力で期待値を返す。"""
        result = new_function(input_value=10)
        assert result == expected_value

    def test_境界値_最小値(self) -> None:
        """最小有効値で動作する。"""
        result = new_function(input_value=0)
        assert result is not None

    def test_境界値_最大値(self) -> None:
        """最大有効値で動作する。"""
        result = new_function(input_value=100)
        assert result >= 0

    def test_エラー系_無効入力(self) -> None:
        """無効な入力でValueErrorを発生させる。"""
        with pytest.raises(ValueError, match="無効な入力"):
            new_function(input_value=-1)

    def test_エラー系_None入力(self) -> None:
        """None入力でTypeErrorを発生させる。"""
        with pytest.raises(TypeError):
            new_function(input_value=None)
```

## Step 2: テストを実行して失敗を確認

```bash
cd /d/Projects/AutoTraderV4
python -m pytest tests/unit/test_new_feature.py -v
# → FAILED（実装がないので正常）
```

## Step 3: 最小実装を書く

```python
# src/autotrader/module.py
from __future__ import annotations


def new_function(input_value: int) -> float:
    """関数の説明。

    Args:
        input_value (int): 入力値（0以上100以下）

    Returns:
        float: 計算結果

    Raises:
        TypeError: input_valueがNoneの場合
        ValueError: input_valueが負の値の場合
    """
    if input_value is None:
        raise TypeError("input_valueはNoneにできません")
    if input_value < 0:
        raise ValueError("無効な入力: 負の値は許可されていません")
    return float(input_value) * 1.5
```

## Step 4: テストが通ることを確認

```bash
python -m pytest tests/unit/test_new_feature.py -v
# → PASSED
```

## Step 5: カバレッジを確認

```bash
python -m pytest tests/ --cov=src/autotrader --cov-report=term-missing -q
# → 80%以上であることを確認
```

---

## AutoTraderV4 固有のパターン

### バックテスト関連テスト

```python
# tests/unit/backtest/test_simulator_feature.py
from __future__ import annotations

import pandas as pd
import pytest
from autotrader.backtest.simulator import TradeSimulator, SimulatorConfig


@pytest.fixture
def simulator_config() -> SimulatorConfig:
    """標準的なテスト用シミュレータ設定。"""
    return SimulatorConfig(
        symbol="USDJPY",
        spread=0.003,
        max_positions=2,
    )


@pytest.fixture
def sample_ohlcv() -> pd.DataFrame:
    """テスト用OHLCVデータ。"""
    dates = pd.date_range("2023-01-01", periods=100, freq="1h")
    return pd.DataFrame({
        "open": [100.0] * 100,
        "high": [101.0] * 100,
        "low": [99.0] * 100,
        "close": [100.5] * 100,
        "volume": [1000] * 100,
    }, index=dates)


class TestSimulatorFeature:
    """シミュレータ新機能のテスト。"""

    def test_新機能_初期化(
        self, simulator_config: SimulatorConfig
    ) -> None:
        """新機能が正しく初期化される。"""
        sim = TradeSimulator(config=simulator_config)
        assert sim is not None

    def test_新機能_計算結果(
        self, simulator_config: SimulatorConfig, sample_ohlcv: pd.DataFrame
    ) -> None:
        """新機能が正しい値を計算する。"""
        sim = TradeSimulator(config=simulator_config)
        result = sim.new_method(sample_ohlcv)
        assert result > 0
```

### モック使用例

```python
from unittest.mock import MagicMock, patch


def test_外部依存をモック(simulator_config):
    """外部データ取得をモックしてテスト。"""
    mock_data = pd.DataFrame({"close": [100.0] * 10})

    with patch(
        "autotrader.backtest.runner.load_data",
        return_value=mock_data
    ):
        result = some_function_using_load_data()
        assert result is not None
```

---

## テストチェックリスト

完了前に確認:

- [ ] 正常系テストがある
- [ ] エラー系テストがある（None, 負値, 空リスト等）
- [ ] 境界値テストがある
- [ ] カバレッジ80%以上
- [ ] テストが独立している（前のテストに依存しない）
- [ ] テスト名が日本語で何をテストするか明確

## よく使うコマンド

```bash
# 特定ファイルのみ高速実行
python -m pytest tests/unit/test_new_feature.py -v

# 失敗時に即停止（開発中に有効）
python -m pytest tests/ -x -q

# カバレッジ確認
python -m pytest tests/ --cov=src/autotrader --cov-report=term-missing -q

# キーワードでフィルタ
python -m pytest tests/ -k "test_simulator" -v
```
