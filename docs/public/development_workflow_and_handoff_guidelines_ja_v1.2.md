# 開発運用・引き継ぎ・ドキュメント管理ガイドライン

```yaml
document_id: development_workflow_and_handoff_guidelines
document_type: guideline
doc_ver: v1.2
language: ja
project_name: Generic Web Intelligence & Decision Platform
project_initiated_at: 2026-09-23 17:35:34 JST
created_at: 2026-10-03 08:14:10 JST
updated_at: 2026-10-05 23:32 JST
author: Nazuna Research
status: active
scope: project-independent development, handoff, and documentation rules
note: Lossless文書更新規約にManual purity、役割/境界/責任、Detail completeness、反復自己監査、Cold-start Handoff Auditを追加
responsibility: project-independent development, documentation, handoff, and self-audit rules
previous_public_version: development_workflow_and_handoff_guidelines_ja_v1.1.md
mvp_completion_state: COMPLETE
current_package: 0.12.4
public_privacy_review: PASS — public copy removes personal author identity and replaces known site-specific example identifiers with non-identifying reserved examples
```

## 0.1 v1.2追加事項 — README / 公開前Privacy / MVP closure

v1.1の全ルールを維持したまま、MVP completion後の運用で必要になった以下を追加する。

- repository root `README.md` の責務と編集方法
- 実装開始前にdir構造を設計し、責務単位で配置を固定して後工程の大規模リファクタリングを抑える原則
- public docs / images / license documentsを公開する前のspecific-identifier / privacy監査
- source非公開・docsのみ公開等、公開範囲が限定されるProjectでの境界管理
- Completion Gate成功後にMVP COMPLETEを宣言する順序

これらは既存のLossless / Manual purity / Role Boundary / Cold-start Handoff Auditを置換しない。

## 0. 目的

この文書は特定repoだけの手順ではなく、AI支援開発・人間開発の双方で再利用できる、実装・引き継ぎ・Markdown文書管理の共通規約である。

最重要原則は次の2つ。

> **ユーザーが明示的に要約・削減を要求しない限り、既存文書を勝手に要約しない。**

> **不可逆な情報欠損を起こす編集をデフォルト動作にしない。**

「update」「cleanup」「modernize」「restructure」「rename」は、要約命令ではない。

## 1. 開発変更の基本原則

### 1.1 Current state first

変更前にrepo、config、tests、Artifact、version、直近logを確認する。古い設計書だけを見てsourceを書き換えない。

### 1.2 One-shot / resume shell

複数ファイル・migration・release gate等は、再現可能な`.sh`へまとめることを推奨する。

標準Step:

```text
Preflight
Backup
Apply / Patch
Dependency / lock update
Format / Lint
Compile
Targeted tests
Full regression
Artifact / Contract gates
Cleanup
Success summary
```

Shell outputは人間が停止位置を即座に判断できるよう、`== Step ==`, `[FAIL]`, `[SUCCESS]`を明示する。

### 1.3 失敗時の扱い

失敗したという理由だけで全rollbackしない。

1. どこまで適用済みかを確認
2. failure causeを特定
3. active treeが壊れていないか確認
4. state-aware resume scriptを作る
5. 失敗Step以降を再開

rollbackが必要なのは、適用済み変更そのものが誤り・破壊的で、部分状態から安全に続行できない場合。

### 1.4 External / live resource節約

認証・API quota・実費・時間が絡むlive fetchは、unit/contract/synthetic/existing Artifactで検証できる範囲を先に閉じ、最後にまとめて行う。

### 1.5 実装時のディレクトリ構造設計

実装時は、**可能な限り最初からdir構造を十分に設計し、関連するsource、config、tests、scripts、assets、generated Artifactその他の構成要素が無秩序に散在しない状態を作る。**

目的は、機能追加を急ぐあまり一時的な配置を恒久化し、後工程で大規模なrename、move、import/path修正、config修正、test修正、docs修正をまとめて行う事態を極力減らすことである。

実装開始前、少なくとも次を確認する。

1. 新規要素がどの責務・subsystem・layerに属するか
2. 既存dirのどこへ置くのが最も自然か
3. source / config / tests / scripts / docs / assets / output / backup / cache等の役割が混ざらないか
4. public / private、tracked / local-only、source / generated Artifactの境界が配置上でも明確か
5. 将来同種の機能・source・profile・Artifactが増えた場合でも同じ構造を拡張できるか
6. root直下や無関係なdirへ「とりあえず」置くことで、後から整理作業を発生させないか

