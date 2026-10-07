---
name: study-material-review
description: >
  技術系の社内勉強会・研修・ハンズオン資料をレビューするためのスキル。
  Rspress、VitePress、Docusaurus 等の Markdown/MDX ベースのドキュメントサイトを主対象とし、
  全ページを横断して技術的正確性、教育設計、説明品質、一貫性、実務適合性を評価する。
  単なる誤字脱字チェックではなく、受講者が順番に理解できる教材になっているかを重視する。
---

# Study Material Review Skill

## Purpose

技術系の社内勉強会、研修、ハンズオン、技術解説サイトを
**教材としてレビューする**。

主な対象:

- Rspress
- VitePress
- Docusaurus
- MkDocs
- その他 Markdown / MDX ベースのドキュメントサイト
- 単一または複数の Markdown 技術資料

このスキルでは、個々の Markdown を独立した文書として見るだけでなく、
**資料全体を一つの学習コンテンツとして横断レビューすること**を重視する。

レビューの優先順位は次の通り。

1. 技術的に正しいか
2. 受講者が順番に理解できるか
3. 勉強会・研修で説明しやすいか
4. 実務につながるか
5. 文章・Markdown として整っているか

細かな誤字脱字を大量に列挙することより、
教材として重要な問題を見つけることを優先する。

---

# When to Use

以下のような依頼で使用する。

- 「社内勉強会資料をレビューして」
- 「Rspress の全 Markdown をレビューして」
- 「研修資料として分かりやすいか確認して」
- 「技術資料の構成をレビューして」
- 「教材として改善点を出して」
- 「Markdown サイト全体の品質を確認して」
- 「ハンズオン資料の技術的誤りを探して」

単なる文章校正のみが目的の場合は、このスキルを全面適用する必要はない。

---

# Review Policy

## 原則

レビューが目的である場合、**勝手に本文を全面修正しない**。

原則として以下を行う。

- 問題点を特定する
- 問題の理由を説明する
- 改善方向を提案する
- 必要に応じて局所的な修正例を示す
- 全体の修正優先順位を提案する

明示的に修正まで依頼されていない限り、以下は行わない。

- Markdown 本文の全面書き換え
- 大規模なファイル移動
- ナビゲーション構成の変更
- dependency update
- library / framework version update
- コードリファクタリング
- design の全面変更
- 技術選定の変更

build / test / lint / link check など、
レビューの検証に必要な操作は実施してよい。

---

# Review Workflow

## Step 1: Repository / Site Structure を把握する

Markdown を個別にレビューする前に、資料全体の構造を確認する。

確認対象の例:

- README
- package.json
- lock file
- Rspress / VitePress / Docusaurus 等の設定
- sidebar
- navigation
- docs directory
- src directory
- assets
- images
- diagrams
- sample code
- plugins
- custom components
- Markdown extensions
- build / lint / test scripts

そのうえで以下を把握または推定する。

- 想定読者
- 前提知識
- 想定される受講順序
- 章構成
- 教材の目的
- 最終的に受講者に理解してほしいこと

推定した場合は、事実と推定を区別する。

---

## Step 2: Review Scope を確定する

原則として、サイトで公開されるすべての Markdown / MDX を対象にする。

通常除外するもの:

- node_modules
- dist
- build
- target
- .git
- generated files
- vendor files
- third-party documentation
- 自動生成 API ドキュメント

除外対象がある場合は、レビュー結果に明記する。

---

## Step 3: 各ページをレビューする

各 Markdown / MDX を以下の観点で確認する。

ただしファイル単位のレビューだけで完了してはいけない。

---

## Step 4: 全ページ読了後に横断レビューする

すべての対象ファイルを読んだ後、
**必ずサイト全体を再評価する**。

ここでは特に以下を確認する。

- 用語の一貫性
- ページ間の矛盾
- 説明の重複
- 前提知識の依存関係
- 学習順序
- サンプル設定値の整合性
- version の整合性
- 実務上のメッセージの一貫性

---

## Step 5: Build / Link / Markdown Validation

可能なら以下を実行する。

- build
- lint
- Markdown lint
- link check
- example code の test
- repository が提供する validation command

問題を発見しても、レビュー依頼の場合は原則としてその場で直さず、
レビュー結果へ記録する。

---

# Review Criteria

## A. Technical Accuracy

最重要項目。

以下を確認する。

- 技術的な誤りがないか
- 用語の定義が正しいか
- API / CLI / config の説明が正しいか
- コマンド例が成立するか
- コード例が成立するか
- dependency が不足していないか
- version 依存の記述が適切か
- deprecated な方法を推奨していないか
- 前提条件が抜けていないか
- 例外条件が抜けていないか
- 「必ず」「常に」「絶対」などの断定が妥当か
- 過度な単純化によって誤解を生まないか
- tutorial 用の方法を production 推奨として説明していないか
- セキュリティ上危険な例を無説明で掲載していないか

