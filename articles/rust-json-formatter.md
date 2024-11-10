---
title: "Rust 入門がてら JSON フォーマッターを書く"
emoji: "😺"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["Rust"]
published: false
---

# これはなに？

Rust 全然わからんフロントエンドエンジニアが、昨今のフロントエンド領域への Rust の進出の勢いに影響を受けて、とりあえず Rust 学んでみるかと思い JSON フォーマッターを作ってみた記録。

**Rust ビギナーが AI に頼りつつ雰囲気で実装したものなので、この記事で Rust を学ぼうとは思わないでください。**

同じように Rust に興味を持っている程度の人が、とりあえず何から始めたら良いかの参考とか刺激になればと思います。

# 前提

Rust そのものの学習は事前に行っており、そちらはスクラップに記録してます。
https://zenn.dev/sa2knight/scraps/b32fbd7c5a3f2e

よって、本記事では Rust のインストールであるとか、基礎文法については触れません。

# 作ったもの

標準入力から JSON 文字列を受け取って、それを字句解析(Lexer)、構文解析(Parser)し、整形(Formatter) したものを標準出力するだけの、N 番煎じの車輪の再発明です。

動作例

```bash
$ echo "{\"key1\":10,\"key2\":[1,2,{\"key3\": [\"Hello\",true, false, null]}]}" | cargo run
{
  "key1": 10,
  "key2": [
    1,
    2,
    {
      "key3": [
        "Hello",
        true,
        false,
        null
      ]
    }
  ]
}
```

なお、本当に最小限のものしか作ってないので、エラーハンドリングなどの当たり前品質も満たしていません。

では自らのおさらいも兼ねて、実装の手順とコード(一部)を掲載します。

# プロジェクト作成

cargo でプロジェクトを作成します。パッケージマネージャーやリンター、テストランナーまで含めてオールインワンというのは良いものですね。

```bash
$ cargo new json-formatter
    Creating binary (application) `json-formatter` package
```

`Cargo.toml` (package.json みたいなやつ) をいじって、依存クレート (npm みたいなやつ) として `indexmap` を追加します。HashMap (object みたいなキーバリュー) に順序を持たせられるやつで、後半で使いたくなりました。

```toml:Cargo.toml
[package]
name = "json-formatter"
version = "0.1.0"
edition = "2021"

[dependencies]
indexmap = "2.6.0"
```

`rustfmt.toml` (prettierrc みたいなやつ)もいじって、一行の最大文字数を伸ばします。いきなりこれを弄るのはアンチパターンな気配もしますが、普段 Prettier でも 120 文字が好みなので…。

```toml:rustfmt.toml
max_width = 120
```

とりあえずスキャフォルとされてる Hello, World が動きます。

```bash
$ cargo run
   Compiling equivalent v1.0.1
   Compiling hashbrown v0.15.1
   Compiling indexmap v2.6.0
   Compiling json-formatter v0.1.0 (/Users/shingo.sasaki/json-formatter)
    Finished `dev` profile [unoptimized + debuginfo] target(s) in 0.79s
     Running `target/debug/json-formatter`
Hello, world!
```

# 字句解析(Lexer)

はじめに JSON 文字列を字句解析する仕組みを作ります。

字句解析では、例えば `{ "age": 30 }` のような文字列を以下のように軸ごとに解析します。

- `{` (LeftBrace)
- `age` (String)
- `:` (Colon)
- `30` (Number)
- `}` (RightBracket)

字句解析のコードは `src/lexer.rs` に書いていきます。

## トークンの定義

まずは字句解析の最小単位であるトークンを定義します。

JSON を構成するのはトークンは(おそらく)これだけです。String と Number のみ、具体的な関連データを持ちますが、ほかは対応する文字(列)が固定で決まっています。

```rust:lexer.rs
pub enum Token {
    LeftBrace,      // {
    RightBrace,     // }
    LeftBracket,    // [
    RightBracket,   // ]
    Colon,          // :
    Comma,          // ,
    String(String), // "string"
    Number(f64),    // 123, 45.67
    True,           // true
    False,          // false
    Null,           // null
}
```

