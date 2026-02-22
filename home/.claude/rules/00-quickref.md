# Claude Code クイックリファレンス

## エージェント起動タイミング（自動起動・確認不要）

| エージェント | いつ起動するか | モデル |
|------------|--------------|--------|
| **planner** | 3ステップ以上・設計判断が必要な機能実装の**前** | opus |
| **code-reviewer** | コードを書いた・修正した**後**（必須） | opus |
| **python-reviewer** | Python ファイルを書いた後の軽量チェック | sonnet |
| **tdd-guide** | 新機能・バグ修正の**前**（テスト先行） | opus |
| **architect** | アーキテクチャ変更・新サービス設計の前 | opus |
| **build-error-resolver** | ビルドエラー・型エラー発生時 | sonnet |
| **security-reviewer** | 認証・入力処理・APIエンドポイント実装後 | opus |
| **refactor-cleaner** | デッドコード削除・大規模リファクタリング | sonnet |
| **e2e-runner** | クリティカルなフロー変更後 | sonnet |

## 基本ワークフロー

```
要求受領 → 複雑？→ EnterPlanMode
         → tdd-guide でテスト先行
         → 実装
         → python-reviewer or code-reviewer
         → テスト確認（動作証明）
         → PR作成
```

## 並列実行（常に最大化）

独立したタスクは**必ず同時に**起動:
```
# 良い例（並列）
Task 1: python-reviewer for simulator.py
Task 2: security-reviewer for auth endpoints
Task 3: tdd-guide for new feature tests

# 悪い例（無駄な逐次実行）
Task 1 完了後に Task 2 を起動...
```

## ルールファイル早見表

| ファイル | 内容 |
|---------|------|
| `core.md` | ワークフロー・完了基準・バグ修正方針 |
| `agents.md` | エージェント並列実行・多角的分析 |
| `coding-style2.md` | Python コーディング規約（PEP8・型ヒント等） |
| `git-workflow1.md` | Git コミット・PR の一般ルール |
| `git-workflow2.md` | AutoTraderV4 専用 worktree ワークフロー |
| `security.md` | セキュリティチェックリスト |
| `testing.md` | テスト要件（80%カバレッジ） |
| `performance.md` | モデル選択（Haiku/Sonnet/Opus）基準 |
| `hooks.md` | フック設定の説明 |

## スキル早見表（`/skill-name` で呼び出し）

| スキル | 用途 |
|-------|------|
| `/python-tdd` | Python/pytest でのTDDワークフロー |
| `/autotrader-backtest` | バックテスト機能開発の標準手順 |
| `/strategic-compact` | コンテキスト節約タイミングの提案 |
| `/continuous-learning` | セッションからパターンを抽出・保存 |
| `/code-review` | PR コードレビュー |

## 自動フック（設定済み）

| イベント | 対象 | 動作 |
|---------|------|------|
| PostToolUse | `.py` ファイル編集後 | ruff check + format 自動実行 |
| PostToolUse | `.py` ファイル編集後 | `print()` 使用を警告 |
| PostToolUse | `.ts/.tsx/.js/.jsx` 編集後 | Prettier 自動実行 |
| Stop | セッション終了時 | Python `print()` 監査 |
| Stop | セッション終了時 | JS/TS `console.log` 監査 |

## MCP サーバー（オプション・追加可能）

インストール後 `~/.claude/settings.json` の `mcpServers` に追加:

| サーバー | インストール | 用途 |
|---------|------------|------|
| `mcp-server-git` | `pip install mcp-server-git` | 高度な git 操作 |
| GitHub Official | `npx @modelcontextprotocol/server-github` | PR・Issue 管理 |
| `semgrep-mcp` | `pipx install semgrep-mcp` | セキュリティスキャン |
| yfinance MCP | `pip install mcp yfinance` | 無料金融データ |

```json
// ~/.claude/settings.json に追加
"mcpServers": {
  "git": {
    "command": "python",
    "args": ["-m", "mcp_server_git", "--repository", "."]
  },
  "github": {
    "command": "npx",
    "args": ["-y", "@modelcontextprotocol/server-github"],
    "env": { "GITHUB_PERSONAL_ACCESS_TOKEN": "<your-token>" }
  }
}
```
