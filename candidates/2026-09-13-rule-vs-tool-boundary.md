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

---

## 調査方針

### 対象
ツール化指標が揃ったため、20リポジトリ全件を対象にできる。
`has_agent_instructions: 1` の16件(表1)と `has_agent_instructions: 0` の
4件(表2)に分けて見る。

**表1: 指示文書あり(16件、doc文字数の降順)**

| repo | segment | doc文字数 | mentions_boundaries | has_skills_dir | skills_count | custom_commands_count | has_hooks_config | mcp_servers_count |
|---|---|---|---|---|---|---|---|---|
| OpenHands/OpenHands | tool | 117,554 | 1 | 0 | 0 | 0 | 0 | 0 |
| browser-use/browser-use | tool | 49,587 | 1 | 0 | 0 | 0 | 0 | 0 |
| langchain-ai/langchain | tool | 38,490 | 1 | 0 | 0 | 0 | 0 | 2 |
| apache/airflow | adopter | 36,390 | 1 | 1 | 4 | 0 | 0 | 0 |
| vercel/next.js | adopter | 30,650 | 1 | 1 | 0 | 0 | 0 | 0 |
| colinhacks/zod | adopter | 21,939 | 1 | 1 | 2 | 0 | 0 | 1 |
| oven-sh/bun | adopter | 17,572 | 1 | 1 | 9 | 6 | 1 | 0 |
| astral-sh/ruff | adopter | 17,252 | 1 | 0 | 0 | 0 | 1 | 0 |
| remix-run/remix | adopter | 8,319 | 1 | 0 | 0 | 0 | 0 | 0 |
| getsentry/sentry | adopter | 7,491 | 1 | 1 | 0 | 3 | 0 | 2 |
| supabase/supabase | adopter | 6,554 | 1 | 1 | 0 | 0 | 1 | 1 |
| cline/cline | tool | 5,263 | 0 | 1 | 6 | 2 | 1 | 0 |
| Significant-Gravitas/AutoGPT | tool | 3,822 | 0 | 1 | 10 | 0 | 0 | 0 |
| crewAIInc/crewAI | tool | 1,867 | 1 | 0 | 0 | 0 | 0 | 0 |
| nuxt/nuxt | adopter | 960 | 1 | 0 | 0 | 0 | 0 | 0 |
| sveltejs/svelte | adopter | 590 | 0 | 0 | 0 | 0 | 0 | 0 |

**表2: 指示文書なし(4件)**

| repo | segment | has_skills_dir | skills_count | custom_commands_count | has_hooks_config | mcp_servers_count |
|---|---|---|---|---|---|---|
| continuedev/continue | tool | 1 | 1 | 0 | 0 | 0 |
| microsoft/autogen | tool | 0 | 0 | 0 | 0 | 0 |
| Aider-AI/aider | tool | 0 | 0 | 0 | 0 | 0 |
| yoheinakajima/babyagi | tool | 0 | 0 | 0 | 0 | 0 |

### 初期観察(定量データのみ、検証前の仮説)

- **OpenHands/OpenHands vs Significant-Gravitas/AutoGPT が好対照**: OpenHands
  は指示文書が117,554字と圧倒的に大きいのにツール化指標は全て0。逆に
  AutoGPTは指示文書が3,822字と薄いのにskills_countが10と全リポジトリ中
  最多。「文書に全部書く」と「薄い文書+ツールで肩代わり」という両極端の
  実例になりそうで、深掘り優先度を上げる
- **`continuedev/continue`だけが例外**: `has_agent_instructions: 0` の4件の
  うち3件(autogen/aider/babyagi)はツール化指標も全て0(文書もツールも
  何もない)。continuedev/continueだけが唯一 skills_count=1 を持つ。
  「指示文書がない=ツールで代替している」という仮説は今のところこの1件
  でしか支持されない
- **文書量とツール化の程度に単純な相関は見えない**: bun(doc17,572字/
  skills9/commands6)のように両方厚いケースもあれば、browser-use
  (doc49,587字/ツール全0)のように文書だけのケースもある。手動読解で
  「何が両立し、何が片方に寄るのか」の理由を見る必要がある

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

### 気づいた不整合(radar側にissue案として提示済み、2026-09-14)
- `Significant-Gravitas/AutoGPT` は `has_agent_instructions: 1` だが本文は
  短い(3,822字/6見出し)。他の`has_agent_instructions: 0`勢(autogen等)
  との判定境界が曖昧に見えるため、検証時に実物を見て確認する
- `has_skills_dir: 1` かつ `skills_count: 0` の食い違いが3件
  (`getsentry/sentry`, `supabase/supabase`, `vercel/next.js`)。検証時に
  手動で実態を確認する
- `schema/SCHEMA.md` がツール化6フィールド追加後も未更新、および
  `snapshots/2026-09-13/metrics.json` が同日中に上書きされ既存の
  `mentions_*` 値も一部変化している点は、radar側にissue案として提示済み
  (このリポジトリからは未起票、対応状況は追って確認する)

---

## 記事構成のたたき台(仮説ベース、検証前)

検証結果次第で大きく変わる前提の仮案。

**タイトル案**
- 「全部書くか、仕組みに任せるか — 20プロジェクトのCLAUDE.md/skillsから
  見るルール化とツール化の境界線」

**想定章立て**
1. 導入: なぜ境界線が問題になるか(指示書の肥大化、逸脱、メンテコスト)
2. 調査方法: metrics.jsonの定量指標(文書量・ツール化指標) + 手動
   コーディングした制約分類の概要(上記「調査方針」を圧縮して記載)
3. 観察1: OpenHands(文書117,554字・ツール化0)とAutoGPT(文書3,822字・
   skills10)の対比 — 同じ「AI開発ツール」でも両極端な戦略が併存する
4. 観察2: 「指示文書がない」ことと「ツール化されている」ことは別問題
   — continuedev/continueを除き、instructionsなしの3件はツール化指標も
   全て0だった(仮説「文書がない分ツールで肩代わり」は支持されなかった)
5. 観察3: ルールとツール化の使い分けの実例(手動調査で見つかった具体例
   を2〜3個引用、cline/clineのバランス型を中心に)
6. 実践知: 自分のリポジトリ(agent-trend-playbook)への適用
   - このCLAUDE.mdの「検証は人間主導」のような役割分担の明示は、調査
     対象と比べてどう位置づくか
7. まとめ: 「まず文書で書き、繰り返し使うものだけをskill/仕組みに落とす」
   といった実践的な指針(検証結果に応じて言い換え)

**次のアクション**: 上記「手順」1〜4を Issue #3(検証の実施)として
`verifications/` に軽量ログを残しながら進める。