## 字句解析の下準備

実際に字句解析を行う、Lexer 型を定義します。対象文字列全体と、現在及び次の文字の情報を持ちます。文字を順に調べていって、トークンを発見していくわけです。

```rust:lexer.rs
pub struct Lexer<'a> {
    input: &'a str,       // 字句解析対象の文字列全体
    position: usize,      // 解析中の現在の文字位置
    read_position: usize, // 解析中の次の文字位置
    ch: Option<char>,     // 現在解析中の文字 (None は EOF)
}
```

Lexer 型を実装していきます。まずはイニシャライザで、初期化。
ライフタイムパラメータ (`<'a>`) は正直よくわかってないです。コンパイルを通すために手探りで書いてます。

```rust:lexer.rs
impl<'a> Lexer<'a> {
    /**
     * 新しい Lexer を生成する
     */
    pub fn new(input: &'a str) -> Self {
        let mut lexer = Lexer {
            input,
            position: 0,
            read_position: 0,
            ch: None,
        };
        lexer.read_char();
        return lexer;
    }
```

インスタンスの初期化時点では何も解析していない状態なので、以下の `read_char()` メソッドを一度呼び出し、1 文字目を解析してる状態に進めます。

```rust:lexer.rs
/**
 * 次の文字を読み込み、現在の位置を更新する
 */
fn read_char(&mut self) {
    // 既に末尾の場合、終了する
    if self.read_position >= self.input.len() {
        self.ch = None;
    }
    // 次の文字があれば読み出す
    else {
        self.ch = self.input[self.read_position..].chars().next();
    }
    // 位置を進める(対象文字のバイト数分進める)
    self.position = self.read_position;
    self.read_position += self.ch.map_or(0, |c| c.len_utf8());
}
```

`read_char` を呼び出すたびに、インスタンスの `ch` に次の文字が入ってくるので、これを用いてトークンを見つけ出していきます。

ただし、テキスト中に含まれるスペースは JSON を解析するうえではノイズでしかないので、それをスキップする `skip_whitespace` メソッドも用意しておきます。(※文字列リテラル中にスペースが含まれる場合もありますが、そういった場面では呼び出しません)

```rust:lexer.rs
 /**
  * ホワイトスペースの間は読み飛ばす
  */
 fn skip_whitespace(&mut self) {
     while let Some(ch) = self.ch {
         if ch.is_whitespace() {
             self.read_char();
         } else {
             break;
         }
     }
 }
```

## 単純なトークンを取り出す

では `read_char()` 及び `skip_whitespace` を用いて、「次のトークンを取り出す」メソッドを実装します。

```rust:lexer.rs
/**
 * 次のトークンを取得する
 */
pub fn next_token(&mut self) -> Option<Token> {
    self.skip_whitespace();
    let token: Option<Token> = match self.ch {
        Some('{') => {
            self.read_char();
            Some(Token::LeftBrace)
        }
        Some('}') => {
            self.read_char();
            Some(Token::RightBrace)
        }
        Some('[') => {
            self.read_char();
            Some(Token::LeftBracket)
        }
        Some(']') => {
            self.read_char();
            Some(Token::RightBracket)
        }
        Some(':') => {
            self.read_char();
            Some(Token::Colon)
        }
        Some(',') => {
            self.read_char();
            Some(Token::Comma)
        }
        Some('"') => {
            // TODO: 文字列の処理を実装する
            let string = self.read_string();
            Some(Token::String(string))
        }
        Some(c) if c.is_digit(10) || c == '-' ||
            // TODO: 数値の処理を実装する
            None
        }
        Some(c) if c.is_alphabetic() => {
            // TODO: true/false/null の処理を実装する
            None
        }
        _ => None
    };
    return token;
}
```

急に長いコードになりましたが、やってることは単純で、現在の文字が特定のトークンに対応するパターンである場合、トークンを取り出すまで文字を進めて、トークンを返しているだけです。

