# Git Worktree ワークフロー原則

## 必須ルール

**mainブランチへの直接コミットを禁止する。**
**メインディレクトリは常に main ブランチのまま維持する。**

作業は必ず `git worktree` を使い、プロジェクト内の `tmp/` 配下で行う。

## 使い方

詳細な手順は `/git-worktree-pr` スキルを参照。
プロジェクト固有の設定（パス・リポジトリ名等）は各プロジェクトの `CLAUDE.md` に記載。

## 禁止事項

- `git checkout -b` をメインディレクトリで実行
- `git push origin main` への直接プッシュ
- `git commit --amend` による公開済みコミットの書き換え
- `--no-verify` によるフックスキップ
- `git push --force` を main/master に実行
