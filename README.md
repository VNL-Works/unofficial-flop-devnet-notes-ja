# FLOP Devnet を CLI で試した記録（非公式・日本語）

> - **非公式のメモ。** FLOP Labs の公式ではない。VNL Works の非公式のメモで、筆者＝VNL Works（GitHub は vanilla3）。
> - **最終確認 2026-11-26 06:10Z**（flop-labs/network の Issue を取得した時刻）。直った・直っていないなどの状態は、この時点のもの。
> - 最終確認から 7 日以上たっていたら、状態の行は古いとみなし、公式 docs と flop-labs/network を見てほしい。
> - **このメモは CLI 0.4.0 で試した記録。今の Devnet には 0.4.0 はつながらない、という参加者の報告がある**（#132。0.4.0 は 11/07 に spec 1014014 で接続を拒み、Devnet の runtime が 11/12 に spec 1014020 に上がった後は 0.4.4 も拒み、11/13〜11/15 の時点では 0.5.0 で動いた、という報告）。その後、runtime は 11/18 ごろ spec 1014027 に上がり、0.5.0 も拒むようになった、という報告がある（#355）。くわしくは下の「環境」。
> - 公式かどうかの確かめ先：https://docs.flop.network/docs/official-links
> - 正本は公式 docs（docs.flop.network、英語）。このメモは遅れることがある（network#2 のコメントの指摘）。
> - docs の testnet-and-timeline（Last updated 2026-10-30、写しの取得は 2026-11-04）の FAQ は、（11/04 の写しでは）「agent は Devnet を使えない」と書いていた。同じページの別の箇所や CLI は使える前提で書かれていて、食い違っていた（network#52、筆者が出した Issue）。保守担当者は 2026-11-20 に、公開している FAQ で直したと書いて #52 を閉じた。筆者が 2026-11-24 に読んだ開始前の FAQ（Last updated 2026-11-20、写しの取得は 2026-11-24）は、「Public Devnet agent access is separate from operator onboarding.」と書いていた。同じ文は、2026-11-24 の遅くには https://docs.flop.network/next/docs/testnet-and-timeline にあり、もとの URL は 2026-11-25 に読むと「Testnet opening FAQ」という別の内容になっていた。
> - Testnet：参加者が Testnet の日付の食い違いを報告した #187 は、2026-11-20 に閉じられた。保守担当者は該当のページを直したと書いている。筆者が 2026-11-25 に読んだ公式のページでは、networks/testnet（Last updated 2026-11-24、写しの取得は 2026-11-25 06:40Z）が「FLOP Testnet went live on 24 November 2026 with six founding validators.」「This opening is unscored. Test FLOP has no monetary value, and the network may restart or reset during testing.」「Voucher earning starts at the publicly announced finalized block of a future upgrade. Earlier activity earns no vouchers and receives no backfill.」と書いている。Testnet opening FAQ（https://docs.flop.network/docs/testnet-and-timeline 、Last updated 2026-11-24、写しの取得は 2026-11-25 07:02Z）は「There is no announced block or date yet. Earlier activity earns zero, with no backfill.」と書いている。X の公式アカウント（@flop_labs）には「FLOP Testnet is live! This is a very early release.」という投稿がある（筆者が 2026-11-25 06:42Z ごろに読んだ）。このメモは Devnet の記録で、Testnet の手順・時期・配分は扱わない。公式 docs を見てほしい。
> - Devnet は報酬も airdrop の重みもなく、使い捨てで何も引き継がない（official-links、Last updated 2026-10-23、写しの取得は 2026-11-04。開始前の FAQ（testnet-and-timeline）、Last updated 2026-11-20、写しの取得は 2026-11-24）。
> - 同じ開始前の FAQ（https://docs.flop.network/next/docs/testnet-and-timeline 、Last updated 2026-11-20、写しの取得は 2026-11-24）には「Creating DIDs earns nothing; posting your DID in a TechnoCore room earns nothing.」とある。詳しくは公式 FAQ を。
> - FLOP は claim・mint・登録のためにシードフレーズや入金を求めない（official-links（Last updated 2026-10-23、写しの取得は 2026-11-04）の「FLOP never asks for a seed phrase or a deposit to claim, mint or register anything.」）。
> - 関係の表示：筆者は FLOP の KOL プログラムに応募している。報酬は受けていない（2026-11-16 時点）。
> - AI の関与：AI（Claude）が下書きし、別の文脈の AI がレビューした。
> - ライセンス：筆者の文は CC BY 4.0（`LICENSE`）。引用した英語のエラー文や docs の文には及ばない。
> - つまずきの早見表は [TROUBLE.md](TROUBLE.md)。

