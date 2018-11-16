## 关于复合组件属性的更多信息
在解析输入和生成输出时，通常需要将一些常量元素和变量部分组合。比如，然我们看看复数（写作`(real, imag)`，`real`和`imag`表示复数的实部和虚部）的解析或格式化的例子。能这么写来实现：  

| 库 | 序列表达式 |
| --- | --- |
| Qi | `'(' >> double_ >> ", " >> double_ >> ')'` |
| Karma | `'(' << double_ << ", " << double_ << ')'` |

幸运的是，字面量（比如'('和", "）不会公开任何属性（其实实际上，他们会公开特殊的`unuses_type`类型，但是在这个上下文中`unuses_type`被解释成好像组件根本没有公开任何属性一样）。明白字面量不会消耗任何传递给组件序列的融合序列的元素是非常重要的。就和刚说的一样，他们不公开任何属性并且不产生（消耗）任何数据。下面的例子说明了这个：  
``` c++  
//下面解析"(1.0, 2.0)"为pair<double, double>
std::string input("(1.0, 2.0)");
std::string::iterator strbegin = input.begin();
std::pair<double, double> p;
qi::parse(strbegin, input.end(),
    '(' >> qi::double_ >> ", " >> qi::double_ >> ')',
    p);
```
下面是等效的`Spirit.Karma`的代码片段：  
``` c++
//下面的代码生成：(1.0, 2.0)
std::string str;
std::back_insert_iterator<std::string> out(str);
generate(out,
    '(' << karma::double_ << ", " << karma::double_ << ')', // 生成器文法（格式化描述）
    p);                                                     // 作为属性使用的数据
```
作为将要生成的数据而传入对组的第一个元素仍然和第一个`double_`相关，第二个元素和第二个`double_`生成器相关。  
这种行为应该是熟悉的，因为它符合其他输入输出格式化库比如`scanf`和`printf`，或者`boost::format`的处理他们变量部分的方式。在这个上下文你可以认为`Spirit.Qi`和`Spirit.Karma`的原生组件（比如上面的`double_`）视为属性值的类型安全占位符。  
> 和上面提供的提示类似，这个例子能够被`Spirit`的多属性API函数重写
``` c++
double d1 = 0.0, d2 = 0.0;
qi::parse(begin, end, '(' >> qi::double_ >> ", " >> qi::double_ >> ')', d1, d2);
karma::generate(out, '(' << karma::double_ << ", " << karma::double_ << ')', d1, d2);
```
> 这提供了清晰且舒适的，非常像`printf`或`boost::format`公开的基于占位符的语法。  

让我们从更正式的角度来看待这个问题。当生成器公开`unused_type`作为他们的属性时，序列属性传播规则定义了一种特殊的行为。（TODO：link-to生成器组件属性规则）

| 库 | 序列属性传播规则 |
| --- | --- |
| Qi | a: A, b: Unused --> (a >> b): A |
| Karma | a: A, b: Unused --> (a << b): A |

其内容如下：
> 给定解析器（或生成器）`a`和`b`，并且`A`是`a`的属性类型，`unused_type`是`b`的属性类型，那么`a >> b`（`a << b`）的属性类型也会是`A`。无论公开`unused_type`的元素的位置如何，这个规则都是适用的。  

一旦涉及字面量，这个规则是理解序列中属性处理的关键。这就好像拥有`unused_type`属性的元素在属性传播过程中消失了一样。尤其是，这不仅在序列而且在任何复合组件中成立。比如，在选择组件中，相应的规则是：  
```
a: A, b: Unused --> (a | b): A
```
再次，允许简化整个表达式的属性类型。  
