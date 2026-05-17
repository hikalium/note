# Claude Code 権限設定 runbook

Claude Code の `~/.claude/settings.json` (および project-local の `.claude/settings.json` / `.claude/settings.local.json`) で permission ルールを書くときの判断基準と落とし穴。次の session の自分や LLM 自身が読み解いて追記できるよう構造化したノート。

## TL;DR (新しい権限を足したい時の決定木)

**「ファイルを Read/Edit/Write したい」**

- 対象が cwd 配下 → 既定で許可。追加不要。
- 対象が cwd 外 → まず `permissions.additionalDirectories` に親ディレクトリを足す。これを忘れると `Read/Edit/Write(/path/**)` の allow ルールを書いても **無視される** (workspace 境界が allow より手前のレイヤ)。
- その上で `Read(/path/**)`, `Edit(/path/**)`, `Write(/path/**)` を allow に追加。

**「特定のコマンドを毎回承認なしで叩きたい」**

- 単一引数なし: `Bash(cmd)` (例: `Bash(pwd)`)
- 任意の引数: `Bash(cmd:*)` (例: `Bash(git:*)`)
- 限定的な subcommand: `Bash(cmd subcmd:*)` または `Bash(cmd:subcmd*)` (両方サポート)
- 絶対パスの実行ファイル: `Bash(/absolute/path/**)` (例: `Bash(/work/**)`)

**「やったら拒否されたが、ルールは正しく書いたつもり」**

- 90% は `additionalDirectories` 不足 (workspace 外)
- 残り 10% は **`&&`/`;`/`|` で複合コマンドにしている** (allowlist は 1 つのコマンドにマッチするので、複合式全体に対しては効かない)

## 設定ファイルの場所と優先順位

| ファイル | scope | gitignore | 用途 |
|---|---|---|---|
| `~/.claude/settings.json` | user 全体 | n/a | クロスプロジェクトの恒久ルール |
| `~/.claude/settings.local.json` | user 全体 | n/a | 機密 / 一時 |
| `<project>/.claude/settings.json` | プロジェクト共有 | tracked | チーム共有ルール |
| `<project>/.claude/settings.local.json` | プロジェクト個人 | gitignored | 個人の一時ルール |

優先順位: user → project → projectLocal → CLI flag → policy (組織管理) の順で **後勝ち**。

## additionalDirectories (workspace 境界)

```jsonc
{
  "permissions": {
    "additionalDirectories": ["/work"],
    "allow": [
      "Read(/work/**)",
      "Edit(/work/**)",
      "Write(/work/**)",
      "Bash(/work/**)"
    ]
  }
}
```

`additionalDirectories` 無し + allow だけ → workspace 境界で block されて手動承認待ち。`additionalDirectories` あり + allow 無し → 操作は可能だが毎回プロンプト。**両方セットで使う**。

cwd が `/work/foo` だが作業は `/work/bar`, `/work/baz` でもしたい、という典型ケースで `additionalDirectories: ["/work"]` が効く。

## glob 構文

Claude Code は Node.js の `path.matchesGlob` を使う。

| パターン | 意味 |
|---|---|
| `*` | 任意の文字列 (path separator 含まない) |
| `**` | 任意の path セグメント (再帰) |
| `?` | 1 文字 |
| `[abc]` | 文字クラス |

検証手順:

```bash
node -e 'console.log(require("path").matchesGlob("/work/qemu/hw/misc/foo.c", "/work/**"))'
# → true
```

## Bash パターンの細かい癖

1. **複合コマンドは不可**: `cmd1 && cmd2` / `cmd1 | cmd2` / `cmd1; cmd2` は単一コマンドに対する permission の対象にならず、複合式全体に対して別途承認待ちになる。
   - 対処: コマンドを 1 件ずつ別 Bash ツール呼び出しに分割する。並列ツール呼び出しなら効率は保てる。
   - どうしても 1 シェルプロセスにまとめたい場合: スクリプトを `Write` で `*.sh` として生成し、`./run.sh` のように単独コマンドとして実行 (これは allow にマッチする)。

2. **リダイレクト**: `> file` / `>> file` / `< file` / `<< EOF` も複合式扱いで allow を貫通させない疑い。回避策は同上 (スクリプト化、または `Write` ツール直接利用)。

3. **prefix マッチング**: `Bash(npm run:*)` は `npm run` で始まる任意のコマンドにマッチ。`Bash(npm:*)` は `npm` のあらゆるサブコマンド。

4. **絶対パス実行**: `Bash(/path/**)` は絶対パスの実行ファイル呼び出しにマッチ。`/work/qemu/build/qemu-system-x86_64 ...` や `/work/scripts/boot-test.sh` などフルパスで叩くケースが楽になる。

## 入れるべきでないルール (anti-patterns)

破壊的・予期しない副作用を持つコマンドは blanket allow を **避ける**。承認プロンプトで人間が場面ごとに判断する方が安全:

