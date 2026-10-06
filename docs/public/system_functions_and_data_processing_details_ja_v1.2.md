# システム全機能・全層データ加工詳細

```yaml
document_id: system_functions_and_data_processing_details
document_type: reference
doc_ver: v1.2
language: ja
project_name: Generic Web Intelligence & Decision Platform
project_initiated_at: 2026-09-23 17:35:34 JST
created_at: 2026-10-03 08:14:10 JST
updated_at: 2026-10-05 22:57 JST
author: Nazuna Research
status: active
scope: all current functions and layer-by-layer data transformation details
note: 既存v1.0内容を保持しつつ、全Layer責務、3 Processingの手順、implementation/config/test/CLI inventoryを追加した深掘りリファレンス
audience: developers / reviewers / LLMs / advanced operators
responsibility: complete functional and transformation reference; implementation-derived inventory included
previous_public_version: system_functions_and_data_processing_details_ja_v1.1.md
mvp_completion_state: COMPLETE
current_package: 0.12.4
public_privacy_review: PASS — public copy removes personal author identity and replaces known site-specific example identifiers with non-identifying reserved examples
```

## 0. MVP COMPLETE Implementation Delta — v0.12.4

本書のImplementation Inventoryは2026-10-03時点の自動抽出snapshotを大量に保持しているため、そのsnapshot自体は削除・再解釈しない。以下はMVP completionまでに追加された差分であり、current stateを判断するときはこの節を既存snapshotより優先する。

### 0.1 Current package / completion

- package: `0.12.4`
- Pipeline 6 profiles: CLOSED
- R12 `full` Live-fetch E2E: PASS
- M5 Hardening / Completion Gate: PASS
- MVP COMPLETE: YES
- `.gitignore` full audit: PASS

### 0.2 GA4 credential path correction

構造整理後の`config/user/sources/ga4/profiles.yaml` / exampleで、旧配置由来の`../credentials/...`が`config/user/sources/credentials/`を指す不整合をR12 preflightで検出した。

Current contractではGA4 default profileもrepository-root `credentials/`へ解決する。credential fileの本文や実identifierはArtifact/docsへ保存しない。

### 0.3 Pipeline GA4 partial-success adapter

GA4 Extraction CLIの既存contractは維持する。

```text
success / success_with_warnings -> exit 0
partial_success                 -> exit 1
```

Pipeline v0.12.3以降はGA4 Extractionだけ、child exitが非0でも次をすべて満たす場合にusable stageとして受理する。

1. 新規GA4 Artifactが正確に1個生成された。
2. Artifact manifestを読める。
3. manifest statusが正確に`partial_success`。

`failed`、manifest missing、複数/0 Artifact、invalid statusは引き続きPipeline failure。

R12実データでは`31 exported / 1 skipped / 0 failed`で`partial_success`となり、`organic_google_search`が`incompatible_dimensions_metrics`でskipされた。これはmissingを成功に偽装する修正ではなく、既存GA4 CLI status contractをPipeline orchestrationが正しく理解するためのadapterである。

### 0.4 R12 full Live E2E

v0.12.3で`500daysAgo -> yesterday`をGA4/GSCへ指定して`full`を実行し、8 stagesを完走した。

- GSC resolved coverage: `2025-05-22 -> 2026-10-03` / 500 days
- Pipeline manifest: PASS
- child Artifact existence/hash lineage: PASS
- requested range contract: PASS
- portability / secret marker scan: PASS
- Visualization final Artifact/contract/hash: PASS
- generated Artifact CSV parse: PASS / 223 files

GSC relative dateは`America/Los_Angeles`のcalendarでresolveする。GA4 relative expressionはGA4 Data APIへexpressionとして渡す。したがって両Sourceの`yesterday`を同じローカルclock前提で解釈しない。

### 0.5 Visualization v0.12.4 percentage correctness

React primary comparison tableの`relative_delta`はratio。旧表示ではratio値にそのまま`%`を付ける経路があり、`-0.3105`を`-0.3105%`と表示し得た。

v0.12.4ではratio formatterを通し、`-0.3105 -> -31.05%`とする。Static audit側と意味を一致させ、regression testを追加した。

### 0.6 M5 validation / determinism / semantic gate

M5はR12のlive Extraction Artifactを再取得せず、v0.12.4 `full_from_extraction`でProcessing以降を再生成した。

監査対象:

- full regression / lint / format / compile
- Manifest / row count / hashes / lineage / schema/config/spec hash
- private absolute path / secret marker
- A+C long-history fixture
- B+C same-requested-period control
- duplicate grain / Cartesian growth
- missing semantics
- query attribution boundary / contextual-only non-additive values
- CTR / average position等non-additive metric semantics
- deterministic repeated-run pair
- Visualization contract / percentage formatting
- Artifact layout / original-input ZIP / portability

`ga4_process`はpackage `tool_version`ではなく独立`processor_version`を持つため、provenance監査はstage-specific fieldを用いる。M5中にこの差を無視したaudit-side誤判定が一度発生したが、product Artifactは正常であり、auditを修正して同一成功runを再利用し全gateをPASSした。

### 0.7 Current test delta

v0.12.2時点の222 testsから、GA4 credential path regression、Pipeline partial-success adapter regression群、Visualization percentage regressionが追加された。MVP completion時はfull regression PASSをCompletion Gateの事実とし、個々の自動抽出Test Inventory節は2026-10-03 snapshotとして読む。将来Inventoryを再生成するときはcurrent repoから再抽出する。

## 1. 文書の目的

本書は、このシステムの全主要機能と、データが各Layerでどのように変換されるかを技術的に追跡するreferenceである。

特に3種類のProcessing、すなわちGA4 Processing / GSC Processing / Integration Processingを区別して説明する。

## 2. 全体Data Flow

```text
Google APIs
  │
  ├─ GA4 Extraction -> GA4 Processing -> GA4 Analysis -> Visualization
  │                         \
  │                          \-> Integration Extraction
  │                               -> Integration Processing
  └─ GSC Extraction -> GSC Processing -> Integration Analysis -> Visualization
                              \
                               \-> GSC Analysis -> Visualization
```

各Layerは前Layerを上書きせず、immutable run Artifactを新規生成する。

## 3. 共通Artifact原則

### 3.1 Immutable run

同じrun directoryを後段都合で書き換えない。

### 3.2 Direct-parent lineage

各Artifactは直接の親Artifact、manifest hash、config/spec hashを追跡できるようにする。

### 3.3 Portable path

Persistent metadataへ`/Users/...`等のmachine-specific absolute pathを保存しない。

### 3.4 Missing semantics

`ZERO / NOT_OBSERVED / UNAVAILABLE / UNKNOWN / NULL`を混同しない。

## 4. GA4 Extraction

### 入力

- GA4 profile/property設定
- report definition
- OAuth credential/token
- date range

### 処理

- Google Analytics Data APIをread-onlyで呼ぶ
- reportごとにdimension/metricを取得
- pagination / API retry / empty handling
- Admin metadataが必要な場合は別APIから取得
- Source値を後段都合で意味変換しない

### 出力

Raw report CSV/metadata/manifestを含むExtraction Artifact。

### 禁止

- URLのbusiness normalizationをExtractionへ混ぜる
- period comparison
- cross-source join
- SEO score

## 5. GA4 Processing — 詳細

GA4 Processingは、GA4 raw reportsを「GA4 Sourceとして意味が閉じた分析可能Dataset」へ変換する。Integrationの都合ではなくGA4固有責務を完了するLayer。

### 5.1 入力検証

- Extraction manifest
- expected report/file
- schema
- date
- source lineage

不正なrunをsilentに読み飛ばさない。

### 5.2 Date canonicalization

GA4 API/rawで得た日付をProcessing以降のcanonical `YYYY-MM-DD`へ揃える。後段で再解釈しない。

### 5.3 URL / Page処理

GA4のpage/landing page URLを、site scopedなpage identityへ変換する。

主要contract:

```text
normalized_page_key
query removed from page identity
fragment removed
path case preserved
percent encoding preserved
trailing slash preserved
host/scheme not part of site-scoped key
```

raw URL/valueを捨てるのではなく、必要なsource fieldsとcanonical fieldを区別して保持する。

### 5.4 Period windows

Source Analysisで使えるcurrent/previous等の期間定義へ接続できるよう、日次Datasetとperiod metadataを整える。

### 5.5 Event / Key Event

GA4 event系reportをsource semanticsに従って整理する。event countやkey eventを別metricとして扱い、dimension違いのrowを不用意に加算しない。

### 5.6 Grain protection

page/date、landing page/date、event/date等、reportごとのgrainを壊さない。異なるgrainを一つのfact tableに押し込んでduplicateさせない。

### 5.7 Output

- processed daily/page tables
- period definition
- URL canonical fields
- event grouping/metrics
- processing manifest
- source lineage

### 5.8 GA4 Processingがしないこと

- GSCとのjoin
- GSC query attribution
- cross-source common period trim
- generic ranking/recommendation

## 6. GA4 Analysis

GA4 Processing ArtifactだけでSource diagnosticを行う。

- summary
- period comparison
- continuity
- dimension period
- metric summaries
- reconciliation
- quality

Source AnalysisはIntegrationの前処理ではなく独立branch。`full` profileでは通らない。

## 7. GSC Extraction

### 入力

- Search Console property
- OAuth
- date range
- presets / dimensions
- search type

### 処理

Search Analytics APIからclicks / impressions / CTR / position等を取得。query/page/date/search_type等のdimensionを保持する。

### Coverage

GSCはreporting lagを持つ。requested rangeとobserved rangeを区別する。

### 禁止

未到着日を0として生成しない。

## 8. GSC Processing — 詳細

GSC Processingは、Search Analytics raw rowsをGSC Sourceとして正規化し、後段で安全に利用できるDatasetへする。

### 8.1 Date

canonical `YYYY-MM-DD`へ統一。

### 8.2 Page identity

GA4と同じcross-source page normalization contractへ合わせる。ただしGSC raw page/query/search_typeはSource informationとして保持する。

### 8.3 Query grain

Queryを持つDatasetとPageのみのDatasetを区別する。query rowへ他grainのmetricを複製して「合計可能」に見せない。

### 8.4 Search type

`web`, image等のsearch_typeが存在する場合、同一page/dateでrow multiplicityが増える。後段でGA4 factをsearch_type数だけ複製しないようSource contextを明示する。

### 8.5 CTR

CTRは`clicks / impressions`というderived ratio。row CTRの単純平均を全体CTRにしない。Aggregationでは分子・分母から再導出する。

### 8.6 Average position

PositionはSUM_SAFEではない。適切なweight/grain semanticsを持つnon-additive metricとして扱う。

### 8.7 Coverage state

requested/resolved/observedを保持し、partial coverageをqualityへ伝える。

### 8.8 Output

- page/day dataset
- query/page/day dataset
- canonical page key
- metric semantics
- coverage metadata
- processing manifest

### 8.9 GSC Processingがしないこと

- queryからGA4 conversion推定
- reporting lagの0埋め
- Integration use-case filter

## 9. GSC Analysis

GSC単体でsummary/comparison/ranking/coverage/reconciliationを行う。CTR/position等のmetric semanticsを壊さない。

## 10. Integration Extraction — 詳細

### 10.1 目的

「全部残して、正しくつなぐ」。

### 10.2 Input

GA4 Processing Artifact + GSC Processing Artifact。Source Analysisは入力ではない。

### 10.3 Join identity

主に`site_key + normalized_page_key + date`等、安全な共通grainで接続。

### 10.4 FULL OUTER相当

- MATCHED
- GA4_ONLY
- GSC_ONLY

を保持。片Sourceにないからrowを落とさない。

### 10.5 Observation state

値が0なのか、sourceが未観測なのか、metric unavailableなのかを分離。

### 10.6 Query context boundary

GSC query-bearing datasetにGA4 metricsをexact query attributionとしてjoinしない。必要ならpage/date contextとしてnon-additiveに扱う。

### 10.7 search_type multiplicity

search_type行数によってGA4 sessions等が複製されないようにする。

### 10.8 Output

- integrated datasets/catalog
- quality
- source/join state
- manifest / lineage

### 10.9 しないこと

- common period trim
- use case filter
- ranking
- recommendation

## 11. Integration Processing — 詳細

Integration Processingは3つのProcessingの中でも役割が異なる。Source固有正規化ではなく、lossless integrated dataから**今回使うDatasetをConfigで組み立てる**。

### 11.1 Config inputs

代表concept:

```text
required_sources
optional_sources
period policy
grain
dimensions
metrics
filters
coverage policy
comparison window
```

### 11.2 Period modes

少なくとも設計上:

- `common_observed`
- `all_available`
- `source_available`
- `explicit_range`
- `latest_n_days`
- `comparison_ready`

複数required sourceのdefaultは`common_observed`。

A+C primaryではGSC observed endが2026-09-24なので、common observedを使えばそこまでが安全な共通期間。

### 11.3 Grain selection

page daily / query-page daily等、Use Caseに必要なgrainを選ぶ。異なるgrainを混ぜてCartesian growthを起こさない。

### 11.4 Metric registry / semantics

例:

- `SUM_SAFE`
- `DERIVED_RATIO`
- `NON_ADDITIVE`
- `WEIGHTED_WITHIN_GRAIN`
- `NON_ADDITIVE_CONTEXT`

#### SUM_SAFE

同一semantic/grain内で加算可能。clicks、impressions等が代表。

#### DERIVED_RATIO

CTR等。比率自体を平均せず、必要ならsum numerator / sum denominatorから再導出。

#### NON_ADDITIVE

Position等、足し算に意味がない。

#### NON_ADDITIVE_CONTEXT

GSC query rowに添えたGA4 page contextなど。row数分を合計してはいけない。

### 11.5 Filters

Config filterで用途対象を選ぶが、どの条件でrowを除外したか追跡できるようreasonを残す。

### 11.6 Coverage

Required Sourceのobserved coverageを満たさないperiodをsilentに完全データ扱いしない。

### 11.7 Output

- processed dataset
- dataset schema
- metric registry snapshot
- period definition
- coverage summary
- processing manifest

### 11.8 しないこと

- Analysis resultの生成
- score/recommendation
- visualization formatting
- Source rawの再正規化

## 12. Integration Analysis — 詳細

Processing Contractに従いsource/use-case branchingを増やさず分析。

### Operations

- summary
- period totals
- safe average
- current vs previous
- YoY
- absolute delta
- relative delta
- trend
- ranking
- share / contribution
- coverage-aware status
- quality-aware suppression

### Status

- OK
- BASELINE_ZERO
- BASELINE_UNAVAILABLE
- CURRENT_UNAVAILABLE
- INSUFFICIENT_COVERAGE

ZEROとUNAVAILABLEを混同しない。

## 13. Visualization Adapter

Analysis ArtifactをGeneric Visualization Input Contractへprojectionする。

AdapterはSource差を吸収するが、新しいanalysisはしない。Generic Contract Adapterにより未知の外部Result Artifactもzero-codeで取り込める設計。

## 14. Visualization Spec / Renderer

Specはcomponent、metric、format、chart hint等を指定。Static/React rendererは同じInput/Specを読む。

React primary。Staticはaudit/fallback。

## 15. Pipeline Orchestration

既存Atomic CLIをstageとして呼ぶ。Layer logicをPipelineへ再実装しない。

6 profilesを定義。Pipeline Manifestでprofile/stages/child Artifact/config hash/failureを追跡。

`full`のresumeでは成功Artifact hashを検証し再利用。手動Artifact差し込みはsuppliedとして記録。

## 16. Quality / Reconciliation

Qualityはデータ削除の理由ではなく解釈情報。Source Analysis/Integration等でsame-grain reconciliationやcoverage differenceを検証する。

## 17. Determinism

Deterministic Layerではstable sort、tie break、null order、float tolerance、config/spec hashを固定。将来stochastic modelを導入する場合はseed/model version/parametersをManifestへ保存する。

## 18. Security / Privacy

- credential/tokenをArtifactへ含めない
- secret scan
- machine absolute pathをPersistent Manifestへ残さない
- read-only analytics APIを基本にする

## 19. Post-MVP Layer

Website / Intervention / Modeling / Experiment / Causal / Decision / Semantic / LLMを追加しても、Observed SourceとDerived/Predicted/Causal evidenceを混同しない。

## 20. Processing 3層の違いを一表で確認

| Layer | 主責務 | 典型入力 | 典型加工 | やらないこと |
|---|---|---|---|---|
| GA4 Processing | GA4 Source固有正規化 | GA4 raw | date/page/event/grain | GSC join |
| GSC Processing | GSC Source固有正規化 | GSC raw | page/query/search_type/CTR/position/coverage | GA4 attribution |
| Integration Processing | Use Case dataset assembly | lossless integrated | period/grain/metric/filter/coverage | Source raw再解釈 |

この責務分離を崩さないことが、Post-MVP拡張時に再仕分けを発生させないための重要条件。

## 21. Observation / Missing stateの詳細

Cross-sourceで数値が存在しない理由を1種類にまとめない。

| State | 意味 | 0扱い |
|---|---|---|
| ZERO | 観測され値が0 | 数値0 |
| NOT_OBSERVED | そのSource/期間で未観測 | 不可 |
| UNAVAILABLE | metric/source契約上取得不能 | 不可 |
| UNKNOWN | 判定不能 | 不可 |
| NULL | field値なし | context依存 |

この区別はAnalysis、Visualization、Decision Supportまで伝播させる。

## 22. Period alignmentの具体例

GA4が2026-09-26まで観測、GSCがreporting lagで2026-09-24まで観測の場合:

- `common_observed`: 09-24まで
- `all_available`: GA4の09-25/26も保持し、GSC NOT_OBSERVEDを明示

09-25/26のGSCを0にすると「検索流入が急落した」という偽signalを作るため禁止。

## 23. Derived Ratioの具体例

Dataset A:

```text
row1 clicks=10 impressions=100 CTR=10%
row2 clicks=90 impressions=900 CTR=10%
```

これは単純平均でも10%だが、一般にはrow sizeが違うため`mean(CTR)`を使わない。

```text
overall CTR = sum(clicks) / sum(impressions)
```

この原則をmetric registryへ持たせる。

## 24. Non-additive contextの具体例

同一page/dateにGSC queryが100件あるとする。GA4 sessions=50を各query rowへcontextとして付けるだけで`50 x 100 = 5,000 sessions`と合計してはいけない。

必要な場合は:

- display-only context
- dedicated non-additive semantic
- safe parent-grain reference

として扱う。

## 25. search_typeの具体例

同一page/dateに`web`と`image`の2 rowがあっても、GA4 page factを2倍にしない。GSC側のdimension multiplicityとGA4 observation grainを分離する。

## 26. Integration ExtractionとProcessingの境界例

「過去90日だけ見たい」という要望があっても、Integration Extractionで90日に切らない。

```text
Integration Extraction: 全available unionを保持
Integration Processing: latest_n_days=90を適用
```

これにより後から16か月分析へ作り直すときExtractionを再取得/再統合しなくてよい。

## 27. Processing Filterの監査

Filterでrowを除外する場合、可能な限りbefore/after row countやreasonをManifest/summaryへ残す。

Silent dropは「データが元から無かった」のか「用途上外した」のか区別できなくなる。

## 28. AnalysisのBaseline Zero

current=100, baseline=0のときrelative deltaを単純に`inf`や`10000%`へしない。`BASELINE_ZERO`として扱い、absolute deltaは100、relativeはstatus-awareに表示する。

## 29. AnalysisのBaseline Unavailable

baseline periodがcoverage外なら、0とみなして増加扱いにしない。`BASELINE_UNAVAILABLE`。

## 30. Ranking / Contributionの安全性

Ranking対象metricがnon-additiveなら、そのmetricのsemanticsに合うgrain/valueを使う。Contributionは分母が意味的に加算可能な場合だけ生成する。

## 31. Dataset Catalog

Integration各Layerでは複数dataset/grainが存在し得るため、名称だけでなくschema/grain/metric semantics/coverageをCatalogで追える設計が重要。

## 32. Config / Spec hash

同じinput Artifactでもconfig/specが変われば別結果。Manifestへhashを残し、file名だけで「同条件」と判断しない。

## 33. PipelineがLayer logicを持たない理由

Pipeline内にProcessing/Analysis計算を再実装すると、Atomic CLIと結果が分岐する。Pipelineはcommand/Artifact orchestrationに限定し、Layer sourceを一つに保つ。

## 34. Pipeline resumeのdownstream invalidation

上流Stageを`supplied`で差し替えた場合、その下流で過去に成功したArtifactを無条件再利用してはいけない。新しいparent Artifactに依存するdownstreamをinvalidateして再実行する必要がある。

## 35. Visualization Input Contractで持つ情報

代表:

```text
metric_id
metric_display_name
value_type
unit
semantic_type
dimensions
time_dimension
grain
chart_hint
format_hint
coverage
status
quality
lineage
result_type
```

RendererはSource名ではなくこれらを読む。

## 36. Generic Contract Adapter

未知のResult Artifactも、required contractを宣言的にmappingできればVisualization Core変更なしで取り込む。これによりPost-MVP source追加のたびにrenderer branchを増やさない。

## 37. Original input ZIP

Visualizationは入力Artifactをdeterministic ZIPとして同梱する。Dashboardからdownloadする対象はこのZIPで、Visualization自身のbundleではない。

## 38. Testsの層

```text
unit
contract
fixture integration
artifact validation
regression
E2E
live E2E
```

外部APIを呼ばなくても大半のContractを検証できる構造を優先する。

## 39. Reference Fixtureで見るべきこと

A+Cでは長期cross-source、B+Cではperiod control。

最低確認:

- MATCHED / GA4_ONLY / GSC_ONLY
- ZERO / unavailable
- partial GSC coverage
- query-bearing URL
- search_type multiplicity
- baseline zero/unavailable

## 40. 将来Source追加時の原則

WebsiteやBusiness KPIを追加するときも、既存GA4/GSC moduleへ雑にfieldを足すのではなく、Source Extraction/Processing/Analysis責務を持つ独立adapter/moduleとして追加する。

## 41. Evidence typeの分離

将来:

- Observed
- Derived
- Predicted
- Causal
- LLM-generated

を明示し、DashboardやDecision Supportで同じ「事実」として表示しない。

## 42. 全層で共通する禁止事項

- raw overwrite
- silent row drop
- missing -> 0 coercion
- unsafe grain merge
- hidden config
- machine-specific persistent path
- secret leakage
- downstream再計算によるsemantic drift



## 43. 詳細版の読み方

本章以降は、前半の概念説明を「実装を追跡できる粒度」へ展開する。

各Layerについて以下を分ける。

```text
Input
Validation
Normalization / Transformation
Metric semantics
Coverage / Missing
Output
Manifest / Lineage
Failure modes
Forbidden responsibility
Verification
```

## 44. Source Extraction共通責務

Extractionは外部SourceからSource Factを取得し、raw性を保持したArtifactを作る。

### 44.1 共通入力

- authentication material
- source/property/site identifier
- requested date expression
- report/preset definition
- source-specific request options
- output root

### 44.2 共通処理

- request parameter解決
- date expression解決
- authentication
- pagination/partition
- retry
- request stats
- source response serialization
- manifest生成

### 44.3 共通でやらないこと

- cross-source join
- use-case period trim
- recommendation
- score
- causal interpretation
- missing dayの0化

## 45. GA4 Extraction 深掘り

### 45.1 Date expression

現行は`YYYY-MM-DD / today / yesterday / NdaysAgo`。Property timezone semanticsに従ってresolveする。

### 45.2 Report definition

GA4はreport definitionごとにdimension/metric/grainが異なる。Extraction時点で異なるreportを無理に単一grainへ統合しない。

### 45.3 Pagination

API limitを超えるreportは複数requestへ分割し、rowを欠損なく収集する。request count/row countはManifestで追跡可能にする。

### 45.4 Empty / degraded

空resultとrequest failureを区別する。expected emptyをerror rowとして捏造しない。

### 45.5 Metadata

利用可能なdimension/metric等のsource metadataを必要に応じ取得し、後段の意味解釈に使えるよう保存する。

## 46. GA4 Processing 完全手順

GA4 ProcessingはGA4 Source内で必要な正規化・canonicalization・Source固有派生を完了する。

