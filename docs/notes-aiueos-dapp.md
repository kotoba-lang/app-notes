# app-notes — itonami（dapp）上の分散型設計

状態: 第 1 スライス実装（2026-10-20）。正本はこの文書。
追跡: Linear JUN-128（親 JUN-116）。

## 位置づけ

app-notes は `mokuroku`（shared catalog kernel、ADR-2608035000）上の Notes
アプリで、itonami（dapp）の app bundle。aiueos の `examples/apps/notes.edn`
（`:app/notes`）が参照実装。capability は `fs/app-data`（app の私有 store。
ホストが provider を供給し、そこで grant を消費する）。

## このスライス: 「title は派生であって保存ではない」の guest 化

README の三つの決定のうち一つ目を判定コアにした。model.cljk の
`derive-title`（最初の非空白行を **trim** して title に）と同じセマンティクスを、
body の UTF-8 バイト列に対する pure 判定として実装。

| 層 | 実体 | 状態 |
|---|---|---|
| 判定コア guest | `aiueos/notes_model.cljk` — title-start（最初の非空白バイト）/ title-length（LF まで + 末尾 trim）/ note-status（0 = title あり / 1 = untitled） | **実装済み・実測** |
| 記述子 | `aiueos/notes.edn`（notes.edn 形。imports 空 — fs/app-data の効果はホスト層） | **実装済み** |
| parity | `test/app_notes/model_parity_test.cljk` — model/derive-title と guest の index / length が一致 | **実装済み・実測** |

実測: `amu test aiueos/notes_model.cljk` → **18/18**（jvm-kir / js / wasm）。
parity → 1 tests / 10 assertions / 0 failures。

## 実測で分かった model の正確なセマンティクス（guest はこれに追従）

- title は **trim される**（前後空白が長さに入らない）。
- `derive-title` の引数は entry ではなく **body 文字列**。
- 空 body・空白のみは untitled（"New Note" を付けるのはホスト。プレースホルダ
  文字列は判断ではない — note-status の 1 が判定）。
- 行区切りは LF（CRLF の CR も trim で落ちる）。空白集合は space / tab / CR / LF。

## 未実装境界

- fs/app-data の provider 接続（ホスト層。examples/apps/notes.edn の grant 経路）。
- aiueos session/grant への載せ（fleet 共通の残課題）。
- pin / sort の判定（「pin は view の決定」— model の view 側。判定ではない）。
- mon（DNA）化は kaikei.ledger / aozora.boundary のレシピ適用が次のスライス。
