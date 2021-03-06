# Rust 初窥 : 设计理念与基本使用

Rust 并非一蹴而就，而是这些年来工业界与学界关于编程语言理论的集大成者。

工作上写 C++，从去年底开始关注 Rust，至今用 Rust 写了一些和交易相关的小程序。总体感觉是如果有一门语言能够取代 C++，那么它只可能是 Rust。

为什么这么说呢？首先我们来说一下为什么很多情况下人们会选择 C++。

很多人用 C++ 的不是因为 C++ 有多好，只是因为如果想要写一个接近实时高性能，稳健，并有足够开发效率的大程序，通常只有 C++ 可选。这有如下几个原因：

要写接近实时，就不要能有垃圾回收 (GC)。GC 对于一些类型的程序几乎是致命的。想像一个低延时交易程序在看到一个交易机会之后因为某种原因触发长达 100ms 的 GC，这个机会铁定就没了。要想达到接近实时且高性能就需要接近底层，即所谓 bare-metal。高性能可以用 C 达到，但考虑足够的开发效率，C 的语言特性缺乏就是一个明显问题。我们也不想在组一个小组开始开发的时候还要来写一遍数据结构基础设施。C++ 的多语言范式提供了这样一些基础的组件，多数情况下你不需要再去写一个 hashmap 或者查找算法。从开发效率和可读可维护性上来说，足够的抽象能力是必须的，但这种抽象必须是没有运行时开销的 (runtime cost)。零开销抽象 (zero cost abstraction) 是 C++ 的设计原则之一。inline 函数，constexpr 程序，template ，都是遵循这一原则，编译器如果发现虚类 (virtual class) 没有真正被用到甚至会优化掉虚表 (virtual table)。稳健的大程序则意味着程序要尽量把错误消灭在进生产环境之前，要达到这个目标，我可以牺牲一点开发效率。这就意味着我需要一个静态类型最好同时是强类型的语言，使得编译器能够早发现问题。<Effective C++> 会强调一些诸如单参数构造函数最好加 explicit 之类的 " 技巧 " 来加强类型检查。C++11 在这个基础上加了如 enum class。各种 override，final 等都是加强编译器在编译时查出错误的能力。总体上来说，就是一个高性能的静态强类型多范式语言。

但同时，C++ 问题很多，这些问题不是语言设计者能力不够，而多数是历史原因与当时抉择所看重的东西导致的。C++ 一开始的时候一个目标是与 C 兼容，即使 C 当年算是极其天才的设计 ( 和 Unix 一样 )，它毕竟是 1972 年的语言，当时设计导致的问题虽然我们现在已经很清楚，C++ 也因为兼容性而把它保留了下来。大家都知道 Python 2 和 Python 3 的故事，甚至 IA64 和 x86_64 的故事，所以也很容易理解 C++ 为什么要一直保持向前兼容。

Rust 由于是一个新语言，所以它完全没有历史包袱。甚至在今年之前的五年中，社区一直完全肆无忌惮的做和之前完全不兼容的改动。如果发现有个新设计确实公认更好，语言作者们不会为了向前兼容而放弃改变。近 30 年的语言理论研究和实际软件工程的经验有足够多的优秀设计可以直接拿来使用。Rust 作为一个新语言显然不会放过这个机会。在可以完全重来的前提下，Rust 相对 C++ 有哪些新特性呢？

无 data-race 的并发。使用类似 golang 的 channel-based 并发非常易用。在不用 lockfree 的情况下，开发和运行效率都足够高，逻辑简单，加上下面的杀手特性避免 data-race。函数式语言的 pattern matching。网站首页**就提供了一个简单的例子，用惯指令式和面向对象的朋友可能需要一段时间发挥它的强大之处。实际上 Rust 还提供非常多的函数式语言特性，包括强大的 closure，由于下面要提到的杀手级特性的保证，Rust 的 closure 十分安全。Generics 和 Trait 粗看起来是 zero cost abstraction 的编译时多态 (compile-time polymorphism)，类似于 C++ 中的 template 和 C++17 里的 Concept。但实际上它设计的精巧**已经远不是 C++ 中 template 的同类了。其中一点就是统一了 compile-time 和 run-time polymorphism，编译时多态叫 trait，运行时多态叫 trait object，省去了不少程序语义方面的复杂性。Trait 这个特性这也是很多把 golang 看成是静态类型系统语言而又发现它竟然没有编译时多态的同学震惊而无奈的离它而去的原因。(golang 作为服务器语言仍然是相当不错的选择，不过现在已经没有多少人还把它看成是系统语言了。) 灵活的 enum 系统，以及衍生的错误处理机制。Rust 没有 exception，错误是通过 enum 返回的。与 golang 在正确时也会返回错误代码不同，Rust 错误代码只有在错误时才返回。这不仅仅只是方便，而有更重要的好处**。简单易用的与 C 语言的互通性。因为没有 runtime，与 C 语言的互通简单是很自然的。这篇文章**讲得很详细。灵活的 module 系统。module 系统以及 pub 和 use 关键字让你灵活的控制所有的访问许可关系。强大的管理系统 Cargo 和中心化的库管理 crates.io**。Cargo 的依赖管理遵循最新的 Semantic Versioning**。只需要适当选择库的依赖版本，一个 cargo update 会自动完成所有版本匹配和下载加载。被 C++ 依赖摧残的小朋友们可以开心了。其它的诸如，缺省可使用的直接打印数据的内容状态，更好用的宏，更漂亮的条件编译，语言自带的简单测试系统，rustdoc 的自动文档生成，都让开发的效率和快感大增。一些小的改动，如缺省是不可变而不是 mut 的声明，match 的 exhaustive checking，都使人更容易写出更正确的代码。

