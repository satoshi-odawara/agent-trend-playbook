# ルール化(CLAUDE.md等)とツール化(skills等)の境界線

- 出典: agent-trend-data snapshots/2026-09-13 (metrics.json, generated_at 2026-09-13T07:11:53Z)
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

---

## 調査方針

### 対象
`has_agent_instructions: 1` の15リポジトリを中心に見る(下表)。特に
`agent_doc_mentions_boundaries: 1` の8件(役割分担に触れている可能性が
高い)を優先的に深掘りする。

| repo | segment | doc文字数 | 見出し数 | mentions_boundaries | mentions_tool_usage |
|---|---|---|---|---|---|
| apache/airflow | adopter | 36,390 | 31 | 1 | 1 |
| langchain-ai/langchain | tool | 38,490 | 72 | 1 | 1 |
| vercel/next.js | adopter | 30,650 | 60 | 1 | 1 |
| browser-use/browser-use | tool | 49,587 | 129 | 1 | 1 |
| colinhacks/zod | adopter | 21,939 | 17 | 1 | 1 |
| astral-sh/ruff | adopter | 17,252 | 25 | 1 | 1 |
| oven-sh/bun | adopter | 17,572 | 20 | 0 | 0 |
| supabase/supabase | adopter | 6,554 | 7 | 1 | 1 |
| getsentry/sentry | adopter | 7,491 | 24 | 1 | 1 |
| remix-run/remix | adopter | 8,319 | 10 | 1 | 1 |
| OpenHands/OpenHands | tool | 117,554 | 32 | 1 | 1 |
| cline/cline | tool | 5,263 | 6 | 0 | 0 |
| crewAIInc/crewAI | tool | 1,867 | 4 | 1 | 0 |
| nuxt/nuxt | adopter | 960 | 1 | 1 | 1 |
| sveltejs/svelte | adopter | 590 | 2 | 0 | 1 |

(`has_agent_instructions: 0` の5件 — microsoft/autogen, continuedev/continue,
Aider-AI/aider, yoheinakajima/babyagi, Significant-Gravitas/AutoGPT(※要確認、
下記「不整合」参照) — は「そもそもルール化していない」比較対象として
別枠で扱う)

### ベンチマーク指標(今回定義し、以後の比較にも使う)

metrics.json 側の指標(定量・既存データで取得可能):
1. **文書量**: `agent_doc_char_count` / `agent_doc_heading_count`
2. **言及トピックの幅**: `agent_doc_mentions_*` の8フラグのうち何個立って
   いるか(トピックカバレッジ)

手動読み込みで新たにコーディングする指標(検証時に人間が実際に読んで
付与):
3. **制約の型分類**(指示文書の主要な記述を3分類し件数を数える)
   - 手順型: 「〜する時は〜の手順を踏む」
   - 禁止型: 「〜しない」「〜してはいけない」
   - 委譲型: 「ここは人間が判断/実施する」など役割分担の明示
4. **ツール化の程度**(リポジトリの実ファイルを確認)
   - skills/カスタムslash commandsの数(例: `.claude/skills/`, `.claude/commands/`)
   - hooksの有無(pre-commit、CI上のlint/testゲート等)
5. **役割分担の明示度**: 3の「委譲型」件数 + 「人間主導」「AI主導」等の
   直接的な文言の有無(0/1)

### 手順
1. 上表の優先リポジトリ(特に mentions_boundaries=1 かつ doc規模の異なる
   もの)から3〜5件を選び、実際のCLAUDE.md/AGENTS.mdおよびskills/hooks
   関連ファイルを読む
   - 候補: `apache/airflow`(大規模+boundaries言及)、`nuxt/nuxt`(極小)、
     `browser-use/browser-use`(最大規模)、`cline/cline`(boundaries言及なし)、
     `Aider-AI/aider`(instructions自体なし、対照群)
2. 各リポジトリについて指標3〜5を手でコーディングし、比較表を作る
3. metrics.jsonの定量指標(1・2)と手動指標(3〜5)を突き合わせ、
   「検証したい問い」に対する仮説の当否を確認する
4. 自分自身(このplaybookリポジトリ)のCLAUDE.mdも同じ指標で採点し、
   他プロジェクトとの相対位置を確認する(実践知として記事に活かす)

### 気づいた不整合(要データ要望化を検討)
- metrics.jsonの `Significant-Gravitas/AutoGPT` は `has_agent_instructions: 1`
  だが `agent_doc_char_count/heading_count` を見る限り本文が短い
  (3,822字/6見出し)。他の`has_agent_instructions: 0`勢(autogen等)との
  境界(何を「ある」と判定しているか)が曖昧に見えるため、検証時に実物を
  見て確認する。もし判定基準の説明が要りそうなら、振り返り(Issue #6)で
  data-requests に起票する

---

## 記事構成のたたき台(仮説ベース、検証前)

検証結果次第で大きく変わる前提の仮案。

**タイトル案**
- 「AIエージェントに何を"ルールで書く"か、何を"仕組みに落とす"か —
  15プロジェクトのCLAUDE.md/skillsから見る境界線」

**想定章立て**
1. 導入: なぜ境界線が問題になるか(指示書の肥大化、逸脱、メンテコスト)
2. 調査方法: metrics.jsonの定量指標 + 手動コーディングした制約分類の
   概要(上記「調査方針」を圧縮して記載)
3. 観察1: 指示文書の規模と役割分担明示(boundaries言及)の関係
   - 仮説: 規模が大きいプロジェクトほど人間/AIの境界を明示する傾向が
     ある
4. 観察2: 「AI開発ツールを作っている側」が自分の指示文書を持たない
   ケース(autogen/aider等)の考察
   - 仮説: ツール自体の複雑さと、開発者向け指示書整備の優先度は独立
5. 観察3: ルールとツール化の使い分けの実例(手動調査で見つかった具体例
   を2〜3個引用)
6. 実践知: 自分のリポジトリ(agent-trend-playbook)への適用
   - このCLAUDE.mdの「検証は人間主導」のような役割分担の明示は、調査
     対象と比べてどう位置づくか
7. まとめ: 「まず文書で書き、繰り返し使うものだけをskill/仕組みに落とす」
   といった実践的な指針(検証結果に応じて言い換え)

**次のアクション**: 上記「手順」1〜4を Issue #3(検証の実施)として
`verifications/` に軽量ログを残しながら進める。
