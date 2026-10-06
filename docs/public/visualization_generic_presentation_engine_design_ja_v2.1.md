# 汎用可視化プレゼンテーションエンジン 設計書

```yaml
document_id: visualization_generic_presentation_engine_design
document_type: design
doc_ver: v2.1
language: ja
project_name: Generic Web Intelligence & Decision Platform
project_initiated_at: 2026-09-23 17:35:34 JST
created_at: 2026-10-01 JST
updated_at: 2026-10-05 22:57 JST
author: Nazuna Research
status: active
scope: MVP implemented visualization boundary + future BI skeleton
note: v2.0までの設計をlossless保持し、M5 percentage correctnessとMVP COMPLETE状態を現行化
previous_note: 2026-10-01設計原本をlossless保存し、M4 closure後の実装状態/M5項目を追加
preserved_source_sha256: b51bd33682b24c741c454868f69c9bd301462377eeb7210526364f85b1bd97be
supersedes: VISUALIZATION_GENERIC_PRESENTATION_ENGINE_MVP_AND_FUTURE_DESIGN_20261001.md
previous_public_version: visualization_generic_presentation_engine_design_ja_v2.0.md
mvp_completion_state: COMPLETE
current_package: 0.12.4
public_privacy_review: PASS — public copy removes personal author identity and replaces known site-specific example identifiers with non-identifying reserved examples
```

## 0.1 MVP Completion / M5 Visualization Closure — v0.12.4

VisualizationのArchitecture境界は変更せず、M5 correctnessまで完了した。

- React primary: maintained
- Static HTML: secondary audit copy
- same Visualization Input / Spec contract: maintained
- no reanalysis: maintained
- original input Artifact download/audit: maintained
- M5: PASS
- MVP COMPLETE: YES

### relative delta percentage fix

M5実Artifact監査で、React primary comparison tableの`relative_delta`がratioであるにもかかわらず、その値へ直接`%`を付ける経路を検出した。

```text
internal ratio:    -0.3105
incorrect display: -0.3105%
correct display:   -31.05%
```

v0.12.4では`formatValue(..., 'ratio')`相当のratio formatterを通す。Static audit側が持っていたratio semanticsとReact primaryを一致させ、regression testを追加した。

この修正はPresentation Layerによる新規分析ではない。upstream Resultに保存されたratioの**表示単位を正しく表現するformat contract correction**である。

### M5 Visualization gate

- final Visualization Artifact/contract/hash: PASS
- original input Artifact lineage: PASS
- generated CSV parse: PASS as part of whole M5/R12 auditing
- portability / secret marker: PASS
- percentage formatter correctness: PASS

Post-MVP polish候補（KPI layout、raw JSON accordion、integer tick等）は引き続きDeferredであり、MVP completionを再openしない。

## 0. 現在の位置づけ

VisualizationはM4で実装・manual reviewまで完了し、MVPとしてclosure済み。

元設計書の「Integration専用ではないGeneric Presentation Engine」という境界は維持され、現在実装ではGA4 Analysis / GSC Analysis / Integration Analysis / Generic Contract入力を同じcontractへ投影する。

## 1. 実装済みMVP Architecture

```text
Analysis / Result Artifact
        ↓
Adapter
        ↓
Generic Visualization Input Contract
        ↓
Visualization Spec
        ↓
Renderer
   ├─ React primary
   └─ Static audit
```

Renderer CoreへSource-specific branchを埋め込まない。

## 2. 対応入力

- GA4 Analysis
- GSC Analysis
- Integration Analysis
- Generic Contract Adapterによる未知/外部Result Artifact

Generic Contract Adapterは新しいPython source-specific adapterを毎回追加せず、契約準拠Artifactを取り込める境界。

## 3. React実装

- React 19.3.0
- ReactDOM 19.3.0
- Vite 8.3.1
- chart libraryなし

React UIとStatic HTMLは同じInput/Specを使う。

## 4. Time Series interaction

MVP closure時点で以下を持つ。

- axes / ticks / unit / date
- crosshair
- hover glow
- drag zoom
- reset
- normal monthly ticks
- zoom時 weekly -> daily
- diagonal labels
- Y軸概ね8 ticks

