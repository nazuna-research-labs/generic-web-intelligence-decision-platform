# ダッシュボード利用マニュアル

```yaml
document_id: dashboard_manual
document_type: manual
doc_ver: v1.2
language: ja
project_name: Generic Web Intelligence & Decision Platform
project_initiated_at: 2026-09-23 17:35:34 JST
created_at: 2026-10-03 08:14:10 JST
updated_at: 2026-10-05 22:57 JST
author: Nazuna Research
status: active
scope: Visualization dashboard operation and interpretation
note: 利用者の画面操作・値の読み方・監査手順に責任を限定し、内部開発メモを排除して利用シナリオを拡充
audience: dashboard users / analysts
responsibility: how to open, operate, interpret, validate, and share dashboard results; no development backlog
previous_public_version: dashboard_manual_ja_v1.1.md
mvp_completion_state: COMPLETE
current_package: 0.12.4
public_privacy_review: PASS — public copy removes personal author identity and replaces known site-specific example identifiers with non-identifying reserved examples
```

## 0. MVP COMPLETE後のDashboard利用上の現在状態

Dashboardの利用境界は維持され、MVPは`0.12.4`でCompletion Gateを通過した。

利用者が特に確認すべき現在の表示契約:

- React primary dashboardが主表示で、Static HTMLはsecondary audit copy。
- relative delta等、内部値がratioである値はpercent表示時に100倍する。例: `-0.3105 -> -31.05%`。
- M5でReact primaryのpercentage formatterを実Artifactと照合し、v0.12.4でcorrectness修正済み。
- ZERO / NOT_OBSERVED / UNAVAILABLE / UNKNOWN / NULLを見た目上同一化しない。
- GSC reporting lagは0ではなくcoverage/missingとして読む。
- Query粒度へ付与されるGA4値はpage-date contextであり、query単位のexact attributionではない。`NON_ADDITIVE_CONTEXT` / `contextual_only`等のstatusを確認し、query横断SUMしない。
- Dashboardで違和感がある場合は、表示だけを信用せず、Visualization manifest / input / original input Artifact / upstream Result Artifactへ戻って監査する。

M5完了後も、KPI layout、Quality/Coverage/Lineage raw JSONのaccordion化、count axis表示等はPost-MVP polish候補であり、現MVPのcorrectness不足を意味しない。

## 1. このマニュアルの対象

この文書は、Visualization Artifactの`dashboard.html`を開いて結果を読む利用者向けの詳細マニュアルである。

対象は開発者だけではない。GA4/GSCの生値を直接追わなくても、Dashboard上で「何が起きているか」「その数字を信用できるか」「元Artifactへどう戻るか」を判断できることを目的とする。

## 2. Dashboardがやること / やらないこと

DashboardはPresentation Layerであり、Analysis Engineではない。

**やること**

- upstream Result ArtifactをVisualization Input Contractへ投影
- KPI、Time Series、Comparison、Ranking等を表示
- unit / format / status / coverage / quality / lineageを表示
- original input Artifactをdownloadできるようにする

**やらないこと**

- YoYを再計算
- CTRを再集計
- average positionを勝手に再加重
- scoreやrecommendationを新規生成
- missing valueを0へ変換

Dashboard表示に違和感がある場合、まずupstream resultとformat contractを確認する。

## 3. 対応入力

MVPでは4系統を想定する。

1. GA4 Analysis Artifact
2. GSC Analysis Artifact
3. Integration Analysis Artifact
4. Generic Visualization Contractに適合する外部/未知Result Artifact

Source固有の専用rendererへ分岐せず、Adapterが共通Visualization Inputへ変換する。

## 4. Artifact構造

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

root直下は`dashboard.html`と`dashboard_data/`の2 entry。

### `dashboard.html`

primary React UI。通常はこちらを開く。

### `dashboard_static.html`

secondary/audit表示。React bundleの問題とデータ問題を切り分けるときにも使える。

### `visualization_input.json`

Rendererが受け取った共通入力。画面に表示された値の元を追うときの第一参照。

### `visualization_spec_snapshot.json`

どのcomponent/format/hintで描画したかを固定したsnapshot。

