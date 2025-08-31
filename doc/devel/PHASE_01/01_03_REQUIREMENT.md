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
- Key-Valueストレージ: Cloudflare KV（セッション管理）
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
  - エラーハンドリング（無効なメール(認証トークンが存在しない)、期限切れ等）

##### FR-2: セッション管理
- 優先度: 必須
- 説明: 認証後のセッション維持と管理
- 詳細:
  - 3日間有効なセッション生成
  - httpOnly Cookieでのセッション保存
  - セッション検証ミドルウェア
  - ログアウト機能
  - セッション自動延長

##### FR-3: KVストア整理
- 優先度: 低
- 説明: KVストアにもう使用されないデータが残ることを回避する
- 詳細:
  - 毎日1回KVストアを整理する
    - 期限切れ認証トークンのKVストアからの削除
    - 期限切れセッションIDのKVストアからの削除

##### FR-4: レート制限
- 優先度: 低
- 説明: 認証リクエストの制限による悪用防止
- 詳細:
  - IPアドレスごとに10回/時の制限
  - メールアドレスごとに5回/時の制限
  - 制限超過時のエラーメッセージ表示

#### 3.1.2 配信スケジュール管理

##### FR-5: スケジュール作成
- 優先度: 必須
- 説明: 新規配信予定の作成機能
- 詳細:
  - タイトル入力（必須、最大100文字）
  - タグ入力(必須)
  - 日時選択（必須、現在時刻以降）
  - 配信プラットフォーム選択（YouTube/Twitch/ニコニコ）
  - 配信タイプ選択（雑談/ゲーム/歌枠/コラボ）
  - 説明文入力（任意、最大500文字）
  - 公開/非公開の設定

##### FR-6: スケジュール一覧表示
- 優先度: 必須
- 説明: 登録済みスケジュールの一覧表示
- 詳細:
  - カード形式での表示
  - 時系列順（直近が上）
  - ページネーション（20件/ページ）
  - 配信タイプ別の色分け
  - 公開/非公開の視覚的区別

##### FR-7: スケジュール編集
- 優先度: 必須
- 説明: 既存スケジュールの編集機能
- 詳細:
  - 全項目の編集可能
  - 変更履歴の保持（最終更新日時）
  - 編集中の自動保存（下書き機能）

##### FR-8: スケジュール削除
- 優先度: 必須
- 説明: スケジュールの削除機能
- 詳細:
  - 削除確認ダイアログ
  - ソフトデリート（論理削除）
  - 削除後30日間は復元可能

#### 3.1.3 公開ページ

##### FR-9: 公開スケジュール表示
- 優先度: 必須
- 説明: リスナー向けの配信予定公開ページ
- 詳細:
  - 認証不要でアクセス可能
  - 公開設定のスケジュールのみ表示
  - 今日を含む1週間分の予定表示（日曜開始）
  - モバイルレスポンシブ対応
  - OGP対応（SNSシェア用）

##### FR-10: カレンダービュー（簡易版）
- 優先度: 低
- 説明: 週間カレンダー形式での表示
- 詳細:
  - 当週のカレンダー表示（日曜開始）
  - 配信予定日にマーカー表示
  - 日付クリックで詳細表示
  - 前週/翌週への移動

### 3.2 非機能要件

#### 3.2.1 使用性

##### NFR-1: ユーザビリティ
- 優先度: 低
- 説明: 直感的で使いやすいインターフェース
- 詳細:
  - 3クリック以内で主要機能にアクセス
  - 明確なナビゲーション
  - 分かりやすいエラーメッセージ
  - ローディング状態の表示
  - 成功/失敗のフィードバック

##### NFR-2: レスポンシブデザイン
- 優先度: 中
- 説明: 各種デバイスでの最適表示
- 詳細:
  - モバイル（〜767px）対応
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

