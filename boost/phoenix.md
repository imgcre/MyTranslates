## 简介
![avatar](https://www.boost.org/doc/libs/1_68_0/libs/phoenix/doc/html/images/banner.png)

Phoenix库在c++中使能了3函数式编程(FP)技术，比如高阶函数，$\lambda$，科里化(并行函数应用)和懒惰求值。Phoenix侧重于实用性，而非纯粹性、优雅性以及严格遵守FP原则。
FP是一个与具体语言无关的编程学科，能应用在许多编程语言中。在c++中，已经有FP技术被应用的例子。c++已经实现了FP中最重要的部分。c++是个多模式的编程语言，它不仅仅是过程式的，也不仅仅是面向对象的。在标准c++库中，STL中的闭包已经能瞥见FP了。显而易见STL的作者知道并且实践了FP。在近期，我们能确定FP将慢慢成为主流。

## Actor细节
主要的概念是Actor。Actor是多态函数对象概念的模型（能接受0到N个参数，N是已经被预定义的最大值）。一个Actor包括一个有效的Phoenix表达式，一个对函数调用操作符的重载的调用，能开始计算过程。
> 你能设置 `BOOST_PHOENIX_LIMIT`，预定义的actor最大actor元数。默认情况下`BOOST_PHOENIX_LIMIT`的值是10。
### 函数调用操作符
有2*N个函数调用操作符，从0个参数到N个参数，actor类接受参数并且正向传递到默认的计算操作。
附加地，存在函数调用操作符接受常量和非常量的引用参数的排列组合。这些操作符为元数N<=`BOOST_PHOENIX_PERFECT_FORWARD_LIMIT`（默认是3）的actor创建。
### 上下文
在actor函数调用的时候，在调用计算函数之前，acotr创建了一个上下文，这个上下文包括一个环境和Action部分。这些包括了所有计算给定表达式的必要信息。
| 表达式 | 语义 |
| :-- | :--: |
| `result_of::context<Env, Actions>::type` | 上下文的类型 |
| `context(e, a)` | 一个包含环境e和actions a的上下文 |
| `result_of::env<Context>::type` | 所包含环境的类型 |
| `env(ctx)` | 环境 |
| `result_of::actions<Context>::type` | 所包含action的类型 |
| `actions(ctx)` | action |
### 环境
环境是个随机访问序列
通过actor的调用操作符的参数被搜集在了环境内部
在库的其他部分，比如`scope`模块拓展了环境的概念来包含局部变量等信息。
### Actions
Actions在Phoenix中负责给一个实际的表达式一个特定的行为。在Phoenix表达式树遍历时这些actions被调用，当匹配到一个特定的语法规则。
```C++
struct actions
{
    template <typename Rule>
    struct when;
};
```
嵌套的when模板需要为[Proto Primitive Transform](TODO)。不用担心，你现在还不需要学习`Boost.Proto`。phoenix提供了一些包装器来允许你简单地定义一个特定的actions而不需要你深入了解`proto`。
**Phoenix**默认使用预定义的`default_actions`类来计算c++语义的表达式。
```C++
struct default_actions
{
    template <tpyename Rule, typename Dummy = void>
    struct when
        : proto::_default<meta_grammar>
    {};
};
```
更多关于如何使用`default_actions`和如何加入自定义计算过程的信息请见[more on actions]()
### 计算
```C++
struct evaluator
{
    template <typename Expr, typename Context>
    unspecified operator()(Expr &, Context &);
};
evaluator const eval = {};
```
Phoenix表达式的计算在对`evaluator`进行调用后开始。
`evaluator`在上下文被建立后被`actor`函数操作符重载调用。比如，这里是个典型的`actor::operator()`，它接受两个参数：
```C++
template <typename T0, typename T1>
typename result_of::actor<Expr, T0 &, T1 &>：：type
operator()(T0 &t0, T1 &t1) const
{
    fusion::vector2<T0 &, T1 &> env(t0, t1);

    return eval(*this, context(env, default_actions()));
}
```
### result_of::actor
为了`actor::operator()`的对称性，有个具体可用的用来表示actor返回值类型的元函数，称为`result_of::actor`。此元函数允许我们直接指定通过`actor::operator()`函数的参数的类型。以下是典型的接受两个参数的`actor_result`
```C++
namespace result_of
{
    template <typename Expr, typename T0, typename T1>
    struct actor
    {
        typedef fusion::vector2<T0, T1> env_type;
        typedef typename result_of::context<env_type, default_actions>::type ctx_type;
        typedef typename boost::result_of<evaluator(Expr const&, ctx_type)>::type type;
    };
}
```