# 汎用Webインテリジェンス意思決定プラットフォーム 全体要件定義書

```yaml
document_id: system_requirements
document_type: requirements
doc_ver: v3.1
language: ja
project_name: Generic Web Intelligence & Decision Platform
project_initiated_at: 2026-09-23 17:35:34 JST
created_at: 2026-09-30 JST
updated_at: 2026-10-05 22:57 JST
author: Nazuna Research
status: active
scope: MVP core + Post-MVP Decision Platform
note: v3.0までのMVP/Post-MVP要件を保持し、R12/M5 acceptanceとMVP COMPLETE状態を現行化
previous_note: v2.0をlossless保存し、M1-M4実装・Pipeline・M5要件を現行化
preserved_source_sha256: 8a0eaaaab1935f24aa3732cf1711cbe8bfbba3aa660be5b8f82b0ffc7bd6c4e4
supersedes: GENERIC_WEB_INTELLIGENCE_DECISION_PLATFORM_全体要件定義書_v2.0_20260930.md
previous_public_version: system_requirements_ja_v3.0.md
mvp_completion_state: COMPLETE
current_package: 0.12.4
public_privacy_review: PASS — public copy removes personal author identity and replaces known site-specific example identifiers with non-identifying reserved examples
```

## 0.1 MVP COMPLETE Acceptance Status — 2026-10-05

本書のMVP requirementsはCompletion Gateを通過した。Post-MVP requirementsは削除・縮約せず、そのまま次Phaseの要件として維持する。

### Current acceptance facts

- package: `0.12.4`
- R12 `full` live GA4/GSC E2E: **PASS**
- Pipeline 8-stage full profile: **PASS**
- M5 Hardening / Completion Gate: **PASS**
- MVP COMPLETE: **YES**
- `.gitignore` full audit: **PASS**
- Source public release: **NOT AUTHORIZED / HOLD**

### Requirements closure evidence

以下のMVP load-bearing requirementsは、R12/M5で実Artifactを用いて確認された。

1. Live GA4/GSC ExtractionからVisualizationまで一貫して完走できる。
2. Pipeline manifest / child Artifact / hash / direct-parent lineageを追跡できる。
3. `ZERO / NOT_OBSERVED / UNAVAILABLE / UNKNOWN / NULL`を混同しない。
4. GSC reporting lagを0へ変換しない。
5. GSC queryをGA4 session / conversion / key eventへexact attributionしない。
6. query粒度のGA4 contextはnon-additiveとして扱い、query横断SUMを許さない。
7. CTR / average position等のnon-additive semanticsを保持する。
8. A+C / B+C fixtureでlong-historyとsame-period controlを分離できる。
9. duplicate canonical grain / Cartesian row explosionを許さない。
10. deterministic layerは同一input/config/specで再現性を監査できる。
11. Persistent manifestにmachine-specific private absolute path / secretを残さない。
12. Visualizationはupstream resultを再分析しない。
13. ratio percentage表示をcorrectness contractどおり100倍して表示する。
14. original input ArtifactをVisualization Artifactから監査可能にする。
15. successful Artifactを後から上書きせず、修正後は新runを生成する。

### MVP completion中に明確化されたruntime requirements

- GA4 Extraction CLIの`partial_success = exit 1` contractはSource CLIの仕様として維持する。
- Pipelineはchild exit codeだけでArtifact usable性を決めず、GA4 Extractionについて新規Artifact manifest statusを確認する。受理可能な非0は`partial_success`だけ。
- GSCのrelative date expressionは`America/Los_Angeles` calendarでresolveする。
- Config移動時もcredential referenceはrepository-root `credentials/`へ正しく解決されること。

### Post-MVP boundary remains unchanged

MVP COMPLETEはPost-MVP要件の削除を意味しない。Website Intelligence、Change/Intervention、Modeling、Experiment、Causal、Decision Support、Semantic/Embedding、Grounded LLM、Additional Sources、Scale/Storage/Ops、Productization等は引き続き本書の後半要件として保持する。

## 0. 文書の位置づけ

本書は、現在のMVP Coreと、その後のWebsite Intelligence / Intervention / Modeling / Experiment / Causal / Decision Support / Semantic / LLM / Scale拡張を一続きのArchitectureとして定義する全体要件定義書である。

元v2.0の要件は削除せず保存付録へ全文保持し、本編冒頭では現在の実装事実と追加要件を優先的に明示する。

## 1. Source of Truth

矛盾時の優先順位は以下。

1. ユーザーの最新明示指示
2. 現在repoの実装・Artifact・testで確認できる事実
3. 本書の現行本文
4. Project Handoff
5. 保存された旧版設計・Roadmap・要件

旧版は削除しないが、現行本文と衝突する箇所は「履歴」として読む。

## 2. 現在のMVP Architecture

```text
GA4 Extraction -> GA4 Processing -> GA4 Analysis -> Visualization
                       \
                        \nGSC Extraction -> GSC Processing -> GSC Analysis -> Visualization
                       \
                        \-> Integration Extraction
                             -> Integration Processing
                             -> Integration Analysis
                             -> Visualization
```

Pipeline `full` のみ、Source Analysisを経由せず、Processingから直接Integrationへ進む。これはSource Analysisが不要という意味ではなく、統合E2Eの不要な再計算・分岐を避けるためのprofile contractである。

### Pipelineの現在状態

- package: `0.12.4`
- state: **DONE / CLOSED**
- detail: Pipeline closure、R12 live E2E、M5 hardening、MVP Completion Gateまで完了。
- live GA4/GSC Extraction E2E: `PASS / R12 CLOSED`

Pipelineは6 profileを持つ。

| Profile | 開始点 | 最終成果 | Live fetch |
|---|---|---|---|
| `full` | GA4/GSC Extraction | Visualization | あり |
| `full_from_extraction` | 既存GA4/GSC Extraction Artifact | Visualization | なし |
| `sources` | GA4/GSC Extraction | GA4 Analysis + GSC Analysis | あり |
| `ga4` | GA4 Extraction | Visualization | あり |
| `gsc` | GSC Extraction | Visualization | あり |
| `integration` | 既存GA4/GSC Processing Artifact | Visualization | なし |

`full` はSource Analysisを経由しない。

```text
GA4 Extraction ─> GA4 Processing ─┐
                                  ├─> Integration Extraction
GSC Extraction ─> GSC Processing ─┘
        -> Integration Processing
        -> Integration Analysis
        -> Visualization
```

`full` はcheckpoint/resumeを前提とし、認証等でExtractionが停止しても、成功済みArtifactを検証して再利用し、失敗Stage以降だけを続行できる設計である。手動復旧したExtraction Artifactは `supplied` として差し込める。


## 3. 不変のArchitecture Principles

優先順位を次のとおり固定する。

1. Data correctness
2. Lossless raw / integration preservation
3. Source semantics preservation
4. Separation of concerns
5. Reproducibility
6. Auditability / lineage
7. Config / Spec driven
8. Genericity
9. Extensibility
10. Determinism
11. Local-first / portability
12. Security / privacy
13. Human-in-the-loop decision support

## 4. Missing / Availability Semantics

以下を別概念として保持する。

- `ZERO`: 観測され、値が0
- `NOT_OBSERVED`: その期間・grainで観測対象外または未観測
- `UNAVAILABLE`: source/metric契約上取得不能
- `UNKNOWN`: 状態を判定できない
- `NULL`: fieldとして値が存在しない

これらを便宜上0へ潰すことを禁止する。

## 5. Cross-source Attribution Boundary

