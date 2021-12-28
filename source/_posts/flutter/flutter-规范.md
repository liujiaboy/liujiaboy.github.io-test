---
title: flutter-规范
date: 2021-12-27 10:04:25
tags:
    - flutter
categories:
    - flutter
---

## 核心原则
代码应该简洁易懂，逻辑清晰；
代码应优先保证正确性、可用性；
在保证程序可用的情况下，代码应该具备可扩展性，易修改，而不是需求有一点改动代码就需要大动干戈；

## 禁止使用print直接提交到发版分支，使用debugPrint替换。

```
void tryCatch(Function f) {
  try {
    f?.call();
  } catch (e, stack) {
    debugPrint('$e');
    debugPrint('$stack');
  }
}
```

##  字符串

### 两个常量字符串（不是变量，是放在引号中的字符串），你不需要使用 + 来连接它们。
推荐的写法

```
print(
    'ERROR: Parts of the spaceship are on fire. Other '
    'parts are overrun by martians. Unclear which are which.');
```

不推荐的写法
```
print('ERROR: Parts of the spaceship are on fire. Other ' +
    'parts are overrun by martians. Unclear which are which.');
```

### 优先使用插值来组合字符串和值。
如果您之前是用其他语言做开发的，那么您习惯使用+的长链来构建文字和其他值的字符串。 这在Dart中有效，但使用插值总是更清晰，更简短：

推荐写法
```
'Hello, $name! You are ${year - birth} years old.';
```

不推荐的写法：

```
'Hello, ' + name + '! You are ' + (year - birth).toString() + ' y...';
```

### 不要在字符串中使用不必要的大括号
当表达式的值可以为真、假或null，并且您需要将结果传递给不接受null的对象时，此规则适用。一个常见的情况是一个判断空值的方法调用被用作条件:

推荐的写法
```
'Hi, $name!'
"Wear your wildest $decade's outfit."
//标识符后面有紧跟着的字母了 加上大括号用以区分
'Wear your wildest ${decade}s outfit.'
```

不推荐的写法

```
'Hi, ${name}!'
"Wear your wildest ${decade}'s outfit."
```

### 使用? ?将空值转换为布尔值。

不推荐的写法
```
if (optionalThing?.isEnabled) {
  print("Have enabled thing.");
}
```

如果optionalThing为空，此代码将抛出异常。（if只支持判断bool值，不支持null）要解决这个问题，您需要将null值“转换”为true或false。虽然您可以使用==来完成此操作，但我们建议使用?? :

推荐的写法

```
//如果你想要optionalThing是空值时返回false
optionalThing?.isEnabled ?? false;
//如果你想要optionalThing是空值时返回true
optionalThing?.isEnabled ?? true;
```

不推荐的写法
```
// 如果你想要optionalThing是空值时返回false
optionalThing?.isEnabled == true;
// 如果你想要optionalThing是空值时返回true
optionalThing?.isEnabled != false;
```

## 集合
### 尽可能的使用集合字面量。
两种方式来构造一个空的可变 list ： [] 和 List() 。 同样，有三种方式来构造一个空的Map map：{}， Map()， 和 LinkedHashMap() 。 如果想创建一个固定不变的 list 或者其他自定义集合类型，这种情况下你需要使用构造函数。 否则，使用字面量语法更加优雅。 核心库中暴露这些构造函数易于扩展，但是通常在 Dart 代码中并不使用构造函数。

推荐的写法
```
var points = [];
var addresses = {};
```

不推荐的写法
```
var points = List();
var addresses = Map();
```

### 如果需要的话，你可以提供一个泛型

推荐的写法
```
var points = <Point>[];
var addresses = <String, Address>{};
```

不推荐的写法
```
var points = List<Point>();
var addresses = Map<String, Address>();
```

注意，对于集合类的 命名 构造函数则不适用上面的规则。 List.from()、 Map.fromIterable() 都有其使用场景。 如果需要一个固定长度的结合，使用 List() 来创建一个固定长度的 list 也是合理的。

### 不要使用 .length 来判断一个集合是否为空。
通过调用 .length 来判断集合是否包含内容是非常低效的。相反，Dart 提供了更加高效率和易用的 getter 函数：.isEmpty 和.isNotEmpty。 使用这些函数并不需要对结果再次取非(list.length ! =0)

推荐的写法
```
if (lunchBox.isEmpty) return 'so hungry...';
if (words.isNotEmpty) return words.join(' ');
```

不推荐的写法
```
if (lunchBox.length == 0) return 'so hungry...';
if (!words.isEmpty) return words.join(' ');
```

### 不要使用 List.from() 除非想修改结果的类型。
给定一个可迭代的对象，有两种常见方式来生成一个包含相同元素的 list：

