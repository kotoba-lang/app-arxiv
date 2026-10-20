# app-arxiv（投稿検証）— itonami（dapp）上の分散型設計

状態: 第 1 スライス実装（2026-10-20）。正本はこの文書。
追跡: Linear JUN-128（親 JUN-116）。

## 位置づけ

app-arxiv は arXiv 投稿の作業場 —— 論文（papers/*/main.tex）と投稿スクリプト
（validate-tex.sh / submit-arxiv.sh）。本スライスは **LaTeX 投稿の検証ゲート
の判定**（validate-tex.sh）を判定コア化。

## このスライス: 投稿検証判定コア

| 層 | 実体 | 状態 |
|---|---|---|
| 判定コア guest | `aiueos/arxiv_validate.cljk` — main-tex-judgment（main.tex 優先 / 単一 .tex は採用 / 0 個はエラー / **複数はエラー「main.tex に改名せよ」**）/ validation-ok（**pass 1 失敗は WARN のみ、判定は pass 2 と PDF 生成**）/ bibtex-needed（citation と .bib の両方のときだけ） | **実装済み・実測** |
| 記述子 | `aiueos/arxiv-validate.edn` | **実装済み** |
| parity | `scripts/arxiv/validate_parity_test.cljk` — 正本 `validate-tex.sh` の分岐一致検査（main 解決 / WARN-ERROR の非対称 / PDF 判定 / bibtex 条件） | **実装済み・実測** |

実測: `amu test aiueos/arxiv_validate.cljk` → **24/24**（jvm-kir / js / wasm）。
parity → 8 assertions / 0 failures。

## 正本の正確な規則（guest はこれに追従）

- **main ファイルの解決**: main.tex が無ければ直下の .tex ちょうど 1 つ。
  0 個はエラー、**複数はエラー**（改名を要求 — どちらが本体か推測しない）。
- **2-pass の非対称ゲート**: pass 1 の失敗は WARN（「may be resolved in
  pass 2」）。**不合格判定は pass 2 の失敗と PDF 未生成**で行う。
- bibtex は aux に citation の記録があり .bib が存在するときだけ。

## 未実装境界

- submit-arxiv.sh（実際の提出フロー）の判定。
- TeX の中身の検証（サイズ・図版・ライセンス等の arXiv 規約）。
- pdflatex 実行自体は host（シェル）仕事。
- mon（DNA）化、aiueos session 載せ（fleet 共通）。
