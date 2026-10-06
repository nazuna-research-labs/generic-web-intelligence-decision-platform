# システム利用マニュアル

```yaml
document_id: user_manual
document_type: manual
doc_ver: v1.2
language: ja
project_name: Generic Web Intelligence & Decision Platform
project_initiated_at: 2026-09-23 17:35:34 JST
created_at: 2026-10-03 08:14:10 JST
updated_at: 2026-10-05 22:57 JST
author: Nazuna Research
status: active
scope: CLI, Pipeline, config, Artifact, recovery operation
note: 利用者の操作だけに責任を限定し、日付指定・Atomic CLI・6 Pipeline・Artifact・復旧・FAQ・実例を大幅拡充した実運用マニュアル
audience: system users / operators
responsibility: how to operate, interpret, recover, and share runs; no internal development backlog
previous_public_version: user_manual_ja_v1.1.md
mvp_completion_state: COMPLETE
current_package: 0.12.4
public_privacy_review: PASS — public copy removes personal author identity and replaces known site-specific example identifiers with non-identifying reserved examples
```

## 0. MVP COMPLETE後の運用上の重要更新

現在packageは`0.12.4`。MVPはCompletion Gateを通過している。

### Live Pipeline

`full` profileは実GA4/GSC live fetchからVisualizationまでR12で完走済み。通常のexact optionsはcurrent `gwidp pipeline full --help`をSource of Truthとする。

代表的なrelative range:

```text
500daysAgo -> yesterday
```

GSCではrelative dateを`America/Los_Angeles`基準でresolveする。日本時間の日付変更とGSCの`yesterday`変更は一致しない場合がある。実行前にresolved/effective rangeを確認する。

### GA4 partial_success

GA4 Extractionでは互換性のないreport等がskipされ、`partial_success`になる場合がある。GA4 CLIはこの状態でexit 1を返す既存contractを持つ。

Pipeline v0.12.3以降は、GA4 Extractionについて「新規Artifactが正確に1個、manifest status=`partial_success`」の場合に限りusableとして続行する。`failed`やArtifact不在を成功扱いしない。

### Credential location

credential本文をdocs/log/Artifactへコピーしない。default GA4/GSC設定はrepository-root `credentials/`を参照する。public repositoryへcredential / token / actual user configを含めない。

### Dashboard percent表示

v0.12.4以降、relative delta等のratioはpercent formatterで表示する。

```text
internal ratio: -0.3105
display:        -31.05%
```

### Recovery

failed Pipeline runは削除せずmanifest/checkpointを保持する。same-version/config/spec条件が満たされる場合は`--resume`を使い、成功済みstageをhash検証してreuseする。runtime behavior/versionが変わった修正では、旧checkpointを無理にresumeせず新runで再検証する。

## 1. このマニュアルの目的

この文書は、Generic Web Intelligence & Decision Platformをrepo rootから実際に操作するための利用マニュアルである。

Atomic CLI、Pipeline、Config、Artifact、認証、resume/recovery、日付指定、結果確認、利用時のトラブルシュートまで扱う。設計仕様そのものはRequirements/Reference/Handoffを参照する。

## 2. 前提

- macOS等のlocal environment
- Python 3.11系
- `uv`
- Google OAuth credentials
- 対象GA4 Property / GSC propertyへの権限

repo rootで実行する。`.venv`の手動activateは必須ではない。

```bash
uv sync --locked
```

## 3. Configの場所

```text
config/
├── README.md
├── user/
└── system/
```

### `config/user/`

利用者がsite/source/runtime選択を変更する場所。

代表例:

```text
config/user/sources/ga4/profiles.yaml
config/user/sources/ga4/reports.yaml
config/user/sources/gsc/gsc.yaml
config/user/processing/ga4/processing.yaml
config/user/integration/integration_extraction.yaml
config/user/pipeline/default.yaml
```

### `config/system/`

metric semantics、Integration Processing dataset spec、Analysis spec、Visualization spec等。通常利用では編集しない。

## 4. まず`--help`を確認する

各CLIのexact optionは実装がSource of Truth。