## このメモの範囲

- `@flop-labs/cli` 0.4.0 で、FLOP Devnet を builder（agent）として試した記録。validator・miner は扱わない。
- 時刻はすべて UTC（末尾の `Z`）。パスは `~/.flop/…` の形で書く。
- アドレス・DID・tx hash・genesis の全桁、鍵やキーの中身は載せない。
- 試した段と、試していない段を分けて書く。試していない段は手順の形にしない。

| 段 | 中身 | このメモでの扱い |
|---|---|---|
| 1 | 端末の時計 | 試した（2026-11-05 の観測） |
| 2〜3 | CLI の版・help・Devnet への接続 | 試した（2026-11-04） |
| 案内 | `agent init`・`agent doctor` | 試した（2026-11-04） |
| 7 | `agent credential`（gateway の API キー） | 試した（6 回とも失敗、2026-11-04〜11-05） |
| 4・5 | 鍵・faucet | `--help` を見ただけ |
| 6 | 残高 | 試した（止めた、#121） |
| 8 | 推論 | 試していない |

## 試した

### 環境

- Windows。Node.js v24.22.1（`node --version`、2026-11-04 に確認）。sdk-references（Last updated 2026-10-30、写しの取得は 2026-11-04）は Node.js 22 以上を求めている。
- CLI はグローバルに入れず、版を固定して `npx` で動かした：`npx --yes --package @flop-labs/cli@0.4.0 flop …`。下の `… flop` はこの形の略。
- 2026-11-06 に npm の registry を読み、`@flop-labs/cli` の latest は 0.4.0 のまま（0.4.0 は 2026-10-29 公開）だった。筆者が 2026-11-26 06:16Z に npm の registry を読むと、latest は 0.9.1（2026-11-26 公開）で、0.9.0 は 2026-11-25 公開だった。
- その後、0.4.0 は今の Devnet につながらない、という参加者の報告がある（#132）。
  - Devnet の runtime が spec 1014014 に上がった後（#132 は 11/07 に出された）、`@flop-labs/cli` 0.4.0 は `Unsupported runtime spec version` で接続を拒む。
  - 0.4.1（11/08 公開と報告）は 1014014 に対応した（#132 の 11/09 のコメント）。
  - その後、Devnet は 11/12 に spec 1014020 に上がった（#132 の 11/13 のコメント）。0.4.4 も拒み、0.5.0 では `balance`・`health`・`model list`・`network status` が動いた、という報告がある（#132 の 11/13・11/15 のコメント。11/13〜11/15 の時点の話）。
  - 11/18 ごろ、Devnet の runtime は spec 1014027 に上がり、`@flop-labs/cli` 0.5.0 も `Unsupported runtime spec version` で拒むようになった、という報告がある（#355、2026-11-18T17:16:39Z。runtime が上がったのは 16:12Z ごろと書いている）。
  - 保守担当者は 2026-11-20 に #355 を閉じ、「SDK `0.7.0` and CLI `0.6.0` admit Devnet specs `1014027` and `1014029`」と書いた。
  - その後も runtime は上がった、という参加者の報告がある：1014029（11/19、#355 の 2026-11-19T15:16:49Z のコメント）、1014031（11/20、#410 の 2026-11-20T18:23:46Z のコメント）、1014034（11/23、#567 の 2026-11-23T15:10:01Z のコメント）。11/24 の時点でも 1014034 という報告がある（#1042 の本文、#649 の 2026-11-24T23:05:19Z のコメント）。11/25 の時点でも Devnet は 1014034 という報告がある（#1042 の 2026-11-25T13:26:11Z のコメント）。その後に上がったかどうかは、この取得では確かめられない。