原則として、**先に配置責務とdir境界を決め、その後に実装する。**

特に新規subsystemや複数fileに跨る機能では、実装後に構造を考えるのではなく、最初にnamespace、dir、責務境界、依存方向、config配置、test配置、Artifact配置まで可能な範囲で決める。これにより、後々の構造整理・移動・参照更新・大規模リファクタリングの手間を極限まで減らす。

ただし、既存repoに既に確定した構造・互換性・Artifact contractがある場合、それらを無視して理想構造へ一括移行してはならない。既存構造の問題が後から判明した場合は、Current state first、影響範囲確認、backup、one-shot / resume shell、tests、Artifact / Contract gatesを通した明示的なmigrationとして扱う。

## 2. ドキュメント更新のLossless原則

### 2.1 更新の定義

デフォルトの更新式は次。

```text
Updated Document
= Existing Meaningful Information
+ New Information
+ Changed Specifications
+ New Decisions
+ Operational Knowledge
+ Useful Historical Context
```

**Updated Document = shorter summary** ではない。

### 2.2 原則削除禁止

明示指示がない限り、意味のある情報を「読みやすくするため」「長いから」「古そうだから」という理由で削除しない。

特に次は保存優先度が高い。

- Requirements
- Architecture / Design
- Roadmap
- Handoff
- ADR / decision history
- Operational recovery knowledge
- Acceptance / Exit Criteria

### 2.3 削除を許容できるもの

原則として以下に限定する。

- 完全な意味重複
- 明確な誤記
- 明確な別プロジェクト混入
- security上残してはいけないsecret
- ユーザーが削除を明示した情報

判断が曖昧なら消さず、`旧仕様 / 現在仕様 / 変更理由 / 移行履歴`として残す。

### 2.4 Summaryは別Artifact

Summaryを作る場合、詳細版をSummaryで置換しない。

```text
detailed_document.md   # Source of Truth / detail
summary_document.md    # navigation / overview
```

Summaryがdetailを上書きする設計は禁止。

### 2.5 不可逆操作の前にBackup

rename / overwrite / mass rewrite / deleteの前に、元文書を復元可能な場所へ保存する。

最低限:

- original files
- file mapping
- original hash
- timestamp

大規模更新では、旧原文を保存付録として新doc内部へ保持する方法も有効。

## 3. Markdownファイル命名規則

Version管理対象の正式docは原則次。

```text
<semantic_name>_<locale>_v<major.minor>.md
```

例:

```text
master_roadmap_ja_v3.1.md
system_requirements_ja_v3.1.md
dashboard_manual_ja_v1.2.md
project_handoff_ja_v2.2.md
```

規則:

- semantic nameは英語
- lowercase
- snake_case
- localeを付ける（このProjectでは`ja`）
- versionはYAML `doc_ver`と一致
- project名prefixを毎回付けない
- file nameにスペース、日本語、日時を混在させない（snapshot文書など明示理由がある場合を除く）
- 同じsemantic documentの新versionを作るときはversionを上げる

### 3.1 README例外

repository root `README.md` はlanding pageであり、versioned filenameにしない。詳細設計の保管場所にも使わない。

## 4. H1とMarkdown構造

### 4.1 1 document = 1 H1

原則1ファイルにH1は1つ。複数H1が存在する旧文書は、内容を削らずH2以下へ再階層化するか、旧版全文を保存付録として保持する。

### 4.2 H1は利用者言語

日本語docではH1を日本語の正式タイトルにする。ファイル名だけ英語で管理する。

### 4.3 H1直下のYAML配置

**H1とYAMLコードフェンスの間には必ず空行を1行置く。**

正:

````markdown
# 日本語の文書タイトル

```yaml
document_id: example
doc_ver: v1.0
```
````

誤:

````markdown
# 日本語の文書タイトル
```yaml
...
```
````

このblank lineは見た目の好みではなく、文書生成規約として固定する。

### 4.4 Heading階層

- H1: 文書タイトルのみ
- H2: 大章
- H3: 章内の責務・トピック
- H4以下: 必要時のみ

階層整理のために本文を要約・削除しない。

## 5. YAML metadata標準

全doctypeをhard-coded enumへ閉じ込めない。共通core + doctype別optional fieldとする。

### 5.1 Core metadata

