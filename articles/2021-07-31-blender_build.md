---
title: ""
emoji: "🐡"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: []
published: false
---

## 開発ツールの用意
Windowsの場合，[scoop]([lukesampson/scoop: A command-line installer for Windows.](https://github.com/lukesampson/scoop))でインストールすると楽です．
```
scoop install git cmake sliksvn
```

[Visual Studio](https://visualstudio.microsoft.com/)はMicrosoftのWebページからインストールします．
C++によるデスクトップ開発をインストールします．

## Blenderのビルド

```ps1
# 適当なフォルダをつくります．
mkdir C:\blender-git
cd C:\blender-git
# ソースコードをダウンロードします．
git clone git:://git.blender.org/blender.git
# 移動します．
cd C:\blender-git\blender
# ライブラリをダウンロードします．
make update
# インストールするか尋ねられるので， y を入力します．
y
# ビルドします．
make
# Powershellの場合 make ではなく， .\make.batとします．
.\make.bat update
.\make.bat
```

