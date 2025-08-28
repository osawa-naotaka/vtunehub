# VTuneHub Phase 1 (MVP) 要求仕様書

## 1. 序論

### 1.1 目的
本文書は、「VTuneHub」システムのPhase 1（MVP: Minimum Viable Product）開発に必要な要求事項を定義するものである。Phase 1では、システムの基盤となる認証機能と基本的な配信スケジュール管理機能の実装に焦点を当てる。

### 1.2 対象範囲
Phase 1では、VTuber活動管理の最も基本的な機能である配信スケジュール管理と、それを支える認証基盤を実装する。Cloudflare Workers/Pages/D1/KVを使用した完全サーバーレスアーキテクチャで構築する。

### 1.3 定義と略語
- **VTuber**: バーチャルYouTuber、配信者
- **MVP**: Minimum Viable Product、実用最小限の製品
- **Magic Link**: パスワードレス認証方式
- **SPA**: Single Page Application
- **Workers**: Cloudflare Workers（エッジコンピューティング環境）
- **D1**: Cloudflare D1（SQLiteベースのデータベース）
- **KV**: Cloudflare KV（Key-Valueストレージ）

## 2. 全体的記述

### 2.1 製品の展望（Phase 1）
VTuneHub Phase 1は、VTuberが配信スケジュールを簡単に管理し、リスナーに公開できる最小限の機能を提供する。技術的知識がないVTuberでも直感的に使用でき、Magic Link認証により安全かつ簡単にアクセスできるシステムを実現する。

### 2.2 製品の機能（Phase 1）
1. Magic Link認証システム
2. 基本的な配信スケジュール管理（CRUD操作）
3. シンプルな公開スケジュールページ
4. レスポンシブ対応の管理画面

### 2.3 ユーザークラスと特性

#### 2.3.1 VTuber（主要ユーザー）
- 技術レベル: 非エンジニア
- 要求: 簡単な認証、直感的なスケジュール管理
- 使用頻度: 週2-3回程度のスケジュール更新

#### 2.3.2 リスナー（二次ユーザー）
- 技術レベル: 一般ユーザー
- 要求: 配信予定の簡単な確認
- 使用頻度: 日常的なスケジュール確認

### 2.4 運用環境

#### 2.4.1 ハードウェア環境
- サーバー: Cloudflare エッジネットワーク（サーバーレス）
- クライアント: PC、タブレット、スマートフォン

#### 2.4.2 ソフトウェア環境
- バックエンド: Cloudflare Workers (JavaScript/TypeScript)
- フロントエンド: React + TypeScript
- データベース: Cloudflare D1 (SQLite)
- セッション管理: Cloudflare KV
- ブラウザ要件: Android/Windows Chrome (Baseline前提)

### 2.5 設計と実装の制約
- Cloudflare無料プランの制限内で動作
- Workers: 10万リクエスト/日
- D1: 10GB容量
- KV: 10万読み取り/日
- メール送信: 外部サービス（Resend等）利用

### 2.6 前提条件
- ユーザーは有効なメールアドレスを所有している
- インターネット接続が利用可能
- モダンブラウザを使用している

## 3. 要求仕様

### 3.1 機能要件

#### 3.1.1 認証システム

##### FR-1: Magic Link認証
- 優先度: 必須
- 説明: メールアドレスのみでパスワードレス認証を実現
- 詳細:
  - メールアドレス入力フォーム
  - 認証リンクの生成（有効期限15分）
  - メール送信（外部API経由）
  - ワンクリック認証完了
  - エラーハンドリング（無効なメール、期限切れ等）

##### FR-2: セッション管理
- 優先度: 必須
- 説明: 認証後のセッション維持と管理
- 詳細:
  - 30日間有効なセッション生成
  - httpOnly Cookieでのセッション保存
  - セッション検証ミドルウェア
  - ログアウト機能
  - セッション自動延長

##### FR-3: レート制限
- 優先度: 必須
- 説明: 認証リクエストの制限による悪用防止
- 詳細:
  - IPアドレスごとに10回/時の制限
  - メールアドレスごとに5回/時の制限
  - 制限超過時のエラーメッセージ表示

#### 3.1.2 配信スケジュール管理

##### FR-4: スケジュール作成
- 優先度: 必須
- 説明: 新規配信予定の作成機能
- 詳細:
  - タイトル入力（必須、最大100文字）
  - 日時選択（必須、現在時刻以降）
  - 配信プラットフォーム選択（YouTube/Twitch/ニコニコ）
  - 配信タイプ選択（雑談/ゲーム/歌枠/コラボ）
  - 説明文入力（任意、最大500文字）
  - 公開/非公開の設定