| Field | 推奨 | 意味 |
|---|---|---|
| `document_id` | 必須 | 文書の安定ID |
| `document_type` | 必須 | 文書の役割 |
| `doc_ver` | 必須 | ファイル名versionと一致 |
| `language` | 必須 | `ja`等 |
| `project_name` | 推奨 | Project正式名 |
| `project_initiated_at` | Project docで推奨 | Project開始時刻 |
| `created_at` | 必須 | 初版作成時刻 |
| `updated_at` | 必須 | 最終更新時刻 |
| `author` | 必須 | 作成主体 |
| `status` | 推奨 | draft / active / superseded等 |
| `scope` | 推奨 | 対象範囲 |
| `note` | optional | 補足 |

必要に応じ、`source_of_truth`, `supersedes`, `related_docs`, `preserved_source_sha256`, `audience`, `owner`等を追加してよい。

### 5.2 Timestamp

新規timestampは原則:

```text
yyyy-mm-dd hh:mm:ss JST
```

既存`created_at`が存在する場合は、後から都合のよい時刻を捏造して書き換えない。Legacy documentでdate precisionしかない場合は元値を保存し、`updated_at`から標準formatを適用する。

`project_initiated_at`や`author`などProject固定値は、明示変更がない限り維持する。

## 6. document_typeの抽象カテゴリ

`document_type`は拡張可能なstringとし、次は推奨カテゴリ例であってclosed enumではない。

| Type family | 役割 | Preservation |
|---|---|---|
| `requirements` | 要件・契約・Acceptance | highest |
| `roadmap` | 時系列計画・現在地 | highest |
| `architecture` / `design` | 構造・境界・trade-off | highest |
| `handoff` | 作業継続・状態移送 | highest |
| `manual` | 操作・理解・復旧 | high |
| `runbook` / `operations` | 実運用・障害対応 | high |
| `technology_selection` | 技術選定と根拠 | high |
| `adr` / `decision_record` | 意思決定履歴 | highest |
| `reference` | 辞書・仕様参照 | high |
| `summary` | 詳細版への入口 | source detailを置換不可 |
| `policy` / `guideline` | 共通規約 | high |
| `release` | release note / gate | medium-high |
| `concept` | 目的・価値・全体像 | high |
| `other` | 新しい役割 | case-by-case |

将来新doctypeが必要なら追加する。既存カテゴリへ無理に押し込まない。

## 7. doctype別の推奨構成

### 7.1 Requirements

```text
位置づけ / Source of Truth
目的 / Scope
Architecture principles
Functional requirements
Non-functional requirements
Data / API / Artifact contracts
Failure / quality semantics
Acceptance criteria
Future requirements
Historical decisions
```

### 7.2 Roadmap

```text
目的
現在地 / progress table
Milestones / phases
Dependencies
Exit criteria
Risks / deferred items
Post-MVP roadmap
Historical baseline
```

### 7.3 Design / Architecture

```text
Problem / purpose
Boundary
Principles
Components
Data flow
Contracts
Failure handling
Trade-offs / prohibited patterns
Extension hooks
Acceptance
Historical design changes
```

### 7.4 Handoff

HandoffはSummaryではない。元チャットを読めない相手が継続できる密度を目標にする。

```text
Source of Truth
Current exact state
Completed work
In-progress / failed state
Architecture contracts
Known issues
Exact next actions
Commands / scripts
Artifacts / fixtures
Do-not-break rules
Historical context
```

Project固有Handoffと、Project非依存の共通workflow guidelineは分離する。

### 7.5 Manual

```text
対象ユーザー
Prerequisites
Quick start
Concept needed for operation
Detailed procedures
Command/UI reference
Inputs / outputs
Interpretation
Failure recovery
Troubleshooting
FAQ / safety boundaries
```

Manualは「機能名一覧」だけでは不十分。初見ユーザーが実際に操作できるまで書く。

### 7.6 Technology Selection

```text
Selection principles
Current stack
Technology -> role mapping
Why selected
Trade-offs
Rejected/deferred alternatives
Version/pinning policy
Reconsideration trigger
Future candidates
```

現在採用済みと将来候補を混ぜない。

### 7.7 Summary

```text
詳細版へのlink
Purpose
Current status
Key concepts
Major flow
Next actions
```

Summaryは意図的に短くしてよい唯一のdoctypeの一つ。ただし詳細版を残すことが前提。

