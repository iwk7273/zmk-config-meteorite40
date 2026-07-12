## 🚀 新機能: Ball Profiles の統合

Meteorite40 のトラックボール操作をさらに細かくカスタマイズできる「Ball Profiles」機能を統合しました！
Meteorite Studio 上で「Ball profiles」タブから、トラックボールの挙動を直感的に設定可能です。

![Ball Profiles 設定画面](https://github.com/iwk7273/zmk-config-meteorite40/releases/download/v4.3.0/ball_profiles_highlight.png)

### 主な設定項目
1. **ACTION SENSITIVITY (感度調整)**
   ボールを転がした時の感度（重さ）をスライダーで直感的に調整できます。
2. **LAYER PROFILES (レイヤーごとの動作)**
   各レイヤー（SYMBOLS, NUM / ARROW など）ごとにボールの動作 (Scroll, Browserなど) を個別に割り当てられます。
3. **USER 1 BINDINGS (カスタム割り当て)**
   "User 1" プロファイル選択時は、上下左右の各方向に対して好きなキー操作を自由に割り当てることが可能です。

---
### ✨ その他の改善とアップデート
- **Studio への Bluetooth 接続対応**
  これまで USB 接続が必要だった Studio アプリへの接続が、**Bluetooth 経由でも可能になりました！** ケーブルを繋ぐことなく、ワイヤレスのままで快適にキーマップや設定の変更が行えます。
- **ファームウェア・アップデート通知機能**
  Studio アプリ接続時にキーボードのファームウェアバージョンを確認し、新しいバージョンがリリースされている場合は通知を受け取れるようになりました。

### 📥 アップデート方法
下部の **Assets** からお使いのモデルに合った .uf2 ファイルをダウンロードし、キーボードに書き込んでアップデートを行ってください。
*(Commit 727ff7e)*