## 5. Artifact layout

```text
visualization_<run_id>/
├── dashboard.html
└── dashboard_data/
    ├── dashboard_static.html
    ├── react/
    ├── visualization_input.json
    ├── visualization_spec_snapshot.json
    ├── visualization_manifest.json
    └── <original_input_artifact_name>.zip
```

root exactly 2 entriesをcontractとする。

## 6. Download contract

Download iconはVisualization Artifactではなく**original input Artifact ZIP**を対象とする。ZIPはinput Artifactとfile-set/bytesの整合をとり、hashをManifestへ保存する。

Browserの`<a download>`に従うためOS download folderの強制指定はしない。

## 7. Quality / Coverage / Lineage

React primary UIの下部に表示。raw JSONを完全に隠さず監査可能性を維持する。

Post-MVPではsummaryを残したままraw JSON部分をaccordionにする候補。

## 8. M5で確認済みのcorrectness

relative deltaがratioの場合のpercent表示。

```text
ratio -0.3105 -> -31.05%
```

`value_type / unit / format_hint`とformatterの契約を実Artifactで検証し、v0.12.4でPASSした。

## 9. Post-MVP polish候補

- desktop KPI 5+2 -> 4+3
- Quality/Coverage/Lineage raw JSON accordion
- count-type Y tickを整数化
- header technical metadataをsecondary化

これらはVisualization closureを解除するMVP blockerではない。

## 10. 将来BI Skeleton

元設計どおり、Website / Modeling / Causal / Decision / Business KPI / Ads / CRM等のResult Artifactを同じPresentation boundaryへ接続できるようにする。

MVPではdashboard builder、arbitrary SQL、semantic query、drag-and-drop BIまで作らない。


## 保存付録 — Visualization Generic Presentation Engine 2026-10-01 原本全文

この付録は、更新前の原本文書に存在した情報を不可逆に失わないための保存領域である。
**現行仕様の判断には本文前半を使用し、この付録は履歴・判断根拠・旧仕様の追跡に使用する。**
原文は内容を要約せず、そのまま保存している。

- preserved_source_sha256: `b51bd33682b24c741c454868f69c9bd301462377eeb7210526364f85b1bd97be`

````markdown
# Visualization Generic Presentation Engine

```yaml
project_initiated_at: 2026-09-23 17:35:34 JST
created_at: 2026-10-01 JST
updated_at: 2026-10-01 JST
author: Nazuna Research
doc_ver: 
note: 
```

## MVP Boundary & Future BI Skeleton Design

- 固定日: 2026-10-01
- 状態: **ユーザー最新指示により固定されたVisualization設計方針**
- 対象Phase: MVP M4 Visualization、およびPost-MVP Visualization / BI拡張
- 性格: 全体要件定義書 v2.0 を置換するものではない。Visualization領域について、2026-10-01時点の最新ユーザー指示をlosslessに固定する補助設計書。
- 優先順位: ユーザー最新明示指示 > 全体要件定義書 v2.0 > 本文書 > Roadmap / Handoff / older docs

---

# 0. この文書の目的

この文書は、2026-10-01の会話で追加・確定したVisualization層の設計方針を、MVP実装開始前に失わないための固定文書である。

今回の変更点の核心は以下。

> **VisualizationはIntegration層の下位層ではない。**
>
> **Visualizationは、一定の表示契約を満たす任意のAnalysis / Result Artifactを描画する、独立したGeneric Presentation Engineとする。**

MVPでは最初の対象入力として以下3系統を扱える骨格を作る。

1. GA4 Analysis Artifact
2. GSC Analysis Artifact
3. Integration Analysis Artifact

しかし設計はこの3種類へ固定しない。

将来は、Website Analysis、Modeling Result、Causal Result、Decision Support Result、Business KPI、Ads / CRM、外部CSV / Parquetなど、**別の何か** が来ても、Visualization本体の大改造なしに受けられる拡張境界を持たせる。

この汎用化により、将来的にVisualization層単体を独立BIツールへ発展させられる骨格を残す。

ただしMVPでBIツールそのものを作らない。

---

# 1. 変更前の見え方と、今回固定する新しい境界

## 1.1 これまで強く見えていた構造