###### emailバリデーション
```typescript
/^[a-zA-Z0-9.!#$%&'*+/=?^_`{|}~-]+@[a-zA-Z0-9](?:[a-zA-Z0-9-]{0,61}[a-zA-Z0-9])?(?:\.[a-zA-Z0-9](?:[a-zA-Z0-9-]{0,61}[a-zA-Z0-9])?)*$/;
```

<input type="email">のバリデーションと一致させる。
https://developer.mozilla.org/ja/docs/Web/HTML/Reference/Elements/input/email#%E6%A4%9C%E8%A8%BC

###### user_idのバリデーション

UUIDv4に準拠。

###### session_id, auth_token, stream_id

UUIDv7に準拠。

###### stateバリデーション

stateは以下のnumberを想定。

```
0: 未公開・未確定
1: 未公開・確定
2: 公開
3: 終了
```

###### tagバリデーション

50文字の文字列を想定。

###### その他

typescriptの型に準拠。

#### 3.2.3 性能

##### NFR-5: 応答時間
- 優先度: 高
- 説明: 快適な操作性を提供する応答速度
- 詳細:
  - 初期ページロード: 2秒以内
  - API応答: 200ms以内（p95）
  - インタラクティブまでの時間: 3秒以内

##### NFR-6: 同時接続
- 優先度: 低
- 説明: 複数ユーザーの同時利用
- 詳細:
  - 2同時接続の処理(別人Vtuber)
  - リソース効率的な実装

#### 3.2.4 セキュリティ

##### NFR-7: 認証セキュリティ
- 優先度: 必須
- 説明: 安全な認証の実装
- 詳細:
  - HTTPS通信必須
  - CSRF対策（SameSite Cookie）
  - XSS対策（React自動エスケープ）
  - CSP設定（self）
  - セキュアなトークン生成
    - crypto.randomUUID()を利用

##### NFR-8: データ保護
- 優先度: 必須
- 説明: ユーザーデータの適切な保護
- 詳細:
  - SQLインジェクション対策（プリペアドステートメント、サニタイズ）
    - サニタイズの具体的な手法は未定
  - 個人情報の最小限収集
  - ログへの機密情報非出力
  - 全ての入力文字に適切な文字数上限を設定（フロントエンド/バックエンド）

## 4. データモデル

### 4.1 データベーススキーマ（D1）

#### users テーブル
```sql
CREATE TABLE users (
  user_id TEXT PRIMARY KEY,
  email TEXT UNIQUE NOT NULL,
  name TEXT,
  created_at INTEGER NOT NULL,
  updated_at INTEGER NOT NULL,
  deleted_at INTEGER
);
CREATE INDEX idx_users_email ON users(email);
```

user_idはUUIDv4を想定。そのまま各人のURLにパスとして含まれる。

#### streams テーブル
```sql
CREATE TABLE streams (
  stream_id TEXT PRIMARY KEY,
  user_id TEXT NOT NULL,
  title TEXT NOT NULL,
  scheduled_at INTEGER NOT NULL,
  platform TEXT NOT NULL,
  stream_type TEXT NOT NULL,
  description TEXT,
  thumbnail_url TEXT,
  tags JSON,
  state INTEGER NOT NULL DEFAULT 0,
  created_at INTEGER NOT NULL,
  updated_at INTEGER NOT NULL,
  deleted_at INTEGER
);
CREATE INDEX idx_streams_user_id ON streams(user_id);
CREATE INDEX idx_streams_scheduled_at ON streams(scheduled_at);
```
stream_idはUUIDv7を想定

### 4.2 KVストレージ構造

#### 認証トークン
```typescript
// key: auth:{auth_token}
interface AuthToken {
  user_email: string;
  created_at: Date;
  used_at: Date;
  expires_at: Date;
};
```

#### セッション
```typescript
// Key: session:{session_id}
interface SessionData {
  user_id: string;
  email: string;
  created_at: Date;
  expires_at: Date;
}
```

#### レート制限
```typescript
// 認証 10回/時
// Key: rate:auth:email:{email}

// API 1000回/時
// Key: rate:api:session:{session_id}

// リクエスト・質問 10回/日
// Key: rate:post:request:{cookie_id}

interface RateLimit {
  count: number;
  reset_at: number;
}
```

### 4.3 typescript型宣言

```typescript
interface Stream {
  info: PublicStream;
  state: number;
  created_at: Date;
  updated_at: Date;
  deleted_at: Date;
}

