# 目录结构
## include
Spirit是个只有头文件的库，不需要链接其他的库。本节文档给出了Spirit头文件的目录结构
Spirit包括5个子库，加上一个'support'模块，里面包含通用的支持类。
- Classic
- Qi
- Karma
- Lex
- Phoenix
- Support

Spirit的顶级目录为
```bash
BOOST_ROOT/boost/spirit
```
目前，这个目录包含
```bash
[actor]     [attribute]         [core]      [debug]
[dynamic]   [error_handling]    [home]      [include]
[iterator]  [meta]              [phoenix]   [repository]
[symbols]   [tree]              [utility]
```
这其中有一些v1.8老版本的目录现在被*贬低*了，他们是actor, attribute, core, debug, dynamic, error_handling, iterator, meta, phoenix, symbols, tree和utility，不能保证这些目录在未来版本的Spirit中会被提供。请注意，这些目录只是用来向后兼容的。
每个目录（除了include, home和repository）有一个相应的头文件，它包含了此目录下包含的相关文件的引用。比如，`boost/spirit/actor.hpp`头文件引用了所有在目录`boost/spirit/actor`下的相关文件。
你能通过检查version文件来区别不同版本的Spirit
```bash
<boost/spirit/version.hpp>
```
并使用预处理定义
```c
SPIRIT_VERSION
```
它是一个十六进制的值，前两位表示主版本，后两位表示子版本，比如
```c
#define SPIRIT_VERSION 0x2010 //版本为2.1
```
include目录为`BOOST_ROOT/boost/spirit/include`，它是个特殊的扁平的文件夹，包含了所有的Spirit头文件。为了与扁平目录结构相联系，头文件名以其子库名称作为前缀。
- classic_
- karma_
- lex_
- phoenix1_
- phoenix_
- qi_
- support_

比如，你之前之前引用`boost/spirit/actor.hpp`，现在是个被贬低的头文件，你需要用引用`boost/spirit/include/classic_actor.hpp`来代替
如果你想要简单地听筒部分主子库名，你可以引用：
- `boost/spirit/include/classic.hpp`
- `boost/spirit/include/karma.hpp`
- `boost/spirit/include/lex.hpp`
- `boost/spirit/include/phoenix1.hpp`
- `boost/spirit/include/phoenix.hpp`
- `boost/spirit/include/qi.hpp`
- `boost/spirit/include/support.hpp`

主目录`BOOST_ROOT/boost/spirit/home`是Spirit的真实*home*，这里是多个子库实际上存在的地方，主目录包括
```bash
[classic]   [karma]     [lex]
[phoenix]   [qi]        [support]
```
通常，这些目录里有他们相应的包含文件。
- `boost/spirit/home/classic.hpp`
- `boost/spirit/home/karma.hpp`
- `boost/spirit/home/lex.hpp`
- `boost/spirit/home/phoenix.hpp`
- `boost/spirit/home/qi.hpp`
- `boost/spirit/home/support.hpp`

多种子库包含文件可以能在每个包含特定子库的子目录下找到，子库的包含结构在他们相应的文档中有描述。为了一致性，每个子库都遵循如上的方案。
**为了简单起见，你需要使用在`boost/spirit/include`中的扁平的包含路径。**
子目录`boost/spirit/repository`不属于主Spirit distribution。
更多信息请见boost官方