- network の CLI のリリースノート（https://github.com/flop-labs/network/tree/main/releases/cli ）は、版ごとに受ける runtime を書いている。
  - `releases/cli/v0.5.0.md`（11/12 の commit）：「every `flop` command connects to runtimes `1014015` through `1014020`」。
  - v0.6.0（11/19 の commit）：「Depends on `@flop-labs/sdk` `^0.7.0`, which admits runtime spec 1,014,029」「Published `0.5.0` resolves SDK `0.6.x`, which stops at `1014020`.」
  - v0.7.1（11/20 の commit）：「Requires `@flop-labs/sdk` 0.7.2, which admits runtime spec 1,014,031」「SDK 0.7.1 and earlier stop at `1014029`.」
  - v0.8.0（11/23 の commit）：「Requires `@flop-labs/sdk` 0.9.0, which admits devnet runtime `1014034`.」
  - v0.8.1〜v0.8.3（v0.8.3 は 11/25 の commit）に、Devnet の対応 spec の新しい文は無い。
  - 2026-11-26 06:10Z の時点でいちばん新しいノートは v0.9.1（11/26 の commit）で、v0.9.0 のノートは無い（`releases/cli/` の一覧では v0.8.3 の次が v0.9.1）。v0.9.1：「Takes `@flop-labs/sdk` 0.12.1, which admits runtime `1014039`」「0.9.0 refuses that spec with `RUNTIME_INCOMPATIBLE`.」。このノートは、runtime が 1014039 に上がった後も chain のコマンドが動くようにする、という趣旨で書かれている。Devnet の runtime がもう 1014039 に上がったかどうかは、ノートからは分からない。
- 筆者が試したのは 0.4.0（11/04〜11/06）だけ。0.4.1 以降の版は試していない。
- このメモの `npx --yes --package @flop-labs/cli@0.4.0 …` の形のまま今流すと、chain につなぐコマンド（報告では `health`・`balance`）は接続の段で止まると読める（推論、#132 の報告から）。
- `agent credential` は 0.4.0 でも gateway の登録の段まで届いた報告がある（#200 の 11/10）。CLI 0.4.0 で登録の段まで届いた報告は 11/15 にもある（#202 の 2026-11-15T02:50:47Z のコメント。コマンド名は書かれていない）。
- 保守担当者は、#4・#5・#12 を閉じたコメント（11/11。runtime が 1014020 に上がった 11/12 より前）で、0.4.0 のままのものは上げるよう書き、docs の古い版の案内は #132 で追う、と書いた。
- 版は公式の docs と network のリリースノートで確かめる。docs の版の表示が遅れているという報告もある（#132 の 11/13・11/15 のコメントと、2026-11-17T13:05:24Z のコメント）。保守担当者も 2026-11-20 に #132 へ、docs の版の案内をそろえ、足りないリリースノートを出すために open のままにする、と書いた。

### 段 2〜3：版・help・Devnet への接続（2026-11-04）

- `… flop --version`：`0.4.0`。npm の warn が 2 行（`node-domexception@1.0.0`・`@substrate/connect@0.8.11` が deprecated）。
- `… flop --help`：コマンドは 24 個（`keys`・`wallet`・`faucet`・`agent` ほか）。`--network` の既定は `local`。なので `--network devnet` を毎回付ける。付け忘れると手元の RPC を待ち続けるという報告がある（network#14、参加者の報告。筆者は付け忘れて試してはいない。TROUBLE.md の CLI の表）。#14 は 2026-11-20 に、保守担当者が CLI 0.6.0（SDK 0.7.0）で直ったと書いて閉じた。直ったとされた版が今の Devnet につながるかは、TROUBLE.md の CLI の表の `Unsupported runtime spec version` の行を見てほしい。
- `… flop --network devnet network status`：profile `live`、rpc reachable・chain `FLOP Devnet`・genesis `0x99c172c0…d631`・spec `tzkv-runtime` 1014003、`matches_expected: null`、faucet reachable・`matches_rpc: true`。