```bash
uv run gwidp --help
uv run gwidp pipeline --help
uv run ga4-export --help
uv run ga4-process --help
uv run ga4-analyze --help
uv run gsc-export --help
uv run gsc-process --help
uv run gsc-analyze --help
uv run integration-extract --help
uv run integration-process --help
uv run integration-analyze --help
uv run visualize --help
```

## 5. 6つのPipeline profile

### 5.1 `full`

通常の一気通貫。

```text
GA4 Extraction
GSC Extraction
-> GA4/GSC Processing
-> Integration Extraction
-> Integration Processing
-> Integration Analysis
-> Visualization
```

Source Analysisは挟まない。

Planだけ確認:

```bash
uv run gwidp pipeline full --plan
```

実行:

```bash
uv run gwidp pipeline full
```

期間override:

```bash
uv run gwidp pipeline full \
  --start 2025-05-26 \
  --end 2026-09-26
```

GA4 profile override:

```bash
uv run gwidp pipeline full --ga4-profile <profile_id>
```

### 5.2 `full_from_extraction`

GA4/GSC Extractionだけ手動で済ませ、Processing以降を一括実行する保険profile。

```bash
uv run gwidp pipeline full_from_extraction \
  --ga4-extraction output/extraction/ga4/.../ga4_extract_... \
  --gsc-extraction output/extraction/gsc/.../gsc_extract_...
```

認証が不安定、ブラウザOAuthをSourceごとに確実に終わらせたい、既存Extractionを再利用したい場合に使う。

### 5.3 `sources`

GA4/GSCをそれぞれSource Analysisまで実行。Integrationしない。

```bash
uv run gwidp pipeline sources
```

成果物はGA4 AnalysisとGSC Analysisの2つ。

### 5.4 `ga4`

```bash
uv run gwidp pipeline ga4
```

GA4 Extraction -> Processing -> Analysis -> Visualization。

### 5.5 `gsc`

```bash
uv run gwidp pipeline gsc
```

GSC Extraction -> Processing -> Analysis -> Visualization。

### 5.6 `integration`

既存Processing Artifactを入力。Live fetchなし。

```bash
uv run gwidp pipeline integration \
  --ga4-processing output/processing/ga4/.../ga4_process_... \
  --gsc-processing output/processing/gsc/.../gsc_process_...
```

## 6. Pipeline Config

Default:

```text
config/user/pipeline/default.yaml
```

現在の代表path:

```yaml
defaults:
  ga4_profile: default

paths:
  ga4_profiles: config/user/sources/ga4/profiles.yaml
  ga4_reports: config/user/sources/ga4/reports.yaml
  ga4_processing: config/user/processing/ga4/processing.yaml
  gsc: config/user/sources/gsc/gsc.yaml
  integration_extraction: config/user/integration/integration_extraction.yaml
  integration_processing: config/system/integration/processing/cross_source_page_daily.yaml
  integration_analysis: config/system/integration/analysis/generic_page_analysis.yaml
  visualization: config/system/visualization/default_dashboard.yaml
```

別configを使う場合:

```bash
uv run gwidp pipeline full --config path/to/pipeline.yaml
```

## 7. `--plan`

`--plan`はconfig/profileをvalidateし、Stage順序を表示するだけで実行しない。Live fetchも起こさない。

本番実行前の確認、profile contract確認、認証を発生させたくないときに使う。

## 8. `full`が途中で失敗した場合

### 8.1 何が保存されるか

Pipeline run directoryに`pipeline_manifest.json`を保存し、成功/失敗Stage、Artifact/hash、attemptを追跡する。

### 8.2 resume

```bash
uv run gwidp pipeline full \
  --resume output/pipeline/full/pipeline_full_<run_id>
```

resume時:

1. profile / stage plan / config整合を検証
2. 既存成功Artifactが存在するか確認
3. manifest hashが変わっていないか確認
4. validなら`reused`
5. failed stageを再試行
6. downstreamを続行

### 8.3 GSC認証だけ手動復旧した例

GA4 Extraction成功後、GSCでOAuthが止まったとする。

手動でGSC Extractionを成功させた後:

```bash
uv run gwidp pipeline full \
  --resume output/pipeline/full/pipeline_full_<run_id> \
  --gsc-extraction output/extraction/gsc/.../gsc_extract_<run_id>
```

