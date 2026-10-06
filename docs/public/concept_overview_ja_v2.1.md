# 汎用Webインテリジェンス意思決定プラットフォーム コンセプト概要

```yaml
document_id: concept_overview
document_type: concept
doc_ver: v2.1
language: ja
project_name: Generic Web Intelligence & Decision Platform
project_initiated_at: 2026-09-23 17:35:34 JST
created_at: 2026-09-27 JST
updated_at: 2026-10-05 22:57 JST
author: Nazuna Research
status: active
scope: MVP concept + extensible end-state
note: v2.0までのConceptをlossless保持し、MVP COMPLETE / R12 / M5 / publication boundaryを現行化
previous_note: 旧MVP Conceptを保持し、Integration/Visualization/Pipelineの現行境界へ更新
preserved_source_sha256: 01695c6d718f0d171a52bd5d065ae3515cb74a6accd84954784c7a456b50bc5b
supersedes: GENERIC_WEB_INTELLIGENCE_PLATFORM_CONCEPT_OVERVIEW_MVP_v1.0_20260927.md
previous_public_version: concept_overview_ja_v2.0.md
mvp_completion_state: COMPLETE
current_package: 0.12.4
public_privacy_review: PASS — public copy removes personal author identity and replaces known site-specific example identifiers with non-identifying reserved examples
```

## 0. MVP COMPLETE時点の現在状態 — 2026-10-05

本書のコンセプト、Layer境界、Post-MVP拡張方針は維持したまま、MVPは**完成済み**となった。

- package: `0.12.4`
- R12 `full` Pipeline Live-fetch E2E: **PASS / CLOSED**
- M5 MVP Hardening / Completion Gate: **PASS / CLOSED**
- MVP COMPLETE: **YES**
- `.gitignore`全面監査: **PASS**
- Source公開: **依頼者の明示許可が出るまでHOLD**
- 公開対象として準備する範囲: documentation / public images / README / license-related documents

MVP完成によってコンセプトの終点が変わったわけではない。MVPは、Observed DataをSource固有意味を保持したまま取得・統合・分析・可視化できる監査可能なCoreを閉じた状態である。Post-MVPでは、このCoreを壊さず、新Source / Website Intelligence / Change / Intervention / Modeling / Experiment / Causal / Decision Support / Semantic / LLM / Scale等を段階追加する。

### MVP completionで実証された経路

```text
GA4 live Extraction ─> GA4 Processing ─┐
                                       ├─> Integration Extraction
GSC live Extraction ─> GSC Processing ─┘
                                       -> Integration Processing
                                       -> Integration Analysis
                                       -> Visualization
```

`full` Live-fetch E2Eは実データで完走し、Artifact existence/hash/lineage、requested range、GSC resolved coverage、portable manifest、secret marker、Visualization contract/hash、generated CSV parseまで監査された。

R12実行過程では、GA4 credential相対pathの構造移動後不整合と、GA4 `partial_success` ArtifactをPipelineが非0 exitだけで失敗扱いするadapter契約漏れが発見され、後者はv0.12.3で修正された。M5ではReact primary dashboardのrelative delta percent表示がratioを100倍せず表示するcorrectness bugを検出し、v0.12.4で修正した。これらはMVP Coreの境界変更ではなく、既存契約を実運用で満たすためのhardeningである。

## 1. このシステムは何か

Generic Web Intelligence & Decision Platformは、Webに関するObserved Dataを取得・正規化・統合・分析・可視化し、人間の意思決定を支える監査可能なローカル基盤である。

現在のMVPはGA4とGoogle Search Consoleを主Sourceとするが、設計上はGA4/GSC専用製品ではない。

```text
Observed Source
  -> Source Extraction
  -> Source Processing
  -> Source Analysis (optional independent branch)
           \
            -> Integration Extraction
            -> Integration Processing
            -> Integration Analysis
            -> Visualization
```