当然，如果只有这些小改进，C++ 程序员可能看了看觉得，“ 不错哟 ”，然后就接者回去写 C++ 了（比如 D）。新语言必须有杀手级的特性，这才是转语言的关键动力。

这一杀手级的特性就是 Ownership 和 Lifetime。Rust 首页宣称的 "prevents nearly all segfaults, and guarantees thread safety" 是超级诱人的，因为这是在没有 GC 和 runtime 的情况下实现的。

C 语言的野指针，和线程安全问题会导致很多极难发现，诊断和修复的 bug。C++ ，尤其是 C++11 费心思去缓和了这一问题，但这仍然是没有 GC 讲求性能语言的核心劣势。Rust 这个特性具体是如何的呢？

Rust 里每一个引用和指针都有一个 lifetime，对象则只不允许在同一时间有两个和两个以上的可变引用。出了 lifetime 编译器在编译时就会静止它被使用。这样，如果一个操作使另一个引用不再可用，编译器就会发现它。C++ 里面的使用 to_string().c_str() 导致 crash 的 bug 应该不止一个人见过，这种 bug 在 rust 里是不会被编译通过的。这种特性使得原来在 C++ 里不敢使用的一些优化做法变得完全可能。比如把一个 c_string 分割成不同分，并在不同的地方使用。如果项目较大，为了完全起见，通常我们会把分割的结果拷贝一份单独处理，这样不需要害怕处理的时候那个 c_string 已经不存在。而使用 Rust 我们可以不用拷贝，而直接使用原来的 c_string 而不用担心野指针，lifetime 设定可以让 compiler 去做这一繁琐的检查，如果有任何的 c_string 在处理分割结果之前被使用，编译器会告诉你。这一特性所导致的编程可以衍生很多新的优化可能，而这都是在保证完全的前提下。实际上，催生 Rust 的浏览器 Servo 项目一个目标就是安全问题，Rust 在安全性让 heartbleed 问题出现的可能大大减小。

最后，Rust 是一个脚踏实地 (Practical) 的语言。这意味着它不遵循某一个范式 (paradigm) 或者是为计算机科学教学而生的语言。这是一个写给开发者的语言，如果特性经过权衡发现对开发有益，语言作者不会因为它破坏了某种范式而不去加它。这种哲学有些类似于当年 Linux 相对 Mimix 选择 monolithic kernel 而不是结构上更干净的 micro kernel，因为 Linux 需要性能。这样的选择会让语言更接近使用者，让使用者更开心。

所以用 C++ 的那些人的那些要求，Rust 都能达到或者甚至改善。

无 GC 实时控制，接近底层没有 overhead。在达到同样安全性的情况下，Rust 不会比 C++ 慢。足够多的语言特性保证开发效率，比 C++ 吸收了更多的现代优秀语言特性。与 C++ 一致的 Zero cost abstraction 杀手级的 ownership 和 lifetime，加上现代语言的类型系统，这是比 C++ 强最多的地方。在语言层面上 Rust 无疑比 C++ 优秀得多的一个高性能静态强类型多范式语言。

如 Rust 1.0 Announcement\** 所说，1.0 标志着 Rust 已经稳定。你能在 1.0 编译器上使用的特性都会保证继续存在，除非重大正确性 (soundness) 问题出现。现在已经没有 “ 下一个版本出来，之前的代码就不能用 ” 的顾虑了。

1.0 的发布是 Rust 发展的一个动力。当然真正长时间的采纳率还要看语言本身质量和社区环境。对这两点我是相当看好的。要想取代 C++ 绝不是一件容易和快速的事，这不紧只是语言的问题，而有很多不可控的其他因素，让我们拭目以待吧。

