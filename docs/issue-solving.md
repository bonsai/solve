# Issueを解くとは何か

「このIssueを解いて」と言われたとき、Agent Workflow（AW）は、いきなり実装を始めるべきではない。

Issueは、単なる作業指示ではない。そこには「何が問題なのか」「なぜそれをするのか」「どこまでできれば終わりなのか」が、完全には整理されていない場合がある。したがってAWにとって「解く」とは、Issueに書かれた文章を処理することではなく、**曖昧な課題を、検証可能な解決状態へ変換すること**である。

そのために、AWはIssueのStatusを観測する。そして、そのStatusに応じたActionを選択する。

最初のIssueは `open` である。

AWが最初に行うActionは `understand` になる。Issue本文、関連するファイル、既存の実装、過去のEventなどを読み、Issueが何を求めているのかを理解する。このとき発生するEventは、たとえば「Issueを取得した」「関連ファイルを発見した」「要求条件を抽出した」「不明点を発見した」といったものである。

重要なのは、ここでAWが「理解したつもり」にならないことである。理解とは、次のActionを実行できるだけの情報が揃った状態である。必要ならば追加調査を行い、Issueの目的やAcceptance Criteriaを具体化する。

十分に理解できれば、IssueのStatusは `understood` になる。

次にAWは `plan` というActionを取る。

ここでは、どうやってIssueを解くかを決める。変更するファイル、利用するSkill、必要なデータ、テスト方法、依存関係などを整理する。たとえば「GoでAPIを変更し、JSONLを生成し、p5.js側で読み込んで表示し、サンプルデータで確認する」というように、抽象的なIssueを具体的な実行計画へ変換する。

この過程では「plan created」「implementation target identified」「dependency discovered」といったEventが発生する。計画が成立するとStatusは `planned` になる。

ここからAWは実際の作業に入る。

`execute` というActionによって、計画された作業を開始する。ここでStatusは `executing` になる。

ただし、`executing` は「コードを書いている」という狭い意味ではない。調査、ファイル作成、コード変更、データ生成、コマンド実行、外部APIの利用など、Issueを解決するための実行フェーズ全体を意味する。

このフェーズでは大量のEventが発生する。「ファイルを作成した」「コードを変更した」「コマンドを実行した」「依存パッケージを追加した」「テストを実行した」「エラーが発生した」「APIからデータを取得した」といったEventの履歴が、AWの仕事そのものになる。

ここで面白いのは、**ActionとEventは同じものではない**ということである。

AWが「テストを実行する」のはActionであり、「テストが失敗した」のはEventである。

AWが「ファイルを編集する」のはActionであり、「ファイルが変更された」のはEventである。

AWが「Issueを検証する」のはActionであり、「Acceptance Criteriaを満たした」のはEventである。

つまりAWは、Actionによって世界に変化を起こし、その変化をEventとして観測する。

そしてEventによって、次に取るべきActionを決める。

これは単純な一本道ではない。

たとえば実装後のテストが失敗したとする。AWは `verify` を実行した結果、`verification_failed` というEventを受け取る。するとIssueをSolvedにすることはできない。AWは失敗原因を理解し、再び実装Actionを取る必要がある。

```text
implemented
    ↓
verify
    ↓
verification_failed
    ↓
investigate
    ↓
fix
    ↓
implemented
    ↓
verify
```

したがって、Solveは単純な一直線のPipelineではなく、**Eventによって分岐するLoop**である。

検証が成功すると、`verified` というStatusに到達する。

ここでAWが行う `verify` は、単に「エラーが出なかった」ことを確認するだけではない。Issueに設定された条件を満たしているかを確認する。実装が存在するか、テストが通るか、期待するデータが生成されるか、画面に正しく表示されるか、既存機能を壊していないか、Issueに書かれた目的を達成しているか。こうした検証Eventが蓄積されることで、「解いた」という主張に根拠が生まれる。

そして最後にAWは `solve` というActionを実行する。

このActionは、新しいコードを書くことではない。**検証された結果をIssueの完了状態として確定するAction**である。

Issue、実装、テスト結果、関連Eventを整理し、必要ならPRやコミットを紐付け、解決内容を記録する。その結果、Statusは `solved` になる。

```text
OPEN
  │
  │ understand
  ▼
UNDERSTOOD
  │
  │ plan
  ▼
PLANNED
  │
  │ execute
  ▼
EXECUTING
  │
  │ implement
  ▼
IMPLEMENTED
  │
  │ verify
  ▼
VERIFIED
  │
  │ solve
  ▼
SOLVED
```

ここでStatusは「現在地」であり、Actionは「AWがすること」、Eventは「その結果として起きたこと」、Skillは「AWがそのActionを実行するために使える能力」である。

したがって、

**Skill → Action → Event → Status**

という関係が、AWの基本的な運動になる。

たとえばGitHubを操作するSkillがあれば、AWはIssueを取得するActionを実行できる。その結果「Issue取得完了」というEventが発生する。コード編集Skillがあれば、ファイル変更Actionを実行できる。その結果「ファイル変更」というEventが発生する。テストSkillがあれば、検証Actionを実行できる。その結果「テスト成功」または「テスト失敗」というEventが発生する。

この構造にすると、AWは「何でもできるAI」ではなくなる。

AWは、現在のIssue Statusを見て、利用可能なSkillから適切なActionを選び、そのActionを実行し、発生したEventを観測し、次のStatusとActionを決める。

つまりAWとは、**Issueを直接解決する魔法の主体ではなく、ActionとEventのLoopによってIssueをSolvedへ運ぶ状態遷移システム**なのである。

そして、この考え方の最大の利点は、「解けなかった場合」も同じモデルで扱えることにある。

調査しても情報が足りない。必要なSkillが存在しない。外部サービスが停止している。テストが失敗し続ける。Issueそのものに矛盾がある。こうした場合、AWは無理に `solved` にする必要がない。

発生したEventを記録し、現在のStatusを維持し、必要なら人間に判断を返す。

「解く」とは、必ずSolvedにすることではない。

**解決可能性を探索し、実行し、検証し、その結果を正確なStatusとして残すこと**までを含めて「Issueを解く」のである。

この意味で `bonsai/solve` は、単なるIssue管理ライブラリではない。

それは、

> **Issueという課題を、ActionとEventを通じて観測可能・実行可能・検証可能な状態遷移へ変換するためのOntologyである。**

AWが「Issueを解いて」と言われた瞬間から始まるのは、コード生成ではない。

最初にあるのは `open` という状態である。そこからAWは観測し、理解し、計画し、行動し、結果を受け取り、失敗すれば戻り、成功すれば検証し、最後にSolvedを確定する。

**Solveとは、答えを一度で出すことではない。Eventを読みながら、Issueを一つずつSolvedへ近づけていくLoopなのである。**