VisualizationはIntegration専用ではなく、GA4 Analysis / GSC Analysis / Integration Analysis / Generic Result Artifactを表示可能な独立Presentation Layerである。

## 2. MVPで解決すること

- GA4のsite behaviorとGSCのsearch visibilityを同一page/date等の安全な軸で扱う
- Source固有の意味を失わず正規化する
- losslessなIntegration Extractionでsource-only observationを残す
- use caseに応じたperiod/grain/metricをIntegration Processingで選ぶ
- Generic Analysis Engineで比較・trend・ranking・contribution・qualityを生成する
- Resultを再分析せずVisualizationへ投影する
- Artifact/Manifest/hash/lineageで結果を追跡できる

## 3. 「統合」の意味

統合は「全部同じ表にして意味を潰す」ことではない。

GA4とGSCは別の観測系であるため、Source factsを保持し、join state / observation state / metric semanticsを残したまま接続する。GSC queryからGA4 conversionを直接帰属させることはしない。

## 4. 利用形態

1. Atomic CLIで各Layerを個別実行
2. `gwidp pipeline`で定型経路を一括実行
3. 既存Artifactから途中再開
4. Dashboard/CSV/JSONを人間や外部分析Workflowへ渡す

### Pipelineの現在状態

- package: `0.12.4`
- state: **DONE / CLOSED**
- detail: Pipeline closure, R12 Live E2E, M5 hardening, and MVP Completion Gate are complete.
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


## 5. 最終形

Post-MVPではWebsite Snapshot、Content、Site Structure、Change/Intervention、Experiment、Prediction、Causal Evidence、Business KPI等を同一の監査可能な基盤へ段階追加する。

SystemはAction CandidateやEvidenceを提示できるが、最終判断そのものを勝手に代替しない。


## 保存付録 — Concept Overview v1.0 原本全文

この付録は、更新前の原本文書に存在した情報を不可逆に失わないための保存領域である。
**現行仕様の判断には本文前半を使用し、この付録は履歴・判断根拠・旧仕様の追跡に使用する。**
原文は内容を要約せず、そのまま保存している。

- preserved_source_sha256: `01695c6d718f0d171a52bd5d065ae3515cb74a6accd84954784c7a456b50bc5b`

````markdown
# Concept Overview

```yaml
project_initiated_at: 2026-09-23 17:35:34 JST
created_at: 2026-09-27 JST
updated_at: 2026-09-27 JST
author: Nazuna Research
doc_ver: v1.0
note: Concept Overview — MVP
```

---

# 1. これは何か
本システムは、GA4とGoogle Search Consoleのデータを取得・整形・統合し、
検索流入とサイト内行動を横断して分析するための汎用ローカル分析基盤である。
単なるGA4 Exporterでも、単なるGSC Exporterでもない。

```text
GA4 + GSC
   ↓
Processing
   ↓
Canonical Integration
   ↓
Generic Integrated Analysis
   ↓
Static HTML / CSV / JSON
```

# 2. 何を解決するか
GSCは主に、query / page / impressions / clicks / CTR / positionを見る。
GA4は主に、sessions / landing page / page views / engagement / events / key eventsを見る。
本システムは、この2つをPage / Date等の共通軸で統合する。

# 3. MVPで出来ること
```text
GA4 Extraction / Processing
GSC Extraction / Processing
Canonical Integration
Integrated Page Analysis
Query × Page Analysis
28日 / 90日比較
Metric-safe aggregation
Opportunity candidate抽出
Quality / limitationの継承
Static HTML可視化
CSV / JSON出力
完全Lineage
```

# 4. Integrationとは何か
Integrationは、2つのSourceを雑に1枚のCSVへ連結する処理ではない。
Source固有の意味を残したまま、共通KeyとGrainを持つCanonical Datasetを作る。

中心Key:
```text
site_key
normalized_page_key
date
query
search_type
```

Canonical Layerは整形済み・統合済みだが、
まだSEO/CVR固有の評価やRecommendationは入れない。