GSC queryはGA4 session / conversion / key eventへのexact attribution keyではない。

許容されるのは、同一page/date等の安全なgrainにおけるcontextとしての併置である。Query grainへGA4 factsを複製し、加算可能なfactのように扱ってはならない。

`search_type`も同様に、GA4 metricを複製するjoin axisとして利用してはならない。

## 6. Integration Layer Contract

### 6.1 Integration Extraction

- Source Processing Artifactを入力とする
- 用途都合でtrimしない
- FULL OUTER相当でsource-only rowを保持
- canonical page identityを形成
- observation state / join state / qualityを保持
- Cartesian explosionとdouble countingを防ぐ

### 6.2 Integration Processing

- Configでperiod / required_sources / grain / dimensions / metrics / filters / coverageを選択
- default period alignmentは複数required source時 `common_observed`
- additive / ratio / weighted / non-additive semanticsに従って安全に加工
- 「今回使うDataset」を作る責務を持つ

### 6.3 Integration Analysis

- Processing Contractを満たすDatasetをsource/use case固有branchなしで分析
- summary / period / comparison / trend / ranking / contribution / coverage / qualityを生成
- baseline zero / unavailable / insufficient coverageを区別
- downstreamが再計算しなくてよいResult Artifactを生成

## 7. Visualization Contract

VisualizationはIntegrationの下位層ではなく、任意のAnalysis / Result Artifactを表示する独立Presentation Layerである。

MVPで少なくとも以下を受ける。

- GA4 Analysis
- GSC Analysis
- Integration Analysis
- Generic Contractを満たす未知/外部Result Artifact

Rendererはupstream resultを再集計・再分析しない。

### 7.1 MVP実装済み境界

- React UIをprimary
- Static HTMLをsecondary/audit
- 同一Visualization Input Contract / Specを共有
- React 19.3.0 / ReactDOM 19.3.0 / Vite 8.3.1 exact pin
- chart library非依存の実装
- rootは `dashboard.html` + `dashboard_data/` の2 entry
- original input Artifact ZIPをDownload可能
- Quality / Coverage / Lineageをmain UI下部に表示

### 7.2 M5 correctness

relative delta等のratio値をpercent表示するとき、format contractに従い100倍表示する。M5で実Artifactに対して検証する。

### 7.3 Post-MVP polish

- KPI grid 5+2 -> 4+3候補
- raw JSON accordion化
- count axis integer tick
- technical metadata secondary化

## 8. Pipeline / Recovery Requirements

PipelineはAtomic CLIを置換しない。既存Layer CLIをorchestrateする。

`full` はcheckpoint/resume可能であること。Stage失敗時にはPipeline Manifestへfailed stageとattemptを保存し、resume時は成功済みArtifactの存在/hashを検証して再利用する。

手動Extraction recoveryを受け入れる場合は、Pipeline自身が生成したArtifactと区別して`supplied`としてlineageへ残す。再利用は`reused`として区別する。

`full_from_extraction` は認証を手動管理したい場合の非live-fetch保険profileとして保持する。

## 9. MVP Hardening / Release Requirements

M5では少なくとも次を行う。

- Full regression / Ruff
- Live GA4+GSC E2E
- A+C primary / B+C control
- row count / manifest hash / config hash / spec hash / direct-parent lineage
- absolute path / secret / duplicated grain / Cartesian growth scan
- determinism
- Visualization percentage format correctness
- `.gitignore`全面再監査
- release ZIPの不要物除外
- release version決定

## 10. Post-MVP End-state Requirements

以下の大領域は元v2.0から継続し、削除しない。詳細Acceptance Criteriaは保存付録を含め本書全体で維持する。

- Website Intelligence
- Change / Intervention
- Feature / Modeling / Forecast / Anomaly
- Experiment Platform
- Causal Effect
- Decision Support
- Semantic / Embedding
- Grounded LLM sidecar
- Additional Source Adapters
- Storage / Scale / Ops
- Advanced Orchestration / Productization

PredictionとCausal Effectを混同しない。LLMをdeterministic metric計算器にしない。最終Action決定は人間または明示Workflowの責務とする。


## 保存付録 — 全体要件定義書 v2.0 原本全文

この付録は、更新前の原本文書に存在した情報を不可逆に失わないための保存領域である。
**現行仕様の判断には本文前半を使用し、この付録は履歴・判断根拠・旧仕様の追跡に使用する。**
原文は内容を要約せず、そのまま保存している。

- preserved_source_sha256: `8a0eaaaab1935f24aa3732cf1711cbe8bfbba3aa660be5b8f82b0ffc7bd6c4e4`

````markdown
# 全体要件定義書 v2.0

```yaml
project_initiated_at: 2026-09-23 17:35:34 JST
created_at: 2026-09-30 JST
updated_at: 2026-09-30 JST
author: Nazuna Research
doc_ver: v2.0
note: Integration / Analysis / Visualization MVP + Post-MVP Decision Platform
業務上の名称: 「定量化されたデータにより、意思決定を補助するシステム」
```

---

# 0. 文書の位置づけ

本書は、本システムの **現行MVP完成までの要件** と、
MVP完成後に段階的に追加する **Website Intelligence / Modeling / Experiment / Causal / Decision Support / LLM / Semantic機能群** を、
一つの連続したArchitectureとして定義する全体要件定義書である。

本書は以下を目的とする。

```text
1. 現在のGA4 / GSC Source Pipelineの完成状態を前提化する
2. Integration Extraction / Processing / Analysis / Visualizationの責務境界を固定する
3. MVPのDefinition of Doneを固定する
4. MVP後に追加する機能群を、曖昧な「将来拡張」ではなく具体的な要件として定義する
5. 追加機能によってMVP Coreを作り直さないArchitectureを保証する
6. Observed / Derived / Predicted / Causal / LLM-generatedを混同しない
7. Data Lineage / Quality / Evidenceを最後まで追跡可能にする
8. 最終的に、人間の意思決定を定量・定性の両面から補助できる基盤へ発展させる
```

本書は実装詳細そのものではない。

Class名、module名、CLI名、具体Library、Chart Library、ML Library、LLM Provider等は、
本書の責務境界・契約・Acceptance Criteriaを満たす範囲で変更可能とする。

---

# 1. 本書の優先順位

本書は、以下の既存文書の内容を統合・更新した最新版である。

```text
GENERIC_WEB_INTELLIGENCE_PLATFORM_INTEGRATION_TO_VISUALIZATION_要件定義書_v1.0_20260927.md
GENERIC_WEB_INTELLIGENCE_PLATFORM_全体設計書_v1.0_20260927.md
GENERIC_WEB_INTELLIGENCE_PLATFORM_MVP_ROADMAP_v1.0_20260927.md
GA4_GSC_統合データ基盤_要件定義書_v2.0_20260926.md
```

今後、上記旧文書と本書で矛盾がある場合は、
**本書 v2.0 を優先する。**

ただし、GA4 / GSCのSource固有実装詳細について、
本書で明示的に変更していない事項は、
現在の最新repoとSource固有の既存契約を維持する。

優先順位:

```text
1. ユーザーの最新明示指示
2. 本書 v2.0
3. 現在の最新repo
4. MVP完全引き継ぎ書
5. 過去の設計書 / Roadmap / 要件定義書
```

---

# 2. システムの最終目的

本システムは、単なるGA4 / GSC集計ツールではない。

最終目的は、

> **検索・行動・コンテンツ・サイト構造・変更履歴・実験・予測・施策効果を、
> 同一の監査可能なデータ基盤へ統合し、
> 人間がより良い意思決定を行うための根拠を提供すること。**