### 7.8 README

READMEはrepository landing page。

```text
What is this?
Current status
Quick start
Major commands
Docs index
License / caution if necessary
```

READMEへ要件定義書・Handoff・全コマンド解説を詰め込まない。詳細docsへ誘導する。

## 8. 今回の12文書へのmapping例

| File | document_type | 主目的 |
|---|---|---|
| `master_roadmap_ja_v3.1.md` | roadmap | MVP〜Post-MVP全体計画 |
| `system_requirements_ja_v3.1.md` | requirements | 全体要件 / Contract |
| `concept_overview_ja_v2.1.md` | concept | System全体像 |
| `project_handoff_ja_v2.2.md` | handoff | Project継続 |
| `development_workflow_and_handoff_guidelines_ja_v1.2.md` | guideline | Project非依存運用規約 |
| `roadmap_summary_ja_v1.1.md` | summary | Roadmap入口 |
| `technology_selection_ja_v1.1.md` | technology_selection | 技術選定 |
| `dashboard_manual_ja_v1.2.md` | manual | Dashboard操作・読解 |
| `user_manual_ja_v1.2.md` | manual | CLI / Pipeline利用 |
| `system_functions_and_data_processing_details_ja_v1.2.md` | reference | 全機能・加工詳細 |
| `system_functions_and_data_processing_summary_ja_v1.2.md` | summary | 全機能詳細への入口 |
| `visualization_generic_presentation_engine_design_ja_v2.1.md` | design | Visualization設計 |
| root `README.md` | repository_landing | 最短の入口 |

このmappingは例であり、future document typeを制限しない。

## 9. 文書品質Gate

大量更新時は機械Gateを推奨する。

- filename regex
- filename version == YAML `doc_ver`
- H1 count == 1
- H1の利用者言語
- H1直後: blank line -> YAML fence
- metadata required field非空
- original source hash / preserved appendix確認
- unrelated project keyword混入scan
- detailed document最低情報量
- summaryがdetailを削除していないこと
- broken link / stale filename scan
- `.gitignore`等、対象外file hash非変更確認

## 10. AI / LLMによる文書編集時の追加ルール

AIは「綺麗にする」「読みやすくする」を理由に情報を勝手に落としやすい。そのため以下を明示的に禁止する。

- 長い文書を暗黙にsummary化
- 未確認事項をDONE扱い
- 旧仕様を理由説明なく削除
- 別Projectの情報を似た構造だから流用
- current repo未確認でversion/test数を断定
- manualを機能一覧だけで済ませる
- READMEへすべてを押し込む

不確実な状態は`PENDING / UNVERIFIED / HISTORICAL`等として明示し、事実を捏造しない。

## 11. 文書Versioningの運用

### 11.1 Major / Minor

基本形は`v<major>.<minor>`。

- Major: 文書のscope・責務・Source of Truth関係が大きく変わる
- Minor: 同じ責務の中で新機能・新判断・新運用を追加

誤字修正だけで毎回versionを上げるかはProject policyで決めるが、ファイル名と`doc_ver`の不一致は許容しない。

### 11.2 Supersedeと履歴

新versionが旧versionを置き換える場合、`supersedes`をYAMLに持たせてよい。旧本文に後から意味がある場合はbackupだけでなく保存付録やversion controlでも追跡可能にする。

### 11.3 Snapshot文書

時点固定が文書の本質である場合は日時をfile nameへ入れる設計も可能。ただし恒久的なRequirements/Roadmap/Manualへ無条件に日付suffixを付けない。

## 12. Rename / Mass Rewriteの手順

1. 現在のfile listを取得
2. old -> new mappingを確定
3. directory全体をbackup
4. original hashを記録
5. 新fileを生成
6. internal links / referencesを更新
7. canonical file setをGate
8. rollback pathを確認

「新しい名前で書いたから旧ファイルを消す」ではなく、**backupとmappingが成立してからactive treeを整理する**。

## 13. Cross-project contamination防止

別Projectの文書をformat referenceとして使う場合、本文内容をsource factsとして扱わない。

安全な手順:

```text
Reference document
  ├─ layout
  ├─ section granularity
  ├─ table style
  └─ metadata style
        ↓
Target document structure only

Target project facts
  -> target document content
```

別Project固有のproduct名、phase名、runtime、technology、status、version、constraintがTargetへ残っていないかkeyword scanする。

## 14. Project Handoffと共通Guidelineの分離

