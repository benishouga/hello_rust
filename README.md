# Rust を初めて触ってみようの会

## 開発環境周りの話

### 環境構築

Linux とか Mac なら環境なら以下のコマンドで "rustup" というツールをインストール

```sh
$ curl https://sh.rustup.rs -sSf | sh
```

rustup は Rust 自体のインストールを管理するツールで、Node.js でいうところの nvm とかと同じようなものの様子。

rustup がインストールされると同時に stable の rustc, cargo などがインストールされる。

- rustc: Rust のコンパイラ
- cargo: Rust のビルドシステム兼、パッケージマネージャ。Node.js でいうところの npm みたい。

インストールされたディレクトリがインストール後に表示されるので、 `.bashrc` などでパスを通してあげる必要あり。

例: インストールされたところが `~/.cargo/bin/rustup` なら以下の感じでパス通す

```
export PATH="$PATH:~/.cargo/bin"
```

### プロジェクト作る

```
$ cargo new hello_cargo --bin
```

ディレクトリが作成されて、 `Cargo.toml` というプロジェクト管理用のファイルと `src/main.rs` というプログラムのエントリポイントとなるファイルが生成される

`src/main.rs` は `Hello, world!` を表示するプログラムとなっている。

`cargo run` と叩くとプログラムのビルドと実行が行われる

```sh
$ cargo run
   Compiling hello_cargo v0.1.0 (/home/xxxxx/work/hello_rust/hello_cargo)
    Finished dev [unoptimized + debuginfo] target(s) in 0.23s
     Running `target/debug/hello_cargo`
Hello, world!
```

プログラムを変更しなければ、ビルドはスキップされて即実行される

### VSCode でコーディングする

公式の拡張（ `rust-lang.rust` ）が用意されている。

ただ、これ環境によっては拡張のインストール後のセットアップに失敗し上手く機能しない様子。

VSCode の拡張がデフォルトで参照するパスがちょっとずれているぽい。

以下のような設定を User 設定に以下を追加してあげると参照できるようになった。

```json
{
  "rust-client.rustupPath": "~/.cargo/bin/rustup"
}
```

また、この拡張を入れると、以下のようにテストコードを記述すると `#[test]` の下辺りに `Run test` という Link ボタンが表示されて任意のテストの実行をサポートしてくれる。

```rs
#[cfg(test)]
mod tests {
    #[test]
    fn it_works() {
        assert_eq!(2 + 2, 4);
    }
}
```

けども、これも環境によっては cargo が見つからないとエラーになってしまう。

以下のような設定値を Workspace の設定に追加してあげると実行できるよになった。

```json
{
  "terminal.integrated.inheritEnv": true
}
```

### VSCode でデバッグする

デバッガを使うため CodeLLDB という拡張をインストール。

`.vscode/launch.json` を作成し以下の設定を追加。

```json
{
  "version": "0.2.0",
  "configurations": [
    {
      "type": "lldb",
      "request": "launch",
      "name": "Launch",
      "cargo": { "args": ["run"] },
      "program": "${cargo:program}",
      "args": []
    },
    {
      "type": "lldb",
      "request": "launch",
      "name": "Test",
      "cargo": {
        "args": ["test", "--no-run"]
      },
      "program": "${cargo:program}",
      "args": []
    }
  ]
}
```

上が、エントリポイントからのデバッグ実行する設定。下が、テストをデバッグ実行する設定。

これで、任意のブレークポイントを VSCode から設定し、デバッグができるようになった感。

公式の拡張でテストコードに追加される `Run test` リンクからデバッグする方法は分からず。。。

あと、なぜか VSCode を ターミナルから `$ code .` などで起動すると上手くデバッガが動かないことがある様子？自分の環境だけ？

また、Windows では CodeLLDB が `PYTHONHOME` という環境変数を参照するらしく、ここに python の `python3.dll` があるディレクトリ指定しておく必要があるみたい。

## 言語仕様や文法の話

### エントリポイント

```rs
fn main() { }
```

### 変数宣言

```rs
let x = 5;
println!("The value of x is: {}", x);
```

