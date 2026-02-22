# AutoTraderV4 ブランチ・PR ワークフロー

## 必須ルール（全セッション）

**mainブランチへの直接コミットを禁止する。** 必ず以下のワークフローを守ること。

**メインディレクトリ（D:\Projects\AutoTraderV4）は常に main ブランチのまま維持すること。**
`git checkout -b` をメインディレクトリで実行してはならない。
代わりに git worktree を使い、作業は `/d/Projects/AutoTraderV4/tmp/` 配下で行う。

**注意**: `/tmp/` はWindowsのEditツールからアクセス不可。プロジェクト内の `tmp/` を使うこと。

## 作業開始時（worktree使用）

```bash
BRANCH="<type>/<kebab-case-description>"
WORKTREE="/d/Projects/AutoTraderV4/tmp/${BRANCH//\//_}"

# worktrees ディレクトリ作成（初回のみ）
mkdir -p /d/Projects/AutoTraderV4/tmp

# main を最新化
git -C /d/Projects/AutoTraderV4 pull origin main

# ブランチ作成（checkout不要）
git -C /d/Projects/AutoTraderV4 branch "$BRANCH"

# worktree 作成
git -C /d/Projects/AutoTraderV4 worktree add "$WORKTREE" "$BRANCH"
```

ファイル編集はすべて `$WORKTREE/` 配下のパスで行うこと。
（例: `/d/Projects/AutoTraderV4/tmp/feat_add_feature/src/autotrader/...`）

ブランチ命名規則：
- `feat/<内容>` - 新機能追加
- `fix/<内容>` - バグ修正
- `refactor/<内容>` - リファクタリング
- `perf/<内容>` - パフォーマンス改善

## 作業完了時

```bash
# worktree 内でコミット・push
git -C "$WORKTREE" add <files>
git -C "$WORKTREE" commit -m "<type>: <日本語で変更内容を簡潔に>"
git -C "$WORKTREE" push -u origin "$BRANCH"

# PR 作成
"C:/Program Files/GitHub CLI/gh.exe" pr create \
  --repo KaePen/AutoTraderV4 --base main \
  --title "<type>: <日本語で変更内容を簡潔に>" \
  --body "## 変更内容\n- \n\n## テスト\n- [ ] pytest 全PASS確認"

# worktree 削除
git -C /d/Projects/AutoTraderV4 worktree remove "$WORKTREE" --force

# ローカルブランチ削除（worktree削除後に必須）
git -C /d/Projects/AutoTraderV4 branch -d "$BRANCH"
```

PRを作成すると `scripts/pr_watcher.py` が自動検知してレビュー・マージ・ブランチ削除を実施する。

## 禁止事項

- `git checkout -b` をメインディレクトリ（D:\Projects\AutoTraderV4）で実行
- `git push origin main` への直接プッシュ
- `git commit --amend` による公開済みコミットの書き換え
- `--no-verify` によるhookスキップ