##### FR-5: スケジュール一覧表示
- 優先度: 必須
- 説明: 登録済みスケジュールの一覧表示
- 詳細:
  - カード形式での表示
  - 時系列順（直近が上）
  - ページネーション（20件/ページ）
  - 配信タイプ別の色分け
  - 公開/非公開の視覚的区別

##### FR-6: スケジュール編集
- 優先度: 必須
- 説明: 既存スケジュールの編集機能
- 詳細:
  - 全項目の編集可能
  - 変更履歴の保持（最終更新日時）
  - 編集中の自動保存（下書き機能）

##### FR-7: スケジュール削除
- 優先度: 必須
- 説明: スケジュールの削除機能
- 詳細:
  - 削除確認ダイアログ
  - ソフトデリート（論理削除）
  - 削除後30日間は復元可能

#### 3.1.3 公開ページ

##### FR-8: 公開スケジュール表示
- 優先度: 必須
- 説明: リスナー向けの配信予定公開ページ
- 詳細:
  - 認証不要でアクセス可能
  - 公開設定のスケジュールのみ表示
  - 今後1ヶ月分の予定表示
  - モバイルレスポンシブ対応
  - OGP対応（SNSシェア用）

##### FR-9: カレンダービュー（簡易版）
- 優先度: 高
- 説明: 月間カレンダー形式での表示
- 詳細:
  - 当月のカレンダー表示
  - 配信予定日にマーカー表示
  - 日付クリックで詳細表示
  - 前月/翌月への移動

### 3.2 非機能要件

#### 3.2.1 使用性

##### NFR-1: ユーザビリティ
- 優先度: 必須
- 説明: 直感的で使いやすいインターフェース
- 詳細:
  - 3クリック以内で主要機能にアクセス
  - 明確なナビゲーション
  - 分かりやすいエラーメッセージ
  - ローディング状態の表示
  - 成功/失敗のフィードバック

##### NFR-2: レスポンシブデザイン
- 優先度: 必須
- 説明: 各種デバイスでの最適表示
- 詳細:
  - モバイル（～767px）対応
  - デスクトップ（768px〜）対応
  - タッチ操作の最適化

#### 3.2.2 信頼性

##### NFR-3: 可用性
- 優先度: 高
- 説明: 高い稼働率の維持
- 詳細:
  - 99.5%以上の稼働率目標
  - Cloudflareのグローバルネットワーク活用
  - エラー時のグレースフルデグラデーション

##### NFR-4: データ整合性
- 優先度: 必須
- 説明: データの正確性と一貫性
- 詳細:
  - トランザクション処理の実装
  - 楽観的ロックによる同時更新制御
  - データバリデーション

#### 3.2.3 性能

##### NFR-5: 応答時間
- 優先度: 高
- 説明: 快適な操作性を提供する応答速度
- 詳細:
  - 初期ページロード: 2秒以内
  - API応答: 200ms以内（p95）
  - インタラクティブまでの時間: 3秒以内

##### NFR-6: 同時接続
- 優先度: 中
- 説明: 複数ユーザーの同時利用
- 詳細:
  - 100同時接続の処理
  - リソース効率的な実装

#### 3.2.4 セキュリティ

##### NFR-7: 認証セキュリティ
- 優先度: 必須
- 説明: 安全な認証の実装
- 詳細:
  - HTTPS通信必須
  - CSRF対策（SameSite Cookie）
  - XSS対策（React自動エスケープ）
  - セキュアなトークン生成

##### NFR-8: データ保護
- 優先度: 必須
- 説明: ユーザーデータの適切な保護
- 詳細:
  - SQLインジェクション対策（プリペアドステートメント）
  - 個人情報の最小限収集
  - ログへの機密情報非出力

## 4. データモデル

### 4.1 データベーススキーマ（D1）

#### users テーブル
```sql
CREATE TABLE users (
  id TEXT PRIMARY KEY,
  email TEXT UNIQUE NOT NULL,
  name TEXT,
  created_at INTEGER NOT NULL,
  updated_at INTEGER NOT NULL,
  deleted_at INTEGER
);
CREATE INDEX idx_users_email ON users(email);
```

#### auth_tokens テーブル
```sql
CREATE TABLE auth_tokens (
  token TEXT PRIMARY KEY,
  user_email TEXT NOT NULL,
  expires_at INTEGER NOT NULL,
  used_at INTEGER,
  created_at INTEGER NOT NULL
);
CREATE INDEX idx_auth_tokens_email ON auth_tokens(user_email);
```

#### streams テーブル
```sql
CREATE TABLE streams (
  id TEXT PRIMARY KEY,
  user_id TEXT NOT NULL,
  title TEXT NOT NULL,
  scheduled_at INTEGER NOT NULL,
  platform TEXT NOT NULL,
  stream_type TEXT NOT NULL,
  description TEXT,
  thumbnail_url TEXT,
  is_public INTEGER NOT NULL DEFAULT 1,
  created_at INTEGER NOT NULL,
  updated_at INTEGER NOT NULL,
  deleted_at INTEGER,
  FOREIGN KEY (user_id) REFERENCES users(id)
);
CREATE INDEX idx_streams_user_id ON streams(user_id);
CREATE INDEX idx_streams_scheduled_at ON streams(scheduled_at);
```