旧来のMVPチェーンだけを見ると、Visualizationは次のように見えやすかった。

```text
Integration Extraction
  ↓
Integration Processing
  ↓
Integration Analysis
  ↓
Visualization
```

この表現自体はIntegration E2Eの一経路としては正しい。

しかし、VisualizationをIntegration Analysis専用層として実装してはいけない。

## 1.2 今回固定する構造

Visualizationは次の3経路を独立して受けられるようにする。

```text
GA4 Processing
  ↓
GA4 Analysis
  ↓
Visualization

GSC Processing
  ↓
GSC Analysis
  ↓
Visualization

GA4 Processing ─┐
                ├─> Integration Extraction
GSC Processing ─┘
  ↓
Integration Processing
  ↓
Integration Analysis
  ↓
Visualization
```

より抽象化すると以下。

```text
GA4 Analysis ─────────────┐
                          │
GSC Analysis ─────────────┼──> Generic Visualization Layer
                          │
Integration Analysis ─────┘
```

そして将来は以下へ拡張可能にする。

```text
Any Analysis / Result Artifact ──> Generic Visualization Layer
```

## 1.3 絶対境界

Visualizationの存在理由は「Integration結果を表示すること」ではない。

正しくは以下。

> **Visualization = Result Artifactを表示契約へ変換し、汎用的に描画する独立Presentation Layer**

IntegrationはVisualizationの1つの入力元にすぎない。

GA4 Analysisも、GSC Analysisも、Integrationを経由せず直接Visualizationへ渡せる。

GA4単体、GSC単体の可視化のためにIntegration層を通す設計は禁止。

---

# 2. Visualization層の設計原則

## 2.1 Source / Integration非依存

Renderer本体は、入力がGA4かGSCかIntegrationかを知る必要がない。

避けるべき実装例:

```python
if analysis_source == "ga4":
    ...
elif analysis_source == "gsc":
    ...
elif analysis_source == "integration":
    ...
```

このようなSource専用dashboard分岐をVisualization Coreに埋め込まない。

## 2.2 Rendererは意味解釈より表示契約を読む

Rendererは以下のようなSource固有知識で描画判断しない。

```text
"GA4 sessionsだから折れ線"
"GSC clicksだから検索用chart"
```

代わりに、Source非依存な表示metadata / Visualization Specを読む。

例:

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

Rendererはこの契約から表示する。

## 2.3 Visualizationは再分析しない

既存ADR 007を維持する。

> Visualization Does Not Re-analyze

VisualizationはAnalysis層の計算ロジックを再実装しない。

Visualizationが勝手に以下を行わない。

- 集計ロジックの再定義
- metric semanticの変更
- 分母・分子の再解釈
- coverage補正
- missing値の勝手な0化
- Analysis statusの上書き
- prediction / causal / observedの意味混同

表示のための並べ替え、format、pagination、presentation-only transformは許容されるが、分析結果の意味を変更してはいけない。

## 2.4 Quality / Coverage / Lineageを一級情報として表示する

MVP要件のまま、Visualizationは数値だけでなく以下を表示可能にする。

- quality
- coverage
- observation state
- status
- lineage
- input artifact / run identity
- spec / config identity where applicable

「値だけ綺麗に表示して根拠を隠す」dashboardにしない。

## 2.5 Evidence / Result Typeを保持する

全体要件v2.0のResult / Evidence TypesをPresentationでも失わない。

- OBSERVED
- DERIVED
- HEURISTIC
- STATISTICAL
- PREDICTED
- CAUSAL_ESTIMATE
- LLM_GENERATED

将来Decision Support Resultを扱う場合も、上記の違いを潰して単なるKPIとして表示してはいけない。

---

# 3. MVP M4で必須とする範囲

## 3.1 MVPで最初に対応する入力

MVP時点の初期対象は以下3系統。

```text
GA4 Analysis Artifact
GSC Analysis Artifact
Integration Analysis Artifact
```

重要なのは「3種類専用Renderer」を作ることではない。

MVPで作るべきものは、**3種類を同じVisualization Coreへ載せられる共通境界**である。

## 3.2 MVPの推奨構造