`let` で宣言すると不変で JavaScript の `const` 的な感じ？

可変にするのであれば `mut` をつける

```rs
let mut x = 5;
println!("The value of x is: {}", x);
x = 6;
println!("The value of x is: {}", x);
```

以下の `const` は定数を宣言するものとしてある。 `const` には `mut` は付けられない。

```rs
const MAX_POINTS: u32 = 100_000;
```

> 定数は、プログラムが走る期間、定義されたスコープ内でずっと有効

とのこと。プログラムの実行中にいろいろなところから使用される値を保持しておけば良い様子

`let` は同じ変数名に対し何度も使用可能。

```rs
let x = 5;
let x = x + 1;
let x = x * 2;
println!("The value of x is: {}", x);
```

> シャドーイングは、変数を mut にするのとは違います。なぜなら、let キーワードを使わずに、 誤ってこの変数に再代入を試みようものなら、コンパイルエラーが出るからです。let を使うことで、 値にちょっとした加工は行えますが、その加工が終わったら、変数は不変になるわけです。

とのこと。

また、あくまで変数宣言で、変数名を使いまわすため、型はその際に新しいものとなる。

```rs
let spaces = "   ";
let spaces = spaces.len();
```

広いスコープで使うと混乱しそうだが、小さいスコープの中で使えば便利な場面多そう。

可変の変数への再代入、型そのままのため、違う型を代入しようとするとちゃんとエラーになる。

```rs
let mut spaces = "   ";
spaces = spaces.len();
```

```sh
$ cargo build
   Compiling sandbox v0.1.0 (/home/xxxxx/work/hello_rust/sandbox)
error[E0308]: mismatched types
 --> src/main.rs:3:14
  |
3 |     spaces = spaces.len();
  |              ^^^^^^^^^^^^ expected `&str`, found `usize`

error: aborting due to previous error
```

### データ型

Rust ではすべての値は、何らかのデータ型となる。

データ型のサブセットとしてスカラー型と複合型がある。

通常 Rust は型を推論してくれますが、複数の型が推論される場合は明確に型を指定する必要がある。

以下は u32 を明示的に指定しているので問題ないが、

```rs
let guess: u32 = "42".parse().expect("parse失敗");
```

以下は型推論を使おうとしているが、parse は複数の数値型を推論するため、エラーになる

```rs
let guess = "42".parse().expect("parse失敗");
// $ cargo run
// error[E0282]: type annotations needed
//  --> src/main.rs:2:9
//   |
// 2 |     let guess = "42".parse().expect("parse失敗");
//   |         ^^^^^ consider giving `guess` a type
```

ためしに小数点ありの文字列を解析したら実行時に ParseIntError となった。

```rs
let guess: u32 = "4.2".parse().expect("parse失敗");
// thread 'main' panicked at 'parse失敗: ParseIntError { kind: InvalidDigit }', src/main.rs:2:22
```

#### スカラー型

Rust では 整数, 浮動小数点数, 論理値, 文字 がスカラー型として扱われる。

整数型は符号付き `i8` `i16` `i32` `i64` `isize` と、符号なし `u8` `u16` `u32` `u64` `usize` の型があり、 `isize` と `usize` はプログラムが動作しているコンピュータの種類に依存し、64 ビットアーキテクチャなら 64 ビット、32 ビットアーキテクチャなら 32 ビットとなり、それぞれ `is`, `us` と省略可能とのこと。整数型の基準型は `i32` 。

数値を表すリテラル は以下のようなものがある。

10 進数 `98_222`, 16 進数 `0xff`, 8 進数 `0o77`, 2 進数 `0b1111_0000`, バイト (u8 だけ) `b'A'`

浮動小数点型は `f32`, `f64` があり、基準型は `f64` 。

論理型は `bool` の１種類で、 値としても `true` `false` の２種類。

文字型は `char` で、`'A'` のようにシングルクォートで宣言して使用する。 `char` 型はユニコードのスカラー値を表していて `'😻'` などの絵文字も入る。ただ注意点もある様子で 8 章「文字列」で説明されるとのこと。

#### 複合型