最終形では以下を扱う。

```text
Observed Data
  GA4
  GSC
  Website Snapshot
  Site Structure
  Content
  Business KPI
  Intervention / Change
  Experiment

Derived Analysis
  集計
  比較
  Trend
  Ranking
  Diagnostic
  Opportunity

Statistical / ML
  Prediction
  Forecast
  Anomaly
  Clustering
  Ranking
  Representation

Causal
  Intervention Effect
  Experiment Effect
  Quasi-experimental Estimate

Decision Support
  Action Candidate
  Expected Impact
  Uncertainty
  Effort
  Cost
  Risk
  Evidence Level

Optional Intelligence
  Semantic / Embedding
  LLM explanation
  LLM synthesis
  Recommendation drafting
```

最終的な行動決定は人間または明示された外部Workflowが行う。

---

# 3. 現在のMVPの意味

現在構築中のMVPは、
最終Decision Platformのうち、以下の基盤を完成させるもの。

```text
GA4 Source Pipeline
GSC Source Pipeline
        ↓
Integration Extraction
        ↓
Integration Processing
        ↓
Integration Analysis
        ↓
Visualization
```

このMVPが完成すると、

```text
GA4 / GSCデータを
取得
↓
Sourceごとに正規化
↓
情報を落とさず統合
↓
用途別Datasetへ加工
↓
汎用分析
↓
Static HTML等で可視化
```

までを一貫して実行できる。

そのArtifactを人間または外部LLMへ渡すだけでも、
実用的な分析・意思決定補助が可能となる。

---

# 4. 現在の完成状態

2026-09-30時点の前提:

```text
GA4
  Extraction  DONE
  Processing  DONE
  Analysis    DONE

GSC
  Extraction  DONE
  Processing  DONE
  Analysis    DONE
```

直近確認時:

```text
package:
  v0.7.3系

tests:
  132 passed

ruff:
  All checks passed
```

今後の実装はSource Processing ArtifactをIntegration主入力とする。

Source Analysisは独立したSource diagnostic branchとして維持する。

---

# 5. 現在確定しているSource Contract

## 5.1 Date

Processing以降:

```text
YYYY-MM-DD
```

Source固有raw dateを後段で再加工しない。

---

## 5.2 Page Identity

主要page join key:

```text
site_key
normalized_page_key
```

Policy:

```text
query              removed
fragment           removed
path case          preserve
percent encoding   preserve
trailing slash     preserve
host in key        false
scheme in key      false
scope              site-scoped path
```

---

## 5.3 Missing / Availability

区別必須:

```text
ZERO
NOT_OBSERVED
UNAVAILABLE
UNKNOWN
NULL
```

禁止:

```text
NOT_OBSERVED -> 0
UNAVAILABLE -> 0
UNKNOWN -> 0
```

---

## 5.4 Persistent Path

Persistent Artifactにmachine固有absolute pathを保存しない。

禁止例:

```text
/Users/...
/home/...
C:\Users\...
```

Project-relative path / run ID / hashを使用。

---

## 5.5 Coverage

```text
requested
resolved
observed
```

を区別。

特にGSC reporting lagをFULLと偽装しない。

---

## 5.6 Metric Semantics

最低限:

```text
SUM_SAFE
DERIVED_RATIO
NON_ADDITIVE
DIRECT_ONLY
WEIGHTED_WITHIN_GRAIN
UNAVAILABLE
NON_ADDITIVE_CONTEXT
```

---

# 6. Architecture Principles

優先順位:

```text
1. Data correctness
2. Lossless raw / integration preservation
3. Source semantics preservation
4. Separation of concerns
5. Reproducibility
6. Auditability
7. Config / Spec driven
8. Genericity
9. Extensibility
10. Determinism
11. Local-first
12. Portability
13. Security / privacy
14. Human-in-the-loop decision support
```

最重要原則:

```text
Source Processing:
  Source固有の加工責任を完了して渡す

Integration Extraction:
  全部残して、正しくつなぐ

Integration Processing:
  Configで、今回使う形に加工する

Integration Analysis:
  来たDatasetを契約に従って汎用分析する

Visualization:
  来た分析結果を契約に従って汎用表示する
```

---

# 7. End-state Architecture

```text
┌──────────────────── SOURCE ZONE ─────────────────────┐

GA4 Extract -> GA4 Process -> GA4 Analysis
                         \
GSC Extract -> GSC Process -> GSC Analysis
                         \
Website Extract -> Website Process -> Website Analysis
                         \
Other Sources -> Source Process
                         \
Intervention / Experiment / Business Data

└───────────────────────────────────────────────────────┘
                         |
                         v
┌────────────── Integration Extraction ────────────────┐
│ Lossless / Canonical Key / Lineage / Quality         │
└───────────────────────────────────────────────────────┘
                         |
                         v
┌────────────── Integration Processing ────────────────┐
│ Use Case / Period / Grain / Metrics / Feature Data   │
└───────────────────────────────────────────────────────┘
                         |
          ┌──────────────┼──────────────────┐
          v              v                  v
  Integration       Modeling          Experiment /
   Analysis                            Causal Layer
          \              |                  /
           \             |                 /
            └────────────┼────────────────┘
                         v
                  Decision Support
                         |
              ┌──────────┴──────────┐
              v                     v
        Visualization        Optional LLM /
                             Semantic Layer
```

LLMはCoreを支配しない。

LLMなしでも全Core pipelineが完動する。

---

# 8. Result / Evidence Type

Artifact上の結果は、可能な限り以下の種別を持つ。

```text
OBSERVED
  Sourceから直接観測

DERIVED
  Deterministicな計算結果

HEURISTIC
  ルールベースの候補 / 診断

STATISTICAL
  統計推定

PREDICTED
  ML / Statistical Modelの予測

CAUSAL_ESTIMATE
  施策効果推定

LLM_GENERATED
  LLMによる説明 / 分類 / Narrative
```

これらを同一の「事実」として扱わない。

---

# PART I — MVP CORE

# 9. MVP Core Scope

MVPに含む:

```text
Integration Extraction
Integration Processing
Integration Analysis
Visualization
Artifact / Manifest / Quality / Lineage
Config / Spec
Live E2E
```

MVPに含めない:

```text
Website crawling
Site snapshot
ML training
Causal inference
Intervention registry
Experiment registry
Decision candidate generation
LLM platform integration
Vector DB
Cloud SaaS
Real-time stream
CMS auto-update
```

ただし後から追加できるHook / Contractを壊さない。

---

# 10. Integration Extraction — 目的

Source Processing Artifactを、
分析都合で削らず、
Source semanticsを維持したまま統合する。

特性:

```text
lossless
analysis-neutral
use-case-neutral
period-neutral
```

---

# 11. Integration Extraction — Input

MVP:

```text
GA4 Processing Artifact
GSC Processing Artifact
```

将来:

```text
Website Processing
Business KPI Processing
Ads Processing
CMS Processing
その他Source Processing
```

---

# 12. Integration Extraction — 必須責務

```text
input selection
manifest validation
input hash validation
schema compatibility
site mapping
date contract validation
page key contract validation
Source quality継承
Source coverage継承
canonical entity key生成
FULL OUTER相当統合
observation state
join state
dataset catalog
quality dataset
manifest
```

---

# 13. Integration Extraction — 禁止

```text
common periodへのtrim
YoY成立期間だけへのtrim
latest N daysへのtrim
use case選択
SEO専用filter
CVR専用filter
ranking
score
recommendation
trend判断
business interpretation
visualization都合の削減
```

---