```text
Analysis / Result Artifact
        ↓
Artifact-specific Adapter
        ↓
Generic Visualization Input Contract
        ↓
Visualization Spec
        ↓
Generic Renderer
        ↓
Static HTML
```

この構造により、Source固有差分はCore RendererではなくAdapter境界へ押し込める。

## 3.3 MVP Adapterの役割

Adapterは次を担える。

- Artifact manifestの読み取り
- dataset catalog / schemaの読み取り
- Analysis output metadataの読み取り
- metric metadataをGeneric Visualization Input Contractへ投影
- dimension / time / status / coverage / lineage metadataの投影
- 既存Analysis Artifactの差分吸収

Adapterは分析をしない。

AdapterはSource固有metricを再計算しない。

Adapterの目的は契約変換であり、意味変更ではない。

## 3.4 既存GA4/GSC Analysis Artifactへの対応方法

2026-10-01時点では、GA4 Analysis / GSC Analysis Artifactが将来のGeneric Visualization Input Contractを完全に満たしているとは決め打ちしない。

M4実装開始時にcurrent repoの実Artifactを改めて監査し、以下を確認してから最小実装を決める。

- manifest structure
- dataset structure
- schema
- metric metadata
- status / quality / coverage
- lineage
- date / grain semantics
- current-vs-previous / ranking / summary等の出力構造

必要なら最小adapterを作る。

必要ならAnalysis側へ最小共通metadataを追加する。

しかし、M4開始前に想像で大規模共通schemaへ作り直さない。

## 3.5 Static HTMLをMVP Canonical Outputとする

全体要件v2.0のとおり、MVPではStatic HTMLをCanonical Visualization Outputとする。

最低限想定する表示能力:

- KPI
- time series
- comparison
- ranking
- table
- quality / lineage
- coverage / status

必要に応じて以下を扱える骨格を持てる。

- bar
- scatter
- filter
- small multiple

ただし、表示機能追加のためにAnalysisをVisualizationへ移植しない。

## 3.6 ReactはMVP必須ではない

2026-10-01時点の固定方針:

- Static HTMLを第一優先とする。
- ReactをMVP完了条件にしない。
- 将来必要そうだからという理由だけでReactを導入しない。
- Static HTMLで十分ならReact版は作らない。
- ただし、意味のあるclient-side dynamic behaviorが必要になり、vanilla JavaScriptが成長し始めるなら、vanilla JSを積み上げずReactへ進む方を優先する。
- Reactを導入する場合でもGeneric Visualization Input Contract / Visualization SpecはUI frameworkから独立させる。

したがって、ReactはPresentation implementation optionであり、Core Contractではない。

---

# 4. Generic Visualization Input Contractの骨格

これは2026-10-01時点の骨格設計であり、M4開始時にcurrent Analysis Artifactを監査した上で最終schemaを固定する。

## 4.1 Contractの目的

任意のAnalysis / Result Artifactを、RendererがSource固有知識なしで扱える形へ変換する。

最低限、以下の情報群を表現可能にする。

### Artifact identity

- artifact_type
- artifact_version
- run_id
- parent / source lineage
- generated_at
- schema_version

### Result identity

- result_type
- dataset_id
- logical_name
- display_name
- description

### Grain / dimensions

- grain
- dimensions
- dimension types
- time dimension
- entity / page / query / site等のdimension metadata

### Metrics

- metric_id
- display_name
- description
- value_type
- unit
- semantic_type
- aggregation behavior
- format hint
- availability / status semantics

### Coverage / status

- requested coverage
- resolved coverage
- observed coverage
- missing / unavailable semantics
- result status
- warnings

### Quality

- validation status
- duplicate / integrity / lineage quality
- quality summaries
- source-specific quality translated without losing meaning

### Lineage

- parent artifact IDs
- parent manifest hashes
- config / spec identity
- dataset hashes or references

### Visualization hints

- preferred display form
- chart_hint
- sort hint
- axis / grouping hint
- compact / percent / currency / duration等のformat hint

Visualization hintは強制命令ではない。

Rendererは利用可能なデータと契約に応じて安全なfallbackを持てる。

## 4.2 Metric semanticをPresentationで保持する

既存Metric semanticsを勝手に増やしたり意味変更しない。