### 46.1 Step 1 — Input Artifact選択

- Extraction run directory
- extraction manifest
- report files
- metadata

を一組として扱う。

### 46.2 Step 2 — Manifest検証

- source/layer
- status
- row counts
- expected files
- date range
- schema/version

を確認する。親runを曖昧な`latest`だけで決めない。

### 46.3 Step 3 — Schema/Type解決

日付、integer/float、dimension stringをProcessing contractへ揃える。型変換失敗をsilent coercionで隠さない。

### 46.4 Step 4 — Date canonicalization

Processing以降のcanonical dateは`YYYY-MM-DD`。Source time semanticsはManifest/metadataに残す。

### 46.5 Step 5 — Page/URL normalization

raw URL/valueとcanonical page identityを分ける。

```text
raw/source field  -> source factとして保持
normalized_page_key -> cross-source identity候補
```

query/fragmentをpage identityから外す一方、元情報を不用意に捨てない。

### 46.6 Step 6 — Report grain保持

page/date、landing page/date、event/date等を別grainとして保持する。dimension欠落を埋めて無理にunionしない。

### 46.7 Step 7 — Event/Key Event semantics

Event Count、Key Event等はsource metric定義を維持する。別report/grainの数値を二重計上しない。

### 46.8 Step 8 — Period-ready data

Source Analysisがcurrent/previous/YoY等を作れるよう、daily canonical datasetとperiod metadataを整える。Processing自体が最終Business judgmentを行うわけではない。

### 46.9 Step 9 — Reconciliation-ready output

raw/processed row countやmetric totalsを比較可能にし、後段Analysisでreconciliationできるようlineageを保持する。

### 46.10 Step 10 — Manifest

Processing config hash、parent manifest hash、schema、output files、row count等を記録する。

## 47. GSC Extraction 深掘り

### 47.1 Date/Retention

Search Consoleは直近16か月を保持する。requested rangeがそれ以前へ広くても、保持範囲外の古いデータは存在しない。

### 47.2 Data state / lag

recent daysはreporting lagを持つ。requested endとobserved endを区別する。

### 47.3 Dimensions

代表dimension:

- date
- page
- query
- country
- device
- search appearance等

preset/reportごとのgrainを壊さない。

### 47.4 Search type

web/image/video/news等のsearch_typeをSource dimension/contextとして保持する。

## 48. GSC Processing 完全手順

### 48.1 Step 1 — Input/Manifest validation

site URL、preset、search type、requested range、observed coverage、output filesを確認する。

### 48.2 Step 2 — Canonical date

`YYYY-MM-DD`へ統一する。

### 48.3 Step 3 — Page normalization

GA4と同じpage key contractへ合わせる。ただしGSC raw pageはSource factとして保持する。

### 48.4 Step 4 — Query/page grain separation

page-onlyとquery-pageを混同しない。query dimension追加でrow数が増えるのはgrainが細かくなったためであり、GA4 metricをquery数だけ加算可能に複製してはいけない。

### 48.5 Step 5 — clicks/impressions

加算可能なcountとして扱う。ただしdimension/grainが同一であることが前提。

### 48.6 Step 6 — CTR

CTRはderived ratio。

```text
CTR = sum(clicks) / sum(impressions)
```

row CTRの算術平均ではない。

### 48.7 Step 7 — Position

Average positionは単純SUMできない。Source semanticsに沿うweight/grainを保護する。

### 48.8 Step 8 — Coverage

requested / resolved / observedを分け、未観測日をZEROへ変換しない。

### 48.9 Step 9 — Manifest/Quality

parent hash、coverage、row count、schema、warningsを記録する。

## 49. Integration Extraction 完全手順

### 49.1 Input

GA4 Processing + GSC Processing。

### 49.2 Parent validation

- source type
- site mapping
- manifest hash
- schema compatibility
- date/page key contract

### 49.3 Join identity

主軸は`site_key + normalized_page_key + date`。必要に応じSource-specific dimensionsを別grainで保持する。

### 49.4 FULL OUTER相当

GA4_ONLY / GSC_ONLY / MATCHEDを保持し、片側にしかないFactを削らない。

### 49.5 Query attribution boundary

GSC queryからGA4 session/conversion/key eventをexact attributionしない。

### 49.6 NON_ADDITIVE_CONTEXT

query grainへGA4 page contextを付ける場合、query横断SUM不可であることをsemantic typeで明示する。

### 49.7 Cartesian growth防止

search_type/query等の多値dimensionとGA4 factのjoinでrow multiplicationを監査する。

## 50. Integration Processing 完全手順

Integration Processingは「統合済みの全情報」から、今回の分析に必要なDatasetをConfigで組み立てるLayer。

### 50.1 Step 1 — Processing Config解決

Configから以下を読む。

- required_sources
- optional_sources
- period mode
- explicit/latest windows
- grain
- dimensions
- metrics
- filters
- coverage policy
- comparison readiness

### 50.2 Step 2 — Input Manifest validation

Integration Extraction Artifactがexpected schema/lineageを満たすか確認する。

### 50.3 Step 3 — Period resolution

代表mode:

- common_observed
- all_available
- source_available
- explicit_range
- latest_n_days
- comparison_ready

`common_observed`はrequired sourceすべてが観測された共通範囲へ揃える。

### 50.4 Step 4 — Source requirement

required source欠落rowをどう扱うか、optional source欠落を許すかをConfigで決める。

### 50.5 Step 5 — Grain resolution

分析Datasetの一意キーを決める。page/dayとquery/page/day等を明確に分ける。

### 50.6 Step 6 — Dimension selection

必要dimensionのみ選ぶが、grainを一意に保つために必要なdimensionを落とさない。

### 50.7 Step 7 — Metric registry

各metricにsemantic typeを持たせる。

- SUM_SAFE
- DERIVED_RATIO
- NON_ADDITIVE
- WEIGHTED_WITHIN_GRAIN
- NON_ADDITIVE_CONTEXT

### 50.8 Step 8 — Safe aggregation

SUM_SAFEだけを無条件SUM対象にする。DERIVED_RATIOは分子分母から再計算。Position等は対応semanticに従う。

### 50.9 Step 9 — Filter

filter適用理由、before/after row countを追跡可能にする。filterでSource Factを上流から削除しない。

### 50.10 Step 10 — Coverage policy

partial coverageやrequired source不足をDataset metadataへ反映する。

### 50.11 Step 11 — Period/comparison metadata

current/previous/YoY等のAnalysisが誤解なく使えるperiod definitionを出力する。

### 50.12 Step 12 — Output contracts

- processed dataset
- dataset schema
- metric registry snapshot
- period definition
- coverage summary
- processing manifest

### 50.13 Step 13 — Determinism

同一input/configならstable sort・tie handling・null ordering等を固定し、再現可能にする。

## 51. Analysis Layerの責務境界

AnalysisはDatasetを「どう読むか」をSpecで決める。

- summary
- totals
- comparison
- trend
- ranking
- contribution
- coverage-aware status
- quality-aware suppression

Source取得やURL normalizationをやり直さない。

## 52. Visualization Layerの責務境界

VisualizationはPresentation。

- AdapterでResult ArtifactをInput Contractへ変換
- Specでcomponent/formatを決める
- React/static rendererが同じcontractを読む
- upstream valueを再集計しない

## 53. Pipeline Layerの責務境界

PipelineはAtomic CLIをorchestrateする。

- profile
- stage plan
- subprocess execution
- checkpoint
- resume
- reused/supplied
- pipeline manifest

Layer logicをPipelineへ再実装しない。

## 54. Config Architecture

```text
config/user   = 利用者選択
config/system = metric/processing/analysis/visualization contract
```

Config hashをManifestへ保存し、同じ名前でも内容が違うrunを区別する。

## 55. Artifact / Manifest Architecture

各runはimmutable。直接親のmanifest hashを保持し、lineage chainを形成する。

Persistent metadataはproject-relative pathを使う。

## 56. Quality / Coverage Architecture

QualityとCoverageは別概念。

Coverage: どこまで観測できたか。

Quality: contract/reconciliation/availability等の健全性。

## 57. Missing Semantics Matrix

| State | 観測 | 値 | 比較可能性 |
|---|---|---|---|
| ZERO | あり | 0 | 原則あり |
| NOT_OBSERVED | なし | 不明 | 制限 |
| UNAVAILABLE | metric利用不可 | なし | 不可/制限 |
| UNKNOWN | 判定不能 | 不明 | 制限 |
| NULL | representation上なし | なし | context依存 |

## 58. Processing 3種の責任比較

### GA4 Processing

GA4 raw -> GA4 canonical Source dataset。

### GSC Processing

GSC raw -> GSC canonical Source dataset + coverage semantics。

### Integration Processing

Lossless integrated dataset -> use-case/analysis-ready dataset。

3つを一つの「Processing」として雑にまとめない。

## 59. 典型的な禁止パターン

- Extractionでuse-case trim
- Processingで別Sourceを勝手にjoin
- Integration Extractionでcommon periodへ削る
- Query grainでGA4 metricをSUM
- CTR row-average
- Position sum
- missing=0
- Visualization再集計
- PipelineへLayer logic複製

## 60. 実装Inventoryの位置づけ

> **Snapshot notice:** 以下のImplementation / Config / CLI / Test Inventoryは2026-10-03時点の自動抽出snapshotをlosslessに保持する。v0.12.4 current deltaは本書冒頭の「MVP COMPLETE Implementation Delta」を優先し、inventoryをcurrent repoと誤認しない。

以下の自動生成Inventoryは、docs生成時のcurrent repoをAST/config/CLIから読み取り、概念説明とsource codeの間を接続する。

これはsource codeの代替ではない。実装が変わった場合はsource codeが最終Source of Truthであり、docs更新時にInventoryを再生成する。

## 61. Implementation Module / Symbol Inventory

### 61.1 `src/web_intelligence/__init__.py`

- Module summary: (module docstringなし)
- Top-level symbol count: 0

### 61.2 `src/web_intelligence/__main__.py`

- Module summary: (module docstringなし)
- Top-level symbol count: 0

### 61.3 `src/web_intelligence/analysis/__init__.py`

- Module summary: Source analysis layers.
- Top-level symbol count: 0

### 61.4 `src/web_intelligence/analysis/ga4/__init__.py`

- Module summary: GA4 analysis layer.
- Top-level symbol count: 0

### 61.5 `src/web_intelligence/analysis/ga4/analyzer.py`

- Module summary: (module docstringなし)
- Top-level symbol count: 9

#### Function `_project_relative_path(path: Path)`

- source line: 51
- kind: function
- doc: (docstringなし)

#### Class `AnalysisError`

- source line: 58
- bases: `Exception`
- doc: (docstringなし)

#### Function `_dataset_entry(name: str, output_file: str, row_count: int, status: str, source_files: list[str], source_statuses: list[str], notes: list[str] | None=None)`

- source line: 62
- kind: function
- doc: (docstringなし)

#### Function `_table_status(index: dict[str, dict[str, Any]], relative: str)`

- source line: 82
- kind: function
- doc: (docstringなし)

#### Function `_write_simple_dataset(processing_run: Path, analysis_run: Path, table_index: dict[str, dict[str, Any]], dataset_name: str, source_relative: str, output_relative: str, fieldnames: list[str], builder: Callable[[Path], list[dict[str, str]]])`

- source line: 89
- kind: function
- doc: (docstringなし)

#### Function `_period_inputs(source_dir: Path)`

- source line: 126
- kind: function
- doc: (docstringなし)

#### Function `_period_datasets(source_dir: Path, output_dir: Path, processing_manifest: dict[str, Any], *, coverage_rows: list[dict[str, str]], key_event_bounds: tuple[date | None, date | None] | None)`

- source line: 134
- kind: function
- doc: (docstringなし)

#### Function `_dimension_period_datasets(source_dir: Path, output_dir: Path, table_index: dict[str, dict[str, Any]], coverage_rows: list[dict[str, str]], event_continuity: dict[str, dict[str, str]])`

- source line: 261
- kind: function
- doc: (docstringなし)

#### Function `analyze_processing_run(processing_run: str | Path, output_root: str | Path, run_id: str, started_at: datetime)`

- source line: 324
- kind: function
- doc: (docstringなし)

### 61.6 `src/web_intelligence/analysis/ga4/cli.py`

- Module summary: (module docstringなし)
- Top-level symbol count: 4

#### Function `_manifest_started_at(path: Path)`

- source line: 16
- kind: function
- doc: (docstringなし)

#### Function `find_latest_processing_run(processing_root: str | Path, property_id: str | None=None)`

- source line: 35
- kind: function
- doc: (docstringなし)

#### Function `build_parser()`

- source line: 62
- kind: function
- doc: (docstringなし)

#### Function `main(argv: list[str] | None=None)`

- source line: 94
- kind: function
- doc: (docstringなし)

### 61.7 `src/web_intelligence/analysis/ga4/continuity.py`

- Module summary: (module docstringなし)
- Top-level symbol count: 6

#### Function `_parse_processed_date(value: str)`

- source line: 17
- kind: function
- doc: (docstringなし)

#### Function `_parse_iso_date(value: str)`

- source line: 23
- kind: function
- doc: (docstringなし)

#### Function `build_event_continuity_rows(events_daily_path: str | Path)`

- source line: 27
- kind: function
- doc: (docstringなし)

#### Function `continuity_index(rows: list[dict[str, str]])`

- source line: 108
- kind: function
- doc: (docstringなし)

#### Function `overall_key_event_bounds(rows: list[dict[str, str]])`

- source line: 114
- kind: function
- doc: (docstringなし)

#### Function `comparison_continuity_status(*, current_value: Decimal | None, baseline_value: Decimal | None, current_start: date, baseline_end: date, first_nonzero: date | None, last_nonzero: date | None)`

- source line: 130
- kind: function
- doc: (docstringなし)

### 61.8 `src/web_intelligence/analysis/ga4/dimension_periods.py`

- Module summary: (module docstringなし)
- Top-level symbol count: 8

#### Class `DimensionPeriodSpec`

- source line: 26
- bases: `(none)`
- doc: (docstringなし)

#### Function `comparison_fieldnames()`

- source line: 74
- kind: function
- doc: (docstringなし)

#### Function `_parse_processed_date(value: str)`

- source line: 96
- kind: function
- doc: (docstringなし)

#### Function `_parse_iso_date(value: str)`

- source line: 102
- kind: function
- doc: (docstringなし)

#### Function `_dimension_value(row: dict[str, str], spec: DimensionPeriodSpec)`

- source line: 106
- kind: function
- doc: (docstringなし)

#### Function `_rows_in_range(rows: list[dict[str, str]], start: date, end: date)`

- source line: 114
- kind: function
- doc: (docstringなし)

#### Function `_metric_value(rows: list[dict[str, str]], metric: str, columns: set[str])`

- source line: 130
- kind: function
- doc: (docstringなし)

#### Function `build_dimension_period_comparison_rows(source_path: str | Path, coverage_rows: list[dict[str, str]], spec: DimensionPeriodSpec, *, source_quality_status: str, event_continuity: dict[str, dict[str, str]] | None=None)`

- source line: 161
- kind: function
- doc: (docstringなし)

### 61.9 `src/web_intelligence/analysis/ga4/io.py`

- Module summary: (module docstringなし)
- Top-level symbol count: 6

#### Class `AnalysisIOError`

- source line: 12
- bases: `Exception`
- doc: Raised when an Analysis input/output artifact cannot be handled safely.

#### Function `sha256_file(path: str | Path)`

- source line: 16
- kind: function
- doc: (docstringなし)

#### Function `read_json(path: str | Path)`

- source line: 24
- kind: function
- doc: (docstringなし)

#### Function `read_csv_rows(path: str | Path)`

- source line: 35
- kind: function
- doc: (docstringなし)

#### Function `write_csv_atomic(path: str | Path, fieldnames: Sequence[str], rows: Iterable[Mapping[str, Any]])`

- source line: 48
- kind: function
- doc: (docstringなし)

#### Function `write_json_atomic(path: str | Path, payload: Mapping[str, Any])`

- source line: 78
- kind: function
- doc: (docstringなし)

### 61.10 `src/web_intelligence/analysis/ga4/metrics.py`

- Module summary: (module docstringなし)
- Top-level symbol count: 6

#### Class `MetricRule`

- source line: 19
- bases: `(none)`
- doc: (docstringなし)

#### Function `metric_rule(name: str)`

- source line: 100
- kind: function
- doc: (docstringなし)

#### Function `parse_decimal(value: str | None)`

- source line: 104
- kind: function
- doc: (docstringなし)

#### Function `decimal_text(value: Decimal | None)`

- source line: 113
- kind: function
- doc: (docstringなし)

#### Function `sum_column(rows: Iterable[Mapping[str, str]], column: str)`

- source line: 124
- kind: function
- doc: (docstringなし)

#### Function `aggregate_metric(rows: list[Mapping[str, str]], rule: MetricRule, columns: set[str])`

- source line: 136
- kind: function
- doc: (docstringなし)

### 61.11 `src/web_intelligence/analysis/ga4/periods.py`

- Module summary: (module docstringなし)
- Top-level symbol count: 5

#### Function `build_metric_semantics_rows()`

- source line: 19
- kind: function
- doc: (docstringなし)

#### Function `build_period_metric_rows(period_windows_path: str | Path, coverage_rows: list[dict[str, str]])`

- source line: 32
- kind: function
- doc: (docstringなし)

#### Function `_to_decimal(value: str)`

- source line: 73
- kind: function
- doc: (docstringなし)

#### Function `_coverage_index(coverage_rows: list[dict[str, str]])`

- source line: 82
- kind: function
- doc: (docstringなし)

#### Function `build_period_comparison_rows(period_metric_rows: list[dict[str, str]], coverage_rows: list[dict[str, str]] | None=None, key_event_bounds: tuple[date | None, date | None] | None=None)`

- source line: 91
- kind: function
- doc: (docstringなし)

### 61.12 `src/web_intelligence/analysis/ga4/quality.py`

- Module summary: (module docstringなし)
- Top-level symbol count: 3

#### Function `source_quality_rows(processing_manifest: dict[str, Any])`

- source line: 7
- kind: function
- doc: (docstringなし)

#### Function `table_quality_index(processing_manifest: dict[str, Any])`

- source line: 50
- kind: function
- doc: (docstringなし)

#### Function `dataset_status(row_count: int, source_statuses: list[str], *, missing: bool=False)`

- source line: 66
- kind: function
- doc: (docstringなし)

### 61.13 `src/web_intelligence/analysis/ga4/reconciliation.py`

- Module summary: (module docstringなし)
- Top-level symbol count: 8

#### Function `reconciliation_fieldnames()`

- source line: 22
- kind: function
- doc: (docstringなし)

#### Function `_period_metric_index(period_metrics_path: Path)`

- source line: 41
- kind: function
- doc: (docstringなし)

#### Function `_rows_for_period(rows: list[dict[str, str]], window_name: str, comparison: str)`

- source line: 57
- kind: function
- doc: (docstringなし)

#### Function `_dimension_coverage(rows: list[dict[str, str]], comparison: str)`

- source line: 83
- kind: function
- doc: (docstringなし)

#### Function `_source_quality(rows: list[dict[str, str]])`

- source line: 96
- kind: function
- doc: (docstringなし)

#### Function `_sum_metric(rows: list[dict[str, str]], metric: str, value_column: str)`

- source line: 111
- kind: function
- doc: (docstringなし)

#### Function `_dimension_aggregate(rows: list[dict[str, str]], metric: str, value_column: str)`

- source line: 128
- kind: function
- doc: (docstringなし)

#### Function `build_reconciliation_rows(analysis_run: str | Path)`

- source line: 152
- kind: function
- doc: (docstringなし)

### 61.14 `src/web_intelligence/analysis/ga4/summaries.py`

- Module summary: (module docstringなし)
- Top-level symbol count: 7

#### Function `_decimal(row: dict[str, str], column: str)`

- source line: 12
- kind: function
- doc: (docstringなし)

#### Function `_ratio(numerator: Decimal, denominator: Decimal)`

- source line: 16
- kind: function
- doc: (docstringなし)

#### Function `acquisition_summary(path: str | Path)`

- source line: 22
- kind: function
- doc: (docstringなし)

#### Function `_normalized_value(row: dict[str, str], raw_column: str)`

- source line: 58
- kind: function
- doc: (docstringなし)

#### Function `landing_page_summary(path: str | Path)`

- source line: 63
- kind: function
- doc: (docstringなし)

#### Function `page_summary(path: str | Path)`

- source line: 108
- kind: function
- doc: (docstringなし)

#### Function `event_summary(events_by_page_path: str | Path, key_events_by_page_path: str | Path)`

- source line: 143
- kind: function
- doc: (docstringなし)

### 61.15 `src/web_intelligence/analysis/gsc/__init__.py`

- Module summary: Google Search Console analysis layer.
- Top-level symbol count: 0

### 61.16 `src/web_intelligence/analysis/gsc/analyzer.py`

- Module summary: (module docstringなし)
- Top-level symbol count: 35

#### Class `GscAnalysisError`

- source line: 25
- bases: `RuntimeError`
- doc: (docstringなし)

#### Function `_tool_version()`

- source line: 90
- kind: function
- doc: (docstringなし)

#### Function `_utc_now()`

- source line: 97
- kind: function
- doc: (docstringなし)

#### Function `_run_id(now: datetime)`

- source line: 101
- kind: function
- doc: (docstringなし)

#### Function `_sha256_file(path: Path)`

- source line: 105
- kind: function
- doc: (docstringなし)

#### Function `_read_json(path: Path)`

- source line: 113
- kind: function
- doc: (docstringなし)

#### Function `_write_json_atomic(path: Path, payload: dict[str, Any])`

- source line: 123
- kind: function
- doc: (docstringなし)

#### Function `_manifest_started(path: Path)`

- source line: 133
- kind: function
- doc: (docstringなし)

#### Function `find_latest_processing(root: Path=Path('output/processing/gsc'))`

- source line: 142
- kind: function
- doc: (docstringなし)

#### Function `_read_csv(path: Path)`

- source line: 156
- kind: function
- doc: (docstringなし)

#### Function `_write_csv_atomic(path: Path, fieldnames: list[str], rows: Iterable[dict[str, object]])`

- source line: 163
- kind: function
- doc: (docstringなし)

#### Function `_format_value(value: object)`

- source line: 185
- kind: function
- doc: (docstringなし)

#### Function `_table_index(manifest: dict[str, Any])`

- source line: 193
- kind: function
- doc: (docstringなし)

#### Function `_window_map(items: object)`

- source line: 204
- kind: function
- doc: (docstringなし)

#### Function `_windows_for_search_type(manifest: dict[str, Any], search_type: str)`

- source line: 215
- kind: function
- doc: (docstringなし)

#### Function `_windows(manifest: dict[str, Any])`

- source line: 228
- kind: function
- doc: (docstringなし)

#### Function `_window_rows(rows: list[dict[str, str]], window_name: str)`

- source line: 234
- kind: function
- doc: (docstringなし)

#### Function `_aggregate_window(rows: list[dict[str, str]], window: dict[str, Any], *, search_type: str)`

- source line: 242
- kind: function
- doc: (docstringなし)

#### Function `_optional_metric_value(aggregate: Aggregate | None, metric: str)`

- source line: 257
- kind: function
- doc: (docstringなし)

#### Function `_aggregate_by(rows: list[dict[str, str]], keys: tuple[str, ...], *, search_type: str)`

- source line: 266
- kind: function
- doc: (docstringなし)

#### Function `_percent_change(current: Decimal | None, baseline: Decimal | None)`

- source line: 282
- kind: function
- doc: (docstringなし)

#### Function `_comparison_status(*, current_coverage: str, baseline_coverage: str, baseline_impressions: Decimal | None)`

- source line: 291
- kind: function
- doc: (docstringなし)

#### Function `_comparison_row(*, search_type: str, keys: tuple[str, ...], key_values: tuple[str, ...], current: Aggregate | None, baseline: Aggregate | None, current_window: dict[str, Any], baseline_window: dict[str, Any], comparison: str, quality_status: str, quality_reasons: list[str])`

- source line: 310
- kind: function
- doc: (docstringなし)

#### Function `_write_metric_semantics(run_dir: Path)`

