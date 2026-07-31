# Rust

!!! abstract "基础信息"

    - **官方网站**：https://www.rust-lang.org/
    - **自带工具链**：rustup、rustc、cargo、rustdoc、rustfmt、clippy、rust-analyzer
    - **学习资源**：[The Rust Programming Language](https://doc.rust-lang.org/book)

《Rustc Dev Guide》
《The Rustonomicon》


在 Rust 中，枚举、泛型、Trait、闭包等，相比于其他编程语言而言，应用更加广泛，在日常开发中会频繁使用到。

Rust 函数通常不用`return`关键字返回，而是用表达式的返回值作为函数的返回值。


| Trait      | 实现范围               | `#[derive]` | 作用                                               |
| ---------- | ---------------------- | :---------: | -------------------------------------------------- |
| Copy       | 基础类型及简单组合类型 | ✓           | 赋值/传参时按位复制，而非移动所有权                |
| Send       | 大多数类型             | 自动         | 类型可以安全移动到其他线程                         |
| Sync       | 大多数类型             | 自动         | 类型的引用可以安全地在多线程间共享                 |
| Unpin      | 大多数类型             | 自动         | 类型可以在内存中安全移动（不受 Pin 约束）          |
| Debug      | 大多数类型             | ✓           | 支持 `{:?}` 格式化输出，用于调试打印               |
| Display    | 需手动实现             | ✗           | 支持 `{}` 格式化输出，面向用户；附带自动实现 `.to_string()` |
| Clone      | 大多数类型             | ✓           | 提供显式的深拷贝能力（`.clone()`）                 |
| PartialEq  | 大多数类型             | ✓           | 支持 `==` / `!=` 比较（允许不可比较值，如 `NaN`）  |
| Eq         | 整数、字符串等         | ✓           | 在 `PartialEq` 基础上保证完全等价关系（无 `NaN`）  |
| PartialOrd | 大多数类型             | ✓           | 支持 `<` / `>` / `<=` / `>=` 比较（可能无序）     |
| Ord        | 整数、字符串等         | ✓           | 提供全序比较，可用于排序（`sort()`）               |
| Hash       | 大多数类型             | ✓           | 支持哈希计算，可作为 `HashMap` / `HashSet` 的键    |


常见内置属性

| 属性 | 作用 |
| :--- | :--- |
| `#[derive(...)]` | 自动实现 trait |
| `#[cfg(...)]` | 条件编译 |
| `#[test]` | 标记测试函数 |
| `#[allow(...)]` | 关闭某个警告 |
| `#[inline]` | 建议编译器内联函数 |


---


- 核心知识点
  - 变量与遮蔽
  - 结构体与枚举（`some(T)`、`Option<T>`、`match`、`if let`）
  - 所有权（引用与借用&）
  - 生命周期（`'a`）
  - 泛型与特征
  - 智能指针
  - 无畏并发
  - 异步编程

---

# rustup

`rustup` 是管理 Rust 版本和相关工具（`cargo`、`rustc`、`rustup`、`rustdoc`）的命令行工具

- 更新：`rustup update`
- 卸载：`rustup self uninstall`

# Cargo

项目管理与运行

- `new`：创建新项目（默认是二进制项目，可加 `--lib` 创建库）
- `init`：在现有目录中初始化 Cargo 项目
- `run`：编译并运行项目
- `build`：编译项目（生成可执行文件在 `target/debug`）
- `build --release`：以优化模式编译（生成在 `target/release`）

测试与检查

- `test`：运行测试（#[test] 标记的函数）
- `check`：快速检查代码能否编译（不生成可执行文件）
- `bench`：运行基准测试（需要 nightly）
- `doc`：生成文档（在 target/doc）

依赖管理

- `add`：添加依赖（需要安装 [cargo-edit](https://crates.io/crates/cargo-edit) 插件）
- `update`：更新依赖到最新版本（符合 Cargo.toml 限制）
- `remove`：移除依赖（需要 cargo-edit）

发布与安装

- `install`：安装二进制 crate 到本地（~/.cargo/bin）
- `uninstall`：卸载已安装的二进制 crate
- `publish`：将 crate 发布到 [crates.io](https://crates.io/)

其他常用

- `fmt`：按官方风格格式化代码（需要 rustfmt）
- `clippy`：静态分析代码并给出优化建议（需要 clippy）
- `tree`：查看依赖树（需要 cargo-tree 插件）

# 所有权与借用

以下是《Linux/Unix 系统编程手册》中的进程内存布局：

![Untitled](%E5%BC%80%E5%8D%B7%E6%9C%89%E7%9B%8A/%E9%98%85%E8%AF%BB%E5%88%97%E8%A1%A8/%E3%80%8ALinux%20Unix%20%E7%B3%BB%E7%BB%9F%E7%BC%96%E7%A8%8B%E6%89%8B%E5%86%8C%E3%80%8B/Untitled.png)

以下是《深入理解计算机系统》中的进程内存布局：

![根据数据类型来说，在大多数编程语言中，数组和对象通常被动态分配在堆上，因为它们的大小不固定，并且需要在运行时进行动态分配和释放内存；字符串的存储位置通常取决于编程语言和字符串的类型。在某些编程语言中，字符串常常是不可变的，因此它们可能存储在常量区或堆上。但在其他情况下，编译器可能会将字符串存储在栈上，特别是对于局部变量的字符串或者短期字符串；基本数据类型（如整数、浮点数）通常会被存储在栈上，因为它们的大小固定。](%E5%BC%80%E5%8D%B7%E6%9C%89%E7%9B%8A/%E9%98%85%E8%AF%BB%E5%88%97%E8%A1%A8/%E3%80%8A%E6%B7%B1%E5%85%A5%E7%90%86%E8%A7%A3%E8%AE%A1%E7%AE%97%E6%9C%BA%E7%B3%BB%E7%BB%9F%E3%80%8B%EF%BC%88%E7%AC%AC%E4%B8%89%E7%89%88%EF%BC%89/Untitled.png)

根据数据类型来说，在大多数编程语言中，数组和对象通常被动态分配在堆上，因为它们的大小不固定，并且需要在运行时进行动态分配和释放内存；字符串的存储位置通常取决于编程语言和字符串的类型。在某些编程语言中，字符串常常是不可变的，因此它们可能存储在常量区或堆上。但在其他情况下，编译器可能会将字符串存储在栈上，特别是对于局部变量的字符串或者短期字符串；基本数据类型（如整数、浮点数）通常会被存储在栈上，因为它们的大小固定。

由于栈是以队列的方式读写的，因此栈中的所有数据都必须占用已知且固定大小的内存空间，假设数据大小是未知的，那么在取出数据时，你将无法取到你想要的数据。

当向堆上放入数据时，需要请求一定大小的内存空间。操作系统在堆的某处找到一块足够大的空位，把它标记为已使用，并返回一个表示该位置地址的**指针**，该过程被称为**在堆上分配内存**，有时简称为 “分配”(allocating)。接着，该指针会被推入**栈**中，因为指针的大小是已知且固定的，在后续使用过程中，你将通过栈中的**指针**，来获取数据在堆上的实际内存位置，进而访问该数据。

堆是一种缺乏组织的数据结构。想象一下去餐馆就座吃饭：进入餐馆，告知服务员有几个人，然后服务员找到一个够大的空桌子（堆上分配的内存空间）并领你们过去。如果有人来迟了，他们也可以通过桌号（栈上的指针）来找到你们坐在哪。

**栈内存的生命周期是静态的**，当变量或指向堆内存的指针离开作用域时，内存会被自动释放。**堆内存的生命周期是动态的**，需要显式管理：堆内存没有被指针引用则应该被清除。

基本数据类型存储在栈上，会被自动拷贝，所有权是堆上变量值的管理。

对于执行较为频繁的代码（热点路径），使用 `clone` 会极大的降低程序性能，需要小心使用！

**所有权同时作用于栈和堆，只是栈上的基础数据类型基本都实现了Copy trait，在所有权转移时，编译器允许新旧两个变量同时作为独立的所有者存在。**

引用和变量默认都是不可变的，需要使用`mut`关键字来改为可变。变量不能同时以可变方式借用给多个变量，也不允许同时存在可变引用和不可变引用（避免不可变引用值被修改），这避免了数据竞争。

适用于堆上的概念：GC垃圾回收，深/浅拷贝

栈与堆的关系与就像关系型数据库（RDB）与对象存储（OSS）：

- 结构化vs扁平化：栈与RDB都是结构化数据，堆和OSS则是扁平化无序的
- 指针/引用的映射关系：栈可以指向堆，RDB也会指向OSS
- 速度与容量：栈和RDB都非常快，但容量较小；堆和OSS则容量很大但速度较慢

胖指针：普通指针只有address信息，而对一些复杂类型（如slices），除了地址外还需要知道长度和容量等信息，因此被称为胖指针。

# 数据类型

Rust 中的变量默认不可变，可用 `mut` 关键字改为可变。变量遮蔽允许复用同一个变量名。

Rust中的数据类型分为标量和复合类型：

- 标量：
  - 整型：默认 `i32`
    - 有符号：`i8`~`i128`、`isize`
    - 无符号：`u8`~`u128`、`usize`
  - 浮点型：`f32` 和 `f64` ，默认 `f64`
  - 布尔：`true` 和 `false`
  - 字符：支持 Unicode 单字符的 `char`（4字节）
- 复合：
  - 元组：与Python元组一致，值不可变且写在一对括号内。与Python不同的是，访问时通过`.`来访问索引
  - 数组：长度固定，与Go中数组概念类似，更常用的是`vector`

## 整型

溢出处理

| 处理方法             | 含义                                                       |
| -------------------- | ---------------------------------------------------------- |
| `.wrapping_add()`    | 环绕加法，溢出后绕回最小值或最大值                         |
| `.saturating_add()`  | 溢出时饱和到该类型的最大值或最小值                         |
| `.overflowing_add()` | 返回数值以及是否发生溢出的布尔值                           |
| `.checked_add()`     | 安全检查溢出，发生溢出返回None，没有发生则返回Some(result) |

## 字符串

简单字符串是 `Vec<char>` 类型，标准库使用 `Vec<u8>` 类型。通过UTF-8得到 `u8`，避免 char 的 Unicode 浪费空间。

`&str` 储存在栈中，作为不可变字符时，指向只读数据区，作为 String 切片时，指向堆

String 的3种创建方法：

- `String::new()`
- `String::from("hello")`
- `"hello".to_string()`

字符串的遍历：

- 按照字符输出：`for c in s.chars()`
- 按照字节输出：`for c in s.bytes()`

由于String的UTF-8支持万国语言，因此不支持索引（不同语言边界不同，如中文3字节，英文1字节）

对 String 字符串的操作：

|             | function                       | 返回值      | 支持&str？ | 修改原变量？ | 备注             |
| ------ | -------- | ----------- | ---------- | ------------ | ------------ |
| Push        | `.push_str(str)`               | `()`        |            | ✓            |                  |
|             | `.push(char)`                  | `()`        |            | ✓            |                  |
| Insert      | `.insert_str(index, str)`      | `()`        |            | ✓            |                  |
|             | `.insert(index, char)`         | `()`        |            | ✓            |                  |
| Replace     | `.replace(old_str, new_str)`       | String      | ✓          |              |                  |
|             | `.replacen(old_str, new_str, num)` | String      | ✓          |              |                  |
|             | `.replace_range(range, new_str)`   | `()`        |            | ✓            |                  |
| Delete      | `.pop()`                           | `Option<T>` |            | ✓            |                  |
|             | `.remove(index)`                   | `T`         |            | ✓            | 按照字节来处理字符串的，如果参数所给的位置不是合法的字符边界，则会发生错误。 |
|             | `.truncate(start_index)`           | `()`        |            | ✓            | 按照字节来处理字符串的，如果参数所给的位置不是合法的字符边界，则会发生错误。 |
|             | `.clear()`                         | `()`        |            | ✓            | 与 `.truncate(0)` 等效                                       |
| Concatenate | `+` or `+=`                        |             |            |              | **操作符右侧需要切片引用类型**。`+` 返回新值(`+`左侧的值发生所有权转移)，`+=` 修改原值 |
|             | `format!()`                        |             |            |              |                  |

## 数组

Vector的常用方法：

`for i in &v` 用于不可变的遍历，`for i in &mut v` 则用于可变遍历。

| function                | 返回值       | 功能                       |
| ----------------------- | ------------ | -------------------------- |
| `.get(index)`           | `Option<&T>` | 带越界检查的取值，性能较低 |
| `.is_empty()`           | Boolean      |                            |
| `.insert(index, value)` | `()`         |                            |
| `.remove(index)`        | `T`          |                            |
| `.push(value)`          | `()`         | 在尾部加入元素             |
| `.pop()`                | `Option<T>`  | 从尾部移出元素             |
| `.append(vec)`          | `()`         | 在尾部追加数组             |
| `.truncate(length)`     |              |                            |

## 哈希表

使用时需先导入 `use std::collections::HashMap;`

| function                                    | 功能                                                         |
| ------------------------------------------- | ------------------------------------------------------------ |
| `HashMap::new()`                            | 创建                                                         |
| `.insert(key, value)`                       | 插入                                                         |
| `.entry(query_key).or_insert(insert_value)` | 键不存在时才插入，返回的是指针                               |
| `.get(key)`                                 | 带键值检查的取值，返回Option类型                             |
| `keys.iter().zip(values.iter()).collect()`  | keys和values分别为键和值的两个数组，通过将两个数组合并组合为HashMap |

# 语句和表达式

- **语句**（*Statements*）：执行一些操作但不返回值的指令，如变量声明与赋值、控制流语句、函数
- **表达式**（*Expressions*）：计算并产生一个值，如算术运算、逻辑运算、三元表达式、函数调用

**分号会使表达式变成语句，并且将返回值变成 `()`**

控制语句：`if`-`else if`-`else`

循环语句：loop（可通过 break+返回值作为变量返回、也支持多层嵌套和标签跳转）、while、for（与Python都用`in`关键字）

# 包与模块

```bash
Workspace
│
├── Package A (Cargo.toml)
│   ├── Library Crate (src/lib.rs)
│   │    ├── Module
│   │    └── Module
│   ├── Binary Crate (src/main.rs)
│   │    ├── Module
│   │    └── Module
│   └── Binary Crate (src/bin/foo.rs)
│
├── Package B
│   └── Library Crate
│
└── Package C
    └── Binary Crate
```

package 允许有多个binary crates，但最多只能有一个library crate，如 https://github.com/astral-sh/ruff的文件结构就是多个package

crate 是编译单元，module 是代码组织单元

module通常有两种方式：

- `module.rs`
- `module/mod.rs`

Rust 中所有条目（函数、方法、struct、enum、模块、常量）默认都是私有的，通过 `pub` 可以改为公有。相对路径可以使用`self`、`super`

结构体常用Trait：

- Default：支持 default() 工厂方法，以默认值创建一个结构体对象
- Debug：允许在print!宏中使用 "{:?}" 打印结构体
- Clone：支持 clone() 方法，类似复制构造函数
- Copy：标记该结构体是简单类型，在进行 clone() 时只需内存拷贝

```rust
use std::collections::HashMap;

// 定义结构体
struct Course {
    id: u32,
    name: String,
    students: Vec<u32>,
    schedule: HashMap<u32, u64>,
}

impl Default for Course {
    fn default() -> Self {
        Self {
            id: 0,
            name: String::from("Default Course"),
            students: vec![0],
            schedule: HashMap::new(),
        }
    }
}

use std::fmt::Debug;
impl Debug for Course {
    fn fmt(&self, f: &mut std::fmt::Formatter<'_>) -> std::fmt::Result {
        f.debug_struct("Course")
            .field("id", &self.id)
            .field("name", &self.name)
            .field("students", &self.students)
            .field("schedule", &self.schedule)
            .finish()
    }
}

impl Clone for Course {
    fn clone(&self) -> Self {
        Self {
            id: self.id,
            name: self.name.clone(),
            students: self.students.clone(),
            schedule: self.schedule.clone(),
        }
    }
}
```

`into_` 之类的，都是拿走所有权，`_mut` 之类的都是可变借用，剩下的就是不可变借用。

# derive

使用 `derive` 可以自动实现常见的 trait，减少代码量

| **Trait**       | **用途**                                   | **示例**                         | **备注**                         |
| --------------- | ------------------------------------------ | -------------------------------- | -------------------------------- |
| **Debug**       | 用于格式化输出，便于调试。                 | `#[derive(Debug)]`               | 常用于 `println!` 和 `dbg!` 宏。 |
| **Clone**       | 允许类型实例的深拷贝。                     | `#[derive(Clone)]`               | 通常与 `Copy` 一起使用。         |
| **Copy**        | 允许类型实例的浅拷贝。                     | `#[derive(Copy, Clone)]`         | 需要所有字段都实现 `Copy`。      |
| **PartialEq**   | 实现部分相等比较（`==` 和 `!=`）。         | `#[derive(PartialEq)]`           | 用于比较部分相等。               |
| **Eq**          | 实现完全相等比较。                         | `#[derive(Eq, PartialEq)]`       | 需要类型满足自反性。             |
| **PartialOrd**  | 实现部分排序比较（`<`, `>`, `<=`, `>=`）。 | `#[derive(PartialOrd)]`          | 用于部分排序。                   |
| **Ord**         | 实现完全排序比较。                         | `#[derive(Ord, PartialOrd, Eq)]` | 需要类型满足全序关系。           |
| **Hash**        | 允许类型实例作为哈希表的键。               | `#[derive(Hash)]`                | 用于 `HashMap` 或 `HashSet`。    |
| **Default**     | 为类型提供默认值。                         | `#[derive(Default)]`             | 生成 `Default::default()` 方法。 |
| **Serialize**   | 用于序列化（需要 `serde` crate）。         | `#[derive(Serialize)]`           | 通常与 `Deserialize` 一起使用。  |
| **Deserialize** | 用于反序列化（需要 `serde` crate）。       | `#[derive(Deserialize)]`         | 通常与 `Serialize` 一起使用。    |

# 结构体

Rust和Python中的方法第一个参数都是`self`

- 只有结构体没有类：C、Go、Rust（设计思想：组合优于继承）
- 只有类没有结构体：Java、Python、JavaScript、PHP

结构体实现了对象属性的封装，方法则通过`impl`（显式）或接收者（隐式）与数据属性绑定：结构体+绑定的方法（`impl`/`Receiver`）≈OOP

以酒店退房为例来理解GC和RAII：

- GC语言：
  - 客人退房后，保洁会根据当前的房间紧张情况判断是立即打扫还是定时打扫
  - 打扫时先把所有需要打扫的房间门敞开，确定待打扫房间，并防止客人误入（标记与STW）
  - 保洁开始打扫房间，期间禁止新的客人入住（清除与STW）
- RAII语言：
  - 每个房间都有RAII来管里，客人退房时（离开作用域drop），房间会像沙箱一样被瞬间重置，酒店不需要保洁，也能随时让新客人入住

PHP 中也存在析构函数 `__destruct()` ，并通过引用计数来实现垃圾回收。GC 主要解决循环引用的问题，工作原理是：通过模拟减1操作，确认循环引用后将 `refcount` 置 0，然后引用计数调用析构函数清理。

由于 PHP 是动态语言，编译器在编译时无法确定变量的生命周期，因此几乎所有的对象、数组都是在堆（Heap）上分配的，必须依赖垃圾回收。Go则会在编译期间对变量进行逃逸分析，如果变量在函数结束后不会被外部引用，则会分配在更加高效的栈上，因此需要在堆上的垃圾清理要少很多。

PHP 和 Python 因其单线程而能使用引用计数，在多线程环境下，每次计数修改都需要原子操作，而原子操作在多核CPU上的开销是非常昂贵的（通常比普通读写慢数十倍）

vector 和 hashmap 都可以像Python那样的`get()`确保非空值读取

# 泛型

泛型通常在函数入参和出参指定

```rust
fn largest<T>(list: &[T]) → &T { // <T> 是声明使用T作为泛型类型
```

泛型类型参数可以声明多个，但过多的泛型意味着代码需要拆分为更小的模块

```rust
struct Point<T, U> {
	x: T,
	y: U,
}

// 实现泛型方法
impl<T> Point<T> {
	fn x(&self) -> &T {
		&self.x
	}
}

// 实现具体方法
impl Point<f32> {

}
```

```bash
enum Option<T>{
	Some(T),
	None,
}
```

# 枚举

Rust中的枚举表示A或B或C，结构体则表示A且B且C

# 生命周期

生命周期是确保引用的作用域，**防止悬垂引用**（引用已经被释放的地址）。每个引用都有生命周期，大多数情况下生命周期都是隐式，且可被推断出来。+

生命周期必须与`'`和小写字母开头，常用`'a` ，如 `&'a mut i32`

```rust
fn longest<'a>(x: &'a str, y: &'a str) -> &'a str {
	if x.len() > y.len() { x } else { y }
}
```

`struct`中的`&str`必须声明生命周期

`'static`表示受影响的引用可以在整个程序的持续时间内存活

```rust
use std::fmt::Display;

fn longest_with_an_announcement<'a, T>(
	x: &'a str,
	y: &'a str,
	ann: T,
) -> &'a str
where 
	T: Display,
{
	println!("announcement! {ann}");
	if x.len() > y.len() { x } else { y }
}
```

# 闭包

Rust 的闭包写法与其他语言的闭包差异较大，闭包不是以`fn`开头，入参需要放在 `|x|` 之间

可通过 `move` 强制将环境变量的所有权转移给闭包

# 宏

```rust
#[derive()]
#[test]
#[cfg()]
```

# 迭代器

- `iter()`：不可变引用的迭代器
- `into_iter()`：能获得变量的所有权，并返回具有所有权的值
- `iter_mut()`：遍历可变的引用

# 零成本抽象

- 迭代器与闭包
- 泛型与静态分发
- 所有权、生命周期、借用检查器
- 异步编程

# 智能指针

智能指针虽然在行为上类似于普通的引用（指针），但包含了比普通引用更多的元数据（Metadata）和额外的功能。

普通指针（如 `&T`）只负责“指向”和“借用”内存，而智能指针通常**拥有**它们所指向的数据的所有权。Rust 的很多标准库类型（如 `String` 和 `Vec<T>`）在底层其实也属于智能指针。

任何实现了`Deref` （允许智能指针像普通引用一样被使用）和 `Drop` （允许自定义智能指针离开作用域时的清理逻辑）Trait 的自定义类型都是智能指针。

常见的标准智能指针：

- `Box<T>`——堆分配指针，该指针可以直接把数据分配到堆上，栈上只保留指向堆内存的指针，适用于以下场景

  - 在编译期间无法确定大小的类型，如链表、树
  - 转移非常庞大的数据结构所有权，同时避免栈上频繁复制数据

  ```rust
  // 递归类型示例：链表节点
  enum List {
    Cons(i32, Box<List>),
    Nil,
  }
  ```

- `Rc<T>`——引用计数（单线程），Rc是Reference Counted的缩写。普通的 Rust 变量只允许有一个所有者，而 **`Rc<T>` 允许在单线程中拥有多个所有者**，比如图的节点。其工作原理与引用计数的GC相同，每次 `.clone()` 时+1，离开作用域-1。该指针内部没有多线程锁保护，只能在单线程中使用。

- `Arc<T>`——原子引用计数（多线程安全），是 Atomically Reference Counted 的缩写。功能与 Rc 完全相同，但是线程安全，代价是开销更高。

- `RefCell<T>`，通过将借用规则的检查从**编译期**推迟到了**运行期，**允许在拥有“不可变引用”的情况下，依然能够**修改内部数据**。如果在运行期违反了“一个可变借用或多个不可变借用”的规则，程序在编译时不会报错，但会在运行时发生 panic。与`Rc<T>`一样，`RefCell<T>`也只适用于单线程场景。

  - `borrow() -> Ref<T>`：获取内部不可变引用，与 `&` 类似，
  - `borrow_mut()->RefMut<T>` ：获取内部可变引用，与 `&mut` 类似，

常见搭配 `Rc<RefCell<T>>`，由于 Rc 是只读的，为了实现“单线程内既有多个所有者，又能修改数据”的需求（例如实现复杂的链表、图结构），通常将两者结合使用。

通过`Rc<T>`和`RefCell<T>`就能创建出循环引用，导致内存泄露

| **智能指针**         | **存储位置** | **线程安全**    | **所有权数量** | **读写权限** | **核心应用场景**                             |
| -------------------- | ------------ | --------------- | -------------- | ------------ | -------------------------------------------- |
| **`Box<T>`**         | 堆           | 是（视 T 而定） | 独占           | 可读写       | 编译期未知大小、避免大对象拷贝               |
| **`Rc<T>`**          | 堆           | **否**          | 共享           | **只读**     | 单线程中的多处数据共享                       |
| **`Arc<T>`**         | 堆           | **是**          | 共享           | **只读**     | 多线程间的数据共享                           |
| **`RefCell<T>`**     | 栈/堆        | 否              | 独占           | 动态可读写   | 需要规避编译期借用规则，实现内部可变性       |
| **`Rc<RefCell<T>>`** | 堆           | 否              | 共享           | 动态可读写   | 单线程多所有者且需要修改数据（如图、树结构） |

# 无畏并发

在Rust中，通过所有权和类型系统在编译时就能发现并发错误，因此被称为无畏并发。

Rust标准库采用1:1的线程实现模型，即程序中的一个语言线程对应一个操作系统线程。

线程有多种创建方式：

- `thread:spawn()` ：通过返回的 `JoinHandle` 来等待子线程结束。如果子线程中使用主线程中的变量，必须使用 `move` 把所有权彻底转移到子线程中
- `thread::scope()` ：更加推荐的作用域线程，子线程临时借用外部变量，而不需要强制转移所有权。

Tokio 是类似于 goroutine 的M:N协程

Rust通过Send（所有权能否跨线程转移）和Sync（多线程能否通过`&T`安全访问）确保代码在多线程下运行不会发生数据竞争和崩溃

## Send

几乎所有Rust类型都是Send，这允许它们的所有权可以在线程间转移

Send类型系统和trait约束确保不会意外地将非Send类型跨线程发送

## Sync

可以安全地从多个线程引用实现该trait的类型

如果`&T`是Send，则类型T是Sync，即该引用可以安全地发送到另一个线程

Sync 是 Rust 中最接近线程安全（特定数据可以被多个并发线程安全使用）的概念

|                     | Send         | Sync         |
| ------------------- | ------------ | ------------ |
| `Rc<T>`             | ✗            | ✗            |
| `RefCell<T>`        | ✓（T是Send） | ✗            |
| `Mutex<T>`          | ✓            | ✓            |
| `MutexGuard<’a, T>` | ✗            | ✓（T是Sync） |

# 互斥锁

mutex 是 mutual exclusion 的组合

传统编程语言（如C/C++）中，互斥锁在保护代码片段，而Rust中的互斥锁是在保护数据。

传统语言中，解锁需要手动操作，如果代码加锁后因为报错或提前`return`或抛出异常，很容易发生忘记解锁而导致的死锁。Rust则将锁的生命周期绑定在了特殊的智能指针 `MutexGuard` 中，离开作用域锁会被自动释放。所以Rust标准库的Mutex只有加锁而没有解锁。

# 异步编程

- `Future`：现在尚未就绪，但未来会就绪的值。`Future`是惰性的。JavaScript中称为promise
- `async`：表示可被中断和恢复，把代码块或函数转换为返回Future的形式
- `await`：等待 Future 的就绪，提供暂停和恢复执行，通过轮询（polling）检查Future的值是否可用
- `yield_now()`：协作式主动让出当前的执行权

`await` 只有在发生阻塞时才会让出控制权，因此如果一个任务 `await` 后任务已就绪，就会一直占用CPU，导致其他任务饥饿。而 `yield_now()` 则无论任务是否就绪，一定会返回 pending，从而避免饿死其他任务。

tokio 是最流行的异步运行时

如果希望处理动态数量的Future，可以使用`join_all`，但前提是它们必须具有相同的类型

如果要处理固定数量的Future，则可以使用`join`函数或`join!`宏，即使它们具有不同的类型

## Stream

stream就是流式I/O读写

Stream就像异步版本的Iterator，可以从任何iterator来创建Stream

异步编程中的Trait：`Future()`、`Pin()`、`Unpin()`、

绝大多数普通类型都自动实现了Unpin Trait

当发生所有权转移时，栈上的Box会被移动到新的内存地址，并且复制其指向堆内存的地址内容，但为了防止Box的内容被可变引用修改，因此需要通过Pin来阻止修改

## Unsafe

允许使用FFI来调用外部语言编写的函数