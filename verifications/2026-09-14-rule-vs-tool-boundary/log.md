# ルール化とツール化の境界線 検証ログ

- 日付: 2026-09-14
- 根拠スナップショット: agent-trend-data snapshots/2026-09-13
  (metrics.json, generated_at 2026-09-13T09:32:39Z。ツール化6指標追加後の版)
- 対象: `candidates/2026-09-13-rule-vs-tool-boundary.md` で選定した5件
  (OpenHands/OpenHands, Significant-Gravitas/AutoGPT, continuedev/continue,
  cline/cline, nuxt/nuxt)

## 何を試したか

各リポジトリの実際のAGENTS.md/CLAUDE.mdと、skills/commands/hooks関連
ファイル(`.claude/`, `.agents/`, `.openhands/`, `.clinerules/` 等、
リポジトリごとに置き場所が異なる)をGitHub上で直接読み、指標④(制約の
型分類: 手順型/禁止型/委譲型)と⑤(役割分担の明示度)を手動でコーディング
した。

## 結果

**OpenHands/OpenHands**: AGENTS.md(32見出し)はリポジトリ構成・テスト
フレームワーク・CI障害対応の手順など手順型中心だが、「PR説明文の
`HUMAN:` セクションはAIエージェントが編集禁止、人間に確認を求めよ」と
いう明確な委譲型の一文がある。**データ不整合を確認**: `.agents/skills/`
に実際は3件のskills(`custom-codereview-guide.md`, `pr-design-doc/`,
`release.md`)が存在するが、metrics.jsonは `has_skills_dir=0,
skills_count=0`。当初「OpenHandsは文書のみでツール化ゼロ」という前提は
誤りだった。

**Significant-Gravitas/AutoGPT**: AGENTS.mdは`autogpt_platform/`配下限定の
コードスタイル規約(手順型/禁止型)のみで、委譲型の言及はなし。CLAUDE.md
は`@AGENTS.md`と書かれた1行の転送ファイルで、nuxt/nuxtのCLAUDE.mdと
バイト単位で同一(SHA一致)——プロジェクト固有ではなく汎用テンプレート。
10件のskillsはほぼ全てPR/リリースのライフサイクル
(open-pr/pr-review/pr-address/pr-polish/pr-test/orchestrate等)。

**continuedev/continue**: AGENTS.md/CLAUDE.mdは実際に存在しない。唯一の
skill(`cn-check`)は自社CLIでdiffレビューを行わせる、リポジトリ運用
ルールというより製品機能のドッグフーディングに近い内容。「指示文書が
ない分をツールで肩代わりしている」という当初仮説は、この1件を見ても
支持されない。

**cline/cline**: AGENTS.mdはコードスタイルではなく、クラウドサンドボックス
特有の環境知識(既知のテスト環境アーティファクト、ポート番号、ビルド順序
の落とし穴など)がほとんど。release/hotfixの手順書(`.claude/commands/`、
実体は`.clinerules/workflows/`へのシンボリックリンク)は手順型だが、
その中に「どのコミットを含めるか人間に確認する」「バージョン番号を人間に
確認する」という委譲型のチェックポイントが埋め込まれている——役割分担が
文書ではなくツール側に実装されている例。

**nuxt/nuxt**: 「何もしていない」という当初の想定は誤り。948字の
AGENTS.mdは「このプロジェクトはAIによる公開文章の作成、自律的な
コントリビューションを禁止する。人間が完全に理解した内容のみ提出可能。
PRタイトル/説明文/Issue/ディスカッションコメントを起草・投稿するな」
という、調査対象中もっとも強い禁止型+役割分担の明示。ツール化は
確認できた範囲でゼロ。最短の文書が最強の統制を実現していた。

## 所感

