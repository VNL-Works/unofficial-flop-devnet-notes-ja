# FLOP Devnet を CLI で試した記録（非公式・日本語）

> - **非公式のメモ。** FLOP Labs の公式ではない。VNL Works の非公式のメモで、筆者＝VNL Works（GitHub は vanilla3）。
> - **最終確認 2026-11-06 15:41Z**（flop-labs/network の Issue を取得した時刻）。直った・直っていないなどの状態は、この時点のもの。
> - 最終確認から 7 日以上たっていたら、状態の行は古いとみなし、公式 docs と flop-labs/network を見てほしい。
> - 公式かどうかの確かめ先：https://docs.flop.network/docs/official-links
> - 正本は公式 docs（docs.flop.network、英語）。このメモは遅れることがある（network#2 のコメントの指摘）。
> - docs の testnet-and-timeline（Last updated 2026-10-30、写しの取得は 2026-11-04）の FAQ は「agent は Devnet を使えない」と書いているが、同じページの別の箇所や CLI は使える前提で書かれている。食い違っている（network#52、筆者が出した Issue）。
> - Devnet は報酬も airdrop の重みもなく、使い捨てで何も引き継がない（official-links、Last updated 2026-10-23。testnet-and-timeline、Last updated 2026-10-30。どちらも写しの取得は 2026-11-04）。
> - 同じ testnet-and-timeline の FAQ（https://docs.flop.network/docs/testnet-and-timeline 、Last updated 2026-10-30、写しの取得は 2026-11-04）には「Creating DIDs earns nothing; posting your DID in a TechnoCore room earns nothing.」とある。詳しくは公式 FAQ を。
> - FLOP は claim・mint・登録のためにシードフレーズや入金を求めない（official-links（Last updated 2026-10-23、写しの取得は 2026-11-04）の「FLOP never asks for a seed phrase or a deposit to claim, mint or register anything.」）。
> - 関係の表示：筆者は FLOP の KOL プログラムに応募している。報酬は受けていない（2026-11-05 時点）。
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
- 2026-11-06 に npm の registry を読み、`@flop-labs/cli` の latest は 0.4.0 のまま（0.4.0 は 2026-10-29 公開）だった。

### 段 2〜3：版・help・Devnet への接続（2026-11-04）

- `… flop --version`：`0.4.0`。npm の warn が 2 行（`node-domexception@1.0.0`・`@substrate/connect@0.8.11` が deprecated）。
- `… flop --help`：コマンドは 24 個（`keys`・`wallet`・`faucet`・`agent` ほか）。`--network` の既定は `local`。なので `--network devnet` を毎回付ける。付け忘れると手元の RPC を待ち続けるという報告がある（network#14、参加者の報告。筆者は付け忘れて試してはいない。TROUBLE.md の CLI の表）。
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
- この先のつまずきは、参加者の報告では、faucet の Grant pending や 1 日の上限、gateway の登録の上限、推論の `429`（channel pool is full）、allowance の約 24 時間の期限などがある。TROUBLE.md の faucet・credential・推論の表へ。

## 秘密のファイル

- `~/.flop/devnet-enroll-wallet.json`：`agent credential` が作る鍵ファイル。秘密鍵。人に見せない・貼らない。CLI 0.4.0 のソースでは、復元用の mnemonic が平文で入る（TROUBLE.md の「秘密のファイル」）。
- `~/.flop/devnet-wallet-agent-key.json`：bearer キーのキャッシュ（#34 の報告）。秘密。人に見せない・貼らない。
- `agent credential --print-key` は bearer キーを画面に出すので使わない。
- #34 の「鍵ファイルで手で署名する」回避策は勧めない。

## 筆者が上流に出したもの

- network#52：docs の FAQ の食い違い（筆者が出した Issue）。
- network#34 へのコメント：Windows で同期しても約 0.19 秒遅れが残り、`agent credential` が通らなかった観測（https://github.com/flop-labs/network/issues/34#issuecomment-6647104706 ）。
- network#121：`agent credential` の wallet のアドレスを CLI で知る手段がなく、`keys inspect` がその鍵ファイルを拒む件（筆者が出した Issue、Feature request）。