複合型としては タプル型 と 配列型 がある。

##### タプル型

タプル型とは複数の型をまとめたもので、以下のように宣言、利用できる。

```rs
let tup: (i32, f64, u8) = (500, 6.4, 1);
println!("{}, {}, {}", tup.0, tup.1, tup.2);
```

分割代入も利用できる。

```rs
let (x, y, z) = tup;
println!("{}, {}, {}", x, y, z);
```

##### 配列型

配列型は他の言語と同じ用に値のコレクションを扱うもの。タプル型と違い同じ型を扱う必要がある。

以下のように宣言して利用できる。

```rs
let a = [1, 2, 3, 4, 5];
println!("{}, {}, {}", a[0], a[1], a[2]);
```

こちらも分割代入が使える。

```rs
let [b, _, _, _, c] = a;
println!("{}, {}", b, c);
```

配列はスタックにデータを確保したいときや固定長のデータを扱いたい場合に使われるとのこと。

可変長のデータを扱いたい場合は ベクタ型 を利用するのが良いとのこと。

配列のサイズを超えてのアクセスなエラーは実行時エラーとなる。

```rs
let a = [1, 2, 3, 4, 5];
let element = a[10];
println!("{}", element);
```

```
$ cargo run
    Finished dev [unoptimized + debuginfo] target(s) in 0.01s
     Running `target/debug/sandbox`
thread 'main' panicked at 'index out of bounds: the len is 5 but the index is 10', src/main.rs:4:19
note: run with `RUST_BACKTRACE=1` environment variable to display a backtrace
```

### 関数

fn キーワードで関数は宣言する。

```rs
fn another_function() {
  println!("Another function.");
}
```

引数は () の中に並べる。

```rs
fn another_function(x: i32, y: i32) {
  println!("{}", x);
  println!("{}", y);
}
```

関数本体はブロックで構成される。ブロックは 文 が並び、最後が 文 or 式 となり、最後が式の場合は、その評価値をそのまま利用できる。

```rs
fn main() {
  let y = {
    let x = 3;
    x + 1
  };
  println!("{}", y);
}
```

このブロック使って 三項演算子的なことをすると以下の感じで書くみたい。

```rs
let c = if a == b { 10 } else { 20 };
```

関数では戻り値の型は `->` に続いて宣言する。最後の式はそのまま評価値として利用されるので return を書かずに以下のよう書ける。

```rs
fn hoge() -> i32 {
  5
}
```

return も使えて、早期リターンしたい場合は以下のような感じ。

```rs
fn hoge(a: bool) -> i32 {
  if a {
    return 5;
  }
  10
}
```

最後の式の場合は戻り値として使われるが、文（セミコロンで終わる）の場合、値を返さないものとして扱われる。

```rs
fn hoge(a: bool) -> i32 {
  10;
}
```

以下のようなエラーとなる。

```
$ cargo run
   Compiling sandbox v0.1.0 (/home/xxxxx/work/hello_rust/sandbox)
error[E0308]: mismatched types
 --> src/main.rs:6:14
  |
6 | fn hoge() -> i32 {
  |    ----      ^^^ expected `i32`, found `()`
  |    |
  |    implicitly returns `()` as its body has no tail or `return` expression
7 |     10;
  |       - help: consider removing this semicolon
```

セミコロン消したら良いかも？と言われてますね。

### コメント

ひとまず `//` を使えば良いっぽい。

### フロー制御

`if`による分岐とループについて。

#### if

`if` 文は以下の感じ

```rs
if number == 5 {
  println!("5!");
} else {
  println!("not 5!");
}
```

`()` が要らないのは少し慣れが必要そう。

`if` で評価される値は、必ず `bool` である必要がある。

`else if` は以下の感じ

```rs
if number == 5 {
  println!("5!");
} else if number == 6 {
  println!("6!");
} else {
  println!("other!");
}
```

先に出てきたとおり、ブロックは最後に評価された式を値として返却するので以下のように書ける

```rs
let a = if number == 5 {
  "5!"
} else if number == 6 {
  "6!"
} else {
  "other!"
};
```