# 14. Integration Extraction — Join

基本:

```text
FULL OUTER相当
```

最低状態:

```text
MATCHED
GA4_ONLY
GSC_ONLY
```

将来Source追加時:

```text
source_presence
source_observation_state
```

を拡張可能にする。

---

# 15. Integration Extraction — 期間差

基準fixture:

```text
A: GA4
  2024-09-27 -> 2026-09-26

C: GSC
  observed 2025-05-26 -> 2026-09-24
```

Extraction outputでは:

```text
2024-09-27 -> 2025-05-25
  GA4あり
  GSC NOT_OBSERVED

2025-05-26 -> 2026-09-24
  GA4あり
  GSCあり

2026-09-25 -> 2026-09-26
  GA4あり
  GSC NOT_OBSERVED / reporting lag
```

をそのまま保持。

---

# 16. Canonical MVP Dataset

最低候補:

```text
canonical_pages
page_daily_ga4
page_daily_gsc
query_page_daily_gsc
page_daily_integrated
query_page_daily_integrated
integration_quality
dataset_catalog
integration_manifest
```

---

# 17. page_daily_integrated安全要件

GSC `search_type`によって、
同一GA4 page-date metricが重複加算可能になる設計は禁止。

安全方式:

```text
1. page-date grainを維持
2. search_type別は別factに保持
3. 複製する場合はNON_ADDITIVE_CONTEXT
```

---

# 18. query_page_daily_integrated安全要件

GSC query-pageへGA4 page contextを付与可能。

ただし:

```text
GA4 metric:
  contextual only

query横断:
  non-additive
```

QueryにGA4 conversionを直接帰属させない。

---

# 19. Integration Processing — 目的

Integration Extraction Artifactを、
具体的な用途へ使用できる分析Datasetに加工する。

ここで初めて以下を決める。

```text
period
required source
optional source
grain
dimension
metric
filter
coverage condition
comparison window
use case
```

---

# 20. Integration Processing — Config First

用途差は可能な限り:

```text
config/
```

へ置く。

禁止:

```text
Core Pythonへ特定site
特定SEO case
特定Client
特定URL
固定threshold
```

---

# 21. Period Alignment Default

規定値:

```text
common_observed
```

A+Cなら:

```text
2025-05-26 -> 2026-09-24
```

へProcessingでtrim。

Extractionでは切らない。

---

# 22. Period Policy

最低限:

```text
common_observed
all_available
source_available
explicit_range
latest_n_days
comparison_ready
```

---

# 23. Source Requirement

Config:

```text
required_sources
optional_sources
```

例:

```text
GA4 + GSC共通分析:
  required = [ga4, gsc]

GA4長期:
  required = [ga4]
  optional = [gsc]
```

---

# 24. Safe Aggregation

CTR:

```text
sum(clicks) / sum(impressions)
```

row-level CTR平均は禁止。

Position:

```text
weight semanticsに従う
```

NON_ADDITIVE:

```text
sum禁止
```

Context metric:

```text
query横断sum禁止
```

---

# 25. Processing Output Contract

最低限:

```text
processed dataset
dataset_schema
metric_registry_snapshot
period_definition
coverage_summary
processing_manifest
```

---

# 26. Integration Analysis — 目的

Integration Processing Output Contractを満たすDatasetを、
Source名やUse Case名に依存せず分析する。

「どんなデータでも」とは、
任意CSVではなく、
**契約を満たすDatasetであればgenericに分析できる**という意味。

---

# 27. Integration Analysis — 禁止

Coreに以下を増殖させない。

```python
if source == "ga4":
    ...

if source == "gsc":
    ...

if use_case == "seo":
    ...
```

判断材料:

```text
dataset schema
grain
metric semantic
dimension role
period metadata
coverage
analysis spec
```

---

# 28. Generic Analysis Operations

MVP最低限:

```text
summary
period total
safe average
current vs previous
current vs YoY
absolute delta
relative delta
trend
ranking
share / contribution
coverage-aware status
quality-aware suppression
```

---

# 29. Analysis Status

最低限:

```text
OK
BASELINE_ZERO
BASELINE_UNAVAILABLE
CURRENT_UNAVAILABLE
INSUFFICIENT_COVERAGE
```

0とUnavailableを混同しない。

---

# 30. Analysis Spec

Processing Configと分離。

```text
Processing Config:
  何を対象にするか

Analysis Spec:
  対象へ何を計算するか
```

同じProcessing Artifactへ複数Analysis Specを適用可能とする。

---

# 31. Visualization — 目的

Integration Analysis Artifactを、
Source / Use Case固有の画面実装へ固定せず表示する。

---

# 32. Visualization Input

```text
Analysis Artifact
Analysis Manifest
Result Dataset Catalog
Visualization Spec
```

Source Processingを直接読み直さない。

---

# 33. Visualization MVP Output

基本:

```text
Static HTML
```

推奨:

```text
Self-contained
Server不要
Portable
```

---

# 34. Visualization Components

最低限:

```text
KPI
Time Series
Comparison Table
Ranking Table
Quality / Coverage
Metadata / Lineage
```

必要に応じ:

```text
Bar
Scatter
Filter
Small multiple
```

---

# 35. Visualizationで再計算しないもの

禁止:

```text
YoY再計算
CTR再集計
Position再集計
Business metric再定義
分析意味判断
score生成
```

表示上のformat / sort / filterは可。

---

# 36. MVP Artifact共通要件

全Layer:

```text
immutable run
manifest
input lineage
input hash
config/spec hash
schema version
tool version
started_at
completed_at
status
row counts
artifact hash
relative path
secret禁止
```

---

# 37. MVP Definition of Done

以下を満たす。

```text
A+C Integration Extraction:
  lossless
  FULL OUTER
  Source-only保持
  lag保持
  no Cartesian explosion
  lineage
  quality

Integration Processing:
  common_observed default
  all_available override
  safe aggregation
  config-driven
  metric registry
  dataset schema

Integration Analysis:
  source-independent
  use-case-independent
  spec-driven
  missing-aware
  coverage-aware
  multiple dataset types

Visualization:
  generic
  static HTML
  quality表示
  lineage表示
  portable

Whole:
  E2E
  deterministic
  tests green
  artifact validation green
```

---

# PART II — POST-MVP: WEBSITE INTELLIGENCE

# 38. Website Snapshot Source — 目的

GA4 / GSCが、

```text
何が起きたか
```

を示すのに対し、
Website Snapshotは、

```text
その時点でPage / Siteが何で構成されていたか
```

を提供する。

これにより、

```text
Performance
x
Content
x
Structure
x
Change
```

を統合可能にする。

---

# 39. Website Source Pipeline

推奨構造:

```text
Website Extraction
  ↓
Website Processing
  ↓
Website Analysis
  ↓
Integration Extraction
```

Source Analysisは任意だが、
単体でSite technical / structure diagnosticを行えることが望ましい。

---

# 40. Website Extraction — Page Data

最低限候補:

```text
requested_url
final_url
status_code
redirect_chain
content_type
canonical
robots meta
title
meta description
language
hreflang
Open Graph
structured data
H1-H6
body text
word count
content hash
HTML hash
fetch timestamp
```

---

# 41. Website Extraction — Structure Data

```text
internal links
external links
anchor text
rel
image src
image alt
navigation
breadcrumbs
footer links
script references
stylesheet references
```

---

# 42. Crawl Safety

Default:

```text
GET / HEAD中心
form submitしない
state-changing endpointへアクセスしない
allowed host明示
rate limit
timeout
max pages
max bytes
concurrency limit
```

認証page:

```text
explicit opt-in
```

---

# 43. Robots Policy

Default:

```yaml
robots_policy: respect
```

Owner管理下のoverrideを実装する場合:

```text
explicit owner_override
```

勝手にoverrideしない。

---

# 44. Dynamic Rendering

初期:

```text
HTTP HTML
```

将来:

```text
Headless Browser Adapter
```

Core dependencyにしない。

Browser failureでHTTP crawlを壊さない。

---

# 45. Website Processing

責務:

```text
URL / page identity mapping
canonical page key
content normalization
heading structure
link normalization
content block extraction
duplicate / near-duplicate metadata
snapshot IDs
hashes
quality
```

Source Processingで行うべきWebsite固有加工を、
Integrationへ押し付けない。

---

# 46. Site Snapshot Versioning

Run:

```text
site_extract_<timestamp>
```

Page:

```text
snapshot_id
site_key
normalized_page_key
html_sha256
content_sha256
fetched_at
```

Before / After比較可能とする。

---

# 47. Content Block Model

Pageを意味のあるBlockへ分解可能にする。

```text
block_id
page_key
block_type
heading_path
text
text_hash
order
snapshot_id
```

用途:

```text
content diff
semantic classification
embedding
LLM analysis
content gap
section-level performance context
```

---

# 48. Internal Link Graph

Edge:

```text
source_page_key
target_page_key
anchor_text
link_context
nofollow
snapshot_id
```

Derived候補:

```text
in-degree
out-degree
crawl depth
orphan candidate
hub candidate
centrality
```

DerivedはWebsite Analysisまたは後段Analysisで行う。

Raw edgeを失わない。

---

# 49. Website Quality

最低限:

```text
crawl success rate
status distribution
redirect errors
canonical mismatch
robots block
parse failure
duplicate page key
content hash duplicates
missing title
missing H1
broken internal links
```

---

# 50. Website Integration

Website ProcessingはIntegration Extractionへ新Sourceとして追加。

既存GA4/GSC Integrationを作り直さず、
Source Adapter追加で接続可能にする。

Join候補:

```text
site_key
normalized_page_key
snapshot / effective date
```

---

# 51. Snapshot Time Semantics

Website SnapshotはPoint-in-time観測。

GA4 / GSC daily factと直接同一grainではない。

必要に応じ:

```text
effective_from
effective_to
snapshot_id
```

等を用いて、
特定期間に有効だったPage Stateを表現する。

勝手なforward-fillはConfig / Processing policyとして明示する。

---

# 52. Website Analysis

候補:

```text
technical health
content completeness
internal link structure
page template pattern
duplicate content
change detection
content age
orphan/hub candidates
```

ただしBusiness performanceとの関連はIntegrated Analysis側で扱う。

---

# 53. Website Source Definition of Done

```text
safe crawl
allowed host
rate limit
snapshot version
page identity
content hash
HTML hash
structure extraction
link graph
content block
quality
lineage
Integration接続
```

---

# PART III — POST-MVP: CHANGE / INTERVENTION DATA

# 54. Change Dataの目的

Performance変化を解釈するには、

```text
何が変わったか
いつ変わったか
意図的変更か
自動検出か
```

が必要。

Changeをfirst-class dataとして扱う。

---

# 55. Detected Change

Website Snapshot差分から生成。

例:

```text
title changed
meta description changed
H1 changed
content changed
internal link changed
CTA changed
layout marker changed
redirect changed
canonical changed
structured data changed
```

---

# 56. Detected Change != Intervention

必ず区別。

```text
Detected Change:
  Systemが差分から検出

Intervention:
  人間 / Workflowが意図して実行した施策
```

自動検出だけで「施策」と断定しない。

---

# 57. Intervention Registry

最低限:

```text
intervention_id
site_key
action_type
target_page_key
target_scope
started_at
completed_at
status
before_snapshot_id
after_snapshot_id
effort
cost
owner optional
notes
source
```

---

# 58. Intervention Types

Hardcodeで閉じない。

例:

```text
title_change
meta_description_change
content_rewrite
heading_change
internal_link_change
cta_change
layout_change
technical_fix
redirect_change
schema_change
sitewide_template_change
```

---

# 59. Intervention Lineage

施策は関連Artifactへ接続。

```text
before snapshot
after snapshot
analysis finding
decision candidate
experiment
effect estimate
```

施策の結果を後から追跡可能にする。

---

# 60. Intervention Quality

最低限:

```text
start/end known
target known
before snapshot exists
after snapshot exists
other concurrent changes
data coverage
confounding warning
```

---

# 61. Change / Intervention Definition of Done

```text
detected changeとintentional interventionを分離
registry
version
lineage
snapshot link
cost/effort
quality flags
Integrationで利用可能
```

---

# PART IV — POST-MVP: EXPERIMENT PLATFORM

# 62. Experiment Registry — 目的

施策の効果を、
単純なBefore / Afterより高いEvidenceで評価するための基盤。

---

# 63. Experiment Contract

最低限:

```text
experiment_id
hypothesis
site_key
unit
treatment
control
start_date
end_date
primary_metric
secondary_metrics
assignment_method
status
intervention_ids
analysis_plan optional
```

---

# 64. Experiment Unit

例:

```text
page
query group
template
user segment
time block
site section
```

UnitはExperimentごとに明示。

---

# 65. Treatment / Control

曖昧な「変更前 / 変更後」だけでなく、

```text
treatment definition
control definition
assignment / selection rule
```

を保存。

---

# 66. Experiment Quality

最低限確認:

```text
overlap period
data availability
assignment integrity
sample size
metric availability
contamination
concurrent intervention
missing observations
```

---

# 67. Evidence Level

標準候補:

```text
E0_HEURISTIC
E1_CORRELATIONAL
E2_PREDICTIVE
E3_QUASI_EXPERIMENTAL
E4_RANDOMIZED_EXPERIMENT
```

Evidence Levelを施策効果の確度と混同せず、
**Evidenceの種類・強さの分類**として使う。

---

# 68. Experiment Definition of Done

```text
registry
treatment/control
metrics
period
assignment metadata
quality
lineage
causal layer input
```

---

# PART V — POST-MVP: MODELING / MACHINE LEARNING

# 69. Modeling Layer — 目的

以下を扱う。

```text
prediction
forecast
anomaly detection
clustering
ranking
representation
```

PredictionとCausal Effectは分離。

---

# 70. Feature Dataset

Integration Processing / Canonical Dataから、
versioned Feature Setを生成。

例:

```text
page_performance_features_v1
query_page_features_v1
content_features_v1
site_structure_features_v1
```

---

# 71. Feature Contract

各Feature:

```text
name
source
formula
lookback
missing policy
type
version
availability
leakage risk optional
```

---

# 72. Feature Lineage

Featureから、

```text
Source Artifact
Integration Artifact
Processing Config
Snapshot
```

へ遡れること。

---

# 73. Model Types

Optional:

```text
linear / regularized regression
tree / gradient boosting
time series model
anomaly detector
clustering
ranking model
embedding / representation model
```

Domain Contractへ特定Library名を露出しない。

---

# 74. ML Validation

Time-dependent dataでは:

```text
past -> train
future -> validation
```

を基本。

random splitをdefaultにしない。

必要に応じ:

```text
rolling validation
backtesting
```

を使用。

Leakage防止を優先。

---

# 75. Model Registry

DB必須にしない。

例:

```text
models/
  <model_name>/
    <model_run_id>/
      manifest.json
      metrics.json
      feature_schema.json
      model_artifact
```

---

# 76. Model Manifest