- 期待と違った点: 「文書が薄い=統制がゆるい」「ツール化ゼロ=文書に
  全部書いている」という定量データだけからの仮説は、5件中3件
  (OpenHands, continuedev/continue, nuxt/nuxt)で実際に読むと修正が
  必要だった。特にnuxt/nuxtは最短文書が最強の統制という逆の結果。
  データだけでは分からず、実際に読まないと分からないことがあると
  実感した(この検証プロセス自体の存在意義を裏付ける結果になった)
- 新たに見えた軸: 「何を制約するか」は同じでも「どこに実装するか
  (プローズ/ツール)」が違う、という観点の方が「量の多寡」より本質的
  だった(cline/clineの委譲型チェックポイントがツール側に埋め込まれて
  いる例が象徴的)
- skillsが複数ツール向けにシンボリックリンクで共有されているケースが
  複数見つかった(OpenHands, AutoGPT, cline)。skills_countが同一実体の
  重複カウントになっている可能性があり、指標としての解釈に注意が必要
- 実際に使えそうか: 記事の切り口としては「定量データだけで見た仮説が、
  実際に読むと3/5件で覆った」という展開自体が強いフックになりそう。
  当初のOpenHands/AutoGPT対比(文書のみ vs ツールのみ)は測定ミスだった
  ため使えないが、「文書量とガバナンスの強さは無関係」「役割分担は文書
  ではなくツールに実装されうる」という2つの発見の方が主張として筋が良い

## 検出したデータ不整合(追加分)

- `OpenHands/OpenHands`: `has_skills_dir=1/skills_count=3` が正しいはずが
  `0/0` になっている(既報告のsentry/supabase/next.jsの逆パターン:
  ディレクトリありなのに未検出)
- skillsディレクトリがシンボリックリンクで複数箇所に存在するケースが
  あり、収集ロジックがシンボリックリンクをたどるかどうかでカウントが
  変わりうる(一般的な注意点として)

## 追記(2026-09-14、radar対応後の再確認)

上記の指摘はradar側で全て対応され、2026-09-14スナップショット
(agent-trend-data snapshots/2026-09-14, generated_at
2026-09-14T14:40:23Z)に反映された。

- **OpenHandsのskills検出漏れ**: 修正された。原因はOpenHands固有では
  なく、`.claude/skills/`規約のみを見ていて`.agents/skills/`規約を
  見落としていたという汎用的なバグだった。副次効果として多数の
  リポジトリでskills_countが増加(remix 0→19等)。remix-run/remixの
  `.agents/skills/`を実際に確認し、19件とも別々の実在するskillsだと
  確認した(過大カウントではない)
- **`continuedev/continue`の記述を訂正**: `has_agent_instructions`が
  `0`→`1`に変わった。実体は `extensions/cli/AGENTS.md`
  (非ルート階層、CLIサブパッケージ限定)で、ビルド/テスト/lintコマンドや
  アーキテクチャ概要を説明する完全に手順型の文書。委譲型・禁止型の記述
  はなし。**この検証ログの当初の記述(「AGENTS.md/CLAUDE.mdは実際に
  存在しない」)は誤りだった**——旧ロジックがルート直下しか検索して
  いなかったための見落とし。「指示文書なし」グループは
  microsoft/autogen, Aider-AI/aider, yoheinakajima/babyagiの3件のみに
  なり、この3件は文書・ツール化とも全指標が0で例外なし
- **`agent_doc_count`フィールドが新設**(見つかった指示文書のディレクトリ数)。
  apache/airflow=14, Significant-Gravitas/AutoGPT=13
  など、指示文書が複数箇所に分散しているリポジトリを検出できるように
  なった。AutoGPTの`autogpt_platform/`限定という当検証の発見(手動で
  確認済み)と整合する
- **統制の強さの指標化**: 新指標は追加されず、SCHEMA.mdに「文書量を
  統制の強さの代理指標として使わないこと」という注意書きが、この検証
  (nuxt/nuxt vs AutoGPT)を根拠として追記された
