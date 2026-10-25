# TODO — Plant Memory Journal

> 元にした設計書: [design.md](./design.md)（§17 段階的実装計画 / §19 人間による設定）
> 凡例: `[ ]` 未着手 / `[~]` 作業中 / `[x]` 完了 / 👤 人間がコンソールで行う作業

---

## Phase 0 — 土台づくり

- [ ] Gitリポジトリ、`CLAUDE.md`、`README.md`を作る
- [ ] `docs/implementation-plan.md`（細かいタスク・依存関係・テスト基準）を作る
- [ ] Flutter Web、Functions（TypeScript）、Firebase の初期化
- [ ] Riverpod、Router、Firebase init の共通基盤
- [ ] Emulator 接続と Fake Adapter の切り替え
- [ ] 認証 / Routing の最低限の Widget テスト
- [ ] Firebase Security Rules の初版と Rules テスト
- [ ] `docs/manual-setup.md` に人間が行う設定を書き出す

**Done:** `flutter run -d chrome` でログイン画面が出て、Emulator でテストが動く。

## Phase 1 — 植物観察日記 MVP

- [ ] Google Authentication でログイン / ログアウト
- [ ] 写真の撮影・選択、クライアントでの圧縮、Storage へのアップロード
- [ ] GPS の許可 / 拒否と概略位置モード
- [ ] Firebase AI Logic を Riverpod Provider で管理
- [ ] 写真から植物候補を JSON で返す（「不明」も許す）
- [ ] 日記の Domain / Repository / Zod / Callable CRUD
- [ ] 感情フィードバックの Genkit Flow と Dotprompt
- [ ] AI 処理の AsyncValue（loading / data / error）
- [ ] 日記の一覧・詳細
- [ ] 画像・感情の削除、セキュリティテスト
- [ ] iPhone Safari で PWA の E2E 確認

**Done:** ログイン→撮影→植物候補→感情→AIの返答→Firestore保存→日記一覧が一通り動く。

## Phase 2 — マルチターンチャット

- [ ] ChatMessage / ChatState / ChatSession
- [ ] `continueChatFlow` / サーバー側の履歴管理
- [ ] `onCallGenkit` などでストリーミングを SDK で実証
- [ ] Flutter `AsyncNotifier` でのチャンク連結
- [ ] スクロール追従、二重送信防止、再試行
- [ ] セッションの終了・中断・再開の方針
- [ ] 固定 Fixture による履歴の整合性テスト

**Done:** 過去のやり取りを踏まえて複数ターン続けられ、閉じた後も履歴が再表示できる。

## Phase 3 — 思い出 RAG / Google Sheets

- [ ] 日記の要約 / embedding 生成 / 768次元ベクトルの格納
- [ ] Firestore Vector Index と Retriever
- [ ] 感情・季節・位置での再ランキング
- [ ] 思い出カードと根拠付きの回答
- [ ] Google Sheets OAuth / 年別ファイル・月別タブ
- [ ] idempotent な upsert / 再試行キュー
- [ ] RAG でユーザー間の混入がゼロであることのテスト

**Done:** 過去の似た体験を引用して提案でき、希望すれば月別 Sheet に同期できる。

## Phase 4 — 拡張 / 運用

- [ ] Google Search Grounding（出典表示・プライバシー）
- [ ] Remote Config / Server Prompt Templates の段階利用
- [ ] Genkit Firebase Telemetry と費用ダッシュボード
- [ ] プロンプト評価、A/B、費用とレイテンシの最適化
- [ ] Genkit Tool の選択と承認フロー
- [ ] Gemini CLI のマルチエージェント開発ワークフロー
- [ ] A2A 対応の外部 Agent 連携（必要性が確認できた場合のみ）

---

## 👤 人間が行う設定（§19）

- [ ] Firebase Project ID の決定・作成
- [ ] Firebase Authentication で Google Provider を有効化
- [ ] Web アプリ登録・Authorized Domains の設定
- [ ] Firestore Database の作成とリージョン選択
- [ ] Cloud Storage バケットの作成とリージョン選択
- [ ] Functions / Hosting の利用プラン・課金 / 予算の設定
- [ ] Firebase AI Logic で Gemini Developer API の利用設定
- [ ] Google AI Studio で API 課金 Tier / API キーの作成と権限確認
- [ ] Secret Manager API の有効化と `GOOGLE_GENAI_API_KEY` の登録
- [ ] App Check の Web 用 reCAPTCHA Enterprise 登録
- [ ] Remote Config のデフォルト値作成
- [ ] Google Sheets API の有効化、OAuth 同意画面、OAuth Client の設定（Phase 3）
- [ ] Google Sheets のスコープとユーザーデータの扱いを確認（Phase 3）
- [ ] 独自ドメインの設定と Google ログインの実機検証（必要なとき）
- [ ] 本番の監視ポリシー、個人情報の保持期間、削除手順の承認

---

## メモ・課題

- [ ] `design.md` §14 のディレクトリ構成では設計書がリポジトリ直下にあるが、実際は `docs/design.md` に置いている。どちらに合わせるか決める
