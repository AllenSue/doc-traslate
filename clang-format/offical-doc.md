翻译: Sue

# Clang-Format格式选项

Clang-Format格式选项描述了LibFormat和ClangFormat支持的可配置选项.

当调用命令行工具或在代码中调用`clang::format::reformat(...)`函数执行clang-format时, 用户可以选择使用预设的配置或创建自定义配置.

## 使用clang-format配置格式

clang-format有两种配置格式的方式: 直接通过命令行选项`-style=`指定具体的格式配置项; 或输入`-style=file`并将配置项写入工程中的`.clang-format`或`_clang-format`文件.

当使用`-style=file`的方式时, clang-format会尝试从距离输入文件最近的上级目录中寻找`.clang-format`文件. 在使用标准输入的情况下, 默认从当前路径开始搜索.

`.clang-format`文件采用YAML格式:

```
key1: value1
key2: value2
# A comment.
...
```

配置文件可以由多个若干个section, 每个section通过`Language:`表明配置作用于哪种语言. 可以从后文的支持语言列表中查看关于语言选项的具体描述. 配置文件中的第一个section可能不指定具体的语言, 这个sectiona将为所有的语言提供默认的配置. 其他指定语言的section中的重复配置会覆盖这些默认值.

下面是一个支持多语言的配置文件的示例:

```
---
# We'll use defaults from the LLVM style, but with 4 columns indentation.
BasedOnStyle: LLVM
IndentWidth: 4
---
Language: Cpp
# Force pointers to the type for C++.
DerivePointerAlignment: false
PointerAlignment: Left
---
Language: JavaScript
# Use 100 columns for JS.
ColumnLimit: 100
---
Language: Proto
# Don't format .proto files.
DisableFormat: true
...
```

如下指令可以获得一个将所有可配置项设为预设值的`.clang-format`文件:

```
clang-format -style=llvm -dump-config > .clang-format
```

当使用`-sytle=`方式指定配置项时, 这些配置会应用于所有的输入文件. 配置格式如下:

```
-style='{key1: value1, key2: value2, ...}'
```

## 禁止一段代码的格式化

Clang-format可以识别代码中特殊的注释来打开或关闭格式化功能. 被包含在诸如`// clang-format off`和`// clang-format on`这类注释之间的代码不会被格式化. 而这两行注释本身会被格式化(主要指文本对齐).

```
int formatted_code;
// clang-format off
void    unformatted_code  ;
// clang-format on
void formatted_code_again;
```

## 在代码中配置格式

使用`clang::format::reformat(...)`时, 具体格式由`clang::format::FormatStyle`指定.

## 配置可选项

本小节列举支持的配置项. 配置值的类型取决于具体的配置项. 枚举类型的配置项可以从C++枚举成员(包含前缀, 如: LS_Auto)或可选的配置项(不包含前缀: Auto)中选择.

### BasedOnStyle(`string`)

预设配置中没有指定所有支持的配置选项.

此选项仅在**clang-format**设置中受支持(包括`-style='{...}'`和`.clang-format`文件).

可选取值:

- `LLVM` 符合LLVM编码标准.
- `Google` 符合Google C++编码风格手册.
- `Chromium` 符合Chromium编码风格手册.
- `Mozilla` 符合Mozilla编码风格手册.
- `WebKit` 符合WebKit编码风格手册.

### AccessModifierOffset(`int`)

修饰符额外的缩进或减少缩进, 比如`public:`

### AlignAfterOpenBracket(`BracketAlignmentStyle`)

为`true`时, 参数纵向对齐到左括号.

该选项应用于圆括号, 尖括号和方括号.

可选取值:

- `BAS_Align`(配置文件中: `Align`) 参数对齐到左括号:

```
someLongFunction(argument1,
argument2);
```

- `BAS_DontAlign`(配置文件中: `DontAlign`) 不对齐参数, 使用`ContinuationIndentWidth`代替:

```
someLongFunction(argument1,
argument2);
```

- `BAS_AlwaysBreak`(配置文件中: `AlwaysBreak`) 当参数不能放在同一行时, 始终从左括号折行:

```
someLongFunction(
argument1, argument2);
```

### AlignConsecutiveAssignments(`bool`)

为`true`时, 对齐连续的参数. (译注: 这个参数应该是指操作符右值对应的参数, 不是变量)

该选项会对齐连续行的参数操作符. 效果如:

```
int aaaa = 12;
int b    = 23;
int ccc  = 23;
```

### AlignConsecutiveDeclarations(`bool`)

为`true`时, 对齐连续的声明.

该选项会对齐连续行的声明变量名. 效果如:

```
int         aaaa = 12;
float       b = 23;
std::string ccc = 23;
```

### AlignEscapedNewlines (`EscapedNewlineAlignmentStyle`)

配置反斜杠折行符的对齐方式.

可选取值:

- `ENAS_DontAlign`(配置文件中: `DontAlign`) 不对齐折行符:

```
#define A \
int aaaa; \
int b; \
int dddddddddd;
```

- `ENAS_Left`(配置文件中: `Left`) 折行符尽可能靠左的对齐:

```
#define A   \
int aaaa; \
int b;    \
int dddddddddd;
```

- `ENAS_Right`(配置文件中: `Right`) 折行符靠右对齐到最右侧一列:

```
#define A                                                                      \
int aaaa;                                                                    \
int b;                                                                       \
int dddddddddd;
```

### AlignOperands(`bool`)

为`true`时, 纵向对齐二元和三元表达式的操作数.

具体来说, 该选项会对齐需要被拆分为多行的单一表达式:

```
int aaa = bbbbbbbbbbbbbbb +
ccccccccccccccc;
```

### AlignTrailingComments(`bool`)

为`true`时, 对齐行尾的注释:

```
true:                                   false:
int a;     // My comment a      vs.     int a; // My comment a
int b = 2; // comment  b                int b = 2; // comment about b
```

### AllowAllParametersOfDeclarationOnNextLine(`bool`)