手動Artifactは`supplied`として履歴へ残す。GA4成功Artifactは`reused`になる。

GA4も同様に`--ga4-extraction`を使える。

## 9. Atomic CLI

PipelineはAtomic CLIを置換しない。調査・再実行・特定Layerだけ作り直す場合に使う。

### 9.1 GA4 Extraction

GA4を単体取得する基本形。

```bash
uv run ga4-export \
  --profiles-config config/user/sources/ga4/profiles.yaml \
  --reports-config config/user/sources/ga4/reports.yaml \
  --profile default \
  --start 500daysAgo \
  --end yesterday \
  --output-root output/extraction/ga4
```

`--start` / `--end`は、現行date contractでは次を受け付ける。

```text
YYYY-MM-DD
today
yesterday
NdaysAgo
```

`monthsAgo`は使用しない。GSCの最大履歴に合わせてGA4も同じrequested rangeで取得したい場合、`500daysAgo -> yesterday`を使うと毎回「16か月前の日付」を計算せずに済む。

GA4自体はGSCより長い期間を取得できるため、長期分析が目的なら`730daysAgo`等へ広げてもよい。

### 9.2 GSC Extraction

GSCを単体取得する基本形。

```bash
uv run gsc-export \
  --config config/user/sources/gsc/gsc.yaml \
  --start 500daysAgo \
  --end yesterday \
  --output-root output/extraction/gsc
```

Search Consoleの保持期間は直近16か月が上限である。16か月は月の日数により固定日数ではないため、このマニュアルでは安全側の運用値として`500daysAgo`を使う。

重要:

- `500daysAgo`は「500日分のGSCデータが必ず存在する」という意味ではない。
- 実際に返るのはSearch Console側で保持・利用可能な範囲。
- 直近日はreporting lagで未到着のことがある。
- 未到着日を0として解釈しない。
- 実際のobserved rangeはExtraction/Processing Artifactのcoverage/manifestで確認する。

固定期間で再現性を優先したい場合は`YYYY-MM-DD`を使う。

### 9.3 GA4 Processing

```bash
uv run ga4-process \
  --input-run <GA4_EXTRACTION_ARTIFACT> \
  --config config/user/processing/ga4/processing.yaml \
  --output-root output/processing/ga4
```

### 9.4 GSC Processing

```bash
uv run gsc-process \
  --input-run <GSC_EXTRACTION_ARTIFACT> \
  --output-root output/processing/gsc
```

### 9.5 GA4 Analysis

```bash
uv run ga4-analyze \
  --input-run <GA4_PROCESSING_ARTIFACT> \
  --output-root output/analysis/ga4
```

### 9.6 GSC Analysis

```bash
uv run gsc-analyze \
  --input-run <GSC_PROCESSING_ARTIFACT> \
  --output-root output/analysis/gsc
```

### 9.7 Integration Extraction

```bash
uv run integration-extract \
  --ga4-input-run <GA4_PROCESSING_ARTIFACT> \
  --gsc-input-run <GSC_PROCESSING_ARTIFACT> \
  --config config/user/integration/integration_extraction.yaml
```

### 9.8 Integration Processing

```bash
uv run integration-process \
  --input-run <INTEGRATION_EXTRACTION_ARTIFACT> \
  --config config/system/integration/processing/cross_source_page_daily.yaml
```

### 9.9 Integration Analysis

```bash
uv run integration-analyze \
  --input-run <INTEGRATION_PROCESSING_ARTIFACT> \
  --spec config/system/integration/analysis/generic_page_analysis.yaml
```

### 9.10 Visualization

```bash
uv run visualize \
  --input-run <ANALYSIS_ARTIFACT> \
  --spec config/system/visualization/default_dashboard.yaml
```

## 10. Artifactの探し方

主なroot:

```text
output/extraction/ga4/
output/extraction/gsc/
output/processing/ga4/
output/processing/gsc/
output/analysis/ga4/
output/analysis/gsc/
output/integration/extraction/
output/integration/processing/
output/integration/analysis/
output/visualization/
output/pipeline/<profile>/
```

Run directoryを直接指定する。`latest`という曖昧な参照より、監査可能な具体runを優先する。

## 11. Manifestの役割

