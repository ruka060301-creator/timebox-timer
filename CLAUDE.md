# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Overview

タイムボックス・タイマー(ポモドーロ風タイマー)。単一ファイル `index.html` に HTML/CSS/JS がすべて含まれる、ビルド・外部依存なしの静的 Web アプリ。GitHub Pages でホストされている(https://ruka060301-creator.github.io/timebox-timer/)。

## Development

- ビルド、パッケージマネージャ、テストフレームワークは存在しない。
- 変更を確認するには `index.html` をブラウザで直接開くか、任意の静的サーバーで配信する(例: `python3 -m http.server`)。
- リント/テストコマンドはない。

## Architecture (`index.html`)

すべてが1ファイルにまとまっている(IIFE内のバニラJS、`<style>`内のCSS)。

- **状態**: `mode`(`work`/`short`/`long`)、`remaining`/`total`(秒)、`running`、`cfg`(作業・小休憩・長休憩の分数、長休憩までのポモ数、自動開始、終了音)、`stats`(完了ポモ数・集中分・サイクル数、当日のみ)をトップレベル変数で保持。
- **タイマーループ**: `setInterval` のドリフトを避けるため、`endAt = Date.now() + remaining*1000` を基準に残り時間を都度再計算する(`tick()`)。単純なカウントダウン減算はしていない。
- **永続化**: `localStorage` キー `timebox.v1` に `{ cfg, stats, day }` を保存/復元。`stats` は `day` が今日の日付と一致する場合のみ復元する(日付が変わるとリセット)。
- **セッション遷移**: `complete()` が作業セッション終了時に `stats.done` をインクリメントし、`stats.done % cfg.interval === 0` で長休憩か小休憩かを判定して次のモードへ遷移。`cfg.auto` が true なら自動的に次のセッションを開始。
- **描画**: `render()` が単一の入口となり、時間表示・SVGリング(`stroke-dashoffset`)・ボタン状態・統計・タブタイトルをまとめて更新する。状態を変えたら `render()` を呼ぶ、という規約。
- **通知**: セッション終了時に Web Audio API でチャイム音を生成(`chime()`、`cfg.sound` が true の場合)、かつ非フォーカス時にタブタイトルを点滅させる(`flashTitle()`/`clearTitleFlash()`、ウィンドウフォーカスで解除)。
- **配色**: ライト/ダークは CSS カスタムプロパティ + `prefers-color-scheme` で切り替え(OS設定に追従、JSでの切り替えロジックはない)。作業/休憩の配色は `body.break` クラスの有無で切り替える。
- **キーボード操作**: `Space`(開始/一時停止)、`R`(リセット)、`S`(スキップ)。入力欄にフォーカスがある間は無効化される。
