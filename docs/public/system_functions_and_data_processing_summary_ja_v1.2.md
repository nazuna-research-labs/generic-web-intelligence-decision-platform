# システム全機能・全層データ加工概要

```yaml
document_id: system_functions_and_data_processing_summary
document_type: summary
doc_ver: v1.2
language: ja
project_name: Generic Web Intelligence & Decision Platform
project_initiated_at: 2026-09-23 17:35:34 JST
created_at: 2026-10-03 08:14:10 JST
updated_at: 2026-10-05 22:57 JST
author: Nazuna Research
status: active
scope: summary of all functions and processing layers
note: 詳細版v1.2への入口としてMVP COMPLETE/current deltaを追加しつつ全Layer概要を保持
previous_note: 詳細版v1.1への入口として全Layer・3 Processing・Artifact/Lineage/Quality/Pipelineを欠落なく俯瞰できる概要版へ拡充
source_document: system_functions_and_data_processing_details_ja_v1.2.md
audience: all readers needing the whole-system map before deep dive
previous_public_version: system_functions_and_data_processing_summary_ja_v1.1.md
mvp_completion_state: COMPLETE
current_package: 0.12.4
public_privacy_review: PASS — public copy removes personal author identity and replaces known site-specific example identifiers with non-identifying reserved examples
```

## 0. MVP COMPLETE Update

MVP Coreは`0.12.4`で完成済み。

- `full` Live-fetch E2E: PASS
- Pipeline 6 profiles / resume / recovery / supplied / reused: CLOSED
- M5: PASS
- MVP COMPLETE: YES
- `.gitignore`: full audit PASS

MVP closureで追加確認された重要点:

- GA4 credential pathはrepository-root `credentials/`へ解決。
- GA4 CLIの`partial_success = exit 1`は維持し、Pipelineはmanifest statusまで確認して限定的にusable扱い。
- GSC relative dateは`America/Los_Angeles`基準でresolve。
- React relative delta ratioはpercent表示時に100倍する。`-0.3105 -> -31.05%`。
- R12 live Artifactを再取得せずM5の`full_from_extraction`へreuseし、A+C / B+C / determinism / missing / attribution / lineage / portability等を監査。

詳細実装差分は`system_functions_and_data_processing_details_ja_v1.2.md`を参照する。

## 1. この文書について

`system_functions_and_data_processing_details_ja_v1.2.md`の概要版。詳細版を置き換えない。

## 2. 全体フロー

```text
GA4 API -> GA4 Extraction -> GA4 Processing -> GA4 Analysis -> Visualization
                                \
                                 \-> Integration Extraction
GSC API -> GSC Extraction -> GSC Processing -> Integration Processing
                                \-> Integration Analysis -> Visualization
                                 \-> GSC Analysis -> Visualization
```

## 3. 各Layerを一言で

| Layer | 責務 |
|---|---|
| Source Extraction | API source valueを取得・保存 |
| Source Processing | Source固有の正規化責務を完了 |
| Source Analysis | Source単体の診断/比較 |
| Integration Extraction | 情報を落とさず安全に接続 |
| Integration Processing | Configで今回使うDatasetへ加工 |
| Integration Analysis | Generic Analysis result生成 |
| Visualization | Resultを再分析せず表示 |
| Pipeline | 既存Layer CLIのorchestration |

## 4. Processing 3種

### GA4 Processing

- date canonicalization
- page/URL normalization
- report grain保護
- event/key event整理
- GA4 Source lineage

### GSC Processing

- date/page normalization
- query/page grain分離
- search_type multiplicity保護
- CTRをderived ratioとして扱う
- positionをnon-additiveとして扱う
- reporting lag / coverage保持

### Integration Processing

- required/optional sources
- period policy
- grain
- dimensions
- metrics
- filters
- coverage
- safe aggregation semantics

Source Processingが「Sourceを正しくする」のに対し、Integration Processingは「今回使うDatasetを選び組み立てる」。

## 5. 重要な禁止事項

- NOT_OBSERVED / UNAVAILABLEを0へ潰さない
- GSC queryをGA4 conversionへexact attributionしない
- search_type/query row数だけGA4 factsを複製しない
- Visualizationで再集計しない
- absolute machine pathやsecretをManifestへ残さない

## 6. Pipeline

6 profilesを持ち、`full`はresume/recovery可能。認証を手動管理する場合は`full_from_extraction`。

## 7. 詳細を読む場所

- 全Layer加工: `system_functions_and_data_processing_details_ja_v1.2.md`
- 操作: `user_manual_ja_v1.2.md`
- Dashboard: `dashboard_manual_ja_v1.2.md`
- 全体契約: `system_requirements_ja_v3.1.md`


## 8. 全Layer責務マップ

