<!--
# Strings
-->
# 文字列

<!--
There are two types of strings in Rust: `String` and `&str`.
-->
Rustには文字列を扱う型が2つあります。`String`と`&str`です。

<!--
A `String` is stored as a vector of bytes (`Vec<u8>`), but guaranteed to
always be a valid UTF-8 sequence. `String` is heap allocated, growable and not
null terminated.
-->
`String`は有効なUTF-8の配列であることを保証されたバイトのベクタ(`Vec<u8>`)として保持されます。ヒープ上に保持され、伸長可能で、末端にnull文字を含みません。

<!--
`&str` is a slice (`&[u8]`) that always points to a valid UTF-8 sequence, and
can be used to view into a `String`, just like `&[T]` is a view into `Vec<T>`.
-->
`&str`は有効なUTF-8の配列のスライス(`&[u8]`)で、いつでも`String`に変換することができます。`&[T]`がいつでも`Vec<T>`に変換できるのと同様です。

```rust,editable
fn main() {
    // (all the type annotations are superfluous)
    // A reference to a string allocated in read only memory
    // （以下の例では型を明示していますが、これらは必須ではありません。）
    // read only memory上に割り当てられた文字列への参照
    let pangram: &'static str = "the quick brown fox jumps over the lazy dog";
    println!("Pangram: {}", pangram);

    // Iterate over words in reverse, no new string is allocated
    // 単語を逆順にイテレートする。新しい文字列の割り当ては起こらない。
    println!("Words in reverse");
    for word in pangram.split_whitespace().rev() {
        println!("> {}", word);
    }

    // Copy chars into a vector, sort and remove duplicates
    // 文字をベクトルにコピー。ソートして重複を除去
    let mut chars: Vec<char> = pangram.chars().collect();
    chars.sort();
    chars.dedup();

    // Create an empty and growable `String`
    // 中身が空で、伸長可能な`String`を作成
    let mut string = String::new();
    for c in chars {
        // Insert a char at the end of string
        // 文字を文字列の末端に挿入
        string.push(c);
        // Insert a string at the end of string
        // 文字列を文字列の末端に挿入
        string.push_str(", ");
    }

    // The trimmed string is a slice to the original string, hence no new
    // allocation is performed
    // 文字列のトリミング（特定文字種の除去）はオリジナルの文字列のスライスを
    // 返すので、新規のメモリ割り当ては発生しない。
    let chars_to_trim: &[char] = &[' ', ','];
    let trimmed_str: &str = string.trim_matches(chars_to_trim);
    println!("Used characters: {}", trimmed_str);

    // Heap allocate a string
    // 文字列をヒープに割り当てる。
    let alice = String::from("I like dogs");
    // Allocate new memory and store the modified string there
    // 新しくメモリを確保し、変更を加えた文字列をそこに割り当てる。
    let bob: String = alice.replace("dog", "cat");

    println!("Alice says: {}", alice);
    println!("Bob says: {}", bob);
}
```

<!--
More `str`/`String` methods can be found under the
[std::str][str] and
[std::string][string]
modules
-->
`str`/`String`のメソッドをもっと見たい場合は[std::str][str]、[std::string][string]モジュールを参照してください。

## Literals and escapes

There are multiple ways to write string literals with special characters in them.
All result in a similar `&str` so it's best to use the form that is the most
convenient to write. Similarly there are multiple ways to write byte string literals,
which all result in `&[u8; N]`.

Generally special characters are escaped with a backslash character: `\`.
This way you can add any character to your string, even unprintable ones
and ones that you don't know how to type. If you want a literal backslash,
escape it with another one: `\\`

String or character literal delimiters occuring within a literal must be escaped: `"\""`, `'\''`.

```rust,editable
fn main() {
    // You can use escapes to write bytes by their hexadecimal values...
    let byte_escape = "I'm writing \x52\x75\x73\x74!";
    println!("What are you doing\x3F (\\x3F means ?) {}", byte_escape);

    // ...or Unicode code points.
    let unicode_codepoint = "\u{211D}";
    let character_name = "\"DOUBLE-STRUCK CAPITAL R\"";

    println!("Unicode character {} (U+211D) is called {}",
                unicode_codepoint, character_name );


    let long_string = "String literals
                        can span multiple lines.
                        The linebreak and indentation here ->\
                        <- can be escaped too!";
    println!("{}", long_string);
}
```

Sometimes there are just too many characters that need to be escaped or it's just
much more convenient to write a string out as-is. This is where raw string literals come into play.

```rust, editable
fn main() {
    let raw_str = r"Escapes don't work here: \x3F \u{211D}";
    println!("{}", raw_str);

    // If you need quotes in a raw string, add a pair of #s
    let quotes = r#"And then I said: "There is no escape!""#;
    println!("{}", quotes);

    // If you need "# in your string, just use more #s in the delimiter.
    // There is no limit for the number of #s you can use.
    let longer_delimiter = r###"A string with "# in it. And even "##!"###;
    println!("{}", longer_delimiter);
}
```

Want a string that's not UTF-8? (Remember, `str` and `String` must be valid UTF-8).
Or maybe you want an array of bytes that's mostly text? Byte strings to the rescue!

```rust, editable
use std::str;

fn main() {
    // Note that this is not actually a `&str`
    let bytestring: &[u8; 21] = b"this is a byte string";

    // Byte arrays don't have the `Display` trait, so printing them is a bit limited
    println!("A byte string: {:?}", bytestring);

    // Byte strings can have byte escapes...
    let escaped = b"\x52\x75\x73\x74 as bytes";
    // ...but no unicode escapes
    // let escaped = b"\u{211D} is not allowed";
    println!("Some escaped bytes: {:?}", escaped);


    // Raw byte strings work just like raw strings
    let raw_bytestring = br"\u{211D} is not escaped here";
    println!("{:?}", raw_bytestring);

    // Converting a byte array to `str` can fail
    if let Ok(my_str) = str::from_utf8(raw_bytestring) {
        println!("And the same as text: '{}'", my_str);
    }

    let _quotes = br#"You can also use "fancier" formatting, \
                    like with normal raw strings"#;

    // Byte strings don't have to be UTF-8
    let shift_jis = b"\x82\xe6\x82\xa8\x82\xb1\x82\xbb"; // "ようこそ" in SHIFT-JIS

    // But then they can't always be converted to `str`
    match str::from_utf8(shift_jis) {
        Ok(my_str) => println!("Conversion successful: '{}'", my_str),
        Err(e) => println!("Conversion failed: {:?}", e),
    };
}
```

For conversions between character encodings check out the [encoding][encoding-crate] crate.

A more detailed listing of the ways to write string literals and escape characters
is given in the ['Tokens' chapter][tokens] of the Rust Reference.

[str]: https://doc.rust-lang.org/std/str/
[string]: https://doc.rust-lang.org/std/string/
[tokens]: https://doc.rust-lang.org/reference/tokens.html
[encoding-crate]: https://crates.io/crates/encoding