- SUM_SAFE
- DERIVED_RATIO
- NON_ADDITIVE
- DIRECT_ONLY
- WEIGHTED_WITHIN_GRAIN
- UNAVAILABLE
- NON_ADDITIVE_CONTEXT

Visualizationはsemanticを見て、例えばNON_ADDITIVE_CONTEXTを単純合算するようなUIを許さない設計へ進化できる。

MVPでは最低限、semanticを失わず表示contractへ伝播する。

---

# 5. Visualization Specの骨格

## 5.1 Input ContractとSpecを分ける理由

Generic Visualization Input Contractは「Result Artifactが何を意味するか」を表す。

Visualization Specは「どう見せるか」を表す。

この2つを混ぜない。

```text
Data / Result Meaning
    = Visualization Input Contract

Presentation Choice
    = Visualization Spec
```

## 5.2 Generic Specの例

将来、以下のようなpresentation typeを扱える。

```text
kpi
line
bar
table
ranking
scatter
distribution
heatmap
small_multiple
quality_panel
lineage_panel
```

MVPで全部実装する必要はない。

重要なのはSource名に依存しないspecであること。

## 5.3 Spec生成の責務

MVPではRule-based / deterministicでよい。

例:

```text
time_dimensionあり
+ numeric metric
+ semantic compatible
→ line candidate
```

```text
categorical dimension
+ additive metric
→ bar / ranking candidate
```

ただしSpec生成がAnalysisの代わりにmetric計算をしてはいけない。

## 5.4 将来のユーザー指定

Post-MVPでは、同じInput Contractに対してユーザー指定Specを適用できる構造へ拡張可能にする。

例:

- metric selection
- dimension selection
- chart selection
- sort
- filter
- grouping
- dashboard layout

ただしこれらはMVP必須ではない。

---

# 6. Generic Rendererの骨格

## 6.1 Rendererの責務

RendererはVisualization Specを受け、静的または動的なpresentationを生成する。

MVP Canonical Renderer:

```text
Static HTML Renderer
```

将来追加可能:

```text
React Renderer
Embedded UI Renderer
BI Workspace Renderer
Export Renderer
```

## 6.2 Renderer Coreが知らないもの

Renderer Coreに以下をハードコードしない。

- GA4 property semantics
- GSC search semantics
- Integration-specific join logic
- Website-specific analysis meaning
- Model training logic
- causal estimation logic
- LLM prompt logic

Rendererは契約を表示する。

## 6.3 Portable Output

MVP Static HTMLは可能な限りportableにする。

全体要件v2.0に従い、ローカルabsolute pathやsecretを埋め込まない。

Artifact / lineage情報を表示する場合もportable referenceを使う。

---

# 7. Extension Hook / Adapter Boundary

## 7.1 目的

今回ユーザーが明示した重要要件:

> GA4 / GSC / Integrationの3種類以外の「何か」が来ても、ある程度なんでも対応できるように超汎用化する。

MVPで完全なplugin systemを作る必要はない。

しかし、将来入力種別追加のたびにRenderer Coreを書き換えなくて済む境界はMVPから用意する。

## 7.2 MVPで残すべきHookの最小形

実装形はM4時点のrepoに合わせるが、概念的には以下。

```text
Artifact Detector / Adapter Registry
        ↓
Adapter Interface
        ↓
Generic Visualization Input Contract
```

最低限、adapter追加によって新Artifactを受けられる構造にする。

MVPでdynamic plugin loadingや外部package discoveryまで要求しない。

単純なregistry / protocol / interfaceでもよい。

重要なのはCore RendererとSource-specific codeの分離。

## 7.3 Adapter追加時の原則

新adapterは次を満たす。

- source-specific reading
- contract validation
- metadata mapping
- no hidden re-analysis
- no silent zero filling
- no semantic loss
- explicit unsupported states
- deterministic conversion

---

# 8. 将来の対応候補

Visualization Layerは将来、以下を同じ骨格へ載せられるようにする。

## 8.1 Platform-native Results

- GA4 Analysis
- GSC Analysis
- Integration Analysis
- Website Analysis
- Modeling Result
- Forecast Result
- Anomaly Result
- Clustering Result
- Ranking Result
- Causal Result
- Experiment Result
- Decision Support Result
- LLM Generated Explanation / Synthesis

