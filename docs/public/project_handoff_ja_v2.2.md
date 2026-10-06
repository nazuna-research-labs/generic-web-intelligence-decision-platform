# 汎用Webインテリジェンス意思決定プラットフォーム プロジェクト引き継ぎ書

```yaml
document_id: project_handoff
document_type: handoff
doc_ver: v2.2
language: ja
project_name: Generic Web Intelligence & Decision Platform
project_initiated_at: 2026-09-23 17:35:34 JST
created_at: 2026-09-30 JST
updated_at: 2026-10-05 22:57 JST
author: Nazuna Research
status: active
scope: project-specific complete continuation handoff
note: v2.1までをlossless保持し、MVP COMPLETE後のcold-start state、publication boundary、exact next actionへ更新
previous_note: v2.0を保持し、Cold-start Handoff Capsule/Audit、docs運用、exact next actionを補強
preserved_source_sha256: 1090831706864d749783614fb9d03044ab94e782f962829338b4ad5b408a46de
supersedes: GENERIC_WEB_INTELLIGENCE_PLATFORM_MVP_HANDOFF_INTEGRATION_TO_VISUALIZATION_v1.1_20260930.md
cold_start_audit: PASS if all mandatory continuation fields below remain true at runtime
previous_public_version: project_handoff_ja_v2.1.md
mvp_completion_state: COMPLETE
current_package: 0.12.4
public_privacy_review: PASS — public copy removes personal author identity and replaces known site-specific example identifiers with non-identifying reserved examples
```

## 0.0 MVP COMPLETE Cold-start Capsule — 2026-10-05

この節が現在の最優先Continuation Stateである。後続の2026-10-03以前の`NEXT`、Pipeline pending、M5 pending等は履歴として保持するが、現在Actionとして実行しない。

### Current state

- Formal project: `Generic Web Intelligence & Decision Platform`
- distribution: `generic-web-intelligence-decision-platform`
- namespace: `web_intelligence`
- current package: **`0.12.4`**
- M1 Integration Extraction: DONE
- M2 Integration Processing: DONE
- M3 Integration Analysis: DONE
- M4 Visualization: DONE / CLOSED
- Pipeline 6 profiles: DONE / CLOSED
- R12 `full` Live-fetch E2E: **PASS / CLOSED**
- M5 MVP Hardening / Completion Gate: **PASS / CLOSED**
- MVP COMPLETE: **YES**
- `.gitignore` full audit: **PASS / CLOSED**
- Source public release: **NOT AUTHORIZED / HOLD**
- Current next: **public docs lossless update + privacy/specific-identifier audit**, then docs/images/README/license-only publication workflow

### R12 final evidence

```text
package: 0.12.3 at R12 execution
requested range: 500daysAgo -> yesterday
Pipeline run: output/pipeline/full/pipeline_full_20261005T035611324478Z
Manifest: output/pipeline/full/pipeline_full_20261005T035611324478Z/pipeline_manifest.json
GSC resolved range: 2025-05-22 -> 2026-10-03 (500 days)
8-stage full profile: PASS
Artifact/hash/lineage: PASS
portability/secret marker scan: PASS
Visualization final contract/hash: PASS
generated Artifact CSV parse: PASS (223 files)
```

R12の途中で以下を修正した。

- GA4 `profiles.yaml` / exampleのcredential relative pathをrepository-root `credentials/`へ正規化。
- GA4 CLIの`partial_success = exit 1`契約は変更せず、Pipelineが「新規GA4 Artifactが正確に1個あり、そのmanifest statusが`partial_success`」の場合だけ非0を受理するadapterをv0.12.3で実装。failed/missing/invalidはfailureのまま。
- v0.12.2 failed R12 checkpointはimmutable historical evidenceとして保持し、version差のためresumeしなかった。

### M5 final evidence

```text
package: 0.12.4
R12 source run: output/pipeline/full/pipeline_full_20261005T035611324478Z
M5 validation: output/pipeline/full_from_extraction/pipeline_full_from_extraction_20261005T105537280277Z
MVP gate: PASS
```

M5ではReact primary `relative_delta`のratio percentage表示を修正した。内部`-0.3105`は`-31.05%`として表示する。M5 deep auditで一度`ga4_process`の`processor_version`をpackage versionと誤比較するaudit bugが発生したが、これはproduct Artifact不良ではなくaudit-side bugで、stage-specific provenance checkへ修正後に同じ成功済みM5 runをreuseして全gateをPASSした。

### MVP closure / `.gitignore`

MVP closure reportで以下を確定した。

- package `0.12.4`
- MVP COMPLETE: YES
- M5 gate: PASS
- `.gitignore` audit: PASS
- `docs/public`: closure時点ではUNCHANGED
- Source push: NOT AUTHORIZED

`.gitignore`では`config/system/**` / examples / required public build artifactsを誤ignoreせず、actual user config / credentials / output / backups / cache / OS / IDE / local one-shot maintenance scripts等を公開対象から分離する。

### Public distribution boundary

現時点の公開準備対象は、public documentation、public images、README、license/notice/terms。Source codeは依頼者の明示許可が出るまで公開しない。

公開docsには個人情報、顧客情報、企業固有情報、実Property ID、実Site URL、local absolute path、credential本文等を含めない。説明に実サイト例が必要な箇所はreserved/non-identifying exampleへ置換し、意味論だけ保持する。

### Do not reopen

MVP completionによって以下を再設計しない。

- Extraction lossless
- Processing config-driven
- Analysis generic
- Visualization no reanalysis
- Pipeline orchestration only
- Artifact immutability/hash/lineage
- ZERO / NOT_OBSERVED / UNAVAILABLE / UNKNOWN / NULL separation
- query attribution boundary / `NON_ADDITIVE_CONTEXT`
- non-additive metric safety
- Source Analysis parallel branch
- Prediction != Causal Effect
- Detected Change != Intervention
- LLM optional / grounded
- human final decision

### v2.1時点のContinuation History

v2.1では次の未完了順序が記録されていた。現在はclosure済みだが、引き継ぎ履歴として保持する。

```text
2. non-fetch Pipeline E2E / resume behaviorをclosure
4. M5 full hardening
7. MVP closure
8. Post-MVPへ移行
Exact command/status-dependent instruction:
```

### Exact next action

本v2.2更新でdocs/public全12文書のlossless更新とpublic privacy/specific-identifier auditを完了対象とする。次の運用Phaseは**docs-only publication**。

1. repo rootへ本v2.2を含む最新版docsを適用し、READMEのversioned-doc linkを同期する。
2. `assets/images/architecture_overview_ja.png`がmetadata除去済みであること、`LICENSE.txt` / `NOTICE.md` / `TERMS_OF_USE.md`がrootにあることを確認する。
3. Git diff / tracked file listを確認し、Source code、credentials、actual user config、output、backup等がpush対象に入っていないことを再確認する。
4. `docs/public/`、public images、README、license-related documentsだけをGitHubへpushする。
5. Source codeは依頼者の明示許可が出るまでpushしない。
6. Post-MVPは別途優先順位を決めて開始する。

## 0. この文書の目的

この文書は、別スレッド・別担当LLM・別実装者へ本プロジェクトを**意味欠損なく継続**させるためのProject固有Handoffである。

要約資料ではない。現在状態、確定Architecture、実装済みArtifact、失敗履歴、未完了Gate、次Action、禁止事項を保持する。

## 1. Source of Truth / 優先順位

1. ユーザーの最新明示指示
2. 現在のrepo実態 / source / config / tests / Artifact
3. `system_requirements_ja_v3.1.md`
4. `master_roadmap_ja_v3.1.md`
5. 本Handoff
6. 保存旧版 / older docs

ARGD/DAGD等、thread initialization / governance用の外部文書が存在しても、target architectureやrepo docsへ混入させない。

## 2. 現在のプロジェクト名・構造

- Formal name: `Generic Web Intelligence & Decision Platform`
- Distribution: `generic-web-intelligence-decision-platform`
- Python namespace: `web_intelligence`
- repo folder: `generic_web_intelligence_decision_platform`

Source layout:

```text
src/web_intelligence/
├── common/
├── extraction/{ga4,gsc}/
├── processing/{ga4,gsc}/
├── analysis/{ga4,gsc}/
├── integration/{extraction,processing,analysis}/
├── visualization/
└── pipeline/
```

Config:

```text
config/
├── README.md
├── user/    # user-editable
└── system/  # contracts/defaults/specs; normally do not edit
```

## 3. 現在の完成状態

### 3.1 Source Pipeline

GA4 / GSCともExtraction / Processing / AnalysisはDONE/HARDENED。

