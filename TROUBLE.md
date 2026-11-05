# FLOP Devnet（builder）つまずき早見表（非公式）

> - **非公式のメモ。** FLOP Labs の公式ではない。VNL Works の非公式のメモ（筆者＝VNL Works、GitHub は vanilla3）。
> - **最終確認 2026-11-05 00:27Z**（flop-labs/network の Issue を取得した時刻）。下の状態の行（直った・直っていない・報告がない）は、この時点のもの。
> - 最終確認から 7 日以上たっていたら、状態の行は古いとみなし、公式 docs と flop-labs/network を見てほしい。
> - 公式かどうかの確かめ先：https://docs.flop.network/docs/official-links
> - 対象：`@flop-labs/cli` 0.4.0 で Devnet を builder（agent）として試すとき。validator・miner は扱わない。
> - 出典：flop-labs/network の Issue とコメント（2026-11-05 00:27Z の取得）。ほとんどは**参加者の報告**で、筆者は確かめていない。「（試した）」は、筆者が自分の端末で見たもの（試した：2026-11-04〜11-05）。
> - 時刻はすべて UTC（末尾の `Z`）。
> - 2026-11-05 00:27Z の時点で、下の Issue はすべて open で、保守担当者の返事はどれにもない（network 全体で保守担当者の返事は #11 の1件だけ。#1 には「新しい Issue には 3 営業日以内に最初の返事」とある）。
> - 正本は公式 docs（docs.flop.network）と flop-labs/network。この表は遅れることがある。docs の testnet-and-timeline の FAQ は「agent は Devnet を使えない」と書いているが、同じページの別の箇所や CLI は使える前提で書かれている（この食い違いは network#52）。
> - Devnet は報酬がなく、使い捨て（公式 official-links・testnet-and-timeline）。FLOP が claim・mint・登録のためにシードフレーズや入金を求めることはない（official-links）。

## 言葉
- **quick fund**：faucet の、本人確認なしの付与（25 FLOP。価値のないテスト用）。**verified**：BrightID などの本人確認を通す経路。
- **credential・bearer キー**：`agent credential` で gateway の API キーを取ること。取れたキーが bearer キーで、持っている人なら誰でも使える。
- **allowance**：gateway が API キーに付ける使用枠。発行から約 24 時間で切れる（#31）。
- **owner／agent**：wallet の持ち主の鍵と、その下で動く agent の鍵。**agent wallet**：`wallet agent create` で chain に作るもの。
- **channel・escrow**：推論のために開く支払いの通り道と、そこに預ける FLOP。
- **WAF**：Cloudflare などが、おかしく見える通信を止める仕組み。**nonce**：その鍵が送った取引の数。

## 先に知っておくこと（全体に効く 5 つ）
1. **同じ請求を繰り返さない。** faucet はアドレスごとに 24 時間に 1 回。タイムアウトしても付与は通っていて、24 時間のクールダウンが始まっていることがある（#22）。SDK の `FaucetClient` は既定で、429 で断られても同じ請求を 3 回まで送るという報告がある（#71）ので、SDK から請求するときは自分で 1 回に抑える。
2. **アドレスや wallet を作り直さない。** 新しいアドレスを作っても同じところで止まった例がある（#43）。agent wallet は owner ごとに 10 個までで、止めても（`deactivate_wallet`）数は減らない（#48）。
3. **時計を合わせてから始める**（#34）。
4. **`/health` が ok でも通るとは限らない。** faucet の `readinessScope` は「dependencies for this client; grant eligibility checked on request」で、請求できるかは請求したときに決まる（#51・#22）。
5. **allowance は約 24 時間で切れ、更新の手順は公開されていない**（#6・#31）。credential を取ると時計が動き出す。推論が 429（channel pool is full）で断られ続けたまま期限が切れた報告がある（#6・#31）。取ったら間を空けて試し、連打はしない。

## 秘密のファイル（人に見せない・貼らない・消さない）
- `~/.flop/devnet-enroll-wallet.json`：`agent credential` が作る鍵ファイル（できるのは試した）。`--keystore` を渡してもこのファイルが使われる、という報告がある（#22・#34）。**秘密鍵。**
- `~/.flop/devnet-wallet-agent-key.json`：bearer キーのキャッシュ（#34 の報告）。
- `agent credential --print-key` は bearer キーを画面に出すので使わない。