### 4.2 KVストレージ構造

#### セッション
```typescript
// Key: session:{sessionId}
interface SessionData {
  userId: string;
  email: string;
  createdAt: number;
  expiresAt: number;
}
```

#### レート制限
```typescript
// Key: rate:auth:ip:{ipAddress}
// Key: rate:auth:email:{email}
interface RateLimit {
  count: number;
  resetAt: number;
}
```

## 5. インターフェース要件

### 5.1 ユーザーインターフェース

#### 5.1.1 認証画面

##### UI-1: ログイン画面
- 必須要素:
  - メールアドレス入力フィールド
  - 送信ボタン
  - エラーメッセージ表示エリア
  - ローディングインジケーター

##### UI-2: 認証完了画面
- 必須要素:
  - 成功メッセージ
  - ダッシュボードへの遷移ボタン

#### 5.1.2 管理画面

##### UI-3: ダッシュボード
- 必須要素:
  - 直近の配信予定（3件）
  - 新規作成ボタン
  - スケジュール一覧へのリンク
  - ユーザーメニュー（ログアウト）

##### UI-4: スケジュール一覧
- 必須要素:
  - スケジュールカード
  - 新規作成ボタン
  - ページネーション
  - 検索/フィルター（Phase 2で実装）

##### UI-5: スケジュール作成/編集
- 必須要素:
  - 入力フォーム
  - 保存/キャンセルボタン
  - バリデーションエラー表示
  - 削除ボタン（編集時のみ）

#### 5.1.3 公開ページ

##### UI-6: 公開スケジュール
- 必須要素:
  - 配信予定リスト
  - 日付/時刻表示
  - プラットフォームアイコン
  - シェアボタン（SNS）

### 5.2 APIインターフェース

#### 5.2.1 認証API

##### API-1: Magic Link送信
```
POST /api/auth/magic-link
Request: { email: string }
Response: { message: string }
```

##### API-2: 認証確認
```
POST /api/auth/verify
Request: { token: string }
Response: { sessionId: string }
```

##### API-3: ログアウト
```
POST /api/auth/logout
Response: { success: boolean }
```

#### 5.2.2 スケジュールAPI

##### API-4: スケジュール作成
```
POST /api/streams
Request: Stream object
Response: Created stream
```

##### API-5: スケジュール取得
```
GET /api/streams?page=1&limit=20
Response: { streams: Stream[], total: number }
```

##### API-6: スケジュール更新
```
PUT /api/streams/:id
Request: Stream object
Response: Updated stream
```

##### API-7: スケジュール削除
```
DELETE /api/streams/:id
Response: { success: boolean }
```

##### API-8: 公開スケジュール取得
```
GET /api/public/streams
Response: { streams: PublicStream[] }
```

## 6. 実装順位

1. 機能仕様作成
2. D1スキーマ作成
3. 環境構築（Cloudflare、開発環境）
4. Magic Link認証基本実装
5. セッション管理
6. スケジュールCRUD API
7. 管理画面基本UI
8. 公開ページ
9. レスポンシブ対応

## 7. 受け入れ基準

### 7.1 機能テスト
- [ ] Magic Link認証が正常に動作する
- [ ] セッションが30日間維持される
- [ ] スケジュールの作成・編集・削除ができる
- [ ] 公開ページで配信予定が確認できる
- [ ] モバイル端末で正常に表示される

### 7.2 非機能テスト
- [ ] ページロード2秒以内
- [ ] API応答200ms以内
- [ ] 100同時接続で正常動作
- [ ] XSS/CSRF攻撃への耐性

### 7.3 ドキュメント
- [ ] APIドキュメント作成
- [ ] セットアップガイド作成
- [ ] 基本的な使い方ガイド作成

## 8. リスクと対策

### 8.1 技術的リスク

| リスク | 影響度 | 発生確率 | 対策 |
|-------|--------|----------|------|
| メール送信API障害 | 高 | 低 | 複数プロバイダー対応準備 |
| D1パフォーマンス問題 | 中 | 中 | インデックス最適化、KVキャッシュ |
| セッション管理の複雑さ | 中 | 中 | 既存ライブラリの活用検討 |

## 9. 成功指標

### 9.1 定量的指標
- 認証成功率: 95%以上
- ページロード時間: 2秒以内達成率90%以上
- エラー率: 1%未満

### 9.2 定性的指標
- 直感的なUIで説明書不要
- スムーズな認証フロー
- 安定した動作