ここで

- `{`
- `}`
- `[`
- `]`
- `:`
- `,`

については、出現した文字だけでトークンが完成するので、文字を見つけた時点で完了です。

一方、 `"` は文字列の気配がしますが、文字列トークンを完成させるには続きの文字も読み進める必要があります。数値や true/false/null も同様のため、それらは別の関数を作成します。

## 複雑なトークンを取り出す

まず文字列トークンを取り出す `read_string` 関数を定義します。

`read_string` 関数は、文字列の開始点である `"` を発見したところで呼び出されるので、次の `"` が出現されるまでの文字列を取り出せば OK です。 (正確には `\n` などのエスケープシーケンスの考慮が必要ですが、ここでは割愛しています)

```rust:lexer.ts
/**
 * 文字列リテラルを読み取る
 * `"` から `"` までの文字列を読み取る
 */
fn read_string(&mut self) -> String {
    let mut result = String::new();
    self.read_char(); // 現在地が先頭の `"` なので読み飛ばす
    while let Some(ch) = self.ch {
        if ch == '"' {
            // 文字列の終端の場合そこで終了
            self.read_char();
            break;
        } else {
            // 通常の文字の場合はそのまま追加
            result.push(ch);
        }
        self.read_char(); // 次の文字へ
    }
    return result;
}
```

次に true/false/string を取り出す `read_literal` 関数です。これらはアルファベットのみで構成されるトークンなので、アルファベットが続く限りの文字列を抜き出します。

```rust:lexer.rs
/**
 * リテラル (true, false, null) を読み取る
 */
fn read_literal(&mut self) -> Option<Token> {
    let mut string = String::new();
    while let Some(ch) = self.ch {
        if ch.is_alphabetic() {
            string.push(ch);
            self.read_char();
        } else {
            break;
        }
    }
    return match string.as_str() {
        "true" => Some(Token::True),
        "false" => Some(Token::False),
        "null" => Some(Token::Null),
        _ => None, // 未知のリテラルは無視する
    };
}
```

できあがった文字列が "true" | "false" | "null" に合致していた場合、それらのトークンを返して終わりです。

最後に数値です。`read_number` 関数で実装しますが、数値は負数や小数、指数でも表現可能なため、それらの文字にも注意します。

```rust:lexer.rs
/**
 * 数値リテラルを読み取る
 * 数値または "-" から始まる数値文字列を読み取る
 */
fn read_number(&mut self) -> String {
    let mut result = String::new();
    while let Some(ch) = self.ch {
        if ch.is_digit(10) || ch == '.' || ch == '-' || ch == '+' || ch == 'e' || ch == 'E' {
            result.push(ch);
            self.read_char();
        } else {
            break;
        }
    }
    return result;
}
```

出来上がった `read_string` `read_literal` `read_number` を、それぞれ `next_token` 関数から呼び出すようにします。数値のみ、文字列を受け取っているので数値(f64)にパースしたものをトークンとします。

```rust:lexer.rs
/**
 * 次のトークンを取得する
 */
pub fn next_token(&mut self) -> Option<Token> {
    self.skip_whitespace();
    let token: Option<Token> = match self.ch {
        // 中略
        Some('"') => {
            let string = self.read_string()
            Some(Token::String(string))
        }
        Some(c) if c.is_digit(10) || c == '-' || c == '+' => {
            let string = self.read_number();
            if let Ok(number) = string.parse::<f64>() {
                Some(Token::Number(number))
            } else {
                None
            }
        }
        Some(c) if c.is_alphabetic() => {
            return self.read_literal();
        }
        // 中略
    }
}
```

これで字句解析は完成です。 これで、`{ "age": 30 }` のような文字列で Lexer 型のインスタンスを生成し、`next_token` メソッドを何度か呼び出すことで、以下のようなトークン一覧を取り出せるようになりました。

- `{` (LeftBrace)
- `age` (String)
- `:` (Colon)
- `30` (Number)
- `}` (RightBracket)

# 構文解析(Parser)

次は構文解析です。字句解析で得られたトークンを元に、オブジェクトや配列、あるいはリテラルの組み合わせの構文を解析していきます。

## JSON の定義

まずは JSON の構文を enum で定義します。

```rust:json.rs
use indexmap::IndexMap;

