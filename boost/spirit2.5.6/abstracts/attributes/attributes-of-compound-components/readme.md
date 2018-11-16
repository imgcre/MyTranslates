## 复合组件属性
`Spirit.Qi`和`Spirit.Karma`为所有组合解析器和生成器实现了良定义的属性值传播规则，比如序列，选择，星号等等。序列主要的属性传播规则，比如：  

| 库 | 序列属性传播规则 |
| --- | --- |
| Qi | a: A, b: B --> (a >> b): tuple<A, B> |
| Karma | a: A, b: B --> (a << b): tuple<A, B> |

读作：给定解析器（或生成器）`a`和`b`，`A`是`a`的属性类型，`B`是`b`的属性类型，那么`a >> b`（`a << b`）的属性类型为`tuple<A, B>`。  
> **注意：**记号`tuple<A, B>`是任何能够容纳类型A和B的融合序列的占位符表达式，比如`boost::fusion::tuple<A, B>`或者`std::pair<A, B>`（详见[Boost.Fusion](https://www.boost.org/doc/libs/1_68_0/libs/fusion/doc/html/index.html)）  

如你所见，为了得到兼容于组合表达式属性类型的类型，必须要：  
- 每个都可转型为属性类型
- 或者它需要暴露特定的功能，比如，它需要符合与组件相兼容的概念

每个复合组件实现了它本身的一系列属性传播规则。为了得到复合生成器消费属性的不同之处的完整列表，请看章节`解析器复合属性规则`和`生成器复合属性规则`。(*TODO*)  
### 序列解析器和生成器的属性
序列需要一个属性类型来暴露融合序列的概念，所有在融合序列中的元数都要可转换为相应的组合序列中的元素，比如，表达式：

| 库 | 序列表达式 |
| --- | --- |
| Qi | `double_ >> double_` |
| Karma | `double_ << double_` |

和任何包含两个都和double相兼容的类型的融合序列相兼容。融合序列的第一个元素必须和第一个`double_`的属性相兼容，第二个融合序列中的元素必须和第二个`double_`的属性相兼容。如果我们假设有一个`std::pair<double, double>`的实例，我们能够直接使用上述两个表达式，解析输入来填充属性：
``` c++
// 下面解析"1.0 2.0"到双浮点的pair中
std::string input("1.0 2.0");
std::string::iterator strbegin = input.begin();
std::pair<double, double> p;
qi::phrase_parse(strbegin, input.end(),
    qi::double_ >> qi::double_,     //解析器文法
    qi::space,                      //定界符文法
    p);                             //用来在解析中填充的属性
```
并且为他生成输出：  
``` c++
//下面从上面填充的pair中生成"1.0 2.0"
std::string str;
std::back_insert_iterator<std::string> out(str);
karma::generate_delimited(out,
    karma::double_ << karma::double_,   //生成器文法（格式化描述）
    karma::space,                       //定界符文法
    p);                                 //被用作属性的数据
```
（karma::space生成器被用作定界符，允许在原生生成器间自动跳过/插入定界的空白字符）  
> 只对序列有效：`Spirit.Qi`和`Spirit.Karma`公开了一系列主要用于序列的API函数。和`scanf`和`printf`之类的函数非常相似，这些函数允许分别传递序列中的每个元素。使用相应的Qi的`parse`或者Karma的`generate`函数，上面的表达式可以写成这样：  
``` c++
double d1 = 0.0, d2 = 0.0;
qi::phrase_parse(begin, end, qi::double_ >> qi::double_, qi::space_, d1, d2);
karma::generate_delimited(out, karma::double_ << karma::double_, karma::space, d1, d2);
```
其中第一个属性用于第一个`double_`，第二个属性用于第二个`double_`。  

### 解析器和生成器中选择表达式的属性
解析器的生成器对选择的支持都很好，为了存储从不同的选项返回的可能的不同的结果（属性）类型，我们使用数据类型`Boost.Variant`。此组件的主要属性的传播规则是：  
```
a: A, b: B --> (a | b): variant<A, B>
```
选择表达式有第二种非常重要的属性传播规则：  
```
a: A, b: A --> (a | b): A
```
通常可以显著简化代码。如果所有的选择的子表达式暴露了相同的属性类型，整个选择表达式将也会暴露同样的属性类型。  