Manifestは単なるlogではない。

- parent Artifact
- hash
- config/spec
- period
- row count
- quality/coverage
- stage status

等を追跡する。Artifactを手で編集した場合、hash検証がfailするのは意図した挙動。

## 12. OAuth / 認証

初回やtoken失効時にbrowser認証が必要になることがある。

`full`で途中停止しても、成功済みSourceを再取得する必要はない。resume/recoveryを使う。

credential/tokenはGitへcommitしない。

## 13. Sourceごとの意味上の注意

### GA4

behavior/event/session系。

### GSC

search visibility/query/page/click/impression/CTR/position。reporting lagあり。

### Cross-source

GSC queryはGA4 conversionへのexact attributionではない。

## 14. 利用時によく遭遇する問題

### 認証画面が出る / 認証で止まる

初回、token失効、scope変更等ではOAuth認証が必要になる。`full`の途中で止まっても、成功済みStageを捨てずresumeを使う。

### GSCの最終日まで値がない

Search Consoleのreporting lagが主因になり得る。`--end yesterday`を指定しても、observed endが1〜数日前になる場合がある。これは0ではない。

### Pipelineが途中でfailedになった

表示されたPipeline run directoryを控え、`pipeline_manifest.json`を確認する。成功済みStageがある場合は`--resume`を使う。

### 手動Extractionを使って続行したい

`full_from_extraction`を新規実行するか、failed `full`へ`--ga4-extraction` / `--gsc-extraction`で回復Artifactを渡す。

### Artifactを指定したらrejectされた

対象site/property、schema、manifest/hash、config、profileが想定と一致しているか確認する。別siteのArtifactを無理に流用しない。

### Dashboardに値がない

まずAnalysis Artifact自体に対象metric/resultがあるか、次にVisualization Input/Spec、Coverage/Qualityを確認する。空欄を0と決めつけない。

## 15. Outputを確認する順序

1. CLI exit / Pipeline status
2. Manifest
3. row count / coverage / quality
4. Result CSV/JSON
5. Dashboard
6. 必要ならparent Artifactへ遡る

## 16. 安全な運用

- `output/`, `credentials/`, `.backups/`を不用意に削除しない
- Persistent Artifactへabsolute path/secretを入れない
- unknown missing stateを0へ変換しない
- config/systemを意味を理解せず編集しない
- live fetchをtestのたびに繰り返さない

## 17. どのprofileを選ぶか

| 目的 | Profile |
|---|---|
| 全部一気に | `full` |
| Extractionを手動管理 | `full_from_extraction` |
| Source分析だけ2系統 | `sources` |
| GA4だけ | `ga4` |
| GSCだけ | `gsc` |
| 既存Processingから統合だけ | `integration` |

## 18. 毎回の標準利用フロー

日常利用では、開発状況を気にするより「どこから実行するか」を決める。

```text
新しくSource取得から全部やる
  -> full

Source取得だけ手動で確実に済ませたい
  -> ga4-export / gsc-export
  -> full_from_extraction

Source単体分析だけ欲しい
  -> ga4 / gsc / sources

既存Processingから統合し直したい
  -> integration

特定Layerだけ再実行したい
  -> Atomic CLI
```

実行前は`--plan`または各CLIの`--help`で引数を確認し、実行後はManifestとCoverage/Qualityを確認する。

## 19. 初回OAuth準備

Google Cloud側でGA4 Data API / Admin API、GSC利用に必要なOAuthを準備する。Desktop App credentialを使う構成ではcredential fileを`credentials/`配下の所定pathへ置く。

実credential/tokenはGit管理しない。example fileだけを共有する。OAuth appがTestingの場合は実行アカウントをTest usersへ追加し、対象Propertyの閲覧権限を持たせる。

## 20. 初回実行の推奨順序

いきなり`full`を長期間で走らせるより、初回は次の順序が安全。

```text
1. uv sync --locked
2. Atomic CLI --help
3. GA4短期間Extraction smoke
4. GSC短期間Extraction smoke
5. Source Processing確認
6. Pipeline --plan
7. 既存Artifactでintegration profile
8. full / full_from_extraction
```

ただし、プロジェクトですでにReference Artifactが検証済みなら毎回live smokeを繰り返す必要はない。