`if else` で連結されたブロックが違う型を返却しようとすると、コンパイルエラーとなる。

```rs
let a = if number == 5 { 5 } else { "other" };
```

```
$ cargo run
   Compiling sandbox v0.1.0 (/home/xxxxx/work/hello_rust/sandbox)
error[E0308]: `if` and `else` have incompatible types
 --> src/main.rs:3:41
  |
3 |     let a = if number == 5 { 5 } else { "other" };
  |                              -          ^^^^^^^ expected integer, found `&str`
  |                              |
  |                              expected because of this
```

というか、コンパイルエラーの説明がものすごく親切。。。

#### ループ

ループの制御は `loop` `while` `for` の 3 種がある。

`loop` は `break` による明示的な命令や例外などが発生しない限り抜け出さない永久ループ。

```rs
let mut c = 0;
loop {
  if c < 3 {
    println!("{}", c);
    c = c + 1;
  } else {
    break;
  }
}
println!("finish");
```

`while` は条件式を 1 つ取るいつもの `while` 。

```rs
let mut c = 0;
while c < 3 {
  println!("{}", c);
  c = c + 1;
}
println!("finish");
```

`for` はいつものやつと違って `in` の後ろにコレクションを指定して使うものとなっている。

```rs
for c in 0..3 {
  println!("{}", c);
}
println!("finish");
```

`0..3` は Range 型というもので、左の値から始まり右の値未満の値を生成する型らしい。

### 所有権

メモリ管理について、多くの言語では、明示的に確保／解放を行う、またはガベージコレクションに任せるという選択肢をとっているが、 Rust では所有権というシステムを使って管理する方法をとっている。

所有権の規則は以下の通りとのこと。

- Rust の各値は、所有者と呼ばれる変数と対応している。
- いかなる時も所有者は一つである。
- 所有者がスコープから外れたら、値は破棄される。

#### 変数スコープ

他の多くの言語と同じく、ブロック内で宣言された変数はその変数の中にある間だけ有効。

```rs
{
  // s の宣言前だから有効ではない

  let s = "hello";

  // s を使える
}
// s はもう有効ではない
```

#### String 型を使った所有権の説明

所有権を説明するために、String 型を使ってみる

文字列リテラルをそのまま変数に入れた場合、文字列はハードコードされ不変なものとして扱われる。

つまり以下のような文字列を変更しようとするとエラーとなる。

```rs
let mut s = "hello";
s.push_str(", world");
```

```
$ cargo run
   Compiling sandbox v0.1.0 (/home/xxxxx/work/hello_rust/sandbox)
error[E0599]: no method named `push_str` found for reference `&str` in the current scope
 --> src/main.rs:3:7
  |
3 |     s.push_str(", world");
  |       ^^^^^^^^ method not found in `&str`

error: aborting due to previous error
```

文字列リテラルをそのまま使用する代わりに `from` 関数を使うと、ヒープ領域に文字列を確保することになり、これであれば上記のエラーは発生せず、正しく動作する。

```rs
let mut s = String::from("hello");
s.push_str(", world");
```

#### ヒープへの確保／解放

Rust では、ブロックでヒープにメモリを確保した際、そのブロックを抜けるタイミングで自動で解放されるとのこと。

```rs
{
  let mut s = String::from("hello");
  s.push_str(", world");
}
// ここでは もう s は解放される。
```

これらのヒープ領域に解放には、その型で定義されている `drop` 関数が使われることになり、ここにメモリを解放する処理を記述することができる。

##### 所有権のムーブ

下記は x, y も正しく、そして自然に使用できる。

```rs
let x = "hello";
let y = x;
println!("x = {}, y = {}", x, y);
```

スタックに積まれるスカラー値も同じように使用できる。

```rs
let x = 5;
let y = x;
println!("x = {}, y = {}", x, y);
```

しかし、以下のようにヒープで確保されたメモリを扱う場合、所有権のムーブというものが行われ、元の変数は使用できなくなる。

```rs
let x = String::from("hello");
let y = x;
println!("x = {}, y = {}", x, y);
```