### 3.2 Integration

- M1 Integration Extraction: DONE
- M2 Integration Processing: DONE
- M3 Integration Analysis: DONE

### 3.3 Visualization

M4はユーザーmanual reviewを含めCLOSED。React primary / static auditのGeneric Presentation Engine。4 input系を確認済み。

### 3.4 Structure migration

v0.11.0でnamespace・src・tests・configを正規化。194 tests / Ruff / 10 Atomic CLI smokeを通過。旧`analytics_exporter` / `search_console` / old repo pathをactive treeから除去。

### 3.5 Pipeline

### Pipelineの現在状態

- package: `0.12.4`
- state: **DONE / CLOSED**
- detail: Pipeline/R12/M5/MVP Completion Gateまで完了。
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


## 4. Pipeline 6 Profiles

```text
full
  GA4/GSC Extraction
  -> Source Processing
  -> Integration Extraction
  -> Integration Processing
  -> Integration Analysis
  -> Visualization

full_from_extraction
  existing GA4/GSC Extraction Artifacts
  -> Source Processing
  -> Integration...
  -> Visualization

sources
  GA4/GSC Extraction
  -> each Processing
  -> each Analysis

ga4
  GA4 Extraction -> Processing -> Analysis -> Visualization

gsc
  GSC Extraction -> Processing -> Analysis -> Visualization

integration
  existing GA4/GSC Processing Artifacts
  -> Integration Extraction -> Processing -> Analysis -> Visualization
```

`full`にGA4/GSC Source Analysisを挟まないことはcontract。

## 5. Resume / Recovery設計

`full`で認証要求・API失敗等が起きても、最初からやり直さない。

- Pipeline run directoryとManifestをcheckpointとして残す
- failed stageを記録
- 成功済みArtifactはmanifest hashを検証して`reused`
- 失敗Stageは再試行
- 手動で生成したGA4/GSC Extraction Artifactを`supplied`として差し込める
- attempt historyをManifestに保持
- `full_from_extraction`は最初から認証を手動管理する保険経路

代表操作:

```bash
uv run gwidp pipeline full --resume <PIPELINE_RUN>

uv run gwidp pipeline full \
  --resume <PIPELINE_RUN> \
  --gsc-extraction <GSC_EXTRACTION_ARTIFACT>
```

## 6. 実装運用 — `.sh` 一括実装

このプロジェクトでは、大きな変更はrepo rootから実行するone-shot / resume `.sh` を作って適用する方式を多用している。

基本形:

```text
Preflight
Backup
Patch / Migration
Lock update (必要時)
Ruff autofix / format
Compile
Targeted tests
Full pytest
Artifact / contract gate
Cleanup
[SUCCESS]
```

原則:

- `.venv` activateを前提にせず `uv run`
- shell outputは`== Step ==`, `[FAIL]`, `[SUCCESS]`を明示
- 失敗したからといって無意味にrollbackしない
- partial stateを検査し、原因を直したresume scriptで続行
- source migrationとfinal gateを分離できるようにする
- live GA4/GSC fetchは最後にまとめ、通常の修正で何度も実行しない
- repo current stateをtruthとしてanchor brittleな置換を避ける

共通運用原則は `development_workflow_and_handoff_guidelines_ja_v1.2.md` を参照。

## 7. 重要Data Contract

### Page

`site_key + normalized_page_key`を主要cross-source identityとする。query/fragmentはpage identityから除外するが、source factsとして必要な情報は保持する。

### Date

Processing以降は`YYYY-MM-DD` canonical date。

### Missing

`ZERO / NOT_OBSERVED / UNAVAILABLE / UNKNOWN / NULL`を別扱い。

### Query boundary

GSC queryをGA4 session/key event/conversionへexact attributionしない。

### Persistent path

`/Users/...`等のmachine-specific absolute pathをManifestへ保存しない。

## 8. Reference Artifacts

Primary/Controlとして以下を使ってきた。

- A: GA4 long 2024-09-27 -> 2026-09-26（730日）
- B: GA4 control 2025-05-26 -> 2026-09-26（489日）
- C: GSC requested 2025-05-26 -> 2026-09-26、observed through 2026-09-24（487日）

A+Cがprimary、B+Cがperiod-control。GSC reporting lagを0で埋めない。

## 9. Visualization重要事項

- Integration専用rendererにしない
- React primary / Static secondary
- upstreamのResult rowを再分析しない
- `dashboard.html` + `dashboard_data/`のroot 2 entry
- original input Artifact ZIPをdownload対象にする
- time series: axes/ticks/unit/date/crosshair/drag zoom/reset
- Quality/Coverage/Lineageを可視化
- M5でpercentage format correctnessを検証済み / v0.12.4 PASS
- Post-MVP候補: KPI 4+3, accordion, integer count ticks, metadata secondary化

## 10. 現在の継続順

MVP実装・hardeningは完了済み。現在は次の順で継続する。

1. `docs/public/`全12文書をlossless更新
2. public docs / images / README / license-related documentsのprivacy / specific-identifier audit
3. READMEのversioned-doc導線を最終同期
4. Sourceを含めず、許可されたdocs/images/README/license filesだけをGitHubへpush
5. 依頼者の許可がない限りSource public releaseは行わない
6. Post-MVP P1-P11は優先順位を決めて別Phaseとして開始

## 11. 絶対に戻してはいけない設計

- VisualizationをIntegration専用に戻す
- Integration Extractionでuse case trimする
- query->GA4 exact attribution
- missing stateを0へ潰す
- source-specific branchをGeneric Analysis/Visualizationへ増殖
- absolute local pathをPersistent Artifactへ書く
- READMEやSummaryを詳細設計の代替にする
- Handoff更新時に旧情報を勝手に要約して欠損させる



## 0.1 Cold-start Resume Capsule

この節は、元スレッドを読めない新しい担当者が最初に読む即時再開用summaryである。詳細仕様を置換しない。

### Current package / Pipeline

- package: `0.12.4`
- Pipeline: **DONE / CLOSED**
- detail: Package is 0.12.4; R12/M5/MVP closure completed.
- Source GA4/GSC: Extraction / Processing / Analysis DONE/HARDENED
- Integration M1/M2/M3: DONE
- Visualization M4: CLOSED
- Structure normalization R1-R6: CLOSED
- Pipeline public profiles: `full`, `full_from_extraction`, `sources`, `ga4`, `gsc`, `integration`

### Current docs

このdocs hardening後の主要継続文書:

```text
project_handoff_ja_v2.2.md
system_requirements_ja_v3.1.md
master_roadmap_ja_v3.1.md
concept_overview_ja_v2.1.md
visualization_generic_presentation_engine_design_ja_v2.1.md
development_workflow_and_handoff_guidelines_ja_v1.2.md
```

操作系:

```text
user_manual_ja_v1.2.md
dashboard_manual_ja_v1.2.md
```

全機能/加工:

```text
system_functions_and_data_processing_details_ja_v1.2.md
system_functions_and_data_processing_summary_ja_v1.2.md
```

### Immediate next action

現在のexact next actionは**public docs update / privacy audit / docs-only publication preparation**。

HISTORICAL: v0.12.2時点ではresume/final gate結果が未確定で、`resume_pipeline_v0_12_2_after_ruff_sim102.sh`がnextだった。この旧commandは履歴としてのみ保持し、現在は実行しない。

### Source of Truth order

1. ユーザー最新明示指示
2. current repo source/config/tests/Artifact/log
3. system requirements
4. master roadmap
5. project handoff
6. design/concept/reference
7. preserved historical appendices

### Never break

- Integration Extractionはlossless/full-outer相当。use-case trimしない。
- ZERO / NOT_OBSERVED / UNAVAILABLE / UNKNOWN / NULLを潰さない。
- GSC queryをGA4 session/conversion/key eventへexact attributionしない。
- queryに付くGA4 contextはNON_ADDITIVE_CONTEXT。
- search_type/queryでCartesian multiplicationを起こさない。
- Visualizationは再分析しない。
- PipelineはLayer logicを再実装しない。
- Persistent Artifactへabsolute local path/secretを残さない。
- docs updateを暗黙要約にしない。

### Live fetch policy

構造/Pipeline修正中にGA4/GSC live fetchを何度も実行しない。local/unit/contract/non-fetch E2Eで閉じられるものを先に閉じ、最後にlive E2Eする。

### `.sh` implementation policy

repo rootからone-shot/resume `.sh`を実行し、Preflight -> Backup -> Patch -> Test/Ruff -> Artifact Gate -> SUCCESSの順で進める。失敗時はpartial stateを診断し、無意味なrollbackをせずresumeする。

