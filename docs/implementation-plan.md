# 実装計画 — Plant Memory Journal

> 元にした設計書: [design.md](./design.md)（§17 段階的実装計画、§23 最初の実行指示）
> 進み具合の管理: [todo.md](./todo.md) / 作業記録: [work_history.md](./work_history.md)

この計画は **Phase 0 と Phase 1（MVP）** を細かいタスクに分けたものです。Phase 2〜4 は大まかな流れだけを書き、着手するときに細かく分けます。

---

## 進め方

各タスクは次の順で進める。

1. コードを書く
2. テストを書いて動かす
3. 直す
4. `todo.md` と `work_history.md` を更新する

**各タスク共通の完了条件**

- Flutter: `flutter analyze` と `flutter test` が通る
- Functions: `npm run typecheck`、`npm run lint`、`npm test` が通る
- 本物の Gemini / Google Sheets を呼ばない（Fake Adapter を使う。本物は `RUN_REAL_AI_TESTS=true` のときだけ）
- SDK の API は公式ドキュメントか、インストールした型定義で確認してから書く
- 設計と違うことをするときは `docs/decisions/` に ADR を書く

**表の見方:** 「依存」は先に終わっている必要があるタスク、👤 は人間の作業が終わるまで待つタスク。

---

## Phase 0 — 土台づくり

### 0-A リポジトリと文書

| ID | タスク | 依存 | テスト・完了の基準 |
|---|---|---|---|
| P0-01 | Git リポジトリ、`.gitignore`（`.env`、秘密情報、ビルド生成物を除外） | — | `git status` に秘密ファイルが出てこない |
| P0-02 | `README.md`（概要、構成、セットアップの入口） | P0-01 | 手順へのリンクが切れていない |
| P0-03 | `CLAUDE.md`（設計書 §0 の絶対ルールを要約） | P0-01 | — |
| P0-04 | `docs/manual-setup.md`（§19 の人間作業をチェックリスト化） | P0-01 | §19 の全項目が入っている |
| P0-05 | `docs/decisions/0001-record-architecture-decisions.md`（ADR のひな形） | P0-01 | — |

### 0-B Flutter Web の基盤（`app/`）

| ID | タスク | 依存 | テスト・完了の基準 |
|---|---|---|---|
| P0-10 | `flutter create --platforms=web app` | P0-01 | `flutter analyze` が通る |
| P0-11 | 依存パッケージの追加（§15.1）とバージョン固定、`pubspec.lock` を commit | P0-10 | `flutter pub get` が通る |
| P0-12 | `core/config/`: 環境設定（`USE_FIREBASE_EMULATORS` など）の読み込み | P0-10 | 設定値の単体テスト |
| P0-13 | `core/errors/`: アプリ共通のエラー型（認証なし・権限なし・回数制限・タイムアウトなど） | P0-10 | エラー対応表の単体テスト |
| P0-14 | `core/firebase/`: Firebase の初期化と Emulator への接続切り替え | P0-11, P0-12, 👤 Project ID | 切り替えロジックの単体テスト（Firebase はモック） |
| P0-15 | `core/router/`: `go_router` で `/login`・`/diaries` と、ログイン状態によるリダイレクト | P0-11 | 未ログインなら `/login` に行く Widget テスト |
| P0-16 | `features/auth/domain`: `AuthUser`、`AuthRepository` インターフェース | P0-10 | Domain の単体テスト |
| P0-17 | `features/auth/data`: `FirebaseAuthRepository` と `FakeAuthRepository` | P0-16 | Fake を使ったテスト |
| P0-18 | `features/auth/application`: ログイン状態の Provider / `AsyncNotifier` | P0-17 | Fake Repository で状態が変わる Riverpod テスト |
| P0-19 | `features/auth/presentation`: ログイン画面（説明文とログインボタン） | P0-15, P0-18 | loading / error 表示の Widget テスト |
| P0-20 | PWA の設定（manifest、アイコン、theme color） | P0-10 | `flutter build web` が通る |

### 0-C Functions の基盤（`functions/`）