# 5. QueryとGA4の関係
GSC QueryをGA4 SessionやConversionへ直接帰属させることはしない。

```text
GSC query
→ GSC page
→ normalized_page_key
→ GA4 page-level behavior
```

したがって「このQueryがこのConversionを生んだ」とは断定しない。

# 6. Dateの扱い
GA4とGSCでは時間の意味が完全には同一ではない。
MVPではDate Labelで統合し、Source側のTime SemanticsをManifestへ残す。

# 7. Qualityの扱い
QualityはLogだけではなくDataとして下流へ伝播する。

```text
coverage
thresholding
truncation
GSC exhaustiveness
unavailable metrics
source warnings
```

また、ZERO / UNKNOWN / UNAVAILABLE / NOT_OBSERVEDを区別する。
取得できなかった値を勝手に0へしない。

# 8. Analysisの考え方
Integrated AnalysisはSEO専用コードへ固定しない。
Dataset / Grain / Filter / Metric / Period / Rank / ThresholdをConfigで指定する。

代表例:
```text
Page Performance
Query × Page Performance
28d current vs previous
90d current vs previous
high impression / low CTR候補
Search visibility × GA4 engagement
Search visibility × Key Events
```

ThresholdはCoreへHardcodeせず、Analysis Specで指定する。

# 9. MVPで出来ないこと
MVPだけでは以下を自動的には判断できない。

```text
Title文言がなぜ弱いか
本文がQuery Intentと合うか
CTA文言が悪いか
内部リンク構造が悪いか
施策Aで何%改善するか
因果効果
将来予測
サイト全文のSemantic Analysis
```

これらはPost-MVPのSite Crawl / ML / Causal / LLM領域。

# 10. Raw Data Warehouseではない
GA4 Data APIはBigQuery Raw Event Exportではない。
GSC Search Analytics APIも検索Raw Logの完全Exportではない。
本システムは「APIが提供するReporting Data」を正しく保存・利用する。

# 11. BI Toolではない
Dashboard ServerやDatabase Serverを必須にしない。
MVPの可視化はStatic HTMLを基本とする。

```text
report.html
+
CSV / JSON
```

Browserで開けば確認できる程度を狙う。

# 12. HTMLは本体ではない
本体はCanonical / Analysis Artifact。
HTMLはPresentation Layerである。
そのためVisualizationだけ変更して再生成できる。

# 13. 再現性と監査
各LayerはRun ID / Config Hash / Parent Manifest Hashを持つ。

```text
Extraction
→ Processing
→ Integration
→ Analysis
→ Visualization
```

数値がどのSource Runから来たか追跡できることを重視する。

# 14. 汎用性
特定のSite / Client / Event / CTA / ThresholdをCore Pythonへ埋め込まない。
別SiteではConfigとProperty / Site指定を変えて再利用する。

# 15. MVPの最終操作イメージ
最終的には概ね以下を目指す。

```bash
analytics-pipeline --config config/project.yaml
```

出力:
```text
output/extraction/
output/processing/
output/integration/
output/analysis/
output/visualization/
output/pipeline/
```

# 16. MVP完成の定義
MVPは、
**GA4とGSCを取得・整形・Canonical Integrationし、
任意のAnalysis SpecでIntegrated Analysisを実行し、
CSV / JSON / Static HTMLとして出力できる状態**
を完成とする。

# 17. MVP後
将来拡張候補:

```text
Website Snapshot / Site Crawl
HTML / Content Analysis
Internal Link Graph
Machine Learning
Experiment / Causal
Expected Value Decision Support
LLM Platform Hook
Embedding / Semantic Search
```

ただし、これらはMVP完成条件ではない。

# 18. 一言で言うと
このMVPは、
**GA4 + GSCを中心に、検索performanceとサイト内behaviorを
一つの監査可能なCanonical Data Modelへ統合し、
汎用Analysisと簡易可視化まで行うWeb分析基盤**
である。
````