## 0.2 Cold-start Handoff Audit Result

このv2.1では次を本文/参照docsと照合した。

| Audit item | Result | Where to resolve |
|---|---|---|
| current version | PASS | この節 / pyproject.toml |
| repo structure | PASS | §2 |
| milestone completion | PASS | §3 / Roadmap |
| incomplete state | PASS | §3.5 / この節 |
| last known failure / unverified state | PASS | Pipeline status / next action |
| exact next command | PASS | この節 |
| Source of Truth | PASS | §1 / この節 |
| architecture contracts | PASS | §7/§11 + Requirements |
| data semantics | PASS | §7 + Requirements/Reference |
| Artifact/Manifest | PASS | §5/§7 + Reference |
| 6 Pipeline profiles | PASS | §4 |
| resume/recovery | PASS | §5 |
| live fetch policy | PASS | §6 / この節 |
| `.sh` workflow | PASS | §6 / Guideline |
| MVP remaining work | PASS | §10 / Roadmap |
| Post-MVP plan | PASS | Roadmap / Requirements |
| README/docs policy | PASS | Guideline |
| known dashboard correctness/polish items | PASS | §9 / Roadmap |

この表のPASSは「情報が配置され、cold-start readerが参照先を一意に辿れる」という意味。current repo/logが変わった場合は再監査する。

## 保存付録 — MVP Handoff v1.1 原本全文

この付録は、更新前の原本文書に存在した情報を不可逆に失わないための保存領域である。
**現行仕様の判断には本文前半を使用し、この付録は履歴・判断根拠・旧仕様の追跡に使用する。**
原文は内容を要約せず、そのまま保存している。

- preserved_source_sha256: `1090831706864d749783614fb9d03044ab94e782f962829338b4ad5b408a46de`

````markdown
# Handoff

```yaml
project_initiated_at: 2026-09-23 17:35:34 JST
created_at: 2026-09-30 JST
updated_at: 2026-09-30 JST
author: Nazuna Research
doc_ver: v1.1
note: Integration Extraction → Integration Processing → Integration Analysis → Visualization, MVP完成までの完全引き継ぎ書
対象: 次スレッドで「現在の最新repo」と「Integration〜Visualization要件定義書」を受け取る担当LLM / 実装者
```

---

# 0. この文書の目的

この文書は、現在のスレッドで確定した設計判断・実装前提・既知の修正履歴・検証済みArtifact・MVP完成条件・実装順序・禁止事項を、
**次スレッドへ1mmも意味を落とさず引き継ぐ**ための文書である。

次スレッドでは、ユーザーが以下をアップロードする前提。

```text
1. 現在の最新repo
   - Source Pipelineまでの最新実装
   - 最終確認時点では package v0.7.3 系
   - GA4 / GSC Extraction / Processing / Analysis が完成状態

2. 要件定義書
   GENERIC_WEB_INTELLIGENCE_PLATFORM_INTEGRATION_TO_VISUALIZATION_要件定義書_v1.0_20260927.md

3. 本引き継ぎ書
```

必要に応じて、Integration live validation用として以下の長期Artifactも追加投入する。

```text
Dataset A: GA4 約2年
Dataset B: GA4 GSC同期間版
Dataset C: GSC 約16か月
```

この文書だけで実装してはいけない。
**次スレッドで必ず最新repoを実際に検査し、要件定義書を読み、その2つをSource of Truthとして実装を開始すること。**

---

# 1. Source of Truth / 優先順位

今後、資料間に矛盾が見つかった場合の優先順位を固定する。

```text
優先度 1:
  ユーザーが次スレッド以降で明示した最新の指示

優先度 2:
  GENERIC_WEB_INTELLIGENCE_PLATFORM_INTEGRATION_TO_VISUALIZATION_要件定義書_v1.0_20260927.md

優先度 3:
  次スレッドでアップロードされる「現在の最新repo」の実装状態
  ※ 現在コードの事実確認に使う
  ※ repoに古い設計が残っていても、要件定義書と衝突する場合は要件側を優先して直す

優先度 4:
  本引き継ぎ書
  ※ 会話上の経緯、確定判断、検証結果、注意点を補完する

優先度 5:
  既存の古い全体設計書 / Roadmap / 要件定義書
```

重要:

`Integration〜Visualization 要件定義書 v1.0` は、
Integration / Generic Analysis / Visualizationに関する既存設計のうち、
同文書と矛盾する箇所を上書きする、と明示済み。

したがって、古い文書に以下のような旧構造が残っていても、そのまま復活させてはいけない。

```text
旧:
  Canonical Integration
      ↓
  Generic Integrated Analysis
      ↓
  Visualization

現在:
  Integration Extraction
      ↓
  Integration Processing
      ↓
  Integration Analysis
      ↓
  Visualization
```

---

# 2. ユーザーの最重要意図

このシステムは、最終的には

> 定量化されたデータにより、意思決定を補助するシステム

である。

現在のMVPは、

```text
GA4
GSC
  ↓
Source Extraction
  ↓
Source Processing
  ↓
Source単体Analysisも可能
  ↓
Integration Extraction
  ↓
Integration Processing
  ↓
Integration Analysis
  ↓
Visualization
  ↓
人間の意思決定を補助
```

という基盤を完成させる段階。

MVP完成後は、分析結果やサイトURL等をLLMへ渡して、
解釈・要約・示唆出しの補助に利用可能。

さらにPost-MVPでは、

```text
サイト構造データ化
ページ本文 / コンテンツ取得
内部リンク構造
サイトSnapshot
機械学習
予測
介入履歴
実験管理
因果推論
LLM組み込み
改善提案
施策期待値シミュレーション
定量 × 定性の統合分析
Decision Support
```

へ拡張する構想。

ただし、**これらPost-MVP要素を今のMVP Coreへ混ぜてはいけない。**

---

# 3. 現在の完成状態 — Source Pipeline

Integration着手前に、GA4 / GSCは以下まで完成扱いとする。

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

最後に確認したコード状態:

```text
Package:
  v0.7.3 系

Tests:
  132 passed

Ruff:
  All checks passed!
```

ただし次スレッドでは、
アップロードされた最新repoに対して必ず再確認すること。

推奨初動:

```bash
uv run --offline --locked pytest -q
uv run --offline --locked ruff check .
```

Versionも実際の `pyproject.toml` を読んで確認し、
会話上の `v0.7.3` を盲信しないこと。

---

# 4. Source Pipelineで既に潰した重要問題

Integration実装者は、
以下を再び別Layerで「吸収」しようとしてはいけない。

Source Processing側で既に契約化した。

---

## 4.1 GSCの4件

GSC v0.7.1系で修正・実Artifact確認済み。

### 4.1.1 requested / resolved と actual observed coverage

要求期間が30日でも、
Googleから実際に返った日付が28日なら、

```text
requested = 30
observed = 28
coverage = PARTIAL
```

とする。

存在しない最新日をFULL扱いしない。

---

## 4.1.2 persistent lineage path

永続Artifact内のpathはproject-relative。

禁止:

```text
/Users/...
/home/...
C:\Users\...
```

Runtime logでabsolute pathを表示すること自体は可。
Persistent manifestへは入れない。

---

## 4.1.3 UNAVAILABLE != ZERO

```text
UNAVAILABLE
NOT_OBSERVED
UNKNOWN
ZERO
```

を混同しない。

特に、

```text
UNAVAILABLE -> 0
NOT_OBSERVED -> 0
```

は禁止。

---

## 4.1.4 Position reconciliation tolerance

浮動小数の極小差でfalse mismatchを起こさない。

例:

```text
MATCH_WITHIN_TOLERANCE
```

を使用。

raw deltaは保持。

---

# 5. GA4で既に潰した重要問題

## 5.1 persistent lineage path

GA4 Processing:

```text
source_run_directory
```

GA4 Analysis:

```text
source_processing_directory
```

はrelative化済み。

---

## 5.2 GA4 Processing date canonicalization

Extraction rawはSource由来の表現を保持可能。

例:

```text
Extraction:
  20260924
```

Processingを出る時点では:

```text
2026-09-24
```

へcanonical化。

AnalysisはProcessing済みのISO dateを読む。

Analysisが再びraw compact dateを変換する責任を持ってはいけない。

この修正後:

```text
132 passed
Ruff All checks passed
```

かつ実Processing Artifactでも確認済み。

---

# 6. GA4 / GSC 共通Page Normalization Contract

Integration前に最重要として揃えた契約。

Join key:

```text
site_key
+
normalized_page_key
```

`normalized_page_key` は:

```text
site-scoped path only
```

Policy:

```text
query:
  removed

fragment:
  removed

path case:
  preserve

percent encoding:
  preserve

trailing slash:
  preserve

host:
  keyに含めない

scheme:
  keyに含めない
```

実際のpolicy metadataとして確認された内容:

```json
{
  "drop_fragment": true,
  "host_in_key": false,
  "key_scope": "site_scoped_path",
  "path_case": "preserve",
  "percent_encoding": "preserve",
  "queryless_url": {
    "lowercase_host": true,
    "lowercase_scheme": true,
    "remove_default_port": true
  },
  "schema_version": 1,
  "scheme_in_key": false,
  "strip_query": true,
  "trailing_slash": "preserve"
}
```

実データ例:

```text
GA4 raw:
  /example/path/

GSC raw:
  https://example.invalid/example/path/

normalized_page_key:
  /example/path/
```

Integrationで別のURL normalization ruleを勝手に作らない。

まずSource Processing manifestのpolicy互換性を検証する。

---

# 7. Date Contract

Processing以降:

```text
YYYY-MM-DD
```

Integrationで、

```text
YYYYMMDD -> YYYY-MM-DD
```

のようなSource修正を再実装しない。

日付表現をSource Processingで揃えた理由は、
後段Layerを増やすたびに変換責任が拡散するのを防ぐため。

重要な設計思想:

> Processingは加工責任を果たしてから次へ渡す。

---

# 8. Timezone Semantics

同じdate labelでも、
Sourceの時間意味論は同一UTC 24時間と断定しない。

GA4:

```text
GA4 property timezone semantics
```

GSC:

```text
Search Analytics側のdate semantics
```

Integrationではdate labelでjoin可能だが、
manifest / metadataでSource time semanticsを保持する。

---

# 9. Query Attribution Boundary

絶対に破ってはいけない。

禁止:

```text
GSC query -> exact GA4 session
GSC query -> exact GA4 conversion
GSC query -> exact GA4 key event
```

許可されるのは:

```text
GSC query
  -> GSC page
  -> normalized_page_key
  -> aggregate GA4 page context
```

この時、query grainへ付くGA4 metricsは:

```text
context-only
NON_ADDITIVE_CONTEXT
```

として扱う。

query横断でGA4 metricをSUMしてはいけない。

---

# 10. Metric Semantics

最低限、後段が理解可能なsemantic typeを持つ。

```text
SUM_SAFE
DERIVED_RATIO
NON_ADDITIVE
DIRECT_ONLY
WEIGHTED_WITHIN_GRAIN
UNAVAILABLE
NON_ADDITIVE_CONTEXT
```

代表例:

```text
GSC clicks:
  SUM_SAFE

GSC impressions:
  SUM_SAFE

GSC ctr:
  DERIVED_RATIO
  sum(clicks) / sum(impressions)で再計算

GSC position:
  WEIGHTED_WITHIN_GRAIN

GA4 page metricをquery grainへ付加:
  NON_ADDITIVE_CONTEXT
```

安全でないaggregationはfailまたは明示warning。

---

# 11. 長期基準Artifact — A / B / C

Integration開発のlive fixtureとして用意済み。

---

## 11.1 Dataset A — GA4 約2年

Files:

```text
ga4_extract_20260927T092256772785Z.zip
ga4_process_20260927T092429883217Z.zip
ga4_analysis_20260927T092430631203Z.zip
```

期間:

```text
2024-09-27 -> 2026-09-26
730日
```

確認状態:

```text
Extraction:
  partial_success
  長期取得正常

Processing:
  success
  schema_version = 3
  YYYY-MM-DD canonical date
  lineage hash = Extractionと一致

Analysis:
  success_with_warnings
  lineage hash = Processingと一致
```

absolute local path残存なし。

用途:

```text
Integration Extraction主fixtureのGA4側
GA4長期保持テスト
GA4-only期間の保持テスト
```

---

## 11.2 Dataset B — GA4 GSC同requested期間版

Files:

```text
ga4_extract_20260927T100200937576Z.zip
ga4_process_20260927T100326406969Z.zip
ga4_analysis_20260927T100327014357Z.zip
```

期間:

```text
2025-05-26 -> 2026-09-26
489日
```

確認状態:

```text
Extraction:
  partial_success

Processing:
  success
  schema_version = 3
  date canonical済み
  lineage一致

Analysis:
  success_with_warnings
  lineage一致
```

用途:

```text
B + C
requested rangeを揃えた対照テスト
期間差問題とjoin問題の切り分け
```

---

## 11.3 Dataset C — GSC 約16か月

Files:

```text
gsc_extract_20260927T092431646099Z.zip
gsc_process_20260927T095857594252Z.zip
gsc_analysis_20260927T095900634012Z.zip
```

requested:

```text
2025-05-26 -> 2026-09-26
489日
```

observed:

```text
2025-05-26 -> 2026-09-24
487 / 489日
```

欠けている日:

```text
2026-09-25
2026-09-26
```

これは古い側の欠落ではなく、
GSC reporting lag側。

確認状態:

```text
Extraction:
  v0.7.3
  success
  416,102 rows
  unexpected skip = 0

Processing:
  schema_version = 3
  success_with_warnings
  Extraction manifest hash = MATCH

Analysis:
  success_with_warnings
  Processing manifest hash = MATCH
  Extraction manifest hash = MATCH
```

全output row_count一致。
absolute local pathなし。
credential pathはredacted。

---

# 12. A + C がIntegration主テスト

**Integration Extractionの主fixtureは B+C ではなく A+C。**

理由:

```text
GA4:
  2024-09-27 ------------------------------ 2026-09-26

GSC:
              2025-05-26 ------------------ 2026-09-24
```

という現実的なSource coverage差がある。

Integration Extractionの正しさは、

```text
2024-09-27 -> 2025-05-25
  GA4 observed
  GSC NOT_OBSERVED

2025-05-26 -> 2026-09-24
  GA4 observed
  GSC observed

2026-09-25 -> 2026-09-26
  GA4 observed
  GSC NOT_OBSERVED / reporting lag
```

を**削らず保持できるか**で確認する。

B+Cは対照テスト。

---

# 13. 現在確定している4層

ここが今回最大のArchitecture Decision。

```text
Integration Extraction
    ↓
Integration Processing
    ↓
Integration Analysis
    ↓
Visualization
```

これは「Integration内部のsub-layer」ではない。

**4つの独立した責務Layer / Artifact Stageである。**

ユーザーが明確に否定した設計:

```text
Integration Layerの中に
  Conform
  Merge
  Serving / Dataset Assembly
を全部押し込む
```

理由:

```text
責務が混ざる
ExtractionとProcessingの境界が崩れる
```

---

# 14. 4層の一行定義

絶対に変えない。

```text
Integration Extraction
  = 全部残して、正しくつなぐ

Integration Processing
  = Configで、今回使う形に加工する

Integration Analysis
  = 来たdatasetを、契約に従って汎用分析する

Visualization
  = 来た分析結果を、契約に従って汎用表示する
```

---

# 15. Integration Extraction — 責務

主入力:

```text
GA4 Processing Artifact
GSC Processing Artifact
```

Source Analysisは主入力ではない。

やること:

```text
input discovery / explicit selection
manifest validation
hash lineage validation
schema compatibility validation
site identity validation
date contract validation
page normalization contract validation
coverage / quality inheritance
canonical key
lossless integration
FULL OUTER相当
observation state
join state
dataset catalog
integration quality
integration manifest
```

---

# 16. Integration Extraction — 絶対にやらないこと

禁止:

```text
common periodへのtrim
latest N daysへのtrim
YoY成立期間だけへtrim
GA4をGSC期間へ合わせる
GSCをGA4期間へ合わせる

SEO use case判断
CVR use case判断
ranking
score
recommendation
good / bad判定
low CTR判定
opportunity判定
analysis windowの意味判断
Visualization都合のdata reduction
```

---

# 17. Integration Extraction — Join

基本:

```text
FULL OUTER相当
```

matchedだけを残すINNER JOINは禁止。

最低限:

```text
MATCHED
GA4_ONLY
GSC_ONLY
```

を判別可能にする。

Sourceごとのobservation state例:

```text
OBSERVED
NOT_OBSERVED
UNAVAILABLE
UNKNOWN
```

値と状態は分ける。

---

# 18. Integration Extraction — Site Identity

GA4 PropertyとGSC Propertyが同一siteかを、
hostname一致だけで雑に断定しない。

MVP:

```text
explicit mapping
or
config mapping
```

をSource of Truthにする。

MVPは1 logical site。

将来:

```text
one GA4 property -> multiple GSC properties
subdomains
multiple origins
```

へ拡張可能。

---

# 19. Integration Extraction — Canonical Dataset候補

最新要件での候補:

```text
canonical_pages.csv

page_daily_ga4.csv
page_daily_gsc.csv
query_page_daily_gsc.csv

page_daily_integrated.csv
query_page_daily_integrated.csv

integration_quality.csv
dataset_catalog.json
integration_manifest.json
```

物理ファイル名は、
最新repoを見て既存Conventionと整合させて最終化してよい。

ただし論理Datasetと責務は失わない。

---

# 20. page_daily_integrated の最重要罠

GSCには `search_type` がある。

これを理由に、

```text
同じGA4 page-date metric
  x
search_type数
```

だけ複製して、
普通のSUM可能metricのように見せるのは禁止。

Requirement上許容される安全方式:

```text
A.
  page-date grainを守り、
  search_type固有GSCを別datasetに保持

B.
  search_type grainへGA4 contextを複製するなら、
  GA4 metricをNON_ADDITIVE_CONTEXT扱い

C.
  その他、losslessでaggregation safetyを保証できる表現
```

**「簡単だから複製」は不可。**

この設計点はIntegration Extraction実装時に必ずテストへ固定する。

---

# 21. query_page_daily_integrated の最重要罠

GSC query-page rowへGA4 page-date contextを付けてもよい。

ただし:

```text
ga4_to_query = contextual_only
ga4_metrics_across_query = forbidden / non_additive
```

をmetadataで表現する。

Query conversion attributionを作らない。

---

# 22. Integration Quality

最低限追跡:

```text
input source status
input schema versions
input date ranges
observed date ranges
row counts
unique page keys
matched page keys
GA4-only page keys
GSC-only page keys
matched page-date keys
GA4-only page-date keys
GSC-only page-date keys
match rates
duplicate key counts
invalid key counts
date contract violations
metric semantic violations
lineage validation
```

注意:

match rateが低いだけでfailにしない。

GA4とGSCはSource visibilityが違うため、
unmatchedは正常に発生する。

---

# 23. Integration Extraction Acceptance Gate

最低限、以下を全部実Artifactで通す。

```text
A+Cを入力

GA4 2年の古い期間が消えない

GSC最新2日のlagが0化されない

GA4_ONLY / GSC_ONLY / MATCHEDが分かる

page key contract compatible

manifest hash lineage一致

absolute local pathなし

secretなし

query attribution安全

search_type起因の重複集計リスクなし

manifest row counts一致

duplicate canonical grainなし

unexpected Cartesian explosionなし
```

ここまで通るまではIntegration Processingへ進まない。

---

# 24. Integration Processing — 責務

Integration Extraction Artifactを、

```text
期間
source requirement
grain
dimension
metric
filter
comparison window
coverage condition
use case
```

に応じて**分析用datasetへ加工するLayer**。

このLayerがServing / Dataset Assembly相当の責務を持つ。

Extractionへ戻してはいけない。

---

# 25. Integration Processing — Config First

用途差は基本:

```text
config/
```

へ置く。

コードへ:

```text
SEO専用
CVR専用
Client A専用
site固有URL
site固有threshold
```

を埋め込まない。

Config hierarchy候補:

```text
config/
  integration_extraction/
    source_mapping.yaml
    strictness.yaml

  integration_processing/
    use_cases/
    dataset_specs/
    metric_registry/
    period_policies/
    filter_specs/

  integration_analysis/
    analysis_specs/

  visualization/
    visualization_specs/
```

実repo Conventionに合わせて調整可。

責務境界は変えない。

---

# 26. Integration Processing — 期間alignmentの規定値

ユーザーが明確に決定した。

**Default = common_observed**

A+Cなら:

```text
GA4 observed:
  2024-09-27 -> 2026-09-26

GSC observed:
  2025-05-26 -> 2026-09-24

Integration Processing default:
  2025-05-26 -> 2026-09-24
```

ここで初めて切る。

Integration Extractionでは切らない。

---

# 27. Period Policy

最低限Configで切替可能にする候補:

```text
common_observed
  DEFAULT

all_available

source_available

explicit_range

latest_n_days

comparison_ready
```

名称は実装時に微調整可。

意味を変えない。

---

# 28. Required / Optional Sources

例:

```yaml
sources:
  required:
    - ga4
    - gsc
```

ならcommon_observedはintersection。

例:

```yaml
sources:
  required:
    - ga4
  optional:
    - gsc
```

ならGA4を主期間にできる。

この仕組みで、

```text
cross-source analysis
GA4-only long-term analysis
GSC-only analysis
```

を同じProcessing Engineで作る。

---

# 29. Integration Processing — Safe Aggregation

Metric Registryを使う。

禁止:

```text
CTRの単純平均
positionの無条件単純平均
Non-additive user metricの単純SUM
context metricのquery横断SUM
```

CTR:

```text
sum(clicks) / sum(impressions)
```

Position:

```text
weight semanticsに従う
```

GA4 context:

```text
NON_ADDITIVE_CONTEXT
```

---

# 30. Processing Config と Analysis Spec を混ぜない

ここも重要。

```text
Processing Config
  = 何を分析対象として渡すか

Analysis Spec
  = 渡された対象へ何の分析をするか
```

例:

```text
Processing:
  GA4/GSC共通観測
  page grain
  90日比較に必要なdataset

Analysis:
  current vs previous
  current vs YoY
  delta
  trend
  ranking
```

Analysis都合を全部Processing Configへ押し込まない。

---

# 31. Integration Processing Output Contract

最低限、AnalysisがSource固有コードなしで理解できるように:

```text
processed dataset(s)
dataset_schema.json
metric_registry_snapshot.json
period_definition.json
coverage_summary
processing_manifest.json
```

を持たせる。

Dataset metadata:

```text
dataset_id
dataset_version
grain
key_columns
dimension_columns
metric_columns
date_column
source_columns
row_count
date_range
coverage
```

Metric metadata:

```text
metric_id
source
data_type
semantic_type
aggregation_rule
unit
numerator optional
denominator optional
weight_metric optional
availability
attribution_scope
```

---

# 32. Integration Processing Acceptance Gate

最低限:

```text
A+C default
  -> common_observedへtrim

A+C all_available
  -> union期間を保持

period policyがmanifestへ残る

filtered row reasonを追跡できる

CTR単純平均なし

NON_ADDITIVE誤SUMなし

use case差をConfigのみで表現可能

Analysis固有判断をProcessingへ入れない

Analysisが必要なschema / metric metadataが出る

同一input + 同一configで再現可能
```

---

# 33. Integration Analysis — 核心

ユーザーの表現:

> ここではどんなデータが来るかわからない。どんなものでも受け取ったら分析する。

この意味は:

```text
任意の無契約CSVを理解する
```

ではない。

正しい意味:

```text
Integration Processing Output Contract
を満たすdatasetであれば、

source名
use case名
特定site
特定metric固定
に依存せず分析できる
```

---

# 34. Integration Analysis — 禁止

Core codeに:

```python
if source == "ga4":
    ...

if source == "gsc":
    ...

if use_case == "seo":
    ...
```

のような分岐を増殖させない。

処理判断は:

```text
dataset schema
grain
dimension role
metric semantic
coverage
period metadata
analysis spec
```

から行う。

---

# 35. Integration Analysis — Generic Operations

MVP最低限:

```text
summary
period totals
semantically validなaverage
current vs previous
current vs YoY
absolute delta
relative delta
percentage point delta where applicable
trend
ranking
share / contribution
coverage-aware status
quality-aware suppression
```

旧Roadmapにはthreshold / top_n / bottom_n等も候補がある。

ただし最終的には要件定義書に従う。

---

# 36. Analysis Status

Baseline unavailableを0扱いしない。

例:

```text
OK
BASELINE_ZERO
BASELINE_UNAVAILABLE
CURRENT_UNAVAILABLE
INSUFFICIENT_COVERAGE
```

`BASELINE_ZERO` と `BASELINE_UNAVAILABLE` を分ける。

---

# 37. Analysis Spec

例概念:

```yaml
analysis:
  id: standard_period_comparison
  version: 1

operations:
  - summary
  - period_comparison
  - trend
  - ranking

ranking:
  top_n: 20

change:
  absolute: true
  relative: true
```

Spec変更だけで別分析ができるようにする。

Processingをやり直さず、
同じProcessing Artifactに異なるAnalysis Specを適用できるのが理想。

---

# 38. Integration Analysis Acceptance Gate

最低限:

```text
GA4専用 / GSC専用分岐なし

異なるProcessing datasetを同じEngineで分析

grainをmetadataから認識

metric semanticsに従う

baseline unavailableを0にしない

coverage不足をstatusへ反映

Specだけでoperation切替

Processing Artifactを書き換えない

deterministic

lineageでProcessing runへ戻れる
```

最低2種類で実証:

```text
1. cross-source page dataset
2. GA4-only long-term dataset
```

Core code変更なしで両方通す。

---

# 39. Built-in Analysis Config / Spec 候補

MVPのgenericity検証として有用。

候補:

```text
Page Performance

Query × Page

Period Change

Opportunity Candidates
  ※ heuristicであることを明示
  ※ thresholdはConfig
  ※ Decision Modelではない

Landing / Behavior Diagnostic
```

ただし「最低3spec」のような旧Roadmap条件は、
最新要件定義書との整合を取って採用する。

Source-specific engineを作らず、
**Specとして提供する**ことが重要。

---

# 40. Visualization — 核心

ユーザーの表現:

> 可視化も一緒。どんなデータが来るかわからない。

正しい意味:

```text
Integration Analysis Output Contractを満たすresultなら、
source / use case固有画面へ固定せず、
metadata / visualization specに従って描画
```

---

# 41. Visualization — Input

原則:

```text
Analysis Artifact
Analysis Manifest
result dataset catalog
visualization metadata / spec
```

VisualizationがSource ProcessingやIntegration Extractionを直接読み、
独自分析し直す設計にしない。

---

# 42. Visualization — やってよいこと / ダメなこと

許可:

```text
formatting
date label formatting
percent display
display sort
client-side display filter
```

禁止:

```text
YoYの新規計算
CTRの再集計
positionの再集計
新business metric定義
分析結果の意味判断
score生成
```

分析結果の意味はAnalysisまでで確定。

---

# 43. Visualization — MVP

MVPは:

```text
Static HTML
```

を基本。

サーバー不要。

```text
report.htmlを開けば見られる
```

形を優先。

最低Components:

```text
KPI
time series
comparison table
ranking table
quality / coverage panel
metadata / lineage panel
```

旧RoadmapのP0候補には:

```text
ranking bar chart
scatter plot
```

もある。

最新要件と必要性に応じて実装。

---

# 44. Visualization — Generic Renderer

禁止:

```text
GA4 dashboard
GSC dashboard
SEO dashboard
```

を個別Hardcode。

代わりに:

```text
result type
dimension roles
metric metadata
visualization spec
```

で表示。

最低2種類のAnalysis Artifactを
同一renderer frameworkで表示してgenericityを証明する。

---

# 45. Visualization — Quality / Limitations

品質情報を隠さない。

表示可能にする:

```text
date range
coverage
source availability
warning count
partial status
input run
analysis run
lineage
```

「なぜ値がないのか」をユーザーが追えること。

---

# 46. Visualization — Portability

禁止:

```text
/Users/... 依存
localhost server必須
credential依存
private token embed
```

single-file HTMLを優先してよい。

外部CDN依存は可能なら避ける。

Chart libraryを使うならRenderer層へ隔離。

---

# 47. Visualization Acceptance Gate

最低限:

```text
異なるAnalysis datasetを同じrendererで描画

source名を知らずmetadataから描画

static HTML生成

coverage / warning表示

Visualization内でmetric再定義なし

absolute local pathなし

Analysis lineageへ戻れる

mobile幅で致命的に崩れない

同じArtifact + Specで再現可能
```

---

# 48. Artifact共通原則

Integration以降すべてで:

```text
run単位 immutable
manifest必須
input lineage必須
config/spec hash必須
schema version必須
tool/package version必須
started_at / completed_at
status
dataset row counts
artifact hash検証可能
machine-local absolute path禁止
secret禁止
```

既存runを上書きしない。

---

# 49. Status Model

最低限:

```text
success
success_with_warnings
failed
```

必要に応じ:

```text
partial_success
```

ただしStatusは「例外が出なかった」だけでなく、
quality / coverage / validationを反映する。

---

# 50. Error Classification

最低候補:

```text
INPUT_NOT_FOUND
MANIFEST_INVALID
HASH_MISMATCH
SCHEMA_UNSUPPORTED
SITE_MAPPING_ERROR
KEY_CONTRACT_ERROR
DATE_CONTRACT_ERROR
DUPLICATE_KEY_ERROR
UNSAFE_AGGREGATION
CONFIG_INVALID
INSUFFICIENT_COVERAGE
OUTPUT_WRITE_ERROR
```

Warning例:

```text
SOURCE_PARTIAL
REPORTING_LAG
LOW_MATCH_RATE
OPTIONAL_DATASET_MISSING
PARTIAL_COVERAGE
NON_ADDITIVE_CONTEXT_PRESENT
```

---

# 51. No Cartesian Explosion

Integration実装の重大注意。

特に:

```text
query
x
page
x
search_type
x
GA4 page metrics
```

を無制限に展開しない。

join前:

```text
grain validation
```

join後:

```text
row count growth validation
duplicate grain validation
```

を行う。

想定外multiplicationはfailまたはwarning。

---

# 52. Memory / Performance

A+Cは数十万row規模。

MVPはsingle-machine / local-first。

注意:

```text
全rowを何重にも巨大listへ積む
巨大Cartesian product
同じCSVの無駄な多重読み
```

を避ける。

旧Roadmapでは:

```text
sort + merge
chunked processing
```

等を検討可能としていた。

ただし、最初から過剰最適化もしない。

**正しさ > 性能。**

実データで問題が出てから最適化する。

---

# 53. Atomic Output

既存repoにatomic write utility / conventionがあれば必ず再利用。

旧設計では:

```text
.part
fsync
replace
```

を基本としていた。

Failed runで、
final CSVだけ中途半端に残る状態を避ける。

---

# 54. CLI Design

最終CLI名は最新repoのConventionに合わせる。

Requirement上必要:

### Integration Extraction

```text
explicit input
--latest
config
output root
validation-only optional
```

Working example:

```bash
uv run integration-extract --ga4-latest --gsc-latest
```

### Integration Processing

```bash
uv run integration-process --latest --config <use-case.yaml>
```

### Integration Analysis

```bash
uv run integration-analyze --latest --spec <analysis.yaml>
```

### Visualization

```bash
uv run visualize --latest --spec <visualization.yaml>
```

旧Roadmapの:

```text
analytics-integrate
analytics-analyze
analytics-visualize
```

をそのまま採用する必要はない。

**現在repoの既存CLI命名規則を見て統一すること。**

---

# 55. Version Strategy

旧Roadmapでは:

```text
v0.7.x
  Source stabilization

v0.8.0
  Canonical Integration

v0.9.0
  Generic Analysis

v1.0.0
  Visualization
  MVP complete
```

という意図だった。

その後Architectureが:

```text
Integration Extraction
Integration Processing
Integration Analysis
Visualization
```

へ整理された。

したがって次スレッドでは、
**version numberを勝手に即決しない。**

まず最新repoのversion / historyを確認。

原則として旧意図:

```text
Integration系 = v0.8.x
Analysis = v0.9.x
Visualization / MVP = v1.0.0
```

を参考にできるが、
具体的なminor versionは実装単位に合わせて決める。

---

# 56. 旧RoadmapのUnified Pipelineについて

旧Roadmapにはv1.0で:

```text
one-command pipeline
```

も記載されている。

しかし最新の
`Integration Extraction / Processing / Analysis / Visualization 要件定義書 v1.0`
が正式に要求しているCoreはこの4層。

したがって:

```text
Unified Pipeline Runnerを
最新要件より優先して実装してはいけない。
```

MVP Coreの4層とEnd-to-Endが完成した後、
ユーザーが必要と判断した場合に追加する。

もし追加する場合も、
各Layerの責務を吸収しない単なるorchestratorとする。

---

# 57. MVP Core 完成条件

最新要件に基づく主経路:

```text
GA4 Processing A
        \
         Integration Extraction
        /
GSC Processing C
        ↓
Integration Extraction Artifact
        ↓
Integration Processing
  default = common_observed
        ↓
Integration Analysis
        ↓
Visualization
        ↓
Static HTML
```

さらに同一Integration Extraction Artifactから:

```text
all_available
GA4-only
different period
different metric set
```

等をConfigだけ変えて再Processing可能。

同一Processing Artifactへ:

```text
Analysis Spec A
Analysis Spec B
```

を適用して、
Processingをやり直さず別Analysisを生成可能。

同一Analysis Artifactへ:

```text
Visualization Spec A
Visualization Spec B
```