当函数声明不能在一行完成时, 允许将所有参数放到下一行, 即使已将`BinPackParameters`设置为`false`:

```
true:
void myFunction(
int a, int b, int c, int d, int e);

false:
void myFunction(int a,
int b,
int c,
int d,
int e);
```

### AllowShortBlocksOnASingleLine(`bool`)

允许将单行的代码段合并到一行.

如`if (a) { return; }`. (译注: 这个重点是指大括号括起来的语句只有一行.)

### AllowShortCaseLabelsOnASingleLine(`bool`)

为`true`时, 短的case标签会被整合到一行:

```
true:                                   false:
switch (a) {                    vs.     switch (a) {
case 1: x = 1; break;                   case 1:
case 2: return;                           x = 1;
}                                         break;
case 2:
return;
}
```

### AllowShortFunctionsOnASingleLine(`ShortFunctionStyle`)

根据取值, 可能将`int f() { return 0; }`合并为单行.

可选取值:

- `SFS_None`(配置文件中: `None`) 从不合并.
- `SFS_InlineOnly`(配置文件中: `InlineOnly`) 只合并类中定义的函数. 与"内联"相同, 但并不包括: 即, 顶层的空函数不会被合并:

```
class Foo {
void f() { foo(); }
};
void f() {
foo();
}
void f() {
}
```

- `SFS_Empty`(配置文件中: `Empty`) 仅合并空函数:

```
void f() {}
void f2() {
bar2();
}
```

- `SFS_Inline`(配置文件中: `Inline`) 仅合并类中定义的函数. 包括空函数:

```
class Foo {
void f() { foo(); }
};
void f() {
foo();
}
void f() {}
```

- `SFS_All`(配置文件中: `All`) 合并所有可单行表示的函数:

```
class Foo {
void f() { foo(); }
};
void f() { bar(); }
```

### AllowShortIfStatementsOnASingleLine(`bool`)

为`true`时, `if (a) return;`可以合并为一行.

### AllowShortLoopsOnASingleLine(`bool`)

为`true`时, `while (true) continue;`可以合并为一行.

### AlwaysBreakAfterDefinitionReturnType(`DefinitionReturnTypeBreakingStyle`)

配置函数定义中返回类型之后的折行方式. 不建议使用该选项, 只留作向后兼容.

可选取值:

- `DRTBS_None`(配置文件中: `None`) 自动决定是否在函数定义中的返回类型之后折行. 取决于`PenaltyReturnTypeOnItsOwnLine`的值.
- `DRTBS_All`(配置文件中: `All`) 始终折行.
- `DRTBS_TopLevel`(配置文件中: `TopLevel`) 始终在顶层的函数定义中折行

### AlwaysBreakAfterReturnType(`ReturnTypeBreakingStyle`)

配置函数定义中返回类型之后的折行方式.

可选取值:

- `RTBS_None`(配置文件中: `None`) 自动决定是否在函数定义中的返回类型之后折行. 取决于`PenaltyReturnTypeOnItsOwnLine`的值:

```
class A {
int f() { return 0; };
};
int f();
int f() { return 1; }
```

- `RTBS_All`(配置文件中: `All`) 始终折行:

```
class A {
int
f() {
return 0;
};
};
int
f();
int
f() {
return 1;
}
```

- `RTBS_TopLevel`(配置文件中: `TopLevel`) 始终在顶层的函数定义中折行:

```
class A {
int f() { return 0; };
};
int
f();
int
f() {
return 1;
}
```

- `RTBS_AllDefinitions`(配置文件中: `AllDefinitions`) 始终在所有的函数定义中折行(译注: 函数声明不折行, 后文同):

```
class A {
int
f() {
return 0;
};
};
int f();
int
f() {
return 1;
}
```

- `RTBS_TopLevelDefinitions`(配置文件中: `TopLevelDefinitions`) 始终再顶层的u函数定义中折行:

```
class A {
int f() { return 0; };
};
int f();
int
f() {
return 1;
}
```

### AlwaysBreakBeforeMultilineStrings(`bool`)

为`true`时, 始终在多行字符串之前折行.

这个选项主要是为了使得多行字符串看起来更为一致. 因此, 只会在包含的字符串导致它从行首缩进`ContinuationIndentWidth`时才会生效:

```
true:                                  false:
aaaa =                         vs.     aaaa = "bbbb"
"bbbb"                                    "cccc";
"cccc";
```

### AlwaysBreakTemplateDeclarations(`BreakTemplateDeclarationsStyle`)

配置模板声明的折行方式.

可选取值:

- `BTDS_No`(配置文件中: `No`) 不强制折行. 此时`PenaltyBreakTemplateDeclaration`选项生效.

```
template <typename T> T foo() {
}
template <typename T> T foo(int aaaaaaaaaaaaaaaaaaaaa,
int bbbbbbbbbbbbbbbbbbbbb) {
}
```

- `BTDS_MultiLine`(配置文件中: `MultiLine`) 当声明会被分割为多行时, 在模板之后强制折行:

```
template <typename T> T foo() {
}
template <typename T>
T foo(int aaaaaaaaaaaaaaaaaaaaa,
int bbbbbbbbbbbbbbbbbbbbb) {
}
```

- `BTDS_Yes`(配置文件中: `Yes`) 始终在模板之后强制折行:

```
template <typename T>
T foo() {
}
template <typename T>
T foo(int aaaaaaaaaaaaaaaaaaaaa,
int bbbbbbbbbbbbbbbbbbbbb) {
}
```

### BinPackArguments(`bool`)

为`false`时, 函数调用时传递的参数可能在同一行或每个参数各占一行(译注: 即, 不会判断文本长度自动将多行参数合并为一行, 后文同):

```
true:
void f() {
f(aaaaaaaaaaaaaaaaaaaa, aaaaaaaaaaaaaaaaaaaa,
aaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaa);
}

false:
void f() {
f(aaaaaaaaaaaaaaaaaaaa,
aaaaaaaaaaaaaaaaaaaa,
aaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaa);
}
```