### `visualization_manifest.json`

input lineage、hash、spec等の監査情報。

### original input ZIP

Dashboard右上のdownload操作で取得する対象。Visualization自身をdownloadするのではなく、**入力元Artifactの原本ZIP**を取得する。

## 5. 画面上部 — Header

HeaderにはDashboard title、対象期間、grain、run情報等が表示される。

見る順序は「technical metadata」より「何のDataset/期間か」を優先する。run idやschema等は監査に必要だが、ビジネス判断の第一情報ではない。

Headerにはrun/input/spec等の識別情報が表示される。分析値を見る前に、意図したArtifactを開いているか確認する。

## 6. KPIセクション

KPIは選択された主要metricのsummary valueを示す。

### 6.1 読み方

1. metric名
2. current value
3. unit
4. comparisonがある場合はdelta/status
5. coverage/availabilityに問題がないか

数字だけを見て判断しない。特に`BASELINE_ZERO`, `BASELINE_UNAVAILABLE`, `CURRENT_UNAVAILABLE`, `INSUFFICIENT_COVERAGE`等のstatusがある場合、単純percent comparisonと意味が異なる。

### 6.2 ZEROとUNAVAILABLE

`0`は観測値0。`UNAVAILABLE`は値そのものを比較できない。UI上で似た空欄に見えても意味が違うため、status/qualityを確認する。

### 6.3 KPI layout

KPIは画面上部にカード群として並ぶ。カード数により複数行になるため、各カードのmetric名・単位・statusを個別に読む。

## 7. Time Series

Time Seriesは日次等の時間grainを視覚化する。

### 7.1 X軸

- 通常表示: 月次を中心に読みやすいtick
- zoom時: weekly -> daily相当へ細かくなる
- labelは必要に応じ斜め表示

### 7.2 Y軸

unitに応じてscale/tickを表示。概ね8 tick程度を目安にする。

count型metricの実値は件数として読む。軸tickは目盛り表示であり、Underlying resultの値型とは分けて解釈する。

### 7.3 Crosshair / Hover

pointerを系列上へ置くとcrosshairとhover emphasisで該当日/値を追える。複数系列が近い場合、線だけでなくtooltip/legendと合わせて読む。

### 7.4 Drag Zoom

横方向にdragして期間を拡大できる。長期間の異常点から局所を確認する用途。

### 7.5 Reset

zoom後はResetで元期間へ戻す。Filterやupstream Datasetそのものを変更する操作ではない。

## 8. Comparison

Comparisonはcurrent vs previous / YoY等のupstream Analysis resultを表示する。

### 8.1 absolute delta

```text
current - baseline
```

metric unitそのものの差。

### 8.2 relative delta / percent

内部resultがratioの場合、`-0.3105`は`-31.05%`として表示すべき。

この解釈は`value_type / unit / format_hint`契約に従う。ratio値`-0.3105`は意味として`-31.05%`である。

### 8.3 baseline zero

baselineが0の場合、relative deltaを無限大や任意%として表示しない。Analysis statusに従う。

## 9. Ranking Table

Rankingはmetric値に従ったrow orderingを表示する。

確認点:

- ranking metric
- dimension / grain
- current value
- status / coverage
- tieの扱い

RankingはDashboardで新規scoreを作っていない。upstream Analysis Artifactのranking rowを表示している。

## 10. Contribution / Share

Contributionは全体に対する寄与・shareを示す。

分母がavailableか、対象grainで加算安全なmetricかをupstream Analysisが判断している。Dashboardで再計算しない。

`UNAVAILABLE`を0として分母へ混ぜない。

## 11. Coverage

Coverageは「その数字がどの期間・Sourceでどの程度観測されているか」を読むための情報。

特にGSCはreporting lagがあり、requested end dateまでAPIが完全観測しているとは限らない。

```text
requested
resolved
observed
```

を区別する。

GSCの未到着日を0 click/0 impressionとして読むのは禁止。

## 12. Quality

QualityはResultを利用する際の注意を示す。

例:

- partial coverage
- unavailable metric
- source-only row
- reconciliation difference
- insufficient baseline