## 21. Pipeline Manifestのstatus

resume対応ManifestではStage attemptの状態を区別する。

- `success`: そのattemptでStageを実行し成功
- `failed`: 実行失敗
- `reused`: 過去成功Artifactをhash検証して再利用
- `supplied`: ユーザーが手動生成したArtifactを回復入力として採用

この区別により、最終Artifactが「どのSourceを自動取得し、どこを手動復旧したか」を追える。

## 22. ResumeでConfigを変える場合の注意

resumeは同一runの継続である。Stage planや重要Configが別物なら、新runとして実行する方が安全。Resume loaderがprofile/stage/config整合をrejectした場合、無理にManifestを書き換えない。

## 23. 既存Artifactを選ぶときの基準

単にmtimeが新しいだけでなく、Manifestを確認する。

- property/site
- requested period
- observed period
- processing config
- schema version
- row count
- quality

異なるsite/propertyのArtifactをIntegrationへ渡さない。

## 24. GSC最大履歴を含める日付指定

毎回カレンダーで「16か月前」を計算しないため、通常は次を使う。

```text
start = 500daysAgo
end   = yesterday
```

GA4:

```bash
uv run ga4-export \
  --profiles-config config/user/sources/ga4/profiles.yaml \
  --reports-config config/user/sources/ga4/reports.yaml \
  --profile default \
  --start 500daysAgo \
  --end yesterday
```

GSC:

```bash
uv run gsc-export \
  --config config/user/sources/gsc/gsc.yaml \
  --start 500daysAgo \
  --end yesterday
```

Pipelineで`--start` / `--end`を使う場合も同じ値に揃える。

```bash
uv run gwidp pipeline full \
  --start 500daysAgo \
  --end yesterday
```

`500daysAgo`を使う理由は「16か月=500日」だからではない。16暦月は固定日数ではないため、保持上限を十分に覆う余裕を持ったrequested startとして使う。

## 25. Integration Processing configを切り替える

System configにはcross-source page daily、all_available、GA4-only、GSC-only、query-page等のdataset definitionが存在し得る。

configを切り替えたら、結果のperiod/grain/metricsが変わる。Dashboardだけ比較して「前runと値が違う」と判断せず、processing config hashを確認する。

## 26. Analysis Specを切り替える

Integration AnalysisはProcessing DatasetとAnalysis Specを分離する。

Processing Config: **何を対象にするか**
Analysis Spec: **対象に何をするか**

同じDatasetから別Analysisを生成できる。

## 27. Visualization Specを切り替える

Visualization Specは表示component/metric/formatを変える。Analysis値そのものを変更するための場所ではない。

## 28. Outputを消す前に

`output/`は単なるcacheではなく検証済みArtifactを含み得る。容量整理で消す場合、Reference ArtifactやHandoffで参照中のrunか確認する。

`.backups/`もmigration recoveryに必要な場合があるため、MVP release前に一括削除しない。

## 29. 別マシン・別ディレクトリで利用を再開する

別PCや別directoryへrepoを移した場合は、まず依存関係とconfig/credentialの配置を確認する。

```bash
uv sync --locked
uv run gwidp --help
uv run gwidp pipeline full --plan
```

`credentials/`やtokenはGitに入れず、安全な方法で新環境へ配置する。既存Artifactをコピーした場合は、Manifestが参照するpersistent pathがproject-relativeであることを確認する。

## 30. `--pipeline-output-root`

Pipeline manifestのrootを変更したい場合に使用できる。Artifact本体の各Layer output rootとは別概念。通常はdefault `output/pipeline`でよい。

## 31. 実行結果の保管と共有

利用結果を後で再確認できるよう、最低限次を残す。

- 実行command
- 実行profile
- input Artifact path
- output Artifact path
- Pipeline run path
- manifest
- config/specの識別情報
- observed period / coverage
- quality status

token、client secret、credential本文は共有しない。

## 32. 利用者向け確認コマンド

```bash
uv run gwidp --help
uv run gwidp pipeline --help
uv run gwidp pipeline full --plan
uv run ga4-export --help
uv run gsc-export --help
uv run visualize --help
```

Artifactの存在確認:

```bash
find output -type f -name '*manifest*.json' | sort
```

Pipeline run確認:

```bash
find output/pipeline -type f -name pipeline_manifest.json | sort
```

これらは利用対象・引数・成果物を確認するためのコマンドであり、データ取得そのものを始めないものを優先している。

## 33. トラブルシュート例 — 認証ブラウザが出ない

- tokenがまだvalidで再認証不要か
- credential pathが正しいか
- OAuth client typeがDesktopか
- test user登録
- callback/browser制約

Pipelineの問題とOAuth provider側の問題を分ける。

## 34. トラブルシュート例 — Integration rowが増えすぎる

- GSC query/search_type grain
- GA4 contextの複製
- join key
- canonical page key
- direct row count before/after

Cartesian growthを「データが多いだけ」と見過ごさない。

## 35. トラブルシュート例 — CTRが変

row CTRの平均を取っていないか確認。clicks/impressionsから安全にderivedされているか、metric registryを確認。

## 36. 利用者が編集してよいもの / 通常触らないもの

| 区分 | 原則 |
|---|---|
| `config/user` | 利用目的に応じ編集可 |
| `config/system` | 通常触らない |
| `src/` | 開発者のみ |
| `output/` | generated Artifact。手編集非推奨 |
| `credentials/` | secret。共有/commit禁止 |
| `docs/public` | versioned正式文書 |

## 37. 操作結果を共有するとき

最低限、command、profile、input run、output run、失敗Stepを共有する。秘密情報やtokenを貼らない。



## 38. 日付指定の完全ガイド

### 38.1 相対日付

現行date expression:

| 表記 | 意味 |
|---|---|
| `today` | 実行日のtoday |
| `yesterday` | 実行日の前日 |
| `30daysAgo` | todayから30日前 |
| `365daysAgo` | todayから365日前 |
| `500daysAgo` | todayから500日前 |
| `730daysAgo` | todayから730日前 |
| `YYYY-MM-DD` | 絶対日付 |

`monthsAgo`は使わない。

### 38.2 日常運用の推奨

GSCを含む通常の最大履歴取得:

```text
--start 500daysAgo --end yesterday
```

短期確認:

```text
--start 30daysAgo --end yesterday
```

固定期間の再現:

```text
--start 2025-05-26 --end 2026-09-26
```

### 38.3 requested rangeとobserved range

`--start` / `--end`はrequested range。Source側にデータがない日はobserved rangeへ現れないことがある。特にGSCはreporting lagがあるため、requested endとobserved endを分けて読む。

## 39. 最短Quick Start

### 39.1 まずPlanだけ

```bash
uv sync --locked
uv run gwidp pipeline full --plan
```

### 39.2 GSC最大履歴を含めて一気通貫

```bash
uv run gwidp pipeline full \
  --start 500daysAgo \
  --end yesterday
```

### 39.3 認証をSourceごとに済ませてから一気通貫

```bash
uv run ga4-export --start 500daysAgo --end yesterday
uv run gsc-export --start 500daysAgo --end yesterday
```

生成された2つのExtraction Artifactを指定して:

```bash
uv run gwidp pipeline full_from_extraction \
  --ga4-extraction <GA4_EXTRACTION_ARTIFACT> \
  --gsc-extraction <GSC_EXTRACTION_ARTIFACT>
```

## 40. Profile選択を迷ったとき

### Source取得も含めて全部

`full`

### OAuthをSourceごとに手動管理

`full_from_extraction`

### GA4/GSCのSource Analysisを両方欲しい

`sources`

### GA4だけ

`ga4`

### GSCだけ

`gsc`

### 取得済み・加工済みSourceから統合だけやり直す

`integration`

## 41. Artifactを選ぶ実践手順

1. 対象Layerのoutput rootを開く。
2. run directoryを候補にする。
3. manifestを開く。
4. site/propertyを確認する。
5. requested/observed periodを確認する。
6. schema/config/specを確認する。
7. row countとqualityを確認する。
8. 親Artifact hashが一致していることを確認する。
9. そのrun directoryを次CLIへ渡す。

mtimeだけで決めない。

## 42. Pipeline失敗からの復旧フロー