## 8.2 Business / External Results

- Business KPI
- Ads
- CRM
- Revenue
- Offline conversion
- Manual annotation result
- external aggregated dataset

## 8.3 Generic data inputs for future BI evolution

将来的にはAdapter経由で、以下のような一般datasetも候補になる。

- CSV
- Parquet
- SQLite / DuckDB result
- API-derived result
- external table/result artifact

ただしMVP M4は「任意CSVをBI分析する機能」を作るPhaseではない。

MVPではGeneric ContractとAdapter boundaryを作り、将来その入口を増やせるようにする。

---

# 9. BI Tool化の将来骨格

## 9.1 長期方向

Visualization層を十分にSource非依存へ保てば、将来的に以下の流れへ拡張できる。

```text
Data / Result Artifact ingestion
        ↓
Schema / Metric Metadata Interpretation
        ↓
Visualization Input Contract
        ↓
Visualization Spec Generation
        ↓
Dashboard Composition
        ↓
Interactive BI UI
```

この時点でVisualizationは単なる最終ページではなく、独立したBI基盤へ進化できる。

## 9.2 将来可能な機能

Post-MVP以降の候補:

- interactive filter
- multi-chart dashboard
- drill-down
- drill-through
- saved view
- reusable visualization spec
- dashboard layout editor
- user-defined metric selection
- dimension selection
- date range selection
- comparison period selection
- workspace
- share/export
- access control
- external dataset connection
- semantic catalog browsing
- query layer

ただし、これらは2026-10-01時点でMVP要件ではない。

## 9.3 BI化しても守るもの

将来BI化しても以下は壊さない。

- metric semantics
- quality
- coverage
- lineage
- evidence type
- missing / unavailable distinction
- reproducibility
- deterministic core behavior where applicable

BI化のためにデータ意味論を犠牲にしない。

---

# 10. MVPで明確にやらないもの

MVP M4で以下を完成させない。

- full BI SaaS
- drag & drop dashboard builder
- arbitrary SQL editor
- arbitrary user-defined calculated columns
- saved workspace system
- multi-user collaboration
- authentication / authorization for BI product
- cloud-hosted dashboard service
- realtime streaming dashboard
- arbitrary external datasource marketplace
- dynamic third-party plugin ecosystem
- full semantic layer management UI
- vector DB dependency
- LLM-driven chart logic dependency

MVPでは「将来これらを足せる骨格」まで。

---

# 11. React / Frontend evolution rule

## 11.1 MVP

Static HTML優先。

React mandatoryではない。

## 11.2 Reactへ進む条件

以下のような状況になったらReactが合理的。

- meaningful interactive filtering
- dashboard state management
- multiple coordinated charts
- drill-down / drill-through
- complex client-side controls
- reusable UI component system
- BI workspace化

この時、vanilla JavaScriptを大量に積み上げるくらいならReactへ進む。

## 11.3 Reactへ進んでもCore Contractは共通

```text
Generic Visualization Input Contract
        ↓
Visualization Spec
        ↓
Static HTML Renderer / React Renderer
```

UI frameworkにデータ契約を従属させない。

---

# 12. MVP Visualization Artifact / Deliveryの考え方

以前のユーザー指示も維持する。

Visualization完成後にcleanなdownloadable ZIPを作る場合、最低限以下を含める。

1. Analysisが実際に使用した入力data
2. そのdataを理解するためのschema / metric registry / period / coverage metadata
3. Visualization結果のStatic HTML
4. React版は実際にMVPで使用・必要になった場合のみ

clean archiveから以下を除外する。

- `__MACOSX/`
- `.DS_Store`
- `._*`
- `.venv`
- cache
- temporary file
- unnecessary backup
- old output
- credential
- token
- secret

packing後にarchive contentを検証する。

今回M1 Artifact ZIPで外包装に`__MACOSX/` / `._*`が付いた事例もあり、最終deliverable packagingでは明示的に除去する。

---

# 13. M4実装時の推奨順序

この順序は骨格案。current repo監査後に具体化する。

