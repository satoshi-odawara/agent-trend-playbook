# ルール化(CLAUDE.md等)とツール化(skills等)の境界線

- 出典: agent-trend-data snapshots/2026-09-13 (metrics.json, generated_at
  2026-09-13T09:32:39Z。データ要望対応によりツール化指標が追加された版。
  当初の候補設定時点(07:11:53Z版)からの差分は下記「更新履歴」参照)
- 概要: 「人とAIエージェントの役割分担」「AIエージェントへの制約」を、各
  プロジェクトが指示文書(CLAUDE.md/AGENTS.md)への記述とツール化
  (skills/カスタムコマンド/hooks)のどちらでどこまで実現しているかを比較する
- 試す価値があると考える理由: サーベイを始めたばかりで比較の基準
  (ベンチマーク)がまだない。指示文書とツール化の役割分担は、今後どの
  プロジェクトを見ても繰り返し登場する軸になりそうなので、最初のテーマ
  として指標を作っておく価値がある
- 検証したい問い:
  - 指示文書が長い/短いプロジェクトと、ツール化(skills等)の活用度に
    傾向はあるか
  - 「ここは人間主導」のような役割分担を明示しているプロジェクトはどの
    程度あるか、明示の有無と指示文書のスタイルに関係はあるか
  - どのような制約は文書(ルール)に書かれ、どのような制約は仕組み
    (hooks/lint/CI等)に落とし込まれる傾向があるか

## 更新履歴
- 2026-09-14: データ要望(has_skills_dir等6フィールド)がradar側で対応され
  たため、ベンチマーク表・調査方針をツール化指標込みで更新。指標④
  (ツール化の程度)は手動読み込みではなくmetrics.jsonから直接取得できる
  ようになった。
- 2026-09-14(2回目): 手動検証(5件)で見つかった不整合をradar側に
  issue案として提示し、対応された(snapshots/2026-09-14,
  generated_at 2026-09-14T14:40:23Z、SCHEMA v1.2)。skills検出漏れが
  多数のリポジトリで修正され、`agent_doc_count`フィールドが新設され、
  `continuedev/continue`の`has_agent_instructions`が0→1に訂正された。
  ベンチマーク表・初期観察・不整合セクションを最新データで更新した。

---

## 調査方針

### 対象
2026-09-14の再収集(skills検出漏れの修正、`agent_doc_count`新設、
`continuedev/continue`の判定訂正を含む)により、20リポジトリ全件を対象に
できる。`has_agent_instructions: 1` の17件(表1)と `has_agent_instructions: 0`
の3件(表2)に分けて見る。

**表1: 指示文書あり(17件、doc文字数の降順。2026-09-14 14:40Z時点)**

| repo | segment | agent_doc_count | doc文字数 | mentions_boundaries | skills_count | custom_commands_count | has_hooks_config | mcp_servers_count |
|---|---|---|---|---|---|---|---|---|
| OpenHands/OpenHands | tool | 1 | 117,554 | 1 | 3 | 0 | 0 | 0 |
| browser-use/browser-use | tool | 1 | 49,587 | 1 | 0 | 0 | 0 | 0 |
| langchain-ai/langchain | tool | 1 | 38,490 | 1 | 0 | 0 | 0 | 2 |
| apache/airflow | adopter | 14 | 36,397 | 1 | 6 | 0 | 0 | 0 |
| vercel/next.js | adopter | 5 | 30,650 | 1 | 21 | 0 | 0 | 0 |
| colinhacks/zod | adopter | 1 | 21,939 | 1 | 2 | 0 | 0 | 1 |
| oven-sh/bun | adopter | 10 | 17,572 | 0 | 9 | 6 | 1 | 0 |
| astral-sh/ruff | adopter | 1 | 17,252 | 0 | 4 | 0 | 1 | 0 |
| remix-run/remix | adopter | 4 | 8,319 | 1 | 19 | 0 | 0 | 0 |
| getsentry/sentry | adopter | 5 | 7,491 | 1 | 28 | 3 | 0 | 2 |
| supabase/supabase | adopter | 4 | 6,554 | 1 | 22 | 0 | 1 | 1 |
| cline/cline | tool | 3 | 5,263 | 0 | 7 | 2 | 1 | 0 |
| continuedev/continue | tool | 1 | 4,302 | 0 | 1 | 0 | 0 | 0 |
| Significant-Gravitas/AutoGPT | tool | 13 | 3,822 | 0 | 10 | 0 | 0 | 0 |
| crewAIInc/crewAI | tool | 3 | 1,867 | 1 | 0 | 0 | 0 | 0 |
| nuxt/nuxt | adopter | 1 | 960 | 1 | 0 | 0 | 0 | 0 |
| sveltejs/svelte | adopter | 1 | 590 | 0 | 1 | 0 | 0 | 0 |

