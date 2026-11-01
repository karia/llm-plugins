---
name: codex-delegate
description: Herdrの別paneで動くCodex CLIに実装や調査を委譲するときに使用する。「codexにやらせて」「別paneのcodex --yoloに対応させて」「codexにオフロードして」「codexがリミットで止まったので再開させて」「不要なcodexを閉じて」のような依頼が対象。
---

# Codexへの委譲手順

自分は指示役に回り、実作業をCodex CLIに任せる。指示役の文脈を節約するため、Codexの画面全体は読まず、完了報告ファイルと差分の要約で結果を確かめる。

コマンドの構文は、インストール済みの`herdr agent`と`herdr pane`のヘルプを正とする。

## (1) 委譲先と作業場所を決める

1. `test "${HERDR_ENV:-}" = 1`でHerdr内にいることを確かめる。失敗したら、Herdr外である旨を依頼者に伝えて止まる。
2. `herdr agent list`で稼働中のCodexを確かめる。依頼者が用意したpaneがあれば、(3)の起動を省いてそのエージェントへ指示する。
3. 作業ディレクトリを決める。リポジトリを変更するタスクには、タスクごとのworktreeを`pr-flow`の手順で先に作る。並行するタスク同士でworktreeを共有しない。

## (2) タスクファイルを書く

リポジトリの外にある一時ディレクトリへタスクファイルを書く。長い指示を端末へ直接流し込まずに済み、再開時にも同じファイルを参照できる。

Codexは指示役の規約ファイルを読むとは限らないため、守らせる規約はタスクファイルに書く。次の項目をすべて埋める。

```markdown
# Task: <1行の目的>

## Read first
<読むべきissue、計画、ファイル>

## Scope
<やること>
<やらないこと>

## Rules
<コメント、commitの粒度と言語、テスト方針などの規約>
Work only in <作業ディレクトリ>.
Commit locally. Leave pushing, opening pull requests, and applying changes to shared environments to the orchestrator.

## Verify
<成功すべき検証コマンド>

## Report
When everything above is done, write <報告ファイルの絶対パス> with these sections:
Summary / Verification results / Open questions
Write this file last.
```

## (3) 起動して指示する

```bash
pane=$(herdr pane split --current --direction right --cwd <作業ディレクトリ> --no-focus | jq -r .result.pane.pane_id)
herdr agent start <名前> --kind codex --pane "$pane" --timeout 60000 -- --yolo
herdr agent prompt <名前> "Read <タスクファイル> and carry out the task it describes."
herdr agent wait <名前> --until working --timeout 20000
```

- 名前はタスクが分かるものにする（例：`issue52`）。
- 既存のpaneが細いときは、`--direction down`で分割する。
- `working`にならなければ、`herdr agent read <名前> --source recent-unwrapped --lines 40`で画面を確かめる。指示が届いている可能性があるため、確かめる前に再送しない。
- 画面に`Waiting for startup`と出たまま進まないときは、(4)へ進まず依頼者に報告する。Codexの信頼済みプロジェクトに作業ディレクトリが含まれているかを確かめる材料として、ディレクトリのパスを添える。

## (4) 完了を待つ

指示を送るたびに`herdr agent wait <名前> --until working --timeout 20000`で開始を確かめる。開始前の`idle`で次の待機が即座に終わるのを防ぐため。

開始を確かめたら、次のコマンドをバックグラウンドで実行し、終了の通知を待つ。通知までは別の作業へ進んでよい。

```bash
herdr agent wait <名前> --timeout 3600000; herdr agent get <名前> | jq -r .result.agent.agent_status
```

## (5) 停止の理由を判定する

待機の終了はエージェントの停止を示すだけで、作業の完了までは示さない。次の表を上から順に当てはめ、最初に一致した行に従う。

### 停止時の状況と対応

| 状況 | 判定 | 対応 |
| --- | --- | --- |
| 状態が`blocked` | 承認や質問の待ち | 画面を読み、依頼者に確認してから`herdr agent send-keys`で答える |
| 報告ファイルがある | 完了候補 | (6)で検証する |
| 報告ファイルがなく、画面に利用上限の表示がある | 利用上限 | (7)で再開する |
| 報告ファイルがなく、画面に質問がある | 判断待ち | 答えられる内容なら`herdr agent prompt`で答え、(4)へ戻る |
| 待機がタイムアウトした | 作業継続中の可能性 | `herdr agent get`で状態を確かめ、(4)へ戻る。2回続けてタイムアウトしたら依頼者に報告する |
| コマンドがプロトコル不一致で失敗する | Herdrのサーバーとクライアントの不整合 | 作業を止めて依頼者に報告する |

画面は`herdr agent read <名前> --source recent-unwrapped --lines 40`で末尾だけ読む。最下部のステータス行には利用枠の残量が出る。Codexの応答本文は画面の末尾に残らないことがあるため、完了の判断には使わない。

## (6) 結果を検証する

1. 報告ファイルを読む。`Open questions`があれば、自分で解決するか依頼者に確認する。
2. `git log --oneline <基点>..HEAD`と`git diff --stat <基点>..HEAD`で、変更がScopeに収まっていることを確かめる。外れていれば、手順4の形で同じエージェントに修正を指示する。
3. Verifyのコマンドを指示役が自分で実行する。報告の記載だけで成功と判断しない。
4. レビューが必要なら`third-party-review`の手順で実施する。指摘の修正は、報告ファイルを削除してから、文脈を持つ同じエージェントへ次の形で指示し、(4)へ戻る。削除するのは、古い報告を完了と誤認しないため。

   ```bash
   herdr agent prompt <名前> "Fix these findings, commit locally, and write the report file again when done: <指摘の一覧>"
   ```

5. pushとPR作成は、指示役が`pr-flow`の手順で行う。

## (7) 利用上限から再開する

1. 画面から解除時刻を読み、依頼者に伝える。読み取れなければ、依頼者に時刻を確認する。
2. 解除時刻の1分後を目安に待ち、次のコマンドをバックグラウンドで実行する。

   ```bash
   sleep <解除までの秒数>; herdr agent get <名前> | jq -r .result.agent.agent_status
   ```

3. 同じエージェントへ再開を指示し、(4)へ戻る。

   ```bash
   herdr agent prompt <名前> "Continue the task in <タスクファイル> from where you stopped."
   ```

上限で再び止まったら、この手順を繰り返す。依頼者が別の指示を出した場合は、それに従う。

## (8) paneを閉じる

対象のPRがマージされたとき、または依頼者に指示されたときに閉じる。それまでは、レビュー指摘の修正に使うため残す。

1. `git status --short`で、作業ディレクトリに未コミットの変更がないことを確かめる。残っていれば依頼者に確認する。
2. paneを閉じる。

   ```bash
   herdr pane close "$(herdr agent get <名前> | jq -r .result.agent.pane_id)"
   ```

worktreeの削除は`merged-branch-cleanup`の手順で行う。

## よくある取りこぼし

| 取りこぼし | 正しくは |
| --- | --- |
| `idle`を見て完了と報告する | 報告ファイルとVerifyの再実行で判断する |
| タイムアウトしたので同じ指示を再送する | 状態と画面を確かめてから次の操作を決める |
| 待機中に短い間隔でpaneを読み続ける | バックグラウンドの待機の通知を受けてから読む |
| 画面全体を読んで結果を把握する | 報告ファイルと差分の要約を読む |
| 利用上限で止まったまま依頼者の指示を待つ | 解除時刻を読み、自分で待機して再開する |
| 修正の指示を新しいエージェントに出す | 文脈を持つ同じエージェントに出す |
