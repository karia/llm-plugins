---
name: codex-delegate
description: Herdrの別paneで動くCodex CLIに実装や調査を委譲するときに使用する。「codexにやらせて」「別paneのcodex --yoloに対応させて」「codexにオフロードして」「codexがリミットで止まったので再開させて」「不要なcodexを閉じて」のような依頼が対象。
---

# Codexへの委譲手順

自分は指示役に回り、実作業をCodex CLIに任せる。指示役の文脈を節約するため、Codexの画面全体は読まず、完了報告ファイルと差分の要約で結果を確かめる。

コマンドの構文は、インストール済みの`herdr agent`と`herdr pane`のヘルプを正とする。

## (1) 委譲先と作業場所を決める

1. `test "${HERDR_ENV:-}" = 1`でHerdr内にいることを確かめる。失敗したら、Herdr外である旨を依頼者に伝えて止まる。
2. `herdr agent list`で稼働中のCodexを確かめる。依頼者が用意したエージェントを使うときは、(3)の起動を省き、次を先に確かめる。
   - 作業ディレクトリが合っていて、状態が`idle`か`done`であること。
   - エージェントの名前。なければ、以降の`<名前>`にpane IDを使う。
   - 起動オプション。確かめられなければ、`<commitの扱い>`を選ぶ前に依頼者に確認する。
3. 作業ディレクトリを決める。リポジトリを変更するタスクには、タスクごとのworktreeを`pr-flow`の手順で先に作る。並行するタスク同士でworktreeを共有しない。
4. 変更の基点として、作業ディレクトリで`git rev-parse HEAD`を実行し、結果を控える。
5. タスクごとの一時ディレクトリをリポジトリの外に作る。タスクファイルと報告ファイルをここに置く。

## (2) タスクファイルを書く

長い指示を端末へ直接流し込まずに済み、再開時にも同じファイルを参照できる。

Codexは指示役の規約ファイルを読むとは限らないため、守らせる規約はタスクファイルに書く。次の項目をすべて埋める。

```markdown
# Task: <1行の目的>

## Read first
<読むべきissue、計画、ファイル>

## Scope
<やること>
<やらないこと>

## Rules
<コメント、テスト方針などの規約>
<commitの扱い>
Work only in <作業ディレクトリ>, except for the report file below.
Leave pushing, opening pull requests, and applying changes to shared environments to the orchestrator.

## Verify
<成功すべき検証コマンド>

## Report
When everything above is done, write <報告ファイルの絶対パス> with these sections:
Summary / Verification results / Open questions
Write this file last.
```

調査だけのタスクでは、`<commitの扱い>`を「Change no files except the report file.」に置き換え、Verifyを省く。

## (3) 起動して指示する

起動オプションとcommitの担当は、依頼に応じて次の表から選ぶ。

### 依頼ごとの起動オプション

| 依頼 | 起動オプション | `<commitの扱い>` | commitの担当 |
| --- | --- | --- | --- |
| `--yolo`の指定なし | `--sandbox workspace-write --add-dir <一時ディレクトリ>` | `Leave changes uncommitted.` | 指示役 |
| 依頼者が`--yolo`を指定 | `--yolo` | `Commit locally in small steps.` | Codex |

`--yolo`は承認とサンドボックスをともに外すため、依頼者が明示したときだけ使う。`workspace-write`ではworktreeのgitメタデータが書き込み範囲の外にあり、commitのたびに承認を求められるので、commitを指示役が受け持つ。

```bash
pane=$(herdr pane split --current --direction right --cwd <作業ディレクトリ> --no-focus | jq -r .result.pane.pane_id) &&
herdr agent start <名前> --kind codex --pane "$pane" --timeout 60000 -- <起動オプション>
```

`agent_status`が`idle`で返ったら指示を送る。

```bash
herdr agent prompt <名前> "Read <タスクファイル> and carry out the task it describes." --wait --until working --until blocked --timeout 30000
```

- 名前はタスクが分かるものにする（例：`issue52`）。
- 既存のpaneが細いときは、`--direction down`で分割する。

### 起動時の問題と対応

| 状況 | 対応 |
| --- | --- |
| `agent start`が`agent_not_ready`を返す | 画面を読む。フォルダの信頼確認なら、答えると設定に保存されるため依頼者に確認する。解消して状態が`idle`になってから指示を送る |
| 画面に`Waiting for startup`と出たまま進まない | (4)へ進まず、作業ディレクトリのパスを添えて依頼者に報告する |
| `agent_prompt_stalled`またはタイムアウトで返る | 画面を読む。指示が届いている可能性があるため、確かめる前に再送しない |

## (4) 完了を待つ

Codexへ指示を送るときは毎回、(3)の形で`--wait --until working --until blocked --timeout 30000`を付けて開始を確かめる。報告ファイルがすでにあれば、送る前に削除する。古い報告を完了と誤認しないため。

開始を確かめたら、次のコマンドをバックグラウンドで実行し、終了の通知を待つ。通知までは別の作業へ進んでよい。バックグラウンドの通知がない環境では、フォアグラウンドで実行する。

```bash
herdr agent wait <名前> --timeout 3600000; herdr agent get <名前> | jq -r .result.agent.agent_status
```

## (5) 停止の理由を判定する

