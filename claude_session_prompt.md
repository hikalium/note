# Claude セッション開始プロンプト: note リポジトリを永続 memory として使う

このリポジトリ (`github.com/hikalium/note`) を、新規 Claude セッションの **クロスセッション memory store** として使うための取扱説明書。下のテンプレートを新しい Claude セッションの最初に貼ると、その session はこのリポジトリを読みに行ってから作業を始める。

---

## 1. 貼り付けるプロンプト

新規 Claude セッション (Claude Code でも claude.ai でも) を開いたら、まず下記を貼る。`<PAT>` の部分は手元で作成した token に置き換える ([PAT 作成手順](#2-fine-grained-pat-の作成-初回のみ) は下にある)。

````
あなたは hikalium と作業します。https://github.com/hikalium/note を、
あなたのクロスセッション memory store として扱ってください。実質的な作業に
入る前に以下のセットアップを実施してください。

## セットアップ

1. PAT を `~/.netrc` に保存 (chmod 600)。token は assistant 応答に echo
   しないこと。

   ```
   machine github.com
     login x-access-token
     password <PAT>
   machine api.github.com
     login x-access-token
     password <PAT>
   ```

2. note リポジトリを clone (既にあれば pull):
   `git clone https://github.com/hikalium/note /work/note`
   `git -C /work/note pull`

3. README.md と関連トピックの .md を読んで context を頭に入れる。今回の
   task と関連しそうなファイルは:
   - `claude_code.md` — Claude Code 権限設定の runbook
   - `git.md`, `github.md` — git/GitHub 周りの慣習
   - `qemu.md`, `qemu-device.md` — QEMU 関連 (該当する場合)
   - その他、ファイル名がトピックと一致するもの

## セッション中の振る舞い

- このリポジトリから情報を **引く** ときは、ファイル名から内容を当ててから
  Read で確認する。grep が必要なら `grep -r 'keyword' /work/note` を使う
- 汎用性の高い知見 (workflow tips, gotcha, runbook 等) を新たに得たら、
  該当ファイルに append するか新規 .md を作成して **push してください**
- スタイル: terse, command-oriented, inline URL で関連リンクを散りばめる
  (例: `qemu.md`, `git.md` を参照)

## 変更の push 方法

`main` ブランチは保護されているので直接 push 不可。手順:

1. 新規 branch を切る (`git -C /work/note checkout -b <topic-branch>
   origin/main`)
2. 変更を commit (author は `hikalium <hikalium@hikalium.com>`)
3. push: `git -C /work/note push -u origin <topic-branch>`
4. PR を API 経由で open:
   ```
   curl -sS -n -X POST \
        -H "Accept: application/vnd.github+json" \
        -H "X-GitHub-Api-Version: 2022-11-28" \
        https://api.github.com/repos/hikalium/note/pulls \
        -d '{"title":"...","head":"<topic-branch>","base":"main","body":"..."}'
   ```
5. PR URL を返答に含める

## 注意

- token は `~/.netrc` 経由でのみ使う。コマンドラインに直書きしない
  (プロセスリストから leak する)
- destructive な commit (force push, history rewrite) は行わない
- 不確実な変更は PR description で明示し、merge 判断は人間に委ねる

セットアップ完了したら、続けて本来の task に取り組んでください。
````

---

## 2. Fine-grained PAT の作成 (初回のみ)

新しい環境 / Claude session を初めて使うときに 1 回だけ行う。

1. https://github.com/settings/personal-access-tokens を開く
2. **Generate new token** をクリック
3. パラメータ:

   | 項目 | 値 |
   |---|---|
   | Token name | `claude-note-<env>` (例: `claude-note-laptop`) |
   | Expiration | 7 days / 30 days (短いほど安全。期限切れたら再発行) |
   | Repository access | **Only select repositories** → `hikalium/note` |
   | Repository permissions | 下記 3 つ |

4. **Repository permissions** で許可:
   - **Contents**: Read and write — branch push に必要
   - **Pull requests**: Read and write — PR open に必要
   - **Metadata**: Read-only — 上 2 つを選ぶと自動付与

   それ以外 (Actions, Secrets, Issues, Packages, Webhooks, ...) は全て **No access**

5. **Generate token** → 表示された `github_pat_XXXX...` をコピー
   (1 回しか表示されないので即保存)

6. token を 1. のプロンプトの `<PAT>` 部分に貼り付ける

---

## 3. main ブランチ保護 (初回のみ、リポジトリ owner 側で実施)

PAT に Contents: Write を渡すと、保護がないと `main` への直接 push も技術的に可能になる。事故防止のため:

1. https://github.com/hikalium/note/settings/branches を開く
2. **Add branch protection rule**
3. 設定:

   | 項目 | 値 |
   |---|---|
   | Branch name pattern | `main` |
   | ☑ Require a pull request before merging | ON |
   | └ Required approvals | 0 (自分だけなら必須レビュー不要) |
   | ☐ Do not allow bypassing the above settings | OFF (自分は緊急時 bypass 可) / ON (完全防御) |
   | ☑ Block force pushes | ON |
   | ☑ Block deletions | ON |

4. Save changes

これで PAT 経由の `git push origin <commit>:main` は GH006 で拒否される。

```
remote: error: GH006: Protected branch update failed for refs/heads/main.
```

---

## 4. memory として使うときのファイル命名/編集ガイドライン

- 既存ファイルがトピックに合致するなら **追記** する (PR description で
  "extend existing file" と明示)
- 新規ファイル名は **小文字 + アンダースコア区切り** (`bash.md`,
  `cisco_ios.md`, `claude_code.md` 等) で揃える
- 1 ファイル = 1 トピック。横断的なメタ知見は本ファイル
  (`claude_session_prompt.md`) のように `claude_` prefix で切る
- 中身は terse な箇条書きと code block。長い解説より、後で grep して
  使えることを優先

## 5. 関連

- このリポジトリの index: [README.md](./README.md)
- Claude Code 権限設定 runbook: [claude_code.md](./claude_code.md)
- git, GitHub 慣習: [git.md](./git.md), [github.md](./github.md)
- GitHub fine-grained PAT 公式: https://docs.github.com/en/authentication/keeping-your-account-and-data-secure/managing-your-personal-access-tokens
- Branch protection 公式: https://docs.github.com/en/repositories/configuring-branches-and-merges-in-your-repository/managing-protected-branches/about-protected-branches