| ID | タスク | 依存 | テスト・完了の基準 |
|---|---|---|---|
| P0-30 | `firebase init functions firestore storage hosting emulators` | 👤 Project ID | `firebase emulators:start` が起動する |
| P0-31 | TypeScript の strict 設定、ESLint、テストランナー（Vitest）、npm scripts | P0-30 | `typecheck` / `lint` / `test` が通る |
| P0-32 | `config/`: サーバー設定（モデル名、上限値）を一か所に | P0-31 | 単体テスト |
| P0-33 | `domain/errors.ts`: ドメインエラーの型 | P0-31 | 単体テスト |
| P0-34 | `middleware/`: `withAuth` / `withValidatedInput` / `withLogging` / エラー変換（高階関数） | P0-33 | 認証なし→`unauthenticated`、不正入力→`invalid-argument` のテスト |
| P0-35 | `adapters/ai/`: `AiAdapter` インターフェースと `FakeAiAdapter` | P0-31 | Fake が決まった値を返すテスト |
| P0-36 | 公開の入口 `index.ts`（中身は持たず、各 Controller を export するだけ） | P0-34 | — |

### 0-D セキュリティルール

| ID | タスク | 依存 | テスト・完了の基準 |
|---|---|---|---|
| P0-40 | `firestore.rules` 初版（`users/{uid}/**` は本人だけ） | P0-30 | Rules テスト: 他人の diary の読み書きを拒否 |
| P0-41 | `storage.rules` 初版（`user-uploads/{uid}/**` は本人だけ、型と容量の制限） | P0-30 | Rules テスト: 他人のパス・大きすぎるファイル・画像以外を拒否 |
| P0-42 | Emulator を使った Rules テストを npm script にする | P0-40, P0-41 | `npm run test:rules` が通る |

**Phase 0 の Done:** `flutter run -d chrome` でログイン画面が表示され、Emulator でテストが動く。

---

## Phase 1 — 植物観察日記 MVP

### 1-A 認証

| ID | タスク | 依存 | テスト・完了の基準 |
|---|---|---|---|
| P1-01 | Google ログイン / ログアウト（Web 用のポップアップまたはリダイレクト） | P0-19, 👤 Google Provider | Auth Emulator でログイン→`/diaries` に移る |
| P1-02 | App Check（Web: reCAPTCHA Enterprise、ローカルは Debug Provider） | P0-14, 👤 App Check 登録 | Callable が App Check なしを拒否する |

### 1-B 日記のドメインとサーバー API

| ID | タスク | 依存 | テスト・完了の基準 |
|---|---|---|---|
| P1-10 | `functions/src/domain/`: `Diary`、`EmotionLabel`、`PlantCandidate`、`GeoPointValue` | P0-31 | 単体テスト |
| P1-11 | `schemas/diary.schema.ts`: create / update / list / get / delete の Zod スキーマ | P1-10 | 正常・異常、範囲外、文字数、未知フィールドを拒否するテスト |
| P1-12 | `repositories/diary.repository.ts`: インターフェースと Firestore 実装 | P1-10 | Firestore Emulator への読み書きテスト |
| P1-13 | `services/diary.service.ts`: 所有者の確認、写真パスが `user-uploads/{uid}/` 配下かの確認、保存 | P1-11, P1-12 | 他人のパスを拒否するテスト、`clientRequestId` で二重作成しないテスト |
| P1-14 | `controllers/diary.controller.ts`: `createDiary` / `listDiaries` / `getDiary` / `updateDiary` / `deleteDiary` | P1-13, P0-34 | 認証なし・他人の ID を拒否するテスト |
| P1-15 | 日記の削除: Firestore と Storage の両方を消す | P1-14 | 削除後に両方とも残らないテスト |

### 1-C AI（植物候補と感情フィードバック）

| ID | タスク | 依存 | テスト・完了の基準 |
|---|---|---|---|
| P1-20 | Genkit の導入（`@genkit-ai/google-genai`）と、`AiAdapter` の Genkit 実装 | P0-35, 👤 API キーを Secret Manager に登録 | 型定義で API を確認した上で、Fake で Flow が動く |
| P1-21 | `prompts/identify-plant.prompt` と `analyzePlant` Flow（「不明」も許す JSON 出力） | P1-20 | Fake Model: 正常、スキーマ違い、失敗時のテスト |
| P1-22 | `prompts/initial-feedback.prompt` と `generateInitialFeedback` Flow | P1-20, P1-14 | Fake Model で feedback と `aiSuggestedEmotion` が保存されるテスト |
| P1-23 | 回数制限（ユーザーごとの一定時間内の上限） | P0-32 | 上限を超えると `resource-exhausted` になるテスト |
| P1-24 | Flutter 側: `firebase_ai` を Provider で管理（植物候補の一時表示用、結果は確定値として保存しない） | P0-14 | Provider の差し替えテスト |