| Layer | 入力 | 主責務 | 出力 | やらないこと |
|---|---|---|---|---|
| GA4 Extraction | Google APIs + config | Source Fact取得 | GA4 raw Artifact | URL business normalization / cross-source |
| GA4 Processing | GA4 Extraction | canonical date/page/grain/source semantics | GA4 Processing Artifact | GSC join |
| GA4 Analysis | GA4 Processing | Source-only analysis | GA4 Analysis Artifact | Integration前処理 |
| GSC Extraction | Search Analytics API | query/page/search_type fact取得 | GSC raw Artifact | missing=0 |
| GSC Processing | GSC Extraction | canonical page/date, CTR/position/coverage semantics | GSC Processing Artifact | GA4 attribution |
| GSC Analysis | GSC Processing | Source-only analysis | GSC Analysis Artifact | Integration前処理 |
| Integration Extraction | GA4/GSC Processing | lossless FULL OUTER相当統合 | Integrated raw/canonical Artifact | use-case trim |
| Integration Processing | Integration Extraction | period/grain/metric/filter/coverage dataset assembly | Analysis-ready dataset | Analysis judgment |
| Integration Analysis | Integration Processing | comparison/trend/ranking/contribution | Result Artifact | Source再加工 |
| Visualization | Result Artifact | presentation | dashboard Artifact | 再分析/再集計 |
| Pipeline | Atomic CLIs | orchestration/checkpoint/resume | Pipeline Manifest + final Artifact | Layer logic再実装 |

## 9. Processing 3種を混同しない

### 9.1 GA4 Processing

GA4 Source固有の加工を閉じる。

- date canonicalization
- page/url canonical fields
- report grain protection
- event/key-event semantics
- source manifest/lineage

### 9.2 GSC Processing

GSC Source固有の加工を閉じる。

- date/page canonicalization
- query/page grain separation
- search_type semantics
- clicks/impressions
- CTR derived ratio
- average position non-additivity
- requested/observed coverage

### 9.3 Integration Processing

複数Sourceを使う「今回のDataset」をConfigで組み立てる。

- period mode
- required/optional source
- grain
- dimensions
- metric registry
- safe aggregation
- filters
- coverage policy

## 10. Integration ExtractionとProcessingの決定的違い

```text
Extraction: 全部残して正しくつなぐ
Processing: 今回使う形へConfigで加工する
```

Extractionでcommon periodへtrimすると、後から別用途へ戻せないため禁止。

## 11. Metric Semantics要約

| Semantic | 代表例 | Aggregation |
|---|---|---|
| SUM_SAFE | clicks, impressions, selected counts | grain内でSUM可 |
| DERIVED_RATIO | CTR | numerator/denominatorから再導出 |
| NON_ADDITIVE | average position等 | 単純SUM不可 |
| WEIGHTED_WITHIN_GRAIN | weightが必要なaverage系 | contractに従う |
| NON_ADDITIVE_CONTEXT | queryに付いたGA4 page context | query横断SUM禁止 |

## 12. Missing Semantics要約

`ZERO != NOT_OBSERVED != UNAVAILABLE != UNKNOWN != NULL`。

特にGSC lagをZEROへ潰さない。

## 13. CoverageとQuality

Coverageは「観測できた範囲」。Qualityは「contract/整合/利用可能性の健全性」。

両方を見ることで「値が小さい」のか「データが届いていない」のかを分ける。

## 14. Query Attribution Boundary

GSC queryをGA4 session/conversion/key eventへexact attributionしない。

許されるのは、query -> GSC page -> canonical page -> aggregate GA4 page context。GA4 contextはnon-additive。

## 15. Artifact / Lineage

各Layerはimmutable run Artifactを作る。直接親のmanifest/hash/config/specを追跡し、結果からSourceまで戻れるようにする。

## 16. Pipeline 6 Profiles

- `full`
- `full_from_extraction`
- `sources`
- `ga4`
- `gsc`
- `integration`

`full`はSource Analysisを経由せず、Source ProcessingからIntegrationへ進む。

## 17. Resume / Recovery

成功済みStageは`reused`、手動回復Artifactは`supplied`として記録できる。認証失敗で最初から全取得し直す必要を減らす。

## 18. Visualization

VisualizationはGeneric Presentation Engine。GA4/GSC/Integration/Generic Result ArtifactをAdapterでInput Contractへ変換し、React primary / static secondaryで表示する。

## 19. 全体の安全原則

- Source Factを前段で削らない
- use-case責務をIntegration Processingへ置く
- AnalysisをSource固有branchだらけにしない
- Visualizationで再計算しない
- missingを0へ潰さない
- persistent absolute path/secretを残さない
- hash/config/specで再現性を持つ

## 20. 詳細版へ進む目安

次の質問に答えたい場合は詳細版を読む。

- GA4 URL normalizationの具体契約は？
- GSC CTR/positionをどう集約する？
- search_typeでCartesian growthをどう防ぐ？
- Integration Processingのperiod modeは何をする？
- どのmodule/function/testがその責務を実装している？
- Artifact filename/config key/CLI optionを実装と照合したい。

詳細版: `system_functions_and_data_processing_details_ja_v1.2.md`