```
var copy1 = iterable.toList();
var copy2 = List.from(iterable);
```
推荐的写法
明显的区别是前一个更短。 更重要的区别在于第一个保留了原始对象的类型参数：
```
// 创建一个 List<int>:
var iterable = [1, 2, 3];
 
// 输出 "List<int>":
print(iterable.toList().runtimeType);
```

不推荐的写法
```
// 创建一个 List<int>:
var iterable = [1, 2, 3];

// 输出 "List<dynamic>":
print(List.from(iterable).runtimeType);
```

### 如果你想要改变原始对象的类型参数，那么可以调用 List.from() ：

推荐的写法
```
var numbers = [1, 2.3, 4]; // List<num>.
numbers.removeAt(1); // 现在集合里只包含int型
var ints = List<int>.from(numbers);
```
但是如果你的目的只是复制可迭代对象并且保留元素原始类型， 或者并不在乎类型，那么请使用 toList() 。

## 参数
### 使用 = 来分隔参数名和参数默认值。
由于遗留原因，Dart 同时支持 : 和 = 作为参数名和默认值的分隔符。 为了与可选的位置参数保持一致，请使用 = 。

推荐的写法
```
void insert(Object item, {int at = 0}) { ... }
```

不推荐的写法
```
void insert(Object item, {int at: 0}) { ... }
```

## 成员
### 不要 为字段创建不必要的 getter 和 setter 方法
推荐的写法
```
class Box {
  var contents;
}
```

不推荐的写法
```
class Box {
  var _contents;
  get contents => _contents;
  set contents(value) {
    _contents = value;
  }
}
```

### 不要使用this. 在重定向命名函数和避免冲突的情况下除外
只有当局部变量和成员变量名字一样的时候，你才需要使用 this. 来访问成员变量。 只有两种情况需要使用 this. 。其中一种情况是要访问的局部变量和成员变量命名一样的时候：

推荐的写法
```
class Box {
  var value;

  void clear() {
    update(null);
  }

  void update(value) {
    this.value = value;
  }
}
```

不推荐的写法
```
class Box {
  var value;

  void clear() {
    this.update(null);
  }

  void update(value) {
    this.value = value;
   }
}
```

### 要尽可能的在定义变量的时候初始化变量值。
如果一个字段不依赖于构造函数中的参数， 则应该在定义的时候就初始化字段值。 这样可以减少需要的代码并可以确保在有多个构造函数的时候你不会忘记初始化该字段。

不推荐的写法
```
class Folder {
  final String name;
  final List<Document> contents;
 
  Folder(this.name) : contents = [];
  Folder.temp() : name = 'temporary'; // Oops! Forgot contents.
}
```

推荐的写法
```
class Folder {
  final String name;
  final List<Document> contents = [];
 
  Folder(this.name);
  Folder.temp() : name = 'temporary';
}
```

当然，对于变量取值依赖构造函数参数的情况以及不同的构造函数取值也不一样的情况， 则不适合本条规则。

## 构造函数
### 不要 使用 new
创建对象不要使用new

推荐的写法
```
Widget build(BuildContext context) {
  return Row(
    children: [
      RaisedButton(
        child: Text('Increment'),
      ),
      Text('Click!'),
    ],
  );
}
```

不推荐的写法
```
Widget build(BuildContext context) {
  return new Row(
    children: [
      new RaisedButton(
        child: new Text('Increment'),
      ),
      new Text('Click!'),
    ],
  );
}
```

### 要用 ; 来替代空的构造函数体 {}。
在 Dart 中，没有具体函数体的构造函数可以使用分号结尾。 （事实上，这是不可变构造函数的要求。）

推荐的写法
```
class Point {
  int x, y;
  Point(this.x, this.y);
}
```

不推荐的写法
```
class Point {
  int x, y;
  Point(this.x, this.y) {}
}
```

### 要尽可能的使用初始化形式。
不推荐的写法
```
class Point {
  num x, y;
  Point(num x, num y) {
    this.x = x;
    this.y = y;
  }
}
```

推荐的写法
```
class Point {
  num x, y,z;
  Point(this.x, this.y,{this.z});
}
```

这里的位于构造函数参数之前的 this. 语法被称之为初始化形式（initializing formal）。 有些情况下这无法使用这种形式。特别是，这种形式下在初始化列表中无法看到变量。 但是如果能使用该方式，就应该尽量使用。（如果使用命名参数）

## 函数返回值
### dart允许方法不写返回值类型，但是在编码过程中建议将返回值类型写清楚。
不推荐
```
foo() {}
```

推荐
```
void foo() {}
```

## 异步
### 推荐 使用 async/await 而不是直接使用底层的特性。
显式的异步代码是非常难以阅读和调试的， 即使使用很好的抽象（比如 future）也是如此。 这就是为何 Dart 提供了 async/await。 这样可以显著的提高代码的可读性并且让你可以在异步代码中使用语言提供的所有流程控制语句。