```text
Pipeline failed
  -> terminalのfailed stage確認
  -> pipeline_manifest.json確認
  -> 成功済みStageを確認
  -> 認証/API/入力の原因を解消
  -> --resume
  -> 必要なら supplied Extraction Artifact
  -> final Artifact確認
```

再実行前にPipeline run directoryを控える。

## 43. `reused`と`supplied`の読み方

`reused`は過去attemptでPipeline自身が成功させたArtifactをhash検証して再利用した状態。

`supplied`は利用者が明示的に与えた回復Artifact。

どちらも「何となく再利用」ではなくManifestに履歴が残る。

## 44. Configを変更するときの順序

1. 何を変えたいかを決める。
2. `config/user`で対応可能か確認する。
3. system contractを変える必要がないなら`config/system`は触らない。
4. 変更前configを保存する。
5. `--plan`で解決結果を確認する。
6. 実行する。
7. Artifactのconfig hashを確認する。

## 45. 結果を読む順番

```text
Pipeline / CLI status
  -> Manifest
  -> Coverage
  -> Quality
  -> Result tables / JSON
  -> Dashboard
  -> Lineage
  -> 必要なら親Artifact
```

Dashboardだけを見て問題を判断しない。CoverageやQualityが悪ければ値の解釈を保留する。

## 46. GA4利用時のチェックリスト

- Propertyは正しいか
- profileは正しいか
- requested periodは正しいか
- reports configは意図したものか
- event/key eventの意味は正しいか
- page/landing-page grainを混同していないか
- Processing Artifactを手編集していないか

## 47. GSC利用時のチェックリスト

- Search Console propertyは正しいか
- `search_type`は意図したものか
- query/page dimensionを混同していないか
- requested endとobserved endを区別したか
- CTRを単純平均として解釈していないか
- average positionを加算可能なmetricとして扱っていないか

## 48. Integration利用時のチェックリスト

- GA4/GSCが同じsiteか
- page normalizationが期待どおりか
- period modeは何か
- grainは何か
- required/optional sourceは何か
- query grainにGA4 contextを合算していないか
- GA4_ONLY / GSC_ONLY / MATCHEDを確認したか

## 49. Dashboardを利用する前の確認

Visualization Artifactの`dashboard.html`を開く前に、少なくともVisualization Manifestがsuccessであること、元input Artifactが意図したrunであることを確認する。

詳細な画面操作は`dashboard_manual_ja_v1.2.md`を参照する。

## 50. Runを再現するとき

同じ結果を再現したい場合は、最低限次を揃える。

- source Artifact
- config/spec
- package version
- parent manifest hash
- period
- profile
- relevant environment dependencies

「同じコマンド文字列」だけではなくArtifact/config hashまで見る。

## 51. Outputを整理するとき

削除前にReference/Handoff/READMEから参照されていないか確認する。特に`output/`には検証済みArtifactが残ることがある。

利用者が安全に削除判断できない場合は、まず別directoryへ退避する。

## 52. セキュリティ

- credential/tokenをdocsやchatへ貼らない
- secretをGitへcommitしない
- Artifact共有前にcredential path / token / private URLを確認する
- persistent manifestへmachine-specific absolute pathを残さない

## 53. よくある質問

### Q. GSCを16か月取りたい。開始日は？

`500daysAgo`を推奨。16暦月は固定日数ではないため、安全側のrequested startとして使う。

### Q. `16monthsAgo`は使える？

現行date contractでは使わない。`NdaysAgo`を使う。

### Q. `yesterday`まで指定したのにGSCが2日前で止まる

reporting lagを疑い、coverage/observed rangeを確認する。0埋めしない。

### Q. `full`がGSC認証で止まった

同じPipeline runを`--resume`する。必要なら手動GSC Extractionを`--gsc-extraction`で渡す。

### Q. GA4はもっと長く取れる？

取れる。GSCと揃える必要がなければ`730daysAgo`等へ広げられる。

### Q. どのArtifactが最新かわからない

mtimeだけでなくmanifestのsite/property/period/config/qualityを確認する。

## 54. Atomic CLI Help Snapshot

以下はこのdocs生成時点で自動取得できた`--help`のsnapshot。利用時は現在の`--help`をSource of Truthとする。

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
