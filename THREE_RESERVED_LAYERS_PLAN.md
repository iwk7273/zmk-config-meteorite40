# Meteorite40 追加3レイヤー Firmware 実装計画

## 1. 目的と完成状態

Meteorite40 / Meteorite40 Low の常設5レイヤーはそのまま維持し、ZMK Studio から追加できる予約レイヤーを3枠増やす。

- 接続直後・settings reset直後: 5 active layers + 3 available layers
- Studioで3回追加後: 8 active layers + 0 available layers
- 追加枠のstock binding: 40キーすべて `&none`
- 追加枠の名称: `RESERVE 1`、`RESERVE 2`、`RESERVE 3`
- 追加枠のBall Profile既定値: `BALL_OFF`

ZMK Studio向けの追加容量は、通常レイヤーを最初から8層表示するのではなく、keymap childを `status = "reserved"` にするZMK標準方式で確保する。これにより、通常の起動状態は既存の5層を変えず、StudioのAdd layer操作で必要な分だけ追加できる。

## 2. 前提と境界

### 採用する前提

- 「キーは全てnone」は、各物理レイアウトで公開される40個の `bindings` をすべて `&none` にする意味とする。
- ロータリーエンコーダーの回転は物理キーとは別の `sensor-bindings` なので、既存レイヤーと同じ左右の音量操作を維持する。
- 追加枠のstock状態は `&none` だが、Studioで編集・保存した後にそのレイヤーをDelete/AddまたはUndo/Redoした場合は、ZMKのstable layer IDに保存済みの内容が復元される。Addのたびに強制初期化する仕様にはしない。

### 変更するリポジトリ

- `zmk-config-meteorite40/` のみ

### 変更しないリポジトリ

- `zmk/`: reserved layer、`available_layers`、Add/Remove/Restore RPCは既存実装で対応済み
- `zmk-feature-meteorite-config/`: Ball Profileは最大16層、指定配列より後ろは `BALL_OFF` になる既存仕様で対応済み
- `zmk-studio-messages/` / `zmk-studio-ts-client/`: keymap layer数は可変長で、schema変更不要
- `zmk-pmw3610-driver/`: センサー内部の変更なし
- `config/west.yml`: revision pin変更なし

## 3. Firmware変更設計

### 3.1 通常版keymap

対象: `config/meteorite40.keymap`

既存 `layer_4` の後ろに、次の3 childを順番固定で追加する。

| Stable layer ID | Node | display-name | status | 物理キー | sensor-bindings |
|---:|---|---|---|---|---|
| 5 | `layer_5` | `RESERVE 1` | `reserved` | 40個すべて `&none` | 既存と同じ左右encoder音量操作 |
| 6 | `layer_6` | `RESERVE 2` | `reserved` | 40個すべて `&none` | 既存と同じ左右encoder音量操作 |
| 7 | `layer_7` | `RESERVE 3` | `reserved` | 40個すべて `&none` | 既存と同じ左右encoder音量操作 |

`bindings` は現行レイアウトの10 + 10 + 12 + 8の並びを保つ。`&trans` は混ぜない。`&none` は下位レイヤーへのfall-throughを止めるため、空レイヤーをToggleで有効化すると解除キーも遮断し得る。この挙動は要件どおりとし、実機試験ではMomentary activationで安全に確認する。

### 3.2 Low版keymap

対象: `config/meteorite40_low.keymap`

通常版と同じstable ID、名称、`reserved`、40個の `&none`、sensor defaultsを追加する。通常版とLow版で予約枠の順序・名称・既定値を一致させ、Studioから見たモデル差を作らない。

### 3.3 Ball Profile defaults

対象: `boards/shields/common/meteorite40.dtsi`

- layer indexコメントを0〜7へ更新する。
- `layer-profiles` を8要素にし、index 5〜7へ `BALL_OFF` を明示する。
- 既存のindex 2だけ `BALL_SCROLL`、その他は `BALL_OFF` という挙動は変えない。

module側は配列の不足分を `BALL_OFF` にするため機能上は5要素のままでも動くが、firmware sourceだけで8層分の既定値を監査できるよう明示的に揃える。

このファイルには計画策定時点で、`POINTER_PROFILE` コメントの未コミット変更がある。実装時はそのユーザー変更を保持し、追加レイヤー用の別領域だけを編集する。

