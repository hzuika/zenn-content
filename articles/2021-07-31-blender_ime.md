---
title: "Blenderで日本語入力を実装するには"
emoji: "❄️"
type: "idea" # tech: 技術記事 / idea: アイデア
topics: ["Blender", "OSS", "ime", "C言語"]
published: true
---

Blenderの日本語入力を改善したいと思っています．しかし，まとまった時間がとれないので考え方だけまとめておきます．これを見て取り組んでくれる方がいると嬉しいです．

# Blenderの日本語入力(IME)サポートの現状

2021/07/31時点の Widnows/Mac での日本語入力のサポート状況です．
* [x] UIのテキストボタン `source/blender/editors/interface/`
* [ ] テキストオブジェクト `source/blender/editors/curve/`
* [ ] テキストエディタ `source/blender/editors/space_text/`
* [ ] Pythonコンソール `source/blender/editors/space_console/`

> Linuxでは処理が別なので，全部サポートされています．

UIのテキストボタンで実装されているIME関連の処理と同様の処理を実装することで他の場所でも日本語入力ができるようになります．

# 準備
Blenderの開発環境を準備します．
基本的に公式Wikiの通りに進めていくとよいかと思います．

[Building Blender - Blender Developer Wiki](https://wiki.blender.org/wiki/Building_Blender)

> 今後，記事を作るかもしれません．

# IME関連の処理は`#WITH_INPUT_IME`マクロを探す
ソースコード中の`#WITH_INPUT_IME`マクロによって，ビルド時にIMEサポートを切り替えることができます(デフォルトで有効)．このマクロを探すことで，IME関連の処理を簡単に見つけることができます．

# IME用語

* コンポジション文字列
    * 変換中の文字列
* 変換結果の文字列
    * Enterを押して変換を確定した文字列

例えば，`雨`と入力したい場合，次のように推移していきます．
```
(コンポジション文字列) あ → あｍ → あめ → 飴 → 雨
<Enter>
(変換結果の文字列) 雨
```

Blenderでは`wmIMEData`構造体にIME関連の情報が格納されています．
```cpp
typedef struct wmIMEData {
  size_t result_len, composite_len;
  char *str_result; // 変換結果の文字列
  char *str_composite; // コンポジション文字列
  int cursor_pos; // IME変換中のカーソル位置
  int sel_start; // IME変換中の選択した文字列の範囲
  int sel_end;
  bool is_ime_composing; // IME変換中かどうかを表すフラグ
} wmIMEData;
```

# 実装

## IMEの有効化と無効化

1. テキスト入力を開始するときに，`wm_window_IME_begin`を呼び出し，IMEを有効にします．
2. テキスト入力を終了するときに，`wm_window_IME_end`を呼び出し，IMEを無効にします．

テキストボタンでは次の関数でこの処理が行われます．
|処理|関数|
|-|-|
|テキスト入力開始|`ui_textedit_begin`|
|IME有効化|`ui_textedit_ime_begin`|
|テキスト入力終了|`ui_textedit_end`|
|IME無効化|`ui_textedit_ime_end`|

## IME関連のイベント処理
テキスト入力中にIME関連のイベントを処理します．

* `WM_IME_COMPOSITE_START`
    * 変換開始(選択範囲の削除)
* `WM_IME_COMPOSITE_EVENT`
    * 変換中(変換が確定した結果文字列の挿入)
* `WM_IME_COMPOSITE_END`
    * 変換終了(特に何もしない)

テキストボタンでは`ui_do_but_textedit`関数内でこの処理が行われます．

## コンポジション文字列の描画

変換中のコンポジション文字列を描画します．

テキストボタンでは`widget_draw_text`関数内でこの処理が行われます．

:::message info
コンポジション文字列もテキストデータの中に挿入する場合は，この処理は必要ありません．その代わり，変換を確定して結果文字列を挿入する際に，コンポジション文字列を削除する必要があります．
:::

## 下線の描画
コンポジション文字列を他と区別するために，文字列に下線を描画します．また，コンポジション文字列の中から変換する範囲を選択できるので，そこだけ他の下線と区別するために，太い下線を描画します．

テキストボタンでは`widget_draw_text_ime_underline`関数がこの処理を行います．

## IMEウィンドウの移動

IMEの変換ウィンドウの位置を指定するために，`wm_window_IME_begin`を呼び出します．

テキストボタンでは`ui_but_ime_reposition`関数がこの処理を行います．

# 補足
## キーボード入力イベント
キーボード入力イベントは次の順に処理されます．
```
OS → GHOST → BlenderのWindow Manager → Blenderの各種エリア

例: テキストボタン
OS(Windows/Mac) → GHOST → Window Manager → テキストボタン(UI)
```

テキストボタンの処理と同様の処理を実装することで，他のエリアで日本語を入力することができます．

> GHOSTとは?
> General Handy Operating System Toolkitの略で，OS固有の処理を抽象化したライブラリです．Blenderは各OSのAPIではなく，GHOSTのAPIを利用します．
> 参考: [Source/File Structure - Blender Developer Wiki](https://wiki.blender.org/wiki/Source/File_Structure)
> ソースコード: `/intern/ghost/`