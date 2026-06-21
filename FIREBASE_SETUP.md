# Time of Empire — Firebase クラウド保存セットアップ

config が空の間はローカル保存だけで普通に動きます。以下を済ませて config を貼ると、クラウド同期が点灯します。

## 手順

1. **Firebase プロジェクト作成**
   [console.firebase.google.com](https://console.firebase.google.com/) →「プロジェクトを追加」。

2. **ウェブアプリ登録**
   プロジェクト概要 → `</>`（ウェブ）アイコン → アプリ名を入れて登録。
   表示される `firebaseConfig` の中身（apiKey / authDomain / projectId / appId）を控える。

3. **匿名ログインを有効化**
   左メニュー Build → Authentication →「始める」→ Sign-in method →
   **Anonymous（匿名）** を有効化。

4. **Firestore を作成**
   Build → Firestore Database →「データベースを作成」→
   ロケション選択（例: asia-northeast1）→ 本番モードで開始。

5. **セキュリティルールを設定**（Firestore → ルール タブに貼って公開）

   ```
   rules_version = '2';
   service cloud.firestore {
     match /databases/{database}/documents {
       // 自分のドキュメントだけ読み書きできる
       match /empires/{uid} {
         allow read: if true;                       // ランキング表示用に読み取りは全員可
         allow write: if request.auth != null
                      && request.auth.uid == uid;    // 書き込みは本人のみ
       }
     }
   }
   ```

   > read を `if true` にしているのは、次のスライス（ランキング/共有）で
   > 他人の帝国を見られるようにするため。書き込みは本人だけに固定。

6. **config を貼る**
   `index.html` 内の `FIREBASE_CONFIG = { ... }` に、控えた値を入れる。
   `projectId` が埋まった瞬間にクラウド有効化。

## 動作

- 起動時に匿名ログイン → `empires/{uid}` に保存。
- ローカルとクラウドが食い違ったら **累計生産量（lifetime）が多い方を採用**。
- 書き込みは最短15秒間隔＋タブを閉じる時に確定（無料枠にやさしい）。
- フッターに状態表示：`📁 ローカル保存` / `☁️ クラウド同期` / 接続失敗。

## メモ

- apiKey 等はクライアントに出てOK（公開前提）。安全性は上のルールで担保。
- 次のスライス：`empires` を lifetime / bestCombo で並べてランキング表示＝記録を競う土台。