を適用して、
Analysisをやり直さず別表示を生成可能。

これでLayer分離が実証される。

---

# 58. 推奨実装順序

## Phase 0 — Repo Audit

まず最新repoを読む。

確認:

```text
pyproject.toml
package version
src layout
tests layout
CLI entry points
common utilities
run id
manifest
hash
atomic writer
config loader
CSV helpers
path helpers
page normalization
date helpers
source manifest schema
GA4 Processing output layout
GSC Processing output layout
```

最初から新Architectureを仮定して
巨大patchを書かない。

---

## Phase 1 — Integration Extraction

順序:

```text
1. Input contract models
2. Input selection
3. manifest/hash validation
4. site mapping
5. dataset/grain contract
6. canonical keys
7. lossless joins
8. observation/join states
9. quality output
10. manifest
11. CLI
12. unit tests
13. small golden fixture
14. A+C live validation
15. B+C comparison
```

Gateを通してから次へ。

---

## Phase 2 — Integration Processing

```text
1. Config schema
2. config validation
3. dataset selection
4. source requirement
5. period policies
6. common_observed default
7. all_available
8. explicit/source period
9. metric registry
10. safe aggregation
11. filter
12. grain handling
13. comparison window metadata
14. coverage
15. output contract
16. manifest
17. CLI
18. unit tests
19. A+C default live test
20. A+C all_available live test
```

---

## Phase 3 — Integration Analysis

```text
1. Processing output reader
2. schema interpretation
3. metric registry reader
4. analysis spec schema
5. summary
6. compare
7. delta
8. relative delta
9. trend
10. ranking
11. coverage/status
12. output artifact
13. manifest
14. CLI
15. genericity tests
16. multiple use cases live test
```

---

## Phase 4 — Visualization

```text
1. Analysis result contract reader
2. Visualization spec
3. generic component mapping
4. KPI
5. time series
6. comparison table
7. ranking
8. quality / limitations
9. lineage
10. static HTML
11. responsive CSS
12. no-secret / no-absolute-path checks
13. genericity tests
14. multiple Analysis Artifact render
```

---

## Phase 5 — MVP Hardening

```text
full regression

all manifests / row counts

hash lineage chain

schema versions

config/spec hashes

absolute path scan

secret scan

duplicate grain scan

Cartesian growth scan

A+C E2E

B+C E2E / comparison

re-run determinism

README / usage docs

release version decision
```

README cleanupは実装Critical Pathの最後でよい。

---

# 59. Test Strategy

必須:

```text
unit tests
contract tests
fixture integration tests
artifact validation tests
live artifact tests
regression tests
```

---

# 60. Golden Fixtureに必ず含めるもの

小さい人工fixtureで:

```text
MATCHED page

GA4_ONLY page

GSC_ONLY page

zero metric

UNAVAILABLE metric

NOT_OBSERVED

missing date

trailing slash差

GA4 query-bearing path

GSC absolute URL

multiple search_type

query-page row

baseline zero

baseline unavailable

partial coverage
```

を含める。

Live Artifactだけでedge caseを検証しない。

---

# 61. Live Test — A+C

絶対確認:

```text
Integration Extraction:
  union期間維持

Integration Processing default:
  common_observedへtrim

Integration Processing all_available:
  union期間保持

GSC reporting lag:
  NOT_OBSERVED / coverageとして残る

page key:
  join成功

lineage:
  source manifest hash一致

paths:
  absolute local pathなし

duplicates:
  canonical grain重複なし
```

---

# 62. Live Test — B+C

目的:

```text
requested periodが同じケース

period difference由来のunmatchedと
join / normalization由来のunmatchedを切り分ける
```

B+Cを主fixtureに格上げしない。

A+Cが主。

---

# 63. 既知のSource正常差

GA4とGSCのpage overlapが100%でないのは正常。

過去30日fixtureでは例として:

```text
GA4 pages unique key:
  262

GSC web page unique key:
  223

intersection:
  177
```

のような差があった。

これはSource visibility差。

Integrationは、

```text
100% matchしない = normalization bug
```

と決めつけない。

---

# 64. Source Analysisは並列Branch

Architecture:

```text
GA4 Processing -> GA4 Analysis
GSC Processing -> GSC Analysis

GA4 Processing ─┐
                ├-> Integration Extraction
GSC Processing ─┘
```

Source Analysis ArtifactをIntegration入力にしない。

Source Analysisは独立diagnosticとして残す。

---

# 65. LLMの位置づけ

MVP CoreはLLMなしで完全動作。

LLMは将来:

```text
optional sidecar / provider abstraction
```

として接続。

適切な用途:

```text
要約
説明
示唆候補
テキスト解釈
定性情報統合
```

CoreでLLMに任せない:

```text
join
metric arithmetic
hash validation
coverage判定
canonical key
lineage
safe aggregation
```

LLM failureでCore pipelineを壊さない。

---

# 66. Post-MVPとして明確にDeferred

次スレッドが勝手にscopeを広げないよう固定。

```text
Website Snapshot / crawl
HTML解析
declared canonical
internal link graph
content blocks

ML feature engineering
model training
prediction registry

intervention registry
experiment registry
causal inference

automatic recommendation engine
expected value decision model

LLM platform integration

cloud deployment
multi-tenant SaaS
real-time streaming
BI server
DB必須化
```

Architecture上の拡張余地は残すが、
MVP完成条件へ追加しない。

---

# 67. System Name / 非エンジニア向け説明

現在の表現:

```text
定量化されたデータにより、意思決定を補助するシステム
```

MVPの概念:

```text
GA4 / GSCのデータを取得
↓
分析可能な形へ加工
↓
Source単体でも分析可能
↓
ロスレス統合
↓
用途別に加工
↓
汎用分析
↓
可視化
↓
人間の意思決定を補助
```

将来:

```text
サイト構造 / 文脈
+
定量データ
+
機械学習
+
LLM
+
因果 / 施策履歴
↓
より高度な改善提案・Decision Support
```

---

# 68. ユーザーの作業スタイル — 必ず守る

次担当がここを外すと非常にストレスになる。

## 68.1 無駄な推論をしない

ユーザーは明確に:

```text
今後無駄な推論禁止
```

の意向。

必要な設計判断は深く考える。
しかし関係ない横道・一般論・初心者解説を長々としない。

---

## 68.2 単純な操作は数行で出す

過去に、

```text
6コマンド順番に実行
```

で済む作業へ巨大validator scriptを作ってしまい、
ユーザーから指摘を受けた。

単純な実行手順なら:

```bash
command 1
command 2
command 3
```

でよい。

---

## 68.3 Source patchのような実装変更ではone-shot script可

コードを複数fileへ確実に適用するpatchは、
one-shot shell scriptが有効。

ただし、

```text
repo実物を読まずに過去状態へpatch
```

は禁止。

必ず現在repoへ合わせる。

---

## 68.4 成功を先に宣言しない

以下が来るまで:

```text
pytest output
ruff output
actual artifact
```

成功扱いしない。

「たぶん」「はず」でDONEにしない。

---

## 68.5 Artifact verification重視

コードgreenだけではなく、
実Artifactまで確認する。

特に:

```text
manifest
hash lineage
row counts
relative paths
coverage
date
page key
metric semantics
```

を確認。

---

## 68.6 READMEは最後でもよい

実装Critical Path中にREADME整形へ時間を使いすぎない。

---


# 68.7 実装の受け渡し方式 — 原則 one-shot `.sh`

このプロジェクトでは、複数fileにまたがる実装変更は、
**原則として実行可能な one-shot shell script (`.sh`) にまとめてユーザーへ渡し、
ユーザーがrepo rootで実行する方式**で進めてきた。

次スレッドでもこの方式を基本とする。

例:

```bash
chmod +x apply_integration_extraction_v0_8_0.sh
./apply_integration_extraction_v0_8_0.sh
```

Script側で可能な限り以下を一括実行する。

```text
repo root確認
前提version / file存在確認
backup取得
source追加・修正
tests追加・修正
config追加
pyproject / CLI更新
必要時のみlock更新
ruff check --fix
ruff format
compile check
pytest
final ruff check
```

重要:

```text
・multi-file実装を、ユーザーに大量の手作業copy/pasteでやらせない
・現在repoを実際に読んでから、そのrepo専用scriptを作る
・過去scriptを再利用してcurrent stateを推測しない
・script実行前にbackupを取る
・script自身でも可能ならbackupを作る
・失敗時は途中状態を判別しやすくする
・API callやArtifact再生成が不要なpatchでは勝手に実行しない
・実Artifact生成が必要な検証は、code patchと分離してユーザーへ明示する
```