### 案内：`agent init` と `agent doctor`（2026-11-04）

- `… flop agent init`：help の説明は「Print a safe agent onboarding plan and MCP configuration」。表示だけのコマンド（help の説明による）。remote の案内として、MCP の設定、`agent doctor`、任意の鍵の作成と agent wallet の作成（どちらも「changes state」と表示される）、`agent launch` などを順に示した。鍵やアドレスは出なかった。案内には `agent credential` と faucet が出てこず、`--network` も付いていなかった。
- `… flop --network devnet agent doctor --mode remote`：`READY`。見たのは `✔ binary:node: available` と `✔ mcp: https://mcp.flop.network/mcp → HTTP 405` の 2 項目だけ。既定では gateway と RPC を見ない。
- `… flop --network devnet agent doctor --mode remote --gateway https://api.flop.network/v1 --check-rpc`：`READY`。node ✔、mcp → HTTP 405 ✔、gateway `https://api.flop.network/v1` → HTTP 200 ✔、rpc `wss://rpc.devnet.flop.network` → FLOP Devnet（11 peers）✔ の 4 項目。gateway と RPC に届くかを見たいなら、この 2 つのオプションを付ける。

### 段 7：`agent credential --json` の 6 回

コマンドは毎回 `npx --yes --package @flop-labs/cli@0.4.0 flop --network devnet agent credential --json`。同じ鍵ファイル（`~/.flop/devnet-enroll-wallet.json`、1 回目に CLI が作った）のまま流した。

| 回 | 時刻（UTC） | 結果（出力の原文） | 対応する Issue |
|---|---|---|---|
| 1 | 2026-11-04T06:56Z ごろ | `{"ok": false, "code": "COMMAND_FAILED", "message": "The operation was aborted due to timeout", "retryable": false}` | network#22 に、同じタイムアウトでも付与は後で届いた報告がある |
| 2 | 2026-11-04T07:12Z ごろ（約 15 分待って 1 回） | `{"ok": false, "code": "COMMAND_FAILED", "message": "devnet faucet returned HTTP 503: Grant pending; retry the original request later", "retryable": false}` | network#43 |
| 3 | 2026-11-04T07:45Z ごろ | 2 回目と同じ（`COMMAND_FAILED`・Grant pending・`retryable: false`） | network#43 |
| 4 | 2026-11-05T01:14Z ごろ（3 回目から約 17.5 時間後） | `{"ok": false, "code": "INVALID_INPUT", "message": "devnet gateway returned an invalid enrollment challenge", "retryable": false}` | network#34 |
| 5 | 2026-11-05T01:16Z ごろ（`w32tm /query /status` の直後） | 4 回目と同じ（`INVALID_INPUT`） | network#34 |
| 6 | 2026-11-05T01:20:16Z の後（`w32tm /resync /force` の後） | 4 回目と同じ（`INVALID_INPUT`） | network#34 |