### BinPackParameters(`bool`)

为`false`时, 函数定义或声明中的参数可能在同一行或每个参数各占一行:

```
true:
void f(int aaaaaaaaaaaaaaaaaaaa, int aaaaaaaaaaaaaaaaaaaa,
int aaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaa) {}

false:
void f(int aaaaaaaaaaaaaaaaaaaa,
int aaaaaaaaaaaaaaaaaaaa,
int aaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaa) {}
```

### BraceWrapping(`BraceWrappingFlags`)

控制不同情况下的大括号是否自动换行.

当`BreakBeforeBraces`为`BS_Custom`时, 采用此处配置决定不同情况下的大括号分别如何处理; 否则忽略此处配置.

```
# Example of usage:
BreakBeforeBraces: Custom
BraceWrapping:
AfterEnum: true
AfterStruct: false
SplitEmptyFunction: false
```

内置配置项包括:

- `bool AfterClass` 类定义是否换行:

```
true:
class foo {};

false:
class foo
{};
```

- `bool AfterControlStatement` 条件控制是否换行:

```
true:
if (foo())
{
} else
{}
for (int i = 0; i < 10; ++i)
{}

false:
if (foo()) {
} else {
}
for (int i = 0; i < 10; ++i) {
}
```

- `bool AfterEnum` 枚举是否换行:

```
true:
enum X : int
{
B
};

false:
enum X : int { B };
```

- `bool AfterFunction` 函数是否换行:

```
true:
void foo()
{
bar();
bar2();
}

false:
void foo() {
bar();
bar2();
}
```

- `bool AfterNamespace` 命名空间是否换行:

```
true:
namespace
{
int foo();
int bar();
}

false:
namespace {
int foo();
int bar();
}
```

- `bool AfterObjCDeclaration` ObjC定义是否换行(interfaces, implemetations...). `@autoreleasepool`和`@synchronized`块是否换行由*AfterControlStatement*决定.

- `bool AfterStruct` 结构体是否换行:

```
true:
struct foo
{
int x;
};

false:
struct foo {
int x;
};
```

- `bool AfterUnion` 联合体是否换行:

```
true:
union foo
{
int x;
}

false:
union foo {
int x;
}
```

- `bool AfterExternBlock` 外部引用是否换行:

```
true:
extern "C"
{
int foo();
}

false:
extern "C" {
int foo();
}
```

- `bool BeforeCatch` `catch`之前是否换行:

```
true:
try {
foo();
}
catch () {
}

false:
try {
foo();
} catch () {
}
```

- `bool BeforeElse` `else`之前是否换行:

```
true:
if (foo()) {
}
else {
}

false:
if (foo()) {
} else {
}
```

- `bool IndentBraces` 是否缩进换行后的大括号.

- `bool SplitEmptyFunction` 为`false`时, 空函数的函数体内容部分可以合并到一行. 该选项仅在函数的左大括号自动换行时有效, 即, *AfterFunction*自动换行模式开启, 并且函数不能/不应该合并为单行(受*AllowShortFunctionsOnASingleLine*和构造函数格式化选项影响).

```
int f()   vs.   inf f()
{}              {
}
```

- `bool SplitEmptyRecord` 为`false`时, 空数据结构(如类, 结构体或联合体)内容部分可以合并到一行. 该选项仅在左大括号自动换行时有效, 即, *AfterClass*自动换行模式开启.

```
class Foo   vs.  class Foo
{}               {
}
```

- `bool SplitEmptyNamespace` 为`false`时, 空命名空间内容部分可以合并到一行. 该选项仅在命名空间的左大括号自动换行时有效, 即, *AfterNamespace*自动换行模式开启.

```
namespace Foo   vs.  namespace Foo
{}                   {
}
```

### BreakAfterJavaFieldAnnotations(`bool`)

Java文件中, 在注释的每个field之后折行:

```
true:                                  false:
@Partial                       vs.     @Partial @Mock DataLoad loader;
@Mock
DataLoad loader;
```

### BreakBeforeBinaryOperators(`BinaryOperatorStyle`)

设置运算符的自动换行方式.

可选取值:

- `BOS_None`(配置文件中: `None`) 在运算符之后换行:

```
LooooooooooongType loooooooooooooooooooooongVariable =
someLooooooooooooooooongFunction();

bool value = aaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaa +
aaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaa ==
aaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaa &&
aaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaa >
ccccccccccccccccccccccccccccccccccccccccc;
```

- `BOS_NonAssignment`(配置文件中: `NonAssignment`) 在非赋值运算符之前换行:

```
LooooooooooongType loooooooooooooooooooooongVariable =
someLooooooooooooooooongFunction();

bool value = aaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaa
+ aaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaa
== aaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaa
&& aaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaa
> ccccccccccccccccccccccccccccccccccccccccc;
```

- `BOS_All`(配置文件中: `All`) 在所有运算符之前换行:

```
LooooooooooongType loooooooooooooooooooooongVariable
= someLooooooooooooooooongFunction();

bool value = aaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaa
+ aaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaa
== aaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaa
&& aaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaa
> ccccccccccccccccccccccccccccccccccccccccc;
```

### `BreakBeforeBraces`(`BraceBreakingStyle`)

设置大括号的换行方式.

可选取值:

- `BS_Attach`(配置文件中: `Attach`) 大括号始终跟随前文:

```
try {
foo();
} catch () {
}
void foo() { bar(); }
class foo {};
if (foo()) {
} else {
}
enum X : int { A, B };
```

- `BS_Linux`(配置文件中: `Linux`) 与`Attach`相似, 但在函数, 命名空间和类定义时, 左大括号换行:

```
try {
foo();
} catch () {
}
void foo() { bar(); }
class foo
{
};
if (foo()) {
} else {
}
enum X : int { A, B };
```