```
$ cargo run
   Compiling sandbox v0.1.0 (/home/xxxxx/work/hello_rust/sandbox)
error[E0382]: borrow of moved value: `x`
 --> src/main.rs:4:24
  |
2 |     let x = String::from("hello");
  |         - move occurs because `x` has type `std::string::String`, which does not implement the `Copy` trait
3 |     let y = x;
  |             - value moved here
4 |     println!("{}, {}", x, y);
  |                        ^ value borrowed here after move
```

この挙動は、ブロックを抜けたタイミングで変数を解放が 2 重に行われないようにするために、一つを無効にするため。

もし、元の変数とともにどちらも使用可能にしたいなら、以下の通り `clone` を使う必要がある。

```rs
let x = String::from("hello");
let y = x.clone();
println!("x = {}, y = {}", x, y);
```

`clone` によりディープコピーが行われ、両方の変数が有効という状態。

#### 所有権と関数

前述の所有権のムーブは関数の呼び出しでも行われます。つまり以下のような呼び出しでは関数を呼び出した際に所有権がムーブされ、元の関数では使用できなる。

```rs
fn main() {
  let x = String::from("hello");
  hoge(x);
  println!("{}", x);
}

fn hoge(y: String) {
  println!("{}", y);
}
```

```
$ cargo run
   Compiling sandbox v0.1.0 (/home/xxxxx/work/hello_rust/sandbox)
error[E0382]: borrow of moved value: `x`
 --> src/main.rs:4:20
  |
2 |     let x = String::from("hello");
  |         - move occurs because `x` has type `std::string::String`, which does not implement the `Copy` trait
3 |     hoge(x);
  |          - value moved here
4 |     println!("{}", x);
  |                    ^ value borrowed here after move
```

また、関数から戻り値が返された場合もその所有権はムーブされる。つまり以下のコードであれば正しく動く。

```rs
fn main() {
  let x = String::from("hello");
  let x = hoge(x);
  println!("{}", x);
}

fn hoge(y: String) -> String {
  println!("{}", y);
  y
}
```

関数内で参照したいだけなどの場合、毎回所有権のムーブが起こっていたのでは面倒な場面もあります。

#### 参照

そんなときのために参照を渡すという選択ができる。

```rs
fn main() {
  let x = String::from("hello");
  hoge(&x);
  println!("{}", x);
}

fn hoge(y: &String) {
  println!("{}", y);
}
```

ただし、参照を借用しているだけで、所有権はないため、その更新を行おうとするとできず、コンパイルエラーとなる。

```rs
fn main() {
  let x = String::from("hello");
  hoge(&x);
  println!("{}", x);
}

fn hoge(y: &String) {
  y.push_str(", world");
}
```

```
$ cargo run
   Compiling sandbox v0.1.0 (/home/xxxxx/work/hello_rust/sandbox)
error[E0596]: cannot borrow `*y` as mutable, as it is behind a `&` reference
 --> src/main.rs:8:5
  |
7 | fn hoge(y: &String) {
  |            ------- help: consider changing this to be a mutable reference: `&mut std::string::String`
8 |     y.push_str(", world");
  |     ^ `y` is a `&` reference, so the data it refers to cannot be borrowed as mutable
```

参照経由で更新も行うためには、可変な参照を使う必要がある。

```rs
fn main() {
  let mut x = String::from("hello");
  hoge(&mut x);
  println!("{}", x);
}

fn hoge(y: &mut String) {
  y.push_str(", world");
}
```

可変な参照には 1 つしか持てないという制約があります。

```rs
fn main() {
  let mut s = String::from("hello");

  let r1 = &mut s;
  let r2 = &mut s;

  println!("{}, {}", r1, r2);
}
```

```
$ cargo run
   Compiling sandbox v0.1.0 (/home/xxxxx/work/hello_rust/sandbox)
error[E0499]: cannot borrow `s` as mutable more than once at a time
 --> src/main.rs:5:14
  |
4 |     let r1 = &mut s;
  |              ------ first mutable borrow occurs here
5 |     let r2 = &mut s;
  |              ^^^^^^ second mutable borrow occurs here
6 |
7 |     println!("{}, {}", r1, r2);
  |                        -- first borrow later used here
```