pub enum JsonValue {
    Object(JsonObject), // {"key": "value"}
    Array(JsonArray),   // [1, 2, 3]
    String(String),     // "hello, world"
    Number(f64),        // 123.456
    True,               // true
    False,              // false
    Null,               // null
}

pub type JsonObject = IndexMap<String, JsonValue>;
pub type JsonArray = Vec<JsonValue>;
```

例えば `{ "key1": 10, "key2": [ 1, 2, { "key3": [ "Hello", true ] } ] }` は、`JsonValue::Object` に合致し、 `key1` に対応する値は `JsonValue::Number(10)` で、 `key2` に対応する値は `JsonValue::Array` となり、後は再帰的に解決できます。

構文解析では、トークンを辿りながら、 `JsonValue` を組み立てていきます。

## パーサーの定義

構文解析は `Parser` 型で行います。 Parser は `src/parser.rs` に実装します。

```rust:parser.rs
pub struct Parser<'a> {
    lexer: Lexer<'a>,
    current_token: Option<Token>,
}

impl<'a> Parser<'a> {
    /**
     * 新しい Parser を生成する
     */
    pub fn new(lexer: Lexer<'a>) -> Self {
        let mut parser = Parser {
            lexer: lexer,
            current_token: None,
        };
        parser.next_token();
        return parser;
    }
}
```

Parser インスタンスは、Lexer を用いて現在のトークンを保持します。 Lexer のイニシャライザと同様、インスタンス生成時点で最初のトークンを取得しておきます。

## 単純な構文をパースする

基本は字句解析のときと同じ仕組みです。現在のトークンに応じて、どの構文に合致するかをパターンマッチして、`JsonValue` のいずれかを返していけば OK です。

```rust:parser.rs
/**
 * JSON値をパースする
 */