interface PublicStream {
  stream_id: string;
  user_id: string;
  title: string;
  scheduled_at: Date;
  platform: 'youtube' | 'twitch' | 'niconico';
  stream_ype: 'chat' | 'game' | 'singing' | 'collab';
  description: string;
  thumbnail_url: Url;
  tag: string[];
}
```

#### TTL

- 認証トークン: TTL 15分
- セッション: TTL 3日
- レート制限: TTL 1時間〜1日（種類による）

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
  - 自動的にダッシュボードへ遷移
  - 失敗メッセージ
    - メールアドレス不一致
    - 認証ID不一致
    - 認証ID expire済み

#### 5.1.2 管理画面

##### UI-3: ダッシュボード
- 必須要素:
  - 直近の配信予定（3件）
  - 新規作成ボタン
  - スケジュール一覧へのリンク
  - ユーザーメニュー（ログアウト）

##### UI-4: スケジュール一覧
- 必須要素:
  - 表示単位選択（リスト・週）
  - スケジュールカード
  - 1週間のタイムテーブル
  - 新規作成ボタン
  - ページネーション
  - 検索/フィルター（Phase 2で実装）

##### UI-5: スケジュール作成/編集
- 必須要素:
  - 入力フォーム
  - 保存/キャンセルボタン
  - バリデーションエラー表示
    - タイトル: 100文字(YouTubeの制約より)
    - description: 2500文字(YouTubeの制約より。海外では5000文字のところもあるようだが、現状2500均一)
  - 削除ボタン（編集時のみ）

#### 5.1.3 公開ページ

##### UI-6: 公開スケジュール
- 必須要素:
  - 表示単位選択（リスト・週）
  - 1週間のタイムテーブル(日曜開始)
  - スケジュールカード
    - 日付/時刻表示
    - プラットフォームアイコン
    - シェアボタン（SNS）
- OGPは今回は作成しない

### 5.2 APIインターフェース

#### 5.2.1 認証API

##### API-1: Magic Link送信
```
POST /api/auth/magic-link
Request: { email: string }
Response: { status: number, message?: string }

status:
  -1: internal error(messageに詳細を記載、emailサーバーが正しく応答しない、もこれ)
  0: success (email sent)
  1: rate limit over (too meny login request)
```

##### API-2: 認証確認
```
POST /api/auth/verify
Request: { auth_token: string }
Response: { status: number, message?: string, session_id: string }

status:
  -1: internal error(messageに詳細を記載)
  0: success (auth successfuly)
  1: rate limit over (too meny login request)
  2: expireed. (auth_token)
  3: auth_token/session_id does not match
```

##### API-3: ログアウト
```
POST /api/auth/logout
Response: { status: number, message?: string }

status:
  -1: internal error(messageに詳細を記載)
  0: success (logout successfuly)
  1: rate limit over (too meny API request)
  2: expireed (session_id)
  3: auth_token/session_id does not match
```

#### 5.2.2 スケジュールAPI

##### API-4: スケジュール作成
```
POST /api/streams
Request: Stream object(without stream_id)
Response: { status: number, message?: string, stream: Stream }

status:
  -1: internal error(messageに詳細を記載)
  0: success (create schedule successfuly)
  1: rate limit over (too meny API request)
  2: expireed (session_id)
  3: auth_token/session_id does not match
```

##### API-5: スケジュール取得
```
GET /api/streams?page=1&limit=20
Response: { status: number, message?: string, streams: Stream[], last_page: number }

pageは1オリジン。今の日時の30分前からはじめて、未来に向かってリストアップする。配信中も含む。過去のスケジュールはこの項目では表示できない。

status:
  -1: internal error(messageに詳細を記載)
  0: success (read schedule successfuly)
  1: rate limit over (too meny API request)
  2: expireed (session_id)
  3: auth_token/session_id does not match
  4: malformed parameters (there are no required fields or too meny fields)
  6: out_of_range(page and limit is too big)