- `BS_Mozilla`(配置文件中: `Mozilla`) 与`Attach`相似, 但在枚举, 函数和数据结构定义时, 左大括号换行:

```
try {
foo();
} catch () {
}
void foo() { bar(); }
class foo
{
};
if (foo()) {
} else {
}
enum X : int
{
A,
B
};
```

- `BS_Stroustrup`(配置文件中: `Stroustrup`) 与`Attach`相似, 但在函数定义时左大括号换行, `catch`和`else`之前换行:

```
try {
foo();
}
catch () {
}
void foo() { bar(); }
class foo {
};
if (foo()) {
}
else {
}
enum X : int { A, B };
```

- `BS_Allman`(配置文件中: `Allman`) 所有左大括号都换行(译注: 从`Mozilla`到此选项翻译时官网的示例和文字说明并不匹配, 此处示例是为译者猜想):

```
try
{
foo();
}
catch ()
{
}
void foo() { bar(); }
class foo
{
};
if (foo())
{
}
else
{
}
enum X : int
{
A,
B
};
```

- `BS_GNU`(配置文件中: `GNU`) 所有左大括号都换行, 并且在流控类语句时产生额外的缩进(定义语句处不产生缩进):

```
try
{
foo();
}
catch ()
{
}
void foo() { bar(); }
class foo
{
};
if (foo())
{
}
else
{
}
enum X : int
{
A,
B
};
```

- `BS_WebKit`(配置文件中: `WebKit`) 与`Attach`相似, 但在函数定义时, 左大括号换行(译注: 此处示例无法展示实际效果, 请自行实验):

```
try {
foo();
} catch () {
}
void foo() { bar(); }
class foo {
};
if (foo()) {
} else {
}
enum X : int { A, B };
```

- `BS_Custom`(配置文件中: `Custom`) 在*BraceWrapping*中分别设置不同场景的大括号.

### BreakBeforeTernaryOperators(`bool`)

为`true`时, 三元操作符放在换行符之后:

```
true:
veryVeryVeryVeryVeryVeryVeryVeryVeryVeryVeryLongDescription
? firstValue
: SecondValueVeryVeryVeryVeryLong;

false:
veryVeryVeryVeryVeryVeryVeryVeryVeryVeryVeryLongDescription ?
firstValue :
SecondValueVeryVeryVeryVeryLong;
```

### BreakConstructorInitializers(`BreakConstructorInitializersStyle`)

设置构造函数初始化格式.

可选取值:

-`BCIS_BeforeColon`(配置文件中: `BeforeColon`) 构造函数初始化器列表中, 在冒号之前和逗号之后换行:

```
Constructor()
: initializer1(),
initializer2()
```

- `BCIS_BeforeComma`(配置文件中: `BeforeComma`) 构造函数初始化器列表中, 在冒号和逗号之前换行, 并将逗号与冒号对齐:

```
Constructor()
: initializer1()
, initializer2()
```

- `BCIS_AfterColon`(配置文件中: `AfterColon`) 构造函数初始化器列表中, 在冒号和逗号之后换行:

```
Constructor() :
initializer1(),
initializer2()
```

- `BreakInheritanceList`(`BreakInheritanceListStyle`)

设置继承格式.

可选取值:

- `BILS_BeforeColon`(配置文件中: `BeforeColon`) 继承列表中, 在冒号之前和逗号之后换行:

```
class Foo
: Base1,
Base2
{};
```

- `BILS_BeforeComma`(配置文件中: `BeforeComma`) 继承列表中, 在冒号和逗号之前换行, 并将逗号与冒号对齐:

```
class Foo
: Base1
, Base2
{};
```

- `BILS_AfterColon`(配置文件中: `AfterColon`) 继承列表中, 在冒号和逗号之后换行:

```
class Foo :
Base1,
Base2
{};
```

### BreakStringLiterals(`bool`)

允许格式化时在字符串中换行.

### ColumnLimit(`unsigned`)

最大列数限制.

设置为`0`表示不限制最大列数. 此时clang-format依据语境和其他规则进行换行.

### CommentPragmas(`std::string`)

一个正则表达式, 包含该表达式的注释具有特殊含义, 不应该被分割或做其他改变.

```
// CommentPragmas: '^ FOOBAR pragma:'
// Will leave the following line unaffected
#include <vector> // FOOBAR pragma: keep
```

### CompactNamespaces(`bool`)

为`true`时, 连续的命名空间声明会被放在同一行; 否则每个命名空间声明会单独放在一行:

```
true:
namespace Foo { namespace Bar {
}}

false:
namespace Foo {
namespace Bar {
}
}
```

当不能放在同一行时, 超出的命名空间声明会自动换行:

```
namespace Foo { namespace Bar {
namespace Extra {
}}}
```

### ConstructorInitializerAllOnOneLineOrOnePerLine(`bool`)

当构造函数中的初始化器不能放在同一行时, 每个初始化器各占一行:

```
true:
FitsOnOneLine::Constructor()
: aaaaaaaaaaaaa(aaaaaaaaaaaaaa), aaaaaaaaaaaaa(aaaaaaaaaaaaaa) {}

DoesntFit::Constructor()
: aaaaaaaaaaaaa(aaaaaaaaaaaaaa),
aaaaaaaaaaaaa(aaaaaaaaaaaaaa),
aaaaaaaaaaaaa(aaaaaaaaaaaaaa) {}

false:
FitsOnOneLine::Constructor()
: aaaaaaaaaaaaa(aaaaaaaaaaaaaa), aaaaaaaaaaaaa(aaaaaaaaaaaaaa) {}

DoesntFit::Constructor()
: aaaaaaaaaaaaa(aaaaaaaaaaaaaa), aaaaaaaaaaaaa(aaaaaaaaaaaaaa),
aaaaaaaaaaaaa(aaaaaaaaaaaaaa) {}
```

### ConstructorInitializerIndentWidth(`unsigned`)

设置初始化器和继承列表的缩进字符数.

### ContinuationIndentWidth(`unsigned`)