最低限:

```text
model_name
model_type
model_version
training_dataset
training_data_hash
feature_set_version
target
train_period
validation_period
parameters
random_seed
metrics
artifact_hash
created_at
```

---

# 77. Prediction Contract

```text
prediction_id
target_entity
target_metric
prediction
lower_bound optional
upper_bound optional
model_run_id
feature_snapshot_id
generated_at
status
quality_flags
```

---

# 78. Prediction != Causal Effect

例:

```text
Prediction:
  現状のままなら来月CTRはいくつか

Causal:
  title変更によりCTRがどれだけ変わるか
```

Predictionから介入効果を推論しない。

---

# 79. Modeling Quality

最低限:

```text
train/validation period
leakage checks
baseline comparison
evaluation metrics
seed
feature version
data hash
uncertainty where available
```

---

# 80. Modeling Definition of Done

```text
feature set version
feature lineage
time-aware validation
model registry
model hash
reproducibility
prediction contract
prediction / causal separation
```

---

# PART VI — POST-MVP: CAUSAL EFFECT ESTIMATION

# 81. Causal Layer — 目的

「何が起きそうか」ではなく、

```text
特定の介入が
特定のMetricへ
どの程度影響したか / しそうか
```

を評価する。

---

# 82. Causal Input

最低限候補:

```text
Intervention
Experiment
Observed performance
Site snapshot
Control definition
Time period
Quality metadata
```

---

# 83. Causal Methods

将来候補:

```text
controlled before/after
difference-in-differences
interrupted time series
synthetic control
matching
uplift modeling
randomized experiment analysis
Bayesian effect estimation
```

特定methodへ固定しない。

---

# 84. Method Selection

Methodは:

```text
data availability
experiment design
control availability
sample size
time structure
assumptions
```

に応じて選択。

Method名だけでEvidence Levelを自動決定しない。

---

# 85. Effect Estimate Contract

最低限:

```text
effect_id
action_type
target_scope
metric
estimated_effect
effect_unit
lower_bound
upper_bound
method
evidence_level
assumptions
data_period
sample_size
quality_flags
intervention_id optional
experiment_id optional
model_run_id optional
supporting_artifacts
```

---

# 86. Assumptions

Causal Estimateには、
Methodが成立するためのAssumptionを保存。

例:

```text
parallel trend
stable treatment
no major concurrent intervention
control comparability
measurement stability
```

Assumption不明を無視しない。

---

# 87. Causal Failure Policy

Evidence不足時:

```text
effect estimateを出さない
または
LOW_EVIDENCE / INSUFFICIENT_DATA
```

として返す。

数値を捏造しない。

---

# 88. Causal Definition of Done

```text
intervention / experiment linkage
method metadata
assumptions
effect interval
evidence level
quality flags
reproducibility
predictionとの分離
```

---

# PART VII — POST-MVP: DECISION SUPPORT

# 89. Decision Support — 目的

Analysis / Prediction / Effect Estimateを、
人間が比較可能なAction Candidateへ変換する。

---

# 90. Decision Candidate Contract

最低限:

```text
candidate_id
action_type
target
objective_metric
expected_impact
uncertainty
estimated_effort
estimated_cost
risk
evidence_level
dependencies
reason_codes
supporting_artifacts
generated_at
status
```

---

# 91. Candidate Source

Candidateは以下から生成可能。

```text
HEURISTIC analysis
STATISTICAL analysis
PREDICTION
CAUSAL_ESTIMATE
manual rule
LLM draft
```

ただしsource typeを必ず明示。

LLM draftだけを高Evidence Candidateとして扱わない。

---

# 92. Expected Value

Utilityは固定しない。

Config例:

```yaml
decision:
  utility:
    click_value: 1.0
    key_event_value: 10.0
    implementation_hour_cost: 5000
    uncertainty_penalty: 0.5
```

---

# 93. Expected Value Contract

最低限:

```text
benefit estimate
cost estimate
effort estimate
risk adjustment
uncertainty penalty
evidence level
utility config version
```

同一Utility Configの範囲で比較。

異なるUtility ConfigのScoreを単純比較しない。

---

# 94. Recommendation Boundary

Systemは:

```text
Candidate生成
比較材料提示
根拠提示
```

まで。

最終actionは:

```text
Human
or
Explicit External Workflow
```

が選ぶ。

自動本番反映は別System / 別承認フロー。

---

# 95. No Fabricated Impact

根拠Model / Effect Estimateがない場合、
具体的なexpected impact数値を作らない。

例:

```text
「CTR改善余地あり」
```

はHeuristicとして可。

```text
「+25 clicks」
```

は根拠なしで出さない。

---

# 96. Decision Support Quality

Candidateに:

```text
evidence level
data coverage
uncertainty
quality flags
supporting artifacts
assumptions
```

を必ず付ける。

---

# 97. Decision Support Definition of Done

```text
candidate contract
evidence-aware
cost / effort / risk
utility config
supporting lineage
no fabricated impact
human decision boundary
```

---

# PART VIII — POST-MVP: SEMANTIC / EMBEDDING

# 98. Semantic Layer — 目的

文字列一致だけでは扱えない、

```text
topic
intent
similarity
content meaning
query-page semantic fit
content gap
```

を扱う。

---

# 99. Input

```text
query text
page title
meta description
content block
body text
anchor text
structured data
```

---

# 100. Chunk Contract

```text
chunk_id
source_artifact
page_key
snapshot_id
block_id optional
text
text_hash
chunk_strategy
chunk_version
```

---

# 101. Embedding Contract

```text
embedding_id
chunk_id
provider
model
model_version
dimension
input_hash
created_at
```

Embedding vector自体の保存形式はimplementation依存。

---

# 102. Vector Index

Optional。

Core Canonical / IntegrationはVector DBへ依存しない。

Indexは再生成可能なDerived Artifactとして扱う。

---

# 103. Semantic Result

例:

```text
query intent
page topic
query-page similarity
content cluster
semantic gap
related pages
```

Result typeは:

```text
STATISTICAL
PREDICTED
or
LLM_GENERATED
```

としてSourceを明確にする。

---

# 104. Semantic Definition of Done

```text
chunk version
input hash
embedding model version
rebuild possible
Core非依存
lineage
```

---

# PART IX — POST-MVP: LLM PLATFORM

# 105. LLMの位置づけ

LLMはOptional Sidecar。

Core pipeline:

```text
Extraction
Processing
Integration
Analysis
Modeling optional
Causal optional
Decision optional
Visualization
```

はLLMなしで完動。

---

# 106. LLM Provider Contract

概念:

```text
capabilities
submit
result
```

Transport:

```text
local Python
CLI
HTTP
job queue
```

のいずれでもよい。

CoreはTransportを意識しない。

---

# 107. LLM Suitable Tasks

```text
query intent classification
page topic classification
semantic similarity補助
content gap explanation
content summary
analysis synthesis
reason-code explanation
recommendation drafting
report narrative
qualitative issue extraction
```

---

# 108. LLMへ任せない処理

```text
row count
join
aggregation
metric calculation
CTR
position aggregation
period comparison
hash
lineage
reconciliation
exact ranking
schema validation
coverage validation
```

Deterministic codeで行う。

---

# 109. LLM Grounding

LLM requestには根拠を明示。

```text
artifact IDs
dataset IDs
page snapshot IDs
row references
metric context
quality flags
evidence level
```

NarrativeからSource Artifactへ追跡可能にする。

---

# 110. Prompt Versioning

```text
prompt_id
prompt_version
prompt_sha256
```

Manifestへ保存。

---

