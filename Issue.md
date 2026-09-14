# Issue.md

MVPの1本目(候補選定 → 検証 → ドラフト → 公開)を完全手動で通すためのタスク
リスト。方針は @CLAUDE.md を参照。GitHub Issue化する権限が使えない環境の
ため、このファイルでIssue相当の単位を管理する。書式は agent-trend-radar の
Issue.md に揃えている。

---

## 起票ルール

Issueは「実装型」「調査・検討型」いずれかのテンプレートで書く。

**実装型**(変更対象ファイル・作業内容が着手前から明確なもの)

```
## #N タイトル

**概要**: 何をするかを1〜3文で。

**変更対象ファイル**
- path(新規/修正)

**タスク**
- [ ] 具体的な作業項目

**完了条件**: 検証可能な達成基準。

**依存**: #M または なし
```

**調査・検討型**(発見事項の整理・方針検討で、着手時にファイルが確定
しないもの)

```
## #N タイトル

**概要**: 何が問題/検討事項かを1〜3文で。

**タスク**
- [ ] 検討・調査項目(未確定なら「着手時に定義する」のみでもよい)

**完了条件**: 着手時に別途定義する、または具体的な基準。

**依存**: #M または なし
```

**共通ルール**
- 見出しは`## #N タイトル`。番号は必ず独立した見出しを持ち、他Issueの
  箇条書きの中に内容をネストしない。
- 作業項目の見出し名は常に「タスク」に統一する。
- ファイル内は`#N`の昇順に並べる。
- 完了・中止・廃止したIssueは、元のセクションを書き換えず末尾に追記
  する形でクローズ記録を残す。
  - 完了/対応済み: `**対応内容(YYYY-MM-DD)**: 何をした/しなかったか、
    結論。`
  - 中止: `**中止理由(YYYY-MM-DD)**: なぜ中止したか。` +
    `**後続対応**: 中止に伴う新規Issueや成果物の再利用方針(該当する
    場合)。`

---

## #1 パイプラインのディレクトリ雛形を作成する

**概要**: CLAUDE.mdに記載のディレクトリ構成(candidates/, verifications/,
drafts/)を作成し、各工程で使う軽量なテンプレートを用意する。

**変更対象ファイル**
- candidates/TEMPLATE.md(新規)
- verifications/TEMPLATE.md(新規)
- drafts/.gitkeep(新規)

**タスク**
- [x] candidates/, verifications/, drafts/ ディレクトリを作成する
- [x] candidates/TEMPLATE.md(候補記載フォーマット)を用意する
- [x] verifications/TEMPLATE.md(検証ログフォーマット: 何を試したか/結果/
      所感/根拠スナップショット日付)を用意する

**完了条件**: 上記ディレクトリとテンプレートがリポジトリにコミットされて
いる。

**依存**: なし

**対応内容(2026-09-13)**: candidates/, verifications/, drafts/ を作成し、
candidates/TEMPLATE.md と verifications/TEMPLATE.md を追加してコミット
(8037493)。

## #2 候補1件の選定

**概要**: agent-trend-data/latest/metrics.json から試す価値がありそうな
候補を1つ選び、candidates/ に記載する。人間主導。

**タスク**
- [x] agent-trend-data/latest/metrics.json が更新されるのを確認する
- [x] 候補を1つ選定し candidates/YYYY-MM-DD-topic.md に記載する

**完了条件**: candidates/ に候補ファイルが1件作成されている。

**依存**: なし

**対応内容(2026-09-13)**: agent-trend-radarのデータ収集が完了し
metrics.json(20リポジトリ分)が揃った。「ルール化(CLAUDE.md等)と
ツール化(skills等)の境界線」を最初のテーマとして選定し、ベンチマーク
指標・調査方針・記事構成のたたき台とあわせて
candidates/2026-09-13-rule-vs-tool-boundary.md に記載した。

## #3 検証の実施

**概要**: 選定した候補を実際に手を動かして試し、結果を軽量な検証ログと
して残す。人間主導。

**タスク**
- [x] 候補を実際に試す
- [x] verifications/YYYY-MM-DD-topic/log.md に「何を試したか/結果/所感」
      と根拠にしたagent-trend-dataのスナップショット日付を記載する

**完了条件**: verifications/ に検証ログが1件作成されている。

**依存**: #2

**対応内容(2026-09-14)**: OpenHands/OpenHands, Significant-Gravitas/AutoGPT,
continuedev/continue, cline/cline, nuxt/nuxt の5件について実際のAGENTS.md/
CLAUDE.mdとskills/commands/hooksを読み、verifications/2026-09-14-rule-vs-tool-boundary/log.md
に記録した。定量データのみで立てた仮説(candidates/のOpenHands vs AutoGPT
対比等)が5件中3件で修正を要することが判明し、candidates/2026-09-13-rule-vs-tool-boundary.md
の初期観察・記事構成も更新した。

**追記(2026-09-14)**: 検証で見つかった不整合(skills検出漏れ、
指示文書のスコープ未計測、統制の強さの指標化)をradar側にissue案として
提示し、対応された(SCHEMA v1.2)。副次効果でcontinuedev/continueの
`has_agent_instructions`が0→1に訂正されたため、verifications/2026-09-14-rule-vs-tool-boundary/log.md
とcandidates/2026-09-13-rule-vs-tool-boundary.mdの両方を最新データで
再更新した。

## #4 記事ドラフト作成

**概要**: 検証ログとhubのデータを入力に、記事の初稿を drafts/ に書く。

**タスク**
- [ ] 検証ログ+hubのデータを参照し drafts/ に初稿を作成する

**完了条件**: drafts/ に記事初稿が1件作成されている。

**依存**: #3

## #5 公開

**概要**: 完成原稿を agent-trend-data/articles/ に反映する。

**タスク**
- [ ] drafts/ の原稿を完成させる
- [ ] agent-trend-data/articles/ に反映する(手動)

**完了条件**: agent-trend-data/articles/ に記事が追加されている。

**依存**: #4

## #6 振り返り

**概要**: 1本通してみてどの工程に時間がかかったかを整理し、次に自動化・
改善すべき箇所を判断する。

**タスク**
- [ ] 各工程の所要時間・詰まった点を記録する
- [ ] 次に自動化・改善すべき箇所を洗い出す
- [ ] 気づいたデータ要望があれば agent-trend-data/data-requests/pending/
      にテンプレートに沿って追記する(agent-trend-radarへの直接のIssue化
      はしない)

**完了条件**: 振り返りの記録が残っており、次の一手が明確になっている。

**依存**: #5