Handoffを1ファイルに何でも入れると、他Projectで再利用できる運用知識とProject固有状態が混ざる。

したがって原則:

- `project_handoff...`: Project固有のcurrent state / exact next action
- `development_workflow_and_handoff_guidelines...`: Project非依存の実装・文書・handoff規約

新しいProjectへ移るとき、共通Guidelineは再利用できるが、Project Handoffのfactsは持ち込まない。

## 15. Manualの品質基準

Manualは最低でも以下の問いに答えられる必要がある。

1. 何を準備するか
2. 最初の一回をどう実行するか
3. どの入力を使うか
4. 何が出力されるか
5. 成功をどう確認するか
6. 失敗したらどこを見るか
7. 再実行で何を壊さないか
8. 値をどう解釈するか
9. 何をしてはいけないか

機能名の列挙だけならManualとは呼ばない。

## 16. Handoffの品質基準

次スレッドや別担当者が過去チャットを一切読めないものとして書く。

最低限:

- current version/status
- current tree / important paths
- completed / pending
- exact failure point if any
- current scripts and whether executed
- verified artifacts / fixtures
- authoritative contracts
- next command/action
- forbidden regressions

「前のスレを見れば分かる」はHandoff failure。

## 17. Lossless Gateの考え方

機械的な文字数比較だけでは十分ではないが、mass rewriteでは有効なearly warningになる。

推奨Gate:

- preserved source appendix hash一致
- old sourceの重要keyword / section存在
- Post-MVP / future sections残存
- Acceptance Criteria残存
- historical decisions残存
- output detail docがsummaryより十分大きい
- original bodyより極端に短くなった場合はfailまたはhuman review

## 18. READMEの品質基準

READMEは短くてよいが、短いから情報を捨ててよいのではない。**情報の置き場所を分離する**。

READMEから削った詳細はRequirements/Manual/Handoff等に存在することを確認する。

目安:

```text
README: discoverability / quick start
Docs: depth / contracts / history / operation
```

## 19. ドキュメント生成`.sh`の推奨Gate

Docs mass update scriptでは以下を自動化する。

```text
preflight repository identity
backup docs + README
current timestamp generation
current package/status detection
write documents
filename/version validation
H1/YAML placement validation
lossless source hash validation
cross-project contamination scan
minimum content-size sanity
README brevity sanity
target-excluded file hash check
final manifest/log
```

これにより「書けた」ではなく「壊していない」までを一回の操作で確認する。



## 20. Document Responsibility Boundary Rule

各文書は、作成前に最低限次を定義する。

```text
Purpose
Audience
Responsibility
Included Scope
Excluded Scope
Source of Truth Responsibility
Expected Detail Level
Completion Criteria
```

「関連するから」という理由だけで別doctypeの責務を混ぜない。

### 20.1 Manualの責任

Manualは利用者が操作・判断・復旧するための文書。

含める:

- prerequisites
- setup
- commands
- options
- inputs/outputs
- normal workflows
- examples
- interpretation
- user-facing limitations
- recovery
- troubleshooting
- safety

含めない:

- 開発TODO
- fix予定
- 次versionで直す予定
- refactor memo
- internal patch history
- test failure diary
- implementation backlog

利用上必要な「現在の制約」は書いてよいが、修正予定メモへ変質させない。

### 20.2 Roadmapの責任

予定、未完了、改善候補、dependency、milestone、exit criteriaはRoadmapへ置く。

### 20.3 Handoffの責任

作業継続に必要なcurrent state、失敗状態、next action、exact command、do-not-break contractを持つ。

### 20.4 Design/Architectureの責任

なぜその構造か、component/data flow/contract/trade-off/failure policy/future extensionを持つ。

### 20.5 Referenceの責任

網羅性と追跡性を優先する。実装・Data Contract・config・Artifact・symbol/test inventory等を深く記述できる。

### 20.6 Summaryの責任

詳細版へのnavigationと全体俯瞰。詳細版を置換しない。

## 21. Manual Purity Rule

Manualは「人が使うための文書」である。

Manualに開発者の内部事情を混ぜると、読者は現在利用できる機能、既知制約、将来予定を区別できなくなる。

したがって:

```text
Current usage        -> Manual
Current limitation   -> Manual（利用判断に必要な場合）
Fix plan             -> Roadmap/Handoff
Patch history        -> Handoff/Release note
Implementation detail-> Reference/Design
```