## 表：エラーの文 → 原因 → どうする

「待つ」は、サーバー側の問題で手元では直せない、という意味。

### faucet（残高をもらう段。`faucet request` と、`agent credential` の中）

| エラーの文（英語の原文） | どこで | 原因（network#N） | どうする |
|---|---|---|---|
| `The operation was aborted due to timeout`（ブラウザや `/fund` を直に呼ぶと HTTP `524` のことも、#37・#66） | `faucet request`（`FAUCET_REQUEST_FAILED`・`retryable: true`）、`agent credential`（`COMMAND_FAILED`・`retryable: false`、試した） | 応答が返らなくても、付与は通っていることがある。数分〜20 分ほどで届いた例（#22、時刻は推定を含む）と、そのあと次の行の Grant pending のまま届かない例（#43 の 3 人）がある | すぐ流し直さない。15 分ほど待つ。`faucet request` なら先に `balance` で残高を見る |
| `Grant pending; retry the original request later`（HTTP 503、`reason: "grant_pending"`、`retryAfterSeconds: 60`）。`agent credential` では `devnet faucet returned HTTP 503: Grant pending; retry the original request later`（試した） | `agent credential`（試した、11/04 に 2 回。その前の 1 回はタイムアウト）、`/fund` を直に呼ぶとき、公式の Quick fund の画面 | 付与が「処理中」で止まる。11/03 には、このあと付与が届いた・credential が通った報告がある（#22）。11/04 にも、06:39Z より前に届いた報告が 1 件ある（#37、時刻は書かれていない）。一方、11/04 の 06:05Z ごろ以降に pending になった分（#43 の 3 人。ほかに、最初の請求がタイムアウトして結果が分からないまま止めた人が #43 に 1 人。ほかに #10、#66 の 2 回目の請求）は、**11/05 00:27Z の時点で解けたという報告がない**。#43 の最初の報告は 11/03 の夜 | 待つ。同じ wallet・アドレスのまま、間を空けて 1 回ずつ |
| `Address funded recently; retry in N min` | `faucet request` | そのアドレスは 24 時間以内に付与を受けている。タイムアウトした最初の請求が実は通っていた例（#22） | `balance` で残高を見る。届いていれば先へ。届いていなければ表示の分だけ待つ |
| `Indexer stale or missing chain progress; retry later`（503） | `faucet request`・`agent credential` | explorer の indexer の遅れ（#8）。参加者の報告では 11/03 に追いついた | 待つ。faucet 0.1.256 からは faucet の `/health` の `dependencies.indexer`（`lagBlocks`）でも遅れが見える（#8） |
| `IP classification unavailable; retry later`（503） | `faucet request`（quick fund。verified の経路でも出た例あり） | faucet が IP の判定をできていない。依存先が落ちているという見方は参加者の推論（#22）。faucet 0.1.245 までは `/health` が ok のまま出ていた。11/04 以降にこの文で断られたという報告はない | 待つ |
| `Faucet daily cap reached; try again later`（429、`reason: "daily_cap"`） | `faucet request` | faucet の 1 日の上限（#51・#8）。上限に届く前に断られた例もある（#51）。11/04 04:39Z ごろ再開したという報告のあとも、断られた人がいる（#59） | 待つ（区切りの時刻は公開されていない） |
| `Identity requested recently; retry in 1440 min`（429）、`Humanity proof failed (brightid): not configured`・`Humanity proof failed (passport): personhood provider not configured`（403） | faucet の verified の経路 | 設定されていない経路や、本人確認は通ったのに付与が来なかった経路でも、24 時間のクールダウンが始まる（#21・#46）。クールダウンはアドレスと経路ごと（#21） | この経路は使わず、CLI の quick fund（`faucet request`）を使う |
| `400 Unknown faucet tier` | faucet の `/fund` を直に呼ぶとき | `{"tier":"quick"}` を付けた。quick fund は tier を付けない（#22） | tier を付けない。ふつうは CLI の `faucet request` を使う |

### credential（gateway の API キーを取る段、`agent credential`）