fn parse(&mut self) -> Option<JsonValue> {
    match &self.current_token {
        Some(Token::LeftBrace) => {
            // TODO: オブジェクトをパースする
            None
        }
        Some(Token::LeftBracket) => {
            // TODO: 配列をパースする
            None
        }
        Some(Token::String(string)) => {
            let cloned_string = string.clone();
            self.next_token();
            Some(JsonValue::String(cloned_string))
        }
        Some(Token::Number(number)) => {
            let copied_number = *number;
            self.next_token();
            Some(JsonValue::Number(copied_number))
        }
        Some(Token::True) => {
            self.next_token();
            Some(JsonValue::True)
        }
        Some(Token::False) => {
            self.next_token();
            Some(JsonValue::False)
        }
        Some(Token::Null) => {
            self.next_token();
            Some(JsonValue::Null)
        }
        _ => None,
    }
}
```

String/Number/True/False/Null は、字句解析で取り出したトークンの時点で既にリテラル値として完成しているため、そのまま JsonValue として返せます。

一方で、`Token::LeftBrace` (`{`) 及び `Token::LeftBracket` (`[`) が現れた場合、それぞれオブジェクト、配列としてパースする必要がある上、中身によっては再帰的にパースしていく必要があるため、個別に対応していきます。

## 複雑な構文をパースする

`Token::LeftBracket` (`[`) が見つかったところから呼び出される `parse_array` 関数を実装します。

```rust:parser.rs
/**
 * 配列をパースする
 */
fn parse_array(&mut self) -> Option<JsonValue> {
    let mut array: JsonArray = Vec::new();
    // 先頭の [ を読み飛ばす
    self.next_token();
    // すぐに ] が来る場合は空配列として即終了
    if let Some(Token::RightBracket) = self.current_token {
        self.next_token();
        return Some(JsonValue::Array(array));
    }
    // 配列の要素の数だけループする
    loop {
        // value (値がオブジェクトや配列である場合のためにここで再帰する)
        if let Some(value) = self.parse() {
            array.push(value);
        }
        // , なら次の要素に続き ] が来たらループ終了
        match &self.current_token {
            Some(Token::Comma) => {
                self.next_token();
            }
            Some(Token::RightBracket) => {
                self.next_token();
                break;
            }
            _ => return None,
        }
    }
    return Some(JsonValue::Array(array));
}
```

`Token::RightBracket` (`]`) が出現するまで配列は繰り返されますが、各要素の中身がなんなのかもパースする必要があるため、`parse` 関数を再帰的に呼び出します。これによって配列の中に配列がある場合でも、`JsonValue::Array` にマッチした値を取り出せます。

同様に `Token::LeftBrace` (`{`) が見つかった場合は `parse_object` 関数を呼び出します。作りは概ね `parse_array` と同じです。

```rust:parser.rs
/**
 * オブジェクトをパースする
 * 現在のトークンが { であることが前提
 */
fn parse_object(&mut self) -> Option<JsonValue> {
    let mut object: JsonObject = IndexMap::new();
    // 先頭の { を読み飛ばす
    self.next_token();
    // すぐに } が来る場合は空オブジェクトとして即終了
    if let Some(Token::RightBrace) = self.current_token {
        self.next_token();
        return Some(JsonValue::Object(object));
    }
    // キーバリューのペアの数だけ繰り返す
    loop {
        // 文字列のキーを控えておく
        let key = if let Some(Token::String(s)) = &self.current_token {
            s.clone()
        } else {
            return None;
        };
        self.next_token();
        // : (読み飛ばす)
        self.next_token();
        // value (値がオブジェクトや配列である場合のためにここで再帰する)
        if let Some(value) = self.parse() {
            object.insert(key, value);
        }
        // , なら次のキーバリューに続き } が来たらループ終了
        match &self.current_token {
            Some(Token::Comma) => {
                self.next_token();
            }
            Some(Token::RightBrace) => {
                self.next_token();
                break;
            }
            _ => return None,
        }
    }
    return Some(JsonValue::Object(object));
}
```

`parse_array` `parse_object` を、`parse` 関数から呼ぶように修正して構文解析も完成です。

```rust:parser.rs
/**
 * JSON値をパースする
 */
pub fn parse(&mut self) -> Option<JsonValue> {
    match &self.current_token {
        Some(Token::LeftBrace) => self.parse_object(),  // { がオブジェクトの開始
        Some(Token::LeftBracket) => self.parse_array(), // [ が配列の開始
        // 以下略
```

# 整形 (Formatter)

ここまでで、JSON 文字列を元に字句解析、構文解析を行って、JSON として解釈できるようになりました。

次に、JSON を整形しながら再び一つの文字列として出力するフォーマッターを実装します。

再掲になりますが、構文解析によって、以下の `JsonValue` の値を取り出せるようになりました。

```rust:json.rs
pub enum JsonValue {
    Object(JsonObject), // {"key": "value"}
    Array(JsonArray),   // [1, 2, 3]
    String(String),     // "hello, world"
    Number(f64),        // 123.456
    True,               // true
    False,              // false
    Null,               // null
}

pub type JsonObject = IndexMap<String, JsonValue>;
pub type JsonArray = Vec<JsonValue>;
```

ここからは JsonValue 型に、`format` 関数を実装します。

はじめに空文字列を用意し、Json::Value の内容に応じて、シンタックス、リテラル、改行、インデントを適宜追記していきます。

構文解析同様、オブジェクト及び配列では中に含まれている値についてもフォーマットを行う必要があるので、`format_value` 関数を再帰的に呼び出しつつ、インデント幅を拡げる必要があります。

```rust:json.rs
impl JsonValue {
    /**
     * JSON全体を整形した文字列を返す
     */
    pub fn format(&self, indent: usize) -> String {
        let mut formatted = String::new();
        self.format_value(indent, &mut formatted);
        return formatted;
    }

    /**
     * JSONに含まれる値を整形した文字列を返す
     * オブジェクトや配列の場合、再帰的に整形を繰り返す
     */
    fn format_value(&self, indent: usize, formatted: &mut String) {
        match self {
            JsonValue::Object(obj) => {
                self.push_str(formatted, "{\n");
                for (i, (key, value)) in obj.iter().enumerate() {
                    self.push_indent(formatted, indent + 2);
                    self.push_str(formatted, "\"");
                    self.push_str(formatted, key);
                    self.push_str(formatted, "\"");
                    self.push_str(formatted, ": ");
                    value.format_value(indent + 2, formatted);
                    if i < obj.len() - 1 {
                        self.push_str(formatted, ",\n");
                    } else {
                        self.push_str(formatted, "\n");
                    }
                }
                self.push_indent(formatted, indent);
                formatted.push_str("}")
            }
            JsonValue::Array(array) => {
                self.push_str(formatted, "[\n");
                for (i, value) in array.iter().enumerate() {
                    self.push_indent(formatted, indent + 2);
                    value.format_value(indent + 2, formatted);
                    if i < array.len() - 1 {
                        self.push_str(formatted, ",\n");
                    } else {
                        self.push_str(formatted, "\n");
                    }
                }
                self.push_indent(formatted, indent);
                formatted.push_str("]")
            }
            JsonValue::String(str) => {
                formatted.push('"');
                formatted.push_str(str);
                formatted.push('"');
            }
            JsonValue::Number(num) => {
                let value = &num.to_string();
                self.push_str(formatted, value);
            }
            JsonValue::True => {
                self.push_str(formatted, "true");
            }
            JsonValue::False => {
                self.push_str(formatted, "false");
            }
            JsonValue::Null => {
                self.push_str(formatted, "null");
            }
        }
    }

    // 出力テキストに文字列を追加
    fn push_str(&self, formatted: &mut String, str: &str) {
        formatted.push_str(str);
    }

    // 出力テキストにインデントを追加
    fn push_indent(&self, formatted: &mut String, indent: usize) {
        for _ in 0..indent {
            formatted.push_str(" ");
        }
    }
```

# 入出力 (main)

字句解析、構文解析、整形の仕組みが完成したので、最後にインタフェース部分を実装して完成です。

```rust:main.rs
mod json;
mod lexer;
mod parser;

use lexer::Lexer;
use parser::Parser;
use std::io::{self, Read};

fn main() {
    // 標準入力からJSON文字列を読み込む
    let mut input = String::new();
    io::stdin()
        .read_to_string(&mut input)
        .expect("テキストの読み込みに失敗しました");

    // 字句解析+構文解析
    let lexer = Lexer::new(&input);
    let mut parser = Parser::new(lexer);
    let json = parser.parse().expect("JSONのパースに失敗しました");

    // パース結果を標準出力
    println!("{}", json.format(0));
}
```

# 動作確認

```bash
$ echo "{\"key1\":10,\"key2\":[1,2,{\"key3\": [\"Hello\",true, false, null]}], \"key4\": [{}]}" | cargo run
{
  "key1": 10,
  "key2": [
    1,
    2,
    {
      "key3": [
        "Hello",
        true,
        false,
        null
      ]
    }
  ],
  "key4": [
    {
    }
  ]
}
```

# おわりに

本記事では、フロントエンドエンジニアが興味本位で Rust に手を出して、お試しで JSON フォーマッターを実装する練習の事例を紹介しました。

記事内ではソースコードの一部しか掲載できませんでしたが、テストコードなどを含むすべてのコードは以下のリポジトリで公開しています。(そんな細かく読む人いないと思うし、普段から Rust を書いてる人が見たらツッコミどころ満載だと思いますが)
https://github.com/s-sasaki-0529/rust-json-formatter

せっかく久しぶりに新しいプログラミング言語を学んだので、当面は個人用の CLI ツールなどもできるだけ Rust でトライしてみようかなと思える良い取り組みになりました。
