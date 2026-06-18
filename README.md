# 祝電管理システム セットアップ手順

## 必要なもの
- GitHubアカウント（お持ちです）
- Firebaseアカウント（お持ちです）
- Node.js（インストール済みの場合はスキップ）

---

## STEP 1：Firebaseプロジェクトの設定

### 1-1. Firestore Database を有効化
1. [Firebase Console](https://console.firebase.google.com/) を開く
2. プロジェクトを選択
3. 左メニュー「Firestore Database」→「データベースの作成」
4. **本番環境モード** で作成（後でルールを設定）
5. ロケーション：**asia-northeast1（東京）** を選択

### 1-2. Storage を有効化
1. 左メニュー「Storage」→「始める」
2. **本番環境モード** で作成
3. ロケーション：**asia-northeast1（東京）** を選択

### 1-3. Firestore セキュリティルールを設定
Firestore → ルール タブで以下を貼り付けて「公開」：
```
rules_version = '2';
service cloud.firestore {
  match /databases/{database}/documents {
    match /shinden/{document=**} {
      allow read, write: if true;
    }
  }
}
```

### 1-4. Storage セキュリティルールを設定
Storage → ルール タブで以下を貼り付けて「公開」：
```
rules_version = '2';
service firebase.storage {
  match /b/{bucket}/o {
    match /shinden/{allPaths=**} {
      allow read, write: if true;
    }
  }
}
```

### 1-5. Webアプリを登録してAPIキーを取得
1. プロジェクトの設定（⚙マーク）→「マイアプリ」→「</>」（Web）
2. アプリのニックネーム：「祝電管理」と入力
3. **「Firebase Hosting も設定する」にチェックを入れる**
4. アプリを登録 → 表示される `firebaseConfig` をメモ（後で使用）

```javascript
// メモする内容（例）
const firebaseConfig = {
  apiKey: "AIzaSy...",
  authDomain: "your-project.firebaseapp.com",
  projectId: "your-project-id",
  storageBucket: "your-project.appspot.com",
  appId: "1:123456789:web:abc..."
};
```

---

## STEP 2：GitHubにアップロード

### 2-1. リポジトリを作成
1. GitHub で新しいリポジトリを作成
   - 名前：`shinden-app`（任意）
   - **Private** を推奨
   - README なしで作成

### 2-2. ファイルをアップロード
ターミナル（またはGit GUI）で以下を実行：
```bash
cd shinden-app
git init
git add .
git commit -m "initial commit"
git branch -M main
git remote add origin https://github.com/あなたのID/shinden-app.git
git push -u origin main
```

---

## STEP 3：Firebase Hosting へのデプロイ設定

### 3-1. Firebase CLIをインストール（初回のみ）
```bash
npm install -g firebase-tools
firebase login
```

### 3-2. GitHub Actions 用のサービスアカウントを取得
```bash
firebase init hosting:github
```
プロンプトに従って操作すると、GitHubリポジトリに自動で以下のSecretsが登録されます：
- `FIREBASE_SERVICE_ACCOUNT`
- `FIREBASE_PROJECT_ID`

### 3-3. 手動でも確認できます
GitHub → リポジトリ → Settings → Secrets and variables → Actions
上記2つが登録されていればOK。

---

## STEP 4：動作確認

1. `main` ブランチにpushすると GitHub Actions が自動でデプロイ開始
2. 完了後のURL（例）：`https://your-project-id.web.app`
3. ブラウザでURLを開く
4. **初回のみ** Firebase設定画面が表示される → STEP 1-5 でメモした値を入力
5. 「保存して接続」を押すと接続完了

---

## STEP 5：各デバイスで使う

デプロイ後のURLを社内で共有するだけで、PC・スマートフォン・タブレットで同時使用できます。

### スマートフォンでホーム画面に追加する方法
**iPhone（Safari）**：共有ボタン → 「ホーム画面に追加」  
**Android（Chrome）**：メニュー → 「ホーム画面に追加」

アプリのように使用できます。

---

## 運用上の注意

| 項目 | 内容 |
|------|------|
| セキュリティ | 現在のルールは社内アクセス前提の簡易設定。外部公開する場合はFirebase Authenticationの追加を推奨 |
| 無料枠 | Firebaseの無料枠（Sparkプラン）：Firestore 1GB、Storage 5GB、Hosting 10GB/月。通常の祝電管理には十分 |
| バックアップ | Firestore は定期的に「エクスポート」でバックアップ推奨 |
| 画像サイズ | スマートフォン撮影の画像はそのままアップロードできます。高解像度のままStorage に保存されます |
