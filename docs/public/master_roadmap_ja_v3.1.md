# 汎用Webインテリジェンス意思決定プラットフォーム 開発ロードマップ

```yaml
document_id: master_roadmap
document_type: roadmap
doc_ver: v3.1
language: ja
project_name: Generic Web Intelligence & Decision Platform
project_initiated_at: 2026-09-23 17:35:34 JST
created_at: 2026-09-30 JST
updated_at: 2026-10-05 22:57 JST
author: Nazuna Research
status: active
scope: MVP + Post-MVP end-state roadmap
note: v3.0までの全情報とPost-MVP roadmapを保持し、R12/M5/MVP COMPLETE/.gitignore closureを現行化
previous_note: v2.0の全情報を保存し、M1-M4完了・構造再編・Pipeline・M5現在地を追加
preserved_source_sha256: 1ac63d243b6e66a107cd3bc6b5e4221e21d8dd6e2509c02ac64b0bc7ae3742fd
supersedes: GENERIC_WEB_INTELLIGENCE_DECISION_PLATFORM_MASTER_ROADMAP_v2.0_20260930.md
previous_public_version: master_roadmap_ja_v3.0.md
mvp_completion_state: COMPLETE
current_package: 0.12.4
public_privacy_review: PASS — public copy removes personal author identity and replaces known site-specific example identifiers with non-identifying reserved examples
```

## 0.1 MVP COMPLETE Update — 2026-10-05

MVPのクリティカルパスは完了した。現在のpackageは`0.12.4`。

| 領域 | 現在状態 | Completion evidence |
|---|---|---|
| Source GA4/GSC | DONE / HARDENED | unit/contract/regression + live R12 |
| Integration M1-M3 | DONE / CLOSED | lossless Extraction / config-driven Processing / Generic Analysis |
| Visualization M4 | DONE / CLOSED | React primary + Static audit + M5 percentage correctness |
| Pipeline | DONE / CLOSED | 6 profiles / checkpoint / resume / supplied / reused / portable manifest / full Live E2E |
| R12 Live `full` | PASS / CLOSED | `pipeline_full_20261005T035611324478Z` |
| M5 Hardening | PASS / CLOSED | `pipeline_full_from_extraction_20261005T105537280277Z` + M5 report |
| MVP Completion Gate | PASS | M5 SUCCESS |
| `.gitignore` full audit | PASS / CLOSED | MVP closure report |
| MVP | **COMPLETE** | 2026-10-05 |
| Source public release | HOLD | requester permission required |
| Public docs/images preparation | NEXT | privacy/specific-identifier audit + lossless docs update |

### v3.0時点の未完了表示 — 履歴保持

MVP completion前のv3.0では、以下がcurrent stateとして残っていた。現在はすべてclosure済みだが、進捗履歴を失わないため原文意味を保持する。