### 21.1 詳細を削る理由にはならない

Manual purityは「Manualを短くする」規則ではない。

むしろ利用者が単独で操作できるよう、役割内情報は過剰なくらい詳しくてよい。

LLMへ投入して質問する利用形態も考慮し、操作例・分岐・失敗時復旧・FAQを十分に持たせる。

## 22. Detail Completeness Rule

文書名やdoctypeが`details / full / complete / reference / specification`相当の責務を持つ場合、章見出しを一通り並べただけで完成扱いしない。

最低限:

- concept
- inputs
- outputs
- transformations
- contracts
- state
- failure modes
- edge cases
- forbidden behavior
- lineage
- configs
- examples
- verification
- implementation mapping

まで追える密度を要求する。

行数自体はDefinition of Doneではないが、対象範囲に対して不自然に薄い場合は「本当にこの密度で全部説明できているか」を再評価する。

## 23. Iterative Document Self-Audit

文書生成は一回書いて終了しない。

```text
Draft
  ↓
Role / Boundary Audit
  ↓
Coverage Audit
  ↓
Detail-density Audit
  ↓
Lossless Audit
  ↓
Cross-document Responsibility Audit
  ↓
Cold-start Audit（Handoff系）
  ↓
不十分なら改稿
  ↓
再監査
  ↓
必要なら再改稿
  ↓
全Gateを満たして完成
```

### 23.1 Role / Boundary Audit

- そのdoctypeが担当すべき内容か
- 別文書のTODO/内部事情を混ぜていないか
- Readerが誤解する責任混在がないか

### 23.2 Coverage Audit

- userが要求したscopeを全部含むか
- feature/layer/command/artifactの取りこぼしがないか

### 23.3 Detail-density Audit

問い:

> この文書の役割に対して、この密度で本当に単独利用可能か？

不十分なら増やす。

### 23.4 Lossless Audit

- 旧版の意味ある情報が消えていないか
- 要約が暗黙に発生していないか
- backup/hash/appendixで復元可能か

### 23.5 Cross-document Responsibility Audit

- ManualへRoadmapを混ぜていないか
- SummaryがDetailを代替していないか
- Handoffへcurrent continuationに必要な情報があるか

## 24. Cold-start Handoff Audit

Handoffは「量が多い」だけでは合格しない。

元スレッドを一切見られない新しい担当者/新LLMが、Handoff必須docsだけを読んで質問なしで作業再開できるかを監査する。

最低確認項目:

```text
現在version
現在repo構造
各Milestone完了状態
未完了状態
直近の失敗内容
未実行/結果未確認のscript
成功済み/未確認の明確な分離
次にやること
実行コマンド
Source of Truth優先順位
絶対壊してはいけないArchitecture Contract
Data semantics
Artifact/Manifest contract
Pipeline profiles
resume/recovery方法
live fetch policy
.sh開発運用
MVP残作業
Post-MVP全体計画
README/docs運用
既知の改善事項
```

### 24.1 合格条件

新担当者が少なくとも次を一意に答えられること。

1. いま何がDONEか。
2. 何が未確認か。
3. 次に実行するexact commandは何か。
4. そのcommandが失敗したらどのstateを保持するか。
5. 何を絶対に壊してはいけないか。
6. どのdoc/sourceを優先するか。
7. MVP後に何が残っているか。

### 24.2 「1mmもブレない」と断言する条件

Cold-start Auditを実施していない状態では、完全引き継ぎを断言しない。

Auditで不足が見つかった場合:

```text
Handoff/Requirements/Roadmap/Designを補強
-> 再Cold-start Audit
-> 不足があれば再補強
-> PASSまで反復
```

## 25. Handoff Packageの役割分担

推奨:

```text
project_handoff
  = immediate continuation / current state / next commands

system_requirements
  = normative requirements/contracts

master_roadmap
  = milestone/history/future plan

concept/design
  = architecture and rationale

development_workflow_and_handoff_guidelines
  = project-independent working/document rules
```

Handoff一冊へ全部コピーして膨張させるだけではなく、参照先と優先順位を明確にする。ただしHandoffだけで「どこを読めばよいか」まで判断できる必要がある。

## 26. 文書更新時の最終質問

完成宣言前に最低限、自分へ次を問う。