- 1 回目のタイムアウトは、どの段で止まったかが出力から分からない。同じタイムアウトでも `faucet request` では `retryable: true`（参加者の報告）、`agent credential` では `retryable: false` と出る。
- 2・3 回目：faucet は `retryAfterSeconds: 60` を返す（#43 の報告）のに、CLI は `retryable: false` と出した。
- 4 回目で Grant pending から gateway の登録の段に進んだ。CLI 0.4.0 の公開ソース（npm の `@flop-labs/cli@0.4.0` の `dist/cli.js`。筆者は読んだだけで、ソースのコードは動かしていない）では、4〜6 回目のエラーの文 `devnet gateway returned an invalid enrollment challenge` は L6572 でだけ作られ、L6612＝残高の確かめの後でだけ使われる。`agent credential` が登録に使う `walletEnrollment` の呼び出しは L6684 の 1 か所で、残高の下限は `DEFAULT_MIN_BALANCE`＝1 FLOP（L6445）。
- この文は残高の確かめ（1 FLOP 以上）を通った後でしか出ないので、その時点で enroll wallet に 1 FLOP 以上あった、とソースから読める（残高そのものは見ていない）。CLI が請求する先は faucet だけなので、faucet の付与が届いたと読むのが自然（推論）。
- `INVALID_INPUT` という code は gateway の返事ではなく、CLI がエラーの文の語（`invalid` など）から付けるもの（L5040）。なので code は根拠にしない。
- 2 回目以降は、間を空けて 1 回ずつ流した。wallet・アドレスは作り直していない。

### 段 1：時計（2026-11-05 の観測）

4〜6 回目のエラーの文 `devnet gateway returned an invalid enrollment challenge` は、network#34 と同じ。#34 は、手元の時計が gateway より遅れていると CLI が challenge を自分で拒む、と報告している。そこで時計を見た。原因が同じというのは推論で（gateway の時計が NTP に合っている前提）、下の「読み方」に分けて書く。

- `w32tm /query /status`（5 回目の直前）：最終正常同期 01:12:09Z、ソース `time.windows.com,0x9`、ポーリング間隔 1024 s。Windows Time は動いて同期していた。設定は変えていない（見てもいない）。
- `w32tm /stripchart /computer:time.windows.com /samples:5 /dataonly`（01:18:12〜01:18:20Z）：5 回とも `+00.185〜+00.190s`（`+` は手元が遅い向き）。同じく `ntp.nict.jp` は 5 回とも応答なし（`0x800705B4`）。
- 管理者の PowerShell で `w32tm /resync /force`：コマンドは正しく完了した。
- 直後の stripchart（01:20:08〜01:20:16Z）：5 回とも `+00.1906〜+00.1963s`。約 0.19 秒の遅れは縮まらなかった。
- そのあと 6 回目を流したが、同じエラーの文だった。
- その後も、CLI 0.4.2（#34 の 2026-11-11T03:04Z のコメント、観測は 11/10 JST と書かれている）と CLI 0.5.0（#241、2026-11-12T07:29Z、時計は NTP に合わせた後と書いている）で同じ文になった、という参加者の報告がある。11/17〜11/25 にも、CLI 0.4.4・0.7.0・0.7.2・0.8.0・0.8.1・0.8.2・0.8.3 で同じ文になった、という報告がある（#34 のコメント、#651・#947・#1065・#1142・#1337・#1349）。#1337（CLI 0.8.3）と #1349（CLI 0.8.2）は、gateway が challenge に書く時刻（`Issued At`）が手元の時刻より先になっている、という測定を書いている（#1337 は約 1〜2 秒、#1349 は約 0.5〜1.5 秒。#1349 は、手元の時計は同期済みと書いている）。0.8.2 で `agent credential` が通ったと読める報告も 1 件ある（#649）。0.8.3 でも同じ文の報告がある（#1337）。保守担当者がこの確かめ（#34）を直したという返事はなく、リリースノート（v0.9.1 まで。v0.9.0 のノートは無い）にも直したという記述はない（2026-11-26 06:10Z の時点）。

読み方（どれも推論）：

- 約 0.19 秒の遅れは、#34 の報告の例（147 ms）と同じ大きさで、このエラーの文の説明として合う（推論。gateway の時計が NTP に合っている前提。参加者の 1 人は gateway の時計を測り、NTP と食い違うとは言えないと #34 の 11/05 05:22Z のコメントに書いている）。
- CLI 0.4.0 のソースでは、challenge の形・genesis・期限が合わないときも同じエラーの文になる（`cli.js` L6571〜6586）。拒んだ理由は時計の遅れ（#34）がいちばんありそうだが、文からはどれか決められない（推論）。
- resync の後も縮まらなかったのは、Windows Time が小さなずれを一度に直さず少しずつ寄せるため、と読む（推論。設定は見ていない）。