**表2: 指示文書なし(3件。`continuedev/continue`は2026-09-14に表1へ移動)**

| repo | segment | agent_doc_count | skills_count | custom_commands_count | has_hooks_config | mcp_servers_count |
|---|---|---|---|---|---|---|
| microsoft/autogen | tool | 0 | 0 | 0 | 0 | 0 |
| Aider-AI/aider | tool | 0 | 0 | 0 | 0 | 0 |
| yoheinakajima/babyagi | tool | 0 | 0 | 0 | 0 | 0 |

この3件は文書・ツール化とも全指標が0で、例外がなくなった(旧版は
continuedev/continueがskills_count=1の例外だったが、それ自体が
`has_agent_instructions`の誤判定によるものだった)。

### 初期観察(定量データのみ、検証前の仮説 → 手動検証・radar対応で修正済み)

以下は検証前(定量データのみ)の仮説。手動検証
(`verifications/2026-09-14-rule-vs-tool-boundary/log.md`)の結果、5件中
3件で修正が必要だった。その後radar側の対応(2026-09-14 2回目収集)で
データ自体も訂正され、手動検証の結果と整合するようになった。

- ~~OpenHands/OpenHands vs Significant-Gravitas/AutoGPT が好対照(文書のみ
  vs ツールのみ)~~ → **否定(手動検証・データ両方で確認)**。OpenHandsは
  実際には`.agents/skills/`に3件のskillsを持っており、旧metrics.jsonの
  `has_skills_dir=0/skills_count=0`はデータ不整合だった。radar側の修正で
  `skills_count=3`に訂正済み。「文書量とツール化の対極」という当初の
  フックは成立しない
- ~~`continuedev/continue`だけが例外(指示文書なし+ツール化あり)~~ →
  **前提自体が誤りと判明**。radar側の修正で`has_agent_instructions`が
  0→1に訂正された(実体は`extensions/cli/AGENTS.md`という非ルート階層の
  ファイルで、旧ロジックがルート直下しか検索していなかったための
  見落とし)。「指示文書なし」グループはmicrosoft/autogen, Aider-AI/aider,
  yoheinakajima/babyagiの3件のみとなり、文書・ツール化とも全指標が0で
  例外なし。「指示文書がない分をツールで肩代わりする」という仮説は
  この3件を見る限り支持する材料がない(単に何も整備していないだけ)
- **手動検証で新たに分かったこと(データの訂正後も有効)**: 文書量と
  ガバナンスの強さは無関係(nuxt/nuxtは最短の文書=948字で「AIによる
  公開文章作成・自律コントリビューションの禁止」という最も強い統制を
  実現していた)。また役割分担はプローズではなくツール側に実装される
  ことがある(cline/clineのrelease手順書に「人間に確認する」チェック
  ポイントが埋め込まれている)。この発見はradar側のSCHEMA.mdにも
  「文書量を統制の強さの代理指標として使わないこと」という注意書きとして
  反映された

### ベンチマーク指標(今回定義し、以後の比較にも使う)

metrics.json 側の指標(定量・既存データで取得可能):
1. **文書量**: `agent_doc_char_count` / `agent_doc_heading_count`
2. **言及トピックの幅**: `agent_doc_mentions_*` の8フラグのうち何個立って
   いるか(トピックカバレッジ)
3. **ツール化の程度**(2026-09-14、データ要望対応により追加): `has_skills_dir`
   / `skills_count` / `has_custom_commands` / `custom_commands_count` /
   `has_hooks_config` / `mcp_servers_count`

手動読み込みで新たにコーディングする指標(検証時に人間が実際に読んで
付与):
4. **制約の型分類**(指示文書の主要な記述を3分類し件数を数える)
   - 手順型: 「〜する時は〜の手順を踏む」
   - 禁止型: 「〜しない」「〜してはいけない」
   - 委譲型: 「ここは人間が判断/実施する」など役割分担の明示
5. **役割分担の明示度**: `mentions_boundaries` が実際にどの文言を指して
   いるかの確認 + 4の「委譲型」件数 + 「人間主導」「AI主導」等の直接的な
   文言の有無(0/1)

(旧版では④「ツール化の程度」を手動確認予定としていたが、データ要望対応
によりmetrics.json側の定量指標に格上げした)

### 手順
1. 「初期観察」で挙げた対照的なケースを中心に5件を選び、実際の
   CLAUDE.md/AGENTS.mdおよびskills/hooks関連ファイルを読む
   - `OpenHands/OpenHands`(doc最大・ツール化0の極端)
   - `Significant-Gravitas/AutoGPT`(doc薄い・skills最多の極端)
   - `continuedev/continue`(instructions自体なしだがskills=1の唯一の例外)
   - `cline/cline`(doc・ツール化とも中程度でバランス型)
   - `nuxt/nuxt`(doc極小・ツール化0、何もしていない対照群)