明確な技術的誤りは最優先で指摘する。

---

## B. Instructional Design

資料を技術文書ではなく**教材**として評価する。

確認項目:

- 初学者が順番に読んで理解できるか
- 未説明の概念を突然使用していないか
- 後で説明する内容を前提にしていないか
- 前提知識が暗黙になっていないか
- ページ / 章の順序が自然か
- 学習ステップの粒度が適切か
- 説明に飛躍がないか
- 一ページに内容を詰め込みすぎていないか
- 同じ内容を何度も説明していないか
- 本筋と関係の薄い内容が入り込んでいないか
- 章ごとの学習目標が明確か

教材全体として、概ね次の流れが成立しているか確認する。

```text
なぜ必要なのか
↓
それは何なのか
↓
どういう仕組みなのか
↓
どう使うのか
↓
具体例
↓
実務ではどう使うのか
```

すべてのページをこの形式にする必要はない。
重要なのは、受講者が置いていかれないことである。

---

## C. Presentation / Teaching Usability

講師がブラウザや資料を画面共有しながら説明する状況を想定する。

確認項目:

- 画面共有しながら説明しやすいか
- 一画面に情報を詰め込みすぎていないか
- 長大な文章が続いていないか
- コード例が大きすぎないか
- 重要ポイントが視覚的に把握できるか
- 箇条書きにした方が良い箇所がないか
- diagram が必要な箇所がないか
- コード例が必要な箇所がないか
- demo が有効な箇所がないか
- 詳細すぎて Appendix に回すべき内容がないか
- 口頭説明がなければ成立しない記述が多すぎないか

各章について、

> この章で受講者に何を理解してほしいのか

が明確であることが望ましい。

---

## D. Reader Comprehension

読者視点で確認する。

- 専門用語が初出時に説明されているか
- 略語が突然登場していないか
- 主語が曖昧でないか
- 「これ」「それ」「上記」等の指示対象が明確か
- 一文が長すぎないか
- 一段落が長すぎないか
- 抽象説明だけで終わっていないか
- 具体例が不足していないか
- 読者が次に何をすればよいか分かるか
- 説明の粒度が急に変わっていないか
- 必要な背景知識が補われているか

---

## E. Code Samples

コードブロックを以下の観点で確認する。

- language 指定
- syntax
- import
- dependency
- API 名
- function / class / variable 名
- 実行可能性
- 前後のサンプルとの整合性
- サンプルとしての簡潔さ
- 本題と無関係な処理の混入
- copy & paste で試せるか
- 省略箇所が明示されているか
- 擬似コードか実行可能コードかが明確か
- security / credential の扱いが適切か
- production で避けるべき記述が tutorial として明示されているか

サンプルが完全なコードでない場合は、少なくとも以下のどれかが分かるようにする。

- 実行可能コード
- 抜粋
- 擬似コード
- 説明用 simplified example

---

## F. Markdown / Documentation Site Quality

Markdown および利用している documentation framework の品質を確認する。

例:

- 見出し構造
- H1 / H2 / H3 階層
- frontmatter
- internal link
- external link
- relative path
- image link
- anchor
- code fence
- table
- admonition / container
- framework 固有記法
- MDX 構文
- broken link
- missing image
- duplicate anchor

Rspress の場合は Rspress 固有構文・設定も確認する。

---

## G. Cross-page Consistency

全ページ読了後に必ず確認する。

- 用語の表記揺れ
- 同じ概念に複数の名称を使用していないか
- 同じ説明が複数ページに重複していないか
- ページ間で説明が矛盾していないか
- コマンドがページごとに異なっていないか
- directory / file path が一致しているか
- port 番号が一致しているか
- project 名が一致しているか
- environment variable 名が一致しているか
- version が一致しているか
- 古い手順と新しい手順が混在していないか
- 前ページと次ページで前提条件が変化していないか

---

## H. Practical Applicability

社内勉強会の場合、
単なる一般技術解説ではなく実務への接続を確認する。

- なぜこの技術を学ぶのか分かるか
- 実務でどこに使えるか分かるか
- 実務で避けるべき使い方が分かるか
- recommendation と example が区別されているか
- tutorial と production の違いが分かるか
- security 上の注意点があるか
- operation 上の注意点があるか
- troubleshooting の入口があるか
- 社内ルールとの接点が必要なら説明されているか

社内事情が不明な場合は勝手に補完せず、
「実務への接続が不足している可能性」として扱う。

---

## I. Content Volume and Time Allocation

資料全体の情報量を評価する。

- 内容が多すぎないか
- 内容が少なすぎないか
- 各章の分量バランス
- 本筋から外れた深掘り
- Appendix に移すべき内容
- 削除しても学習効果を損なわない内容
- 逆に説明不足の内容

