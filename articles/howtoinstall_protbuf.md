---
title: "Macで特定のバージョンのProtocol BuffersのInstall方法" # 記事のタイトル
emoji: "💻" # アイキャッチとして使われる絵文字（1文字だけ）
type: "tech" # tech: 技術記事 / idea: アイデア記事
topics: ["Mac", "gRPC", "Protocol Buffers", "go" "golang"] # タグ。["markdown", "rust", "aws"]のように指定する
published: true # 公開設定（falseにすると下書き）
---
# コマンド

以下のコマンドをベースに好きなバージョンに書き換えれば、Installできます。

```bash

PROTOC_ZIP=protoc-3.12.0-osx-x86_64.zip
curl -OL https://github.com/protocolbuffers/protobuf/releases/download/v3.12.0/$PROTOC_ZIP
sudo unzip -o $PROTOC_ZIP -d /usr/local bin/protoc
sudo unzip -o $PROTOC_ZIP -d /usr/local ‘include/*’
rm -f $PROTOC_ZIP
```

## 参考Web
http://google.github.io/proto-lens/installing-protoc.html

