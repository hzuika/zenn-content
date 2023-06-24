---
title: "DirectWriteで「フォントを列挙する方法」をwindows-rsで書く"
emoji: "🦀"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["rust", "windows"]
published: true
---

Microsoftのドキュメントには[フォントを列挙する方法](https://learn.microsoft.com/ja-jp/windows/win32/directwrite/font-enumeration)という記事が存在し，DirectWriteを使用してフォントファミリ名を列挙する方法について説明されています．
https://learn.microsoft.com/ja-jp/windows/win32/directwrite/font-enumeration

ここで説明された方法をRustの[windows crate](https://crates.io/crates/windows)を使用して書いてみます．

# `IDWriteFactory` を作成します

公式の記事ではいきなり`GetSystemFontCollection()`を呼び出してシステムフォントコレクションを取得するところから始めています．しかし，`GetSystemFontCollection()`は`IDWriteFactory`のメソッドなので，まずは`IDWriteFactory`を作成します．

`IDWriteFactory`を作成する方法については，Microsoftのドキュメント[IDWriteFactory インターフェイスの注釈](https://learn.microsoft.com/ja-jp/windows/win32/api/dwrite/nn-dwrite-idwritefactory#remarks)に記載されており，[`DWriteCreateFactory()`](https://learn.microsoft.com/ja-jp/windows/win32/api/dwrite/nf-dwrite-dwritecreatefactory)を使います．

[windows crateのドキュメント](https://microsoft.github.io/windows-docs-rs/doc/windows/index.html)から[`DWriteCreateFactory()`](https://microsoft.github.io/windows-docs-rs/doc/windows/Win32/Graphics/DirectWrite/fn.DWriteCreateFactory.html)を探します．ドキュメントには "Win32_Graphics_DirectWrite" が必要と記載されているので，`Cargo.toml`に追加します．

> Required features: "Win32_Graphics_DirectWrite"

```toml:Cargo.toml
[dependencies.windows]
version = "0.48"
features = [
    "Win32_Graphics_DirectWrite",
]
```

`DWriteCreateFactory()`の引数`factorytype`は，[IDWriteFactory インターフェイスの注釈](https://learn.microsoft.com/ja-jp/windows/win32/api/dwrite/nn-dwrite-idwritefactory#remarks)の例と同じ[`DWRITE_FACTORY_TYPE_SHARED`](https://microsoft.github.io/windows-docs-rs/doc/windows/Win32/Graphics/DirectWrite/constant.DWRITE_FACTORY_TYPE_SHARED.html)を指定しました．
```rust:main.rs
fn main() {
    unsafe {
        let dwrite_factory: IDWriteFactory =
            DWriteCreateFactory(DWRITE_FACTORY_TYPE_SHARED).unwrap();
    }
}
```

:::message
windows crate の関数の多くはunsafeです．
:::

# 手順 1: システム フォント コレクションを取得します

ここからは公式の記事と同じ手順です．[`GetSystemFontCollection()`](https://microsoft.github.io/windows-docs-rs/doc/windows/Win32/Graphics/DirectWrite/struct.IDWriteFactory.html#method.GetSystemFontCollection)を使用して，フォントコレクションを取得します．

Required featuresにしたがって，`Cargo.toml`に追加します．
> Required features: "Win32_Foundation"

```toml:Cargo.toml
features = [
    # 追加
    "Win32_Foundation",
]
```

引数`fontcollection`の型は，[C++](https://learn.microsoft.com/ja-jp/windows/win32/api/dwrite/nf-dwrite-idwritefactory-getsystemfontcollection)では`IDWriteFontCollection **`であるため，`NULL`を代入していました．[Rust](https://microsoft.github.io/windows-docs-rs/doc/windows/Win32/Graphics/DirectWrite/struct.IDWriteFactory.html#method.GetSystemFontCollection)では`*mut Option<IDWriteFontCollection>`になっているため，`None`を代入した変数の可変参照`&mut T`を渡します．

:::message
可変参照から可変ポインタ`*mut T`にキャストしたほうが分かりやすいかもしれないです．
```diff
-&mut fontcollection
+&mut fontcollection as *mut _
```
:::

引数`checkforupdates`の説明はwindows crateのドキュメントには記載されていないので，[Win32 APIのドキュメント](https://learn.microsoft.com/ja-jp/windows/win32/api/dwrite/nf-dwrite-idwritefactory-getsystemfontcollection)を参考にして `false` を指定しました．`checkforupdates`の型は`P0: IntoParam<BOOL>`となっており，Win32の[`BOOL`型](https://microsoft.github.io/windows-docs-rs/doc/windows/Win32/Foundation/struct.BOOL.html)を使用しても良いのですが，より簡潔にRustの`bool`型を使用しました．

`GetSystemFontCollection()`実行後は，`Option<IDWriteFontCollection>`の中身を取り出します．今回は`unwrap()`を使用しています．

```rust:main.rs
fn main() {
    unsafe {
        // 続き
        let mut fontcollection: Option<IDWriteFontCollection> = None;
        dwrite_factory
            .GetSystemFontCollection(&mut fontcollection, false)
            .unwrap();
        let fontcollection = fontcollection.unwrap();
    }
}
```

# 手順 2: フォント ファミリ数を取得します
[`GetFontFamilyCount()`](https://microsoft.github.io/windows-docs-rs/doc/windows/Win32/Graphics/DirectWrite/struct.IDWriteFontCollection.html#method.GetFontFamilyCount)を使用して，フォントファミリ数を取得します．

```rust:main.rs
fn main() {
    unsafe {
        // 続き
        let font_family_count = fontcollection.GetFontFamilyCount();
    }
}
```

# For Loopを作成します
取得したフォントファミリ数でループします．残りの手順はループ内で実行します．

```rust:main.rs
fn main() {
    unsafe {
        // 続き
        for i in 0..font_family_count {
            // 残りの手順
        }
    }
}
```

# 手順 3: フォント ファミリを取得します
[`GetFontFamily()`](https://microsoft.github.io/windows-docs-rs/doc/windows/Win32/Graphics/DirectWrite/struct.IDWriteFontCollection.html#method.GetFontFamily)を使用して，フォントファミリ([`IDWriteFontFamily`](https://microsoft.github.io/windows-docs-rs/doc/windows/Win32/Graphics/DirectWrite/struct.IDWriteFontFamily.html))を取得します．

```rust:main.rs
        for i in 0..font_family_count {
            let fontfamily = fontcollection.GetFontFamily(i).unwrap();
        }
```

# 手順 4: ファミリ名を取得します
[`IDWriteFontFamily`](https://microsoft.github.io/windows-docs-rs/doc/windows/Win32/Graphics/DirectWrite/struct.IDWriteFontFamily.html)の[`GetFamilyNames()`](https://microsoft.github.io/windows-docs-rs/doc/windows/Win32/Graphics/DirectWrite/struct.IDWriteFontFamily.html#method.GetFamilyNames)を使用して，[`IDWriteLocalizedStrings`](https://microsoft.github.io/windows-docs-rs/doc/windows/Win32/Graphics/DirectWrite/struct.IDWriteLocalizedStrings.html)型のファミリ名を取得します．

```rust:main.rs
        for i in 0..font_family_count {
            // 続き
            let familynames = fontfamily.GetFamilyNames().unwrap();
        }
```
# 手順 5: ロケール名を見つけます
[`IDWriteLocalizedStrings`](https://microsoft.github.io/windows-docs-rs/doc/windows/Win32/Graphics/DirectWrite/struct.IDWriteLocalizedStrings.html)の[`FindLocaleName()`](https://microsoft.github.io/windows-docs-rs/doc/windows/Win32/Graphics/DirectWrite/struct.IDWriteLocalizedStrings.html#method.FindLocaleName)を使用して，指定したロケールのファミリ名が存在するか(`exists`)，存在するとしたら何番目か(`index`)を取得します．

ロケールは最初にユーザのデフォルトロケールで探した後，見つからなければ`en-us`ロケールで再度探します．それでも見つからなければ0番目を指定します．ユーザのデフォルトロケールは[`GetUserDefaultLocaleName()`](https://microsoft.github.io/windows-docs-rs/doc/windows/Win32/Globalization/fn.GetUserDefaultLocaleName.html)を使用して取得します．引数`lplocalename`の型は`&mut [u16]`なので，長さが[`LOCALE_NAME_MAX_LENGTH`](https://microsoft.github.io/windows-docs-rs/doc/windows/Win32/System/SystemServices/constant.LOCALE_NAME_MAX_LENGTH.html)の`u16`の配列の可変参照を渡しています．私の環境ではデフォルトロケールは`"ja-JP"`でした．

:::message
C++では`wchar_t`ですが，Rustでは`u16`を使用します．
:::

[`LOCALE_NAME_MAX_LENGTH`](https://microsoft.github.io/windows-docs-rs/doc/windows/Win32/System/SystemServices/constant.LOCALE_NAME_MAX_LENGTH.html)と[`GetUserDefaultLocaleName()`](https://microsoft.github.io/windows-docs-rs/doc/windows/Win32/Globalization/fn.GetUserDefaultLocaleName.html)のRequired featuresを`Cargo.toml`に追加します．

```toml:Cargo.toml
features = [
    # 追加
    "Win32_System_SystemServices",
    "Win32_Globalization",
]
```

`GetUserDefaultLocaleName()`で取得したロケール(`[u16; _]`)から，`FindLocaleName()`の引数`localename`の型`P0: IntoParam<PCWSTR>`を作成します．`PCWSTR`の[`from_raw()`](https://microsoft.github.io/windows-docs-rs/doc/windows/core/struct.PCWSTR.html#method.from_raw)を使うと`*const u16`から`PCWSTR`を作成できます．また，[`w!()`](https://microsoft.github.io/windows-docs-rs/doc/windows/macro.w.html)マクロを使うと文字列リテラルから`PCWSTR`を作成できます．

引数`exists`の型は`*mut BOOL`なので，`false`から`BOOL`型を作り，可変参照を渡します．if条件式で使うために`BOOL`型を`bool`として扱う場合は[`as_bool()`](https://microsoft.github.io/windows-docs-rs/doc/windows/Win32/Foundation/struct.BOOL.html#method.as_bool)を使用します．
```rust:main.rs
        for i in 0..font_family_count {
            // 続き
            let mut index = 0_u32;
            let mut exists = BOOL::from(false);

            let mut localename = [0_u16; LOCALE_NAME_MAX_LENGTH as usize];
            let default_locale_success = GetUserDefaultLocaleName(&mut localename);

            if default_locale_success > 0 {
                familynames
                    .FindLocaleName(
                        PCWSTR::from_raw(localename.as_ptr()),
                        &mut index,
                        &mut exists,
                    )
                    .unwrap();
            }

            if !exists.as_bool() {
                familynames
                    .FindLocaleName(w!("en-us"), &mut index, &mut exists)
                    .unwrap();
            }
            
            if !exists.as_bool() {
                index = 0;
            }
        }
```

:::message
デフォルトロケールの取得はforループの外側で実行したほうが良さそうです．
:::

# 手順 6: ファミリ名の文字列を取得します
[`GetStringLength()`](https://microsoft.github.io/windows-docs-rs/doc/windows/Win32/Graphics/DirectWrite/struct.IDWriteLocalizedStrings.html#method.GetStringLength)を使用して，文字列の長さを取得します．その後，長さ分の文字列バッファを用意して，[`GetString()`](https://microsoft.github.io/windows-docs-rs/doc/windows/Win32/Graphics/DirectWrite/struct.IDWriteLocalizedStrings.html#method.GetString)を使用して文字列を取得します．

`PCWSTR`には`String`型に変換する[`to_string()`](https://microsoft.github.io/windows-docs-rs/doc/windows/core/struct.PCWSTR.html#method.to_string)メソッドが用意されているので，最後に`println!`で表示するために使用します．

```rust:main.rs
        for i in 0..font_family_count {
            let length = familynames.GetStringLength(index).unwrap();
            let mut name = vec![0_u16; length as usize + 1];
            familynames.GetString(index, &mut name).unwrap();
            let name = PCWSTR::from_raw(name.as_ptr()).to_string().unwrap();
            println!("{}", name);
        }
```

# ソースコード

```toml:Cargo.toml
[package]
name = "enum_fonts"
version = "0.1.0"
edition = "2021"

[dependencies.windows]
version = "0.48"
features = [
    "Win32_Graphics_DirectWrite",
    "Win32_Foundation",
    "Win32_System_SystemServices",
    "Win32_Globalization",
]

```

```rust:src/main.rs
use windows::{
    core::PCWSTR,
    w,
    Win32::{
        Foundation::BOOL,
        Globalization::GetUserDefaultLocaleName,
        Graphics::DirectWrite::{
            DWriteCreateFactory, IDWriteFactory, IDWriteFontCollection, DWRITE_FACTORY_TYPE_SHARED,
        },
        System::SystemServices::LOCALE_NAME_MAX_LENGTH,
    },
};

fn main() {
    unsafe {
        let dwrite_factory: IDWriteFactory =
            DWriteCreateFactory(DWRITE_FACTORY_TYPE_SHARED).unwrap();

        let mut fontcollection: Option<IDWriteFontCollection> = None;
        dwrite_factory
            .GetSystemFontCollection(&mut fontcollection, false)
            .unwrap();
        let fontcollection = fontcollection.unwrap();

        let font_family_count = fontcollection.GetFontFamilyCount();

        for i in 0..font_family_count {
            let fontfamily = fontcollection.GetFontFamily(i).unwrap();
            let familynames = fontfamily.GetFamilyNames().unwrap();

            let mut index = 0_u32;
            let mut exists = BOOL::from(false);

            let mut localename = [0_u16; LOCALE_NAME_MAX_LENGTH as usize];
            let default_locale_success = GetUserDefaultLocaleName(&mut localename);
            if default_locale_success > 0 {
                familynames
                    .FindLocaleName(
                        PCWSTR::from_raw(localename.as_ptr()),
                        &mut index,
                        &mut exists,
                    )
                    .unwrap();
            }

            if !exists.as_bool() {
                familynames
                    .FindLocaleName(w!("en-us"), &mut index, &mut exists)
                    .unwrap();
            }

            if !exists.as_bool() {
                index = 0;
            }

            let length = familynames.GetStringLength(index).unwrap();
            let mut name = vec![0_u16; length as usize + 1];
            familynames.GetString(index, &mut name).unwrap();
            let name = PCWSTR::from_raw(name.as_ptr()).to_string().unwrap();
            println!("{}", name);
        }
    }
}

```