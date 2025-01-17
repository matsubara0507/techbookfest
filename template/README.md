# techbookfest-template

[Re:VIEW](https://github.com/kmuto/review) と言う組版システムを使います。
手元でビルドする場合は、Docker があると便利です。

## PDF の作り方

### clone

```
$ git clone git@github.com:mixi-inc/techbookfest-template.git
```

### Docker

[vvakame/review](https://hub.docker.com/r/vvakame/review/) のイメージをベースにして、textlint による静的解析を行うために諸々インストールしたイメージです。

```
$ docker build -t techbookfest .
```

### ビルド

上記の Docker イメージを利用して静的検査して `book.pdf` が作られる

```
$ docker run --rm -v `pwd`:/work techbookfest /bin/sh -c "cd /work && textlint *.re && review-pdfmaker config.yml"
```

`pwd` の部分はリポジトリまでの絶対パス

## 初稿の出し方

1. master から適当にブランチを切ります
2. トップに `namae.re` をおきます
3. `catalog.yml` の `CHAPS` に `namae.re` を追記します
4. 執筆します
    - フォーマットは [Re:VIEW (v3.2)](https://github.com/kmuto/review/tree/v3.2.0)
    - 画像類は `image` ディレクトリに適当にサブディレクトリを作っておいてください
5. `atogaki.re` にあとがきを書きます(例は `atogaki.re` に書いてある)
6. プッシュして PR を作ります(特にフォーマットはない)
7. 誰かがレビューして、添削して、マージされます

(`namae.re` は各位で適当に読み替えてネ)

### vs. textlint

PR を出した時に CodeBuild 上で [textlint](https://github.com/textlint/textlint) による自動校正を行なっています。

エラー文言の通りに直せるのならば特に問題ないのですが、場合によっては(もともと引用の文言であるとか宗教上の理由とか) textlint の校正結果を無視したいでしょう。
その方法はいくつかあるので以下に列挙しておきます:

1. `prh.yaml` を書き換える
    - ただし、他の人も影響する点に注意
2. `#@# textlint-disable` `#@# textlint-enable` で対象の行を囲む
    - ```
      #@# textlint-disable

      ここは textlint が無視される

      #@# textlint-enable
      ```
    - `atogaki.re` ではそのようにしています
    - 空行を挟む必要がある点に注意

他にも方法があれば追記して ;)

## 設定

CI などのためにスクリプトに下記の値を追加する

- script/exec_textlint.sh の `REPO` にリポジトリ名 (org/repo)
- script/exec_textlint.sh の `GH_TOKEN` に GitHub のトークン (シークレットなどで)
- script/upload-pdf-to-slack の `REPO` にリポジトリ名 (org/repo)
- script/upload-pdf-to-slack の `SLACK_CHANNEL` に通知したい Slack チャンネル名
- script/upload-pdf-to-slack の `SLACK_TOKEN` に通知したい Slack のトークン (シークレットなどで)

## 表紙と裏表紙

電子版を作成するときには表紙と裏表紙を設定します。
設定するには `config.yml` の `images/cover.png` と `images/backcover.png` を配置して `coverimage` と `backcover` のコメントアウトを外すだけです。