Quality flagは「悪いデータだから削除した」という意味ではなく、**残したデータをどう解釈すべきか**の情報。

## 13. Lineage

Lineageでは、このDashboardがどのAnalysis Artifact、Config/Spec、Manifestへ遡れるかを確認する。

調査時の順序:

```text
Dashboard value
-> visualization_input.json
-> Visualization manifest
-> input Analysis manifest
-> Processing / Extraction parent
```

Persistent lineageはproject-relative path/hashを使い、machine固有absolute pathを前提にしない。

## 14. GA4 Dashboardの読み方

GA4系ではsessions / views / engagement / events / key events等をSource Analysis resultとして読む。

GSC queryは含まれない。GA4単体可視化のためにIntegrationへ通す必要はない。

## 15. GSC Dashboardの読み方

clicks / impressions / CTR / average position等を扱う。

CTRは単純平均ではなく、必要なgrainでclicks/impressionsから意味的に正しく導出されたresultを使う。Average positionも単純加算metricではない。

## 16. Integration Dashboardの読み方

同一page/date等でGA4とGSC contextを並べて見る。

ここで重要なのは「検索queryがGA4 conversionを起こした」と断定しないこと。Integrationは観測系を安全に接続するもので、cross-source causal attributionではない。

## 17. Download

右上download iconはoriginal input Artifact ZIPを保存する。

ブラウザの通常`<a download>`挙動を使うため、OSのDownloads folderへ強制保存することはできない。保存先はブラウザ設定に依存する。

ZIPはupstream input Artifactとbyte/file-set整合するようdeterministicに生成され、hashをManifestで追跡する。

## 18. Static版との比較

React版とStatic版は同じVisualization Input/Specを使う。

Reactにだけ値が違う場合:

1. `visualization_input.json`確認
2. static版確認
3. React formatting/rendering確認

値そのものが両方違う場合はupstream Adapter/Input Contractを確認。

## 19. よくある誤読

### 「空欄 = 0」

誤り。statusを確認。

### 「GSC queryの上位 = GA4 conversionに寄与したquery」

誤り。exact attributionではない。

### 「DashboardのpercentがおかしいからAnalysis値がおかしい」

formattingだけが問題の可能性もある。raw inputとformat hintを照合。

### 「StaticとReactがあるので別計算」

誤り。同じcontractを描画する別renderer。

## 20. トラブルシュート

### Dashboardが開かない

- `dashboard.html`の存在
- `dashboard_data/react/`のbuild assets
- browser console
- `dashboard_static.html`が開けるか

### 値がない

- Visualization inputにrowがあるか
- upstream Analysis status
- coverage
- selected metric/spec

### Downloadできない

- `dashboard_data/<input>.zip`存在
- manifestにdownload metadataがあるか
- browser download setting

## 21. 表示を読むときの注意

DashboardはPresentation Layerであり、Analysis Artifactを再計算しない。

- KPIの並び方は画面幅・カード数で変わり得る。
- Count metricは件数として読む。
- Relative deltaがratioなら`-0.3105 = -31.05%`という意味。
- Raw JSON/technical metadataは監査情報であり、KPI自体とは別。
- 表示値に疑問がある場合はVisualization Inputと元Analysis Artifactへ遡る。

「表示上の見た目」と「上流Artifactの値」を混同しない。

## 22. Dashboardを開く

Artifact directoryへ移動して`dashboard.html`をbrowserで開く。macOSならFinderから開くか、必要に応じて次のようにする。

```bash
open output/visualization/.../visualization_<run_id>/dashboard.html
```

Dashboardはserver-side applicationではなくArtifactとして完結することを基本にしている。環境によってfile URL制限がある場合はStatic版とbrowser consoleを確認する。

## 23. 値の型と表示形式

Visualization Contractは少なくとも次の表示情報を区別する。

| 種別 | 例 | 表示時の注意 |
|---|---|---|
| count | sessions, clicks | 原則整数的に読む |
| ratio | CTR, relative delta | ratio/percent contractを確認 |
| duration | engagement time | unitを確認 |
| position | GSC position | 小さい値が上位、非加算 |
| status | unavailable等 | 数値とは別意味 |

unitを無視して数値だけ比較しない。