この制約のおかげで、データの書き込みの競合などを、コンパイル時に検査できるようになるとのこと。

読み込みは何人でも可、書き込める人は常に 1 人という プライマリ／レプリカ な関係を言語仕様としてサポートしているということなのかな。

参照は関数の中に渡すことはできるが、外に渡すことはできない。

```rs
fn main() {
  let x = hoge();
}

fn hoge() -> &String {
  let s = String::from("hello");
  &s
}
```

```
$ cargo run
   Compiling sandbox v0.1.0 (/home/xxxxx/work/hello_rust/sandbox)
error[E0106]: missing lifetime specifier
 --> src/main.rs:5:14
  |
5 | fn hoge() -> &String {
  |              ^ help: consider giving it a 'static lifetime: `&'static`
  |
  = help: this function's return type contains a borrowed value, but there is no value for it to be borrowed from
```

これができてしまうと、関数内で確保されたメモリがそのブロックから抜けるタイミングで解放されてしまうにもかかわらず、参照してしまうという非常に危険な状態に陥ってしまうため。

関数の外に返すのであれば前述の所有権をムーブする必要がある。

#### スライス

所有権のデータの一部の参照を作りたくなることがあった場合は、スライスという方法がある。

```rs
let s = String::from("hello");
let slice = &s[1..4];
println!("{}", slice); // -> ell
```

スライスとして切り出せるのは参照となる。また、文字列リテラルもこのスライスらしい。

関数の中に所有権があるものは、そのライフサイクルが切れたことになるため、そのスライスだけを外に返すことはできず、エラーとなる。

```rs
fn main() {
  let slice = hoge();
  println!("{}", slice);
}

fn hoge() -> &str {
  let s = String::from("hello");
  &s[1..4]
}
```

```
$ cargo run
   Compiling sandbox v0.1.0 (/home/xxxxx/work/hello_rust/sandbox)
error[E0106]: missing lifetime specifier
 --> src/main.rs:6:14
  |