ただし、単純な操作までshell script化しない。

例えば、

```text
Extraction
Processing
Analysis
```

を順番に実行するだけなら、
巨大validator scriptではなく数行のcommandをそのまま提示する。

原則:

```text
multi-file implementation / patch
  -> one-shot .sh

単純な実行手順
  -> 必要最小限のcommand列
```

Integration Extraction以降の実装でも、
ユーザーから別の指定がない限りこの受け渡し方式を継続する。


# 69. 次スレッドでやってはいけない初動

禁止:

```text
昔のsrc.zipを基準に実装開始

昔のpatch scriptsを順番に適用してcurrent repoを再現

要件定義書を読まず「一般的なデータマート」を作る

Integration Extractionでcommon periodに切る

Integration Extractionでuse caseを決める

Integration ProcessingをExtraction内部へ埋め込む

Integration AnalysisをSEO専用にする

VisualizationをGA4/GSC専用dashboardにする

Source Processingで済んだdate / page normalizationを後段で再実装

NOT_OBSERVEDを0化

queryへGA4 conversionを帰属

search_typeでGA4 metricを安全性なしに複製
```

---

# 70. 次スレッドの最初の行動

ユーザーが:

```text
current repo ZIP
requirements doc
handoff doc
```

をアップロードしたら、

まず:

```text
1. Repo inventory
2. pyproject/version
3. package tree
4. CLI entry points
5. current tests
6. common utilities
7. GA4/GSC Processing manifests / schemas
8. requirement doc read
9. gap analysis
```

を行う。

その後ユーザーへ、

```text
Integration Extraction実装で
どのfilesを新規/変更するか
どの既存utilityを再利用するか
どういうtestsを作るか
```

を短く提示。

そのまま実装へ進む。

不要な再要件定義会議はしない。

---

# 71. Repo Auditで特に探すもの

次担当は既存実装を最大再利用する。

探す:

```text
common/run_id
common/hash
common/path
common/csv
common/json
common/atomic write
common/config
common/status
common/manifest
common/page_keys
common/date parsing
processing config loaders
analysis spec patterns
CLI argument parser
output directory resolver
latest run resolver
```

存在するものを重複実装しない。

---

# 72. 新Module名はRepoを見てから決める

この引き継ぎ書で、

```text
src/.../integration/extraction.py
```

等を決め打ちしない。

なぜなら次スレッドでは最新repoがSource of Truth。

ただし概念上は:

```text
integration extraction
integration processing
integration analysis
visualization
```

の境界をpackage structureでも可能な限り見えるようにする。

---

# 73. Manifest Chainの最終形

最終的に追えるchain:

```text
GA4 Extraction
  -> GA4 Processing
       \
        -> Integration Extraction
       /
GSC Extraction
  -> GSC Processing
       ↓
Integration Processing
       ↓
Integration Analysis
       ↓
Visualization
```

各子manifestは、
直接親のrun_id / manifest hashを持つ。

VisualizationからSourceまで、
manifest chainで遡れること。

---

# 74. Reproducibility

同一:

```text
input artifact hashes
config/spec hashes
tool version
schema version
```

なら、
論理結果を再生成可能。

run_id / timestamp差は許容。

sort / tie break / null orderingはdeterministicにする。

---

# 75. Security

Artifactへ絶対保存しない:

```text
OAuth client_secret
access token
refresh token
authorization code
credential absolute path
local username path
```

過去にユーザーがOAuth情報をログで貼ったことがあるが、
今後も再掲しない。

---

# 76. MVPでDBを必須にしない

CSV / JSON / HTML中心でよい。

将来:

```text
Parquet
SQLite
DuckDB
```

へ拡張可能。

Domain Contractをfile formatに依存させない。

---

# 77. 「データマート + ダッシュボード」的だが、それだけではない

MVPのIntegration Processingは、
用途別dataset assembly / 小型データマートに近い。

Visualizationは、
軽量専用dashboard / reportに近い。

ただしこのシステムはさらに:

```text
quality
coverage
lineage
metric semantics
availability states
attribution safety
```

を持つため、
単なるBIデータマートより監査可能性・分析安全性を重視している。

---

# 78. 重要な設計哲学

## 78.1 前段で情報を捨てない

```text
Integration Extraction:
  最大保持

Integration Processing:
  用途に応じて削減

Integration Analysis:
  分析結果へ集約

Visualization:
  表示へ絞る
```

前段で捨てたものを後段で復元する設計は禁止。

---

## 78.2 責務を次Layerへ押し付けない

過去にdate formattingについて、

```text
Integrationで揃えればよい
```

という誤った判断があり、修正した。

教訓:

```text
そのLayerが持つべき加工責任は、そのLayerで完結させる。
```

Source Processingの仕事をIntegrationへ押し付けない。

Integration Processingの仕事をIntegration Analysisへ押し付けない。

Analysisの仕事をVisualizationへ押し付けない。

---

# 79. MVP Release直前チェックリスト

## Source

```text
[ ] GA4 tests green
[ ] GSC tests green
[ ] Source Analysis regression green
```

## Integration Extraction

```text
[ ] A+C lossless
[ ] B+C comparison
[ ] FULL OUTER
[ ] GA4-only
[ ] GSC-only
[ ] matched
[ ] no duplicate grain
[ ] no Cartesian explosion
[ ] quality
[ ] lineage
```

## Integration Processing

```text
[ ] common_observed default
[ ] all_available
[ ] explicit/source period
[ ] required/optional source
[ ] safe aggregation
[ ] config hashes
[ ] filter reason
```

## Integration Analysis

```text
[ ] generic engine
[ ] multiple dataset types
[ ] no source-specific branch proliferation
[ ] missing != zero
[ ] coverage-aware
[ ] period compare
[ ] ranking/trend
[ ] analysis spec hash
```

## Visualization

```text
[ ] static HTML
[ ] generic renderer
[ ] multiple Analysis Artifacts
[ ] quality
[ ] lineage
[ ] responsive
[ ] no secret
[ ] no absolute local path
```

## Whole MVP

```text
[ ] manifest chain
[ ] row counts
[ ] schema versions
[ ] deterministic result
[ ] live E2E
[ ] docs / usage
[ ] version/release decision
```

---

# 80. 次スレッド冒頭でユーザーが貼れる依頼文

以下をそのまま使える。

```text
前スレからの続き。

添付:
1. 現在の最新repo
2. GENERIC_WEB_INTELLIGENCE_PLATFORM_INTEGRATION_TO_VISUALIZATION_要件定義書_v1.0_20260927.md
3. MVP完成までの完全引き継ぎ書

Source側（GA4 / GSCのExtraction / Processing / Analysis）は完成済み。
次はIntegration Extraction実装から開始。

最重要:
- Integration Extractionはlossless。期間を切らない。use caseを決めない。
- Integration Processingがconfigで期間・用途・grain・metrics等を決める。
- Integration Processingの期間alignment defaultはcommon_observed。
- Integration AnalysisはProcessing contractを満たすdatasetをgenericに分析する。
- Visualizationもgeneric。Analysis結果を再分析しない。
- A（GA4 2年）+ C（GSC 16か月）がIntegration主fixture。
- current repoを最初に精査し、過去patchを再適用して状態再現しない。
- 要件定義書を最優先で守る。
- 無駄な推論はせず、しかし設計・実装・検証は厳密に。
- pytest / ruff / 実Artifact確認まで成功扱いしない。

まずrepoを精査して、Integration Extractionの実装gapと変更対象を確定し、そのまま実装に入って。
```

---

# 81. 最後に — 次担当への最重要メッセージ

このプロジェクトで最も重要なのは、
「GA4とGSCを結合したCSVを作ること」ではない。

重要なのは:

```text
Sourceの意味を壊さず
情報を早い段階で捨てず
missing / unavailable / zeroを区別し
metricの集計安全性を守り
lineageとqualityを残し
用途の選択はProcessingへ閉じ込め
Analysisをgenericにし
Visualizationをgenericにし
MVP後のML / Causal / LLM / Decision Supportへ
同じ基盤を拡張可能にする
```

ことである。

責務境界は以下から動かさない。

```text
Integration Extraction
  = 全部残して、正しくつなぐ

Integration Processing
  = Configで、今回使う形に加工する

Integration Analysis
  = 来たdatasetを、契約に従って汎用分析する

Visualization
  = 来た分析結果を、契約に従って汎用表示する
```

この4段を、
Source側と同じレベルで、

```text
tests
manifest
hash lineage
quality
schema
config/spec
live artifacts
```

まで完成させた時点でMVP Core完成とする。

以上。
````