- `Bash(mv:*)` — 任意の場所に移動可能、誤コマンドで消失リスク
- `Bash(cp:*)` — 同上 (上書きで破壊)
- `Bash(rm:*)` — 言うまでもなく
- `Bash(chmod 777:*)` — 権限緩和系
- `Bash(sudo:*)` — root 昇格全般
- `Bash(git push:*)` — public visibility / irreversibility
- `Bash(git reset --hard:*)` / `Bash(git push --force:*)` — 履歴破壊

これらは個別シナリオで承認するか、`Bash(mv:/work/specific/**)` のように **絞った形** で限定的に入れる。

## デバッグ手順 (権限が効いてない / 拒否されたら)

1. 拒否されたコマンドそのものを書き出す (例: `git -C /work/foo log ...`)
2. `&&`/`;`/`|` が含まれていないか確認
3. 含まれていたら → 分割 + 単独実行で再試行
4. 含まれていなければ → `~/.claude/settings.json` の `allow` を grep して該当 prefix があるか確認
5. 該当 prefix がある場合 → 対象 path が cwd 外なら `additionalDirectories` を確認
6. それでも拒否されたら → glob 構文を node の `matchesGlob` で検証
7. それでも拒否なら → 別の settings file (project local 等) が override している可能性、`find ~/.claude $(pwd) -name "settings*.json"` で見つける

## 機密情報の扱い

PAT / API key などを `Write` でファイル保存する場合は注意。Claude のツール呼び出しは ログに残るため、token 自体が transcript で永続する。

作法:

- 受領 → 即 `Write` で chmod 600 ファイルに保存 → 以降 token 値を assistant 応答に echo しない
- `git` / `curl` から使うときは `~/.netrc` 経由 (`machine github.com login x-access-token password <PAT>`)。コマンドラインに直書きしないことでプロセスリスト経由の leak を防ぐ
- 不要になったら provider 側で revoke + ローカルファイル削除

例:

```
machine github.com
  login x-access-token
  password ghp_xxxxxxxxxxxxxxxxxxxx
machine api.github.com
  login x-access-token
  password ghp_xxxxxxxxxxxxxxxxxxxx
```

## サンプル: 一通り盛り込んだ user settings.json

```jsonc
{
  "theme": "dark",
  "permissions": {
    "additionalDirectories": ["/work"],
    "allow": [
      // tracked dev tools (no destructive args by default)
      "Bash(git:*)",
      "Bash(make:*)",
      "Bash(ninja:*)",
      "Bash(meson:*)",
      "Bash(gcc:*)", "Bash(g++:*)", "Bash(clang:*)",
      "Bash(ld:*)", "Bash(as:*)", "Bash(ar:*)",
      "Bash(nm:*)", "Bash(objdump:*)", "Bash(readelf:*)",
      "Bash(python:*)", "Bash(python3:*)",
      "Bash(pip:install*)", "Bash(pip3:install*)",

      // package management (install-only, no remove/purge)
      "Bash(apt-get install:*)",
      "Bash(apt-get update:*)",
      "Bash(apt-get update)",
      "Bash(dpkg:*)",            // read-only by default
      "Bash(apt-cache:*)",

      // safe read-only utilities
      "Bash(ls:*)", "Bash(cat:*)", "Bash(head:*)", "Bash(tail:*)",
      "Bash(grep:*)", "Bash(find:*)", "Bash(awk:*)", "Bash(sed:*)",
      "Bash(diff:*)", "Bash(cmp:*)", "Bash(file:*)", "Bash(stat:*)",
      "Bash(wc:*)", "Bash(sort:*)", "Bash(uniq:*)",
      "Bash(which:*)", "Bash(whereis:*)",
      "Bash(du:*)", "Bash(df:*)",
      "Bash(pwd)",

      // creation but not destruction
      "Bash(mkdir:*)",
      "Bash(install:*)",     // /usr/bin/install — for placing files with mode
      "Bash(ln:*)",
      "Bash(chmod:*)",
      "Bash(tar:*)", "Bash(unzip:*)", "Bash(gunzip:*)", "Bash(zcat:*)",
      "Bash(xargs:*)",

      // path-scoped binaries / scripts
      "Bash(/work/**)",
      "Bash(./scripts/*)",
      "Bash(./test/*)",
      "Bash(./run.sh)",
      "Bash(./build.sh)",

      // R/E/W on the broader workspace
      "Read(/work/**)",
      "Edit(/work/**)",
      "Write(/work/**)"
    ]
  }
}
```

`mv`, `cp`, `rm`, `sudo`, `git push`, `git reset --hard` を **意図的に** 入れていない点に注意。

## 関連

- https://docs.claude.com/en/docs/claude-code (公式)
- Node.js `path.matchesGlob`: https://nodejs.org/api/path.html#pathmatchesglobpath-pattern
