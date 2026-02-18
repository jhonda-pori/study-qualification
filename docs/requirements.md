# 資格クイズアプリ テンプレート - 要件定義書

## 1. プロジェクト概要

| 項目 | 内容 |
|------|------|
| 本リポジトリの役割 | **どんな資格にも対応できるクイズアプリのテンプレート** |
| プラットフォーム | Flutter（Android優先、iOS対応予定） |
| 収益モデル | 無料 + 広告（AdMob） |
| 運用方式 | 1資格 = 1リポジトリ（テンプレートをcloneして設定差し替え） |

## 2. テンプレート構成

### 2.1 新しい資格アプリを作る手順

```
1. このリポジトリをclone
2. app_config.json を編集（資格名・分野・パッケージID等）
3. assets/questions.json を配置（問題データ）
4. アイコン・スプラッシュ画像を差し替え
5. flutter build → 完成！
```

### 2.2 資格ごとに差し替えるファイル（これだけ変える）

```
assets/
├── config/
│   └── app_config.json    ← 資格名・分野定義・アプリ設定
├── questions/
│   └── questions.json     ← 問題データ（Claude APIで事前生成）
└── images/
    ├── icon.png           ← アプリアイコン
    └── splash.png         ← スプラッシュ画像
```

### 2.3 app_config.json の構造

```json
{
  "app_name": "宅建士 一問一答",
  "qualification_name": "宅地建物取引士",
  "package_id": "com.example.takken_quiz",
  "theme_color": "#1976D2",
  "categories": [
    {
      "id": "rights",
      "name": "権利関係",
      "description": "民法、借地借家法、不動産登記法、区分所有法",
      "icon": "gavel"
    },
    {
      "id": "business_law",
      "name": "宅建業法",
      "description": "宅建業法全般、37条書面、35条書面",
      "icon": "business"
    },
    {
      "id": "regulations",
      "name": "法令上の制限",
      "description": "都市計画法、建築基準法、国土利用計画法",
      "icon": "account_balance"
    },
    {
      "id": "tax_other",
      "name": "税・その他",
      "description": "不動産取得税、固定資産税、鑑定評価、統計",
      "icon": "payments"
    }
  ],
  "test_question_counts": [10, 20, 30, 50],
  "ad_banner_enabled": true,
  "ad_interstitial_on_test_complete": true
}
```

### 2.4 questions.json の構造

```json
{
  "version": "1.0.0",
  "qualification": "宅地建物取引士",
  "total_count": 500,
  "questions": [
    {
      "id": "q001",
      "category_id": "rights",
      "subcategory": "民法（総則）",
      "question": "問題文...",
      "choices": [
        "選択肢1",
        "選択肢2",
        "選択肢3",
        "選択肢4"
      ],
      "correct_index": 0,
      "explanation": "解説文..."
    }
  ]
}
```

## 3. テンプレートが提供する共通機能

### 3.1 一問一答モード
- 分野を選択して学習できる（分野はconfigから動的生成）
- 1問ずつ出題 → 回答 → 正誤表示 → 解説表示
- 次の問題に進む / 前の問題に戻る
- 間違えた問題にフラグを自動付与
- 終了タイミングは自由（いつでもやめられる）

### 3.2 テストモード
- 出題数を選択（configで定義: デフォルト 10/20/30/50 問）
- 全分野からランダムに出題（分野指定も可能）
- 全問回答後に採点・結果画面を表示
- 結果画面: 点数、正答率、分野別正答率
- テスト完了時にインタースティシャル広告を表示

### 3.3 復習機能
- 間違えた問題一覧を確認できる
- フラグ付き問題だけで一問一答できる
- 手動でフラグの付け外しが可能

### 3.4 学習記録・進捗管理
- テスト履歴（日時、点数、正答率）の保存・一覧表示
- 分野別の正答率表示（苦手分野の可視化）
- 全体の学習進捗（解いた問題数 / 全問題数）

### 3.5 広告
- 画面下部にバナー広告（常時表示、configでON/OFF）
- テストモード完了時にインタースティシャル広告（configでON/OFF）

