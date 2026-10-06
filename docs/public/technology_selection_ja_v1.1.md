# 汎用Webインテリジェンス意思決定プラットフォーム 技術選定

```yaml
document_id: technology_selection
document_type: technology_selection
doc_ver: v1.1
language: ja
project_name: Generic Web Intelligence & Decision Platform
project_initiated_at: 2026-09-23 17:35:34 JST
created_at: 2026-10-03 08:14:10 JST
updated_at: 2026-10-05 22:57 JST
author: Nazuna Research
status: active
scope: current technology stack and selection rationale
note: 旧ファイルの別プロジェクト本文は採用せず、本Projectの実装技術だけで新規作成
previous_public_version: technology_selection_ja_v1.0.md
mvp_completion_state: COMPLETE
current_package: 0.12.4
public_privacy_review: PASS — public copy removes personal author identity and replaces known site-specific example identifiers with non-identifying reserved examples
```

## 0. MVP COMPLETE時点のTechnology State

MVP Completion時点のpackageは`0.12.4`。基本Technology選定は変更していない。

MVP closureでTechnology運用上確認された点:

- Python runtime targetは引き続き3.11系。実MVP gateではPython 3.11.9環境を使用。
- `uv --locked`をdependency/runnerの基準とする。
- PipelineはSource CLIを置換せずsubprocess/CLI contractをorchestrateする。
- GA4 CLIの非0 `partial_success`をPipeline側がArtifact manifestを読んで限定受理することで、Source CLIのexit contractとorchestration contractを分離した。
- React primary / Static auditの二経路を維持。v0.12.4でReact ratio-percentage formatterを修正。
- local-first Artifact（CSV/JSON/HTML/ZIP）とhash/manifest/lineageをM5で再確認。
- root one-shot / resume `.sh`は開発・migration・hardening用途。恒久product APIと混同しない。

Post-MVPでDB/warehouse/ML/LLM/BI frameworkを追加する場合も、現在のArtifact contractと監査可能性を壊さず、必要性を根拠に選定する。

## 1. 選定方針

技術は「流行しているか」ではなく、現在の責務を最小の複雑性で満たし、後から置換可能かで判断する。

重視する観点:

- data correctness
- reproducibility / deterministic execution
- local-first
- portability
- auditability / artifact lineage
- external APIなしでもtestしやすい
- unnecessary framework lock-inを避ける
- MVPで過剰なstorage/BI/ML基盤を導入しない

## 2. 現在採用している主要技術

| 領域 | 技術 | 現在の役割 |
|---|---|---|
| Runtime | Python 3.11.9 | Core/CLI/data processing |
| Package | uv | dependency / lock / `uv run` |
| Test | pytest | unit / contract / regression |
| Lint/Format | Ruff | static lint / formatter |
| Config | YAML / PyYAML | user/system config / specs |
| Google API | google-analytics-data | GA4 Data API |
| Google API | google-analytics-admin | GA4 Admin metadata |
| Auth | google-auth / google-auth-oauthlib | shared OAuth |
| HTTP | requests / AuthorizedSession | GSC API等 |
| Artifact | CSV / JSON / HTML / ZIP | local-first interoperable output |
| Frontend | React 19.3.0 | primary dashboard UI |
| Frontend runtime | ReactDOM 19.3.0 | browser rendering |
| Build | Vite 8.3.1 | React bundle |
| Chart | custom renderer, no chart library | MVP dependency最小化 |
| Shell automation | POSIX/Bash `.sh` | migration / one-shot / resume gates |

package versionはrepoの`pyproject.toml`をSource of Truthとし、MVP completion時点は`0.12.4`。

## 3. Python 3.11 + uv

### 採用理由

- Google client libraryと成熟したdata tooling
- CLI/JSON/CSV/YAML処理が簡潔
- pytest/Ruff ecosystem
- `uv.lock`によるdependency固定
- `.venv`を人間がactivateしなくても`uv run`で再現可能

### 運用上の注意

repo rename時、venv console script shebangに旧absolute pathが残る場合がある。venvを再構築する。

Pipeline child CLIの解決では、macOSのsymlinkされた`.venv/bin/python`を`resolve()`してsystem interpreterへ飛ばさず、active launcher directoryを基準にする。

## 4. Google API / OAuth

GA4とGSCは同じGoogle ecosystemでもAPI semanticsが異なる。Auth共通化は行うが、Source Extraction/Processing responsibilityは統合しない。

認証はlive E2Eを不安定にし得るため、Pipelineにresume/recoveryを持たせ、`full_from_extraction`も用意する。

## 5. YAML Config architecture

```text
config/user/
  user editable / site / source / runtime selection

config/system/
  defaults / specs / contracts
  normally do not edit
```

pathは`web_intelligence.common.config_paths`等のcentral resolverを使用し、各moduleがliteral pathを散在させない。