- source line: 364
- kind: function
- doc: (docstringなし)

#### Function `_write_quality(run_dir: Path, tables: list[dict[str, Any]])`

- source line: 389
- kind: function
- doc: (docstringなし)

#### Function `_write_full_summaries(processing_run: Path, run_dir: Path, table_index: dict[tuple[str, str], dict[str, Any]])`

- source line: 421
- kind: function
- doc: (docstringなし)

#### Function `_write_period_metrics(processing_run: Path, run_dir: Path, table_index: dict[tuple[str, str], dict[str, Any]], windows_by_search_type: dict[str, dict[str, dict[str, Any]]])`

- source line: 492
- kind: function
- doc: (docstringなし)

#### Function `_comparison_pairs(windows: dict[str, dict[str, Any]])`

- source line: 586
- kind: function
- doc: (docstringなし)

#### Function `_write_overall_comparisons(processing_run: Path, run_dir: Path, table_index: dict[tuple[str, str], dict[str, Any]], windows_by_search_type: dict[str, dict[str, dict[str, Any]]])`

- source line: 609
- kind: function
- doc: (docstringなし)

#### Function `_comparison_fields(keys: tuple[str, ...])`

- source line: 663
- kind: function
- doc: (docstringなし)

#### Function `_write_dimension_period_comparisons(processing_run: Path, run_dir: Path, table_index: dict[tuple[str, str], dict[str, Any]], windows_by_search_type: dict[str, dict[str, dict[str, Any]]])`

- source line: 693
- kind: function
- doc: (docstringなし)

#### Function `_aggregate_csv(path: Path, *, search_type: str)`

- source line: 806
- kind: function
- doc: (docstringなし)

#### Function `_reconciliation_status(left: Decimal | None, right: Decimal | None, *, metric: str | None=None)`

- source line: 814
- kind: function
- doc: (docstringなし)

#### Function `_write_reconciliation(processing_run: Path, run_dir: Path, table_index: dict[tuple[str, str], dict[str, Any]])`

- source line: 836
- kind: function
- doc: (docstringなし)

#### Function `analyze_search_console(processing_run: Path, *, output_root: Path=Path('output/analysis/gsc'))`

- source line: 987
- kind: function
- doc: (docstringなし)

### 61.17 `src/web_intelligence/analysis/gsc/cli.py`

- Module summary: (module docstringなし)
- Top-level symbol count: 2

#### Function `_parser()`

- source line: 17
- kind: function
- doc: (docstringなし)

#### Function `main()`

- source line: 34
- kind: function
- doc: (docstringなし)

### 61.18 `src/web_intelligence/analysis/gsc/metrics.py`

- Module summary: (module docstringなし)
- Top-level symbol count: 5

#### Function `decimal_or_zero(value: object)`

- source line: 15
- kind: function
- doc: (docstringなし)

#### Function `position_supported(search_type: str)`

- source line: 24
- kind: function
- doc: (docstringなし)

#### Class `Aggregate`

- source line: 29
- bases: `(none)`
- doc: (docstringなし)
- methods:
  - `add(self, row: Mapping[str, object], *, allow_position: bool)` — line 35 — (docstringなし)
  - `ctr(self)` — line 52 — (docstringなし)
  - `position(self)` — line 58 — (docstringなし)

#### Function `aggregate_rows(rows: Iterable[Mapping[str, object]], *, search_type: str)`

- source line: 64
- kind: function
- doc: (docstringなし)

#### Function `metric_value(aggregate: Aggregate, metric: str)`

- source line: 76
- kind: function
- doc: (docstringなし)

### 61.19 `src/web_intelligence/cli.py`

- Module summary: (module docstringなし)
- Top-level symbol count: 2

#### Function `build_parser()`

- source line: 8
- kind: function
- doc: (docstringなし)

#### Function `main(argv: list[str] | None=None)`

- source line: 22
- kind: function
- doc: (docstringなし)

### 61.20 `src/web_intelligence/common/__init__.py`

- Module summary: Cross-cutting utilities shared by independent pipeline layers.
- Top-level symbol count: 0

### 61.21 `src/web_intelligence/common/config_paths.py`

- Module summary: (module docstringなし)
- Top-level symbol count: 2

#### Function `user_config_path(*parts: str)`

- source line: 10
- kind: function
- doc: Return a project-relative path below config/user.

#### Function `system_config_path(*parts: str)`

- source line: 15
- kind: function
- doc: Return a project-relative path below config/system.

### 61.22 `src/web_intelligence/common/date_ranges.py`

- Module summary: (module docstringなし)
- Top-level symbol count: 5

#### Class `DateRangeError`

- source line: 11
- bases: `ValueError`
- doc: (docstringなし)

#### Class `ResolvedDateRange`

- source line: 16
- bases: `(none)`
- doc: (docstringなし)
- methods:
  - `start_iso(self)` — line 24 — (docstringなし)
  - `end_iso(self)` — line 28 — (docstringなし)
  - `day_count(self)` — line 32 — (docstringなし)

#### Function `today_in_timezone(timezone_name: str)`

- source line: 36
- kind: function
- doc: (docstringなし)

#### Function `resolve_date_expression(value: str, *, today: date)`

- source line: 44
- kind: function
- doc: (docstringなし)

#### Function `resolve_date_range(start: str, end: str, *, today: date, source_time_semantics: str)`

- source line: 70
- kind: function
- doc: (docstringなし)

### 61.23 `src/web_intelligence/common/google_auth.py`

- Module summary: (module docstringなし)
- Top-level symbol count: 7

#### Class `GoogleAuthenticationError`

- source line: 21
- bases: `Exception`
- doc: Raised when Google OAuth authentication cannot be completed.

#### Function `normalize_scopes(scopes: Iterable[str])`

- source line: 25
- kind: function
- doc: (docstringなし)

#### Function `_token_scopes(token_file: Path)`

- source line: 34
- kind: function
- doc: (docstringなし)

#### Function `token_has_scopes(token_file: Path, required_scopes: Iterable[str])`

- source line: 58
- kind: function
- doc: (docstringなし)

#### Function `_save_credentials(credentials: Credentials, token_file: Path)`

- source line: 69
- kind: function
- doc: (docstringなし)

#### Function `_load_cached_credentials(token_file: Path, required_scopes: tuple[str, ...])`

- source line: 90
- kind: function
- doc: (docstringなし)

#### Function `get_google_oauth_credentials(*, client_secrets_file: Path, token_file: Path, required_scopes: Iterable[str])`

- source line: 111
- kind: function
- doc: (docstringなし)

### 61.24 `src/web_intelligence/common/page_keys.py`

- Module summary: (module docstringなし)
- Top-level symbol count: 3

#### Class `NormalizedPage`

- source line: 10
- bases: `(none)`
- doc: (docstringなし)

#### Function `page_normalization_policy()`

- source line: 18
- kind: function
- doc: Return the shared GA4/GSC page identity contract for manifests.

#### Function `normalize_page(value: str, *, strip_query: bool=True, drop_fragment: bool=True)`

- source line: 38
- kind: function
- doc: Normalize a page identity without overwriting the raw source value.

### 61.25 `src/web_intelligence/common/runs.py`

- Module summary: (module docstringなし)
- Top-level symbol count: 10

#### Class `RunDiscoveryError`

- source line: 13
- bases: `Exception`
- doc: Raised when a requested source run cannot be discovered safely.

#### Function `_require_aware(value: datetime)`

- source line: 17
- kind: function
- doc: (docstringなし)

#### Function `make_run_id(prefix: str, started_at: datetime)`

- source line: 23
- kind: function
- doc: (docstringなし)

#### Function `ga4_extraction_run_id(started_at: datetime)`

- source line: 30
- kind: function
- doc: (docstringなし)

#### Function `ga4_processing_run_id(started_at: datetime)`

- source line: 34
- kind: function
- doc: (docstringなし)

#### Function `_timestamp_token_from_run_id(run_id: str)`

- source line: 38
- kind: function
- doc: (docstringなし)

#### Function `parse_run_timestamp(run_id: str)`

- source line: 48
- kind: function
- doc: Parse both legacy bare timestamps and namespaced run IDs.

#### Function `_read_manifest_started_at(path: Path)`

- source line: 76
- kind: function
- doc: (docstringなし)

#### Function `_run_sort_key(manifest_path: Path)`

- source line: 96
- kind: function
- doc: (docstringなし)

#### Function `find_latest_extraction_run(extraction_root: str | Path, property_id: str | None=None)`

- source line: 102
- kind: function
- doc: Find the latest extraction run across legacy and namespaced layouts.

### 61.26 `src/web_intelligence/extraction/__init__.py`

- Module summary: Read-only source extraction adapters.
- Top-level symbol count: 0

### 61.27 `src/web_intelligence/extraction/ga4/__init__.py`

- Module summary: Google Analytics 4 extraction adapter.
- Top-level symbol count: 0

### 61.28 `src/web_intelligence/extraction/ga4/admin.py`

- Module summary: (module docstringなし)
- Top-level symbol count: 4

#### Class `AnalyticsAdminError`

- source line: 10
- bases: `Exception`
- doc: Raised when Google Analytics Admin API access fails.

#### Class `PropertySummary`

- source line: 15
- bases: `(none)`
- doc: (docstringなし)

#### Function `_extract_resource_id(resource_name: str)`

- source line: 23
- kind: function
- doc: (docstringなし)

#### Function `iter_property_summaries(credentials: Credentials)`

- source line: 30
- kind: function
- doc: (docstringなし)

### 61.29 `src/web_intelligence/extraction/ga4/auth.py`

- Module summary: (module docstringなし)
- Top-level symbol count: 2

#### Class `AuthenticationError`

- source line: 20
- bases: `Exception`
- doc: Raised when GA4 authentication cannot be completed.

#### Function `get_credentials(profile: Profile)`

- source line: 24
- kind: function
- doc: (docstringなし)

### 61.30 `src/web_intelligence/extraction/ga4/cli.py`

- Module summary: (module docstringなし)
- Top-level symbol count: 16

#### Function `_configure_logging(level_name: str)`

- source line: 51
- kind: function
- doc: (docstringなし)

#### Function `_sha256_file(path: Path)`

- source line: 62
- kind: function
- doc: (docstringなし)

#### Function `_package_version(name: str)`

- source line: 70
- kind: function
- doc: (docstringなし)

#### Function `_validate_ga_date(value: str)`

- source line: 77
- kind: function
- doc: (docstringなし)

#### Function `_validate_date_pair(start_date: str, end_date: str)`

- source line: 90
- kind: function
- doc: (docstringなし)

#### Function `_all_reports(config: ReportsConfig)`

- source line: 104
- kind: function
- doc: (docstringなし)

#### Function `_select_reports(reports: tuple[ReportConfig, ...], requested_names: list[str] | None)`

- source line: 126
- kind: function
- doc: (docstringなし)

#### Function `_write_schema_csvs(metadata: PropertyMetadata, run_directory: Path)`

- source line: 142
- kind: function
- doc: (docstringなし)

#### Function `_filter_manifest_value(filter_config: StringFilterConfig | None)`

- source line: 205
- kind: function
- doc: (docstringなし)

#### Function `_order_manifest_value(order_bys: tuple[OrderByConfig, ...])`

- source line: 220
- kind: function
- doc: (docstringなし)

#### Function `_exported_manifest_entry(prepared: PreparedReport, result: ReportExportResult)`

- source line: 231
- kind: function
- doc: (docstringなし)

#### Function `_skipped_manifest_entry(skipped: SkippedReport)`

- source line: 262
- kind: function
- doc: (docstringなし)

#### Function `_failed_manifest_entry(prepared: PreparedReport, exc: Exception)`

- source line: 278
- kind: function
- doc: (docstringなし)

#### Function `_run_status(report_entries: list[dict])`

- source line: 297
- kind: function
- doc: (docstringなし)

#### Function `_build_parser()`

- source line: 313
- kind: function
- doc: (docstringなし)

#### Function `main(argv: list[str] | None=None)`

- source line: 348
- kind: function
- doc: (docstringなし)

### 61.31 `src/web_intelligence/extraction/ga4/config.py`

- Module summary: (module docstringなし)
- Top-level symbol count: 15

#### Class `ConfigError`

- source line: 10
- bases: `Exception`
- doc: Raised when a configuration file is invalid.

#### Class `OAuthConfig`

- source line: 15
- bases: `(none)`
- doc: (docstringなし)

#### Class `Profile`

- source line: 21
- bases: `(none)`
- doc: (docstringなし)

#### Class `StringFilterConfig`

- source line: 28
- bases: `(none)`
- doc: (docstringなし)

#### Class `OrderByConfig`

- source line: 36
- bases: `(none)`
- doc: (docstringなし)

#### Class `ReportConfig`

- source line: 43
- bases: `(none)`
- doc: (docstringなし)

#### Class `ReportsConfig`

- source line: 52
- bases: `(none)`
- doc: (docstringなし)

#### Function `_load_yaml(path: Path)`

- source line: 70
- kind: function
- doc: (docstringなし)

#### Function `_resolve_path(config_file: Path, value: str)`

- source line: 89
- kind: function
- doc: (docstringなし)

#### Function `_parse_string_list(value: Any, field_name: str)`

- source line: 96
- kind: function
- doc: (docstringなし)

#### Function `_parse_dimension_filter(value: Any, field_name: str)`

- source line: 115
- kind: function
- doc: (docstringなし)

#### Function `_parse_order_bys(value: Any, field_name: str)`

- source line: 157
- kind: function
- doc: (docstringなし)

#### Function `effective_order_bys(report: ReportConfig)`

- source line: 194
- kind: function
- doc: (docstringなし)

#### Function `load_profile(config_file: str | Path, profile_name: str='default')`

- source line: 208
- kind: function
- doc: (docstringなし)

#### Function `load_reports_config(config_file: str | Path)`

- source line: 258
- kind: function
- doc: (docstringなし)

### 61.32 `src/web_intelligence/extraction/ga4/data.py`

- Module summary: (module docstringなし)
- Top-level symbol count: 11

#### Class `AnalyticsDataError`

- source line: 46
- bases: `Exception`
- doc: Raised when Google Analytics Data API access fails.

#### Class `DimensionMetadata`

- source line: 51
- bases: `(none)`
- doc: (docstringなし)

#### Class `MetricMetadata`

- source line: 61
- bases: `(none)`
- doc: (docstringなし)

#### Class `PropertyMetadata`

- source line: 74
- bases: `(none)`
- doc: (docstringなし)

#### Class `ReportRow`

- source line: 81
- bases: `(none)`
- doc: (docstringなし)

#### Class `ReportPage`

- source line: 87
- bases: `(none)`
- doc: (docstringなし)

#### Function `_message_to_dict(message: Any)`

- source line: 99
- kind: function
- doc: (docstringなし)

#### Function `_build_dimension_filter(config: StringFilterConfig | None)`

- source line: 107
- kind: function
- doc: (docstringなし)

#### Function `_build_order_bys(report: ReportConfig)`

- source line: 125
- kind: function
- doc: (docstringなし)

#### Function `_build_order_by(config: OrderByConfig)`

- source line: 134
- kind: function
- doc: (docstringなし)

#### Class `GA4DataSource`

- source line: 152
- bases: `(none)`
- doc: (docstringなし)
- methods:
  - `__init__(self, credentials: Credentials)` — line 153 — (docstringなし)
  - `get_property_metadata(self, property_id: str)` — line 156 — (docstringなし)
  - `check_report_compatibility(self, property_id: str, report: ReportConfig)` — line 203 — (docstringなし)
  - `run_report_page(self, property_id: str, report: ReportConfig, start_date: str, end_date: str, *, limit: int, offset: int)` — line 226 — (docstringなし)

### 61.33 `src/web_intelligence/extraction/ga4/exporter.py`

- Module summary: (module docstringなし)
- Top-level symbol count: 5

#### Class `ExtractionError`

- source line: 17
- bases: `Exception`
- doc: Raised when report extraction cannot complete safely.

#### Class `ReportExportResult`

- source line: 22
- bases: `(none)`
- doc: (docstringなし)

#### Function `_report_file_name(report_name: str)`

- source line: 34
- kind: function
- doc: (docstringなし)

#### Function `export_report_to_csv(source: GA4DataSource, property_id: str, report: ReportConfig, start_date: str, end_date: str, output_directory: Path, *, page_size: int=250000)`

- source line: 52
- kind: function
- doc: (docstringなし)

#### Function `write_json_atomic(path: Path, data: dict[str, Any])`

- source line: 180
- kind: function
- doc: (docstringなし)

### 61.34 `src/web_intelligence/extraction/ga4/planning.py`

- Module summary: (module docstringなし)
- Top-level symbol count: 3

#### Class `PreparedReport`

- source line: 14
- bases: `(none)`
- doc: (docstringなし)

#### Class `SkippedReport`

- source line: 22
- bases: `(none)`
- doc: (docstringなし)

#### Function `prepare_report(source: GA4DataSource, property_id: str, metadata: PropertyMetadata, report: ReportConfig)`

- source line: 29
- kind: function
- doc: (docstringなし)

### 61.35 `src/web_intelligence/extraction/ga4/presets.py`

- Module summary: (module docstringなし)
- Top-level symbol count: 2

#### Function `_event_filter(event_name: str)`

- source line: 6
- kind: function
- doc: (docstringなし)

#### Function `get_preset(name: str)`

- source line: 270
- kind: function
- doc: (docstringなし)

### 61.36 `src/web_intelligence/extraction/ga4/quality.py`

- Module summary: (module docstringなし)
- Top-level symbol count: 4

#### Function `_active_schema_restrictions(metadata: dict[str, Any])`

- source line: 15
- kind: function
- doc: (docstringなし)

#### Function `build_warnings(pages: Iterable[dict[str, Any]])`

- source line: 27
- kind: function
- doc: (docstringなし)

#### Function `warning_reasons(warnings: dict[str, bool])`

- source line: 51
- kind: function
- doc: (docstringなし)

#### Function `exported_report_status(row_count: int, warnings: dict[str, bool], *, dropped_metrics: tuple[str, ...]=())`

- source line: 55
- kind: function
- doc: (docstringなし)

### 61.37 `src/web_intelligence/extraction/gsc/__init__.py`

- Module summary: Google Search Console extraction adapter.
- Top-level symbol count: 0

### 61.38 `src/web_intelligence/extraction/gsc/cli.py`

- Module summary: (module docstringなし)
- Top-level symbol count: 3

#### Function `_parser()`

- source line: 36
- kind: function
- doc: (docstringなし)

#### Function `_print_plan(config, site)`

- source line: 80
- kind: function
- doc: (docstringなし)

#### Function `main()`

- source line: 92
- kind: function
- doc: (docstringなし)

### 61.39 `src/web_intelligence/extraction/gsc/config.py`

- Module summary: (module docstringなし)
- Top-level symbol count: 6

#### Class `GscConfigurationError`

- source line: 21
- bases: `ValueError`
- doc: (docstringなし)

#### Class `GscExportConfig`

- source line: 26
- bases: `(none)`
- doc: (docstringなし)
- methods:
  - `to_manifest_dict(self)` — line 40 — (docstringなし)

#### Function `parse_search_types(value: str | list[str] | tuple[str, ...] | None)`

- source line: 49
- kind: function
- doc: (docstringなし)

#### Function `_deep_merge(base: dict[str, Any], override: dict[str, Any])`

- source line: 70
- kind: function
- doc: (docstringなし)

#### Function `_load_yaml(path: Path | None)`

- source line: 80
- kind: function
- doc: (docstringなし)

#### Function `build_config(*, config_path: Path | None, site_url: str | None=None, start: str | None=None, end: str | None=None, preset: str | None=None, search_types: str | None=None, data_state: str | None=None, aggregation_type: str | None=None, row_limit: int | None=None, max_retries: int | None=None, output_root: Path | None=None, client_secrets_file: Path | None=None, token_file: Path | None=None)`

- source line: 97
- kind: function
- doc: (docstringなし)

### 61.40 `src/web_intelligence/extraction/gsc/data.py`

- Module summary: (module docstringなし)
- Top-level symbol count: 11

#### Class `SearchConsoleApiError`

- source line: 13
- bases: `RuntimeError`
- doc: (docstringなし)
- methods:
  - `__init__(self, message: str, *, status_code: int, payload: Any=None)` — line 14 — (docstringなし)

#### Class `RequestStats`

- source line: 27
- bases: `(none)`
- doc: (docstringなし)

#### Class `QueryPage`

- source line: 33
- bases: `(none)`
- doc: (docstringなし)

#### Function `_decode_response(response: Any)`

- source line: 39
- kind: function
- doc: (docstringなし)

#### Function `_safe_error_message(payload: Any, status_code: int)`

- source line: 53
- kind: function
- doc: (docstringなし)

#### Function `request_json(session: Any, method: str, url: str, *, json_body: dict[str, Any] | None=None, max_retries: int, stats: RequestStats, timeout: int=60, sleep_fn=time.sleep)`

- source line: 63
- kind: function
- doc: (docstringなし)

#### Function `sites_url()`

- source line: 105
- kind: function
- doc: (docstringなし)

#### Function `search_analytics_url(site_url: str)`

- source line: 109
- kind: function
- doc: (docstringなし)

#### Function `build_query_body(*, start_date: str, end_date: str, dimensions: tuple[str, ...], search_type: str, data_state: str, aggregation_type: str, row_limit: int, start_row: int)`

- source line: 117
- kind: function
- doc: (docstringなし)

#### Function `query_once(session: Any, *, site_url: str, start_date: str, end_date: str, dimensions: tuple[str, ...], search_type: str, data_state: str, aggregation_type: str, row_limit: int, start_row: int, max_retries: int, stats: RequestStats)`

- source line: 142
- kind: function
- doc: (docstringなし)

#### Function `iter_query_pages(session: Any, *, site_url: str, start_date: str, end_date: str, dimensions: tuple[str, ...], search_type: str, data_state: str, aggregation_type: str, row_limit: int, max_retries: int, stats: RequestStats, max_pages: int=1000)`

- source line: 189
- kind: function
- doc: (docstringなし)

### 61.41 `src/web_intelligence/extraction/gsc/discovery.py`

- Module summary: (module docstringなし)
- Top-level symbol count: 4

#### Class `SiteSelectionError`

- source line: 13
- bases: `ValueError`
- doc: (docstringなし)

#### Class `SiteEntry`

- source line: 18
- bases: `(none)`
- doc: (docstringなし)

#### Function `list_sites(session: Any, *, max_retries: int, stats: RequestStats)`

- source line: 23
- kind: function
- doc: (docstringなし)

#### Function `resolve_site(requested_site: str | None, sites: tuple[SiteEntry, ...])`

- source line: 56
- kind: function
- doc: (docstringなし)

### 61.42 `src/web_intelligence/extraction/gsc/exporter.py`

- Module summary: (module docstringなし)
- Top-level symbol count: 25

#### Function `_tool_version()`

- source line: 44
- kind: function
- doc: (docstringなし)

#### Function `_utc_now()`

- source line: 51
- kind: function
- doc: (docstringなし)

#### Function `_run_id(now: datetime)`

- source line: 55
- kind: function
- doc: (docstringなし)

#### Function `site_key(site_url: str)`

- source line: 59
- kind: function
- doc: (docstringなし)

#### Function `_sha256_json(payload: dict[str, Any])`

- source line: 70
- kind: function
- doc: (docstringなし)

#### Function `_json_safe(value: Any)`

- source line: 81
- kind: function
- doc: (docstringなし)

#### Function `_write_json_atomic(path: Path, payload: dict[str, Any])`

- source line: 93
- kind: function
- doc: (docstringなし)

#### Class `_CsvWriter`

- source line: 109
- bases: `(none)`
- doc: (docstringなし)
- methods:
  - `__init__(self, path: Path, fieldnames: list[str])` — line 110 — (docstringなし)
  - `write(self, row: dict[str, str])` — line 123 — (docstringなし)
  - `close(self)` — line 127 — (docstringなし)
  - `abort(self)` — line 133 — (docstringなし)

#### Function `_scalar_text(value: Any)`

- source line: 140
- kind: function
- doc: (docstringなし)

#### Function `_decimal(value: Any)`

- source line: 146
- kind: function
- doc: (docstringなし)