## 4. Settings互換性

- 既存firmwareの5要素 `keymap/layer_order` を読み込む場合、先頭5 IDを復元し、追加3枠はinvalid/reservedのまま残ることを確認する。
- 既存のkey binding settingsはstable ID 0〜4なので、ID 5〜7の追加で再解釈しない。
- custom config payloadは既に最大16層分を保存するため、payload migrationは行わない。
- Save後に追加済みレイヤーを再接続しても、layer order、名称、binding、Ball Profileが維持されることを確認する。
- Reset後は追加枠が再びreservedへ戻り、5 active + 3 availableになることを確認する。

## 5. 実装手順

1. 実装開始時に `zmk-config-meteorite40` のbranch、ahead commit、未コミット差分を再確認し、既存差分を混ぜないfeature branchを用意する。
2. `meteorite40.keymap` へstable ID 5〜7の予約childを追加する。
3. `meteorite40_low.keymap` へ同一構成の予約childを追加する。
4. `meteorite40.dtsi` のBall Profileコメントと8要素defaultsを更新する。
5. 静的確認で、両keymapの予約枠数、順序、40 binding、全 `&none`、sensor binding数を照合する。
6. ローカルbuildで通常版・Low版をpristine buildする。
7. 対応firmwareを実機へ書き込み、Studio RPCでreserved capacityと保存/reset lifecycleを確認する。
8. firmware repo単独でcommitする。Studio側のfixture/test変更とはcommitを分ける。

## 6. 検証計画

### 静的確認

- `layer_5`、`layer_6`、`layer_7` が両keymapに各1回だけ存在する。
- 3 childすべてが `status = "reserved"` である。
- 各childの `bindings` が40個で、全要素が `&none` である。
- `sensor-bindings` が左右2件で、通常版とLow版が一致する。
- Ball Profile defaultsが8件で、index 2だけ `BALL_SCROLL`、index 5〜7が `BALL_OFF` である。
- `config/west.yml`、ZMK fork、protobuf/client pinに差分がない。

### Build

`build-workspace/AGENTS.md` と `LOCAL_BUILD.md` に従い、`build-workspace/build.ps1` で通常版・Low版をpristine buildする。

確認対象:

- `build-workspace/output/meteorite40.uf2`
- `build-workspace/output/meteorite40_low.uf2`
- wrapperが同時生成するdebug版2件
- build log上でdevicetree binding数、layer数、flash/RAM超過エラーがないこと

### 実機 + Studio統合確認

1. settings reset後に5レイヤーだけ表示され、Add layerが有効である。
2. firmware RPCの `availableLayers` が3である。
3. Addを1回行うと `RESERVE 1` がindex 5へ追加され、40キーがすべてNone表示になる。
4. 3回のAddで `RESERVE 1`〜`RESERVE 3` が順に追加され、`availableLayers` が0になり、4回目を操作できない。
5. 新規レイヤーのBall ProfileがOffである。
6. 新規レイヤーの左右encoderが計画どおり音量操作として表示・動作する。
7. 1キーを編集してSaveし、再接続後も追加レイヤーと編集内容が残る。
8. Delete、Undo、Redoでstable IDと内容が維持される。
9. Resetで追加3層が消え、5 active + 3 availableへ戻る。
10. 通常版とLow版の両方で同じ結果になる。

## 7. 完了条件

- Meteorite40 / Lowのfirmwareがどちらも5 active + 3 reservedとしてbuildできる。
- Studioから予約枠を3つ追加でき、stock時の全40キーが `&none` である。
- 追加枠の名称・stable ID・Ball Profile・encoder既定値が両variantで一致する。
- Save/再接続/Delete/Undo/Redo/Resetの既存semanticsが回帰しない。
- ZMK core、Meteorite module、protobuf、TS client、west pinを変更していない。
- 既存のローカル差分とahead commitを計画外のcommitへ混ぜていない。

## 8. 今回の非ゴール

- 起動直後から8レイヤーすべてをactive表示すること
- 追加レイヤーへ到達するキーをstock keymapへ割り当てること
- Addのたびに保存済みslot内容を強制的に `&none` へ消去すること
- encoderを完全無効化するための新しいsensor behavior追加
- 最大16層への拡張、schema変更、firmware RPC変更