无论是使用 Rust 开发 Node.js 插件，还是 [Rust 默认支持 WebAssembly](https://parg.co/UPo)

[Rust 官方的教程](https://parg.co/UPm)

可以使用如下脚本，或者在 Windows 上下载 [rustup-init.exe](https://static.rust-lang.org/rustup/dist/i686-pc-windows-gnu/rustup-init.exe)；离线安装的话则可以下载[离线安装包](https://www.rust-lang.org/en-US/other-installers.html)。

```s
$ curl https://sh.rustup.rs -sSf | sh -s -- --help
```

在 Rust 开发环境中，所有工具都安装到 %USERPROFILE%\.cargo\bin 目录， 并且您能够在这里找到 Rust 工具链，包括 rustc、cargo 及 rustup。

```rs
fn main() {
    println!("Hello, world!");
}
```

```s
$ rustc main.rs
$ ./main
Hello, world!
```

```rs
let v = vec![1, 2, 3, 4, 5]; // v: Vec<i32>

let v = vec![0; 10]; // A vector of ten zeroes.
```

```rs
let v = vec![1, 2, 3, 4, 5];

println!("The third element of v is {}", v[2]);
```

```rs
extern crate rand;

use std::io;
use std::cmp::Ordering;
use rand::Rng;

fn main() {
    println!("Guess the number!");

    let secret_number = rand::thread_rng().gen_range(1, 101);

    loop {
        println!("Please input your guess.");

        let mut guess = String::new();

        io::stdin().read_line(&mut guess)
            .expect("Failed to read line");

        let guess: u32 = match guess.trim().parse() {
            Ok(num) => num,
            Err(_) => continue,
        };

        println!("You guessed: {}", guess);

        match guess.cmp(&secret_number) {
            Ordering::Less    => println!("Too small!"),
            Ordering::Greater => println!("Too big!"),
            Ordering::Equal   => {
                println!("You win!");
                break;
            }
        }
    }
}
```

# Cargo

[Cargo](http://doc.crates.io/) 是 Rust 内置的代码组织管理工具，它借鉴了 Maven, Gradle, Go, Npm 等依赖管理、项目构建工具的优点，提供了一系列的工具，从项目的建立、构建到测试、运行直至部署，为 Rust 项目的管理提供尽可能完整的手段。Cargo 目前包含在了官方的分发包中，Rust 开发环境安装完毕后，我们可以直接在命令行中使用 `cargo` 命令来创建新的项目：

```sh
# --bin 表示该项目将生成可执行文件，否则表示生成库文件
$ cargo new hello-rust --bin
```

`cargo new` 指令会在当前目录下新建了基于 cargo 项目管理的 Rust 项目，我们自动生成了基本运行所必须的所有代码：

```rs
fn main() {
    println!("Hello, world!");
}
```

然后我们可以使用 `cargo build` 命令进行项目编译，编译完毕后 Cargo 会自动生成如下的文件目录 :

```sh
├─src
└─target
    └─debug
        ├─.fingerprint
        │  └─hello-rust-af2f81aa8cdfcf12
        ├─build
        ├─deps
        ├─examples
        ├─incremental
        └─native
```

最后使用 `cargo run` 即自动运行生成的代码：

```sh
Finished dev [unoptimized + debuginfo] target(s) in 0.0 secs
    Running `target\debug\hello-rust.exe`
Hello, world!
```

如果我们希望生成发布包，则可以使用 `cargo build --release` 来进行带优化操作的编译：

```sh
$ cargo build --release
   Compiling hello-rust v0.1.0 (file:///path/to/project/hello-rust)
```

对于现有的 Cargo 项目，我们同样可以直接利用 build 指令进行编译：

```sh
$ git clone https://github.com/rust-lang-nursery/rand.git
$ cd rand

$ cargo build
   Compiling rand v0.1.0 (file:///path/to/project/rand)
```

[Cargo Guide](http://doc.crates.io/guide.html)

## 依赖管理

Cargo 的依赖管理主要依托于 Cargo.toml 与 Cargo.lock 这两个文件，Cargo.toml 文件存储了项目的所有信息；Cargo.lock 则是根据同一项目的 toml 文件生成的项目依赖详细清单文件，类似于 yarn.lock，其可以避免泛版本号带来的依赖版本混乱问题。Cargo 使用了集中式地包管理 [crates.io](https://crates.io/)，我们可以在上面搜索需要的第三方依赖，并将其添加到 Cargo.toml 中 :

```sh
[dependencies]
time = "0.1.12"
regex = "0.1.41"
```

Rust 包管理使用 crate 格式的压缩包存储和发布库，托管在 AWS S3 上；国内中科大则是提供了[资源镜像](https://lug.ustc.edu.cn/wiki/mirrors/help/rust-crates)，我们可以在项目根目录的 `.cargo/config` 或者全局的 `$HOME/.cargo/config` 文件中添加如下 Registry 配置 :

```toml
[source.crates-io]
registry = "https://github.com/rust-lang/crates.io-index"
replace-with = 'ustc'

[source.ustc]
registry = "git://mirrors.ustc.edu.cn/crates.io-index"
```

更详细的 Cargo Configuration 查看[这里](http://doc.crates.io/config.html)。

## 项目结构

# 简单 Web 服务器

[nickel.rs](https://github.com/nickel-org/nickel.rs)

[Creating a basic webservice in Rust](https://parg.co/UPW)