#### Function `page_has_observed_data(page: QueryPage)`

- source line: 153
- kind: function
- doc: (docstringなし)

#### Function `_row_to_csv(row: dict[str, Any], dimensions: tuple[str, ...])`

- source line: 162
- kind: function
- doc: (docstringなし)

#### Function `_iter_dates(start: date, end: date)`

- source line: 177
- kind: function
- doc: (docstringなし)

#### Function `_metadata_value(metadata: dict[str, Any], *names: str)`

- source line: 184
- kind: function
- doc: (docstringなし)

#### Function `_collect_page_metadata(page: QueryPage, aggregation_types: set[str], incomplete_dates: set[str], incomplete_hours: set[str])`

- source line: 191
- kind: function
- doc: (docstringなし)

#### Function `_report_output_path(run_dir: Path, search_type: str, report_name: str)`

- source line: 215
- kind: function
- doc: (docstringなし)

#### Function `_date_strings(start: date, end: date)`

- source line: 223
- kind: function
- doc: (docstringなし)

#### Function `_observed_date_coverage_from_csv(path: Path, *, requested_start: date, requested_end: date)`

- source line: 227
- kind: function
- doc: (docstringなし)

#### Function `_classify_observed_coverage(*, observed: dict[str, Any] | None)`

- source line: 263
- kind: function
- doc: (docstringなし)

#### Function `_build_search_type_coverage(*, run_dir: Path, resolved: ResolvedDateRange, probes: list[dict[str, Any]], reports: list[dict[str, Any]])`

- source line: 279
- kind: function
- doc: (docstringなし)

#### Function `_preflight_report(session: Any, *, config: GscExportConfig, site: SiteEntry, date_range: ResolvedDateRange, spec: SearchReportSpec, search_type: str, stats: RequestStats)`

- source line: 343
- kind: function
- doc: (docstringなし)

#### Function `_planned_skip(spec: SearchReportSpec, search_type: str, reason: str)`

- source line: 369
- kind: function
- doc: (docstringなし)

#### Function `_export_report(session: Any, *, config: GscExportConfig, site: SiteEntry, date_range: ResolvedDateRange, run_dir: Path, spec: SearchReportSpec, search_type: str, stats: RequestStats)`

- source line: 396
- kind: function
- doc: (docstringなし)

#### Function `_probe_search_type(session: Any, *, config: GscExportConfig, site: SiteEntry, date_range: ResolvedDateRange, search_type: str, stats: RequestStats)`

- source line: 555
- kind: function
- doc: (docstringなし)

#### Function `extract_search_console(session: Any, *, config: GscExportConfig, site: SiteEntry)`

- source line: 610
- kind: function
- doc: (docstringなし)

### 61.43 `src/web_intelligence/extraction/gsc/planning.py`

- Module summary: (module docstringなし)
- Top-level symbol count: 2

#### Class `PlanDecision`

- source line: 9
- bases: `(none)`
- doc: (docstringなし)

#### Function `plan_report(spec: SearchReportSpec, search_type: str)`

- source line: 14
- kind: function
- doc: (docstringなし)

### 61.44 `src/web_intelligence/extraction/gsc/presets.py`

- Module summary: (module docstringなし)
- Top-level symbol count: 3

#### Class `SearchReportSpec`

- source line: 7
- bases: `(none)`
- doc: (docstringなし)

#### Function `get_preset(name: str)`

- source line: 87
- kind: function
- doc: (docstringなし)

#### Function `preset_names()`

- source line: 95
- kind: function
- doc: (docstringなし)

### 61.45 `src/web_intelligence/integration/__init__.py`

- Module summary: Cross-source integration layers.
- Top-level symbol count: 0

### 61.46 `src/web_intelligence/integration/analysis/__init__.py`

- Module summary: Generic Integration Analysis engine.
- Top-level symbol count: 0

### 61.47 `src/web_intelligence/integration/analysis/cli.py`

- Module summary: (module docstringなし)
- Top-level symbol count: 3

#### Function `build_parser()`

- source line: 24
- kind: function
- doc: (docstringなし)

#### Function `_configure_logging(level_name: str)`

- source line: 57
- kind: function
- doc: (docstringなし)

#### Function `main(argv: list[str] | None=None)`

- source line: 64
- kind: function
- doc: (docstringなし)

### 61.48 `src/web_intelligence/integration/analysis/config.py`

- Module summary: (module docstringなし)
- Top-level symbol count: 18

#### Class `IntegrationAnalysisConfigError`

- source line: 13
- bases: `ValueError`
- doc: Raised when an Integration Analysis spec is invalid.

#### Class `CoverageSpec`

- source line: 24
- bases: `(none)`
- doc: (docstringなし)

#### Class `QualitySpec`

- source line: 30
- bases: `(none)`
- doc: (docstringなし)

#### Class `SafeAverageSpec`

- source line: 35
- bases: `(none)`
- doc: (docstringなし)

#### Class `ComparisonSpec`

- source line: 41
- bases: `(none)`
- doc: (docstringなし)

#### Class `TrendSpec`

- source line: 47
- bases: `(none)`
- doc: (docstringなし)

#### Class `RankingSpec`

- source line: 53
- bases: `(none)`
- doc: (docstringなし)

#### Class `ContributionSpec`

- source line: 61
- bases: `(none)`
- doc: (docstringなし)

#### Class `OperationsSpec`

- source line: 70
- bases: `(none)`
- doc: (docstringなし)

#### Class `IntegrationAnalysisSpec`

- source line: 81
- bases: `(none)`
- doc: (docstringなし)

#### Function `_mapping(value: Any, field: str)`

- source line: 91
- kind: function
- doc: (docstringなし)

#### Function `_string(value: Any, field: str)`

- source line: 97
- kind: function
- doc: (docstringなし)

#### Function `_positive_int(value: Any, field: str)`

- source line: 103
- kind: function
- doc: (docstringなし)

#### Function `_bool(value: Any, field: str)`

- source line: 109
- kind: function
- doc: (docstringなし)

#### Function `_string_tuple(value: Any, field: str, *, allow_star: bool=False)`

- source line: 115
- kind: function
- doc: (docstringなし)

#### Function `load_spec(path: str | Path)`

- source line: 133
- kind: function
- doc: (docstringなし)

#### Function `spec_payload(spec: IntegrationAnalysisSpec)`

- source line: 295
- kind: function
- doc: (docstringなし)

#### Function `spec_sha256(spec: IntegrationAnalysisSpec)`

- source line: 299
- kind: function
- doc: (docstringなし)

### 61.49 `src/web_intelligence/integration/analysis/discovery.py`

- Module summary: (module docstringなし)
- Top-level symbol count: 3

#### Class `IntegrationAnalysisRunDiscoveryError`

- source line: 8
- bases: `RuntimeError`
- doc: Raised when an Integration Processing run cannot be discovered safely.

#### Function `_started_at(manifest_path: Path)`

- source line: 12
- kind: function
- doc: (docstringなし)

#### Function `find_latest_integration_processing_run(root: str | Path, *, site_key: str | None=None)`

- source line: 31
- kind: function
- doc: (docstringなし)

### 61.50 `src/web_intelligence/integration/analysis/processor.py`

- Module summary: (module docstringなし)
- Top-level symbol count: 45

#### Class `IntegrationAnalysisError`

- source line: 47
- bases: `RuntimeError`
- doc: Raised when Integration Processing output cannot be analyzed safely.

#### Class `MetricDef`

- source line: 52
- bases: `(none)`
- doc: (docstringなし)

#### Class `ValidatedAnalysisInput`

- source line: 67
- bases: `(none)`
- doc: (docstringなし)

#### Class `WindowDef`

- source line: 89
- bases: `(none)`
- doc: (docstringなし)
- methods:
  - `date_count(self)` — line 95 — (docstringなし)

#### Class `AggregateResult`

- source line: 100
- bases: `(none)`
- doc: (docstringなし)

#### Class `ResultArtifact`

- source line: 108
- bases: `(none)`
- doc: (docstringなし)

#### Function `_utc_now()`

- source line: 119
- kind: function
- doc: (docstringなし)

#### Function `_sha256_file(path: Path)`

- source line: 123
- kind: function
- doc: (docstringなし)

#### Function `_read_json(path: Path)`

- source line: 131
- kind: function
- doc: (docstringなし)

#### Function `_read_csv(path: Path)`

- source line: 141
- kind: function
- doc: (docstringなし)

#### Function `_portable_path(path: Path)`

- source line: 152
- kind: function
- doc: (docstringなし)

#### Function `_tool_version()`

- source line: 161
- kind: function
- doc: (docstringなし)

#### Function `_parse_date(raw: str, *, field: str)`

- source line: 174
- kind: function
- doc: (docstringなし)

#### Function `_date_text(value: date)`

- source line: 188
- kind: function
- doc: (docstringなし)

#### Function `_decimal(raw: str | None, *, field: str)`

- source line: 192
- kind: function
- doc: (docstringなし)

#### Function `_decimal_text(value: Decimal | None)`

- source line: 203
- kind: function
- doc: (docstringなし)

#### Function `_ratio_text(value: Decimal | None)`

- source line: 214
- kind: function
- doc: (docstringなし)

#### Function `_artifact_ref(manifest: Mapping[str, Any], name: str)`

- source line: 218
- kind: function
- doc: (docstringなし)

#### Function `_validate_artifact_hash(run_directory: Path, manifest: Mapping[str, Any], name: str)`

- source line: 242
- kind: function
- doc: (docstringなし)

#### Function `_parse_metrics(registry: Mapping[str, Any], fieldnames: set[str])`

- source line: 259
- kind: function
- doc: (docstringなし)

#### Function `validate_processing_input(input_run: str | Path, *, spec: IntegrationAnalysisSpec)`

- source line: 376
- kind: function
- doc: (docstringなし)

#### Function `_selected_window(validated: ValidatedAnalysisInput)`

- source line: 540
- kind: function
- doc: (docstringなし)

#### Function `_comparison_windows(validated: ValidatedAnalysisInput)`

- source line: 549
- kind: function
- doc: (docstringなし)

#### Function `_rows_for_window(rows: Sequence[Mapping[str, str]], *, date_column: str, window: WindowDef)`

- source line: 572
- kind: function
- doc: (docstringなし)

#### Function `_source_coverage(validated: ValidatedAnalysisInput, metric: MetricDef, window: WindowDef, spec: IntegrationAnalysisSpec)`

- source line: 587
- kind: function
- doc: (docstringなし)

#### Function `_sum_metric(rows: Sequence[Mapping[str, str]], metric: MetricDef)`

- source line: 630
- kind: function
- doc: (docstringなし)

#### Function `_aggregate_value(rows: Sequence[Mapping[str, str]], metric: MetricDef, by_id: Mapping[str, MetricDef], *, coverage_ok: bool, coverage_ratio: Decimal | None)`

- source line: 644
- kind: function
- doc: (docstringなし)

#### Function `_period_total_result(rows: Sequence[Mapping[str, str]], metric: MetricDef, *, coverage_ok: bool, coverage_ratio: Decimal | None)`

- source line: 746
- kind: function
- doc: (docstringなし)

#### Function `_safe_average_result(rows: Sequence[Mapping[str, str]], metric: MetricDef, by_id: Mapping[str, MetricDef], *, coverage_ok: bool, coverage_ratio: Decimal | None, date_count: int)`

- source line: 783
- kind: function
- doc: (docstringなし)

#### Function `_comparison_status(current: AggregateResult, baseline: AggregateResult)`

- source line: 830
- kind: function
- doc: (docstringなし)

#### Function `_metric_metadata(metric: MetricDef)`

- source line: 847
- kind: function
- doc: (docstringなし)

#### Function `_build_period_metrics(validated: ValidatedAnalysisInput, spec: IntegrationAnalysisSpec, windows: Mapping[str, WindowDef])`

- source line: 860
- kind: function
- doc: (docstringなし)

#### Function `_build_comparisons(validated: ValidatedAnalysisInput, spec: IntegrationAnalysisSpec, windows: Mapping[str, WindowDef])`

- source line: 931
- kind: function
- doc: (docstringなし)

#### Function `_build_trend(validated: ValidatedAnalysisInput, spec: IntegrationAnalysisSpec, windows: Mapping[str, WindowDef])`

- source line: 1032
- kind: function
- doc: (docstringなし)

#### Function `_group_rows(rows: Sequence[Mapping[str, str]], group_by: Sequence[str])`

- source line: 1082
- kind: function
- doc: (docstringなし)

#### Function `_window_for_name(windows: Mapping[str, WindowDef], name: str)`

- source line: 1091
- kind: function
- doc: (docstringなし)

#### Function `_ranking_sort_key(item: tuple[tuple[str, ...], AggregateResult])`

- source line: 1098
- kind: function
- doc: (docstringなし)

#### Function `_build_ranking(validated: ValidatedAnalysisInput, spec: IntegrationAnalysisSpec, windows: Mapping[str, WindowDef])`

- source line: 1105
- kind: function
- doc: (docstringなし)

#### Function `_build_contribution(validated: ValidatedAnalysisInput, spec: IntegrationAnalysisSpec, windows: Mapping[str, WindowDef])`

- source line: 1159
- kind: function
- doc: (docstringなし)

#### Function `_build_coverage_status(validated: ValidatedAnalysisInput, spec: IntegrationAnalysisSpec, windows: Mapping[str, WindowDef])`

- source line: 1236
- kind: function
- doc: (docstringなし)

#### Function `_status_counts(rows: Iterable[Mapping[str, str]])`

- source line: 1269
- kind: function
- doc: (docstringなし)

#### Function `_write_result_csv(output_dir: Path, *, result_id: str, filename: str, fieldnames: list[str], rows: list[dict[str, str]], grain: tuple[str, ...], operation: str)`

- source line: 1274
- kind: function
- doc: (docstringなし)

#### Function `_result_catalog(validated: ValidatedAnalysisInput, spec: IntegrationAnalysisSpec, results: Sequence[ResultArtifact], status_by_result: Mapping[str, Mapping[str, int]])`

- source line: 1296
- kind: function
- doc: (docstringなし)

#### Function `_analysis_quality(validated: ValidatedAnalysisInput, result_rows: Mapping[str, Sequence[Mapping[str, str]]])`

- source line: 1327
- kind: function
- doc: (docstringなし)

#### Function `analyze_integration(*, input_run: str | Path, output_root: str | Path, spec: IntegrationAnalysisSpec, spec_path: str | Path, started_at: datetime | None=None)`

- source line: 1356
- kind: function
- doc: (docstringなし)

### 61.51 `src/web_intelligence/integration/extraction/__init__.py`

- Module summary: Lossless Integration Extraction for Source Processing artifacts.
- Top-level symbol count: 0

### 61.52 `src/web_intelligence/integration/extraction/cli.py`

- Module summary: (module docstringなし)
- Top-level symbol count: 3

#### Function `build_parser()`

- source line: 24
- kind: function
- doc: (docstringなし)

#### Function `_configure_logging(level_name: str)`

- source line: 68
- kind: function
- doc: (docstringなし)

#### Function `main(argv: list[str] | None=None)`

- source line: 75
- kind: function
- doc: (docstringなし)

### 61.53 `src/web_intelligence/integration/extraction/config.py`

- Module summary: (module docstringなし)
- Top-level symbol count: 7

#### Class `IntegrationExtractionConfigError`

- source line: 12
- bases: `ValueError`
- doc: Raised when Integration Extraction config is invalid.

#### Class `SiteMapping`

- source line: 17
- bases: `(none)`
- doc: (docstringなし)

#### Class `IntegrationExtractionConfig`

- source line: 25
- bases: `(none)`
- doc: (docstringなし)

#### Function `_require_string(payload: dict[str, Any], key: str)`

- source line: 32
- kind: function
- doc: (docstringなし)

#### Function `load_config(path: str | Path)`

- source line: 39
- kind: function
- doc: (docstringなし)

#### Function `config_payload(config: IntegrationExtractionConfig)`

- source line: 86
- kind: function
- doc: (docstringなし)

#### Function `config_sha256(config: IntegrationExtractionConfig)`

- source line: 102
- kind: function
- doc: (docstringなし)

### 61.54 `src/web_intelligence/integration/extraction/discovery.py`

- Module summary: (module docstringなし)
- Top-level symbol count: 3

#### Class `IntegrationRunDiscoveryError`

- source line: 8
- bases: `RuntimeError`
- doc: Raised when a Processing run cannot be discovered safely.

#### Function `_started_at(manifest_path: Path)`

- source line: 12
- kind: function
- doc: (docstringなし)

#### Function `find_latest_processing_run(root: str | Path, *, source_identity: str, run_prefix: str)`

- source line: 31
- kind: function
- doc: (docstringなし)

### 61.55 `src/web_intelligence/integration/extraction/processor.py`

- Module summary: (module docstringなし)
- Top-level symbol count: 39

#### Class `IntegrationExtractionError`

- source line: 45
- bases: `RuntimeError`
- doc: Raised when Processing artifacts cannot be integrated safely.

#### Class `ValidatedInput`

- source line: 50
- bases: `(none)`
- doc: (docstringなし)

#### Class `MetricAggregate`

- source line: 59
- bases: `(none)`
- doc: (docstringなし)
- methods:
  - `add(self, row: Mapping[str, str])` — line 65 — (docstringなし)
  - `ctr(self)` — line 76 — (docstringなし)
  - `position(self)` — line 80 — (docstringなし)

#### Class `Ga4Aggregate`

- source line: 87
- bases: `(none)`
- doc: (docstringなし)
- methods:
  - `add(self, row: Mapping[str, str])` — line 92 — (docstringなし)

#### Class `QualityCollector`

- source line: 99
- bases: `(none)`
- doc: (docstringなし)
- methods:
  - `add(self, category: str, metric: str, value: object, *, source: str='integration', status: str='OK', details: str='')` — line 103 — (docstringなし)

#### Function `_tool_version()`

- source line: 127
- kind: function
- doc: (docstringなし)

#### Function `_utc_now()`

- source line: 144
- kind: function
- doc: (docstringなし)

#### Function `_run_id(now: datetime)`

- source line: 148
- kind: function
- doc: (docstringなし)

#### Function `_sha256_file(path: Path)`

- source line: 152
- kind: function
- doc: (docstringなし)

#### Function `_portable_path(path: Path)`

- source line: 160
- kind: function
- doc: (docstringなし)

#### Function `_read_json(path: Path)`

- source line: 164
- kind: function
- doc: (docstringなし)

#### Function `_decimal(value: object)`

- source line: 174
- kind: function
- doc: (docstringなし)

#### Function `_decimal_text(value: Decimal | None)`

- source line: 183
- kind: function
- doc: (docstringなし)

#### Function `_iso_date(value: str, *, location: str)`

- source line: 191
- kind: function
- doc: (docstringなし)

#### Function `_read_csv(path: Path)`

- source line: 205
- kind: function
- doc: (docstringなし)

#### Function `_manifest_table(manifest: Mapping[str, Any], output_file: str)`

- source line: 215
- kind: function
- doc: (docstringなし)

#### Function `_validate_declared_csv(validated: ValidatedInput, output_file: str)`

- source line: 227
- kind: function
- doc: (docstringなし)

#### Function `_validate_url_policy(manifest: Mapping[str, Any], source: str)`

- source line: 247
- kind: function
- doc: (docstringなし)

#### Function `_validate_ga4(run_directory: Path, config: IntegrationExtractionConfig)`

- source line: 254
- kind: function
- doc: (docstringなし)

#### Function `_validate_gsc(run_directory: Path, config: IntegrationExtractionConfig)`

- source line: 292
- kind: function
- doc: (docstringなし)

#### Function `validate_inputs(*, ga4_run: str | Path, gsc_run: str | Path, config: IntegrationExtractionConfig)`

- source line: 331
- kind: function
- doc: (docstringなし)

#### Function `_validate_upstream_manifest_lineage(validated: ValidatedInput)`

- source line: 364
- kind: function
- doc: (docstringなし)

#### Function `_selected_input_datasets(ga4: ValidatedInput, gsc: ValidatedInput)`

- source line: 394
- kind: function
- doc: (docstringなし)

#### Function `_gsc_table_entries(manifest: Mapping[str, Any], report_name: str)`

- source line: 429
- kind: function
- doc: (docstringなし)

#### Function `_date_range(values: Iterable[str])`

- source line: 443
- kind: function
- doc: (docstringなし)

#### Function `_load_ga4_page_daily(ga4: ValidatedInput, *, site_key: str, quality: QualityCollector)`

- source line: 452
- kind: function
- doc: (docstringなし)

#### Function `_load_ga4_observed_dates(ga4: ValidatedInput)`

- source line: 507
- kind: function
- doc: (docstringなし)

#### Function `_load_gsc_daily(gsc: ValidatedInput, *, site_key: str, report_name: str, quality: QualityCollector)`

- source line: 517
- kind: function
- doc: (docstringなし)

#### Function `_gsc_observed_dates(manifest: Mapping[str, Any])`

- source line: 590
- kind: function
- doc: (docstringなし)

#### Function `_canonical_pages(*, site_key: str, ga4_variants: Mapping[str, set[str]], gsc_variants: Mapping[str, set[str]], gsc_hosts: Mapping[str, set[str]])`

- source line: 620
- kind: function
- doc: (docstringなし)

#### Function `_state_for_absent(date_value: str, observed_dates: set[str])`

- source line: 654
- kind: function
- doc: (docstringなし)

#### Function `_page_daily_integrated(*, site_key: str, ga4_aggregate: Mapping[tuple[str, str], Ga4Aggregate], gsc_aggregate: Mapping[tuple[str, ...], MetricAggregate], ga4_observed_dates: set[str], gsc_observed_dates_by_type: Mapping[str, set[str]])`

- source line: 658
- kind: function
- doc: (docstringなし)

#### Function `_page_integrated_row(*, site_key: str, date_value: str, key: str, search_type: str, join_state: str, ga4_state: str, gsc_state: str, ga4: Ga4Aggregate | None, gsc: MetricAggregate | None)`

- source line: 723
- kind: function
- doc: (docstringなし)

#### Function `_query_page_daily_integrated(*, site_key: str, ga4_aggregate: Mapping[tuple[str, str], Ga4Aggregate], gsc_query_aggregate: Mapping[tuple[str, ...], MetricAggregate], ga4_observed_dates: set[str])`

- source line: 760
- kind: function
- doc: (docstringなし)

#### Function `_duplicate_count(rows: Iterable[Mapping[str, str]], fields: tuple[str, ...])`

- source line: 808
- kind: function
- doc: (docstringなし)

#### Function `_dataset_entry(*, run_directory: Path, dataset_id: str, filename: str, row_count: int, grain: list[str], dimensions: list[str], metrics: dict[str, str], dates: Iterable[str]=(), notes: list[str] | None=None)`

- source line: 820
- kind: function
- doc: (docstringなし)

#### Function `_validate_catalog_contract(*, run_directory: Path, catalog: Mapping[str, Any])`

- source line: 847
- kind: function
- doc: (docstringなし)

#### Function `_write_dataset(run_directory: Path, filename: str, fieldnames: list[str], rows: list[dict[str, str]])`

- source line: 914
- kind: function
- doc: (docstringなし)

#### Function `extract_integration(*, ga4_run: str | Path, gsc_run: str | Path, output_root: str | Path, config: IntegrationExtractionConfig, config_path: str | Path | None=None, started_at: datetime | None=None)`

- source line: 923
- kind: function
- doc: (docstringなし)

### 61.56 `src/web_intelligence/integration/processing/__init__.py`

- Module summary: Config-driven Integration Processing.
- Top-level symbol count: 0

### 61.57 `src/web_intelligence/integration/processing/cli.py`

- Module summary: (module docstringなし)
- Top-level symbol count: 3

#### Function `build_parser()`

- source line: 24
- kind: function
- doc: (docstringなし)

#### Function `_configure_logging(level_name: str)`

- source line: 59
- kind: function
- doc: (docstringなし)

#### Function `main(argv: list[str] | None=None)`

- source line: 66
- kind: function
- doc: (docstringなし)

### 61.58 `src/web_intelligence/integration/processing/config.py`

- Module summary: (module docstringなし)
- Top-level symbol count: 21

#### Class `IntegrationProcessingConfigError`