## 24. 期間比較を読む手順

1. Current periodを確認
2. Baseline periodを確認
3. 両期間のcoverageを確認
4. absolute deltaを見る
5. relative delta/statusを見る
6. Time Seriesで変化が期間末だけの一時点か確認
7. Ranking/Contributionでどのdimensionが寄与したか確認

この順序にすると、単一KPIのpercentだけで結論を出しにくい。

## 25. CTRの例

例えばclicks 100 / impressions 1,000ならCTR 10%。別rowがclicks 1 / impressions 10でCTR 10%でも、row CTRを単純平均する設計に依存しない。upstreamで分子/分母をsafe aggregationして導出したResultをDashboardが表示する。

## 26. Average Positionの例

GSC positionは「合計して意味があるmetric」ではない。Dashboardで棒グラフの高さが大きいほど良い、という一般的count metricの感覚を持ち込まない。metric description/unitを確認する。

## 27. Source-only rowを読む

IntegrationではGA4_ONLY / GSC_ONLYが存在し得る。これはデータ欠損を隠さず残した結果。

- GA4_ONLY: GA4では観測、GSC側に対応観測なし
- GSC_ONLY: GSCでは観測、GA4側に対応観測なし

source-onlyを「エラーrow」として自動除外しない。

## 28. Dashboard監査ワークフロー

数字へ疑問があるとき:

```text
1. UI表示値とstatusを記録
2. visualization_input.jsonで同resultを確認
3. static版でも同値か確認
4. visualization_manifest.jsonでinput Artifactを特定
5. input Analysis ArtifactのJSON/CSVを確認
6. Analysis manifest -> Processing parentへ遡る
7. 必要ならExtraction rawまで遡る
```

UIだけを見てsource dataを直接修正しない。

## 29. 画面サイズ / responsive利用

MVPはdesktopでの監査・分析を主とする。狭い画面ではtable/metadataの可読性が落ちる可能性があるため、詳細監査はdesktopを推奨する。狭い画面ではtable/metadataの可読性が落ちるため、詳細監査はdesktopを推奨する。

## 30. Dashboardを意思決定へ使う際の境界

DashboardはEvidenceを見やすくする。自動的に「この施策を実施すべき」と決定するものではない。

Observed trend、prediction、causal evidence、LLM narrativeが将来同じUIへ載る場合も、evidence typeを明示して混同しない。

## 31. Troubleshooting matrix

| 症状 | 最初に見るもの | 次に見るもの |
|---|---|---|
| HTMLが開かない | file existence | browser console / static版 |
| KPIが空 | status | visualization_input |
| percentが100倍ずれる | format_hint | React formatter |
| GSC終端が0 | coverage | observed end |
| Ranking重複 | upstream grain | Integration/Analysis result |
| Download失敗 | input ZIP existence | manifest/browser setting |
| ReactとStatic差 | same input確認 | renderer formatting |

## 32. 利用後に残すべき情報

分析結果を共有するときはScreenshotだけでなく、可能ならVisualization run idまたはoriginal input Artifactを併記する。後から同じEvidenceへ戻れることが重要。



## 33. Dashboardを読む基本順序

初見では次の順で読む。

1. Headerで対象Artifactを確認
2. KPIで全体量を把握
3. Time Seriesで時系列形状を確認
4. Comparisonで期間差を見る
5. Rankingで寄与の大きいdimensionを見る
6. Contributionで構成比を見る
7. Coverageで観測範囲を見る
8. Qualityで警告を確認
9. Lineageで元Artifactへ戻れることを確認

## 34. KPIの深い読み方

KPIは「大きいほど良い」とは限らない。metricごとの意味を読む。

- clicks / sessions / users等: amount/count
- CTR / rate: ratio
- average position: non-additive average-like metric
- relative delta: baselineに対する比率変化

値だけでなくstatusも合わせて見る。

## 35. Time Seriesで確認すること

- 長期的な増減
- 突発的なspike/drop
- 欠測区間
- source coverage差
- 比較期間の境界
- holiday/campaign等、外部要因を別途照合すべき地点

グラフは因果関係を証明しない。

## 36. Comparisonの読み方

