# construction-estimation-system

建築図面 PDF → 自動数量拾い出し・積算書生成システム

## プロジェクト概要

既存の `raster-to-cad` プロジェクト（PDF→DXF 変換済み）を拡張し、AI セマンティック分類 + 公的積算資料マスタを活用して、図面から自動的に数量拾い出し・積算書を生成するエンドツーエンドシステム。

## ディレクトリ構造

```
construction-estimation-system/
├── CLAUDE.md                          ← このファイル（Claude Code コンテキスト）
├── requirements.txt                   ← 全エージェント共通依存
├── README.md                          ← プロジェクト説明
├── agents/
│   ├── A_pdf_loader/        # PDF 取込・ベクター化（raster-to-cad L1 拡張）
│   ├── B_classifier/        # 図面種別・部材分類（Vision LLM）
│   ├── C_extractor/         # 数量拾い出し（Shapely）
│   ├── D_estimator/         # 単価計算（SQLite マスタ）
│   └── E_reporter/          # Excel/DXF 出力
├── schemas/
│   └── DrawingDocument.schema.json    ← Agent A 出力フォーマット（定義済み）
├── mocks/
│   └── （テスト用 JSON データ）
└── tests/
    └── （各エージェント単体テスト）
```

## エージェント仕様

| # | 名称 | 責務 | 入力 | 出力 | 状態 |
|----|------|------|------|------|------|
| A | PDF Loader | PDF 取込・ベクター化 | PDF file | DrawingDocument.json | 🔵 未着手 |
| B | Classifier | 図面種別・部材分類 | DrawingDocument.json | ClassifiedDrawing.json | 🔵 未着手 |
| C | Extractor | 数量拾い出し | ClassifiedDrawing.json | QuantitySheet.json | 🔵 未着手 |
| D | Estimator | 単価計算・積算 | QuantitySheet.json | CostEstimate.json | 🔵 未着手 |
| E | Reporter | Excel/DXF 出力 | CostEstimate.json | Excel, DXF | 🔵 未着手 |

## エージェント間データフロー

```
PDF file
  ↓
[Agent A] PDF Loader
  ↓ (DrawingDocument.json)
[Agent B] Classifier
  ↓ (ClassifiedDrawing.json)
[Agent C] Extractor
  ↓ (QuantitySheet.json)
[Agent D] Estimator
  ↓ (CostEstimate.json)
[Agent E] Reporter
  ↓
Excel / DXF
```

## JSON Schema（インターフェース定義）

### DrawingDocument.schema.json

Agent A の出力フォーマット。詳細は `schemas/DrawingDocument.schema.json` を参照。

**⚠️ 重要**: `drawing_type` は実装フェーズ 2 で観察後に enum を確定する（事前定義しない）。

## 既存コード流用

### raster-to-cad
- **パス**: `/Users/in/projects/raster-to-cad/scripts/centerline_to_dxf.py`
- **用途**: Agent A の L1 ベクター化エンジン
- **状態**: R12 JWCAD 互換性確認済み

### JWW Parser
- **パス**: `/Users/in/jw-cad-mac/` (Phase 0 完成)
- **用途**: Agent A に JWW 直接入力対応を追加（Phase 4 予定）

## 実装ロードマップ

### Phase 1: 初期化（完了）
- [x] GitHub リポジトリ作成
- [x] CLAUDE.md 配置
- [ ] ディレクトリ構造初期化
- [ ] DrawingDocument.schema.json 確定（drawing_type: open）
- [ ] requirements.txt スケルトン

### Phase 2: Agent A 実装 & 観察
- [ ] centerline_to_dxf.py を agents/A_pdf_loader/ に統合
- [ ] kyouwachou/a/ から 3-5 枚をテスト処理
- [ ] 出現した drawing_type を全て記録
- [ ] DrawingDocument.json 出力確認
- [ ] 単体テスト作成

### Phase 3: Schema 確定
- [ ] 観察結果から drawing_type enum を確定
- [ ] DrawingDocument.schema.json を固定
- [ ] 残り 10 枚を一括処理でバリデーション

### Phase 4+: Agent B-E 実装
- [ ] Agent B: Vision LLM 分類ロジック
- [ ] Agent C: 数量計算エンジン（Shapely）
- [ ] Agent D: 単価マスタ + 積算エンジン
- [ ] Agent E: Excel/DXF 出力

## 開発ルール

### Schema 設計
- ❌ Enum を推測値で事前定義しない
- ✅ 実装・観察から逆算して確定
- ✅ Schema 確定後は互換性を重視

### エージェント独立性
- ✅ 各エージェントは入力 JSON のみに依存
- ✅ 上流エージェント未完成でも Mock で単体テスト可能
- ❌ エージェント間の処理ロジック混在

### テスト戦略
- ✅ 実装フェーズ 2 で kyouwachou サンプルで検証
- ✅ Phase 3 以降は全 15 枚で統合テスト
- ❌ Mock なし初期状態での開発開始

## 禁止事項（このプロジェクト固有）

- ❌ Schema 未確定で enum 固定
- ❌ Agent 間での責務混在
- ❌ Mock データなし開発（テスト効率悪化）
- ❌ drawing_type を推測値で実装開始

## 参考資料

- 技術設計書: `/Users/in/.claude/projects/-Users-in/memory/construction-estimation-system.md`
- 既存成果: `/Users/in/projects/raster-to-cad/` (centerline 方式, 8,448 entities)
- メモリシステム: LLMLogger FTS5 全文検索