- source line: 53
- bases: `ValueError`
- doc: Raised when Integration Processing config is invalid.

#### Class `UseCaseConfig`

- source line: 58
- bases: `(none)`
- doc: (docstringなし)

#### Class `InputConfig`

- source line: 64
- bases: `(none)`
- doc: (docstringなし)

#### Class `PeriodConfig`

- source line: 70
- bases: `(none)`
- doc: (docstringなし)

#### Class `SourceRequirementConfig`

- source line: 79
- bases: `(none)`
- doc: (docstringなし)

#### Class `MetricConfig`

- source line: 85
- bases: `(none)`
- doc: (docstringなし)

#### Class `FilterConfig`

- source line: 104
- bases: `(none)`
- doc: (docstringなし)

#### Class `CoverageConfig`

- source line: 114
- bases: `(none)`
- doc: (docstringなし)

#### Class `ComparisonConfig`

- source line: 120
- bases: `(none)`
- doc: (docstringなし)

#### Class `IntegrationProcessingConfig`

- source line: 127
- bases: `(none)`
- doc: (docstringなし)

#### Function `_require_mapping(value: Any, field: str)`

- source line: 143
- kind: function
- doc: (docstringなし)

#### Function `_require_string(value: Any, field: str)`

- source line: 149
- kind: function
- doc: (docstringなし)

#### Function `_optional_string(value: Any, field: str)`

- source line: 155
- kind: function
- doc: (docstringなし)

#### Function `_require_positive_int(value: Any, field: str)`

- source line: 161
- kind: function
- doc: (docstringなし)

#### Function `_string_tuple(value: Any, field: str, *, allow_empty: bool)`

- source line: 167
- kind: function
- doc: (docstringなし)

#### Function `_parse_period(payload: Any)`

- source line: 185
- kind: function
- doc: (docstringなし)

#### Function `_parse_metrics(value: Any)`

- source line: 214
- kind: function
- doc: (docstringなし)

#### Function `_parse_filters(value: Any)`

- source line: 318
- kind: function
- doc: (docstringなし)

#### Function `load_config(path: str | Path)`

- source line: 369
- kind: function
- doc: (docstringなし)

#### Function `config_payload(config: IntegrationProcessingConfig)`

- source line: 481
- kind: function
- doc: (docstringなし)

#### Function `config_sha256(config: IntegrationProcessingConfig)`

- source line: 485
- kind: function
- doc: (docstringなし)

### 61.59 `src/web_intelligence/integration/processing/discovery.py`

- Module summary: (module docstringなし)
- Top-level symbol count: 3

#### Class `IntegrationProcessingRunDiscoveryError`

- source line: 8
- bases: `RuntimeError`
- doc: Raised when an Integration Extraction run cannot be discovered safely.

#### Function `_started_at(manifest_path: Path)`

- source line: 12
- kind: function
- doc: (docstringなし)

#### Function `find_latest_integration_extraction_run(root: str | Path, *, site_key: str | None=None)`

- source line: 31
- kind: function
- doc: (docstringなし)

### 61.60 `src/web_intelligence/integration/processing/processor.py`

- Module summary: (module docstringなし)
- Top-level symbol count: 35

#### Class `IntegrationProcessingError`

- source line: 34
- bases: `RuntimeError`
- doc: Raised when an Integration Extraction artifact cannot be processed safely.

#### Class `ValidatedIntegrationInput`

- source line: 39
- bases: `(none)`
- doc: (docstringなし)

#### Class `ResolvedPeriod`

- source line: 53
- bases: `(none)`
- doc: (docstringなし)

#### Class `SourceCoverage`

- source line: 63
- bases: `(none)`
- doc: (docstringなし)

#### Function `_utc_now()`

- source line: 72
- kind: function
- doc: (docstringなし)

#### Function `_sha256_file(path: Path)`

- source line: 76
- kind: function
- doc: (docstringなし)

#### Function `_read_json(path: Path)`

- source line: 84
- kind: function
- doc: (docstringなし)

#### Function `_read_csv(path: Path)`

- source line: 94
- kind: function
- doc: (docstringなし)

#### Function `_portable_path(path: Path)`

- source line: 106
- kind: function
- doc: (docstringなし)

#### Function `_tool_version()`

- source line: 115
- kind: function
- doc: (docstringなし)

#### Function `_parse_date(value: str, *, field: str)`

- source line: 129
- kind: function
- doc: (docstringなし)

#### Function `_date_range(start: date, end: date)`

- source line: 143
- kind: function
- doc: (docstringなし)

#### Function `_date_summary(values: Iterable[date])`

- source line: 151
- kind: function
- doc: (docstringなし)

#### Function `_decimal(value: str | None, *, field: str)`

- source line: 162
- kind: function
- doc: (docstringなし)

#### Function `_decimal_text(value: Decimal | None)`

- source line: 173
- kind: function
- doc: (docstringなし)

#### Function `_manifest_dataset_entry(manifest: Mapping[str, Any], dataset_id: str)`

- source line: 184
- kind: function
- doc: (docstringなし)

#### Function `_catalog_dataset_entry(catalog: Mapping[str, Any], dataset_id: str)`

- source line: 200
- kind: function
- doc: (docstringなし)

#### Function `validate_integration_input(input_run: str | Path, *, config: IntegrationProcessingConfig)`

- source line: 218
- kind: function
- doc: (docstringなし)

#### Function `_source_coverages(manifest: Mapping[str, Any])`

- source line: 400
- kind: function
- doc: (docstringなし)

#### Function `_shift_year(value: date, years: int=-1)`

- source line: 457
- kind: function
- doc: (docstringなし)

#### Function `_window(start: date, days: int)`

- source line: 465
- kind: function
- doc: (docstringなし)

#### Function `_comparison_windows(*, anchor_end: date, current_days: int, previous: bool, yoy: bool)`

- source line: 469
- kind: function
- doc: (docstringなし)

#### Function `_window_dates(windows: Mapping[str, tuple[date, date]])`

- source line: 483
- kind: function
- doc: (docstringなし)

#### Function `_resolve_period(config: IntegrationProcessingConfig, coverages: Mapping[str, SourceCoverage])`

- source line: 490
- kind: function
- doc: (docstringなし)

#### Function `_filter_matches(row: Mapping[str, str], spec: FilterConfig)`

- source line: 595
- kind: function
- doc: (docstringなし)

#### Function `_aggregate_metric(rows: list[dict[str, str]], metric: MetricConfig, metrics_by_id: Mapping[str, MetricConfig])`

- source line: 624
- kind: function
- doc: (docstringなし)

#### Function `_aggregate_rows(rows: list[dict[str, str]], config: IntegrationProcessingConfig)`

- source line: 727
- kind: function
- doc: (docstringなし)

#### Function `_config_file_sha256(path: Path | None)`

- source line: 757
- kind: function
- doc: (docstringなし)

#### Function `_output_aggregation_rule(metric: MetricConfig)`

- source line: 763
- kind: function
- doc: (docstringなし)

#### Function `_metric_registry(config: IntegrationProcessingConfig)`

- source line: 775
- kind: function
- doc: (docstringなし)

#### Function `_dataset_schema(*, config: IntegrationProcessingConfig, row_count: int, output_dates: set[date], coverage_summary: dict[str, Any])`

- source line: 807
- kind: function
- doc: (docstringなし)

#### Function `_coverage_summary(*, config: IntegrationProcessingConfig, period: ResolvedPeriod, coverages: Mapping[str, SourceCoverage], output_rows: list[dict[str, str]])`

- source line: 839
- kind: function
- doc: (docstringなし)

#### Function `_validate_coverage_policy(*, config: IntegrationProcessingConfig, summary: Mapping[str, Any])`

- source line: 894
- kind: function
- doc: (docstringなし)

#### Function `_validate_output_contract(*, run_directory: Path, config: IntegrationProcessingConfig, rows: list[dict[str, str]], dataset_schema: Mapping[str, Any], metric_registry: Mapping[str, Any])`

- source line: 922
- kind: function
- doc: (docstringなし)

#### Function `process_integration(*, input_run: str | Path, output_root: str | Path, config: IntegrationProcessingConfig, config_path: str | Path | None=None, started_at: datetime | None=None)`

- source line: 970
- kind: function
- doc: (docstringなし)

### 61.61 `src/web_intelligence/pipeline/__init__.py`

- Module summary: Pipeline orchestration for the Generic Web Intelligence & Decision Platform.
- Top-level symbol count: 0

### 61.62 `src/web_intelligence/pipeline/cli.py`

- Module summary: (module docstringなし)
- Top-level symbol count: 5

#### Function `_add_common(parser: argparse.ArgumentParser, *, live_fetch: bool, uses_ga4: bool)`

- source line: 18
- kind: function
- doc: (docstringなし)

#### Function `_add_extraction_inputs(parser: argparse.ArgumentParser, *, help_prefix: str)`

- source line: 65
- kind: function
- doc: (docstringなし)

#### Function `configure_pipeline_parser(parser: argparse.ArgumentParser)`

- source line: 82
- kind: function
- doc: (docstringなし)

#### Function `_run(args: argparse.Namespace)`

- source line: 145
- kind: function
- doc: (docstringなし)

#### Function `main(argv: list[str] | None=None)`

- source line: 179
- kind: function
- doc: (docstringなし)

### 61.63 `src/web_intelligence/pipeline/config.py`

- Module summary: (module docstringなし)
- Top-level symbol count: 4

#### Class `PipelineConfig`

- source line: 16
- bases: `(none)`
- doc: (docstringなし)
- methods:
  - `config_paths(self)` — line 27 — (docstringなし)

#### Function `_mapping(value: Any, label: str)`

- source line: 40
- kind: function
- doc: (docstringなし)

#### Function `_path(mapping: dict[str, Any], key: str)`

- source line: 46
- kind: function
- doc: (docstringなし)

#### Function `load_pipeline_config(path: str | Path=DEFAULT_PIPELINE_CONFIG)`

- source line: 53
- kind: function
- doc: (docstringなし)

### 61.64 `src/web_intelligence/pipeline/manifest.py`

- Module summary: (module docstringなし)
- Top-level symbol count: 7

#### Function `utc_now()`

- source line: 18
- kind: function
- doc: (docstringなし)

#### Function `make_pipeline_run_id(profile: str)`

- source line: 22
- kind: function
- doc: (docstringなし)

#### Function `_write_json_atomic(path: Path, payload: dict[str, Any])`

- source line: 27
- kind: function
- doc: (docstringなし)

#### Function `artifact_payload(artifact: ArtifactRef)`

- source line: 37
- kind: function
- doc: (docstringなし)

#### Function `_tool_version()`

- source line: 46
- kind: function
- doc: (docstringなし)

#### Function `_config_payload(config_path: Path, config: PipelineConfig)`

- source line: 53
- kind: function
- doc: (docstringなし)

#### Class `PipelineManifest`

- source line: 71
- bases: `(none)`
- doc: (docstringなし)
- methods:
  - `__init__(self, *, profile: str, stage_plan: tuple[str, ...], run_id: str, run_dir: Path, config_path: Path, config: PipelineConfig, initial_artifacts: dict[str, ArtifactRef])` — line 72 — (docstringなし)
  - `load_for_resume(cls, *, run_dir: Path, expected_profile: str, expected_stage_plan: tuple[str, ...], config_path: Path, config: PipelineConfig)` — line 118 — (docstringなし)
  - `write(self)` — line 173 — (docstringなし)
  - `stage_state(self, stage_id: str)` — line 176 — (docstringなし)
  - `initial_artifact_payloads(self)` — line 182 — (docstringなし)
  - `begin_resume(self, *, supplied_artifacts: dict[str, ArtifactRef])` — line 185 — (docstringなし)
  - `record_attempt(self, record: StageRecord)` — line 206 — (docstringなし)
  - `complete(self, *, final_artifacts: dict[str, ArtifactRef])` — line 215 — (docstringなし)
  - `fail(self, error: str, *, failed_stage: str)` — line 229 — (docstringなし)
  - `sha256(self)` — line 236 — (docstringなし)

### 61.65 `src/web_intelligence/pipeline/models.py`

- Module summary: (module docstringなし)
- Top-level symbol count: 4

#### Class `PipelineError`

- source line: 8
- bases: `RuntimeError`
- doc: Base exception for pipeline orchestration failures.

#### Class `ArtifactRef`

- source line: 13
- bases: `(none)`
- doc: (docstringなし)

#### Class `PipelineOptions`

- source line: 21
- bases: `(none)`
- doc: (docstringなし)

#### Class `StageRecord`

- source line: 36
- bases: `(none)`
- doc: (docstringなし)
- methods:
  - `as_dict(self)` — line 47 — (docstringなし)

### 61.66 `src/web_intelligence/pipeline/orchestrator.py`

- Module summary: (module docstringなし)
- Top-level symbol count: 13

#### Function `_artifact_dict(artifact: ArtifactRef)`

- source line: 46
- kind: function
- doc: (docstringなし)

#### Function `_plan_lines(profile: str)`

- source line: 50
- kind: function
- doc: (docstringなし)

#### Function `render_plan(profile: str)`

- source line: 61
- kind: function
- doc: (docstringなし)

#### Function `_artifact_from_payload(payload: dict[str, str], *, strict: bool)`

- source line: 67
- kind: function
- doc: (docstringなし)

#### Function `_initial_artifacts_new(options: PipelineOptions)`

- source line: 105
- kind: function
- doc: (docstringなし)

#### Function `_initial_artifacts_resume(manifest: PipelineManifest)`

- source line: 145
- kind: function
- doc: (docstringなし)

#### Function `_supplied_stage_artifacts(options: PipelineOptions)`

- source line: 156
- kind: function
- doc: (docstringなし)

#### Function `_execute_stage(stage_id: str, config: PipelineConfig, options: PipelineOptions, artifacts: dict[str, ArtifactRef])`

- source line: 187
- kind: function
- doc: (docstringなし)

#### Function `_direct_inputs(stage_id: str, artifacts: dict[str, ArtifactRef])`

- source line: 233
- kind: function
- doc: (docstringなし)

#### Function `_resume_run_dir(path: Path)`

- source line: 260
- kind: function
- doc: (docstringなし)

#### Function `_reusable_stage_artifact(manifest: PipelineManifest, stage_id: str)`

- source line: 269
- kind: function
- doc: (docstringなし)

#### Function `_record_nonexecuted_stage(*, manifest: PipelineManifest, stage_id: str, status: str, artifact: ArtifactRef, artifacts: dict[str, ArtifactRef])`

- source line: 282
- kind: function
- doc: (docstringなし)

#### Function `run_pipeline(*, config_path: Path, config: PipelineConfig, options: PipelineOptions)`

- source line: 305
- kind: function
- doc: (docstringなし)

### 61.67 `src/web_intelligence/pipeline/profiles/__init__.py`

- Module summary: (module docstringなし)
- Top-level symbol count: 0

### 61.68 `src/web_intelligence/pipeline/profiles/definitions.py`

- Module summary: (module docstringなし)
- Top-level symbol count: 1

#### Class `ProfileDefinition`

- source line: 7
- bases: `(none)`
- doc: (docstringなし)

### 61.69 `src/web_intelligence/pipeline/stages/__init__.py`

- Module summary: Reusable pipeline stage implementations.
- Top-level symbol count: 0

### 61.70 `src/web_intelligence/pipeline/stages/execution.py`

- Module summary: (module docstringなし)
- Top-level symbol count: 6

#### Class `ArtifactLocator`

- source line: 14
- bases: `(none)`
- doc: (docstringなし)
- methods:
  - `snapshot(self)` — line 19 — (docstringなし)
  - `artifact_dir(self, manifest: Path)` — line 24 — (docstringなし)

#### Function `sha256_file(path: Path)`

- source line: 31
- kind: function
- doc: (docstringなし)

#### Function `project_relative(path: Path)`

- source line: 39
- kind: function
- doc: (docstringなし)

#### Function `resolve_console_script(name: str)`

- source line: 50
- kind: function
- doc: (docstringなし)

#### Function `run_stage_command(*, stage_id: str, executable: str, args: Iterable[str], locator: ArtifactLocator)`

- source line: 62
- kind: function
- doc: (docstringなし)

#### Function `existing_artifact(*, stage_id: str, artifact_dir: Path, manifest_name: str)`

- source line: 102
- kind: function
- doc: (docstringなし)

### 61.71 `src/web_intelligence/pipeline/stages/integration.py`

- Module summary: (module docstringなし)
- Top-level symbol count: 3

#### Function `integration_extract(config: PipelineConfig, ga4_processing: ArtifactRef, gsc_processing: ArtifactRef)`

- source line: 26
- kind: function
- doc: (docstringなし)

#### Function `integration_process(config: PipelineConfig, input_artifact: ArtifactRef)`

- source line: 46
- kind: function
- doc: (docstringなし)

#### Function `integration_analyze(config: PipelineConfig, input_artifact: ArtifactRef)`

- source line: 63
- kind: function
- doc: (docstringなし)

### 61.72 `src/web_intelligence/pipeline/stages/source.py`

- Module summary: (module docstringなし)
- Top-level symbol count: 6

#### Function `ga4_extract(config: PipelineConfig, options: PipelineOptions)`

- source line: 38
- kind: function
- doc: (docstringなし)

#### Function `gsc_extract(config: PipelineConfig, options: PipelineOptions)`

- source line: 64
- kind: function
- doc: (docstringなし)

#### Function `ga4_process(config: PipelineConfig, input_artifact: ArtifactRef)`

- source line: 86
- kind: function
- doc: (docstringなし)

#### Function `gsc_process(input_artifact: ArtifactRef)`

- source line: 105
- kind: function
- doc: (docstringなし)

#### Function `ga4_analyze(input_artifact: ArtifactRef)`

- source line: 121
- kind: function
- doc: (docstringなし)

#### Function `gsc_analyze(input_artifact: ArtifactRef)`

- source line: 137
- kind: function
- doc: (docstringなし)

### 61.73 `src/web_intelligence/pipeline/stages/visualization.py`

- Module summary: (module docstringなし)
- Top-level symbol count: 1

#### Function `visualize(config: PipelineConfig, input_artifact: ArtifactRef)`

- source line: 19
- kind: function
- doc: (docstringなし)

### 61.74 `src/web_intelligence/processing/__init__.py`

- Module summary: Source processing layers.
- Top-level symbol count: 0

### 61.75 `src/web_intelligence/processing/ga4/__init__.py`

- Module summary: GA4 processing layer.
- Top-level symbol count: 0

### 61.76 `src/web_intelligence/processing/ga4/cli.py`

- Module summary: (module docstringなし)
- Top-level symbol count: 4

#### Function `_configure_logging(level_name: str)`

- source line: 24
- kind: function
- doc: (docstringなし)

#### Function `_find_latest_run(extraction_root: Path, property_id: str | None)`

- source line: 31
- kind: function
- doc: (docstringなし)

#### Function `build_parser()`

- source line: 38
- kind: function
- doc: (docstringなし)

#### Function `main(argv: list[str] | None=None)`

- source line: 76
- kind: function
- doc: (docstringなし)

### 61.77 `src/web_intelligence/processing/ga4/config.py`

- Module summary: (module docstringなし)
- Top-level symbol count: 16

#### Class `ProcessingConfigError`

- source line: 12
- bases: `Exception`
- doc: Raised when processing configuration is invalid.

#### Class `UrlProcessingConfig`

- source line: 25
- bases: `(none)`
- doc: (docstringなし)

#### Class `EventGroupConfig`

- source line: 31
- bases: `(none)`
- doc: (docstringなし)

#### Class `EventProcessingConfig`

- source line: 37
- bases: `(none)`
- doc: (docstringなし)
- methods:
  - `enabled(self)` — line 42 — (docstringなし)

#### Class `PeriodWindowConfig`

- source line: 47
- bases: `(none)`
- doc: (docstringなし)

#### Class `PeriodProcessingConfig`

- source line: 53
- bases: `(none)`
- doc: (docstringなし)

#### Class `ProcessingConfig`

- source line: 67
- bases: `(none)`
- doc: (docstringなし)

#### Function `default_processing_config()`

- source line: 73
- kind: function
- doc: (docstringなし)

#### Function `_non_empty_string(value: Any, field_name: str)`

- source line: 77
- kind: function
- doc: (docstringなし)

#### Function `_string_list(value: Any, field_name: str)`

- source line: 83
- kind: function
- doc: (docstringなし)

#### Function `_parse_url_config(data: Any)`

- source line: 93
- kind: function
- doc: (docstringなし)

#### Function `_parse_event_config(data: Any)`

- source line: 110
- kind: function
- doc: (docstringなし)

#### Function `_parse_period_config(data: Any)`

- source line: 161
- kind: function
- doc: (docstringなし)

#### Function `load_processing_config(path: str | Path | None=None)`

- source line: 218
- kind: function
- doc: (docstringなし)

#### Function `processing_config_payload(config: ProcessingConfig)`

- source line: 246
- kind: function
- doc: (docstringなし)

#### Function `processing_config_sha256(config: ProcessingConfig)`

- source line: 250
- kind: function
- doc: (docstringなし)

### 61.78 `src/web_intelligence/processing/ga4/events.py`

- Module summary: (module docstringなし)
- Top-level symbol count: 4

#### Class `EventCatalogEntry`

- source line: 13
- bases: `(none)`
- doc: (docstringなし)

#### Function `event_group_map(config: EventProcessingConfig)`

- source line: 20
- kind: function
- doc: (docstringなし)

#### Function `classify_event_name(event_name: str, config: EventProcessingConfig)`

- source line: 28
- kind: function
- doc: (docstringなし)

#### Function `build_event_catalog(csv_paths: Iterable[Path], config: EventProcessingConfig)`

- source line: 37
- kind: function
- doc: (docstringなし)

### 61.79 `src/web_intelligence/processing/ga4/io.py`

- Module summary: (module docstringなし)
- Top-level symbol count: 4

#### Class `ProcessingIOError`

- source line: 11
- bases: `Exception`
- doc: Raised when a processing artifact cannot be written safely.

#### Function `_prepare_output(path: Path)`

- source line: 15
- kind: function
- doc: (docstringなし)

#### Function `write_json_atomic(path: Path, payload: Any)`

- source line: 25
- kind: function
- doc: (docstringなし)

#### Function `write_csv_atomic(path: Path, fieldnames: list[str], rows: Iterable[dict[str, Any]])`

- source line: 35
- kind: function
- doc: (docstringなし)

### 61.80 `src/web_intelligence/processing/ga4/periods.py`

- Module summary: (module docstringなし)
- Top-level symbol count: 7

#### Class `PeriodProcessingError`

- source line: 10
- bases: `Exception`
- doc: Raised when a period comparison dataset cannot be built safely.

#### Class `PeriodCoverage`

- source line: 15
- bases: `(none)`
- doc: (docstringなし)

#### Function `parse_ga4_date(value: str)`

- source line: 25
- kind: function
- doc: (docstringなし)

#### Function `canonicalize_ga4_date(value: str)`

- source line: 34
- kind: function
- doc: (docstringなし)

#### Function `_shift_year(value: date, years: int)`

- source line: 38
- kind: function
- doc: (docstringなし)

#### Function `_coverage_status(expected_days: int, available_days: int)`

- source line: 47
- kind: function
- doc: (docstringなし)

#### Function `build_period_dataset(rows: list[dict[str, str]], config: PeriodProcessingConfig)`

- source line: 55
- kind: function
- doc: (docstringなし)

### 61.81 `src/web_intelligence/processing/ga4/processor.py`

- Module summary: (module docstringなし)
- Top-level symbol count: 14

#### Function `_project_relative_path(path: Path)`

- source line: 38
- kind: function
- doc: (docstringなし)

#### Class `ProcessingError`

- source line: 45
- bases: `Exception`
- doc: Raised when a source extraction run cannot be processed safely.

#### Function `_sha256_file(path: Path)`

- source line: 49
- kind: function
- doc: (docstringなし)

#### Function `_load_source_manifest(input_run: Path)`

- source line: 57
- kind: function
- doc: (docstringなし)

#### Function `_source_csv_paths(input_run: Path)`

- source line: 73
- kind: function
- doc: (docstringなし)

#### Function `_processing_output_directory(output_root: str | Path, property_id: str, processing_run_id: str)`

