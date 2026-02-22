---
name: git-worktree-pr
description: Use this skill when starting work on any new feature, bug fix, or refactoring. Provides the standard git worktree + PR workflow: keeps main branch clean by using isolated worktrees, commits, and pull requests.
---

# Git Worktree + PR ワークフロー

mainブランチを常にクリーンに保つ、再利用可能なブランチ戦略。

---

## セットアップ（プロジェクト変数を設定）

```bash
# ── プロジェクトに合わせて変更する箇所 ──────────────────
PROJECT_ROOT="/path/to/your/project"   # 例: /d/Projects/MyApp
WORKTREE_BASE="$PROJECT_ROOT/tmp"      # worktree を置くディレクトリ
GITHUB_REPO="owner/repo"               # 例: KaePen/AutoTraderV4
# ─────────────────────────────────────────────────────

BRANCH="<type>/<kebab-case-description>"
WORKTREE="$WORKTREE_BASE/${BRANCH//\//_}"
```

---

## 作業開始

```bash
# worktree ディレクトリを確保
mkdir -p "$WORKTREE_BASE"

# main を最新化
git -C "$PROJECT_ROOT" pull origin main

# ブランチ作成（main ディレクトリは checkout しない）
git -C "$PROJECT_ROOT" branch "$BRANCH"

# worktree 作成（ここで作業する）
git -C "$PROJECT_ROOT" worktree add "$WORKTREE" "$BRANCH"
```

> **ポイント**: メインディレクトリは常に `main` ブランチのまま。
> `git checkout -b` をメインディレクトリで実行してはならない。

---

## ブランチ命名規則

| プレフィックス | 用途 |
|-------------|------|
| `feat/<name>` | 新機能追加 |
| `fix/<name>` | バグ修正 |
| `refactor/<name>` | リファクタリング |
| `perf/<name>` | パフォーマンス改善 |
| `chore/<name>` | 設定・依存更新 |

---

## コミット（worktree 内で実施）

```bash
# ファイルを追加
git -C "$WORKTREE" add <files>

# コミット（Conventional Commits 形式）
git -C "$WORKTREE" commit -m "<type>: <変更内容を簡潔に>"
```

### Conventional Commits 形式

```
<type>: <description>

[optional body]
```

types: `feat` / `fix` / `refactor` / `docs` / `test` / `chore` / `perf` / `ci`

---

## PR 作成

```bash
# push
git -C "$WORKTREE" push -u origin "$BRANCH"

# PR 作成（GitHub CLI）
gh pr create \
  --repo "$GITHUB_REPO" \
  --base main \
  --title "<type>: <タイトル>" \
  --body "$(cat <<'EOF'
## 変更内容
-

## テスト
- [ ] テスト全PASS確認
- [ ] 動作確認済み
EOF
)"
```

---

## クリーンアップ

```bash
# worktree 削除
git -C "$PROJECT_ROOT" worktree remove "$WORKTREE" --force

# ローカルブランチ削除（必須）
git -C "$PROJECT_ROOT" branch -d "$BRANCH"
```

> **注**: PR マージ後はリモートブランチも削除すること（GitHub の自動削除設定推奨）。

---

## 禁止事項

- `git checkout -b` をメインディレクトリで実行
- `git push origin main` への直接プッシュ
- `git commit --amend` による公開済みコミットの書き換え
- `--no-verify` によるフックスキップ
- `git push --force` を main/master に実行

---

## Windows 固有の注意

```bash
# Windows で gh コマンドが見つからない場合
"C:/Program Files/GitHub CLI/gh.exe" pr create ...

# worktree のパスは Unix 形式で指定
# 例: /d/Projects/MyApp/tmp/feat_xxx
#      ↑ D:\Projects\MyApp\tmp\feat_xxx の Unix 形式
```

---

## プロジェクト別設定メモ

プロジェクトの `CLAUDE.md` に以下を記録しておくと、このスキルを呼び出したときにすぐ使える:

```markdown
## Git Worktree 設定
- PROJECT_ROOT: /d/Projects/MyApp
- WORKTREE_BASE: /d/Projects/MyApp/tmp
- GITHUB_REPO: owner/repo-name
```