### 3.6 問題形式
- 4択問題のみ
- 各問題に解説付き

## 4. 技術スタック

| レイヤー | 技術 |
|----------|------|
| フレームワーク | Flutter (Dart) |
| ローカルDB | SQLite (sqflite) |
| 問題データ | JSON（アセットとしてバンドル） |
| 状態管理 | Riverpod |
| 広告 | Google AdMob (google_mobile_ads) |
| UI | Material Design 3 |

## 5. データベース設計（ローカルSQLite）

### テーブル: quiz_results（テスト結果）
| カラム | 型 | 説明 |
|--------|-----|------|
| id | INTEGER PK | 自動採番 |
| mode | TEXT | テストモード種別 |
| total_questions | INTEGER | 出題数 |
| correct_count | INTEGER | 正答数 |
| category_filter | TEXT | 分野フィルター（null=全分野） |
| created_at | TEXT | 実施日時 |

### テーブル: question_flags（問題フラグ）
| カラム | 型 | 説明 |
|--------|-----|------|
| question_id | TEXT PK | 問題ID |
| is_flagged | INTEGER | フラグ状態（1=ON） |
| wrong_count | INTEGER | 間違えた回数 |
| correct_count | INTEGER | 正解した回数 |
| last_answered_at | TEXT | 最終回答日時 |

## 6. 画面構成

```
ホーム画面（資格名・進捗サマリー表示）
├── 一問一答モード
│   ├── 分野選択画面（configから動的生成）
│   └── 出題画面（問題→回答→解説）
├── テストモード
│   ├── 設定画面（問題数・分野選択）
│   ├── 出題画面（問題→回答→次へ）
│   └── 結果画面（スコア・分野別正答率）
├── 復習（間違えた問題）
│   └── フラグ付き問題一覧 → 出題画面
├── 学習記録
│   ├── テスト履歴一覧
│   └── 分野別正答率
└── 設定
    └── （将来拡張用）
```

## 7. アーキテクチャ（ディレクトリ構成案）

```
lib/
├── main.dart
├── app.dart
├── config/
│   └── app_config.dart          ← app_config.json を読み込むモデル
├── models/
│   ├── question.dart            ← 問題データモデル
│   ├── quiz_result.dart         ← テスト結果モデル
│   └── question_flag.dart       ← フラグモデル
├── providers/
│   ├── config_provider.dart     ← アプリ設定の状態管理
│   ├── question_provider.dart   ← 問題データの状態管理
│   ├── quiz_provider.dart       ← クイズ進行の状態管理
│   └── stats_provider.dart      ← 学習記録の状態管理
├── database/
│   └── database_helper.dart     ← SQLite操作
├── screens/
│   ├── home_screen.dart
│   ├── practice/
│   │   ├── category_select_screen.dart
│   │   └── practice_screen.dart
│   ├── test/
│   │   ├── test_setup_screen.dart
│   │   ├── test_screen.dart
│   │   └── test_result_screen.dart
│   ├── review/
│   │   └── review_screen.dart
│   └── stats/
│       └── stats_screen.dart
├── widgets/
│   ├── question_card.dart
│   ├── choice_button.dart
│   ├── explanation_card.dart
│   ├── banner_ad_widget.dart
│   └── category_chip.dart
└── utils/
    └── ad_helper.dart           ← AdMob ID管理
```

## 8. デプロイ

| 項目 | 内容 |
|------|------|
| Phase 1 | Google Play リリース |
| Phase 2 | App Store リリース |
| 最低対応OS | Android 6.0+ / iOS 14+ |

## 9. 対応予定の資格（例）

| 資格 | リポジトリ | ステータス |
|------|-----------|-----------|
| 宅地建物取引士 | study-takken | 最初に実装 |
| （今後追加） | - | - |

## 10. 将来の拡張案（スコープ外）
- ダークモード対応
- 問題データのオンライン更新（サーバーから差分取得）
- リアルタイム生成モード（Claude API連携）
- ランキング機能
- 学習リマインダー通知
- 問題の難易度設定
- 問題生成用のCLIツール（Claude APIで自動生成）