## 6. CSV / JSON / HTMLをMVPで使う理由

- inspectionしやすい
- diffしやすい
- third-party toolで開ける
- storage engineを早期固定しない
- Artifact hash/lineageを単純に保てる

Parquet / DuckDBは将来scale requirementが成立したときに導入候補。MVPの必須依存ではない。

## 7. React + Vite

VisualizationはStatic HTMLのみから、React primary + static auditへ拡張した。

exact pin:

```text
react       19.3.0
react-dom   19.3.0
vite         8.3.1
```

### chart libraryを採用しなかった理由

MVPで必要なtime series / tables / interactionを満たす範囲では、外部chart library依存を増やさずGeneric ContractとRenderer責務を先に固める方を優先した。

将来、高密度interactive BIが必要になれば再評価する。

## 8. Artifact / Manifestを中心にした設計

Database中心ではなく、各runがimmutable Artifact + Manifestを生成し、direct parent/hash/config/specを追跡する。

メリット:

- replay / audit
- failure isolation
- layer間責務が明確
- existing Artifactを使ったnon-live E2E
- Pipeline resume

## 9. pytest / Ruff

unitだけでなく、contract / source structure / portable lineage / artifact validationをtestにする。

Ruffはstyleだけでなく、単純化可能なbranch等のcode quality gateにも使う。自動fixで直らない指摘は無視せず、意図が明確になるようsourceを修正する。

## 10. `.sh` automation

大量編集をチャット上の断片patchで行わず、repo rootから実行可能なscriptへまとめる。

- preflight
- backup
- patch
- test
- artifact gate
- success/fail reporting

を一つの再現可能な操作にする。

## 11. 現在採用していない / Deferred技術

| 技術領域 | 現状 | 再検討trigger |
|---|---|---|
| Parquet | deferred | CSVがI/O/size bottleneck |
| DuckDB | deferred | cross-artifact analytical query増加 |
| BigQuery | **現時点では未使用** | local-first Artifactでは不足し、warehouse / large-scale analytical queryの明確な要件が成立した場合に再検討 |
| RDBMS | deferred | transactional state/catalog必要 |
| Cloud orchestration | deferred | local-firstでは運用困難 |
| Kubernetes | not needed | multi-node productization |
| Chart library | deferred | current rendererで表現不足 |
| ML framework | Post-MVP | modeling phase開始 |
| LLM provider SDK | optional/Post-MVP | grounded narrative requirement |

## 12. Post-MVP技術選定原則

新技術はArchitecture boundaryを壊さずAdapterとして導入する。StorageやLLM providerを導入しても、Observed / Derived / Predicted / Causal / LLM-generatedのevidence typeを混同しない。

## 13. Dependency pinning / reproducibility

Python dependenciesは`pyproject.toml`と`uv.lock`を正とする。frontendはpackage manifest/lockとbuild artifactを対応させる。

exact versionを書いたDesign docは、将来version更新されたとき「historical implementation value」と「current value」を区別する。

## 14. Frontend Artifactをrepoに保持する理由

Visualization Artifactは利用者がNode toolchainを意識せず開けることを重視する。MVPではbuild済みReact assetsをVisualization outputへ含める。最終`.gitignore`監査では必要なbuild assetまで誤ってignoreしないことを確認する。

## 15. Storage evolution policy

現在のCSV/JSON中心構成からParquet/DuckDBへ進む場合も、format変更を理由にArtifact contract/lineageを失わない。

移行時に要求するもの:

- schema version
- source manifest linkage
- deterministic conversion
- old artifact readability or migration path
- size/performance benefitの実測

## 16. ML / Causal技術選定はまだ固定しない

Post-MVP Modelingではscikit-learn、statsmodels、specialized time-series library等が候補になり得るが、現時点では要件が固まる前にhard dependencyへしない。

選定時はprediction accuracyだけでなく、time split、leakage、uncertainty、model lineage、deterministic seedを評価する。

Causalはprediction libraryのextensionとして雑に実装せず、assumption/evidence/effect intervalを扱える手法を別途選定する。

## 17. LLM技術選定はProvider非依存

LLMを導入する場合もCore deterministic calculationから分離する。Provider SDKはAdapterに閉じ込め、disabledでもCoreが動作することを要求する。

評価軸:

- grounding
- prompt/version/hash
- cache
- timeout/failure isolation
- cost/budget
- external data policy

## 18. Technology decisionを見直すtrigger

技術は一度選んだら永久固定ではない。次のような実測事象をtriggerとする。

- CSV処理時間/sizeが実用限界
- renderer表現力不足
- local-only運用がteam scaleを阻害
- API clientのdeprecated
- security requirement変更
- reproducibilityが現在toolingで維持できない

「新しいlibraryが出た」だけではmigration理由にしない。