### absolute delta

単位を維持した差。

### relative delta

基準期間に対する割合。baseline zero/unavailableでは通常の%比較が成立しないことがある。

### status

`BASELINE_ZERO`, `BASELINE_UNAVAILABLE`, `CURRENT_UNAVAILABLE`, `INSUFFICIENT_COVERAGE`等がある場合、数値だけを独立して読まない。

## 37. Rankingの読み方

Rankingは指定metric・period・dimensionに基づく並びであり、「重要度の総合点」ではない。

確認:

- metric
- direction
- period
- dimension
- coverage
- tie/order

## 38. Contributionの読み方

Contributionは対象totalの構成比。分母が何かを先に確認する。

異なるgrainやsource contextを横断して単純合算しない。

## 39. GA4 Dashboard利用例

1. KPIでsessions/users/events等を確認。
2. Time Seriesで変化点を確認。
3. Comparisonでprevious/YoY差を確認。
4. Rankingでpage/landing page等のdimensionを見る。
5. Quality/LineageでSource Processingまで追跡する。

## 40. GSC Dashboard利用例

1. clicks/impressionsを確認。
2. CTRとaverage positionを別metricとして読む。
3. query/page rankingを見る。
4. reporting lagをCoverageで確認。
5. recent missing daysを0と誤認しない。

## 41. Integration Dashboard利用例

1. MATCHED / GA4_ONLY / GSC_ONLY等の存在を理解する。
2. cross-source page metricを読む。
3. query grainではGA4 contextをnon-additiveとして扱う。
4. source coverage差を確認する。
5. LineageからIntegration Processing/Analysisへ戻る。

## 42. Percentageの具体例

Analysis Artifactのrelative ratio:

```text
0.125   -> 12.5%
-0.3105 -> -31.05%
1.0     -> 100%
```

`0.125%`と読むのではない。単位contractを確認する。

## 43. Missing/Unavailableの具体例

| 表示状態 | 解釈 |
|---|---|
| ZERO | 観測され値が0 |
| NOT_OBSERVED | 観測自体がない |
| UNAVAILABLE | metricを利用できない |
| UNKNOWN | 状態を確定できない |
| NULL | 値が存在しない/表現上null |

これらを一律0として比較しない。

## 44. Coverageの確認手順

1. requested periodを見る。
2. observed periodを見る。
3. source別coverageを見る。
4. partial/lagを確認する。
5. Comparison/Rankingの解釈可能性を判断する。

## 45. Qualityの確認手順

警告がある場合、対象metric/resultへ影響するかを確認する。Qualityは「Dashboardが壊れた」という意味ではなく、入力・coverage・contract等の注意を伝える。

## 46. Lineageの確認手順

- Visualization input
- Analysis Artifact
- Processing Artifact
- Extraction Artifact
- config/spec hash

まで辿れることを確認する。元データへ戻れない結果は監査性が低い。

## 47. Original Input ZIPの使い方

右上downloadから、Visualizationに渡したoriginal input Artifactを取得できる。

用途:

- 他人へ同じinputを渡す
- Dashboard表示との照合
- 再現検証
- LLMへ分析素材として渡す

## 48. Static版を使う場面

React版がprimary。Static版はaudit/fallback用途。

- JS executionを避けたい
- 同じcontractの静的表示を確認したい
- React表示と値が一致するか監査したい

Static版でも上流値を再計算しない。

## 49. Dashboardを共有するとき

共有する情報:

- Visualization run ID
- input Artifact名
- period
- metric
- relevant quality/coverage
- 必要ならoriginal input ZIP

Screenshotだけではlineageが切れるため、重要な判断ではrun情報も併記する。

## 50. Dashboard利用チェックリスト

- [ ] 正しいrunを開いた
- [ ] KPIのunitを確認した
- [ ] Time Seriesのperiodを確認した
- [ ] Comparison statusを確認した
- [ ] Rankingのmetric/dimensionを確認した
- [ ] Coverageを確認した
- [ ] Qualityを確認した
- [ ] Lineageを確認した
- [ ] Missingを0と誤認していない
- [ ] queryをGA4 conversionへexact attributionしていない