1. current GA4 Analysis Artifactを実物監査
2. current GSC Analysis Artifactを実物監査
3. M3 Integration Analysis Artifactを実物監査
4. 3者の共通要素と差分を抽出
5. Generic Visualization Input Contractを最小固定
6. Adapter protocol / interfaceを固定
7. GA4 adapter
8. GSC adapter
9. Integration adapter
10. Visualization Spec contract
11. deterministic spec generation
12. Generic Static HTML Renderer
13. KPI / time series / comparison / ranking / table
14. quality / lineage / coverage panels
15. portable artifact output
16. unit tests
17. contract tests
18. golden fixture
19. GA4-only live visualization
20. GSC-only live visualization
21. Integration live visualization
22. same renderer core verification
23. determinism / portability / secret / path validation
24. clean delivery artifact validation

重要:

M4実装前にcurrent repoとactual Analysis Artifactを読む。

この文書だけを見て想像でschemaを決めない。

---

# 14. M4 Acceptance Criteria骨格

M4完了時には最低限以下を満たす方向。

## 14.1 Architecture

- Visualization CoreがGA4/GSC/Integrationへ直接依存しない
- Source-specific差分はAdapter境界に限定
- Integrationを通さずGA4 Analysis単体を可視化できる
- Integrationを通さずGSC Analysis単体を可視化できる
- Integration Analysisも同じCoreへ載せられる

## 14.2 Contract

- Generic Visualization Input Contractがversioned
- Visualization Specがversioned
- unsupported stateが明示される
- metric semanticが失われない
- quality / coverage / lineageが失われない

## 14.3 Rendering

- Static HTML output
- KPI
- time series
- comparison
- ranking
- table
- quality / lineage visibility
- portable output

## 14.4 Safety / Correctness

- no re-analysis
- no silent zero filling
- no semantic rewrite
- no machine-local absolute path
- no secret
- deterministic where same input/spec

## 14.5 Extensibility

- new adapter追加で新Result Artifactを受けられる
- Renderer CoreのSource-specific conditional追加を要求しない
- future React / BI frontendへCore Contractを持ち越せる

---

# 15. M1 / M2 / M3への影響

## 15.1 M1 Integration Extraction

今回のVisualization方針変更はM1のdata contractを変更しない。

M1は引き続き、GA4 Processing + GSC Processingのlossless Integration Extractionを担当する。

## 15.2 M2 Integration Processing

VisualizationのためにM2へpresentation-specific加工を入れない。

M2はuse case / period / grain / metrics / feature data加工を担当する。

## 15.3 M3 Integration Analysis

M3はGeneric Analysis EngineとしてResult Artifactを出す。

M3 outputはVisualizationへ渡せるよう、可能な範囲でquality / coverage / lineage / metric semanticを保持する。

ただしVisualization都合でM3へSource-specific presentation logicを混入させない。

## 15.4 Source Analysis

GA4 Analysis / GSC Analysisはparallel diagnostic branchのまま維持する。

今回のVisualization独立化により、Source AnalysisをIntegrationとは独立に残している意味が直接活きる。

---

# 16. Post-MVP各Resultとの関係

## 16.1 Website Analysis

Website Analysis ResultをadapterでGeneric Contractへ変換可能にする。

## 16.2 Modeling

Prediction / Forecast / Anomaly等をSTATISTICAL / PREDICTEDとして意味を保持したまま表示する。

predictionをcausal effectとして表示しない。

## 16.3 Causal

Causal Estimateは以下をPresentationへ持ち越せるようにする。

- effect
- interval
- method
- evidence
- assumptions
- sample
- quality

## 16.4 Decision Support

Decision candidateは以下をGeneric Presentationへ載せられるようにする。

- action
- target
- objective
- expected impact
- uncertainty
- effort
- cost
- risk
- evidence
- dependencies
- reason codes
- supporting artifacts

Visualizationは最終意思決定を代行しない。

## 16.5 LLM Generated Result

LLM出力はLLM_GENERATEDとして表示し、Observed / Derived / Causal等と混同しない。

LLMがVisualization Coreの必須dependencyにならない。

---

# 17. Storage / Serialization非依存

MVPではCSV / JSON / HTMLが中心。

将来以下へ広げてもVisualization Coreの意味契約を保つ。