- source line: 77
- kind: function
- doc: (docstringなし)

#### Function `_source_reports_by_csv(source_manifest: dict[str, Any])`

- source line: 86
- kind: function
- doc: (docstringなし)

#### Function `_source_quality_provenance(source_file: str, reports_by_csv: dict[str, dict[str, Any]])`

- source line: 104
- kind: function
- doc: (docstringなし)

#### Function `_derived_headers(fieldnames: list[str], config: ProcessingConfig)`

- source line: 133
- kind: function
- doc: (docstringなし)

#### Function `_iter_enriched_rows(source_path: Path, url_columns: tuple[str, ...], has_event: bool, config: ProcessingConfig)`

- source line: 167
- kind: function
- doc: (docstringなし)

#### Function `_process_tables(csv_paths: tuple[Path, ...], output_directory: Path, config: ProcessingConfig, reports_by_csv: dict[str, dict[str, Any]])`

- source line: 206
- kind: function
- doc: (docstringなし)

#### Function `_write_event_catalog(csv_paths: tuple[Path, ...], output_directory: Path, config: ProcessingConfig)`

- source line: 257
- kind: function
- doc: (docstringなし)

#### Function `_write_period_outputs(input_run: Path, output_directory: Path, config: ProcessingConfig)`

- source line: 286
- kind: function
- doc: (docstringなし)

#### Function `process_extraction_run(input_run: str | Path, output_root: str | Path, config: ProcessingConfig)`

- source line: 376
- kind: function
- doc: (docstringなし)

### 61.82 `src/web_intelligence/processing/ga4/urls.py`

- Module summary: (module docstringなし)
- Top-level symbol count: 2

#### Function `derive_url_fields(value: str, hierarchy_depth: int=2)`

- source line: 6
- kind: function
- doc: Derive conservative URL/path fields without modifying the raw value.

#### Function `_path_hierarchy(path: str, depth: int)`

- source line: 36
- kind: function
- doc: (docstringなし)

### 61.83 `src/web_intelligence/processing/gsc/__init__.py`

- Module summary: Google Search Console processing layer.
- Top-level symbol count: 0

### 61.84 `src/web_intelligence/processing/gsc/cli.py`

- Module summary: (module docstringなし)
- Top-level symbol count: 2

#### Function `_parser()`

- source line: 17
- kind: function
- doc: (docstringなし)

#### Function `main()`

- source line: 34
- kind: function
- doc: (docstringなし)

### 61.85 `src/web_intelligence/processing/gsc/periods.py`

- Module summary: (module docstringなし)
- Top-level symbol count: 5

#### Class `PeriodWindow`

- source line: 8
- bases: `(none)`
- doc: (docstringなし)
- methods:
  - `contains(self, value: date)` — line 16 — (docstringなし)
  - `to_dict(self)` — line 19 — (docstringなし)

#### Function `_shift_year(value: date, years: int)`

- source line: 30
- kind: function
- doc: (docstringなし)

#### Function `_iter_dates(start: date, end: date)`

- source line: 38
- kind: function
- doc: (docstringなし)

#### Function `_coverage(window_start: date, window_end: date, extraction_start: date, extraction_end: date, observed_dates: frozenset[date] | None)`

- source line: 45
- kind: function
- doc: (docstringなし)

#### Function `build_period_windows(*, extraction_start: date, extraction_end: date, observed_dates: frozenset[date] | None=None, observed_start: date | None=None, observed_end: date | None=None, sizes: tuple[int, ...]=(28, 90))`

- source line: 68
- kind: function
- doc: (docstringなし)

### 61.86 `src/web_intelligence/processing/gsc/processor.py`

- Module summary: (module docstringなし)
- Top-level symbol count: 18

#### Class `GscProcessingError`

- source line: 22
- bases: `RuntimeError`
- doc: (docstringなし)

#### Function `_tool_version()`

- source line: 29
- kind: function
- doc: (docstringなし)

#### Function `_utc_now()`

- source line: 36
- kind: function
- doc: (docstringなし)

#### Function `_run_id(now: datetime)`

- source line: 40
- kind: function
- doc: (docstringなし)

#### Function `_sha256_file(path: Path)`

- source line: 44
- kind: function
- doc: (docstringなし)

#### Function `_read_json(path: Path)`

- source line: 52
- kind: function
- doc: (docstringなし)

#### Function `_write_json_atomic(path: Path, payload: dict[str, Any])`

- source line: 62
- kind: function
- doc: (docstringなし)

#### Function `_manifest_started(path: Path)`

- source line: 72
- kind: function
- doc: (docstringなし)

#### Function `find_latest_extraction(root: Path=Path('output/extraction/gsc'))`

- source line: 81
- kind: function
- doc: (docstringなし)

#### Function `_observed_dates_from_daily_csv(path: Path)`

- source line: 97
- kind: function
- doc: (docstringなし)

#### Function `_observed_dates_by_search_type(input_run: Path, source_manifest: dict[str, Any])`

- source line: 118
- kind: function
- doc: (docstringなし)

#### Function `_coverage_summary(*, extraction_start: date, extraction_end: date, observed_dates: frozenset[date])`

- source line: 148
- kind: function
- doc: (docstringなし)

#### Function `_portable_manifest_path(path: Path)`

- source line: 188
- kind: function
- doc: (docstringなし)

#### Function `_quality_for_report(report: dict[str, Any])`

- source line: 195
- kind: function
- doc: (docstringなし)

#### Function `_period_fieldnames(windows: tuple[PeriodWindow, ...])`

- source line: 213
- kind: function
- doc: (docstringなし)

#### Function `_augment_row(row: dict[str, str], *, windows: tuple[PeriodWindow, ...])`

- source line: 217
- kind: function
- doc: (docstringなし)

#### Function `_process_csv(source: Path, destination: Path, *, windows: tuple[PeriodWindow, ...])`

- source line: 244
- kind: function
- doc: (docstringなし)

#### Function `process_search_console(input_run: Path, *, output_root: Path=Path('output/processing/gsc'))`

- source line: 286
- kind: function
- doc: (docstringなし)

### 61.87 `src/web_intelligence/visualization/__init__.py`

- Module summary: Generic Visualization presentation layer.
- Top-level symbol count: 0

### 61.88 `src/web_intelligence/visualization/adapters/__init__.py`

- Module summary: (module docstringなし)
- Top-level symbol count: 1

#### Function `select_adapter(root: Path)`

- source line: 23
- kind: function
- doc: (docstringなし)

### 61.89 `src/web_intelligence/visualization/adapters/base.py`

- Module summary: (module docstringなし)
- Top-level symbol count: 5

#### Function `read_json(path: Path)`

- source line: 12
- kind: function
- doc: (docstringなし)

#### Function `read_csv(path: Path)`

- source line: 16
- kind: function
- doc: (docstringなし)

#### Function `relative_path_text(path: Path)`

- source line: 21
- kind: function
- doc: (docstringなし)

#### Function `humanize_metric(metric_id: str)`

- source line: 25
- kind: function
- doc: (docstringなし)

#### Class `VisualizationAdapter`

- source line: 40
- bases: `ABC`
- doc: (docstringなし)
- methods:
  - `matches(cls, manifest: dict[str, Any])` — line 46 — (docstringなし)
  - `__init__(self, root: Path)` — line 49 — (docstringなし)
  - `build_contract(self)` — line 57 — (docstringなし)
  - `artifact_identity(self, artifact_type: str, site_key: str)` — line 60 — (docstringなし)
  - `adapter_metadata(self)` — line 72 — (docstringなし)

### 61.90 `src/web_intelligence/visualization/adapters/ga4_analysis.py`

- Module summary: (module docstringなし)
- Top-level symbol count: 1

#### Class `GA4AnalysisAdapter`

- source line: 14
- bases: `VisualizationAdapter`
- doc: (docstringなし)
- methods:
  - `matches(cls, manifest: dict[str, Any])` — line 18 — (docstringなし)
  - `build_contract(self)` — line 23 — (docstringなし)

### 61.91 `src/web_intelligence/visualization/adapters/generic_contract.py`

- Module summary: (module docstringなし)
- Top-level symbol count: 1

#### Class `GenericContractAdapter`

- source line: 16
- bases: `VisualizationAdapter`
- doc: Pass through any already-normalized Generic Visualization Input Contract.
- methods:
  - `can_open(cls, root)` — line 28 — (docstringなし)
  - `matches(cls, manifest: dict[str, Any])` — line 32 — (docstringなし)
  - `__init__(self, root)` — line 35 — (docstringなし)
  - `build_contract(self)` — line 42 — (docstringなし)

### 61.92 `src/web_intelligence/visualization/adapters/gsc_analysis.py`

- Module summary: (module docstringなし)
- Top-level symbol count: 1

#### Class `GSCAnalysisAdapter`

- source line: 14
- bases: `VisualizationAdapter`
- doc: (docstringなし)
- methods:
  - `matches(cls, manifest: dict[str, Any])` — line 18 — (docstringなし)
  - `build_contract(self)` — line 21 — (docstringなし)

### 61.93 `src/web_intelligence/visualization/adapters/integration_analysis.py`

- Module summary: (module docstringなし)
- Top-level symbol count: 1

#### Class `IntegrationAnalysisAdapter`

- source line: 13
- bases: `VisualizationAdapter`
- doc: (docstringなし)
- methods:
  - `matches(cls, manifest: dict[str, Any])` — line 17 — (docstringなし)
  - `build_contract(self)` — line 20 — (docstringなし)

### 61.94 `src/web_intelligence/visualization/cli.py`

- Module summary: (module docstringなし)
- Top-level symbol count: 3

#### Function `_version()`

- source line: 16
- kind: function
- doc: (docstringなし)

#### Function `build_parser()`

- source line: 23
- kind: function
- doc: (docstringなし)

#### Function `main()`

- source line: 43
- kind: function
- doc: (docstringなし)

### 61.95 `src/web_intelligence/visualization/contracts.py`

- Module summary: (module docstringなし)
- Top-level symbol count: 3

#### Function `sha256_file(path: Path)`

- source line: 21
- kind: function
- doc: (docstringなし)

#### Function `canonical_json_bytes(payload: Any)`

- source line: 29
- kind: function
- doc: (docstringなし)

#### Function `validate_contract(payload: dict[str, Any])`

- source line: 36
- kind: function
- doc: (docstringなし)

### 61.96 `src/web_intelligence/visualization/presentation.py`

- Module summary: (module docstringなし)
- Top-level symbol count: 2

#### Function `metric_presentation(metric: dict[str, Any])`

- source line: 116
- kind: function
- doc: (docstringなし)

#### Function `decorate_contract(contract: dict[str, Any])`

- source line: 146
- kind: function
- doc: (docstringなし)

### 61.97 `src/web_intelligence/visualization/processor.py`

- Module summary: (module docstringなし)
- Top-level symbol count: 11

#### Class `VisualizationError`

- source line: 23
- bases: `RuntimeError`
- doc: (docstringなし)

#### Function `_project_relative(path: Path)`

- source line: 27
- kind: function
- doc: (docstringなし)

#### Function `_safe_site(value: str)`

- source line: 37
- kind: function
- doc: (docstringなし)

#### Function `_run_id(now: datetime | None=None)`

- source line: 42
- kind: function
- doc: (docstringなし)

#### Function `_write_json(path: Path, payload: Any)`

- source line: 47
- kind: function
- doc: (docstringなし)

#### Function `_write_react_data(path: Path, contract: dict[str, Any], spec: dict[str, Any], delivery: dict[str, Any])`

- source line: 51
- kind: function
- doc: (docstringなし)

#### Function `_write_input_artifact_archive(input_run: Path, target: Path)`

- source line: 68
- kind: function
- doc: Write a deterministic ZIP containing the exact input Artifact directory.

#### Function `_copy_react_dist(react_dist: Path, target: Path)`

- source line: 100
- kind: function
- doc: (docstringなし)

#### Function `_write_main_dashboard_from_react(react_index: Path, target: Path)`

- source line: 106
- kind: function
- doc: (docstringなし)

#### Function `validate_input(input_run: Path, spec_path: Path, react_dist: Path | None=None)`

- source line: 120
- kind: function
- doc: (docstringなし)

#### Function `create_visualization(*, input_run: Path, spec_path: Path, output_root: Path, react_dist: Path, tool_version: str)`

- source line: 132
- kind: function
- doc: (docstringなし)

### 61.98 `src/web_intelligence/visualization/render_static.py`

- Module summary: (module docstringなし)
- Top-level symbol count: 15

#### Function `_num(value: Any)`

- source line: 10
- kind: function
- doc: (docstringなし)

#### Function `_fmt(value: Any, unit: str | None=None)`

- source line: 20
- kind: function
- doc: (docstringなし)

#### Function `_metric_map(contract: dict[str, Any])`

- source line: 37
- kind: function
- doc: (docstringなし)

#### Function `_metric_label(metric: dict[str, Any] | None, metric_id: str)`

- source line: 41
- kind: function
- doc: (docstringなし)

#### Function `_metric_description(metric: dict[str, Any] | None, metric_id: str)`

- source line: 46
- kind: function
- doc: (docstringなし)

#### Function `_unit_label(metric: dict[str, Any] | None)`

- source line: 55
- kind: function
- doc: (docstringなし)

#### Function `_format_unit(metric: dict[str, Any] | None)`

- source line: 60
- kind: function
- doc: (docstringなし)

#### Function `_status_label(value: Any)`

- source line: 65
- kind: function
- doc: (docstringなし)

#### Function `_baseline_label(value: Any)`

- source line: 83
- kind: function
- doc: (docstringなし)

#### Function `_preferred_kpis(contract: dict[str, Any])`

- source line: 94
- kind: function
- doc: (docstringなし)

#### Function `_date_label(raw: str)`

- source line: 131
- kind: function
- doc: (docstringなし)

#### Function `_month_label(raw: str)`

- source line: 139
- kind: function
- doc: (docstringなし)

#### Function `_monthly_tick_indices(points: list[tuple[str, float]])`

- source line: 147
- kind: function
- doc: (docstringなし)

#### Function `_trend_svg(rows: list[dict[str, Any]], metric_id: str, metric: dict[str, Any] | None)`

- source line: 160
- kind: function
- doc: (docstringなし)

#### Function `render_static_html(contract: dict[str, Any], spec: dict[str, Any])`

- source line: 235
- kind: function
- doc: (docstringなし)

### 61.99 `src/web_intelligence/visualization/spec.py`

- Module summary: (module docstringなし)
- Top-level symbol count: 3

#### Class `VisualizationSpecError`

- source line: 11
- bases: `ValueError`
- doc: (docstringなし)

#### Function `_project_relative(path: Path)`

- source line: 15
- kind: function
- doc: (docstringなし)

#### Function `load_visualization_spec(path: Path)`

- source line: 25
- kind: function
- doc: (docstringなし)


## 62. Config File Inventory

### 62.1 `config/system/integration/analysis/generic_page_analysis.yaml`

- bytes: 762
- key paths (structure only; values are not copied here):
  - `schema_version`
  - `spec_id`
  - `spec_version`
  - `metrics`
  - `coverage`
  - `coverage.minimum_ratio`
  - `coverage.require_provable_contiguous_coverage`
  - `quality`
  - `quality.accepted_input_statuses`
  - `operations`
  - `operations.summary`
  - `operations.period_total`
  - `operations.safe_average`
  - `operations.safe_average.enabled`
  - `operations.safe_average.basis`
  - `operations.comparisons`
  - `operations.comparisons.enabled`
  - `operations.comparisons.baselines`
  - `operations.trend`
  - `operations.trend.enabled`
  - `operations.trend.grain`
  - `operations.ranking`
  - `operations.ranking.enabled`
  - `operations.ranking.group_by`
  - `operations.ranking.window`
  - `operations.ranking.limit`
  - `operations.contribution`
  - `operations.contribution.enabled`
  - `operations.contribution.group_by`
  - `operations.contribution.window`
  - `operations.contribution.limit`
  - `operations.contribution.include_other`

### 62.2 `config/system/integration/processing/cross_source_page_daily.yaml`

- bytes: 3072
- key paths (structure only; values are not copied here):
  - `schema_version`
  - `config_id`
  - `config_version`
  - `use_case`
  - `use_case.id`
  - `use_case.version`
  - `input`
  - `input.dataset`
  - `input.date_column`
  - `period`
  - `period.mode`
  - `sources`
  - `sources.required`
  - `sources.optional`
  - `grain`
  - `dimensions`
  - `metrics`
  - `metrics.source_column`
  - `metrics.display_name`
  - `metrics.source`
  - `metrics.data_type`
  - `metrics.source_semantic_type`
  - `metrics.semantic_type`
  - `metrics.aggregation_rule`
  - `metrics.context_grain`
  - `metrics.unit`
  - `metrics.availability`
  - `metrics.attribution_scope`
  - `metrics.source_column`
  - `metrics.display_name`
  - `metrics.source`
  - `metrics.data_type`
  - `metrics.source_semantic_type`
  - `metrics.semantic_type`
  - `metrics.aggregation_rule`
  - `metrics.context_grain`
  - `metrics.unit`
  - `metrics.availability`
  - `metrics.attribution_scope`
  - `metrics.source_column`
  - `metrics.display_name`
  - `metrics.source`
  - `metrics.data_type`
  - `metrics.source_semantic_type`
  - `metrics.semantic_type`
  - `metrics.aggregation_rule`
  - `metrics.context_grain`
  - `metrics.unit`
  - `metrics.availability`
  - `metrics.attribution_scope`
  - `metrics.source_column`
  - `metrics.display_name`
  - `metrics.source`
  - `metrics.data_type`
  - `metrics.source_semantic_type`
  - `metrics.semantic_type`
  - `metrics.aggregation_rule`
  - `metrics.unit`
  - `metrics.availability`
  - `metrics.attribution_scope`
  - `metrics.source_column`
  - `metrics.display_name`
  - `metrics.source`
  - `metrics.data_type`
  - `metrics.source_semantic_type`
  - `metrics.semantic_type`
  - `metrics.aggregation_rule`
  - `metrics.unit`
  - `metrics.availability`
  - `metrics.attribution_scope`
  - `metrics.source_column`
  - `metrics.display_name`
  - `metrics.source`
  - `metrics.data_type`
  - `metrics.source_semantic_type`
  - `metrics.semantic_type`
  - `metrics.aggregation_rule`
  - `metrics.numerator`
  - `metrics.denominator`
  - `metrics.unit`
  - `metrics.availability`
  - `metrics.attribution_scope`
  - `metrics.source_column`
  - `metrics.display_name`
  - `metrics.source`
  - `metrics.data_type`
  - `metrics.source_semantic_type`
  - `metrics.semantic_type`
  - `metrics.aggregation_rule`
  - `metrics.weight_metric`
  - `metrics.unit`
  - `metrics.availability`
  - `metrics.attribution_scope`
  - `filters`
  - `coverage`
  - `coverage.minimum_ratio`
  - `coverage.missing_policy`
  - `comparison`
  - `comparison.current_days`
  - `comparison.previous`
  - `comparison.yoy`

### 62.3 `config/system/integration/processing/cross_source_page_daily_all_available.yaml`

- bytes: 3084
- key paths (structure only; values are not copied here):
  - `schema_version`
  - `config_id`
  - `config_version`
  - `use_case`
  - `use_case.id`
  - `use_case.version`
  - `input`
  - `input.dataset`
  - `input.date_column`
  - `period`
  - `period.mode`
  - `sources`
  - `sources.required`
  - `sources.optional`
  - `grain`
  - `dimensions`
  - `metrics`
  - `metrics.source_column`
  - `metrics.display_name`
  - `metrics.source`
  - `metrics.data_type`
  - `metrics.source_semantic_type`
  - `metrics.semantic_type`
  - `metrics.aggregation_rule`
  - `metrics.context_grain`
  - `metrics.unit`
  - `metrics.availability`
  - `metrics.attribution_scope`
  - `metrics.source_column`
  - `metrics.display_name`
  - `metrics.source`
  - `metrics.data_type`
  - `metrics.source_semantic_type`
  - `metrics.semantic_type`
  - `metrics.aggregation_rule`
  - `metrics.context_grain`
  - `metrics.unit`
  - `metrics.availability`
  - `metrics.attribution_scope`
  - `metrics.source_column`
  - `metrics.display_name`
  - `metrics.source`
  - `metrics.data_type`
  - `metrics.source_semantic_type`
  - `metrics.semantic_type`
  - `metrics.aggregation_rule`
  - `metrics.context_grain`
  - `metrics.unit`
  - `metrics.availability`
  - `metrics.attribution_scope`
  - `metrics.source_column`
  - `metrics.display_name`
  - `metrics.source`
  - `metrics.data_type`
  - `metrics.source_semantic_type`
  - `metrics.semantic_type`
  - `metrics.aggregation_rule`
  - `metrics.unit`
  - `metrics.availability`
  - `metrics.attribution_scope`
  - `metrics.source_column`
  - `metrics.display_name`
  - `metrics.source`
  - `metrics.data_type`
  - `metrics.source_semantic_type`
  - `metrics.semantic_type`
  - `metrics.aggregation_rule`
  - `metrics.unit`
  - `metrics.availability`
  - `metrics.attribution_scope`
  - `metrics.source_column`
  - `metrics.display_name`
  - `metrics.source`
  - `metrics.data_type`
  - `metrics.source_semantic_type`
  - `metrics.semantic_type`
  - `metrics.aggregation_rule`
  - `metrics.numerator`
  - `metrics.denominator`
  - `metrics.unit`
  - `metrics.availability`
  - `metrics.attribution_scope`
  - `metrics.source_column`
  - `metrics.display_name`
  - `metrics.source`
  - `metrics.data_type`
  - `metrics.source_semantic_type`
  - `metrics.semantic_type`
  - `metrics.aggregation_rule`
  - `metrics.weight_metric`
  - `metrics.unit`
  - `metrics.availability`
  - `metrics.attribution_scope`
  - `filters`
  - `coverage`
  - `coverage.minimum_ratio`
  - `coverage.missing_policy`
  - `comparison`
  - `comparison.current_days`
  - `comparison.previous`
  - `comparison.yoy`

### 62.4 `config/system/integration/processing/ga4_page_daily.yaml`

- bytes: 1358
- key paths (structure only; values are not copied here):
  - `schema_version`
  - `config_id`
  - `config_version`
  - `use_case`
  - `use_case.id`
  - `use_case.version`
  - `input`
  - `input.dataset`
  - `input.date_column`
  - `period`
  - `period.mode`
  - `sources`
  - `sources.required`
  - `sources.optional`
  - `grain`
  - `dimensions`
  - `metrics`
  - `metrics.source_column`
  - `metrics.display_name`
  - `metrics.source`
  - `metrics.data_type`
  - `metrics.source_semantic_type`
  - `metrics.semantic_type`
  - `metrics.aggregation_rule`
  - `metrics.unit`
  - `metrics.availability`
  - `metrics.attribution_scope`
  - `metrics.source_column`
  - `metrics.display_name`
  - `metrics.source`
  - `metrics.data_type`
  - `metrics.source_semantic_type`
  - `metrics.semantic_type`
  - `metrics.aggregation_rule`
  - `metrics.unit`
  - `metrics.availability`
  - `metrics.attribution_scope`
  - `metrics.source_column`
  - `metrics.display_name`
  - `metrics.source`
  - `metrics.data_type`
  - `metrics.source_semantic_type`
  - `metrics.semantic_type`
  - `metrics.aggregation_rule`
  - `metrics.unit`
  - `metrics.availability`
  - `metrics.attribution_scope`
  - `filters`
  - `coverage`
  - `coverage.minimum_ratio`
  - `coverage.missing_policy`
  - `comparison`
  - `comparison.current_days`
  - `comparison.previous`
  - `comparison.yoy`

### 62.5 `config/system/integration/processing/gsc_page_daily.yaml`

