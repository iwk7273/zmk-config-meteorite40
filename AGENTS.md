# AGENTS.md

このリポジトリは Meteorite40 / Meteorite40 Low の firmware config repository です。
shield、layout、keymap、`.conf`、west manifest を管理し、実機へ焼く firmware artifact の入力になります。

## 作業方針

- 主な編集対象は `config/`、`boards/shields/`、`Kconfig`、`build.yaml`、`config/west.yml` です。
- build output や local west workspace の生成物は commit しません。
- `main` は実機 firmware build の基準 branch として扱います。feature work は `feat/*` branch で行い、main に取り込んだ後は古い feature branch を削除する運用にします。
- fork / module の revision を変える場合は、対応 editor / ZMK RPC / TS client の branch と揃っているかを確認してください。
- `.keymap` / `.conf` / `.dtsi` は firmware source です。browser editor の Save は実機 settings 保存であり、この repository の source は更新しません。

## Branch / Fork 運用

- この repo の firmware build 正本 branch は `main` です。
- `config/west.yml` が firmware build で利用する fork / module branch の正本です。
- 現行 manifest は `iwk7273/zmk#feat/meteorite-custom-config-rpc` と `iwk7273/zmk-feature-meteorite-config#feat/meteorite-custom-config-rpc` を参照します。
- ZMK Studio message schema は `zmk/app/west.yml` 経由で `iwk7273/zmk-studio-messages#feat/meteorite-custom-config-rpc` を参照します。
- editor 側は `iwk7273/zmk-studio-ts-client#feat/meteorite-custom-config-rpc` を npm dependency として使います。firmware / protocol branch を変えたら editor dependency も確認してください。
- main へ push した後は、取り込まれた古い feature branch を削除する運用にします。

## Meteorite 拡張の依存関係

- `config/west.yml` は Meteorite 対応 ZMK fork と `zmk-feature-meteorite-config` module を参照します。
- custom config UI を使う firmware には、`meteorite.config` capability を返す ZMK fork と Meteorite module が必要です。
- rotary encoder UI を表示する firmware には、`keymap.sensor_bindings` capability を返す ZMK fork と `Layer.sensor_bindings` / `setLayerSensorBinding` RPC が必要です。

## Rotary Encoder

- Meteorite40 では encoder action を通常 keymap の追加 slot として管理します。
- `boards/shields/*/*-layout.dtsi` には editor / ZMK Studio が keymap position として扱える pseudo slot を含めます。
- `config/*.keymap` の各 layer 末尾には encoder 用 binding を並べます。
- overlay 側の `sensor-bindings` は `zmk,behavior-sensor-rotate-var` behavior を参照し、encoder CW/CCW の `param1` / `param2` を layer ごとに編集します。
- slot order は module 側の `slots` property と一致させます。現行は左 CW、左 CCW、右 CW、右 CCW です。

## Custom Config

- `custom_config.dtsi` と関連 behavior / dt-bindings を keymap から include します。
- CPI、scroll divisor、rotation、scroll reverse、scaling、scroll layer、OS mode の実機設定は firmware settings に保存されます。
- source の `.keymap` にある `&ccfg` は操作キーです。editor の Custom Config view は Meteorite RPC を通じて current/saved/defaults を編集します。

## 検証

- GitHub Actions の firmware build artifact を主な確認元にできます。
- local build を行う場合は、この repo の `config/west.yml` を使った west workspace で Meteorite40 と Meteorite40 Low の両方を確認してください。
- firmware update 後に USB device として認識されない場合は、まず board/shield target、USB UART / Studio RPC snippet、bootloader artifact の取り違えを確認してください。