6 | fn hoge() -> &str {
  |              ^ help: consider giving it a 'static lifetime: `&'static`
  |
  = help: this function's return type contains a borrowed value, but there is no value for it to be borrowed from
```

外に所有権があるもののスライスを作るのは問題なくできる。

```rs
fn main() {
  let s = String::from("hello");
  let slice = hoge(&s);
  println!("{}", slice);
}

fn hoge(s: &str) -> &str {
  &s[1..4]
}
```

### 構造体

以下のように定義する

```rs
struct User {
  username: String,
  email: String,
  sign_in_count: u64,
  active: bool,
}
```

使いたいときはこう。 `.` でメンバーにアクセスできる。

```rs
let user1 = User {
  email: String::from("someone@example.com"),
  username: String::from("someusername123"),
  active: true,
  sign_in_count: 1,
};
println!("{}", user1.email);
```

値を可変にしたいのであれば `mut` をつける

```rs
let mut user1 = User {
  email: String::from("someone@example.com"),
  username: String::from("someusername123"),
  active: true,
  sign_in_count: 1,
};
user1.email = String::from("anotheremail@example.com");
println!("{}", user1.email);
```

構造体を作るような関数

```rs
fn main() {
  let user = build_user(String::from("aaa"), String::from("bbb"));
  println!("{}", user.email);
}

fn build_user(email: String, username: String) -> User {
  User {
    email: email,
    username: username,
    active: true,
    sign_in_count: 1,
  }
}
```

ショートハンドで書ける。

```rs
fn build_user(email: String, username: String) -> User {
  User {
    email,
    username,
    active: true,
    sign_in_count: 1,
  }
}
```

別のインスタンスからプロパティを流用することもできる。

```rs
fn main() {
  let user1 = User {
    email: String::from("aaaaa@example.com"),
    username: String::from("aaaaausername567"),
    active: true,
    sign_in_count: 1,
  };

  let user2 = User {
    email: String::from("another@example.com"),
    username: String::from("anotherusername567"),
    ..user1
  };
}
```

なお、この場合も所有権がムーブは行われる。つまり以下のように書くとエラーとなる。

```rs
fn main() {
  let user1 = User {
    email: String::from("aaaaa@example.com"),
    username: String::from("aaaaausername567"),
    active: true,
    sign_in_count: 1,
  };

  let user2 = User {
    email: String::from("another@example.com"),
    ..user1
  };

  println!("{}, {}", user1.email, user1.username);
  println!("{}, {}", user2.email, user2.username);
}
```

```
$ cargo run
   Compiling sandbox v0.1.0 (/home/xxxxx/work/hello_rust/sandbox)
error[E0382]: borrow of moved value: `user1.username`
  --> src/main.rs:21:37
   |
16 |       let user2 = User {
   |  _________________-
17 | |         email: String::from("another@example.com"),
18 | |         ..user1
19 | |     };
   | |_____- value moved here
20 |
21 |       println!("{}, {}", user1.email, user1.username);
   |                                       ^^^^^^^^^^^^^^ value borrowed here after move
   |
   = note: move occurs because `user1.username` has type `std::string::String`, which does not implement the `Copy` trait
```

タプルからも構造体を作れる

```rs
struct Color(i32, i32, i32);
struct Point(i32, i32, i32);

let black = Color(0, 0, 0);
let origin = Point(0, 0, 0);
```

構造体にトレイトを継承する注釈を追加することで、そのままログ用に出力してくれるようになったりする。

```rs
#[derive(Debug)]
struct Rectangle {
  width: u32,
  height: u32,
}

fn main() {
  let rect1 = Rectangle { width: 30, height: 50 };
  println!("rect1 is {:?}", rect1);
}
```

```
$ cargo run
   Compiling sandbox v0.1.0 (/home/xxxxx/work/hello_rust/sandbox)
    Finished dev [unoptimized + debuginfo] target(s) in 0.24s
     Running `target/debug/sandbox`
rect1 is Rectangle { width: 30, height: 50 }
```

`{:?}` の代わりに `{:#?}` を使うと改行も使って見やすく表示してくれる

```
$ cargo run
   Compiling sandbox v0.1.0 (/home/xxxxx/work/hello_rust/sandbox)
    Finished dev [unoptimized + debuginfo] target(s) in 0.24s
     Running `target/debug/sandbox`
rect1 is Rectangle {
    width: 30,
    height: 50,
}
```

構造体にメソッドを紐付けて定義する

```rs
#[derive(Debug)]
struct Rectangle {
  width: u32,
  height: u32,
}

impl Rectangle {
  fn area(&self) -> u32 {
    self.width * self.height
  }
}

fn main() {
  let rect1 = Rectangle { width: 30, height: 50 };
  println!("{}", rect1.area());
}
```

`impl` キーワードを使って別途定義する。これはあと付けでできる様子で C# の拡張メソッドとかに近い印象。`impl` ブロックは複数定義できる。

引数を入れる場合は `&self` 以降に続けて宣言する。

```rs
#[derive(Debug)]
struct Rectangle {
  width: u32,
  height: u32,
}

impl Rectangle {
  fn can_hold(&self, other: &Rectangle) -> bool {
    self.width > other.width && self.height > other.height
  }
}

fn main() {
  let rect1 = Rectangle {
    width: 30,
    height: 50,
  };
  let rect2 = Rectangle {
    width: 10,
    height: 40,
  };
  let rect3 = Rectangle {
    width: 60,
    height: 45,
  };
  println!("Can rect1 hold rect2? {}", rect1.can_hold(&rect2));
  println!("Can rect1 hold rect3? {}", rect1.can_hold(&rect3));
}
```

関連関数という、Java などでいう クラスメソッド のようなものは以下のように定義する。呼び出しは `::` を使う。

```rs
#[derive(Debug)]
struct Rectangle {
  width: u32,
  height: u32,
}

impl Rectangle {
  fn square(size: u32) -> Rectangle {
    Rectangle { width: size, height: size }
  }
}

fn main() {
  println!("{:?}", Rectangle::square(10));
}
```

一旦ここで終わり。