# 111. LLM Cache

Cache key候補:

```text
provider
model
prompt_template_version
input_hash
parameters
```

---

# 112. LLM Compute Budget

Config:

```text
enabled
provider
max_jobs
max_tokens
max_pages
max_wall_time
cost budget optional
```

Compute不足でCoreを止めない。

---

# 113. LLM Failure Isolation

LLM失敗時:

```text
Core success
LLM enrichment skipped / failed
```

とする。

---

# 114. External Provider Data Policy

外部LLMへデータ送信する場合:

```text
Explicit Opt-in
```

を必須。

勝手にlocal artifactを外部送信しない。

Sensitive情報は送信前にPolicy適用。

---

# 115. LLM Output Type

LLM出力は:

```text
LLM_GENERATED
```

として明示。

Observed FactやCausal Estimateと混ぜない。

---

# 116. LLM Definition of Done

```text
optional provider
disabledでもCore完動
grounding
prompt version
input hash
cache
budget
failure isolation
external opt-in
no metric arithmetic
```

---

# PART X — FUTURE SOURCE ADAPTERS

# 117. Additional Source Framework

新Source追加時の基本:

```text
Source Extraction
  ↓
Source Processing
  ↓
Optional Source Analysis
  ↓
Integration Extraction
```

Integration CoreをSourceごとに書き換えない。

---

# 118. Additional Source候補

将来候補:

```text
Website / Sitemap
Google Ads / Other Ads
CMS
Business KPI
Revenue / CRM
Conversion backend
Search ranking provider
Intervention Log
Experiment Log
Manual annotation
```

これらは必要になった時点で個別要件定義する。

---

# 119. Business KPI Source

目的:

```text
Traffic / Search metric
↓
Business Outcome
```

への接続。

候補:

```text
revenue
lead
appointment
qualified conversion
margin
customer value
offline conversion
```

Privacy / access controlをSource契約へ含める。

---

# 120. CMS Source

用途:

```text
content metadata
publish time
author
template
category
revision
draft/published state
```

CMS write-backは別権限・別System扱い。

Default read-only。

---

# PART XI — OPERATION / SCALE / STORAGE

# 121. Pipeline DAG

各Nodeは独立Artifactを生成。

将来Orchestratorを追加しても、
各Layerの責務を吸収しない。

---

# 122. Incremental / Resume

データ量増加時:

```text
incremental extraction
incremental processing
resume
checkpoint
```

を追加可能。

ただしpartial stateをFinal Artifactとして偽装しない。

---

# 123. Partial Failure

Source失敗時:

```text
成功Artifactは残す
failed nodeを明示
downstream eligibilityを判断
```

Pipeline全体を無条件削除しない。

---

# 124. Storage Strategy

MVP:

```text
CSV
JSON
HTML
```

将来Optional:

```text
Parquet
SQLite
DuckDB
```

Domain Contractをstorage engineへ依存させない。

---

# 125. Parquet

導入目的:

```text
size
read speed
typed schema
large dataset
```

CSV Artifactを完全に廃止するかは別Decision。

---

# 126. DuckDB

用途:

```text
large local analytical query
ad-hoc joins
Parquet query
```

Core必須dependencyにするかは将来判断。

---

# 127. Artifact Catalog

将来全Artifactを横断検索可能にする。

最低概念:

```text
artifact_id
run_id
type
schema_version
created_at
parent_artifacts
hash
status
```

DBなしでも実現可能。

---

# 128. Schema Registry

Layerごとのschemaをversion管理。

Breaking change:

```text
schema version bump
migration / compatibility rule
```

Silent reinterpretation禁止。

---

# PART XII — QUALITY / PROVENANCE / SECURITY

# 129. Quality Model

Qualityは単一scoreに潰さない。

最低次元:

```text
coverage
completeness
validity
consistency
lineage
freshness
join quality
measurement limitation
model quality
causal quality
```

---

# 130. Provenance

任意の最終Candidate / Chart / Narrativeから、
可能な限り以下へ遡れる。

```text
Visualization
Analysis
Model / Effect
Processing
Integration
Source Processing
Source Extraction
Raw source / Snapshot
```

---

# 131. Security

保存禁止:

```text
OAuth client_secret
access token
refresh token
authorization code
credential absolute path
private local username path
```

---

# 132. External Data Transfer

Cloud / External LLM / External ML serviceへ送る場合:

```text
explicit config
data classification
opt-in
audit metadata
```

を要求。

---

# 133. Determinism

Deterministic Layerでは:

```text
stable sort
tie break
null ordering
float tolerance
config hash
```

を固定。

Stochastic Modelでは:

```text
seed
parameters
model version
```

をManifestへ保存。

---

# PART XIII — CONFIG / SPEC ARCHITECTURE

# 134. Config Separation

```text
Source Config
Integration Extraction Config
Integration Processing Config
Analysis Spec
Model Spec
Experiment Spec
Causal Spec
Decision Config
Visualization Spec
LLM Config
```

を意味的に分離。

---

# 135. Config Versioning

全Config / Spec:

```text
id
version
sha256
```

をManifestで追跡可能にする。

---

# 136. Site固有設定

Coreへ埋め込まず:

```text
config/
```

へ置く。

例:

```text
site mapping
URL rules
business metric value
crawl allowed host
decision utility
external provider policy
```

---

# PART XIV — TESTING

# 137. Test Layers

```text
unit
contract
fixture integration
artifact validation
live artifact
regression
E2E
```

---

# 138. MVP Golden Fixture

最低:

```text
MATCHED
GA4_ONLY
GSC_ONLY
ZERO
UNAVAILABLE
NOT_OBSERVED
missing date
query-bearing URL
absolute URL
multiple search_type
query-page
baseline zero
baseline unavailable
partial coverage
```

---

# 139. Website Fixture

最低:

```text
200 page
redirect
404
canonical
robots noindex
duplicate content
internal link
external link
missing H1
changed title
changed content
```

---

# 140. Modeling Fixture

最低:

```text
time split
feature missing
baseline model
deterministic seed
prediction interval optional
leakage detection case
```

---

# 141. Causal Fixture

最低:

```text
intervention
control
pre/post
insufficient data
concurrent change
effect interval
assumption flag
```

---

# 142. LLM Test

Unit testで外部LLMを直接呼ばない。

```text
mock provider
grounding payload
cache
failure
timeout
disabled
```

を検証。

---

# PART XV — ROADMAP / DEPENDENCY

# 143. Phase 0 — Current

```text
GA4 Source DONE
GSC Source DONE
```

---

# 144. Phase 1 — MVP Core

```text
Integration Extraction
Integration Processing
Integration Analysis
Visualization
MVP Hardening
```

完了条件:

```text
v1.0相当
```

Version番号自体はrepo状態に合わせて決定。

---

# 145. Phase 2 — Website Intelligence

```text
Website Extraction
Website Processing
Website Analysis
Snapshot
Content Block
Internal Link Graph
Integration接続
Site-aware Analysis
```

---

# 146. Phase 3 — Change / Intervention

```text
Snapshot diff
Detected Change
Intervention Registry
Before / After linkage
```

---

# 147. Phase 4 — Modeling

```text
Feature Dataset
Model Registry
Prediction
Forecast
Anomaly
```

Interventionより前に一部Modelingを実装することも可能。

DependencyはFeature Data。

---

# 148. Phase 5 — Experiment / Causal

```text
Experiment Registry
Effect Estimation
Evidence Level
```

Intervention / Experiment dataが前提。

---

# 149. Phase 6 — Decision Support