勉強会時間が分からない場合、時間を勝手に仮定しない。

必要に応じて、

- 60 分の場合
- 90 分の場合
- 120 分の場合

など複数ケースでコメントする。

---

# Severity

各指摘には Severity を付ける。

## Critical

以下のような問題。

- 明確な技術的誤り
- 実行すると危険
- 重大な security issue
- 教える内容そのものが間違っている
- destructive operation を安全策なしで推奨している

---

## High

以下のような問題。

- 読者が重大な誤解をする
- 重要な前提条件が不足
- 学習順序に重大な問題がある
- 主要サンプルが動かない
- ページ間で重大な矛盾がある
- 実務で誤用する可能性が高い

---

## Medium

以下のような問題。

- 分かりにくい
- 説明不足
- 構成改善
- 具体例不足
- 重複
- 用語説明不足
- diagram / example があると大幅に理解しやすくなる

---

## Low

以下のような問題。

- 表記揺れ
- 誤字脱字
- 文体
- Markdown style
- 軽微な改善
- formatting

---

# Finding Format

各指摘は可能な限り以下の形式にする。

```text
Severity:
File:
Section / Heading:
Line:
Category:

Issue:
Why it matters:
Suggested improvement:
```

単に、

> 分かりにくい

では不十分。

以下まで説明する。

- 何が問題なのか
- なぜ問題なのか
- 誰が誤解しそうか
- どう改善するとよいか

全文書き換えは不要。

---

# Overall Evaluation

レビュー終了後、以下を 5 点満点で評価する。

| 評価軸 | Score |
|---|---:|
| 技術的正確性 | /5 |
| 教育設計 | /5 |
| 文章・説明品質 | /5 |
| サイト全体の一貫性 | /5 |
| 実務への適合性 | /5 |

点数だけでなく評価理由を書く。

必要に応じて総合コメントも付ける。

---

# Recommended Review Report Structure

レビュー報告書を作成する場合は、原則として以下の構成を使用する。

```markdown
# 勉強会資料レビュー

## 1. Executive Summary

## 2. 対象範囲

## 3. 総合評価

### 技術的正確性
### 教育設計
### 文章・説明品質
### サイト全体の一貫性
### 実務への適合性

## 4. Critical Issues

## 5. High Priority Issues

## 6. Medium Priority Issues

## 7. Low Priority Issues

## 8. ファイル別レビュー

## 9. ページ横断の問題

### 用語・表記
### 重複
### 矛盾
### 前提知識
### ページ順序

## 10. 不足している説明

## 11. 図・コード例・デモを追加するとよい箇所

## 12. 削減または Appendix 化を推奨する内容

## 13. 推奨する修正順序

## 14. Build / Link / Markdown 検証結果

## 15. 良い点
```

---

# Positive Findings

問題だけを列挙しない。

以下も明示する。

- 分かりやすい説明
- 維持すべき構成
- 良いコードサンプル
- 良い diagram
- 適切な実務例
- ページ間の良い導線
- 初学者に配慮された説明

改善時に壊してはいけない部分を明確にする。

---

# Recommended Fix Order

修正作業へ進む場合は、原則として以下の順番を推奨する。

## Phase 1: Critical / High Technical Issues

- 技術的誤り
- 危険な手順
- 動かないサンプル
- 重大な矛盾

## Phase 2: Learning Flow

- ページ順序
- 前提知識
- 説明の飛躍
- 章構成

## Phase 3: Explanatory Quality

- 説明不足
- 具体例
- diagram
- demo
- 実務との接続

## Phase 4: Editorial Quality

- 表記揺れ
- 文体
- 誤字
- Markdown style

---

# Final Checklist

レビュー完了前に以下を確認する。

- [ ] 対象 Markdown / MDX を網羅した
- [ ] 除外したファイルを把握した
- [ ] 技術的正確性を確認した
- [ ] 教育設計を確認した
- [ ] コードサンプルを確認した
- [ ] Markdown / site framework を確認した
- [ ] 全ページ読了後に横断レビューした
- [ ] 用語・version・設定値の整合性を確認した
- [ ] 実務適合性を確認した
- [ ] Critical / High を明確にした
- [ ] Positive Findings を記載した
- [ ] 修正優先順位を示した
- [ ] build / lint / link check の結果を記録した
- [ ] レビュー依頼なのに勝手な全面修正をしていない

---

# Core Principle

このスキルで最も重要なのは、

> 「文章として正しいか」ではなく、
> 「受講者がこの順序で学んだとき、正しく理解して実務へ持ち帰れるか」

という観点で資料全体を見ることである。

個別ページの完成度が高くても、
教材全体として学習順序・前提知識・用語・実務への接続が壊れていれば、
高品質な勉強会資料とは評価しない。