推荐的写法
```
Future<int> countActivePlayers(String teamName) async {
  try {
    var team = await downloadTeam(teamName);
    if (team == null) return 0;

    var players = await team.roster;
    return players.where((player) => player.isActive).length;
  } catch (e) {
    log.error(e);
    return 0;
  }
}
```

不推荐写法
```
Future<int> countActivePlayers(String teamName) {
  return downloadTeam(teamName).then((team) {
    if (team == null) return Future.value(0);

    return team.roster.then((players) {
      return players.where((player) => player.isActive).length;
    });
  }).catchError((e) {
    log.error(e);
    return 0;
  });
}
```

## 标识符

在 Dart 中标识符有三种类型。 • UpperCamelCase 每个单词的首字母都大写，包含第一个单词。 • lowerCamelCase 每个单词的首字母都大写，除了第一个单词， 第一个单词首字母小写，即使是缩略词。 • lowercase_with_underscores 只是用小写字母单词，即使是缩略词， 并且单词之间使用 _ 连接。

### 使用 UpperCamelCase 风格命名类型。
Classes（类名）、 enums（枚举类型）、 typedefs（类型定义）、 以及 type parameters（类型参数）应该把每个单词的首字母都大写（包含第一个单词）， 不使用分隔符。

```
class SliderMenu { ... }

class HttpRequest { ... }

typedef Predicate = bool Function<T>(T value);
```

### 要在库，包，文件夹，源文件中使用 lowercase_with_underscores 方式命名要用 lowercase_with_underscores 风格命名库和源文件名。
推荐的写法

```
library peg_parser.source_scanner;

import 'file_system.dart';
import 'slider_menu.dart';
```

不推荐的写法
```
library pegparser.SourceScanner;
import 'file-system.dart';
import 'SliderMenu.dart';
```

### 要使用 lowercase_with_underscores 风格命名导入的前缀
推荐的写法
```
import 'dart:math' as math;
import 'package:angular_components/angular_components'
    as angular_components;
import 'package:js/js.dart' as js;
```

不推荐的写法
```
import 'dart:math' as Math;
import 'package:angular_components/angular_components'
    as angularComponents;
import 'package:js/js.dart' as JS;
```

### 要 使用 lowerCamelCase 风格来命名其他的标识符。
类成员、顶级定义、变量、参数以及命名参数等 除了第一个单词，每个单词首字母都应大写，并且不使用分隔符。

推荐的写法
```
var item;

HttpRequest httpRequest;

void align(bool clearItems) {
  // ...
}
```

### 要把超过两个字母的首字母大写缩略词和缩写词当做普通单词。
首字母大写缩略词比较难阅读， 特别是多个缩略词连载一起的时候会引起歧义。 例如，一个以 HTTPSFTP 开头的名字， 没有办法判断它是指 HTTPS FTP 还是 HTTP SFTP 。 为了避免上面的情况，缩略词和缩写词要像普通单词一样首字母大写， 两个字母的单词除外。 （像 ID 和 Mr. 这样的双字母缩写词仍然像一般单词一样首字母大写。）

推荐的写法
```
HttpConnectionInfo
uiHandler
IOStream
HttpRequest
Id
DB
```

不推荐的写法
```
HTTPConnection
UiHandler
IoStream
HTTPRequest
ID
Db
```

• acronyms ：首字母缩略词，指取若干单词首字母组成一个新单词，如：HTTP = HyperText Transfer Protocol • abbreviations : 缩写词，指取某一单词的部分字母（或其他缩短单词的方式）代表整个单词，如：ID = identification


### 不要 使用前缀字母
推荐的写法
```
defaultTimeout
```

不推荐的写法
```
kDefaultTimeout
```

### 要 使用 googlestyle 格式化你的代码
格式化是一项繁琐的工作，尤其在重构过程中特别耗时。 庆幸的是，你不必担心。 使用Android studio默认的googlestyle。

### 要对所有流控制结构使用花括号。
这样可以避免 dangling else （else悬挂）的问题。

```
if (isWeekDay) {
  print('Bike to work!');
} else {
  print('Go dancing or read a book!');
}
```

这里有一个例外：一个没有 else 的 if 语句， 并且这个 if 语句以及它的执行体适合在一行中实现。 在这种情况下，如果您愿意，可以不用括号

```
if (arg == null) return defaultValue;
```

但是，如果执行体包含下一行，请使用大括号：

推荐的写法

```
if (overflowChars != other.overflowChars) {
  return overflowChars < other.overflowChars;
}
```

不推荐的写法
```
if (overflowChars != other.overflowChars)
  return overflowChars < other.overflowChars;
```