### 1-D Flutter の日記機能（`features/diary/`）

| ID | タスク | 依存 | テスト・完了の基準 |
|---|---|---|---|
| P1-30 | `domain/`: Diary・Emotion のモデル | P0-10 | モデル検証の単体テスト |
| P1-31 | `data/`: Callable を呼ぶ `DiaryRepository` と `FakeDiaryRepository` | P1-30, P1-14 | Fake を使ったテスト |
| P1-32 | 写真の撮影・選択（`image_picker`）、クライアントでの圧縮、Storage へのアップロード | P0-41 | 写真なし・圧縮後のサイズのテスト |
| P1-33 | GPS（`geolocator`）: 許可 / 拒否 / 概略位置モード | P1-30 | 拒否しても日記を作れる Widget テスト |
| P1-34 | `application/`: 日記作成の `AsyncNotifier`（撮影→植物候補→感情→AI 返答→保存） | P1-31, P1-32, P1-33 | Fake で状態が順に変わる Riverpod テスト |
| P1-35 | `presentation/`: 日記作成の画面（loading / data / error、中断・再試行） | P1-34 | 各状態の Widget テスト、植物が「不明」のときの表示 |
| P1-36 | `presentation/`: 日記の一覧・詳細 | P1-31 | 一覧と詳細の Widget テスト |
| P1-37 | 写真・感情の削除の導線 | P1-15, P1-36 | 削除後に一覧から消える Widget テスト |

### 1-E 仕上げと確認

| ID | タスク | 依存 | テスト・完了の基準 |
|---|---|---|---|
| P1-40 | Integration テスト（Emulator + Fake AI）: ログイン→日記保存→一覧 | P1-35, P1-36 | `integration_test` が通る |
| P1-41 | ログに個人情報の生データ（写真、GPS の生値、感情の原文）を出していないかの確認 | P1-22 | ログ出力のテスト |
| P1-42 | CI（GitHub Actions）: §16.3 のゲート | P0-31, P0-11 | CI が通る |
| P1-43 | 👤 iPhone Safari で PWA の E2E 確認（ホーム画面に追加、カメラ、位置、ログイン） | P1-40, Hosting へのデプロイ | 手動チェックリストで確認 |
| P1-44 | README とセットアップ手順の更新 | P1-43 | — |

**Phase 1 の Done:** ログイン→撮影→植物候補→感情→AI の返答→Firestore 保存→日記一覧が一通り動く。

---

## Phase 2〜4（大まかな流れ）

- **Phase 2 — マルチターンチャット:** ChatMessage / ChatSession のドメイン → `continueChat` / `completeChat` の API → ストリーミングを SDK で実証（無理なら非ストリーミングにする）→ Flutter でチャンク連結とスクロール追従
- **Phase 3 — 思い出 RAG / Google Sheets:** 要約と embedding（768次元）→ Vector Index と Retriever（必ず本人の UID だけ）→ 再ランキング → 思い出カード → Sheets の OAuth と冪等な upsert
- **Phase 4 — 拡張 / 運用:** Search Grounding、Remote Config、Telemetry、プロンプト評価、Tool の承認フロー

MVP では RAG・Google Sheets・Search Grounding・A2A は実装しない。必要ならインターフェースだけ用意する。

---

## 人間の作業を待つところ

次のタスクは 👤 の作業が終わるまで始められない。詳細は `docs/manual-setup.md` に書く。

| 待っている作業 | 止まるタスク |
|---|---|
| Firebase Project ID の決定・作成 | P0-14、P0-30 以降 |
| Google Provider の有効化 | P1-01 |
| App Check（reCAPTCHA Enterprise）の登録 | P1-02 |
| Gemini API キーを Secret Manager に登録 | P1-20 以降で本物の AI を使うとき（Fake なら先に進められる） |
| 課金・予算の設定 | 本番デプロイ、P1-43 |