连续行的缩进字符数.

```
ContinuationIndentWidth: 2

int i =         //  VeryVeryVeryVeryVeryLongComment
longFunction( // Again a long comment
arg);
```

### Cpp11BracedListStyle(`bool`)

为`true`时, 格式大括号列表为符合C++11的形式.

重要不同点:
- 列表中没有空格.
- 列表中间不会换行.
- 以连续行而不是块的形式缩进.

根本来说, C++11的列表的格式实际上更像函数调用. 当列表有名字时(比如类名或变量名), clang-format会按照函数调用时的括号一样格式化大括号; 当没有名字时, clang-format会假设存在一个0长度的名字.

```
true:                                  false:
vector<int> x{1, 2, 3, 4};     vs.     vector<int> x{ 1, 2, 3, 4 };
vector<T> x{{}, {}, {}, {}};           vector<T> x{ {}, {}, {}, {} };
f(MyMap[{composite, key}]);            f(MyMap[{ composite, key }]);
new int[3]{1, 2, 3};                   new int[3]{ 1, 2, 3 };
```

### DerivePointerAlignment(`bool`)

为`true`时, 会分析在已格式化的文件中`*`和`&`最常见的对齐方式. 指针和引用会按照该方式进行格式化. `PointerAlignment`只作为备选项.

### DisableFormat(`bool`)

完全禁止格式化.

### ExperimentalAutoDetectBinPacking(`bool`)

为`true`时, clang-format检测函数调用和定义是否需要每个参数各占一行.

每个函数都可以是bin-packed(译注: 应该是根据具体长度决定换行的情况), 每个参数各占一行或者不确定. 如果不确定, 如完全在一行, 但需要决定采取那种情况, clang-format会分析输入文件中的是否存在其他bin-packed的情况.

注: 这是个测试选项, 将来可能会被取消或重命名. 请不要使用该选项, 否则风险需自负.

### FixNamespaceComments(`bool`)

为`true`时, clang-format会自动添加并且修正存在但不合适的命名空间结束时的注释.

```
true:                                  false:
namespace a {                  vs.     namespace a {
foo();                                 foo();
} // namespace a;                      }
```

### ForEachMacros(`std::vector<std::string>`)

一个宏的向量, 内容应该被解释为foreach循环而不是函数调用.

这是期望的格式:

```
FOREACH(<variable-declaration>, ...)
<loop-body>
```

在`.clang-format`文件中, 可以以这种方式进行设置:

```
ForEachMacros: ['RANGES_FOR', 'FOREACH']
```

例如: BOOST_FOREACH.

### IncludeBlocks(`IncludeBlocksStyle`)

根据取值, 多个`#include`块可以按照整体或者多个类别进行排序.

可选取值:

- `IBS_Preserve`(配置文件中: `Preserve`) 分别排序每个`#include`块:

```
#include "b.h"               into      #include "b.h"

#include <lib/main.h>                  #include "a.h"
#include "a.h"                         #include <lib/main.h>
```

- `IBS_Merge`(配置文件中: `Merge`) 将多个`#include`块合并到一起并排序:

```
#include "b.h"               into      #include "a.h"
#include "b.h"
#include <lib/main.h>                  #include <lib/main.h>
#include "a.h"
```

- `IBS_Regroup`(配置文件中: `Regroup`) 先将多个`#include`块合并到一起进行排序. 然后根据类别优先级进行分组. 参考`IncludeCategories`:

```
#include "b.h"               into      #include "a.h"
#include "b.h"
#include <lib/main.h>
#include "a.h"                         #include <lib/main.h>
```

### IncludeCategories(`std::vector<IncludeCategory>`

一组用于管理`#includes`的表示不同`#include`分组规则的正则表达式.

支持**POSIX扩展**正则表达式.

这些表达式按顺序与include(包括""和<>)的文件名进行匹配. 每个文件按照第一个匹配上的正则表达式分配优先级, 所有的`#includes`先按照分组优先级升序排序, 然后每组中再按照字母顺序排序.

若没有匹配的正则表达式, 文件会被分配到INT_MAX组. 一个源文件的主包含项自动被分配到第0组, 所以通常会被排在`#includes`中的最前面. 在任何情况下, 都可以通过设置优先级为负值的分组来强制将某些包含项放在最前面.

在`.clang-format`文件中以如下方式进行配置:

```
IncludeCategories:
- Regex:           '^"(llvm|llvm-c|clang|clang-c)/'
Priority:        2
- Regex:           '^(<|"(gtest|gmock|isl|json)/)'
Priority:        3
- Regex:           '<[[:alnum:].]+>'
Priority:        4
- Regex:           '.*'
Priority:        1
```

### IncludeIsMainRegex(`std::string`)

指定一个描述合法后缀的正则表达式, 表示文件与主包含项的映射关系.

当判断一个`#include`是否为主包含项时(见上文, 是否被分配到第0组), 使用该表达式与头文件名的主题进行匹配. 进行部分匹配, 其中:
- ""表示任意后缀
- "$"表示没有后缀

例如, 设置为"(\_test)?$"时, "a.h"被认为是"a.cc"和"a\_test.cc"的主包含项.

### IndentCaseLabels('bool')

将switch语句中的case标签缩进一层.

为`false`时, case标签的缩进与swicth语句在同一层. 执行的表达式始终比case标签多缩进一层:

```
false:                                 true:
switch (fool) {                vs.     switch (fool) {
case 1:                                  case 1:
bar();                                   bar();
break;                                   break;
default:                                 default:
plop();                                  plop();
}                                      }
```

### IndentPPDirectives(`PPDirectiveIndentStyle`)

设置预编译选项的缩进格式.

可选取值:

- `PPDIS_None`(配置文件中: `None`) 不缩进:

```
#if FOO
#if BAR
#include <foo>
#endif
#endif
```

- `PPDIS_AfterHash`(配置文件中: `AfterHash`) 在井号之后缩进:

```
#if FOO
#  if BAR
#    include <foo>
#  endif
#endif
```

### IndentWidth(`unsigned`)

设置缩进字符数.

```
IndentWidth: 3

void f() {
someFunction();
if (true, false) {
f();
}
}
```

### IndentWrappedFunctionNames(`bool`)

当函数的定义或声明在返回类型之后产生自动换行时进行缩进.

```
true:
LoooooooooooooooooooooooooooooooooooooooongReturnType
LoooooooooooooooooooooooooooooooongFunctionDeclaration();

false:
LoooooooooooooooooooooooooooooooooooooooongReturnType
LoooooooooooooooooooooooooooooooongFunctionDeclaration();
```

### JavaScriptQuotes(`JavaScriptQuoteStyle`)

设置JavaScript字符串引号的格式.

可选取值:

- `JSQS_Leave`(配置文件中: `Leave`) 保持原状.

```
string1 = "foo";
string2 = 'bar';
```

- `JSQS_Single`(配置文件中: `Single`) 始终为单引号.

```
string1 = 'foo';
string2 = 'bar';
```

- `JSQS_Double`(配置文件中: `Double`) 始终为双引号.

```
string1 = "foo";
string2 = "bar";
```

### JavaScriptWrapImports(`bool`)

是否对JavaScript的import/export自动换行.

```
true:
import {
    VeryLongImportsAreAnnoying,
    VeryLongImportsAreAnnoying,
    VeryLongImportsAreAnnoying,
} from 'some/module.js'

false:
import {VeryLongImportsAreAnnoying, VeryLongImportsAreAnnoying, VeryLongImportsAreAnnoying,} from "some/module.js"
```

### KeepEmptyLinesAtTheStartOfBlocks(`bool`)

为`true`时, 代码段开始的空行会被保留.

```
true:                                  false:
if (foo) {                     vs.     if (foo) {
                                         bar();
  bar();                               }
}
```

### Language(`LanguageKind`)

格式配置项作用于哪种语言.

可选取值:

- `LK_None`(配置文件中: `None`) 不使用.
- `LK_Cpp`(配置文件中: `Cpp`) 用于C/C++.
- `LK_Java`(配置文件中: `Java`) 用于Java.
- `LK_JavaScript`(配置文件中: `JavaScript`) 用于JavaScript.
- `LK_ObjC`(配置文件中: `ObjC`) 用于Objective-C, Objective-C++.
- `LK_Proto`(配置文件中: `Proto`) 用于Protocol Buffers(https://developers.google.com/protocol-buffers/).
- `LK_TableGen`(配置文件中: `TableGen`) 用于TableGen编码.
- `LK_TextProto`(配置文件中: `TextProto`) 用于文本格式的Protocol Buffer信息.

### MacroBlockBegin(`std::string`)

一个正则表达式, 匹配的宏定义被认为是一个代码块的开始.

```
# With:
MacroBlockBegin: "^NS_MAP_BEGIN|\
NS_TABLE_HEAD$"
MacroBlockEnd: "^\
NS_MAP_END|\
NS_TABLE_.*_END$"

NS_MAP_BEGIN
  foo();
NS_MAP_END

NS_TABLE_HEAD
  bar();
NS_TABLE_FOO_END

# Without:
NS_MAP_BEGIN
foo();
NS_MAP_END

NS_TABLE_HEAD
bar();
NS_TABLE_FOO_END
```

### MacroBlockEnd(`std::string`)

一个正则表达式, 匹配的宏定义被认为是一个代码块的结束.

### MaxEmptyLinesToKeep(`unsigned`)

最大保留的空行数.

```
MaxEmptyLinesToKeep: 1         vs.     MaxEmptyLinesToKeep: 0
int f() {                              int f() {
  int = 1;                                 int i = 1;
                                           i = foo();
  i = foo();                               return i;
                                       }
  return i;
}
```

### NamespaceIndentation(`NamespaceIndentationKind`)

设置命名空间内部的缩进格式.

可选取值:

- `NI_None`(配置文件中: `None`) 命名空间内部不缩进.

```
namespace out {
int i;
namespace in {
int i;
}
}
```

- `NI_Inner`(配置文件中: `Inner`) 仅在内层的命名空间(嵌套在其他的命名空间中)内部缩进.

```
namespace out {
int i;
namespace in {
  int i;
}
}
```

- `NI_All`(配置文件中: `All`) 所有命名空间内部均缩进.

```
namespace out {
  int i;
  namespace in {
    int i;
  }
}
```

### ObjCBinPackProtocolList(`BinPackStyle`)

当超过`ColumnLimit`时, 控制Objective-C中的一致性列表(译注: 原文conformance list, 不清楚在ObjC中的对应术语)元素尽量减少占用的行数.

为`Auto`(默认值)时, 取决于`BinPackParameters`的值. 当`BinPackParameters`为`true`时, 当超过`ColumnLimit`时会尽量少的占用行数.

为`Always`时, 当超过`ColumnLimit`时总是会尽量少的占用行数.

为`Never`时, 当超过`ColumnLimit`时, 一致性列表中的元素每个各占一行.

```
Always (or Auto, if BinPackParameters=true):
@interface ccccccccccccc () <
    ccccccccccccc, ccccccccccccc,
    ccccccccccccc, ccccccccccccc> {
}

Never (or Auto, if BinPackParameters=false):
@interface ddddddddddddd () <
    ddddddddddddd,
    ddddddddddddd,
    ddddddddddddd,
    ddddddddddddd> {
}
```

可选取值: 

- `BPS_Auto`(配置文件中: `Auto`) 自动决定如何对参数执行bin-packing.
- `BPS_Always`(配置文件中: `Always`) 总是对参数bin-pack.
- `BPS_Never`(配置文件中: `Never`) 从不对参数bin-pack.

### ObjCBlockIndentWidth(`unsigned`)

设置ObjC代码块的缩进字符数.

```
ObjCBlockIndentWidth: 4

[operation setCompletionBlock:^{
    [self onOperationDone];
}];
```

### ObjCSpaceAfterProperty(`bool`)

在Objective-C中的`@property`后面添加空格. 即表示为`@property (readonly)`而不是`@property(readonly)`.

### ObjCSpaceBeforeProtocolList(`bool`)

在Objective-C中的协议列表(protoc list)之前添加空格. 即表示为`Foo <Protocol>`而不是` Foo<Protocol>`.

### PenaltyBreakAssignment(`unsigned`)

赋值运算符两侧换行的判定值(penalty).

译注: 这一系列的参数应该时当一行代码超过了`ColumnLimit`时, 通过这些设置的值来决定优先进行自动折行的位置. 似乎数值越小优先级越高. 请自行测试效果.

### PenaltyBreakBeforeFirstCallParameter(`unsigned`)

在函数调用的第一个入参之前换行的判定值.

### PenaltyBreakComment(`unsigned`)

在注释内部换行的判定值.

### PenaltyBreakFirstLessLess(`unsigned`)

在第一个`<<`之前换行的判定值.

### PenaltyBreakString(`unsigned`)

在字符串内部换行的判定值.

### PenaltyBreakTemplateDeclaration(`unsigned`)

在模板声明后换行的判定值.

### PenaltyExcessCharacter(`unsigned`)

当超过最大列数时, 在字符中间换行的判定值.

译注: 一般预设配置这个值都特别大, 应该需要保证这种自动换行绝不发生, 否则推测可能有直接打断变量名称的情况.

### PenaltyReturnTypeOnItsOwnLine(`unsigned`)

在函数返回类型后换行的判定值.

### PointerAlignment(`PointerAlignmentStyle`)

设置指针和引用的对齐方式.

可选取值: 

- `PAS_Left`(配置文件中: `Left`) 靠左对齐.

```
int* a;
```

- `PAS_Right`(配置文件中: `Right`) 靠右对齐.

```
int *a;
```

- `PAS_Middle`(配置文件中: `Middle`) 居中对齐.

```
int * a;
```

### RawStringFormats(`std::vector<RawStringFormat>`)

定义在原始字符串中检测支持语言的提示.

带有匹配的定界符或封闭函数名的字符串会根据`.clang-format`中指定的语言定义的设置重新格式化. 若`.clang-format`中没有指定语言的相关设置, 则会使用`BasedOnStyle`中的预设格式. 若`BasedOnStyle`也不存在, 则使用llvm风格. 字符串中的定界符优先级要高于封闭函数名的优先级.

必要时, 若指定了一个标准定界符, 基于该语言的其他定界符会被更新为该标准定界符.

每种语言应该最多指定一个检测标准, 并且每种定界符或封闭函数名不应出现在多个标准中.

采用如下方法在`.clang-format`进行配置:

```
RawStringFormats:
  - Language: TextProto
      Delimiters:
        - 'pb'
        - 'proto'
      EnclosingFunctions:
        - 'PARSE_TEXT_PROTO'
      BasedOnStyle: google
  - Language: Cpp
      Delimiters:
        - 'cc'
        - 'cpp'
      BasedOnStyle: llvm
      CanonicalDelimiter: 'cc'
```

### ReflowComments(`bool`)

为`true`时, clang-format会尝试重新排列注释.

```
false:
// veryVeryVeryVeryVeryVeryVeryVeryVeryVeryVeryLongComment with plenty of information
/* second veryVeryVeryVeryVeryVeryVeryVeryVeryVeryVeryLongComment with plenty of information */

true:
// veryVeryVeryVeryVeryVeryVeryVeryVeryVeryVeryLongComment with plenty of
// information
/* second veryVeryVeryVeryVeryVeryVeryVeryVeryVeryVeryLongComment with plenty of
 * information */
```

### SortIncludes(`bool`)

为`true`时, clang-format会将`#include`排序.

```
false:                                 true:
#include "b.h"                 vs.     #include "a.h"
#include "a.h"                         #include "b.h"
```

### SortUsingDeclarations(`bool`)

为`true`时, clang-format会将using声明排序.

using的顺序确定方式: 将字符串以"::"为分界符分割, 并且丢弃初始值为空的字符串. 每个列表中的最后一个元素为一个非命名空间的名称; 其他的字符串都是命名空间名称. 将这些名称列表按照字母顺序排序, 其中非命名空间名称优先于命名空间名称, 并且在这些分组中不区分大小写.

```
false:                                 true:
using std::cout;               vs.     using std::cin;
using std::cin;                        using std::cout;
```

### SpaceAfterCStyleCast(`bool`)

为`true`时, C格式的类型转换符后会插入一个空格.

```
true:                                  false:
(int) i;                       vs.     (int)i;
```

### SpaceAfterTemplateKeyword(`bool`)

为`true`时, `template`关键字后会插入一个空格.

```
true:                                  false:
template <int> void foo();     vs.     template<int> void foo();
```

### SpaceBeforeAssignmentOperators(`bool`)

为`false`时, 赋值运算符之前(译注: 看效果是两侧)的空格会被移除.

```
true:                                  false:
int a = 5;                     vs.     int a=5;
a += 42                                a+=42;
```

### SpaceBeforeCpp11BracedList(`bool`)

为`false`时, 表示继承的冒号之前的空格会被移除.

```
true:                                  false:
class Foo : Bar {}             vs.     class Foo: Bar {}
```

### SpaceBeforeParens(`SpaceBeforeParensOptions`)

定义何时在左括号之前插入空格.

可选取值:

- `SBPO_Never`(配置文件中: `Never`) 从不在左括号之前插入空格.

```
void f() {
  if(true) {
    f();
  }
}
```

- `SBPO_ControlStatements`(配置文件中: `ControlStatements`) 仅在流控语句(for/if/while...)中的左括号之前插入空格.

```
void f() {
  if (true) {
    f();
  }
}
```

- `SBPO_Always`(配置文件中: `Always`) 在不违背语法规则(如类似函数的宏定义中)或其他配置规则(一元运算符, 连续的左括号等)的情况下, 始终插入空格.

```
void f () {
  if (true) {
    f ();
  }
}
```

### SpaceBeforeRangeBasedForLoopColon(`bool`)

为`false`时, 会移除表示循环范围的冒号之前的空格.

```
true:                                  false:
for (auto v : values) {}       vs.     for(auto v: values) {}
```

译注: 从描述看重点应该在于`:`之前的空格, for之后的空格具体有什么影响暂不确定.

### SpaceInEmptyParentheses(`bool`)

为`true`时, `()`之中可能会插入空格.

```
true:                                false:
void f( ) {                    vs.   void f() {
  int x[] = {foo( ), bar( )};          int x[] = {foo(), bar()};
  if (true) {                          if (true) {
    f( );                                f();
  }                                    }
}                                    }
```

### SpacesBeforeTrailingComments(`unsigned`)

设置行尾注释之前的空格数(`//`).

由于块注释(`/\*`)通常有不同的使用方式和几个特殊场景, 该设置不对其造成影响.

```
SpacesBeforeTrailingComments: 3
void f() {
  if (true) {   // foo1
    f();        // bar
  }             // foo
}
```

### SpacesInAngles(`bool`)

为`true`时, 会在表示模板参数列表的`<`之后和`>`之前插入空格.

```
true:                                  false:
static_cast< int >(arg);       vs.     static_cast<int>(arg);
std::function< void(int) > fct;        std::function<void(int)> fct;
```

### SpacesInCStyleCastParentheses(`bool`)

为`true`时, C风格的类型转换符中间会插入空格.

```
true:                                  false:
x = ( int32 )y                 vs.     x = (int32)y
```

### SpacesInContainerLiterals(`bool`)

为`true`时, 在容器包含的内容(如, ObjC和JavaScript数组, 及字典的内容)中插入空格.

```
true:                                  false:
var arr = [ 1, 2, 3 ];         vs.     var arr = [1, 2, 3];
f({a : 1, b : 2, c : 3});              f({a: 1, b: 2, c: 3});
```

### SpacesInParentheses(`bool`)

为`true`时, 会在`(`之后和`)`之前插入空格.

```
true:                                  false:
t f( Deleted & ) & = delete;   vs.     t f(Deleted &) & = delete;
```

### SpacesInSquareBrackets(`bool`)

为`true`时, 会在`[`之后和`]`之前插入空格. 该选项不影响匿名函数和不定长数组.

```
true:                                  false:
int a[ 5 ];                    vs.     int a[5];
std::unique_ptr<int[]> foo() {} // Won't be affected
```

### Standard(`LanguageStandard`)

设置语法标准的兼容格式, 如在`LS_Cpp03`中使用`A<A<int> >`而不是`A<A<int>>`.

可选取值:

- `LS_Cpp03`(配置文件中: `Cpp03`) 使用C++03兼容语法.
- `LS_Cpp11`(配置文件中: `Cpp11`) 使用C++11, C++14和C++1z的特性(如, 使用`A<A<int>>`而不是`A<A<int> >`).
- `LS_Auto`(配置文件中: `Auto`) 根据输入文件自动检测.

### TabWidth(`unsigned`)

制表符的对齐宽度.

### UseTab(`UseTabStyle`)

输出文件中使用制表符的方式.

可选取值: 

- `UT_Never`(配置文件中: `Never`) 从不使用制表符.
- `UT_ForIndentation`(配置文件中: `ForIndentation`) 仅在缩进时使用制表符.
- `UT_ForContinuationAndIndentation`(配置文件中: `ForContinuationAndIndentation`) 仅在缩进延续的行时使用制表符.
- `UT_Always`(配置文件中: `Always`) 当需要插入超越下个制表符停止位置的空格时, 总是使用制表符.

## 添加额外的配置项

每条额外的配置项会使clang-format项目付出更多代价. 其中一些影响clang-format本身的开发工作, 因为需要确保任何配置项组合都能正常工作, 并且新的特性在任何情况下都不会破坏已有的配置项. 最终用户也会因此付出额外的代价, 配置项变得越来越难以发现, 并且用户必须考虑他们原本不关心的选项.

clang-format的主要目的是尽可能将有限格式的格式化工作做好, 而不是漫无目的的支持所有的语言格式化. 当然, clang-format确实想要支持所有主要的工程, 因此为增加新配置项设立了以下规则. 每条新的配置项必须:

- 使用在足够庞大的项目中(拥有几十个开发者).
- 拥有可公开访问的代码风格规范手册.
- 拥有愿意开发和维护补丁包的开发者.

## 示例

与Linux Kernel风格相近的配置:

```
BasedOnStyle: LLVM
IndentWidth: 8
UseTab: Always
BreakBeforeBraces: Linux
AllowShortIfStatementsOnASingleLine: false
IndentCaseLabels: false
```

输出结果(假设这里的缩进都是制表符):

```
void test()
{
        switch (x) {
        case 0:
        case 1:
                do_something();
                break;
        case 2:
                do_something_else();
                break;
        default:
                break;
        }
        if (condition)
                do_something_completely_different();

        if (x == y) {
                q();
        } else if (x > y) {
                w();
        } else {
                r();
        }
}
```

与Visual Studio风格相近的配置:

```
UseTab: Never
IndentWidth: 4
BreakBeforeBraces: Allman
AllowShortIfStatementsOnASingleLine: false
IndentCaseLabels: false
ColumnLimit: 0
```

输出结果:

```
void test()
{
    switch (suffix)
    {
    case 0:
    case 1:
        do_something();
        break;
    case 2:
        do_something_else();
        break;
    default:
        break;
    }
    if (condition)
        do_somthing_completely_different();

    if (x == y)
    {
        q();
    }
    else if (x > y)
    {
        w();
    }
    else
    {
        r();
    }
}
```
