## 关于复合组件属性的更多信息
在解析输入和生成输出时，通常需要将一些常量元素和变量部分组合。比如，然我们看看复数（写作`(real, imag)`，`real`和`imag`表示复数的实部和虚部）的解析或格式化的例子。能这么写来实现：  

| 库 | 序列表达式 |
| --- | --- |
| Qi | `'(' >> double_ >> ", " >> double_ >> ')'` |
| Karma | `'(' << double_ << ", " << double_ << ')'` |

幸运的是，字面量（比如'('和", "）不会暴露任何属性（其实实际上，他们会暴露特殊的`unuses_type`类型，但是在这个上下文中`unuses_type`被解释成好像组件根本没有暴露任何属性一样）。明白字面量不会消耗任何传递给组件序列的融合序列的元素是非常重要的。就和刚说的一样，他们不暴露任何属性并且不产生（消耗）任何数据。下面的例子说明了这个：  
``` c++  
//下面解析"(1.0, 2.0)"为pair<double, double>
std::string input("(1.0, 2.0)");
std::string::iterator strbegin = input.begin();
std::pair<double, double> p;
qi::parse(strbegin, input.end(),
    '(' >> qi::double_ >> ", " >> qi::double_ >> ')',
    p);
```