```

```
GET /api/streams?year=2025&week=1
Response: { status: number, message?: string, streams: Stream[] }

weekは今年の何週目かを与える。1オリジン。日曜始まり。
1か月のデータはweekを4~5回呼んで取得

status:
  -1: internal error(messageに詳細を記載)
  0: success (read schedule successfuly)
  1: rate limit over (too meny API request)
  2: expireed (session_id)
  3: auth_token/session_id does not match
  4: malformed parameters (there are no required fields or too meny fields)
  6: out_of_range(year and week is out of range)
```

```
GET /api/streams/:id
Response: { status: number, message?: string, streams: Stream[], last_page: number }

status:
  -1: internal error(messageに詳細を記載)
  0: success (read schedule successfuly)
  1: rate limit over (too meny API request)
  2: expireed (session_id)
  3: auth_token/session_id does not match
  4: malformed parameters (there are no required fields or too meny fields)
  5: id does not found(stream_id)
```

##### API-6: スケジュール更新
```
PUT /api/streams/:id
Request: Stream object
Response: { status: number, message?: string }

status:
  -1: internal error(messageに詳細を記載)
  0: success (update schedule successfuly)
  1: rate limit over (too meny API request)
  2: expireed (session_id)
  3: auth_token/session_id does not match
  4: malformed parameters (there are no required fields or too meny fields)
  5: id does not found(stream_id)
```

##### API-7: スケジュール削除
```
DELETE /api/streams/:id
Response: { status: number, message?: string }

status:
  -1: internal error(messageに詳細を記載)
  0: success (delete schedule successfuly)
  1: rate limit over (too meny API request)
  2: expireed (session_id)
  3: auth_token/session_id does not match
  4: malformed parameters (there are no required fields or too meny fields)
  5: id does not found(stream_id)
```

##### API-8: 公開スケジュール取得
```
GET /api/public/streams?page=1&limit=20
Response: { status: number, message?: string, streams: PublicStream[], last_page: number }

pageは1オリジン。今の日時の30分前からはじめて、未来に向かってリストアップする。配信中も含む。過去のスケジュールはこの項目では表示できない。

status:
  -1: internal error(messageに詳細を記載)
  0: success (read schedule successfuly)
  1: rate limit over (too meny API request)
  4: malformed parameters (there are no required fields or too meny fields)
  6: out_of_range(page and limit is too big)
```

```
GET /api/public/streams?year=2025&week=1
Response: { status: number, message?: string, streams: PublicStream[] }

weekは今年の何週目かを与える。1オリジン。日曜始まり。
1か月のデータはweekを4~5回呼んで取得

status:
  -1: internal error(messageに詳細を記載)
  0: success (read schedule successfuly)
  1: rate limit over (too meny API request)
  4: malformed parameters (there are no required fields or too meny fields)
  6: out_of_range(year and week is out of range)
```

## 6. 実装優先順位

### 6.1 必須機能
1. 環境構築（Cloudflare、開発環境）
2. D1スキーマ作成
3. Magic Link認証基本実装
4. セッション管理

### 6.2 コア機能
1. スケジュールCRUD API
2. 管理画面基本UI
3. 公開ページ
4. レスポンシブ対応

## 7. 受け入れ基準

### 7.1 機能テスト
- [ ] Magic Link認証が正常に動作する
- [ ] セッションが3日間維持される
- [ ] アクセスすると、その時点から3日間セッションが維持される
- [ ] スケジュールの作成・編集・削除ができる
- [ ] 公開ページで配信予定が確認できる
- [ ] モバイル端末で正常に表示される

### 7.2 非機能テスト
- [ ] ページロード2秒以内
- [ ] API応答200ms以内
- [ ] 2同時接続で正常動作
- [ ] XSS/CSRF攻撃への耐性

### 7.3 ドキュメント
- [ ] APIドキュメント作成
- [ ] 基本的な使い方ガイド作成

## 8. リスクと対策

### 8.1 技術的リスク

| リスク | 影響度 | 発生確率 | 対策 |
|-------|--------|----------|------|
| メール送信API障害 | 高 | 低 | 対応しない |
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