- bytes: 1744
- key paths (structure only; values are not copied here):
  - `schema_version`
  - `config_id`
  - `config_version`
  - `use_case`
  - `use_case.id`
  - `use_case.version`
  - `input`
  - `input.dataset`
  - `input.date_column`
  - `period`
  - `period.mode`
  - `sources`
  - `sources.required`
  - `sources.optional`
  - `grain`
  - `dimensions`
  - `metrics`
  - `metrics.source_column`
  - `metrics.display_name`
  - `metrics.source`
  - `metrics.data_type`
  - `metrics.source_semantic_type`
  - `metrics.semantic_type`
  - `metrics.aggregation_rule`
  - `metrics.unit`
  - `metrics.availability`
  - `metrics.attribution_scope`
  - `metrics.source_column`
  - `metrics.display_name`
  - `metrics.source`
  - `metrics.data_type`
  - `metrics.source_semantic_type`
  - `metrics.semantic_type`
  - `metrics.aggregation_rule`
  - `metrics.unit`
  - `metrics.availability`
  - `metrics.attribution_scope`
  - `metrics.source_column`
  - `metrics.display_name`
  - `metrics.source`
  - `metrics.data_type`
  - `metrics.source_semantic_type`
  - `metrics.semantic_type`
  - `metrics.aggregation_rule`
  - `metrics.numerator`
  - `metrics.denominator`
  - `metrics.unit`
  - `metrics.availability`
  - `metrics.attribution_scope`
  - `metrics.source_column`
  - `metrics.display_name`
  - `metrics.source`
  - `metrics.data_type`
  - `metrics.source_semantic_type`
  - `metrics.semantic_type`
  - `metrics.aggregation_rule`
  - `metrics.weight_metric`
  - `metrics.unit`
  - `metrics.availability`
  - `metrics.attribution_scope`
  - `filters`
  - `coverage`
  - `coverage.minimum_ratio`
  - `coverage.missing_policy`
  - `comparison`
  - `comparison.current_days`
  - `comparison.previous`
  - `comparison.yoy`

### 62.6 `config/system/integration/processing/gsc_query_page_daily.yaml`

- bytes: 2964
- key paths (structure only; values are not copied here):
  - `schema_version`
  - `config_id`
  - `config_version`
  - `use_case`
  - `use_case.id`
  - `use_case.version`
  - `input`
  - `input.dataset`
  - `input.date_column`
  - `period`
  - `period.mode`
  - `sources`
  - `sources.required`
  - `sources.optional`
  - `grain`
  - `dimensions`
  - `metrics`
  - `metrics.source_column`
  - `metrics.display_name`
  - `metrics.source`
  - `metrics.data_type`
  - `metrics.source_semantic_type`
  - `metrics.semantic_type`
  - `metrics.aggregation_rule`
  - `metrics.unit`
  - `metrics.availability`
  - `metrics.attribution_scope`
  - `metrics.source_column`
  - `metrics.display_name`
  - `metrics.source`
  - `metrics.data_type`
  - `metrics.source_semantic_type`
  - `metrics.semantic_type`
  - `metrics.aggregation_rule`
  - `metrics.unit`
  - `metrics.availability`
  - `metrics.attribution_scope`
  - `metrics.source_column`
  - `metrics.display_name`
  - `metrics.source`
  - `metrics.data_type`
  - `metrics.source_semantic_type`
  - `metrics.semantic_type`
  - `metrics.aggregation_rule`
  - `metrics.unit`
  - `metrics.availability`
  - `metrics.attribution_scope`
  - `metrics.source_column`
  - `metrics.display_name`
  - `metrics.source`
  - `metrics.data_type`
  - `metrics.source_semantic_type`
  - `metrics.semantic_type`
  - `metrics.aggregation_rule`
  - `metrics.unit`
  - `metrics.availability`
  - `metrics.attribution_scope`
  - `metrics.source_column`
  - `metrics.display_name`
  - `metrics.source`
  - `metrics.data_type`
  - `metrics.source_semantic_type`
  - `metrics.semantic_type`
  - `metrics.aggregation_rule`
  - `metrics.unit`
  - `metrics.availability`
  - `metrics.attribution_scope`
  - `metrics.source_column`
  - `metrics.display_name`
  - `metrics.source`
  - `metrics.data_type`
  - `metrics.source_semantic_type`
  - `metrics.semantic_type`
  - `metrics.aggregation_rule`
  - `metrics.unit`
  - `metrics.availability`
  - `metrics.attribution_scope`
  - `metrics.source_column`
  - `metrics.display_name`
  - `metrics.source`
  - `metrics.data_type`
  - `metrics.source_semantic_type`
  - `metrics.semantic_type`
  - `metrics.aggregation_rule`
  - `metrics.unit`
  - `metrics.availability`
  - `metrics.attribution_scope`
  - `filters`
  - `coverage`
  - `coverage.minimum_ratio`
  - `coverage.missing_policy`
  - `comparison`
  - `comparison.current_days`
  - `comparison.previous`
  - `comparison.yoy`

### 62.7 `config/system/visualization/default_dashboard.yaml`

- bytes: 878
- key paths (structure only; values are not copied here):
  - `schema_version`
  - `spec_id`
  - `spec_version`
  - `title`
  - `locale`
  - `sections`
  - `sections.kind`
  - `sections.kind`
  - `sections.kind`
  - `sections.kind`
  - `sections.kind`
  - `static_html`
  - `static_html.canonical`
  - `static_html.retained_for_audit`
  - `react`
  - `react.enabled`
  - `react.primary_entrypoint`
  - `react.quality_coverage_lineage_always_visible`
  - `react.surface_hover_feedback`
  - `react.interactions`
  - `react.interactions.metric_selector`
  - `react.interactions.comparison_selector`
  - `react.interactions.ranking_top_n`
  - `react.interactions.dimension_filter`
  - `react.interactions.quality_lineage_detail`
  - `react.interactions.trend_crosshair`
  - `react.interactions.trend_drag_zoom`
  - `react.interactions.trend_reset_zoom`
  - `react.interactions.trend_dynamic_axes`
  - `react.interactions.trend_x_axis_month_week_day`
  - `react.interactions.trend_diagonal_date_labels`

### 62.8 `config/system/visualization/generic_input_contract.example.json`

- bytes: 1512
- key paths (structure only; values are not copied here):

### 62.9 `config/user/integration/integration_extraction.example.yaml`

- bytes: 236
- key paths (structure only; values are not copied here):
  - `schema_version`
  - `config_id`
  - `config_version`
  - `site_mapping`
  - `site_mapping.site_key`
  - `site_mapping.ga4_property_id`
  - `site_mapping.gsc_site_key`
  - `site_mapping.gsc_site_url`

### 62.10 `config/user/integration/integration_extraction.yaml`

- bytes: 227
- key paths (structure only; values are not copied here):
  - `schema_version`
  - `config_id`
  - `config_version`
  - `site_mapping`
  - `site_mapping.site_key`
  - `site_mapping.ga4_property_id`
  - `site_mapping.gsc_site_key`
  - `site_mapping.gsc_site_url`

### 62.11 `config/user/pipeline/default.yaml`

- bytes: 608
- key paths (structure only; values are not copied here):
  - `schema_version`
  - `defaults`
  - `defaults.ga4_profile`
  - `paths`
  - `paths.ga4_profiles`
  - `paths.ga4_reports`
  - `paths.ga4_processing`
  - `paths.gsc`
  - `paths.integration_extraction`
  - `paths.integration_processing`
  - `paths.integration_analysis`
  - `paths.visualization`

### 62.12 `config/user/processing/ga4/processing.example.yaml`

- bytes: 675
- key paths (structure only; values are not copied here):
  - `version`
  - `url`
  - `url.columns`
  - `url.hierarchy_depth`
  - `event_groups`
  - `event_groups.unmatched_label`
  - `event_groups.groups`
  - `periods`
  - `periods.source_report`
  - `periods.date_column`
  - `periods.include_previous`
  - `periods.include_yoy`
  - `periods.windows`
  - `periods.windows.days`
  - `periods.windows.days`

### 62.13 `config/user/processing/ga4/processing.yaml`

- bytes: 675
- key paths (structure only; values are not copied here):
  - `version`
  - `url`
  - `url.columns`
  - `url.hierarchy_depth`
  - `event_groups`
  - `event_groups.unmatched_label`
  - `event_groups.groups`
  - `periods`
  - `periods.source_report`
  - `periods.date_column`
  - `periods.include_previous`
  - `periods.include_yoy`
  - `periods.windows`
  - `periods.windows.days`
  - `periods.windows.days`

### 62.14 `config/user/sources/ga4/profiles.example.yaml`

- bytes: 183
- key paths (structure only; values are not copied here):
  - `version`
  - `profiles`
  - `profiles.default`
  - `profiles.default.auth`
  - `profiles.default.auth.type`
  - `profiles.default.auth.client_secrets_file`
  - `profiles.default.auth.token_file`

### 62.15 `config/user/sources/ga4/profiles.yaml`

- bytes: 183
- key paths (structure only; values are not copied here):
  - `version`
  - `profiles`
  - `profiles.default`
  - `profiles.default.auth`
  - `profiles.default.auth.type`
  - `profiles.default.auth.client_secrets_file`
  - `profiles.default.auth.token_file`

### 62.16 `config/user/sources/ga4/reports.example.yaml`

- bytes: 564
- key paths (structure only; values are not copied here):
  - `version`
  - `property_id`
  - `preset`
  - `reports`
  - `reports.dimensions`
  - `reports.metrics`
  - `reports.dimension_filter`
  - `reports.dimension_filter.field`
  - `reports.dimension_filter.match_type`
  - `reports.dimension_filter.value`
  - `reports.dimension_filter.case_sensitive`
  - `reports.order_bys`
  - `reports.order_bys.type`
  - `reports.order_bys.descending`

### 62.17 `config/user/sources/ga4/reports.yaml`

- bytes: 690
- key paths (structure only; values are not copied here):
  - `version`
  - `property_id`
  - `preset`
  - `reports`
  - `reports.dimensions`
  - `reports.metrics`
  - `reports.dimensions`
  - `reports.metrics`
  - `reports.dimensions`
  - `reports.metrics`
  - `reports.dimensions`
  - `reports.metrics`

### 62.18 `config/user/sources/gsc/gsc.example.yaml`

- bytes: 498
- key paths (structure only; values are not copied here):
  - `version`
  - `site_url`
  - `dates`
  - `dates.start`
  - `dates.end`
  - `preset`
  - `search_types`
  - `api`
  - `api.data_state`
  - `api.aggregation_type`
  - `api.row_limit`
  - `api.max_retries`
  - `output`
  - `output.root`
  - `oauth`
  - `oauth.client_secrets_file`
  - `oauth.token_file`

### 62.19 `config/user/sources/gsc/gsc.yaml`

- bytes: 498
- key paths (structure only; values are not copied here):
  - `version`
  - `site_url`
  - `dates`
  - `dates.start`
  - `dates.end`
  - `preset`
  - `search_types`
  - `api`
  - `api.data_state`
  - `api.aggregation_type`
  - `api.row_limit`
  - `api.max_retries`
  - `output`
  - `output.root`
  - `oauth`
  - `oauth.client_secrets_file`
  - `oauth.token_file`


## 63. Atomic / Platform CLI Help Snapshot

### `ga4-export --help`

```text
usage: web_intelligence [-h] [--profiles-config PROFILES_CONFIG]
                        [--reports-config REPORTS_CONFIG] [--profile PROFILE]
                        [--report REPORT_NAMES] [--start START] [--end END]
                        [--page-size PAGE_SIZE] [--output-root OUTPUT_ROOT]
                        [--log-level {DEBUG,INFO,WARNING,ERROR}]

Generic Google Analytics 4 Data Extraction Tool

options:
  -h, --help            show this help message and exit
  --profiles-config PROFILES_CONFIG
  --reports-config REPORTS_CONFIG
  --profile PROFILE
  --report REPORT_NAMES
                        Extract only this report. Repeat for multiple reports.
                        If omitted, extract the entire configured preset.
  --start START
  --end END
  --page-size PAGE_SIZE
  --output-root OUTPUT_ROOT
  --log-level {DEBUG,INFO,WARNING,ERROR}
```

### `ga4-process --help`

```text
usage: ga4-process [-h] (--input-run INPUT_RUN | --latest)
                   [--property PROPERTY] [--extraction-root EXTRACTION_ROOT]
                   [--config CONFIG] [--output-root OUTPUT_ROOT]
                   [--log-level LOG_LEVEL]

Process GA4 extraction artifacts without modifying RAW CSV files.

options:
  -h, --help            show this help message and exit
  --input-run INPUT_RUN
                        Path to an extraction run directory.
  --latest              Process the latest run under --extraction-root.
  --property PROPERTY   Optional property ID used to scope --latest.
  --extraction-root EXTRACTION_ROOT
                        Extraction root used with --latest (default:
                        output/extraction/ga4).
  --config CONFIG       Processing YAML. If omitted,
                        config/user/processing/ga4/processing.yaml is used
                        when it exists; otherwise safe defaults are used.
  --output-root OUTPUT_ROOT
                        Processing output root (default:
                        output/processing/ga4).
  --log-level LOG_LEVEL
```

### `ga4-analyze --help`

```text
usage: ga4-analyze [-h] (--latest | --input-run INPUT_RUN)
                   [--property PROPERTY] [--processing-root PROCESSING_ROOT]
                   [--output-root OUTPUT_ROOT] [--log-level LOG_LEVEL]

Analyze a GA4 Processing run with safe metric semantics.

options:
  -h, --help            show this help message and exit
  --latest              Analyze the latest Processing run.
  --input-run INPUT_RUN
                        Path to a specific ga4_process_* run directory.
  --property PROPERTY   Optional GA4 property filter used with --latest.
  --processing-root PROCESSING_ROOT
                        Processing root used by --latest.
  --output-root OUTPUT_ROOT
                        Analysis output root.
  --log-level LOG_LEVEL
```

### `gsc-export --help`

```text
usage: gsc-export [-h] [--config CONFIG] [--site SITE] [--start START]
                  [--end END] [--preset {analysis_full,hourly_recent}]
                  [--search-types SEARCH_TYPES]
                  [--data-state {final,all,hourly_all}]
                  [--aggregation-type {auto,byPage,byProperty,byNewsShowcasePanel}]
                  [--row-limit ROW_LIMIT] [--max-retries MAX_RETRIES]
                  [--output-root OUTPUT_ROOT]
                  [--client-secrets-file CLIENT_SECRETS_FILE]
                  [--token-file TOKEN_FILE] [--list-sites] [--dry-plan]
                  [--verbose]

Extract Google Search Console Search Analytics data.

options:
  -h, --help            show this help message and exit
  --config CONFIG
  --site SITE
  --start START
  --end END
  --preset {analysis_full,hourly_recent}
  --search-types SEARCH_TYPES
                        Comma-separated search types, or 'all'.
  --data-state {final,all,hourly_all}
  --aggregation-type {auto,byPage,byProperty,byNewsShowcasePanel}
  --row-limit ROW_LIMIT
  --max-retries MAX_RETRIES
  --output-root OUTPUT_ROOT
  --client-secrets-file CLIENT_SECRETS_FILE
  --token-file TOKEN_FILE
  --list-sites          List accessible Search Console properties and exit.
  --dry-plan            Show the resolved report plan without extracting data.
  --verbose
```

### `gsc-process --help`

```text
usage: gsc-process [-h] [--input-run INPUT_RUN | --latest]
                   [--output-root OUTPUT_ROOT] [--verbose]

Process a GSC extraction run.

options:
  -h, --help            show this help message and exit
  --input-run INPUT_RUN
  --latest
  --output-root OUTPUT_ROOT
  --verbose
```

### `gsc-analyze --help`

```text
usage: gsc-analyze [-h] [--input-run INPUT_RUN | --latest]
                   [--output-root OUTPUT_ROOT] [--verbose]

Analyze a processed GSC run.

options:
  -h, --help            show this help message and exit
  --input-run INPUT_RUN
  --latest
  --output-root OUTPUT_ROOT
  --verbose
```

### `integration-extract --help`

```text
usage: integration-extract [-h] (--ga4-input-run GA4_INPUT_RUN | --ga4-latest)
                           (--gsc-input-run GSC_INPUT_RUN | --gsc-latest)
                           [--ga4-processing-root GA4_PROCESSING_ROOT]
                           [--gsc-processing-root GSC_PROCESSING_ROOT]
                           --config CONFIG [--output-root OUTPUT_ROOT]
                           [--validate-only] [--log-level LOG_LEVEL]

Losslessly integrate GA4/GSC Processing artifacts while preserving coverage,
lineage, metric semantics, and attribution boundaries.

options:
  -h, --help            show this help message and exit
  --ga4-input-run GA4_INPUT_RUN
  --ga4-latest
  --gsc-input-run GSC_INPUT_RUN
  --gsc-latest
  --ga4-processing-root GA4_PROCESSING_ROOT
  --gsc-processing-root GSC_PROCESSING_ROOT
  --config CONFIG       Integration Extraction site-mapping config.
  --output-root OUTPUT_ROOT
  --validate-only       Validate inputs/contracts without writing an
                        Integration artifact.
  --log-level LOG_LEVEL
```

### `integration-process --help`

```text
usage: integration-process [-h] (--input-run INPUT_RUN | --latest)
                           [--integration-extraction-root INTEGRATION_EXTRACTION_ROOT]
                           [--site-key SITE_KEY] --config CONFIG
                           [--output-root OUTPUT_ROOT] [--validate-only]
                           [--log-level LOG_LEVEL]

Assemble analysis-ready datasets from Integration Extraction artifacts using
explicit period, source, grain, metric, filter, and coverage config.

options:
  -h, --help            show this help message and exit
  --input-run INPUT_RUN
  --latest
  --integration-extraction-root INTEGRATION_EXTRACTION_ROOT
  --site-key SITE_KEY   Optional site_key scope for --latest discovery.
  --config CONFIG
  --output-root OUTPUT_ROOT
  --validate-only       Validate config/input contracts without writing an
                        artifact.
  --log-level LOG_LEVEL
```

### `integration-analyze --help`

```text
usage: integration-analyze [-h] (--input-run INPUT_RUN | --latest)
                           [--integration-processing-root INTEGRATION_PROCESSING_ROOT]
                           [--site-key SITE_KEY] --spec SPEC
                           [--output-root OUTPUT_ROOT] [--validate-only]
                           [--log-level LOG_LEVEL]

Analyze Integration Processing artifacts generically from
dataset/metric/period/coverage contracts and an Analysis Spec.

options:
  -h, --help            show this help message and exit
  --input-run INPUT_RUN
  --latest
  --integration-processing-root INTEGRATION_PROCESSING_ROOT
  --site-key SITE_KEY   Optional site_key scope for --latest discovery.
  --spec SPEC
  --output-root OUTPUT_ROOT
  --validate-only       Validate Processing/Analysis contracts without writing
                        an artifact.
  --log-level LOG_LEVEL
```

### `visualize --help`

```text
usage: visualize [-h] --input-run INPUT_RUN [--spec SPEC]
                 [--output-root OUTPUT_ROOT] [--react-dist REACT_DIST]
                 [--validate-only]

Render a generic Result Artifact or Generic Visualization Input Contract as
Visualization output.

options:
  -h, --help            show this help message and exit
  --input-run INPUT_RUN
  --spec SPEC
  --output-root OUTPUT_ROOT
  --react-dist REACT_DIST
  --validate-only
```

### `gwidp --help`

```text
usage: gwidp [-h] {pipeline} ...

Generic Web Intelligence & Decision Platform.

positional arguments:
  {pipeline}
    pipeline  Run an end-to-end or partial platform pipeline.

options:
  -h, --help  show this help message and exit
```


## 64. Test Contract Inventory

### 64.1 `tests/contract/common/test_portable_paths.py`

- test count: 1
  - line 4: `test_active_source_has_no_hardcoded_local_machine_paths`

### 64.2 `tests/contract/common/test_project_structure.py`

- test count: 4
  - line 12: `test_new_namespace_and_source_names_are_canonical`
  - line 32: `test_config_is_split_between_user_and_system`
  - line 46: `test_examples_root_is_removed`
  - line 50: `test_active_python_does_not_reference_legacy_namespace`

### 64.3 `tests/contract/ga4/test_portable_lineage_paths.py`

- test count: 2
  - line 7: `test_ga4_lineage_directories_are_project_relative`
  - line 33: `test_ga4_lineage_path_helper_uses_relative_path`

### 64.4 `tests/contract/gsc/test_portable_lineage.py`

- test count: 1
  - line 8: `test_source_manifest_path_is_relative`

### 64.5 `tests/contract/pipeline/test_pipeline_structure.py`

- test count: 2
  - line 4: `test_pipeline_files_are_grouped_by_responsibility`
  - line 33: `test_pipeline_config_stays_grouped_under_user_pipeline`

### 64.6 `tests/unit/analysis/ga4/test_analysis_quality.py`

- test count: 2
  - line 7: `test_quality_propagates_degraded_source`
  - line 25: `test_missing_dataset_is_unavailable`

### 64.7 `tests/unit/analysis/ga4/test_analyzer.py`

- test count: 1
  - line 22: `test_analysis_run_is_offline_lineaged_and_metric_safe`

### 64.8 `tests/unit/analysis/ga4/test_cli.py`

- test count: 1
  - line 16: `test_latest_processing_supports_flat_and_legacy_layouts`

### 64.9 `tests/unit/analysis/ga4/test_continuity.py`

- test count: 1
  - line 13: `test_measurement_continuity_marks_late_first_key_event`

### 64.10 `tests/unit/analysis/ga4/test_dimension_periods.py`

- test count: 1
  - line 17: `test_dimension_period_comparison_and_percentage_points`

### 64.11 `tests/unit/analysis/ga4/test_metrics.py`

- test count: 5
  - line 12: `test_sum_safe_sessions`
  - line 24: `test_engagement_rate_is_recomputed_not_averaged`
  - line 39: `test_active_users_are_never_summed_from_daily_rows`
  - line 51: `test_average_session_duration_is_direct_only`
  - line 63: `test_decimal_text_is_stable`

### 64.12 `tests/unit/analysis/ga4/test_periods.py`

- test count: 2
  - line 58: `test_period_metrics_and_comparisons`
  - line 99: `test_incomplete_coverage_blocks_comparison`

### 64.13 `tests/unit/analysis/ga4/test_reconciliation.py`

- test count: 3
  - line 71: `test_reconciliation_detects_nonzero_dimension_delta`
  - line 126: `test_reconciliation_recomputes_ratio_from_counts`
  - line 178: `test_reconciliation_marks_incomplete_coverage_unavailable`

### 64.14 `tests/unit/analysis/ga4/test_summaries.py`

- test count: 4
  - line 19: `test_acquisition_recomputes_rate`
  - line 54: `test_landing_normalizes_query_variants`
  - line 95: `test_page_summary_uses_additive_metrics_only`
  - line 125: `test_event_summary_does_not_roll_up_distinct_users`

### 64.15 `tests/unit/analysis/gsc/test_metrics.py`

- test count: 2
  - line 9: `test_ctr_is_recomputed_and_position_is_impression_weighted`
  - line 33: `test_position_unavailable_for_discover_and_google_news`

### 64.16 `tests/unit/analysis/gsc/test_period_coverage_semantics.py`

- test count: 3
  - line 9: `test_unavailable_window_returns_no_aggregate`
  - line 35: `test_partial_current_comparison_is_not_ok`
  - line 46: `test_unavailable_baseline_is_explicit`

### 64.17 `tests/unit/analysis/gsc/test_reconciliation_semantics.py`

- test count: 4
  - line 8: `test_reconciliation_match`
  - line 18: `test_reconciliation_nonzero_delta`
  - line 28: `test_position_precision_delta_matches_within_tolerance`
  - line 40: `test_non_position_delta_remains_nonzero`

### 64.18 `tests/unit/common/test_config_paths.py`

- test count: 2
  - line 11: `test_config_roots_are_canonical`
  - line 16: `test_config_path_helpers_are_project_relative`

### 64.19 `tests/unit/common/test_date_ranges.py`

- test count: 4
  - line 12: `test_resolve_relative_dates`
  - line 19: `test_resolve_absolute_date`
  - line 26: `test_date_range_is_inclusive`
  - line 38: `test_rejects_reversed_range`

### 64.20 `tests/unit/common/test_google_auth.py`