2. 各リポジトリについて指標4・5を手でコーディングし、比較表を作る
3. metrics.jsonの定量指標(1〜3)と手動指標(4・5)を突き合わせ、
   「検証したい問い」に対する仮説の当否を確認する
4. 自分自身(このplaybookリポジトリ)のCLAUDE.mdも同じ指標で採点し、
   他プロジェクトとの相対位置を確認する(実践知として記事に活かす)

### 気づいた不整合 — 対応状況(2026-09-14時点)

**解消済み(radar側のSCHEMA v1.1/v1.2で対応)**
- `has_skills_dir: 1` かつ `skills_count: 0` の食い違い
  (`getsentry/sentry`, `supabase/supabase`, `vercel/next.js`) →
  `.claude/skills/`規約しか見ていなかった検出漏れが原因。修正され、
  多くのリポジトリでskills_countが増加した(remix 0→19等。
  remix-run/remixで実在を確認済み、過大カウントではない)
- `OpenHands/OpenHands`の逆方向の不整合(`.agents/skills/`実在なのに
  `0/0`) → 同じ検出漏れの一環として修正され、`skills_count=3`に訂正
- `schema/SCHEMA.md`のツール化6フィールド未記載 → v1.1で反映済み
- 統制の強さを測る指標がない件(issue案#3) → 案C(新指標を追加せず注意書き
  のみ追記)で対応
- 指示文書のスコープ(モノレポでの部分適用)が測れない件(issue案#2) →
  `agent_doc_count`フィールド新設(v1.2)で部分対応。AutoGPT=13,
  apache/airflow=14など、分散度合いを示す代理指標として機能している
- 同日再収集時にスナップショットを上書きしていた件 → 運用側で
  「既存日付へは書き込みをスキップし警告、latest/のみ更新」という方針に
  修正済み(2026-09-14、口頭確認)。ただしこの挙動自体の文書化は
  まだ提案段階(agent-trend-data側での追記を提案中)

**未解消**
- skillsがシンボリックリンクで複数ディレクトリ(`.claude/`, `.agents/`,
  `.clinerules/`等)に共有されているケースで、カウント方法(シンボリック
  リンクを1件として数えるか、参照先の実体を数えるか)の一般的な扱いは
  未確認。ただし個別の検出漏れ自体は解消されている
- `Significant-Gravitas/AutoGPT`は`has_agent_instructions: 1`だが本文は
  `autogpt_platform/`限定(手動検証済み、判定自体は妥当)

---

## 記事構成のたたき台(仮説ベース、検証前)

検証結果次第で大きく変わる前提の仮案。

**タイトル案**
- 「文書の量では分からなかった — 5つのAIコーディングツールを実際に読んで
  見えた"ルール化とツール化"の境界線」

**想定章立て**(2026-09-14手動検証後に全面差し替え)
1. 導入: 定量データだけで立てた仮説(文書が薄い=統制がゆるい/ツール化ゼロ
   =文書に全部書いている)が、実際に読むと5件中3件で覆った、という
   検証プロセスそのものの発見から入る
2. 調査方法: metrics.jsonの定量指標 + 実ファイルを読んでの手動コーディング
   (制約の型分類・役割分担の明示度)の概要
3. 観察1: nuxt/nuxtの逆転 — 最短の文書(948字)が「AIによる公開文章
   作成・自律コントリビューションの禁止」という調査対象中最強の統制を
   実現していた。文書量とガバナンスの強さは無関係
4. 観察2: 役割分担は文書ではなくツールに実装されうる — cline/clineの
   release手順書(skills/commands)に「人間に確認する」委譲型チェック
   ポイントが埋め込まれている実例
5. 観察3: OpenHands/AutoGPTの当初の対比は測定ミスだった、という顛末
   — データだけを見て記事を書いていたら誤った結論になっていたという
   反省を含めて紹介(検証の価値そのものを示す)。この指摘はサーベイの
   収集側にフィードバックし、実際にデータが修正された/注意書きが
   追記されたところまで書けると、検証ログの蓄積という本リポジトリの
   資産価値を示す良い実例になる
6. 実践知: 自分のリポジトリ(agent-trend-playbook)への適用
   - このCLAUDE.mdの「検証は人間主導」のような役割分担の明示は、調査
     対象と比べてどう位置づくか
7. まとめ: 「文書量やツールの有無という表面的な指標ではなく、"何を"
   "どこに"制約として実装しているかを見るべき」という実践的な指針

**次のアクション**: 上記「手順」1〜4はIssue #3として完了し、
`verifications/2026-09-14-rule-vs-tool-boundary/log.md`に結果を記録済み
(2026-09-14、radar対応後の追記込み)。次はIssue #4(記事ドラフト作成)。