- 意味なく短くしていないか。
- 旧情報を復元不能にしていないか。
- Readerの役割に不要な内部事情を混ぜていないか。
- 必要な操作/仕様を薄くしていないか。
- 詳細版と名乗る密度か。
- Summaryが詳細版を置換していないか。
- Handoffはcold start可能か。
- READMEは逆に簡潔な入口になっているか。

一つでもNo/不明なら完成扱いしない。

## 27. Repository root `README.md` 作成・編集規則

### 27.1 READMEの責務

repository root `README.md` は**landing page / public entry point / navigation**である。

READMEへ詳細Requirements、全CLI reference、全Architecture rationale、全Roadmap、全Handoff情報を集約しない。詳細をREADMEへコピーすると、versioned docsとの二重Source of Truthになりdriftしやすいため禁止する。

READMEの基本責務:

1. 公開範囲・重要な利用条件を冒頭で明示する。
2. Projectが何であるかを数段落以内の超概要で説明する。
3. Architecture overview image等、入口として有効なpublic visualがあれば導線を置く。
4. `docs/public/`の主要文書へ役割付きで案内する。
5. License / Notice / Termsの超概要と正本への導線を置く。
6. 必要なら短いEnglish Summaryを末尾へ置く。
7. current public/non-public boundaryを明示する。

### 27.2 READMEへ書かないもの

明示要求がない限り、次はversioned docsへ置く。

- 実装技術の詳細
- package/module inventory
- 全CLI option
- migration history
- debugging log
- internal TODO
- one-shot `.sh`手順の全履歴
- Requirements全文
- Handoff全文
- Post-MVPの詳細Exit Criteria
- credential / actual property / actual site identifier

READMEで必要なのは「何を見るべきか」であり、「すべてをREADMEだけで完結させること」ではない。

### 27.3 README更新時のLosslessの考え方

READMEはDetail documentではないため、versioned detail docsと同じ「行数を必ず増やす」基準を機械適用しない。

ただし、README固有の意味ある情報――公開制限、project identity、主要導線、license導線、language summary等――を理由なく削除してはいけない。

技術詳細がREADMEから削除される場合でも、その情報が必要ならversioned docsに正本が存在することを確認する。READMEを簡潔化した結果、Project全体として情報が消える状態は禁止する。

### 27.4 versioned docs変更との同期

versioned filenameを変更した場合、README内のdocument linkを同じ変更セットで更新する。

Gate:

```text
README link target exists
README does not point to superseded active filename
README does not become a second Requirements/Handoff
public/non-public boundary is current
license links exist
public image links exist
```

### 27.5 Privacy / Specific Identifier

Public READMEには以下を置かない。

- 個人名・個人連絡先
- 顧客/依頼者の識別情報
- 実Site URL / Property ID / account identifier
- credential / token / client secret
- local absolute path
- private repository URL
- 非公開Sourceへの直接内容引用

Projectの公開理由や開発経緯を説明する必要がある場合も、個人・顧客を特定しない表現を使う。

### 27.6 Public sourceが制限されるProject

Source code公開の許可がない場合、READMEで公開可否を曖昧にしない。

例:

```text
Documentation / public images: public distribution scope
Source code: not authorized for public release
```

この境界はLicense / Notice / Termsおよび実際のGit push対象と一致させる。

## 28. Public Documentation Privacy / Specific-Identifier Audit

public docs、README、public images、license-related documentsを外部公開する前に、内容とmetadataを監査する。

最低scan対象:

- personal name / email / phone / social account
- customer / requester / company-specific name
- actual domain / site URL / property ID / account ID
- credential / token / secret
- local absolute path / username
- private repository / private issue URL
- raw Artifact中の特定可能値
- image metadata / EXIF / ancillary textual metadata

説明例として特定サイトを使う必要がある場合は、reserved / non-identifying example（例: `example.invalid`）へ置換し、意味論だけ維持する。

Privacy cleanupはLossless原則の例外になり得るが、**なぜ削除・匿名化したかをpublic-safeな形で記録**し、システム要件・契約そのものまで削除しない。

## 29. MVP COMPLETE宣言とPublicationの分離

MVP COMPLETE、Public Release、Source Release、Docs Releaseは別状態として管理する。

```text
MVP Completion Gate PASS
  -> MVP COMPLETE
  -> .gitignore / privacy / packaging audit
  -> docs/images publication may proceed if authorized
  -> source publication only if separately authorized
```

MVPが完成しただけでSource公開許可を推論してはいけない。

