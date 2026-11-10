# FCStd ビューワー

FreeCADのファイル(.FCStd)をiPadやスマホのブラウザでそのまま表示するビューワーです。
開いたファイルは閲覧者の端末のブラウザ内だけで処理され、サーバーには送信されません。

## ファイル構成

| ファイル | 内容 |
|---|---|
| index.html | ビューワー本体 |
| occt-import-js.wasm | 形状エンジン(OpenCascade)。index.html と同じ場所に置いてください |
| icon.png | ホーム画面に追加したときのアイコン |
| LICENSE-*.txt | 形状エンジンのライセンス(LGPL)。削除せず一緒に置いてください |

## GitHub Pagesでの公開手順

1. GitHubにログインし、右上の「+」→「New repository」で新しいリポジトリを作る
   - 名前の例: `fcstd-viewer`
   - 公開設定は **Public**(無料プランでPagesを使う場合)
2. 作成後の画面で「uploading an existing file」を押し、このフォルダの中身をすべてドラッグ&ドロップして「Commit changes」
   - occt-import-js.wasm は約7.7MBです。ブラウザからのアップロード上限(25MB)内なのでそのまま上げられます
3. リポジトリの「Settings」→ 左メニュー「Pages」を開く
4. 「Build and deployment」の Source を **Deploy from a branch**、Branch を **main** / **(root)** にして「Save」
5. 1〜2分待つと、同じ画面の上部に公開URLが表示されます
   - 例: `https://<ユーザー名>.github.io/fcstd-viewer/`

## 使い方(渡す相手向け)

1. iPadのSafariで公開URLを開く
2. 共有ボタン →「ホーム画面に追加」でアプリのように使えます
3. 右上の「FCStdを開く」から、ファイルアプリ(iCloud Drive等)の .FCStd を選ぶ

- 1本指: 回転 / 2本指: ズーム・移動 / タップ: 部品を選択
- 下のツールバー: 全体表示、視点切替、断面、2点間の距離計測、表示モード切替

## 注意

- 表示用のライブラリ(three.js、JSZip、occt-import-js)をインターネットのCDNから読み込むため、使用時はネット接続が必要です
- スケッチのみのファイル、外部ファイルへのリンクは表示できません