- test count: 9
  - line 32: `test_shared_readonly_scopes_include_ga4_and_gsc`
  - line 39: `test_token_has_scopes_accepts_full_scope_set`
  - line 57: `test_token_has_scopes_rejects_legacy_ga4_only_token`
  - line 72: `test_token_has_scopes_supports_space_delimited_string`
  - line 87: `test_token_without_scope_metadata_is_treated_as_insufficient`
  - line 107: `test_normalize_scopes_deduplicates_preserving_order`
  - line 120: `test_normalize_scopes_rejects_empty_scope_set`
  - line 153: `test_valid_cached_credentials_are_reused_in_shared_auth`
  - line 177: `test_expired_cached_credentials_are_refreshed_and_saved_in_shared_auth`

### 64.21 `tests/unit/common/test_page_keys.py`

- test count: 6
  - line 7: `test_normalize_page_uses_site_scoped_path_key`
  - line 15: `test_absolute_and_relative_pages_share_the_same_page_key`
  - line 24: `test_normalize_page_preserves_trailing_slash_difference`
  - line 31: `test_normalize_page_preserves_path_case_and_percent_encoding`
  - line 37: `test_normalize_page_removes_default_port_from_queryless_url`
  - line 44: `test_page_normalization_policy_declares_shared_join_contract`

### 64.22 `tests/unit/common/test_runs.py`

- test count: 6
  - line 16: `test_namespaced_run_ids_are_unambiguous`
  - line 22: `test_run_id_requires_aware_datetime`
  - line 27: `test_parse_run_timestamp_supports_legacy_and_namespaced_ids`
  - line 43: `test_latest_uses_manifest_time_across_legacy_and_new_names`
  - line 55: `test_latest_can_scope_property`
  - line 66: `test_latest_rejects_empty_root`

### 64.23 `tests/unit/extraction/ga4/test_auth.py`

- test count: 3
  - line 19: `test_get_credentials_rejects_unsupported_auth_type`
  - line 32: `test_get_credentials_delegates_to_shared_google_auth`
  - line 62: `test_get_credentials_translates_shared_auth_error`

### 64.24 `tests/unit/extraction/ga4/test_config.py`

- test count: 3
  - line 12: `test_report_filter_and_order_parse`
  - line 48: `test_default_order_is_dimension_order`
  - line 73: `test_invalid_order_reference_rejected`

### 64.25 `tests/unit/extraction/ga4/test_daily_dimension_presets.py`

- test count: 1
  - line 4: `test_analysis_full_has_daily_dimension_reports`

### 64.26 `tests/unit/extraction/ga4/test_exporter.py`

- test count: 5
  - line 49: `test_two_page_export_and_atomic_finalization`
  - line 74: `test_row_count_change_is_error`
  - line 97: `test_zero_rows_before_completion_is_error`
  - line 117: `test_header_drift_is_error`
  - line 144: `test_refuses_overwrite`

### 64.27 `tests/unit/extraction/ga4/test_ga4_query.py`

- test count: 3
  - line 14: `test_build_exact_dimension_filter`
  - line 33: `test_build_dimension_order_bys`
  - line 47: `test_explicit_metric_order_by`

### 64.28 `tests/unit/extraction/ga4/test_planning.py`

- test count: 3
  - line 73: `test_full_compatible_count`
  - line 86: `test_skipped_fallback_count_is_preserved`
  - line 110: `test_fully_skipped_report_reports_all_calls`

### 64.29 `tests/unit/extraction/ga4/test_presets.py`

- test count: 4
  - line 9: `test_event_reports_are_semantically_filtered`
  - line 26: `test_scroll_report_does_not_claim_depth`
  - line 32: `test_key_event_breakdown_reports_exist`
  - line 42: `test_preset_reports_have_deterministic_default_order`

### 64.30 `tests/unit/extraction/ga4/test_quality.py`

- test count: 3
  - line 7: `test_empty_schema_restriction_is_not_warning`
  - line 20: `test_active_schema_restriction_is_warning`
  - line 35: `test_quality_statuses`

### 64.31 `tests/unit/extraction/gsc/test_config.py`

- test count: 5
  - line 13: `test_parse_all_search_types`
  - line 17: `test_parse_subset_deduplicates`
  - line 21: `test_invalid_search_type_is_rejected`
  - line 26: `test_row_limit_is_validated`
  - line 34: `test_yaml_and_cli_merge`

### 64.32 `tests/unit/extraction/gsc/test_coverage.py`

- test count: 4
  - line 10: `test_full_observed_coverage`
  - line 22: `test_missing_date_is_partial`
  - line 34: `test_no_observed_dates_is_empty`
  - line 38: `test_csv_coverage_detects_missing_latest_date`

### 64.33 `tests/unit/extraction/gsc/test_data.py`

- test count: 4
  - line 30: `test_build_query_body_uses_type_not_deprecated_search_type`
  - line 46: `test_request_json_retries_429`
  - line 68: `test_request_json_does_not_retry_400`
  - line 85: `test_pagination_advances_start_row_until_short_page`

### 64.34 `tests/unit/extraction/gsc/test_discovery.py`

- test count: 3
  - line 33: `test_list_sites_parses_entries`
  - line 42: `test_resolve_single_site_automatically`
  - line 47: `test_resolve_multiple_sites_requires_selection`

### 64.35 `tests/unit/extraction/gsc/test_exporter_helpers.py`

- test count: 1
  - line 4: `test_site_key_is_safe_and_deterministic`

### 64.36 `tests/unit/extraction/gsc/test_planning.py`

- test count: 5
  - line 5: `test_search_appearance_with_other_dimension_is_planned_skip`
  - line 15: `test_discover_query_is_planned_skip`
  - line 21: `test_discover_device_is_planned_skip`
  - line 27: `test_google_news_query_is_planned_skip`
  - line 33: `test_web_query_page_is_supported`

### 64.37 `tests/unit/extraction/gsc/test_presets.py`

- test count: 2
  - line 4: `test_analysis_full_report_names_are_unique`
  - line 10: `test_analysis_full_contains_core_grains`

### 64.38 `tests/unit/extraction/gsc/test_probe_quality.py`

- test count: 2
  - line 7: `test_zero_only_probe_is_empty`
  - line 23: `test_probe_with_impressions_has_data`

### 64.39 `tests/unit/integration/analysis/test_cli.py`

- test count: 1
  - line 4: `test_cli_accepts_latest_and_spec`

### 64.40 `tests/unit/integration/analysis/test_config.py`

- test count: 2
  - line 11: `test_default_generic_page_spec_loads`
  - line 21: `test_invalid_metric_star_mix_is_rejected`

### 64.41 `tests/unit/integration/analysis/test_discovery.py`

- test count: 1
  - line 18: `test_find_latest_processing_run`

### 64.42 `tests/unit/integration/analysis/test_processor.py`

- test count: 10
  - line 262: `test_generic_period_total_safe_average_ratio_and_weighted`
  - line 280: `test_comparison_baseline_zero_is_distinct_from_unavailable`
  - line 305: `test_insufficient_coverage_suppresses_values`
  - line 320: `test_ranking_and_contribution_are_generic_and_deterministic`
  - line 356: `test_manifest_catalog_hashes_and_lineage_are_verifiable`
  - line 375: `test_hash_mismatch_is_rejected`
  - line 383: `test_core_has_no_ga4_gsc_or_use_case_branch_literals`
  - line 390: `test_non_additive_context_is_never_summed`
  - line 421: `test_sum_safe_group_unavailable_is_not_coerced_to_zero`
  - line 463: `test_ranking_and_contribution_exclude_unavailable_groups_but_keep_true_zero`

### 64.43 `tests/unit/integration/extraction/test_cli.py`

- test count: 1
  - line 4: `test_cli_requires_independent_ga4_and_gsc_selection`

### 64.44 `tests/unit/integration/extraction/test_config.py`

- test count: 2
  - line 12: `test_load_config_and_hash_are_deterministic`
  - line 35: `test_config_requires_explicit_site_mapping`

### 64.45 `tests/unit/integration/extraction/test_discovery.py`

- test count: 2
  - line 21: `test_find_latest_processing_run_scopes_source_identity`
  - line 39: `test_find_latest_processing_run_rejects_missing`

### 64.46 `tests/unit/integration/extraction/test_processor.py`

- test count: 6
  - line 265: `test_validate_inputs_rejects_site_mapping_mismatch`
  - line 278: `test_extract_is_lossless_and_aggregation_safe`
  - line 334: `test_manifest_has_direct_parent_hashes_and_relative_paths`
  - line 361: `test_url_policy_mismatch_fails_before_output`
  - line 377: `test_catalog_row_counts_and_hashes_match_outputs`
  - line 396: `test_catalog_contract_uses_real_columns_and_only_allowed_metric_semantics`

### 64.47 `tests/unit/integration/processing/test_config.py`

- test count: 2
  - line 16: `test_shipped_m2_configs_load_and_hash_deterministically`
  - line 31: `test_ratio_dependency_must_be_selected`

### 64.48 `tests/unit/integration/processing/test_discovery.py`

- test count: 1
  - line 20: `test_latest_discovery_can_scope_by_site`

### 64.49 `tests/unit/integration/processing/test_processor.py`

- test count: 9
  - line 235: `test_common_observed_safe_aggregation_and_filter_tracking`
  - line 281: `test_all_available_preserves_union_and_not_observed`
  - line 303: `test_conflicting_non_additive_context_fails`
  - line 314: `test_filter_reason_is_recorded`
  - line 347: `test_period_modes_resolve_without_silent_fallback`
  - line 392: `test_comparison_ready_requires_current_previous_and_yoy_coverage`
  - line 408: `test_deterministic_non_manifest_artifacts`
  - line 436: `test_strict_coverage_policy_fails_below_minimum`
  - line 456: `test_exclude_incomplete_tracks_coverage_row_removal`

### 64.50 `tests/unit/pipeline/test_cli.py`

- test count: 4
  - line 4: `test_top_level_cli_parses_full_plan`
  - line 11: `test_full_cli_parses_resume_and_manual_extraction_override`
  - line 27: `test_full_from_extraction_cli_parses_inputs`
  - line 44: `test_top_level_cli_parses_processing_inputs_for_integration`

### 64.51 `tests/unit/pipeline/test_config.py`

- test count: 1
  - line 6: `test_default_pipeline_config_loads`

### 64.52 `tests/unit/pipeline/test_execution.py`

- test count: 3
  - line 13: `test_artifact_locator_uses_configured_parent_depth`
  - line 31: `test_existing_artifact_rejects_missing_manifest`
  - line 40: `test_resolve_console_script_uses_active_launcher_directory`

### 64.53 `tests/unit/pipeline/test_manifest.py`

- test count: 1
  - line 30: `test_pipeline_manifest_v2_records_stage_attempt_history`

### 64.54 `tests/unit/pipeline/test_orchestrator.py`

- test count: 9
  - line 16: `test_render_full_plan_does_not_contain_source_analysis`
  - line 23: `test_render_full_from_extraction_plan_is_no_fetch`
  - line 31: `test_render_integration_plan_is_no_fetch`
  - line 38: `test_unknown_profile_is_rejected`
  - line 43: `test_pipeline_options_default_output_root`
  - line 86: `test_failed_stage_is_persisted_with_resume_hint`
  - line 122: `test_resume_reuses_success_and_retries_failed_stage`
  - line 186: `test_full_resume_accepts_supplied_failed_extraction`
  - line 256: `test_full_from_extraction_requires_both_inputs`

### 64.55 `tests/unit/pipeline/test_profiles.py`

- test count: 5
  - line 4: `test_profiles_are_exactly_the_six_public_profiles`
  - line 15: `test_full_skips_source_analysis`
  - line 31: `test_full_from_extraction_begins_at_processing`
  - line 46: `test_sources_finishes_with_two_source_analyses`
  - line 53: `test_integration_has_no_live_fetch`

### 64.56 `tests/unit/processing/ga4/test_date_passthrough.py`

- test count: 1
  - line 4: `test_processor_source_keeps_date_bearing_tables`

### 64.57 `tests/unit/processing/ga4/test_ga4_date_contract.py`

- test count: 5
  - line 12: `test_ga4_processing_canonicalizes_compact_source_date`
  - line 16: `test_ga4_processing_accepts_already_iso_source_date`
  - line 20: `test_ga4_processing_rejects_invalid_source_date`
  - line 25: `test_analysis_consumes_canonical_processing_date`
  - line 29: `test_analysis_does_not_reprocess_raw_ga4_compact_date`

### 64.58 `tests/unit/processing/ga4/test_processing_config.py`

- test count: 3
  - line 11: `test_default_processing_config_has_safe_defaults`
  - line 18: `test_event_groups_are_config_driven`
  - line 37: `test_event_cannot_belong_to_multiple_groups`

### 64.59 `tests/unit/processing/ga4/test_processing_events.py`

- test count: 2
  - line 20: `test_event_classification_is_exact_and_config_driven`
  - line 26: `test_event_catalog_records_reports_and_occurrences`

### 64.60 `tests/unit/processing/ga4/test_processing_lineage_layout.py`

- test count: 3
  - line 10: `test_processing_output_directory_is_flat_by_run`
  - line 30: `test_source_quality_provenance_inherits_degraded_report`
  - line 55: `test_source_quality_provenance_is_explicit_when_manifest_entry_missing`

### 64.61 `tests/unit/processing/ga4/test_processing_periods.py`

- test count: 2
  - line 8: `test_parse_ga4_date_supports_compact_and_iso`
  - line 13: `test_period_dataset_tags_without_aggregating_metrics`

### 64.62 `tests/unit/processing/ga4/test_processing_processor.py`

- test count: 1
  - line 22: `test_process_extraction_run_preserves_raw_and_writes_separate_artifacts`

### 64.63 `tests/unit/processing/ga4/test_processing_urls.py`

- test count: 3
  - line 4: `test_path_query_is_removed_but_raw_semantics_are_conservative`
  - line 18: `test_absolute_url_uses_same_site_scoped_page_key`
  - line 30: `test_not_set_is_not_invented_into_path_hierarchy`

### 64.64 `tests/unit/processing/gsc/test_page_normalization_contract.py`

- test count: 1
  - line 5: `test_ga4_and_gsc_share_the_same_normalized_page_key`

### 64.65 `tests/unit/processing/gsc/test_periods.py`

- test count: 3
  - line 8: `test_period_windows_and_coverage`
  - line 29: `test_missing_latest_observed_date_marks_current_partial`
  - line 51: `test_internal_missing_date_prevents_full_coverage`

### 64.66 `tests/unit/visualization/test_visualization.py`

- test count: 18
  - line 315: `test_generic_contract_adapter_accepts_unknown_result_without_code_specific_adapter`
  - line 333: `test_generic_contract_unknown_result_renders_static_and_react`
  - line 358: `test_integration_adapter_builds_generic_contract`
  - line 371: `test_static_renderer_preserves_unavailable`
  - line 395: `test_renderer_is_source_independent`
  - line 410: `test_spec_is_versioned`
  - line 417: `test_processor_writes_static_and_react_outputs`
  - line 442: `test_processor_output_payload_is_deterministic_except_manifest`
  - line 476: `test_manifest_hashes_match_outputs`
  - line 497: `test_react_source_has_required_interactions`
  - line 517: `test_visualization_contract_adds_japanese_metric_presentation`
  - line 542: `test_static_renderer_is_japanese_explained_and_has_visible_axes`
  - line 588: `test_react_source_has_japanese_explanations_crosshair_and_dynamic_zoom`
  - line 615: `test_dashboard_html_is_react_primary_and_static_is_secondary`
  - line 648: `test_react_main_ui_has_dynamic_axis_hover_and_bottom_audit`
  - line 684: `test_dashboard_html_accepts_root_level_vite_app_js`
  - line 713: `test_artifact_root_has_two_entries_and_downloads_original_input_artifact`
  - line 758: `test_primary_dashboard_references_nested_react_assets`


## 65. Artifact Filename / Extension Hints Found in Source

### `src/web_intelligence/analysis/ga4/analyzer.py`

- `acquisition/source_medium_summary.csv`
- `analysis_manifest.json`
- `comparisons/period_comparisons.csv`
- `events/event_summary.csv`
- `events/measurement_continuity.csv`
- `landing/landing_page_summary.csv`
- `overview/metric_semantics.csv`
- `overview/period_metrics.csv`
- `pages/page_summary.csv`
- `periods/period_coverage.csv`
- `periods/period_windows_daily.csv`
- `processing_manifest.json`
- `quality/source_quality.csv`
- `reconciliation/overall_vs_dimension.csv`
- `tables/events_by_page.csv`
- `tables/events_daily.csv`
- `tables/key_events_by_page.csv`
- `tables/landing_page_acquisition.csv`
- `tables/landing_pages.csv`
- `tables/landing_pages_daily.csv`
- `tables/pages.csv`
- `tables/pages_daily.csv`
- `tables/session_acquisition_daily.csv`

### `src/web_intelligence/analysis/ga4/cli.py`

- `*/*/ga4_process_*/processing_manifest.json`
- `*/ga4_process_*/processing_manifest.json`
- `{property_id}/*/ga4_process_*/processing_manifest.json`
- `{property_id}/ga4_process_*/processing_manifest.json`

### `src/web_intelligence/analysis/ga4/dimension_periods.py`

- `acquisition/source_medium_period_comparisons.csv`
- `events/event_period_comparisons.csv`
- `landing/landing_page_period_comparisons.csv`
- `pages/page_period_comparisons.csv`
- `tables/events_daily.csv`
- `tables/landing_pages_daily.csv`
- `tables/pages_daily.csv`
- `tables/session_acquisition_daily.csv`

### `src/web_intelligence/analysis/ga4/reconciliation.py`

- `overview/period_metrics.csv`

### `src/web_intelligence/analysis/gsc/analyzer.py`

- `*/gsc_process_*/processing_manifest.json`
- `analysis_manifest.json`
- `appearance/search_appearance_summary.csv`
- `countries/country_period_comparisons.csv`
- `countries/country_summary.csv`
- `devices/device_period_comparisons.csv`
- `devices/device_summary.csv`
- `full_range_vs_daily.csv`
- `metric_semantics.csv`
- `overall_vs_dimension.csv`
- `overview/metric_semantics.csv`
- `pages/page_period_comparisons.csv`
- `pages/page_summary.csv`
- `period_comparisons.csv`
- `period_metrics.csv`
- `processing_manifest.json`
- `quality/source_quality.csv`
- `queries/query_period_comparisons.csv`
- `queries/query_summary.csv`
- `query_pages/query_page_period_comparisons.csv`
- `query_pages/query_page_summary.csv`
- `source_quality.csv`

### `src/web_intelligence/common/runs.py`

- `*/*/manifest.json`
- `*/manifest.json`

### `src/web_intelligence/extraction/ga4/cli.py`

- `dimensions.csv`
- `manifest.json`
- `metadata.json`
- `metrics.csv`
- `profiles.yaml`
- `reports.yaml`
- `schema/dimensions.csv`
- `schema/metrics.csv`

### `src/web_intelligence/extraction/ga4/exporter.py`

- `{safe_name}.csv`

### `src/web_intelligence/extraction/gsc/config.py`

- `credentials/default_token.json`
- `credentials/google_oauth_client.json`

### `src/web_intelligence/extraction/gsc/exporter.py`

- `manifest.json`
- `metadata.json`
- `{report_name}.csv`

### `src/web_intelligence/integration/analysis/discovery.py`

- `*/integration_process_*/processing_manifest.json`
- `{site_key}/integration_process_*/processing_manifest.json`

### `src/web_intelligence/integration/analysis/processor.py`

- `analysis_manifest.json`
- `analysis_quality.json`
- `analysis_spec_snapshot.json`
- `analysis_summary.json`
- `comparisons.csv`
- `contribution.csv`
- `coverage_status.csv`
- `coverage_summary.json`
- `dataset_schema.json`
- `metric_registry_snapshot.json`
- `period_definition.json`
- `period_metrics.csv`
- `processing_manifest.json`
- `ranking.csv`
- `result_catalog.json`
- `trend.csv`

### `src/web_intelligence/integration/extraction/discovery.py`

- `{run_prefix}_*/processing_manifest.json`

### `src/web_intelligence/integration/extraction/processor.py`

- `canonical_pages.csv`
- `dataset_catalog.json`
- `integration_manifest.json`
- `integration_quality.csv`
- `manifest.json`
- `page_daily_ga4.csv`
- `page_daily_gsc.csv`
- `page_daily_integrated.csv`
- `processing_manifest.json`
- `query_page_daily_gsc.csv`
- `query_page_daily_integrated.csv`
- `tables/daily_overview.csv`
- `tables/pages_daily.csv`

### `src/web_intelligence/integration/processing/discovery.py`

- `*/integration_extract_*/integration_manifest.json`
- `{site_key}/integration_extract_*/integration_manifest.json`

### `src/web_intelligence/integration/processing/processor.py`

- `coverage_summary.json`
- `dataset_schema.json`
- `filter_summary.csv`
- `integration_manifest.json`
- `metric_registry_snapshot.json`
- `period_definition.json`
- `processed_dataset.csv`
- `processing_manifest.json`

### `src/web_intelligence/pipeline/config.py`

- `default.yaml`

### `src/web_intelligence/pipeline/manifest.py`

- `pipeline_manifest.json`

### `src/web_intelligence/pipeline/orchestrator.py`

- `manifest.json`
- `pipeline_manifest.json`
- `processing_manifest.json`

### `src/web_intelligence/pipeline/stages/integration.py`

- `*/integration_analysis_*/analysis_manifest.json`
- `*/integration_extract_*/integration_manifest.json`
- `*/integration_process_*/processing_manifest.json`

### `src/web_intelligence/pipeline/stages/source.py`

- `*/ga4_analysis_*/analysis_manifest.json`
- `*/ga4_extract_*/manifest.json`
- `*/ga4_process_*/processing_manifest.json`
- `*/gsc_analysis_*/analysis_manifest.json`
- `*/gsc_extract_*/manifest.json`
- `*/gsc_process_*/processing_manifest.json`

### `src/web_intelligence/pipeline/stages/visualization.py`

- `*/visualization_*/dashboard_data/visualization_manifest.json`

### `src/web_intelligence/processing/ga4/cli.py`

- `processing.yaml`

### `src/web_intelligence/processing/ga4/config.py`

- `daily_overview.csv`

### `src/web_intelligence/processing/ga4/processor.py`

- `*.csv`
- `event_catalog.csv`
- `manifest.json`
- `period_coverage.csv`
- `period_windows_daily.csv`
- `processing_manifest.json`

### `src/web_intelligence/processing/gsc/processor.py`

- `*/gsc_extract_*/manifest.json`
- `]}.csv`
- `manifest.json`
- `processing_manifest.json`

### `src/web_intelligence/visualization/adapters/__init__.py`

- `analysis_manifest.json`

### `src/web_intelligence/visualization/adapters/base.py`

- `analysis_manifest.json`

### `src/web_intelligence/visualization/adapters/ga4_analysis.py`

- `metric_semantics.csv`
- `page_summary.csv`
- `period_comparisons.csv`
- `period_metrics.csv`
- `source_quality.csv`

### `src/web_intelligence/visualization/adapters/generic_contract.py`

- `visualization_input.json`

### `src/web_intelligence/visualization/adapters/gsc_analysis.py`

- `metric_semantics.csv`
- `page_summary.csv`
- `period_comparisons.csv`
- `period_metrics.csv`
- `source_quality.csv`

### `src/web_intelligence/visualization/adapters/integration_analysis.py`

- `analysis_quality.json`
- `analysis_summary.json`
- `comparisons.csv`
- `contribution.csv`
- `coverage_status.csv`
- `period_metrics.csv`
- `ranking.csv`
- `result_catalog.json`
- `trend.csv`

### `src/web_intelligence/visualization/cli.py`

- `config/system/visualization/default_dashboard.yaml`

### `src/web_intelligence/visualization/processor.py`

- `analysis_manifest.json`
- `dashboard.html`
- `dashboard_data/dashboard_static.html`
- `dashboard_data/react/index.html`
- `dashboard_data/visualization_input.json`
- `dashboard_data/visualization_spec_snapshot.json`
- `dashboard_static.html`
- `index.html`
- `visualization_input.json`
- `visualization_manifest.json`
- `visualization_spec_snapshot.json`
- `{input_run.name}.zip`

