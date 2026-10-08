# CLAUDE.md

このファイルはClaude Code（claude.ai/code）がこのリポジトリで作業する際のガイド。

## このリポジトリは何か

`convis` / VirtualRetina の網膜モデルが何をどう計算しているかを、大学数学や微積分が
怪しい人向けにゼロから説明する教科書。単一HTMLファイル（`retina_textbook.html`）。

元は [convis-legacy](https://github.com/tairar/convis-legacy)（網膜シミュレーション→
刺激パターン→外部エンコード/デコード→再構成→SSIM/PSNR評価、という実験ワークスペース）
を解説していた過程でClaude Artifactとして作成し、読者層が全く違う（研究者向けの実験
ログ vs 数学が苦手な人向けの教材）ため別リポジトリに分離した。

## Claude Artifactとの関係

このHTMLは **Claude Artifactとしても公開されている**（非公開設定）:

```
https://claude.ai/artifact/HDHdwfHV8iYwbkHs7piyvx
```

`retina_textbook.html`を編集したら、Artifactツールで同じURLに向けて republish
（`action: "publish"`、`url`に上記を指定）して、公開版も同期すること。
**編集前に一度`action: "read"`でArtifact側の最新版を取得し、ローカルのファイルと
食い違っていないか確認してから作業する**（Artifactのページ自体が別経路で更新されている
可能性があるため）。

## ページの技術的な制約（artifact-design skillの契約）

このページはClaude ArtifactのCSP制約下で作られているので、編集時は以下を守ること:

- 外部スクリプトは `cdnjs.cloudflare.com` 等の許可リストのみ。外部スタイルシートは
  `fonts.googleapis.com` のみ（他は読み込まれない）
- **数式はKaTeX**で組版している。KaTeXのCSS本体は`fonts.googleapis.com`以外からの
  スタイルシート読み込みが禁止されているため、`cdnjs`から取得した`katex.min.css`を
  素のCSS文字列としてページに**インライン埋め込み**し、フォントファイルは
  `data:font/woff2;base64,...`のdata URIとして`@font-face`に直接埋め込んでいる
  （`<style id="katex-embedded">`ブロック）。KaTeX本体の`katex.min.js`は
  `<script src="https://cdnjs.cloudflare.com/ajax/libs/KaTeX/...">`で読み込み可
- 数式を追加するときは、本文に`<span data-tex="...">`（displayModeなら
  `<div data-tex="...">`）でLaTeX文字列を書き、ページ下部のレンダリングスクリプトが
  `[data-tex]`要素を全部`katex.render()`する仕組み。手書きCSS組版（`.frac`/`.v`/`.gk`
  等のクラス）は旧方式の名残で、新しく式を追加するときはdata-tex方式を使う
- ライト/ダークテーマ両対応（`:root`でライト値、`@media (prefers-color-scheme: dark)`
  + `[data-theme]`でダーク値）。新しい色を追加するときはこのトークン方式を踏襲する
- タイトルは`<title>網膜モデル教科書</title>`で固定（republish時に変えない）

## 構成（全9章、0始まり）

0. はじめに — この本の読み方
1. 準備運動：出てくる数学の道具（畳み込み・微分・積分・簡単な微分方程式）
2. 網膜モデル全体の地図
3. 畳み込みとガウシアンフィルタ
4. 時間のフィルタ：指数移動平均（center_E/surround_Eに対応、漸化式の導出を含む）
5. OPL — 中心と周辺の差分
6. 双極細胞とコントラストゲイン制御
7. 神経節細胞とスパイク生成
8. まとめ：全体の数式地図

現在、数式（`data-tex`）は26個（4章・5章・6章・7章に実測値つきのSVG図を追加済み。6章は6.5節を新設し、GanglionInputの実コード上の静的非線形性を記載）。

## convis-legacyとの関係・参照資料

`docs/references/`に`convis_2018.pdf`・`virtual_retina_2009.pdf`を置いている
（`convis-legacy/docs/references/`からのコピー）。**`.gitignore`で除外しており、
このリポジトリのgit履歴には含めていない**（ローカル参照用のみ）。
もし無い場合は`convis-legacy`リポジトリ（同じ論文が`docs/references/`にある）から
コピーする。内容の正確性を確認する際はこの2本の論文、または`convis-legacy`の
`convis/convis/filters/retina.py`等の実際のソースコードを当たること
（コード片を引用するときは省略・言い換えせず、実際の行をそのまま引用する）。

## 更新の進め方

内容を大きく追加・変更する場合は、`artifact-design` skillを読み込んでページ契約
（上記の制約）を再確認してから編集する。コードや数式の説明を書く際は、実際に
手計算・数値検証（pythonでの確認等）をしてから教科書に載せる（検証していない数値を
書かない）。