```text
Action Candidate
Expected Impact
Uncertainty
Effort
Cost
Risk
Expected Value
Evidence
```

Analysis / Prediction / Causalを統合。

---

# 150. Phase 7 — Semantic / LLM

```text
Chunk
Embedding
Semantic Analysis
LLM Provider
Narrative
Explanation
Draft Recommendation
```

Hookは先に設計可能。
本実装はCore安定後。

---

# 151. Phase 8 — Scale / Operational Enhancement

```text
Parquet
DuckDB
Incremental
Resume
Artifact Catalog
Unified Orchestrator
```

必要性に応じて導入。

---

# 152. Dependency Rule

必須依存:

```text
Decision Support
  <- Analysis / Model / Effect artifacts

Causal
  <- Intervention / Experiment + Observed Data

Experiment Analysis
  <- Experiment Registry

Site-aware analysis
  <- Website Snapshot

Semantic / LLM
  <- Grounded Artifact Contracts
```

不要依存:

```text
Core Analysis -> LLM
Core Integration -> ML
Visualization -> LLM
GA4/GSC Source -> Website Crawler
```

---

# PART XVI — ACCEPTANCE CRITERIA

# 153. MVP Acceptance

```text
Integration Extraction lossless
Integration Processing config-driven
common_observed default
Integration Analysis generic
Visualization generic
Quality / Lineage
Live A+C
E2E
```

---

# 154. Website Acceptance

```text
safe crawl
snapshot version
page identity
content / structure
link graph
content hash
quality
integration
```

---

# 155. Intervention Acceptance

```text
intentional vs detected separation
registry
before/after
cost/effort
lineage
```

---

# 156. Modeling Acceptance

```text
feature version
training lineage
time validation
seed
model hash
prediction contract
uncertainty
causal separation
```

---

# 157. Experiment / Causal Acceptance

```text
registry
treatment/control
method
assumptions
effect interval
evidence level
quality
```

---

# 158. Decision Support Acceptance

```text
candidate
expected impact source
uncertainty
effort/cost/risk
evidence
supporting artifacts
human boundary
no fabricated impact
```

---

# 159. LLM Acceptance

```text
disabledでCore完動
provider abstraction
grounding
prompt version
input hash
cache
budget
failure isolation
external opt-in
no deterministic metric calculation
```

---

# PART XVII — ARCHITECTURE DECISIONS

# 160. ADR-001 Raw Preservation

Raw / Source Artifactを後段都合で上書きしない。

---

# 161. ADR-002 Source Processing Responsibility

Source固有の加工はSource Processingで完結させる。

---

# 162. ADR-003 Integration Extraction is Lossless

Integration Extractionで用途都合の行削減をしない。

---

# 163. ADR-004 Integration Processing Owns Use Case

期間・用途・grain・metric選択はIntegration Processing。

---

# 164. ADR-005 common_observed Default

複数Required Sourceを使う場合、
Integration Processingの期間alignment defaultはcommon_observed。

---

# 165. ADR-006 Analysis is Generic

Source / Use Caseごとの専用Analysis Coreを増殖させない。

---

# 166. ADR-007 Visualization Does Not Re-analyze

VisualizationはPresentation Layer。

---

# 167. ADR-008 Prediction is not Causal

PredictionをIntervention Effectとして扱わない。

---

# 168. ADR-009 Intervention is First-class Data

施策履歴をNotesではなく構造化Artifactとして扱う。

---

# 169. ADR-010 Website is First-class Source

Website自体をSource Pipelineとして扱う。

---

# 170. ADR-011 LLM is Optional

LLMなしでCore完動。

---

# 171. ADR-012 LLM is Grounded

LLM narrativeから根拠Artifactへ追跡可能にする。

---

# 172. ADR-013 Decision Support is Human-in-the-loop

SystemはCandidateを提示。
最終actionは人間または明示Workflow。

---

# 173. ADR-014 Evidence is Explicit

Heuristic / Prediction / Causal / LLMを同一Evidenceとして扱わない。

---

# 174. ADR-015 No Premature Storage Lock-in

CSV / JSON / HTMLから開始し、
必要に応じParquet / DuckDBへ拡張。

---

# PART XVIII — END-STATE USER EXPERIENCE

# 175. MVP User Experience

ユーザーは:

```text
GA4 / GSCを取得
↓
処理
↓
統合
↓
用途Config選択
↓
分析Spec選択
↓
HTMLを見る
```

だけで利用可能。

---

# 176. Post-MVP User Experience

最終的には:

```text
1. GA4 / GSC / Site / Business Dataを取得

2. Source Processing

3. Lossless Integration

4. Use Case Dataset Assembly

5. Generic Analysis

6. 必要ならPrediction / Forecast

7. Intervention / ExperimentがあればEffect Estimation

8. Action Candidate生成
   - Expected Impact
   - Uncertainty
   - Evidence
   - Effort
   - Cost
   - Risk

9. Optional Semantic / LLMで
   - 内容理解
   - 説明
   - Narrative
   - 提案文案

10. Visualizationで根拠を確認

11. 人間が意思決定
```

---

# 177. End-state Decision Artifact Example

```json
{
  "candidate_id": "candidate-001",
  "target": "/example/",
  "action_type": "title_change",
  "expected_impact": {
    "metric": "gsc_clicks_28d",
    "estimate": 25,
    "lower": 5,
    "upper": 42
  },
  "result_type": "CAUSAL_ESTIMATE",
  "evidence_level": "E3_QUASI_EXPERIMENTAL",
  "estimated_effort_hours": 0.5,
  "estimated_cost": null,
  "risk": "LOW",
  "quality_flags": [],
  "supporting_artifacts": [
    "analysis.page_ctr_opportunity",
    "effect.title_change_v2"
  ]
}
```

根拠がない場合は数値を出さない。

---

# 178. End-state LLM Artifact Example

```json
{
  "task": "page_content_diagnosis",
  "result_type": "LLM_GENERATED",
  "provider": "local_platform",
  "prompt_version": "v3",
  "input_hash": "...",
  "source_artifacts": [
    "page_snapshot",
    "analysis.query_page_performance"
  ],
  "status": "success"
}
```

---

# 179. Strong Architectureの完成条件

以下を満たす。

```text
Source追加でAnalysis Coreを書き直さない
Analysis追加でExtractionを書き直さない
Visualization変更でAnalysis計算を書き直さない
Website追加でGA4/GSC Sourceを壊さない
ML停止でもAnalysisが動く
LLM停止でもCoreが動く
Crawler失敗でもGA4/GSC Artifactは残る
過去RunからLineageを追える
PredictionとCausalを区別できる
RecommendationにEvidenceがある
数値の出所を辿れる
特定site名がCoreにない
```

---

# 180. Final Architecture Statement

本システムの最終形は、

> **検索・行動・コンテンツ・サイト構造・変更履歴・実験・予測・施策効果を、
> 同一の監査可能なデータ基盤上で扱い、
> 記述分析から予測、因果効果推定、意思決定支援、可視化、LLM補助まで段階的に拡張できる
> Generic Web Intelligence & Decision Platform**

である。

MVPはその最初の完成形として、

```text
GA4 / GSC
↓
Integration Extraction
↓
Integration Processing
↓
Integration Analysis
↓
Visualization
```

を完成させる。

MVP後はCoreを作り直すのではなく、
同じArtifact / Contract / Lineage思想の上へ、

```text
Website Intelligence
Change / Intervention
Experiment
Modeling
Causal Effect
Decision Support
Semantic / Embedding
Optional LLM
Scale / Storage
```

を順次追加する。

以上。
````
