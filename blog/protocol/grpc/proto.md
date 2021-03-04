# Proto（协议缓冲区）
- [Proto介绍（协议缓冲区）](#proto%E4%BB%8B%E7%BB%8D%E5%8D%8F%E8%AE%AE%E7%BC%93%E5%86%B2%E5%8C%BA)
  - [简介](#%E7%AE%80%E4%BB%8B)
  - [定义消息体](#%E5%AE%9A%E4%B9%89%E6%B6%88%E6%81%AF%E4%BD%93)
    - [分配字段编号](#%E5%88%86%E9%85%8D%E5%AD%97%E6%AE%B5%E7%BC%96%E5%8F%B7)
    - [参数定义规则](#%E5%8F%82%E6%95%B0%E5%AE%9A%E4%B9%89%E8%A7%84%E5%88%99)
    - [保留字段](#%E4%BF%9D%E7%95%99%E5%AD%97%E6%AE%B5)
  - [数据类型](#%E6%95%B0%E6%8D%AE%E7%B1%BB%E5%9E%8B)
  - [默认值](#%E9%BB%98%E8%AE%A4%E5%80%BC)
  - [枚举](#%E6%9E%9A%E4%B8%BE)
  - [使用其他类型](#%E4%BD%BF%E7%94%A8%E5%85%B6%E4%BB%96%E7%B1%BB%E5%9E%8B)
  - [导入proto](#%E5%AF%BC%E5%85%A5proto)
  - [嵌套类型](#%E5%B5%8C%E5%A5%97%E7%B1%BB%E5%9E%8B)
  - [更新消息类型](#%E6%9B%B4%E6%96%B0%E6%B6%88%E6%81%AF%E7%B1%BB%E5%9E%8B)
  - [未知字段](#%E6%9C%AA%E7%9F%A5%E5%AD%97%E6%AE%B5)
  - [任何](#%E4%BB%BB%E4%BD%95)
  - [Oneof](#oneof)
    - [使用](#%E4%BD%BF%E7%94%A8)
    - [功能](#%E5%8A%9F%E8%83%BD)
    - [向后兼容问题](#%E5%90%91%E5%90%8E%E5%85%BC%E5%AE%B9%E9%97%AE%E9%A2%98)
  - [Map](#map)
  - [包](#%E5%8C%85)
  - [定义服务](#%E5%AE%9A%E4%B9%89%E6%9C%8D%E5%8A%A1)
  - [JSON对比](#json%E5%AF%B9%E6%AF%94)
  - [自定义选项](#%E8%87%AA%E5%AE%9A%E4%B9%89%E9%80%89%E9%A1%B9)
  - [资源与版本](#%E8%B5%84%E6%BA%90%E4%B8%8E%E7%89%88%E6%9C%AC)

## 简介
协议缓冲区是Google的与语言无关，与平台无关的可扩展机制，用于对结构化数据进行序列化–以XML为例，但更小，更快，更简单。您定义要一次构造数据的方式，然后可以使用生成的特殊源代码轻松地使用各种语言在各种数据流中写入和读取结构化数据。

协议缓冲区当前支持Java，Python，Objective-C和C ++生成的代码。使用我们新的proto3语言版本，您还可以使用Dart，Go，Ruby和C＃，并提供更多语言。

## 定义消息体
```
// 登陆请求消息体
message LoginRequest {
    string username = 1;
    string password = 2;
}
```
### 分配字段编号
消息定义中的每个字段都有一个唯一的编号。
这些数字用于标识消息二进制格式的字段，一旦使用了消息类型，就不应更改这些数字。
注意，范围为**1到15**的字段编号需要一个字节来编码，包括字段编号和字段的类型（您可以在Protocol Buffer Encoding中找到有关此内容的更多信息）。**16到2047之间的字段编号占用两个字节**。
因此，应该为非常频繁出现的消息元素保留字段编号1到15。记住要留出一些空间，以便将来可能会添加一些经常出现的元素。

您可以指定最小的场数是1，最大为2 ^29-1，或536870911。
您也**不能使用数字19000到19999**（FieldDescriptor::kFirstReservedNumber至FieldDescriptor::kLastReservedNumber），因为它们是为协议缓冲区实现保留的-如果您在中使用这些保留数之一，则协议缓冲区编译器会抱怨.proto。同样，您不能使用任何以前保留的字段号。

### 参数定义规则
为proto2数据类型前缀，proto3以简化
+ singular: 格式正确的 包含零个或一个此字段（但不能超过一个）。这是（proto3）语法的默认字段规则。
+ required: 格式正确的 必要参数, （proto2语法）
+ optional: 格式正确的 可选参数, （proto2语法）
+ repeated: 格式正确的 复用字段，可以重复任意次（包括零次）。重复值的顺序将保留
```
repeated int32 samples = 4 [packed=true];
```
在proto3中，repeated标量数字类型的字段packed默认情况下使用编码

### 保留字段

```
message Foo{
    reserved 2,15,9 to 11;
    reserved "foo","bar";
}
```
如果您通过完全删除字段或将其注释掉来更新消息类型，将来的用户可以在自己对类型进行更新时重用该字段号。
如果他们以后加载相同版本的旧版本，可能会导致严重的问题.proto，包括数据损坏，隐私错误等。
确保不会发生这种情况的一种方法是，将已删除字段的字段编号（和/或名称，也可能导致JSON序列化的问题）指定为reserved。
如果将来有任何用户尝试使用这些字段标识符，则协议缓冲区编译器会报错。
注意，您不能在同reserved一条语句中混用字段名称和字段编号。

## 数据类型
|.proto Type|       Notes	    | C++ Type	        |   Java Type	|   Python Type[2]	|   Go Type |
| --------- | ----------------- | ----              |       ----    |           ----    |    ----   |
|double	    | 浮点双精度	        |double	            |double         |float	            |*float64   |
|float      | 浮点单精度	        |float	            |float	        |float              |*float32   |
|int32	    | 使用可变长度编码。负数编码效率低下–如果您的字段可能具有负值，请改用sint32。|	int32|	int|	int|	*int32|
|int64	    | 使用可变长度编码。负数编码效率低下–如果您的字段可能具有负值，请改用sint64。|	int64|	long|	int/long[3]|	*int64|
|uint32	    | 可变长度编码        |uint32	            |int[1]	        |int/long[3]        |*uint32    |
|uint64	    | 可变长度编码        |uint64	            |long[1]	    |int/long[3]	    |*uint64    |
|sint32	    | 使用可变长度编码。有符号的int值。与常规int32相比，它们更有效地编码负数	|int32	|int	|int	|*int32|
|sint64	    | 使用可变长度编码。有符号的int值。与常规int64相比，它们更有效地编码负数	|int64	|long	|int/long[3]	|*int64|
|fixed32    | 始终为四个字节。如果值通常大于228，则比uint32更有效。	|uint32	|int[1]	|int/long[3]	|*uint32|
|fixed64    | 始终为八个字节。如果值通常大于256，则比uint64更有效。	|uint64	|long[1]|int/long[3]	|*uint64|
|sfixed32   | 始终为四个字节       |int32	            |int	        |int	            |*int32     |
|sfixed64   | 始终为八个字节	    |int64	            |long	        |int/long[3]	    |*int64     |  
|bool	    |	                |bool	            |boolean	    |bool	            |*bool      |
|string	    | 字符串必须始终包含UTF-8编码或7位ASCII文本	|string	|String	|unicode (Python 2) or str (Python 3)	|*string|
|bytes	    | 可以包含任意字节序列	|string	            |ByteString	    |bytes	            |[]byte     |


## 默认值
| 类型   | 默认值 |
| ---   | ----- |
|string |null   |
|byte   |空字节  |
|boolean|false  |
|int    |0      |
|enum   |第一个定义的枚举值， 0|
|message|取决语言|

## 枚举
```
message UserInfoRequest{
    string id = 1;
    string name = 2;
    enum Gender{
        MAN = 0;
        WOMAN = 1;
        NONE = 2;
    }
    Gender gender = 3;
}
```
+ 必须有一个零值，以便我们可以使用0作为数字默认值。
+ 零值必须是第一个元素，以便与proto2语义兼容，其中第一个枚举值始终是默认值。

```
message Request1{
    enum MyEnum{
        option allow_alias = true;
        YES = 0;
        NO = 1;
        ANY = 2;
    }
}

message Request2{
    enum MyEnumTest{
        YES = 0;
        NO = 1;
    }
}
```
将相同的值分配给不同的枚举常量来定义别名。
将allow_alias选项设置为true，否则协议别名会在找到别名时生成一条错误消息。

## 使用其他类型
```
message SearchResponse{
    repeated Response res = 1;
}

message Response{
    int32 code = 1;
    string msg = 2;
    repeated string data = 3;
}
```
您可以将其他消息类型用作字段类型。
例如，假设你想包括Response每个消息的SearchResponse消息-要做到这一点，你可以定义一个Response在同一个消息类型.proto，然后指定类型的字段Response中SearchResponse

## 导入proto
```
import "proto/user.proto"
// 虚拟位置映射，转发改变过的地址
import public "new.proto"
```
协议编译器使用-I/ --proto_path标志在协议编译器命令行上指定的一组目录中搜索导入的文件。
如果未给出任何标志，它将在调用编译器的目录中查找。
通常，应将--proto_path标志设置为项目的根，并对所有导入使用完全限定的名称。

## 嵌套类型
```
message BookReq{
    message Book{
        int32 id = 1;
        string name = 2;
    }
    repeated Book book = 1;
}

或

book.proto
message BookReq{    Level 0
    message Book{   Level 1
        int32 id = 1;
        string name = 2;
    }
    repeated Book book = 1;
}

request.proto

import "book.proto"
message Req{
    BookReq.Book book = 1;
}
```

## 更新消息类型
如果现有消息类型不再满足您的所有需求（例如，您希望消息格式具有一个额外的字段），但是您仍然希望使用以旧格式创建的代码，请不要担心！在不破坏任何现有代码的情况下更新消息类型非常简单。只要记住以下规则：

不要更改任何现有字段的字段编号。
如果添加新字段，则仍可以使用新生成的代码来解析使用“旧”消息格式通过代码序列化的任何消息。您应记住这些元素的默认值，以便新代码可以与旧代码生成的消息正确交互。同样，新代码创建的消息也可以由旧代码解析：旧的二进制文件在解析时只会忽略新字段。有关详细信息，请参见“ 未知字段”部分。
只要在更新后的消息类型中不再使用字段号，就可以删除字段。您可能想要重命名该字段，或者添加前缀“ OBSOLETE_”，或者将字段编号保留，以使您的将来的用户.proto不会意外重用该编号。
int32，uint32，int64，uint64，和bool都是兼容的-这意味着你可以在现场从这些类型到另一种改变不破坏forwards-或向后兼容。如果从电线中解析出一个不适合相应类型的数字，则将获得与在C ++中将该数字强制转换为该类型一样的效果（例如，如果将64位数字读取为int32，它将被截断为32位）。
sint32并且sint64彼此兼容，但与其他整数类型不兼容。
string并且bytes只要字节是有效的UTF-8即可兼容。
bytes如果字节包含消息的编码版本，则嵌入式消息与之兼容。
fixed32与兼容sfixed32，并fixed64用sfixed64。
对于string，bytes和消息字段，optional与兼容repeated。给定重复字段的序列化数据作为输入，如果期望该字段为optional原始类型字段，则期望该字段的客户端将使用最后一个输入值；如果是消息类型字段，则将合并所有输入元素。请注意，这不是一般的数值类型，包括布尔变量和枚举安全。重复的数字类型字段可以以打包格式序列化，当需要一个optional字段时，将无法正确解析该格式。
enum与兼容int32，uint32，int64，和uint64电线格式条款（请注意，如果他们不适合的值将被截断）。但是请注意，客户端代码在反序列化消息时可能会以不同的方式对待它们：例如，无法识别的proto3 enum类型将保留在消息中，但是在反序列化消息时如何表示这取决于语言。Int字段始终只是保留其值。
将单个值更改为新 值的成员oneof是安全且二进制兼容的。oneof如果您确定没有代码一次设置多个字段，那么将多个字段移动到新字段中可能是安全的。将任何字段移到现有字段中oneof都是不安全的

## 未知字段
未知字段是格式正确的协议缓冲区序列化数据，表示解析器无法识别的字段。
例如，当旧二进制文件使用新字段解析新二进制文件发送的数据时，这些新字段将成为旧二进制文件中的未知字段。

最初，proto3消息在解析过程中总是丢弃未知字段，但是在版本3.5中，我们重新引入了保留未知字段以匹配proto2行为的功能。
在版本3.5和更高版本中，未知字段将在解析期间保留并包含在序列化输出中。

## 任何
```
import "google/protobuf/any.proto";

message LoginResponse{
  int32 code = 1;
  string msg = 2;
  repeated google.protobuf.Any data = 3;
}
```

## Oneof

### 使用
```
message SampleMessage {
  oneof test_oneof {
    string name = 4;
    SubMessage sub_message = 9;
  }
}
```
将oneof字段添加到oneof定义。您可以添加任何类型的map字段，但字段和repeated字段除外。
oneof字段具有与常规字段相同的getter和setter。您还将获得一种特殊的方法来检查oneof中的哪个值（如果有）

### 功能
+ 设置oneof字段将自动清除oneof的所有其他成员。因此，如果您设置了多个字段，则只有您设置的最后一个字段仍具有值。
+ 如果解析器在线路上遇到同一个对象的多个成员，则在解析的消息中仅使用最后看到的成员。
+ 一个不能是repeated。
+ 反射API适用于其中一个字段。
+ 如果将oneof字段设置为默认值（例如将int32 oneof字段设置为0），则将设置该oneof字段的“大小写”，并且该值将在线路上序列化。

### 向后兼容问题
添加或删除字段之一时请多加注意。如果检查oneof的值返回`None/ NOT_SET`，则可能意味着尚未设置oneof或已将其设置为不同版本的oneof中的字段。由于无法知道导线上的未知字段是否是oneof的成员，因此无法分辨出差异。

**标签重用问题**

将字段移入或移出oneof：在对消息进行序列化和解析后，您可能会丢失一些信息（某些字段将被清除）。但是，您可以安全地将单个字段移动到新字段中，并且如果知道只设置了一个字段，则可以移动多个字段。
删除一个oneof字段并将其添加回：序列化和解析消息后，这可能会清除您当前设置的oneof字段。
拆分或合并其中一个：与移动常规字段有类似的问题。

## Map
```
快捷语法：
map<key_type, value_type> map_field = N;

示例：
map<string, Result> results = 3;
```
其中`key_type`可以是**任何整数或字符串类型**（因此，除浮点类型和以外的任何标量类型bytes）。
请注意，**enum无效key_type**。的value_type可以是任何类型的除另一Map。

+ Map字段不能为repeated。
+ Map值的线格式排序和Map迭代排序是不确定的，因此您不能依赖于Map项的特定顺序。
+ 为生成文本格式时.proto，Map会按键排序。数字键按数字排序。
+ 从导线解析或合并时，如果存在重复的映射键，则使用最后看到的键。从文本格式解析Map时，如果键重复，则解析可能会失败。
+ 如果为映射字段提供键但没有值，则序列化字段时的行为取决于语言。在C ++，Java和Python中，类型的默认值是序列化的，而在其他语言中，则没有序列化的值。

向后兼并
```
message ResultMap{
    key_type key = 1;
    value_type value = 2;
}

repeated ResultMap map = N;
```
## 包
防止协议消息类型冲突，可以定义包
```
package com.test;
message TestFoo{...}

message Test{
    com.test.TestFoo foo = 1;
}
```

## 定义服务
```
service BookService{
    rpc Search(BookReq) retruns (BookRes);
}
```
如果要将消息类型与RPC（远程过程调用）系统一起使用，则可以在.proto文件中定义RPC服务接口，并且协议缓冲区编译器将以您选择的语言生成服务接口代码和存根。

## JSON对比
![json](http://file.uzykj.com/proto_json1.png)
![json](http://file.uzykj.com/proto_json2.png)

## 自定义选项
+ java_package 用于生成的Java类的包
```
option java_package = "com.test.book";
```
+ java_multiple_files 导致在包级别定义顶级消息，枚举和服务，而不是在以.proto文件命名的外部类中定义。
```
option java_multiple_files = true;
```
+ java_outer_classname 生成的最外层Java类的类名（以及文件名）
```
option java_outer_classname = "Ponycopter";
```
+ optimize_for SPEED, CODE_SIZE, LITE_RUNTIME
```
option optimize_for = SPEED;
```
+ SPEED（默认）：协议缓冲区编译器将生成代码，用于对消息类型进行序列化，解析和执行其他常见操作。
    此代码已高度优化。
    
+ CODE_SIZE：协议缓冲区编译器将生成最少的类，并将依赖于基于反射的共享代码来实现序列化，解析和其他各种操作。
    因此，生成的代码将比使用的代码小得多SPEED，但操作会更慢。
    类仍将实现与SPEED模式下完全相同的公共API 。
    此模式在包含大量.proto文件的应用程序中最有用，并且不需要所有文件都可以使人眼花fast乱。
    
+ LITE_RUNTIME：协议缓冲区编译器将生成仅依赖于“精简版”运行时库（libprotobuf-lite而不是libprotobuf）的类。
    精简版运行时比完整库要小得多（大约小一个数量级），但省略了某些功能，例如描述符和反射。
    这对于在受限平台（例如手机）上运行的应用程序特别有用。
    编译器仍将像在SPEED模式下一样快速生成所有方法的实现。
    生成的类将仅以MessageLite每种语言实现该接口，该语言仅提供完整Message接口的方法的子集。
    
+ cc_enable_arenas 启用C ++生成代码

+ objc_class_prefix 设置Objective-C类的前缀，该前缀将附加到所有Objective-C生成的类以及此.proto枚举。没有默认值。您应该使用Apple推荐的 3-5个大写字符之间的前缀。请注意，Apple保留所有2个字母前缀。

+ deprecated 如果设置为true，则表明该字段已弃用，并且不应被新代码使用。在大多数语言中，这没有实际效果。在Java中，这成为@Deprecated注释。
    将来，其他特定于语言的代码生成器可能会在字段的访问器上生成弃用注释，这反过来将导致在编译尝试使用该字段的代码时发出警告。
    如果该字段未被任何人使用，并且您想阻止新用户使用它，请考虑使用保留语句替换字段声明。
```
int32 old_field = 6 [deprecated = true];
```

## 资源与版本

+ [官网文档](https://developers.google.com/protocol-buffers)

+ 版本
    + [proto2](https://developers.google.com/protocol-buffers/docs/proto)
    + [proto3](https://developers.google.com/protocol-buffers/docs/proto3) (demo使用)


---
收录时间: 2020/08/19

<Vssue :title="$title" />
