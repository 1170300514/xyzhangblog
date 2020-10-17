---
title: Block使用总结
date: 2020-10-17 20:54:03
tags: 
    - ProtoBuf
    - 序列化
	- 反序列化
---

## 0x00 序列化

**序列化是指将数据结构或者对象的状态转换成可存储（文件/内存）、可传输的格式** 反之从存储或传输形式转换为数据结构或对象的形式成为反序列化

常见的序列化格式有：XML、JSON、YAML、plist...

## 0x01 Protocol Buffers简介

> ProtoBuf：google推出的语言中立，平台无关，可扩展的序列化数据的格式，可用于通信协议，数据存储等；相比XML更小巧、快速、简单。一旦定义要处理的数据的数据结构后，就可以利用ProtoBuf的代码生成工具生成相关代码。只需使用Protobuf对数据结构进行一次描述，就可以利用不同语言或从不同数据流中对结构化数据进行读写。

Protocol buffers适合做数据存储或RPC数据交换格式。可用于通讯协议、数据存储等领域的语言无关、平台无关、可扩展的序列化结构数据格式。Protocol buffers包含序列化格式的定义、对应语言的库和IDL编译器，正常情况下需要定义proto文件，然后使用IDL编译器编译成需要的语言。

**protocol buffers 诞生之初是为了解决服务器端新旧协议(高低版本)兼容性问题，名字也很体贴，“协议缓冲区”。只不过后期慢慢发展成用于传输数据**。

## 0x02 Protocol Buffers的特点及用法

### 特点：

- 便捷引入新字段，且无需了解所有字段服务器即可进行数据的解析和传递
- 数据格式更具自我描述性
- 序列化与反序列化代码可以自动生成，避免手动解析

### 用法：

proto中所有结构化数据都被称为`message`

```proto
syntax = "proto3";

message SearchRequest {
  string query = 1;
  int32 page_number = 2;
  int32 result_per_page = 3;
}
```

### 1. 分配字段编号

每个消息定义中的每个字段都有**唯一的编号**。这些字段编号用于标识消息二进制格式中的字段，并且在使用消息类型后不应更改。请注意，范围 1 到 15 中的字段编号需要一个字节进行编码，包括字段编号和字段类型。范围 16 至 2047 中的字段编号需要两个字节。所以你应该保留数字 1 到 15 作为非常频繁出现的消息元素。请记住为将来可能添加的频繁出现的元素留出一些空间。

可以指定的最小字段编号为1，最大字段编号为2^29^-1 或 536,870,911。也不能使用数字 19000 到 19999（FieldDescriptor :: kFirstReservedNumber 到 FieldDescriptor :: kLastReservedNumber），因为它们是为 Protocol Buffers实现保留的。

如果在 .proto 中使用这些保留数字中的一个，Protocol Buffers 编译的时候会报错。

同样，您不能使用任何以前 Protocol Buffers 保留的一些字段号码。保留字段是什么，下一节详细说明。

### 2. 保留字段

如果您通过完全删除某个字段或将其注释掉来更新消息类型，那么未来的用户可以在对该类型进行自己的更新时重新使用该字段号。如果稍后加载到了的旧版本 `.proto` 文件，则会导致服务器出现严重问题，例如数据混乱，隐私错误等等。确保这种情况不会发生的一种方法是指定删除字段的字段编号（或名称，这也可能会导致 JSON 序列化问题）为 `reserved`。如果将来的任何用户试图使用这些字段标识符，Protocol Buffers 编译器将会报错。

```proto
message Foo {
  reserved 2, 15, 9 to 11;
  reserved "foo", "bar";
}
```

**注意，不能在同一个 `reserved` 语句中混合字段名称和字段编号**。如有需要需要像上面这个例子这样写。

### 3. 默认字段规则

- 字段名不能重复，必须唯一。
- repeated 字段：可以在一个 message 中重复任何数字多次(包括 0 )，不过这些重复值的顺序被保留。

## 0x03 Protocol Buffers的优缺点

### 优点：

- 简单
- 数据体积小
- 反序列化速度快
- 自动化生成易于编码方式使用的数据访问类

#### 举个例子：

如果要编码一个用户的名字和 email 信息，用 XML 的方式如下：

```xml
  <person>
    <name>John Doe</name>
    <email>jdoe@example.com</email>
  </person>
```

相同需求，如果换成 protocol buffers 来实现，定义文件如下：

```c
# Textual representation of a protocol buffer.
# This is *not* the binary format used on the wire.
person {
  name: "John Doe"
  email: "jdoe@example.com"
}
```

protocol buffers 通过编码以后，以二进制的方式进行数据传输，最多只需要 28 bytes 空间和 100-200 ns 的反序列化时间。但是 XML 则至少需要 69 bytes 空间（经过压缩以后，去掉所有空格）和 5000-10000 的反序列化时间。

上面说的是性能方面的优势。接下来说说编码方面的优势。

protocol buffers 自带代码生成工具，可以生成友好的数据访问存储接口。从而开发人员使用它来编码更加方便。例如上面的例子，如果用 C++ 的方式去读取用户的名字和 email，直接调用对应的 get 方法即可（所有属性的 get 和 set 方法的代码都自动生成好了，只需要调用即可）

```c
  cout << "Name: " << person.name() << endl;
  cout << "E-mail: " << person.email() << endl;
```

而 XML 读取数据会麻烦一些：

```xml
  cout << "Name: "
       << person.getElementsByTagName("name")->item(0)->innerText()
       << endl;
  cout << "E-mail: "
       << person.getElementsByTagName("email")->item(0)->innerText()
       << endl;
```

Protobuf 语义更清晰，无需类似 XML 解析器的东西（因为 Protobuf 编译器会将 .proto 文件编译生成对应的数据访问类以对 Protobuf 数据进行序列化、反序列化操作）。

使用 Protobuf 无需学习复杂的文档对象模型，Protobuf 的编程模式比较友好，简单易学，同时它拥有良好的文档和示例，对于喜欢简单事物的人们而言，Protobuf 比其他的技术更加有吸引力。

protocol buffers 最后一个非常棒的特性是，即“向后”兼容性好，人们不必破坏已部署的、依靠“老”数据格式的程序就可以对数据结构进行升级。这样您的程序就可以不必担心因为消息结构的改变而造成的大规模的代码重构或者迁移的问题。因为添加新的消息中的 field 并不会引起已经发布的程序的任何改变(因为存储方式本来就是无序的，k-v 形式)。

当然 protocol buffers 也并不是完美的，在使用上存在一些局限性。

由于文本并不适合用来描述数据结构，所以 Protobuf 也不适合用来对基于文本的标记文档（如 HTML）建模。另外，由于 XML 具有某种程度上的自解释性，它可以被人直接读取编辑，在这一点上 Protobuf 不行，它以二进制的方式存储，除非你有 `.proto` 定义，否则你没法直接读出 Protobuf 的任何内容。



## 0x04 参考资料：

pb的编码原理、更详细的用法分析：https://halfrost.com/protobuf_encode/

pb的性能分析参考：https://halfrost.com/protobuf_decode/#toc-13