| エラーの文（英語の原文） | どこで | 原因（network#N） | どうする |
|---|---|---|---|
| `devnet gateway returned an invalid enrollment challenge`（`INVALID_INPUT`） | `agent credential` | 手元の時計が gateway より遅れていると、CLI が challenge を自分で拒む（#34）。報告の例は 147 ms・0.6 秒（#28）・5.5 秒（Windows Time サービスが止まっていた、#34）。Windows Time が動いて同期している端末で（設定は変えていない）、`w32tm /resync /force` の後も time.windows.com より約 0.19 秒遅く、通らなかった（試した。筆者が #34 にコメント） | 観測：同期と resync では縮まらなかった（試した）。推し：1 回の同期や、止まっている Windows Time を動かすのはよい（#34 の本文は macOS で同期すれば直るはずとし、#34 のコメントには Windows Time が止まっていた例がある）。同期のしくみの設定（間隔・時刻源など）を変えることは、この表では試していないので扱わない。手で署名する回避策は勧めない。CLI の修正を待つ（#34） |
| `Wallet enrollment is temporarily unavailable.`（503 `wallet_enrollment_unavailable`） | `agent credential`（faucet の後、gateway に登録する段） | 登録そのものが止まっている（#28。下の上限とは別のエラー） | 待つ |
| `The daily wallet enrollment cap is reached; retry later.`（503 `wallet_enrollment_cap_reached`） | 同上 | gateway の登録の 1 日の上限と読まれている（参加者の推論、#28）。11/04 は 01:50Z ごろから 08:10Z まで断られた報告がある（#28、ほかに #66）一方、05:02Z に通った人もいる（#31）。`Retry-After` なし | 待つ（区切りの時刻は公開されていない） |
| `403 wallet_unfunded` | 同上 | 登録する wallet に残高がない（#59、1 人の報告） | faucet の付与が届くのを待つ |

### 推論（`/v1/chat/completions`）

| エラーの文（英語の原文） | どこで | 原因（network#N） | どうする |
|---|---|---|---|
| `402 payment_required` | キーなしの `/v1/chat/completions` | 想定どおり。チャットにはキーが要る（公式 sdk-references「Model discovery needs no key; chat completions do.」） | 先に credential を取る。モデルの一覧（`GET /v1/models`）はキーなしで読める |
| `BrokerOverloadedError: gateway channel pool is full`（429、`code: "overloaded"`） | `/v1/chat/completions` | gateway の channel が埋まっている（#31・#6）。11/03 は `Retry-After: 1` が付いたが、11/04 の報告（1 人）では付かず、`type` も `server_error` に変わった（#31）。11/03 19:42Z ごろ一部のキーで通った例のあと、通ったという報告はない。11/04 18:40Z まで断られ続けた報告と、断られたまま 15:06Z に期限が切れた報告（21:50Z に投稿）がある（#31）。**11/05 00:27Z の時点で直ったという報告はない**。429 では残高・quota は減らなかったという報告（#31・#6） | 連打しない。間を空けて待つ |
| `401 api_key_expired`、`devnet_enrollment_expired — This devnet allowance expired.`、CLI では `devnet allowance expired; ask the operator to renew it` | `/v1/keys/me`・チャット・`agent credential` | allowance の期限切れ（発行から約 24 時間、#31）。`agent credential` を流し直しても更新されない（#6・#31） | 更新の手順は公開されていない（#6 で頼んでいる人がいるが返事はない）。wallet を作り直して取り直そうとしない |

### CLI（wallet・その他）

