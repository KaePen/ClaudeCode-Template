# Agent Orchestration

## 即座に起動するシーン（ユーザーへの確認不要）

1. 複雑な機能要求 → **planner** エージェント
2. コードを書いた・修正した → **code-reviewer** エージェント
3. バグ修正・新機能 → **tdd-guide** エージェント
4. アーキテクチャ決定 → **architect** エージェント

## 並列実行（必須）

独立したタスクは常に並列起動。逐次実行は無駄:

```
# GOOD: 並列実行
Task 1: auth.py のセキュリティ分析
Task 2: simulator.py のコードレビュー
Task 3: 新機能のTDDガイド

# BAD: 不要な逐次実行
Task 1 完了後に Task 2 を起動...
```

## 多角的分析（複雑な問題）

Split role でサブエージェントを使い分ける:
- 事実確認レビュアー
- シニアエンジニア視点
- セキュリティ専門家視点
- 一貫性チェッカー

## 利用可能エージェント（`~/.claude/agents/`）

| エージェント | 目的 | モデル |
|------------|------|--------|
| planner | 実装計画 | opus |
| architect | システム設計 | opus |
| tdd-guide | TDD強制 | opus |
| code-reviewer | コードレビュー | opus |
| security-reviewer | セキュリティ分析 | opus |
| build-error-resolver | ビルドエラー修正 | sonnet |
| e2e-runner | E2Eテスト | sonnet |
| refactor-cleaner | デッドコード整理 | sonnet |
| doc-updater | ドキュメント更新 | sonnet |
