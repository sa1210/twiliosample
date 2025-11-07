# Twilio SMS/音声認証 Cloud Functions

TwilioとGoogle Cloud Functionsを使用したSMS・音声通話による認証システムです。

## 機能

- SMS送信による認証コード送信
- 音声通話による認証コード読み上げ
- Firestoreでの認証コード管理
- 認証コードの検証
- 有効期限管理（5分）

## 必要な環境

- Node.js 20以上
- Google Cloud SDK
- Twilioアカウント
- Firebase/Firestoreプロジェクト

## セットアップ

### 1. 依存関係のインストール

```bash
npm install
```

### 2. Google Cloud プロジェクトの設定

```bash
# Google Cloudにログイン
gcloud auth login

# プロジェクトを設定
gcloud config set project sakuma-pandatest
```

### 3. デプロイ

```bash
# 個別にデプロイする場合
gcloud functions deploy sendVerificationCode \
  --runtime nodejs20 \
  --trigger-http \
  --allow-unauthenticated \
  --env-vars-file .env.yaml

gcloud functions deploy verifyCode \
  --runtime nodejs20 \
  --trigger-http \
  --allow-unauthenticated \
  --env-vars-file .env.yaml

# または npm script を使用
npm run deploy
```

## API 使用方法

### 1. 認証コード送信 (sendVerificationCode)

SMS送信の場合:

```bash
curl -X POST https://REGION-PROJECT_ID.cloudfunctions.net/sendVerificationCode \
  -H "Content-Type: application/json" \
  -d '{
    "phoneNumber": "+819012345678",
    "method": "sms"
  }'
```

音声通話の場合:

```bash
curl -X POST https://REGION-PROJECT_ID.cloudfunctions.net/sendVerificationCode \
  -H "Content-Type: application/json" \
  -d '{
    "phoneNumber": "+819012345678",
    "method": "voice"
  }'
```

レスポンス例:

```json
{
  "success": true,
  "message": "Verification code sent via sms",
  "twilioSid": "SM...",
  "expiresAt": "2024-01-01T12:05:00.000Z"
}
```

### 2. 認証コード検証 (verifyCode)

```bash
curl -X POST https://REGION-PROJECT_ID.cloudfunctions.net/verifyCode \
  -H "Content-Type: application/json" \
  -d '{
    "phoneNumber": "+819012345678",
    "code": "123456"
  }'
```

レスポンス例:

```json
{
  "success": true,
  "message": "Verification successful",
  "phoneNumber": "+819012345678"
}
```

## エラーハンドリング

### sendVerificationCode

- `400`: パラメータ不足または無効なmethod
- `500`: Twilio送信エラーまたはFirestoreエラー

### verifyCode

- `400`: パラメータ不足、無効なコード、期限切れ、または既に使用済み
- `404`: 認証コードが見つからない
- `500`: Firestoreエラー

## Firestoreデータ構造

コレクション名: `tw_verification_codes`

ドキュメントID: 電話番号 (E.164形式)

フィールド:

```javascript
{
  code: "123456",              // 6桁の認証コード
  phoneNumber: "+819012345678", // 電話番号
  method: "sms",               // 送信方法: "sms" or "voice"
  expiresAt: Timestamp,        // 有効期限
  createdAt: Timestamp,        // 作成日時
  verified: false,             // 検証済みフラグ
  verifiedAt: Timestamp        // 検証日時（検証後のみ）
}
```

## ローカルテスト

Cloud Functions Frameworkを使用してローカルでテストできます:

```bash
# インストール
npm install -g @google-cloud/functions-framework

# ローカルで起動
export TWILIO_ACCOUNT_SID="AC..."
export TWILIO_AUTH_TOKEN="..."
export TWILIO_PHONE_NUMBER="+1..."

functions-framework --target=sendVerificationCode --port=8080
```

## セキュリティ注意事項

- 本番環境では `--allow-unauthenticated` を外して認証を追加してください
- レート制限の実装を検討してください
- 環境変数ファイル (.env.yaml) は絶対にGitにコミットしないでください
- サービスアカウントのJSONファイルは安全に管理してください

## トラブルシューティング

### Twilioからメッセージが届かない

- Twilioの電話番号が有効か確認
- Twilioアカウントの残高を確認
- 送信先の電話番号がE.164形式であることを確認

### Firestoreエラー

- サービスアカウントに適切な権限があるか確認
- Firestoreが有効になっているか確認

## ライセンス

MIT