待機の終了はエージェントの停止を示すだけで、作業の完了までは示さない。次の表を上から順に当てはめ、最初に一致した行に従う。

### 停止時の状況と対応

| 状況 | 判定 | 対応 |
| --- | --- | --- |
| コマンドがプロトコル不一致で失敗する | Herdrのサーバーとクライアントの不整合 | 作業を止めて依頼者に報告する |
| `herdr agent get`が`agent_not_found`を返す | Codexの終了 | `herdr pane read <pane ID> --source recent-unwrapped --lines 40`で原因を読み、依頼者に報告する |
| 状態が`blocked` | 承認や質問の待ち | 画面を読み、依頼者に確認してから`herdr agent send-keys`で答え、(4)の待機へ戻る |
| 待機がタイムアウトした | 作業継続中の可能性 | (4)の待機へ戻る。2回続けてタイムアウトしたら依頼者に報告する |
| 報告ファイルがある | 完了候補 | (6)で検証する |
| 画面に利用上限の表示がある | 利用上限 | (7)で再開する |
| 画面に質問がある | 判断待ち | 答えられる内容なら`herdr agent prompt`で答え、(4)へ戻る |
| 上のどれにも一致しない | 不明な停止 | 下の手順で切り分ける |

画面は`herdr agent read <名前> --source recent-unwrapped --lines 40`で末尾だけ読む。最下部のステータス行には利用枠の残量が出る。Codexの応答本文は画面の末尾に残らないことがあるため、完了の判断には使わない。

不明な停止は次の順で切り分ける。

1. パネルが開いていれば、`herdr agent send-keys <名前> esc`で閉じる。Hooksのレビューなどのパネルが開いていると、指示が実行されないまま消え、状態が`done`になることがある。パネル上のキーは信頼の設定を変えるものがあるため、`esc`以外を送らない。
2. ステータス行のコンテキスト使用量が`0%`なら指示は実行されていないため、一度だけ同じ指示を送り、(4)へ戻る。
3. それ以外は、`--lines 200`で読み直した内容を添えて依頼者に報告する。

## (6) 結果を検証する

作業ディレクトリで実行する。調査だけのタスクは手順1だけでよい。

1. 報告ファイルを読む。`Open questions`があれば、自分で解決するか依頼者に確認する。
2. `git status --short`と`git diff --stat <基点>`で、変更がScopeに収まっていることを確かめる。Codexがcommitを担当するときは、未コミットの変更が残っていないことも確かめる。
3. Verifyのコマンドを指示役が自分で実行する。報告の記載だけで成功と判断しない。
4. 手順2か3で問題が見つかったら、同じエージェントへ修正を指示し、(4)へ戻る。文脈を持つエージェントのほうが、新しいエージェントより説明が少なく済むため。

   ```bash
   herdr agent prompt <名前> "Fix these problems and write the report file again when done: <問題の一覧>" --wait --until working --until blocked --timeout 30000
   ```

5. レビューが必要なら`third-party-review`の手順で実施する。指摘の修正は手順4の形で指示する。
6. 指示役がcommitを担当するときはここでcommitする。pushとPR作成は`pr-flow`の手順で行う。

## (7) 利用上限から再開する

1. 画面から解除時刻を読み、依頼者に伝える。読み取れなければ、依頼者に時刻を確認する。
2. 解除時刻の1分後までの秒数を計算し、`sleep <秒数>`をバックグラウンドで実行して通知を待つ。
3. 同じエージェントへ再開を指示し、(4)へ戻る。

   ```bash
   herdr agent prompt <名前> "Continue the task in <タスクファイル> from where you stopped." --wait --until working --until blocked --timeout 30000
   ```

上限で再び止まったら、この手順を繰り返す。依頼者が別の指示を出した場合は、それに従う。

## (8) paneを閉じる

実装のタスクは、対象のPRがマージされたとき、または依頼者に指示されたときに閉じる。それまでは、レビュー指摘の修正に使うため残す。調査だけのタスクは、報告ファイルを確かめた後に閉じる。

1. `git status --short`で、作業ディレクトリに未コミットの変更がないことを確かめる。残っていれば依頼者に確認する。
2. `herdr pane close <pane ID>`でpaneを閉じる。pane IDは起動時の値を使う。エージェントが稼働中なら、`herdr agent get <名前> | jq -r .result.agent.pane_id`でも取得できる。

worktreeの削除は`merged-branch-cleanup`の手順で行う。

## よくある取りこぼし

| 取りこぼし | 正しくは |
| --- | --- |
| `idle`や`done`を見て完了と報告する | 報告ファイルとVerifyの再実行で判断する |
| タイムアウトしたので同じ指示を再送する | 状態と画面を確かめてから次の操作を決める |
| 待機中に短い間隔でpaneを読み続ける | バックグラウンドの待機の通知を受けてから読む |
| 画面全体を読んで結果を把握する | 報告ファイルと差分の要約を読む |
| 利用上限で止まったまま依頼者の指示を待つ | 解除時刻を読み、自分で待機して再開する |
| 修正の指示を新しいエージェントに出す | 文脈を持つ同じエージェントに出す |
| 承認ダイアログや信頼確認に自分の判断で答える | 依頼者に確認してから答える |