この観測は、筆者が network#34 にコメントした（https://github.com/flop-labs/network/issues/34#issuecomment-6647104706 ）。手で署名する回避策（#34 にある、鍵ファイルを使って手で署名するもの）は勧めない。CLI の修正を待つ。

### 段 6：残高（2026-11-06、止めた）

- `… flop balance --help`：説明は `Check account balance`。引数は SS58 のアドレス（`address`、「Account address (SS58)」）だけで、ほかは `--json`。
- 渡すアドレスを CLI から知る手段を探した。`… flop keys inspect --help` の説明は「Inspect public metadata from an encrypted Polkadot JSON keystore」。`agent credential` が作った鍵ファイル `~/.flop/devnet-enroll-wallet.json` を 1 回渡した：`… flop keys inspect --json <鍵ファイル>`。
- 出力：`{"ok": false, "code": "INVALID_INPUT", "message": "invalid Polkadot JSON keystore", "retryable": false}`。秘密は出ていない。
- `wallet`・`wallet agent`・`agent wallet`・`agent keys` の help にも、アドレスを出す手段はなかった（`wallet` の下は `agent` だけ。`wallet agent`・`agent wallet` は `create`、`agent keys` は `generate` だけで、どれも状態を変えるので流していない）。
- → 段 6 はここで止めた。筆者が #121 を出した（Feature request、https://github.com/flop-labs/network/issues/121 ）。鍵ファイルは開いていない。

## まだ試していない

- 段 4（鍵）と段 5（faucet）：`flop keys --help`・`flop keys generate --help`・`flop faucet --help` を見ただけ。`keys generate` は流していない（鍵ファイル `~/.flop/devnet-enroll-wallet.json` は `agent credential` が作った）。`faucet request` は流していない。ただし `agent credential` の中で faucet への請求は起きていて、その結果は上の 6 回の表のとおり。
- 段 8（推論、`/v1/chat/completions`）：試していない。
- この先のつまずきは、参加者の報告では、faucet の Grant pending や 1 日の上限、gateway の登録の上限、推論の `429`（channel pool is full）、allowance の約 24 時間の期限などがある。allowance が 15 時間ほどで切れたという報告も 1 件ある（#6 の 2026-11-15T07:35:20Z のコメント）。約 24 時間で切れた例もある（#237）。TROUBLE.md の faucet・credential・推論の表へ。

## 秘密のファイル

- `~/.flop/devnet-enroll-wallet.json`：`agent credential` が作る鍵ファイル。秘密鍵。人に見せない・貼らない。CLI 0.4.0 のソースでは、復元用の mnemonic が平文で入る（TROUBLE.md の「秘密のファイル」）。CLI 0.4.4 でも、`agent credential` は `--keystore`・`--password-env` を使わず、暗号化していない wallet のファイルを自分で書く、という参加者の報告がある（#330。筆者の #121 にも触れている）。
- `~/.flop/devnet-wallet-agent-key.json`：bearer キーのキャッシュ（#34 の報告）。秘密。人に見せない・貼らない。
- `agent credential --print-key` は bearer キーを画面に出すので使わない。
- #34 の「鍵ファイルで手で署名する」回避策は勧めない。

## 筆者が上流に出したもの

- network#52：docs の FAQ の食い違い（筆者が出した Issue）。2026-11-20 に、保守担当者が公開している FAQ で直したと書いて閉じた（completed）。
- network#34 へのコメント：Windows で同期しても約 0.19 秒遅れが残り、`agent credential` が通らなかった観測（https://github.com/flop-labs/network/issues/34#issuecomment-6647104706 ）。
- network#121：`agent credential` の wallet のアドレスを CLI で知る手段がなく、`keys inspect` がその鍵ファイルを拒む件（筆者が出した Issue、Feature request）。open のままで、保守担当者の返事はない（2026-11-26 06:10Z の時点）。#330・#411 が #121 に触れている。