- Parquet
- SQLite
- DuckDB
- other artifact stores

Visualization Input Contractは可能な限りstorage formatと分離する。

---

# 18. 実装時に避けるべきアンチパターン

以下は禁止方向。

## 18.1 Source-specific Renderer

```text
GA4DashboardRenderer
GSCDashboardRenderer
IntegrationDashboardRenderer
```

だけで完結させ、共通Coreを持たない設計。

## 18.2 巨大Source switch

```python
if source == ...
```

をRenderer Coreへ積み上げる。

## 18.3 Visualizationで再計算

Analysis resultではなくraw metricをVisualizationで再集計して意味を再定義する。

## 18.4 Missing = 0

NOT_OBSERVED / UNAVAILABLE / UNKNOWN / NULLを勝手に0表示する。

## 18.5 UI framework lock-in

React component structureを先に正本化し、data / spec contractをReactへ従属させる。

## 18.6 MVPでBI全部盛り

将来BI化できるからという理由でM4にdashboard builder等を持ち込む。

## 18.7 Future hookなし

最初の3Artifactだけをハードコードし、4つ目が来た瞬間Core改造が必要な設計。

---

# 19. 2026-10-01時点の固定決定まとめ

以下を今後のM4前提として固定する。

```text
1. VisualizationはIntegration層の下位層ではない。

2. Visualizationは独立Generic Presentation Engine。

3. MVPの最初の対象入力は:
   - GA4 Analysis
   - GSC Analysis
   - Integration Analysis

4. GA4 / GSC単体のAnalysis結果はIntegrationを経由せず可視化できる。

5. Visualization CoreはSource名を知らない方向で作る。

6. Source差分はAdapter / Hook境界で吸収する。

7. Generic Visualization Input ContractとVisualization Specを分ける。

8. RendererはAnalysisを再実行・再定義しない。

9. metric semantic / quality / coverage / lineage / evidence typeを失わない。

10. MVP Canonical outputはStatic HTML。

11. ReactはMVP必須ではない。

12. vanilla JSが成長するほどdynamic behaviorが必要ならReactを優先する。

13. Reactへ進んでもCore ContractはUI framework非依存。

14. GA4/GSC/Integration以外のResult Artifactを将来adapter追加で受けられる骨格をMVPから残す。

15. MVPで完全plugin ecosystemは作らない。extension boundaryだけ作る。

16. 将来的にはVisualization層単体をBI Toolへ発展可能にする。

17. ただしMVPではBI Toolそのものを作らない。

18. M4開始時にcurrent GA4/GSC/M3 Analysis Artifactを実物監査してから最終contractを固定する。

19. 今回の方針変更はM1 Integration Extractionの責務を変更しない。

20. Visualization完成時のclean delivery ZIPはAnalysis実入力data + metadata + Static HTMLを基本とし、Reactは実際に採用した場合だけ含める。
```

---

# 20. 一行定義

今後、Visualization層を一行で説明する場合は以下を正とする。

> **Visualization is an independent, source-agnostic Generic Presentation Engine that renders versioned Analysis / Result Artifacts through adapters, a generic visualization input contract, and a presentation spec, while preserving semantics, quality, coverage, lineage, and evidence; the MVP ships a static HTML implementation and extension hooks, while keeping a clean path toward a future standalone BI tool.**

日本語:

> **Visualizationは、SourceやIntegrationに依存せず、versionedなAnalysis / Result ArtifactをAdapter・Generic Visualization Input Contract・Visualization Specを介して描画し、metric semantics・quality・coverage・lineage・evidenceを保持する独立Generic Presentation Engineである。MVPではStatic HTMLと拡張境界までを実装し、将来的な独立BIツール化へつながる骨格を残す。**

---

# 21. 実装開始時の注意

この文書は骨格設計を固定するもの。

M4実装時には必ずcurrent repoとactual Artifactを再読する。

特に以下を推測で決めない。

- GA4 Analysis current schema
- GSC Analysis current schema
- M3 Integration Analysis final schema
- adapterの具体file layout
- Visualization Input Contractの最終JSON schema
- Visualization Specの最終schema
- HTML component structure
- React採否

最新実物を確認してから実装する。

以上。
````