| エラーの文・症状 | どこで | 原因（network#N） | どうする |
|---|---|---|---|
| `Tx failed: {"module":{"index":95,"error":"0x00000000"}}` | `wallet agent create` | 95 は `AgentWallet`、error 0 は `WalletAlreadyExists`：その agent の wallet はもうある（#17、metadata からの解読）。すぐ失敗せず止まったまま返らないこともある（#45） | 作り直さない。既にある wallet を使う |
| `Tx failed: {"module":{"index":95,"error":"0x09000000"}}` | `wallet agent create` | `TooManyWallets`：owner ごとに 10 個まで。消す呼び出しはなく、`deactivate_wallet` しても数は減らない（#48、metadata からの解読）。タイムアウトしても nonce は進み、失敗した処理にも hash が付くので、hash だけでは成功か分からない（#48） | 作り直しを繰り返さない |
| `API-WS: disconnected from ws://127.0.0.1:9944: 1006:: Abnormal Closure` が数秒ごとに出て、`flop health` などが返らない | RPC に届かないとき | `--network` の既定は `local`（試した）なので、`--network devnet` を付け忘れると手元の RPC を待ち続ける。届かない RPC には止まらずに再試行し続ける（#14） | Ctrl+C。`--network devnet` を付けたかを見る |
| `Expected input with 32 bytes (256 bits), found N bytes`、最後に `DEPENDENCY_MISSING`（`model not found: …`） | `flop chat --model <名前>` | `--model` はモデルの hash しか受けない（#4） | hash を渡しても、keystore を渡していなければ `INVALID_INPUT`（`explicit --signer or --keystore is required on devnet`）、渡しても次の行で止まる（#4）。`flop chat` は使う前に「流す前に読む」を見る |
| `Supply a registered HTLC appraisal profile, version, model hash and measurement hash`（`COMMAND_FAILED`） | `flop chat`・`flop proxy`（hash を渡したあと） | devnet に、この経路が求める登録済みの appraisal profile がない（参加者の調べ、#18・#4）。CLI から渡す方法もない | 待つ |
| `name`・`version` が読めないバイト列 | `flop model list --json` | #5 | モデルの名前は `GET /v1/models` で見る |
| `total` が `free` と同じ | `flop balance` | `reserved` を足していない（#12）。channel に預けた escrow も出ない（#40） | `free` と `reserved` を別に見る |
| `retryable: false` | `agent credential --json`（試した） | faucet は 60 秒後の再試行を案内している（#43 の `retryAfterSeconds: 60`）のに、CLI は `retryable: false` と出した（試した。この食い違いの報告はない。`retryable: false` が紛らわしいという指摘は時計の場面で #34 にある） | `retryable` の値で諦めない／連打もしない。上の Grant pending の行のとおり |
| `READY` なのに、そのあと gateway や RPC で失敗する | `agent doctor --mode remote` | 既定では node と MCP しか見ない（試した。network に報告はまだない） | `--gateway https://api.flop.network/v1 --check-rpc` を付けて流す（試した：4 項目とも見る） |
| `error code: 1010`（HTTP 403、Cloudflare） | gateway（`/v1/models`）・RPC への読むだけの確認 | Cloudflare の WAF が client を拒んだ（#47、1 人の報告）。#8 では別の client で同じ 1010 が出ており、どの client が止まるかは決まっていない | 公式の CLI で確かめる（試した：`agent doctor` に `--gateway https://api.flop.network/v1 --check-rpc` を付けて届いた）。CLI でも 1010 なら、client やヘッダーを替えて試さず #47 へ |
| `1010`・`Invalid Transaction`・`Inability to pay some fees`（上の WAF の 1010 とは別物） | chain への取引（報告は SDK からの送金 #38 と、miner の手続き #37） | 手数料を払う残高がない、または思っていたのと別のアカウントで署名している（#37・#38） | 署名に使ったアカウントと、その残高を確かめる |

### 流す前に読む（エラーではないが、取り返しがつきにくいもの）
- `flop chat`・`flop proxy`・`flop session run`：失敗した推論の試みが channel を開いて FLOP を escrow した、という報告がある（#40）。報告の例では、あとで `computeChannel.timeout` の呼び出しで全額戻った（#40 のコメント）が、戻すための CLI のコマンドはなく、escrow は `flop balance` にも出ない。到達できない miner だと何も出ずに rc=0 で終わった、という報告もある（#39、1 人）。
- `wallet agent create` の繰り返し：上の CLI の表（#17・#45・#48）。
- `agent credential` の付与先は `~/.flop/devnet-enroll-wallet.json` の鍵のアドレス。ファイルを開いて中身を人に見せない。

## この表の作り方
- エラーの文は Issue の本文とコメントの原文から写した。
- 「11/05 00:27Z の時点」の状態は、その時刻の取得による。
