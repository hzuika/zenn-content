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

(テキストオブジェクトは少しずつ実装していくかもしれません)

> Linuxでは処理が別なので，全部サポートされています．

UIのテキストボタンで実装されているIME関連の処理と同様の処理を実装することで他の場所でも日本語入力ができるようになります．

# 準備
Blenderの開発環境を準備します．
基本的に公式Wikiの通りに進めていくとよいかと思います．

[Building Blender - Blender Developer Wiki](https://wiki.blender.org/wiki/Building_Blender)

> 今後，記事を作るかもしれません．

# IME関連の処理は`#WITH_INPUT_IME`マクロを探す
ソースコード中の`#WITH_INPUT_IME`マクロによって，ビルド時にIMEサポートを切り替えることができます(デフォルトで有効)．このマクロを探すことで，IME関連の処理を簡単に見つけることができます．
ただし，ソースコードにIME処理を追加したときは，CMakeLists.txtにWITH_INPUT_IMEマクロに関する記述が必要です．

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
  size_t result_len, composite_len; // 各文字列の長さ(utf-8なのでascii1文字は1，日本語1文字は3)
  char *str_result; // 変換結果の文字列(utf-8)
  char *str_composite; // コンポジション文字列(utf-8)
  int cursor_pos; // IME変換中のカーソル位置(utf-8換算で0が先頭．日本語であれば3の倍数)
  int sel_start; // IME変換中の選択した文字列の範囲(utf-8換算で0が先頭)
  int sel_end; // 雨が|降る|ように (||で囲まれた範囲の場合 sel_start==6, sel_end==12)
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

## DebugビルドとReleaseビルド

`make`でReleaseビルドができて，`make debug`でDebugビルドができます．

ここからは，個人的な使い分けですが．．．
Releaseビルドの方が体感早い気がするので，`printf`書いてReleaseビルドで確認した方が早い時もあります．
バグが起きたり，動作がわからなくなったりしたら，Debugビルドをして，デバッガを使ったりします．

> 自分はninjaとccacheを使っているのでコマンドは`make ccache ninja`と`make debug ccache ninja`になります．
> 比較していないのですが，体感ビルド時間が早いような気がします．

---

# テキストオブジェクトに実装するには

## IMEの有効化と無効化

こんな感じでしょうか．
何も入力できませんが，変換ウィンドウだけ表示されると思います．

https://github.com/hzuika/blender/commit/94ce267462f2383bfef3a7c971476e59b03c2c1c

テキストオブジェクトであれば、編集モードに入ったときがテキスト入力の開始で、オブジェクトモードになったときが、テキスト入力の終了になるでしょう．
編集モード関連の処理はobject_edit.cにあるので、テキストオブジェクトを表す`OB_FONT`で検索すると見つけられると思います。

`wm_window_IME_begin`と`wm_window_IME_end`の引数に`wmWindow`が必要で，これは`wmWindow *CTX_wm_window(const bContext *C)`を使用して取得します．
つまり，`C`があるところでないと実行できません．

UIやTabキーを使って，編集モードに入る処理と出る処理は次の関数で実行されるようですね．
`static int editmode_toggle_exec(bContext *C, wmOperator *op)`

> * `bool ED_object_editmode_enter(bContext *C, int flag)`かと思いましたが，この関数はどこにも参照されていないようです．
> * `bool ED_object_editmode_exit(bContext *C, int flag)`も`object_batch_delete_hierarchy_fn`でしか参照されていないので，削除処理でしか使わないみたいです．

## INE関連のイベント処理

こんな感じでしょうか．
変換中は文字列が表示されず，変換ウィンドウの位置もおかしくなっていますが，変換結果の文字列は入力できると思います．

https://github.com/hzuika/blender/commit/f904fd8d5ec1661ce9a9b66af346d45575a511ee

テキストオブジェクトの入力処理はeditfont.cのinsert_text_invoke内で行われます．

> insert_text_execはキーボード入力では呼ばれないようです．

しかし、そのままでは`WM_IME_CONPOSITE_*`イベントがinvoke関数まで来ないようです．．
wm_eventmatch関数内でIMEイベントも含めるように処理を追加します．

## IMEウィンドウの移動

カーソルの下にウィンドウを表示します．
テキストオブジェクトのカーソル(長方形)の4点のローカル座標(x, y)は`EditFont`の`textcurs[0][8]`に左下から反時計回りに格納されています．
