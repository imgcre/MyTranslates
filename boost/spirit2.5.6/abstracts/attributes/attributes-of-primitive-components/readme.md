## 原生组件属性
在`Spirit`中的解析器和生成器都是完全属性化的。`Spirit.Qi`的解析器总是公开一个特定于其类型的属性。这被称作*合成属性*，它从一个成功的匹配中返回，并且代表了匹配的输入序列。比如，数值解析器，比如`int_`或者`double_`，返回从匹配的输入序列中转换的`int`或者`double`值。其他的原生解析器组件有其他直观的属性类型，比如`int_`有`int`，或者`ascii::char_`有`char`。对于原生解析器有C++可转型规则：只要解析器的属性类型能够转换成所提供的类型，你能使用任何C++类型来得到一个被解析的值。下面的例子展示了合成解析器属性(`int`值)是如何通过调用API函数`qi::parse`抽出的。  
``` c++
int value = 0;
std::string str("123");
std::string::iterator strbegin = str.begin();
qi::parse(strbegin, str.end(), int_, value);   //value == 123
```
生成器的属性类型被定义为为了产生输出此生成器能够消费的数据类型。`Spirit.Karma`的生成器总是期望一个特定于他们类型的属性。这被称作*消费属性*，它期望被生成器通过。*消费属性*大多数情况下是设计生成器输出的值。对于原生生成器可应用普通的C++的可转型规则。任何可转型为原生生成器属性类型的数据类型能够被用来提供待生成数据。我们在下面提供提供一个和上例相似的例子，这次被消费的`int_`生成器的属性（`int`值）使用API函数`Karma::generate`经过。  
``` c++
int value = 123;
std::string str;
std::back_insert_iterator<std::string> out(str);
karma::generate(out, int_, value);   //str == "123"
```
其他的原生操作符组件有其他的直观属性类型，和相应的解析器组件类似。比如`ascii::char_`生成器有一个`char`作为消费属性。为了康完整可用的可用解析器和生成器的原生操作符和他们的属性类型，可以看`Qi解析器`和`Karma生成器`章节。(*TODO*)  