```text
| MVP Hardening / Release | IN PROGRESS | live E2E、correctness、release gateが残る |

### 1.2 現在のMVPクリティカルパス
Non-fetch E2E / resume recovery確認
GA4 + GSC live-fetch E2E
M5 correctness / determinism / lineage / security hardening
.gitignore全面監査
Release packaging / version decision
MVP closure

### 1.3 M5で必ず確認するVisualization correctness
````

当時の説明は「これは見た目の好みではなくformat contractのcorrectnessであり、M5で実値・`value_type`・`unit`・`format_hint`とReact表示処理を照合する。」であった。v0.12.4でこのGateを実際に閉じた。

```text
これは見た目の好みではなくformat contractのcorrectnessであり、M5で実値・`value_type`・`unit`・`format_hint`とReact表示処理を照合する。
```

### 完了までに実際に閉じた追加事項

1. R12 preflightでGA4 credential relative pathの構造移動後不整合を検出し修正。
2. GA4 Extractionが`31 exported / 1 skipped / 0 failed`の`partial_success`を生成した際、GA4 CLIの既存`partial_success = exit 1`契約をPipeline v0.12.2が理解していなかったため、v0.12.3でArtifact statusを確認するadapterへ修正。
3. R12 `full` Live-fetch E2Eをv0.12.3で完走。8 stages、hash/lineage、range、portability、Visualization、223 generated CSV parse gateをPASS。
4. M5でReact primaryのrelative delta percentage表示correctness bugを検出。v0.12.4で`ratio -> percent` formatterへ修正しregressionを追加。
5. R12 Extraction Artifactを再取得せず、v0.12.4 `full_from_extraction`で下流を再生成してM5全監査。
6. A+C / B+C、grain uniqueness、Cartesian growth、missing semantics、query attribution boundary、CTR/position semantics、determinism、Artifact/lineage/security/portabilityを監査。
7. MVP COMPLETEを正式判定し、`.gitignore`を全面監査。`config/system/**`等の標準Specをtrackし、actual user config / credentials / output / backups / cache等をignoreする境界を確定。

### 現在のRoadmap遷移

```text
MVP COMPLETE
  ↓
Public docs lossless update + privacy/specific-identifier audit    DONE / this v3.1 update
  ↓
Docs / public images / README / license-related documents-only publication    NEXT
  ↓
Post-MVP P1-P11（必要性と優先度に応じて着手）
```

**Source code公開はPost-MVP開始条件ではない。依頼者の許可が出るまでSource public releaseはHOLDする。**

## 0. 文書の目的

本書は、Generic Web Intelligence & Decision Platformの開発開始からMVP完成、さらにPost-MVPのDecision Platform拡張までを一本のロードマップとして管理する。

本更新では、元のv2.0に存在したPost-MVP要件・Phase・判断根拠を削除せず保持しつつ、2026-10-03時点の実装事実を前段へ追加した。

## 1. 現在地

### 1.1 完了済みの主要層

| 領域 | 状態 | 備考 |
|---|---|---|
| GA4 Extraction | DONE / HARDENED | OAuth/API取得、Artifact/Manifest |
| GA4 Processing | DONE / HARDENED | Source固有正規化、Period、Lineage |
| GA4 Analysis | DONE / HARDENED | Source単体分析、Quality/Reconciliation |
| GSC Extraction | DONE / HARDENED | Search Analytics、Coverage、OAuth共有 |
| GSC Processing | DONE / HARDENED | Source固有正規化、Metric semantics |
| GSC Analysis | DONE / HARDENED | Source単体分析、Coverage/Reconciliation |
| Integration Extraction | DONE | lossless、FULL OUTER相当、canonical page |
| Integration Processing | DONE | Config-driven dataset assembly |
| Integration Analysis | DONE | Generic Analysis Engine |
| Visualization | DONE / CLOSED | Generic Presentation Engine、React primary |
| Repo/namespace/config構造再編 | DONE | v0.11.0、194 tests確認 |
| Pipeline orchestration | DONE / CLOSED | 6 profiles、resume/recovery、R12 Live E2Eを含む |
| MVP Hardening / Completion | DONE / CLOSED | R12、M5、`.gitignore` full auditまでPASS |

### Pipelineの現在状態

- package: `0.12.4`
- state: **DONE / CLOSED**
- detail: Pipeline closure、R12 Live E2E、M5 Completion Gateまで完了。
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


### 1.2 MVPクリティカルパス完了記録

```text
Pipeline final gate                 DONE
  ↓
Non-fetch E2E / resume recovery    DONE
  ↓
GA4 + GSC live-fetch E2E (R12)    DONE
  ↓
M5 correctness / determinism / lineage / security hardening    DONE
  ↓
.gitignore全面監査                 DONE
  ↓
MVP completion decision            DONE
```

Release / publicationはMVP completionとは分離する。Source public releaseは依頼者の明示許可が出るまでHOLD。

### 1.3 M5で確認済みのVisualization correctness

比較値のrelative deltaが内部ではratioで保持される場合、UI表示で百分率へ正しく変換する。

```text
internal: -0.3105
correct display: -31.05%
incorrect display: -0.3105%
```

これは見た目の好みではなくformat contractのcorrectnessであり、M5で実値・`value_type`・`unit`・`format_hint`とReact表示処理を照合し、v0.12.4でPASSした。

### 1.4 MVP後のDashboard polish候補

以下はMVP closureを妨げない改善候補として保持する。

- desktop KPI配置を5+2から4+3へ見直す案
- Quality / Coverage / Lineageのraw JSONをaccordion化し、summaryは常時表示
- count型Y軸tickを整数表示
- headerのtechnical metadataをsecondary informationへ移す

## 2. MVP完成条件

MVP完成は「画面が出る」だけではない。最低限、次を同時に満たす。

1. 全層unit/contract/full regressionがgreen。
2. GA4/GSC live fetchを含む代表E2Eがgreen。
3. A+C primary fixtureとB+C period-controlでlineage、row count、coverageを確認。
4. Persistent Artifactへabsolute local pathやsecretを残さない。
5. ZERO / NOT_OBSERVED / UNAVAILABLE / UNKNOWN / NULLを混同しない。
6. GSC queryをGA4 session/conversionへexact attributionしない。
7. search_typeやquery grainでGA4 factsを重複加算しない。
8. Deterministic layerは同一input/config/specで再現可能。
9. Visualizationはupstream resultを再分析しない。
10. `.gitignore`をMVP completion直後に全面再監査する。
11. Release Artifactとpublic docsを整合させる。

## 3. Post-MVPロードマップ — 維持する将来拡張

元Roadmapに定義された将来構想は削除しない。大区分は以下を維持する。

| Phase | 領域 | 目的 |
|---|---|---|
| P1 | Website Intelligence | Crawl / Snapshot / Content / Link Graph |
| P2 | Change / Intervention | Change Detection / Intervention Registry |
| P3 | Modeling / ML | Feature / Prediction / Forecast / Anomaly |
| P4 | Experiment Platform | Experiment Registry / Treatment / Control |
| P5 | Causal Effect | Effect Estimation / Evidence |
| P6 | Decision Support | Action Candidate / EV / Cost / Risk |
| P7 | Semantic / Embedding | Chunk / Embedding / Similarity |
| P8 | LLM Platform | Grounded narrative / synthesis / drafting |
| P9 | Additional Sources | Business KPI / CMS / Ads / CRM等 |
| P10 | Scale / Storage / Ops | Parquet / DuckDB / Incremental / Catalog |
| P11 | Productization | Advanced orchestration / UI / Cloud等 |

これらはMVP Coreの責務境界を壊さず、Source / Artifact / Contract / Adapterを追加する形で段階拡張する。

## 4. バージョンとMilestoneの扱い

バージョン番号をMilestoneの代用にしない。`M4 = v0.x` のように固定せず、repo実態と変更規模に応じてversionを決める。

MVP completion時点のpackageは`0.12.4`。MVP COMPLETEを理由だけに`v1.0.0`へ番号合わせせず、実変更とrelease policyに基づいてversionを決める。


## 保存付録 — Master Roadmap v2.0 原本全文

この付録は、更新前の原本文書に存在した情報を不可逆に失わないための保存領域である。
**現行仕様の判断には本文前半を使用し、この付録は履歴・判断根拠・旧仕様の追跡に使用する。**
原文は内容を要約せず、そのまま保存している。

- preserved_source_sha256: `1ac63d243b6e66a107cd3bc6b5e4221e21d8dd6e2509c02ac64b0bc7ae3742fd`

````markdown
# Master Development Roadmap v2.0

```yaml
project_initiated_at: 2026-09-23 17:35:34 JST
created_at: 2026-09-30 JST
updated_at: 2026-09-30 JST
author: Nazuna Research
doc_ver: v2.0
note: 再初期化後の開発開始 → MVP完成 → Post-MVP拡張群
上位要件: GENERIC_WEB_INTELLIGENCE_DECISION_PLATFORM_全体要件定義書_v2.0_20260930.md
```

---

# 0. この文書の目的

本書は、本プロジェクトを再構築・再初期化して開発を始めた時点から、
GA4 / GSC Source Pipelineの完成、MVPのIntegration / Analysis / Visualization、
さらにMVP後のWebsite Intelligence / Intervention / Experiment / Modeling /
Causal / Decision Support / Semantic / LLM / Scale拡張までを、
**一本の開発ロードマップとして管理するための最新版Master Roadmap**である。

本書の役割は以下。

```text
1. これまで何を完了したかを明確にする
2. 現在どこにいるかを明確にする
3. 次に何を作るかを明確にする
4. 各Phaseの依存関係とExit Criteriaを固定する
5. MVP完成条件を明確にする
6. MVP後の拡張順序と依存関係を明確にする
7. 将来機能をMVPへ混ぜ込まない
8. Version番号と機能境界を混同しない
9. 各Phaseで実Artifact検証まで行う
10. 最終的なDecision Platformまでの道筋を見失わない
```

本書は詳細な要件定義書ではない。
各機能の厳密な責務・Contract・Acceptance Criteriaは、
最新版の全体要件定義書を優先する。

---

# 1. 進捗サマリー

## 1.1 Status Legend

```text
DONE
  実装 + test + 実Artifact確認まで完了

DONE / HARDENED
  完了後に追加の契約修正・長期検証まで実施済み

IN PROGRESS
  実装中

READY
  前提が揃い、すぐ着手可能

NOT STARTED
  未着手

DEFERRED
  意図的に後段へ送っている

OPTIONAL
  必須ではなく、必要時に導入
```

---

# 2. 全体進捗表

| No. | Phase / Milestone | 主な成果 | Status | 現在の位置づけ |
|---:|---|---|---|---|
| R0 | 再初期化・要件再構築 | 要件復元、Repo再整理、開発方針固定 | DONE | 歴史 |
| R1 | GA4 Extraction | GA4 API取得、Manifest、Artifact | DONE | Source完成 |
| R2 | GA4 Processing | Raw→分析可能データ、Period、Config | DONE / HARDENED | Source完成 |
| R3 | Layered Layout / Tooling | Source層整理、Backup、Run構造 | DONE | 基盤 |
| R4 | GA4 Analysis | Source Analysis、Period比較、診断 | DONE / HARDENED | Source完成 |
| R5 | GA4 Analysis Reconciliation | Reconciliation / Quality / Lineage | DONE | Source完成 |
| R6 | Shared Google Auth + GSC Extraction | OAuth共通化、GSC取得 | DONE | Source完成 |
| R7 | GSC Processing / Analysis | GSC加工、分析、Coverage、Reconciliation | DONE / HARDENED | Source完成 |
| R8 | Cross-source Contract Hardening | Page key、Date、Lineage、Missing semantics | DONE / HARDENED | Integration前提完成 |
| R9 | Long-period Reference Artifacts | GA4 2年、GA4同期間、GSC16か月 | DONE | A/B/C fixture完成 |
| M1 | Integration Extraction | Lossless統合、FULL OUTER、Quality | NOT STARTED | **次に着手** |
| M2 | Integration Processing | Config-driven Dataset Assembly | NOT STARTED | MVP |
| M3 | Integration Analysis | Generic Analysis Engine | NOT STARTED | MVP |
| M4 | Visualization | Generic Static HTML Renderer | NOT STARTED | MVP |
| M5 | MVP Hardening / Release | E2E、Docs、Release Gate | NOT STARTED | MVP完成 |
| P1 | Website Intelligence | Crawl / Snapshot / Content / Link Graph | NOT STARTED | Post-MVP |
| P2 | Change / Intervention | Change Detection / Intervention Registry | NOT STARTED | Post-MVP |
| P3 | Modeling / ML | Feature / Prediction / Forecast / Anomaly | NOT STARTED | Post-MVP |
| P4 | Experiment Platform | Experiment Registry / Treatment / Control | NOT STARTED | Post-MVP |
| P5 | Causal Effect | Effect Estimation / Evidence | NOT STARTED | Post-MVP |
| P6 | Decision Support | Action Candidate / EV / Cost / Risk | NOT STARTED | Post-MVP |
| P7 | Semantic / Embedding | Chunk / Embedding / Similarity | NOT STARTED | Post-MVP |
| P8 | LLM Platform | Grounded LLM Sidecar / Narrative / Draft | NOT STARTED | Post-MVP |
| P9 | Additional Source Adapters | Business KPI / CMS / Ads等 | NOT STARTED | Post-MVP |
| P10 | Scale / Storage / Ops | Parquet / DuckDB / Incremental / Catalog | NOT STARTED | Post-MVP / Optional |
| P11 | Advanced Orchestration / Productization | Unified DAG / UI / Cloud等 | DEFERRED | 将来判断 |

---

# 3. 現在地

2026-09-30時点の現在地:

```text
GA4 Extraction     DONE
GA4 Processing     DONE
GA4 Analysis       DONE

GSC Extraction     DONE
GSC Processing     DONE
GSC Analysis       DONE

Cross-source Contract
  Page normalization  DONE
  Date canonical      DONE
  Relative lineage    DONE
  Coverage semantics  DONE
  UNAVAILABLE != ZERO DONE
  Position tolerance  DONE

Reference Artifacts
  A: GA4 2年          DONE
  B: GA4同期間        DONE
  C: GSC約16か月     DONE

--------------------------------
NEXT:
  Integration Extraction
--------------------------------
```

---

# 4. 現在の基準Artifact

## Dataset A — GA4 約2年

```text
2024-09-27 -> 2026-09-26
730日
```

主用途:

```text
Integration Extraction主fixture
GA4-only historical period保持
long-term GA4 validation
```

---

## Dataset B — GA4 GSC同requested期間版

```text
2025-05-26 -> 2026-09-26
489日
```

主用途:

```text
B + C対照test
requested period同一時のjoin検証
```

---

## Dataset C — GSC 約16か月

```text
requested:
  2025-05-26 -> 2026-09-26

observed:
  2025-05-26 -> 2026-09-24
```

主用途:

```text
A + C主integration fixture
reporting lag
Source coverage差
```

---

# 5. Roadmapの大区分

```text
HISTORICAL SOURCE BUILD
  R0 - R9

MVP CORE
  M1 - M5

POST-MVP INTELLIGENCE
  P1 - P10

ADVANCED PRODUCTIZATION
  P11+
```

---

# PART I — HISTORICAL SOURCE BUILD

# 6. R0 — 再初期化・要件再構築

## Goal

既存の断片的実装・要件・Artifactを整理し、
再び継続開発可能な基盤へ戻す。

## 主な作業

```text
Context reconstruction
再要件定義
Repo状態確認
Python / uv環境固定
Artifact構造確認
Manifest方針
Run単位Output方針
Security / Secret方針
```

## Exit

```text
現在repoをSource of Truthとして開発可能
再現可能なBuild / Test
要件と実装の対応が追える
```

Status:

```text
DONE
```

---

# 7. R1 — GA4 Extraction

## Goal

GA4 Data APIから、
分析に必要なSource Dataを安定取得。

## 主成果

```text
Report definitions
Date range
Pagination
Retry
Empty / Skip / Degraded handling
Manifest
Metadata
Run Artifact
```

## Exit

```text
Live extraction可能
Raw Artifact保持
Manifest row counts
Expected skip識別
```

Status:

```text
DONE
```

---

# 8. R2 — GA4 Processing

## Goal

GA4 Raw Artifactを、
Source固有の加工責任を完了した分析可能Datasetへ変換。

## 主成果

```text
URL processing
Period windows
Daily tables
Event grouping
Config
Processing Manifest
Lineage
```

## 後続Hardening

```text
relative source_run_directory
normalized_page_key
date YYYY-MM-DD
schema version update
```

Status:

```text
DONE / HARDENED
```

---

# 9. R3 — Layered Layout / Tooling

## Goal

Source Extraction / Processing / Analysisの境界を物理構造にも反映。

## 主成果

```text
Layered output layout
Run ID
Backup tooling
Latest resolver
Shared utilities
```

Status:

```text
DONE
```

---

# 10. R4 — GA4 Analysis

## Goal

GA4 Sourceだけでも分析可能にする。

## 主成果

```text
Period comparison
Continuity
Dimension periods
Summary
Source diagnostic
Analysis Manifest
```

## Hardening

```text
Processing ISO dateのみを読む
relative source_processing_directory
```

Status:

```text
DONE / HARDENED
```

---

# 11. R5 — GA4 Reconciliation / Quality

## Goal

Analysis結果と元Datasetの整合性を追跡。

## 主成果

```text
same-grain reconciliation
quality state
lineage validation
difference reporting
```

Status:

```text
DONE
```

---

# 12. R6 — Shared Google Auth + GSC Extraction

## Goal

GA4 / GSCでOAuth基盤を共有し、
GSC Search Analytics Dataを取得。

## 主成果

```text
Shared Google OAuth
AuthorizedSession
GSC presets
Search type
Query / Page dimensions
Manifest
Coverage metadata
```

Status:

```text
DONE
```

---

# 13. R7 — GSC Processing / Analysis

## Goal

GSCをGA4同様、
Extraction / Processing / Analysisまで独立完成。

## 主成果

```text
GSC Processing
GSC Analysis
Period windows
Coverage
Page normalization
Metric semantics
Position semantics
Reconciliation
Quality
```

## Hardening

```text
observed coverage
relative source_manifest_path
UNAVAILABLE != ZERO
position tolerance
```

Status:

```text
DONE / HARDENED
```

---

# 14. R8 — Cross-source Contract Hardening

## Goal

Integrationへ進む前に、
Source Processing間の共通契約を揃える。

## 主成果

```text
normalized_page_key:
  site-scoped path

date:
  YYYY-MM-DD

persistent lineage:
  relative

missing semantics:
  explicit

url normalization:
  same policy
```

## 重要教訓

```text
Source Processingの責任を
Integrationへ押し付けない。
```

Status:

```text
DONE / HARDENED
```

---

# 15. R9 — Long-period Reference Artifact

## Goal

MVP Integrationを短期間fixtureだけでなく、
実運用に近い長期データで検証できる状態にする。

## Output

```text
A: GA4 2年
B: GA4 16か月相当
C: GSC 16か月相当
```

## Exit

```text
Lineage一致
Row count一致
Canonical date
Relative path
Coverage正常
```

Status:

```text
DONE
```

---

# PART II — MVP CORE

# 16. MVP Version Strategy

MVPのVersion意図:

```text
v0.7.x
  Source stabilization
  -> DONE

v0.8.x
  Integration Extraction
  Integration Processing

v0.9.x
  Integration Analysis
  Analysis Specs
  Hardening

v1.0.0
  Visualization
  Full E2E
  MVP Release
```

実際のminor / patchは、
最新repoのversion履歴と実装単位を見て決定。

無意味なversion bumpをしない。

---

# 17. M1 — Integration Extraction

## Goal

GA4 / GSC Processing Artifactを、
**一切の用途都合で削らずlosslessに統合**。

## Core Principle

```text
全部残して、正しくつなぐ
```

## Input

```text
GA4 Processing
GSC Processing
```

## 必須機能

```text
Input selection
Manifest validation
Hash validation
Schema compatibility
Site mapping
Date contract
Page key contract
FULL OUTER相当
Source observation state
Join state
Canonical page identity
Quality
Dataset Catalog
Manifest
CLI
```

## 禁止

```text
common period trim
latest N trim
YoY ready trim
use case
score
ranking
recommendation
SEO-specific judgment
```

## 主fixture

```text
A + C
```

## Secondary fixture

```text
B + C
```

## Major Risk

```text
search_type x GA4 metrics
query grain x GA4 context
Cartesian explosion
double counting
```

## Exit Criteria

```text
A+C union期間保持
GA4_ONLY / GSC_ONLY / MATCHED
GSC lagを0化しない
No duplicate canonical grain
No unexpected row multiplication
Query attribution safety
Lineage hash
Quality artifact
No absolute path
No secret
All tests green
Live Artifact green
```

Status:

```text
NOT STARTED
```

---

# 18. M2 — Integration Processing

## Goal

Integration Extraction Artifactを、
Configに従って具体的Use Case用のDatasetへ加工。

## Core Principle

```text
Configで、今回使う形に加工する
```

## Default Period Policy

```text
common_observed
```

A+C default:

```text
2025-05-26 -> 2026-09-24
```

## 必須Period Modes

```text
common_observed
all_available
source_available
explicit_range
latest_n_days
comparison_ready
```

## 必須Concept

```text
required_sources
optional_sources
grain
dimensions
metrics
filters
coverage policy
period
comparison window
```

## Safe Aggregation

```text
SUM_SAFE
DERIVED_RATIO
NON_ADDITIVE
WEIGHTED_WITHIN_GRAIN
NON_ADDITIVE_CONTEXT
```

## Output

```text
processed dataset
dataset_schema
metric_registry_snapshot
period_definition
coverage_summary
processing_manifest
```

## Exit Criteria

```text
A+C default -> common_observed
A+C all_available -> union保持
GA4-only config
GSC-only config
safe CTR
safe position
non-additive保護
filter reason tracking
config hash
deterministic
all tests green
live Artifact green
```

Status:

```text
NOT STARTED
```

---

# 19. M3 — Integration Analysis

## Goal

Processing Contractを満たすDatasetなら、
Source / Use CaseをCore codeで決め打ちせず分析。

## Core Principle

```text
来たDatasetを、契約に従って汎用分析する
```

## 必須Operation

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

## Result Status

```text
OK
BASELINE_ZERO
BASELINE_UNAVAILABLE
CURRENT_UNAVAILABLE
INSUFFICIENT_COVERAGE
```

## Analysis Spec

```text
Processing Config:
  対象を決める

Analysis Spec:
  処理を決める
```

## Genericity Gate

同一EngineをCore変更なしで最低:

```text
cross-source page dataset
GA4-only long-term dataset
```

へ適用。

## Exit Criteria

```text
No source-specific branch proliferation
No use-case-specific branch proliferation
Metric semantics aware
Coverage aware
Missing aware
Spec driven
Deterministic
Lineage
Multiple dataset live test
```

Status:

```text
NOT STARTED
```

---

# 20. M4 — Visualization

## Goal

Analysis Artifactを、
Source / Use Case固有dashboardへ固定せず可視化。

## Core Principle

```text
来た分析結果を、契約に従って汎用表示する
```

## MVP Output

```text
Static HTML
```

## 必須Component

```text
KPI
Time Series
Comparison Table
Ranking Table
Quality / Coverage
Metadata / Lineage
```

## Optional MVP Component

```text
Bar Chart
Scatter Plot
Simple Filter
Small Multiple
```

## 禁止

```text
YoY再計算
CTR再集計
Position再集計
Business metric再定義
score生成
```

## Exit Criteria

```text
Generic renderer
Multiple Analysis Artifact
Static HTML
Portable
Responsive minimum
Quality visible
Lineage visible
No absolute path
No secret
```

Status:

```text
NOT STARTED
```

---

# 21. M5 — MVP Hardening / Release

## Goal

GA4 / GSCからStatic HTMLまでを、
1本の監査可能なProductとして完成。

## Full E2E

```text
GA4 Processing A
        \
         Integration Extraction
        /
GSC Processing C
        ↓
Integration Processing
        ↓
Integration Analysis
        ↓
Visualization
        ↓
Static HTML
```

## 必須Validation

```text
Unit
Contract
Golden fixture
Live Artifact
Regression
E2E
Determinism
Manifest chain
Hash chain
Row count
Schema version
Config / Spec hash
Secret scan
Absolute path scan
Duplicate grain scan
Cartesian growth scan
```

## Documentation

最低限:

```text
Install
Source pipeline
Integration pipeline
Config example
Analysis spec example
Visualization usage
Artifact layout
Known limitations
```

## Release Gate

```text
Source regression green
Integration Extraction green
Integration Processing green
Integration Analysis green
Visualization green
A+C E2E green
B+C comparison green
No known data corruption
No known contract ambiguity
```

## MVP Complete

```text
v1.0.0相当
```

Status:

```text
NOT STARTED
```

---

# PART III — POST-MVP INTELLIGENCE

# 22. Post-MVP Development Principle

Post-MVPは、
MVP Coreを作り直すためのPhaseではない。

原則:

```text
新機能
  ↓
新Source / 新Artifact / 新Specとして追加
  ↓
既存Integration / Analysis Contractへ接続
```

避ける:

```text
Website対応のためGA4/GSCを再設計
LLM導入のためAnalysisをLLM依存
ML導入のためCanonical Modelを破壊
Decision SupportのためObserved Factを書き換え
```

---

# 23. Post-MVP Version Policy

MVP後のversion番号は、
現時点では固定しない。

推奨考え方:

```text
v1.x:
  additive feature expansion

v2.x:
  public contractの大規模breaking changeが必要な場合のみ
```

Phase番号 P1-P11 はVersion番号ではない。

---

# 24. P1 — Website Intelligence

## Goal

Performance Dataだけでなく、

```text
「その時点でサイトがどういう状態だったか」
```

をfirst-class dataとして取得。

## Sub-phases

```text
P1.1 Website Extraction
P1.2 Website Processing
P1.3 Snapshot Versioning
P1.4 Content Block
P1.5 Internal Link Graph
P1.6 Website Analysis
P1.7 Integration Connection
```

## Website Extraction

```text
requested URL
final URL
status
redirect
canonical
robots
title
meta
headings
body text
structured data
links
images
language
hashes
timestamp
```

## Safety

```text
GET / HEAD
allowed host
rate limit
timeout
max pages
max bytes
no form submit
no state-changing action
robots respect default
```

## Processing

```text
page identity
content normalization
heading tree
link normalization
snapshot ID
hash
duplicate metadata
```

## Internal Link Graph

```text
source
target
anchor
context
nofollow
snapshot
```

## Exit Criteria

```text
safe crawl
snapshot version
page identity
content hash
HTML hash
content block
link graph
quality
Integration接続
```

Dependency:

```text
MVP Core
```

Status:

```text
NOT STARTED
```

---

# 25. P2 — Change Detection / Intervention Registry

## Goal

Performance変化へ、

```text
何が変わったか
いつ変わったか
意図的施策か
```

というContextを与える。

## P2.1 Snapshot Diff

Detected Change:

```text
title
meta
heading
content
link
CTA
canonical
redirect
structured data
```

## P2.2 Intervention Registry

Intentional action:

```text
intervention_id
action_type
target
start/end
before/after snapshot
effort
cost
status
notes
```

## 重要

```text
Detected Change != Intervention
```

自動検出を施策と断定しない。

## Exit Criteria

```text
diff
registry
before/after linkage
cost/effort
concurrent change flags
lineage
Integration接続
```

Dependency:

```text
P1 Website Snapshot
```

Status:

```text
NOT STARTED
```

---

# 26. P3 — Modeling / Machine Learning

## Goal

記述分析に加えて:

```text
Prediction
Forecast
Anomaly
Clustering
Ranking
Representation
```

を扱う。

## P3.1 Feature Dataset

```text
versioned feature set
feature schema
feature lineage
missing policy
lookback
leakage flags
```

## P3.2 Model Training

候補:

```text
linear / regularized
tree / boosting
time series
anomaly detector
clustering
ranking
representation
```

## P3.3 Validation

Time-dependent:

```text
past -> train
future -> validation
```

必要に応じ:

```text
rolling validation
backtest
```

## P3.4 Model Registry

```text
model run
training data hash
feature version
parameters
seed
metrics
artifact hash
```

## P3.5 Prediction Contract

```text
prediction
target
interval
model run
feature snapshot
quality
```

## 重要

```text
Prediction != Causal Effect
```

## Exit Criteria

```text
feature version
time-aware validation
model registry
reproducible seed
baseline comparison
prediction contract
no causal overclaim
```

Dependency:

```text
MVP Core
```

Website / Interventionは必須ではないが、
Feature richnessのため有用。

Status:

```text
NOT STARTED
```

---

# 27. P4 — Experiment Platform

## Goal

施策をExperimentとして構造化し、
効果検証可能にする。

## Registry

```text
experiment_id
hypothesis
unit
treatment
control
start/end
primary metric
secondary metrics
assignment
status
intervention IDs
```

## Quality

```text
assignment integrity
sample size
metric availability
contamination
concurrent intervention
missing observation
```

## Exit Criteria

```text
registry
treatment/control
assignment metadata
metric contract
quality
lineage
```

Dependency:

```text
MVP Core
P2 Intervention推奨
```

Status:

```text
NOT STARTED
```

---

# 28. P5 — Causal Effect Estimation

## Goal

単なる相関 / Predictionではなく、

```text
特定介入がMetricへ与えた効果
```

を推定。

## Candidate Methods

```text
controlled before/after
difference-in-differences
interrupted time series
synthetic control
matching
uplift
randomized experiment analysis
Bayesian effect estimation
```

## Effect Contract

```text
effect
interval
method
evidence level
assumptions
period
sample size
quality flags
intervention / experiment
supporting artifacts
```

## Evidence Level

```text
E0_HEURISTIC
E1_CORRELATIONAL
E2_PREDICTIVE
E3_QUASI_EXPERIMENTAL
E4_RANDOMIZED_EXPERIMENT
```

## Failure Policy

Data不足:

```text
INSUFFICIENT_DATA
LOW_EVIDENCE
```

数値を捏造しない。

## Exit Criteria

```text
method metadata
assumptions
effect interval
evidence
quality
reproducibility
prediction separation
```

Dependency:

```text
P2 and/or P4
Observed performance
```

Status:

```text
NOT STARTED
```

---

# 29. P6 — Decision Support

## Goal

Analysis / Prediction / Causal Artifactを、
Action Candidateへ変換。

## Candidate Contract

```text
action
target
objective
expected impact
uncertainty
effort
cost
risk
evidence
dependencies
reason codes
supporting artifacts
```

## Expected Value

Config-driven:

```text
benefit
cost
effort
risk
uncertainty penalty
evidence
```

## Boundary

System:

```text
Candidate生成
根拠提示
比較材料提示
```

Human:

```text
最終意思決定
```

## No Fabrication

Effect evidenceなし:

```text
具体的Impact数値を捏造しない
```

## Exit Criteria

```text
candidate contract
evidence-aware
uncertainty
cost/effort/risk
utility config
lineage
human boundary
```

Dependency:

```text
M3 Analysis minimum

Better:
  P3 Prediction
  P5 Causal
```

Status:

```text
NOT STARTED
```

---

# 30. P7 — Semantic / Embedding

## Goal

文字列一致を超えた:

```text
topic
intent
similarity
semantic fit
content gap
related content
```

を扱う。

## P7.1 Chunk

```text
chunk ID
page
snapshot
block
text
text hash
strategy
version
```

## P7.2 Embedding

```text
provider
model
model version
dimension
input hash
```

## P7.3 Derived Semantic Artifact

```text
query intent
page topic
query-page similarity
cluster
semantic gap
related pages
```

## Core Principle

Vector DBはOptional。

Core Integration / AnalysisはVector DB非依存。

## Exit Criteria

```text
chunk version
embedding version
input hash
rebuildable
lineage
Core非依存
```

Dependency:

```text
MVP Core
P1 Website推奨
```

Status:

```text
NOT STARTED
```

---

# 31. P8 — LLM Platform

## Goal

Deterministic Coreを壊さず、
定性理解・説明・Narrativeを追加。

## Suitable Tasks

```text
intent classification
topic classification
content diagnosis
summary
analysis synthesis
reason explanation
recommendation draft
report narrative
```

## LLMに任せない

```text
join
metric arithmetic
CTR
position
period comparison
hash
coverage
schema validation
exact reconciliation
```

## Provider Abstraction

```text
capabilities
submit
result
```

Provider / transportをCoreから隔離。

## Grounding

```text
Artifact IDs
Dataset IDs
Page Snapshot
Metric context
Evidence
Quality
```

## Governance

```text
prompt version
input hash
cache
budget
failure isolation
external transfer opt-in
```

## Exit Criteria

```text
LLM disabledでもCore完動
Grounded
Prompt versioned
Provider optional
Failure isolated
No deterministic metric calculation
External opt-in
```

Dependency:

```text
MVP Core

Better:
  P1 Website
  P7 Semantic
  P6 Decision Support
```

Status:

```text
NOT STARTED
```

---

# 32. P9 — Additional Source Adapters

## Goal

GA4 / GSC / Website以外のDataを追加可能にする。

## Candidate Sources

```text
Business KPI
Revenue
CRM
Offline Conversion
Google Ads / Other Ads
CMS
Search ranking provider
Manual annotation
Experiment Log
External market data
```

## Priority Candidate

### Business KPI

```text
revenue
lead
appointment
qualified conversion
margin
customer value
```

### CMS

```text
publish date
author
template
category
revision
status
```

### Ads

```text
spend
click
impression
conversion
campaign
keyword
```

## Principle

```text
Source Extraction
↓
Source Processing
↓
Optional Source Analysis
↓
Integration Extraction
```

Integration CoreをSourceごとに改造しない。

Status:

```text
NOT STARTED
```

---

# 33. P10 — Scale / Storage / Operational Enhancement

## Goal

Data量・運用頻度増加に応じ、
性能と運用性を強化。

## Optional Components

```text
Parquet
DuckDB
Incremental Extraction
Incremental Processing
Resume
Checkpoint
Artifact Catalog
Schema Registry
Retention Policy
Compression
Partitioning
```

## Parquet

導入理由:

```text
size
speed
type
large dataset
```

## DuckDB

導入理由:

```text
local analytical SQL
Parquet query
large join
ad-hoc analysis
```

## Artifact Catalog

```text
artifact ID
run ID
type
schema
created
parents
hash
status
```

## Schema Registry

```text
schema version
compatibility
migration
```

## Principle

必要性の前に導入しない。

MVPをDB必須にしない。

Status:

```text
NOT STARTED
```

---

# PART IV — ADVANCED ORCHESTRATION / PRODUCTIZATION

# 34. P11 — Unified Orchestrator

## Goal

完成した独立Layerを、
One-command / DAGとして実行可能にする。

## Important

Orchestratorが:

```text
Extraction
Processing
Analysis
Visualization
```

の責務を吸収してはいけない。

単なる:

```text
dependency resolution
execution
resume
status
```

を行う。

## Example Future Flow

```text
pipeline run
  -> source extract
  -> source process
  -> integration extract
  -> integration process
  -> analysis
  -> visualize
```

Status:

```text
DEFERRED
```

---

# 35. P12 — Product UI / Web Application

## Goal

必要になった場合のみ、
CLI / Artifact中心の基盤へUIを載せる。

候補:

```text
Run selection
Config editor
Report browser
Artifact explorer
Lineage viewer
Decision candidate review
Experiment dashboard
```

MVPでは不要。

Status:

```text
DEFERRED
```

---

# 36. P13 — Cloud / Multi-user / SaaS

将来必要になった場合のみ。

候補:

```text
Auth
Tenant isolation
Queue
Object storage
DB
Worker
Scheduler
Audit log
Role
Quota
Billing
```

現時点ではScope外。

Status:

```text
DEFERRED
```

---

# PART V — CROSS-PHASE DEPENDENCY

# 37. Dependency Graph

```text
R0-R9
  ↓
M1 Integration Extraction
  ↓
M2 Integration Processing
  ↓
M3 Integration Analysis
  ↓
M4 Visualization
  ↓
M5 MVP Release

M5
 ├─> P1 Website Intelligence
 │      ↓
 │    P2 Change / Intervention
 │      ↓
 │    P4 Experiment
 │      ↓
 │    P5 Causal
 │
 ├─> P3 Modeling
 │
 ├─> P7 Semantic
 │
 └─> P9 Additional Sources

P3 ───────────────┐
P5 ───────────────┼─> P6 Decision Support
M3 Analysis ──────┘

P1 / P7 / P6 ───────> P8 LLM Platform

Data scale growth ───> P10 Scale / Storage

Stable layers ───────> P11 Orchestrator
```

---

# 38. Parallelizable Work

MVP完成後、以下は一部並列可能。

```text
P1 Website Intelligence
P3 Modeling
P7 Semantic
P9 Additional Sources
```

ただし:

```text
P5 Causal
```

はIntervention / ExperimentのData Contractが整うほど価値が高い。

```text
P6 Decision Support
```

はAnalysisだけでもHeuristic Candidateを作れるが、
Prediction / Causal追加後にEvidence Qualityが大きく向上する。

---

# 39. Non-parallel Hard Dependencies

```text
Integration Processing
  requires Integration Extraction

Integration Analysis
  requires Integration Processing Contract

Visualization
  requires Analysis Output Contract

Causal
  requires Intervention / Experiment / Observed Data

Website Diff
  requires Website Snapshot

Grounded page-level LLM
  benefits from Website Snapshot / Content Block

Decision EV
  requires explicit Utility Config
```

---

# PART VI — EVIDENCE MATURITY ROADMAP

# 40. Evidence Evolution

本システムは、単に機能を増やすのではなく、
Evidenceの質を段階的に高める。

```text
Stage 1:
  OBSERVED
  「何が起きたか」

Stage 2:
  DERIVED
  「どう変化したか」

Stage 3:
  HEURISTIC / DIAGNOSTIC
  「どこに問題・機会がありそうか」

Stage 4:
  PREDICTED
  「今後どうなりそうか」

Stage 5:
  CAUSAL_ESTIMATE
  「施策が何を変えた可能性が高いか」

Stage 6:
  DECISION SUPPORT
  「何を優先すべきかを比較する」

Stage 7:
  LLM_GENERATED EXPLANATION
  「根拠を人間が理解・活用しやすく説明する」
```

---

# 41. Evidenceの混同禁止

```text
Observed != Derived
Derived != Heuristic
Heuristic != Prediction
Prediction != Causal
Causal != Guaranteed Outcome
LLM Narrative != Evidence
```

この分類は将来Artifactへ明示する。

---

# PART VII — QUALITY MATURITY ROADMAP

# 42. Quality Level 1 — Source

```text
row count
schema
coverage
source status
lineage
```

DONE。

---

# 43. Quality Level 2 — Integration

```text
match rate
source-only
join multiplication
key validity
metric semantics
```

M1。

---

# 44. Quality Level 3 — Analysis

```text
coverage-aware result
baseline availability
semantic safety
```

M3。

---

# 45. Quality Level 4 — Website

```text
crawl completeness
snapshot freshness
parse quality
link quality
```

P1。

---

# 46. Quality Level 5 — Modeling

```text
leakage
validation
baseline
drift
uncertainty
```

P3。

---

# 47. Quality Level 6 — Causal

```text
assumptions
control quality
sample
concurrent intervention
evidence level
```

P5。

---

# 48. Quality Level 7 — Decision

```text
evidence
uncertainty
cost
effort
risk
utility config
```

P6。

---

# PART VIII — LINEAGE MATURITY ROADMAP

# 49. MVP Lineage

```text
GA4 Extraction
 -> GA4 Processing
    \
     Integration Extraction
    /
GSC Processing
 <- GSC Extraction

Integration Extraction
 -> Integration Processing
 -> Integration Analysis
 -> Visualization
```

---

# 50. Post-MVP Lineage

```text
Website Snapshot
Intervention
Experiment
Feature Set
Model Run
Prediction
Effect Estimate
Decision Candidate
LLM Narrative
```

全て親Artifactへ追跡可能にする。

---

# 51. Ultimate Lineage Goal

最終画面上の:

```text
「この施策候補」
```

から、

```text
どのMetric
どのAnalysis
どのModel
どのEffect Estimate
どのConfig
どのSnapshot
どのSource Run
```

へ遡れる。

---

# PART IX — DEVELOPMENT PRACTICE

# 52. 実装方式

複数fileへまたがる実装:

```text
原則 one-shot .sh
```

Script側:

```text
repo root確認
前提version / file
backup
source変更
tests
config
lock必要時更新
ruff --fix
ruff format
compile
pytest
final ruff
```

単純な実行:

```text
command数行
```

巨大Scriptにしない。

---

# 53. Phase Gate

各Phase:

```text
Code done
≠
Phase done
```

最低:

```text
tests
lint
actual artifact
manifest
row count
lineage
quality
```

まで確認してDONE。

---

# 54. Backward Regression

新Layer実装時も:

```text
GA4 Source tests
GSC Source tests
```

を壊さない。

Post-MVPでもMVP E2E regressionを残す。

---

# 55. Documentation Timing

Critical Path:

```text
implementation
test
artifact validation
```

を優先。

README / polished docsはRelease Gateで整える。

---

# PART X — RELEASE GATES

# 56. MVP Release Gate

```text
[ ] GA4 Source green
[ ] GSC Source green
[ ] Integration Extraction green
[ ] Integration Processing green
[ ] Integration Analysis green
[ ] Visualization green
[ ] A+C E2E green
[ ] B+C comparison green
[ ] Lineage chain green
[ ] No secret
[ ] No machine path
[ ] Deterministic
[ ] User-facing docs
```

---

# 57. Website Intelligence Release Gate

```text
[ ] Safe crawler
[ ] Snapshot version
[ ] Page identity
[ ] Content
[ ] Link graph
[ ] Quality
[ ] Integration
[ ] Change-ready
```

---

# 58. Modeling Release Gate

```text
[ ] Feature version
[ ] Data lineage
[ ] Time validation
[ ] Seed
[ ] Baseline
[ ] Model registry
[ ] Prediction contract
[ ] No causal overclaim
```

---

# 59. Causal Release Gate

```text
[ ] Intervention / Experiment linkage
[ ] Method
[ ] Assumptions
[ ] Effect interval
[ ] Evidence
[ ] Quality
[ ] Reproducibility
```

---

# 60. Decision Support Release Gate

```text
[ ] Candidate contract
[ ] Evidence aware
[ ] No fabricated impact
[ ] Cost / effort / risk
[ ] Utility version
[ ] Human boundary
[ ] Supporting lineage
```

---

# 61. LLM Release Gate

```text
[ ] Optional
[ ] Core works disabled
[ ] Grounded
[ ] Prompt version
[ ] Input hash
[ ] Cache
[ ] Budget
[ ] Failure isolation
[ ] External transfer opt-in
[ ] No metric arithmetic
```

---

# PART XI — PRIORITY AFTER MVP

# 62. 推奨優先順位

MVP完成直後の推奨順:

```text
Priority A:
  Website Intelligence

Priority B:
  Change / Intervention Registry

Priority C:
  Modeling基盤
  Experiment Registry

Priority D:
  Causal Effect

Priority E:
  Decision Support

Priority F:
  Semantic / Embedding
  LLM Platform

Priority G:
  Scale / Storage
  Additional Sources
  Productization
```

理由:

```text
Websiteが入ると
「Performanceの変化」
と
「Page / Siteが何だったか」
を結びつけられる。

Interventionが入ると
「何を変えたか」
を扱える。

Causalが入ると
「変えた結果どうなったか」
を評価できる。

Decision Supportが入ると
「次に何をするか」
へ繋がる。
```

ただしBusiness用途によっては、
P9 Business KPI SourceをWebsiteより先に入れる合理性もある。

---

# 63. 最終形までの一本道

```text
[再初期化]
      ↓
GA4 Source
      ↓
GSC Source
      ↓
Source Contract Hardening
      ↓
Integration Extraction
      ↓
Integration Processing
      ↓
Integration Analysis
      ↓
Visualization
      ↓
=====================
      MVP COMPLETE
=====================
      ↓
Website Intelligence
      ↓
Change / Intervention
      ↓
Experiment / Modeling
      ↓
Causal Effect
      ↓
Decision Support
      ↓
Semantic / Embedding
      ↓
Optional LLM
      ↓
Scale / Operationalization
      ↓
=====================
Generic Web Intelligence
 & Decision Platform
=====================
```

---

# 64. 最終到達点

最終的に、本システムは:

```text
検索
行動
成果
コンテンツ
サイト構造
変更履歴
実験
予測
施策効果
コスト
リスク
定性情報
```

を、
同一の監査可能なData / Artifact / Lineageの考え方で扱う。

そして、

```text
何が起きているか
なぜ起きていそうか
何が起きそうか
何を変えた結果どうなったか
次に何をする候補があるか
その候補の根拠は何か
```

を順に提供し、

> **定量化されたデータにより、意思決定を補助する**

ところまで到達する。

---

# 65. 現在の次の一手

このRoadmapの次の実作業は明確。

```text
M1:
Integration Extraction
```

実装開始前:

```text
最新repoを精査
最新版全体要件定義書 v2.0を読む
最新引き継ぎ書を読む
```

その後:

```text
Integration Extraction実装
↓
A+C live validation
↓
Gate通過
↓
Integration Processing
```

ここから再開する。

以上。
````
