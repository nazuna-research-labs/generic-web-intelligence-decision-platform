# 汎用Webインテリジェンス意思決定プラットフォーム ロードマップ概要

```yaml
document_id: roadmap_summary
document_type: summary
doc_ver: v1.1
language: ja
project_name: Generic Web Intelligence & Decision Platform
project_initiated_at: 2026-09-23 17:35:34 JST
created_at: 2026-10-03 08:14:10 JST
updated_at: 2026-10-05 22:57 JST
author: Nazuna Research
status: active
scope: summary of master roadmap
note: 旧ファイルの別プロジェクト本文は採用せず、本Project専用に新規作成
source_document: master_roadmap_ja_v3.1.md
previous_public_version: roadmap_summary_ja_v1.0.md
mvp_completion_state: COMPLETE
current_package: 0.12.4
public_privacy_review: PASS — public copy removes personal author identity and replaces known site-specific example identifiers with non-identifying reserved examples
```

## 0. MVP COMPLETE Update

2026-10-05時点でMVPは完成済み。

```text
GA4/GSC Source Pipeline      DONE / HARDENED
Integration M1-M3           DONE / CLOSED
Visualization M4            DONE / CLOSED
Pipeline                    DONE / CLOSED
R12 full Live E2E           PASS / CLOSED
M5 Hardening                PASS / CLOSED
.gitignore audit            PASS / CLOSED
MVP                         COMPLETE
package                     0.12.4
Source public release       HOLD / requester permission required
```

本v1.1更新でpublic docsのlossless更新とprivacy/specific-identifier auditを完了対象とする。次の運用Phaseは、Sourceを含めないdocs/public images/README/license-related documentsのみのGitHub publication。Post-MVPはMVP Coreを壊さず、Master Roadmapの依存順に追加する。

## 1. この文書の役割

この文書は`master_roadmap_ja_v3.1.md`の概要版である。詳細Roadmapを置換しない。

## 2. 現在地

```text
GA4 Source Pipeline         DONE / HARDENED
GSC Source Pipeline         DONE / HARDENED
Integration Extraction     DONE
Integration Processing     DONE
Integration Analysis       DONE
Visualization              DONE / CLOSED
Repo structure migration   DONE
Pipeline                   DONE / CLOSED
MVP Hardening / Completion DONE / CLOSED
```

package: `0.12.4`

## 3. 現在のMVPフロー

```text
GA4 Extraction ─> GA4 Processing ─┐
                                  ├─> Integration Extraction
GSC Extraction ─> GSC Processing ─┘
                                  -> Integration Processing
                                  -> Integration Analysis
                                  -> Visualization
```

GA4 / GSC単体では`Extraction -> Processing -> Analysis -> Visualization`も可能。

## 4. Pipeline

6 profiles:

- `full`
- `full_from_extraction`
- `sources`
- `ga4`
- `gsc`
- `integration`

`full`は認証/API失敗時のcheckpoint/resumeと手動Extraction Artifact差し込みを持つ設計。

## 5. MVP closureまで

1. R12 `full` Live-fetch E2E — DONE
2. M5 data/lineage/determinism/security hardening — DONE
3. percentage format correctness — DONE / v0.12.4
4. `.gitignore`全面監査 — DONE
5. MVP COMPLETE — YES
6. public docs lossless update / privacy audit — CURRENT
7. Source public release — HOLD until requester permission

## 6. VisualizationのMVP後候補

- KPI grid 4+3案
- Quality/Coverage/Lineage raw JSON accordion
- count Y-axis integer ticks
- technical metadata secondary化

percentage ratio表示はpolishではなくM5 correctness。

## 7. Post-MVP

```text
Website Intelligence
-> Change / Intervention
-> Modeling / ML
-> Experiment
-> Causal
-> Decision Support
-> Semantic / Embedding
-> Grounded LLM
-> Additional Sources
-> Scale / Storage / Ops
-> Productization
```

詳細な依存関係・Exit CriteriaはMaster Roadmapを参照。

## 8. Phase別の見方

| 区分 | 状態 | 意味 |
|---|---|---|
| Source GA4/GSC | 完了 | Source単体で取得〜分析可能 |
| Integration M1-M3 | 完了 | lossless統合〜Generic Analysis |
| Visualization M4 | 完了 | Generic Presentation Engine closure |
| Pipeline | 完了 | 6 profile + resume/recovery + Live E2E |
| M5 | 完了 | hardening / completion gate PASS |
| Post-MVP | 未着手 | Website〜Decision Platform拡張 |

## 9. 主要な設計固定点

- Integration Extractionはlossless
- Integration ProcessingがUse Case selectionを所有
- AnalysisはSource/use case branchを増殖させない
- Visualizationは再分析しない
- Prediction != Causal
- LLMはoptionalかつgrounded
- Human-in-the-loopを維持

## 9.1 v1.0運用原則の保持

v1.0で明示されていた「Docsはlossless updateを原則化」は、MVP completion後も維持する。

```text
- Docsはlossless updateを原則化
```

## 10. 現在のRisk / Watch Item

- Pipeline / Live E2E / M5 / `.gitignore`はCLOSED
- Source public releaseは依頼者許可までHOLD
- public docsはlossless updateとspecific-identifier/privacy auditを必須化
- Post-MVPをMVP completionへ逆混入させない

