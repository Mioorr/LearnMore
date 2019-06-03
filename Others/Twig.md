# Twig

**什么是 Twig？**

- 是一款 PHP 模板引擎。Twig 将模板编译为纯粹的、最优化的 PHP 代码，它的开销与常规的 PHP 代码相比，降到了及低。
- Twig 拥有沙盒模式，用于评估未受信任的模板代码。因此 Twig 可以用于允许用户自行修改模板设计的应用程序中。
- Twig 由一个灵活的词法分析器和解析器驱动。因此开发者可以自定义标签和过滤器，并创建自己的 DSL。

---

## 1.安装

**安装 Twig PHP package：**

通过 Composer 安装（推荐），先安装 Composer，然后运行以下命令获取最新版 Twig：

```shell
composer require twig/twig:~1.0
```

**安装压缩包版：**

1. 从[下载页面](https://github.com/twigphp/Twig/tags)下载最新的压缩包。
2. 验证压缩包的完整性 <http://fabien.potencier.org/article/73/signing-project-releases>。
3. 解压压缩包。
4. 将文件移入项目内容合适的目录中。

**安装开发版：**

```shell
git clone git://github.com/twigphp/Twig.git
```

**安装 C 扩展：**

> Twig v1.4 版本加入的新东西。
>
> <font color=orange>Notice：</font>C 扩展是可选的，它带来一些性能提升。扩展程序不是 PHP 代码的替代品，它只实现了部分 PHP 代码在运行时的性能提升，原生 PHP 代码仍是需要安装的。

Twig 的 C 扩展增强了 Twig 运行引擎的性能。像安装其他 PHP 扩展那样安装它：

```shell
cd ext/twig
phpize
./configure
make
make install
```

Windows 平台：

1. 参照[ PHP 文档](https://wiki.php.net/internals/windows/stepbystepbuild)设置构建环境。
2. 将 Twig 的 C 扩展源代码放入`C:\php-sdk\phpdev\vcXX\x86\php-source-directory\ext\twig`
3. 使用`configure --disable-all --enable-cli --enable-twig=shared` 命令，替代第十四步。
4. `nmake`
5. 复制`C:\php-sdk\phpdev\vcXX\x86\php-source-directory\Release_TS\php_twig.dll`文件到PHP设置中。

最后，在`php.ini`配置文件中启用扩展。

```shell
extension=twig.so #For Unix systems
extension=php_twig.dll #For Windows systems
```

> 启用后，Twig 将利用 C 扩展自动编译你的模板。C 扩展并未替代 PHP 代码，但提供了`Twig_Template::getAttribute()`方法的优化版。

---

## 2.基础语法

### 2.1 概述

模板其实就是一个简单的文本文件。它可以生成任意基于文本的格式（HTML、XML、 CSV、 LaTeX 等）。它没有特定的扩展，`.html`或`.xml`都行。

模板包含变量或表达式，在评估模板时，这些带值的变量或表达式会被替换；另外，还有控制模板逻辑的标签。

```twig
<!DOCTYPE html>
<html>
	<head>
  	<title>My Webpage</title>
  </head>
  <body>
  	<ul id="navigation">
    	{% for item in navigation %}
      	<li><a href="{{ item.href }}">{{ item.caption }}</a></li>
      {% endfor %}
    </ul>
    <h1>My Webpage</h1>
    {{ a_variable }}
  </body>
</html>
```

Twig 的分隔符有两种形式：

1. `{% ... %}`：用于执行语句（如 for 循环）。
2. `{{ ... }}`：将表达式的结果输出到模板中。

> <font color=orange>Notice：</font>花括号只是一个打印声明，不是变量的一部分。访问标签内部变量时，不要将其放在花括号中。

### 2.2 变量

应用程序将变量传入模板中进行处理。变量可以包含可访问的属性或元素。变量的可视化表现形式很大程度上取决于提供变量的应用程序。

访问变量的属性/方法/PHP 对象的属性/PHP数组单元 有2种方式：`.`和`[]`。

```twig
{{ foo.bar }}
{{ foo['bar'] }}
```

`attribute()`函数可以访问变量属性（包括动态属性），当属性包含特殊字符时（如`-`会被解析为减号）：

```twig
{# 相当于不工作的foo.data-foo #}
{{ attribute(foo, 'data-foo') }}
```

如果模板中使用无效变量（变量名无效或属性不存在），当`strict_variables`（是否使用严格变量模式）为`false`，将收到`null`；若为`true`将抛出一个错误（查看环境选项）。

**实现：**

为方便起见，变量属性的访问在 PHP 层做了以下工作（以`foo.bar`为例）：

```
foo.bar
|_ foo是有效数组 => bar是一个有效元素
|_ foo是对象
|  |_ bar是有效属性    _
|  |_ bar是有效方法    _|__ 都不是 => 返回nul
|  |_ getBar是有效方法 _|
|  |_ isBar是有效方法  _|
|_ foo是数字
   |_ bar是有效元素
   |_ bar是无效元素 => 返回null
```

`foo['bar']`只适用于 PHP 数组。

**全局变量：**

以下变量在模板中始终可用：

- `_self`: 引用当前模板；
- `_context`: 引用当前上下文；
- `_charset`: 引用当前字符集；

**设置变量：**

用`set`标签可以声明变量。

### 2.3 过滤器

可以通过过滤器来修改变量。过滤器中，用`|`分隔多个变量，还可以在括号中加入可选参数。可以链接多个过滤器。一个过滤器的输出结果将用于下一个过滤器中。

```twig
{# 删除所有带有name和title-cases的HTML标签 #}
{{ name|striptags|title }}
```

过滤器接收由圆括号包裹的参数。

```twig
{# 加入由', '分隔的参数列表 #}
{{ list|join(', ') }}
```

要在一段代码中应用过滤器，需要将它包裹在`filter`标签中：

```twig
{% filter upper %}
	This text becomes uppercase
{% endfilter %}
```

### 2.4 函数

函数可被调用，用于生产内容。函数通过函数名被调用，其后紧跟圆括号(`()`)，它还可以设置参数。

```twig
{# range返回一个包含整数等差数列的列表 #}
{% for i in range(0, 3) %}
	{{ i }},
{% endfor %}
```

**具名实参：**

> Twig v1.12 新加入的具名实参支持。

```twig
{% for i in range(low=1, high=10, step=2) %}
	{{ i }},
{% endfor %}
```

使用具名实参，使模板中作为参数被传递的值更清晰：

```twig
{{ data|convert_encoding('UTF-8', 'iso-2022-jp') }}
{# 使用具名实参 #}
{{ data|convert_encoding(from='iso-2022-jp', to='UTF-8') }}
```

具名实参可以跳过某些不需要改变默认值的参数：

```twig
{# 第一个参数是日期格式, 如果为空, 默认是全局日期格式 #}
{{ "now"|date(null, "Europe/Paris") }}
{# 或者, 通过为时区使用一个具名实参来跳过格式值 #}
{{ "now"|date(timezone="Europe/Paris") }}
```

可以在一次调用中，同时使用位置参数和具名实参，此时，位置参数必须放在具名实参前：

```twig
{{ "now"|date('d/m/Y H:i', timezone="Europe/Paris") }}
```

### 2.5 控制结构

控制结构指控制程序流程的所有东西——条件语句（例如 `if`/`elseif`/`else`）、`for`循环、程序块等。控制结构出现在 `{% ... %}`块中。

```twig
{# 要显示一个被调用的user变量中提供的user列表，使用for标签 #}
<h1>Members</h1>
<ul>
	{% for user in users %}
  	<li>{{ user.username|e }}</li>
	{% endfor %}
</ul>
```

if 标签可以用来测试表达式：

```twig
{% if users|length > 0 %}
	<ul>
  	{% for user in users %}
    	<li>{{ user.username|e }}</li>
    {% endfor %}
  </ul>
{% endif %}
```

### 2.6 注释

要在模板中注释某一行，使用注释语法`{# ...#}`。

```twig
{# note: disabled template because we no longer use this
	{% for user in users %}
  	...
  {% endfor %}
#}
```

### 2.7 引入其他模板

Twig 提供的`include()`函数用于在模板中引入模板，并将该模板已渲染后的内容返回到当前模板：

```twig
{{ include('sidebar.html') }}
```

默认被引入的模板可以使用当前模板的上下文，因此在主模板中定义的任意变量，在被引入的模板中同样可用。

```twig
{% for box in boxes %}
	{{ include('render_box.html') }}
{% endfor %}
```

被引入的模板`render_box.html`可以使用`box`变量。

模板的文件名，取决于模板加载器（如`Twig_Loader_Filesystem`允许通过给定文件名称访问其他模板），可以使用斜线来访问子目录内的模板：

```twig
{{ include('sections/articles/sidebar.html') }}
```

这个行为取决于内嵌 Twig 的应用程序。

### 2.8 HTML 转义

由模板生成HTML总会存在一个风险：包含字符的变量会影响最终生成的HTML。有两种办法来处理：

1. 手动转义每个变量；
2. 默认自动转义所有。

Twig两者都支持，自动转义是默认启用的。

> <font color=orange>Remember：</font>只有在 escaper 扩展启用的情况下（这个扩展默认是启用的），才支持自动转义。

**使用手动转义：**

如果选择了手动转义，就需要自己转义所有需要的变量。

转义变量通过`escape`或`e`过滤器进行：

```twig
{{ user.username|e }}
```

`escape`过滤器默认使用`html`策略，但取决于转义的上下文，可能需要显式地使用其他可用策略：

```twig
{{ user.username|e('js') }}
{{ user.username|e('css') }}
{{ user.username|e('url') }}
{{ user.username|e('html_attr') }}
```

**使用自动转义：**

不论是否启用了自动转义，都可以在模板中，使用`autoescape`标签标记某一部分是否已被转义：

```twig
{% autoescape %}
	本块内所有东西都会被以HTML策略自动转义掉
{% endautoescape %}
```

自动转义默认使用`html`转义策略。如果在其他语境中输出变量，必须使用合适的转义策略显式转义：

```twig
{% autoescape 'js' %}
  Everything will be automatically escaped in this block (using the JS strategy)
{% endautoescape %}
```

**转义**

忽略 Twig 模板的某一部分，有时是可取的，甚至必要的，被忽略的部分作为变量或代码块处理（如使用默认的语法时，想在模板中以原生字符串的形式使用`{{`，而不是以变量的开头定界符来使用，此时便存在一个风险）。

最简单的办法就是使用变量表达式来输出变量定界符(`{{`) ：

```Twig
{{ '{{' }}
```

对于较大的段落，它也能一字不差的处理。参考`verbatim`标签。

### 2.9 表达式

Twig 允许在任意位置使用表达式。表达式非常类似常规的 PHP，甚至你不需要用到PHP，你会感到非常舒适。

> <font color=orange>Notice：</font>
>
> 运算符优先级，从低到高：`b-and`、`b-xor`、`b-or`、`or`、`and`、`==`、`!=`、`<`、`>`、`>=`、`<=`、`in`、`matches`、`starts with`、`ends with`、`..`、`+`、`-`、`~`、`*`、`/`、`//`、 `%`、`is`、`**`、`|`、`[]`、 `.` ：

> ```twig
> {% set greeting = 'Hello ' %}
> {% set name = 'Fabien' %}
> {{ greeting ~ name|lower }} {# Hello fabien #}
> {# 使用()更改优先级 #}
> {{ (greeting ~ name)|lower }} {# hello fabien #}
> ```

**字面值**

> 在 Twig 1.5中加入的，支持使用散列键作为名称和表达式。

表达式的最简单形式就是字面值。字面值是对PHP类型的陈述，比如字符串、数字、以及数组。以下字面值都是存在的：

- `"Hello World"`：在双引号或单引号中的任何内容都是一个字符串。字符串可以包含由反斜线（`\`）开头的定界符（如`'It\'s good'`），它可以转义特殊字符。
- `42` / `42.23`：整型数和浮点数只需写下它们即可创建。`.`表示该数字是浮点数，没`.`是整型数。
- `["foo", "bar"]`：字符串，由一对方括号`[]`包裹的由逗号`,`分隔的表达式序列组成。
- `{"foo": "bar"}`：散列，由一对花括号`{}`包裹的以逗号分隔的键值对列表构成。

```twig
{# keys as string #}
{ 'foo': 'foo', 'bar': 'bar' }
{# keys as names (equivalent to the previous hash) -- as of Twig 1.5 #}
{ foo: 'foo', bar: 'bar' }
{# keys as integer #}
{ 2: 'foo', 4: 'bar' }
{# keys as expressions (the expression must be enclosed into parentheses) -- as of Twig 1.5 #}
{ (1 + 1): 'foo', (a ~ 'b'): 'bar' }
```

- `true` / `false`: `true` 表示正确的值，`false`表示错误的值。
- `null`: `null` 表示没有具体的值。这是在变量没有值时返回的结果。`none`是`null`的别名。

数组和散列可以嵌套：

```twig
{% set foo = [1, {"foo": "bar"}] %}
```

> <font color=blue>Reminder：</font>使用单引号或双引号字符串在性能上没有区别，但只有双引号字符串支持字符串插值。

**数学运算**

Twig允许值计算。这很少用在模版中，但由于完整性的缘故而存在。Twig支持以下运算符：

- `+`: 加，`{{1 + 1 }}` => 2。
- `-`: 减，`{{ 3 - 2 }}` => 1。
- `/`: 除，`{{ 1 / 2 }}` => 0.5。
- `*`: 乘，`{{ 2 * 2 }}` => 4。
- `%`: 取余，`{{ 11 % 7 }}` => 4。
- `//`: 取整，`{{ 20 // 7 }}` is `2`, `{{ -20 // 7 }}` => -3，（它只是`:doc:run<filters/round>`filter）。
- `**`: 幂，`{{ 2 ** 3 }}`=> 8。

**逻辑**

你可以使用以下操作符来组合多个表达式：

- `and`: 与。
- `or`: 或。
- `not`: 非。
- `(expr)`: 聚合表达式。

> <font color=orange>Notice：</font>Twig 还支持位操作符（`b-and`, `b-xor`, 和 `b-or`）。
>
> <font color=orange>Reminder：</font>运算操作符是大小写敏感的。

**比较运算符**

这些比较运算符可以用于任意表达式中：`==`、`!=`、`<`、`>`、`>=`、`<=`。

检查字符串是否由另一个字符串开头`starts with`或结尾`ends with`：

```twig
{% if 'Fabien' starts with 'F' %}
{% endif %}
{% if 'Fabien' ends with 'n' %}
{% endif %}
```

> <font color=orange>Notice：</font>对于复杂的字符串比较，`matches`匹配操作符允许使用正则表达式：

> ```twig
> {% if phone matches '/^[\\d\\.]+$/' %}
> {% endif %}
> ```

**包含操作符**

包含操作符 `in` 用于测试是否存在包含关系。

如果左侧运算对象包含在右侧运算对象中，则返回 `true`：

```twig
{# returns true #}
{{ 1 in [1, 2, 3] }}
{{ 'cd' in 'abcde' }}
```

> <font color=blue>Reminder：</font>可以用这个过滤器对字符串、数组、实现`Traversable`接口的对象进行包含关系测试。

`not in`操作符用于测试是否存在不包含关系：

```twig
{% if 1 not in [1, 2, 3] %}
{# 全等于 #}
{% if not (1 in [1, 2, 3]) %}
```

**测试操作符**

测试操作符`is`用于针对变量和一般表达式之间的关系进行测试。右侧操作数即是测试的名称：

```twig
{# 检查变量是否是奇数 #}
{{ name is odd }}
```

测试可以接受参数：

```twig
{% if post.status is constant('Post::PUBLISHED') %}
```

 `is not` 操作符进行否测试：

```twig
{% if post.status is not constant('Post::PUBLISHED') %}
{# 相当于 #}
{% if not (post.status is constant('Post::PUBLISHED')) %}
```

**其他操作符**

> 在Twig 1.12.0版加入了对扩展的三元操作符的支持。

以下操作符不适用于其他类型中：

- `|`：使用一个过滤器。

- `..`：创建一个基于操作符前后操作数的序列（这只是range函数的语法糖）：

  ```twig
  {{ 1..5 }}
  {# equivalent to #}
  {{ range(1, 5) }}
  ```

  > <font color=orange>Notice：</font>由于操作符优先级规则的原因，在将本操作符与过滤器组合时必须使用括号包裹：

  ```twig
  (1..5)|join(', ')
  ```

- `~`: 将所有操作数转换为字符串并连接它们。`{{ "Hello " ~ name ~ "!" }}` 将会返回（嘉定`name` 是 `'John'`） `Hello John!`。

- `.`、`[]`：获取对象的属性。

- `?:`：三元操作符。

  ```twig
  {{ foo ? 'yes' : 'no' }}
  {# 从Twig v1.12.0开始 #}
  {{ foo ?: 'no' }} <==> {{ foo ? foo : 'no' }}
  {{ foo ? 'yes' }} <==> {{ foo ? 'yes' : '' }}
  ```

- `??`: 非空操作符。

  ```twig
  {# 如果foo已被定义非空值，则返回foo的值，否则返回'no' #}
  {{ foo ?? 'no' }}
  ```

**字符串插值**

> 在 Twig1.5 中加入的字符串插值

字符串插值(`#{expression}`)允许在==双引号字符串==中出现任意有效的表达式。表达式的结果插入到字符串中。

```twig
{{ "foo #{bar} baz" }}
{{ "foo #{1 + 2} baz" }}
```

**空白控制**

> 在Twig 1.1版，加入了标签级的空白控制。

模板标签后的第一个换行会被自动移除（如同PHP）。空白不是由模板引擎进一步修改的，所以每个空白（空格、制表符、换行）都被未改变地返回。

使用 `spaceless` 标签一处*HTML之间*的空白：

```twig
{% spaceless %}
	<div>
  	<strong>foo bar</strong>
  </div>
{% endspaceless %}
{# 将会输出 <div><strong>foo bar</strong></div> #}
```

除了 spaceless 标签，还可以针对每个标签级进行空白控制。通过对标签使用空白控制修改器，可以移除首尾的空白：

```twig
{% set value = 'no spaces' %}
{#- 没有前导/尾随空格 -#}
{%- if true -%}
	{{- value -}}
{%- endif -%}

{# 可以用它来消除标签某一侧的空白 #}
{% set value = 'no spaces' %}
<li>    {{- value }}    </li>
{# outputs '<li>no spaces    </li>' #}
```

**扩展**

Twig可被轻松扩展。

---

## 3.API

### 3.1 基础

Twig 使用 environment 中心对象（`Twig_Environment`类的对象）。这个类的实例用于存储配置和扩展，以及从文件系统或其他位置加载模版。

大多数应用程序在初始化时创建一个 `Twig_Environment` 对象，并用它来加载模板。在某些情况下，如果在使用多个不同的配置，是可以让多个环境存在使用的。

为应用程序配置 Twig 加载模板的最简单方式大概是这样的：

```php
require_once '/path/to/lib/Twig/Autoloader.php';
Twig_Autoloader::register();
$loader = new Twig_Loader_Filesystem('/path/to/templates');
$twig = new Twig_Environment($loader, array(
	'cache' => '/path/to/compilation_cache',
));
/* 这将会创建一个带默认设置的模板环境，以及一个在'/path/to/templates/'查找模版的加载器 */
/* 不同的加载器都可用，如果从数据库或其他来源加载模版，可以自己写一个加载器 */
```

> <font color=orange>Notice：</font>environment 的第二个参数是一个环境选项数组。`cache`表示编译缓存目录，为避免解析阶段的重复请求，Twig 将编译过的模板缓存在这个目录中。这与你可能想要为评估后的模版添加的缓存有很大的区别。对于这样的需求，你可以使用其他PHP缓存库。

要从环境中加载模版，只需调用`loadTemplate()`方法即可，它会返回一个`Twig_Template`实例：

```php
$template = $twig->loadTemplate('index.html');
```

要渲染带有变量的模板，调用`render()`方法：

```php
echo $template->render(array('the' => 'variables', 'go' => 'here'));
```

> <font color=orange>Notice：</font>`display()`方法是直接输出模版的快捷方式。

还可以一步完成模板的加载和渲染：

```php
echo $twig->render('index.html', array('the' => 'variables', 'go' => 'here'));
```

### 3.2 环境选项

在创建一个新的 `Twig_Environment` 实例时，你可以传递一个选项数组作为构造函数的第二个参数：

```php
$twig = new Twig_Environment($loader, array('debug' => true));
```

有以下这些可用选项：

- `debug`（*Boolean*）：为`true`时，生成的模版会有一个 `__toString()` 方法，可以用它来宣誓生成的节点（默认是`false`）。

- `charset`（*String (默认utf-8)*）：用于模板的字符集。

- `base_template_class`*（String (默认 Twig_Template)*）：用于生成的模板的基础模板类。

- `cache`（*string|false*）：存储编译后的模板的绝对路径，或者设为 `false`来禁用缓存（默认`false`）。

- `auto_reload`（*boolean*）：在使用Twig进行开发时，这有助于在源代码修改时随时进行重新编译。如果不为`auto_reload`选项设置值，它会自动根据`debug`选项的值被设定。

- `strict_variables`（*boolean*）：如果为 `false`，Twig 会静默忽略无效的变量（包括变量、不存在的属性和方法），并以`null`值替换它们。如果将其设置为 `true`，Twig则会抛出一个异常（默认是`false`）。

- `autoescape`（*string|boolean*）：如果为`true`，会默认为所有模板弃用HTML自动转义（默认是`true`）。

  从Twig v1.8开始，可以将转义策略设置为（`html`、`js`或用`false`禁用）。

  从Twig v1.9开始，可以将转义策略设置为（`css`, `url`, `html_attr`，或者 PHP 回调函数接受模板"filename"并且必须返回要使用的转义策略——回调函数不能是函数名——来不免与内置转移策略发生冲突）。

  从Twig v1.17开始 , `filename`转义策略决定了用于基于模板文件名扩展的模板的转义策略（由于自动转义在编译时就已经完成，所以本策略不会产生任何开销）

- `optimizations`*integer*

  此处的标记表示哪种优化方式将被应用（默认是`-1` —— 所有的优化设置都被弃用；设置为`0`，则禁用）。

\##加载器（Loader）

加载器负责从文件系统之类的来源加载模板。

\###编译缓存

所有模板加载器都可以在文件系统中缓存编译后的模板以便将来重用。模板只需要编译一次就能为Twig提速不少，并且如果使用了APC之类的加速器，还能带来更大的性能提升。查看前文中`Twig_Environment`的`cache`和`auto_reload`两个选项了解更多信息。

\###内置的加载器

这是一个由Twig提供的内置加载器列表：

\####`Twig_Loader_Filesystem`

> 在 Twig 1.10版中，加入了 `prependPath()`，并支持命名空间。

`Twig_Loader_Filesystem`从文件系统加载模板。这个加载器可以从文件系统目录中找到模板，这是加载模板的最佳方式：

```
$loader = new Twig_Loader_Filesystem($templateDir);
```

它还可以从由多个目录组成的数组中寻找模板：

```
$loader = new Twig_Loader_Filesystem(array($templateDir1, $templateDir2));
```

按照这样的配置，Twig会首先在 `$templateDir1` 中查找模板，如果没有，则回退，然后在 `$templateDir2` 中继续查找模板。

还可以通过`addPath()` 和 `prependPath()`方法添加或预设路径：

```
$loader->addPath($templateDir3);
$loader->prependPath($templateDir4);
```

文件系统加载器还支持命名空间模板。这允许将拥有各种路径的模板组织到不同的命名空间下》

在使用 `setPaths()`, `addPath()`, 和 `prependPath()` 方法时，将命名空间指定为第二个参数，如果没有指定，这些方法会调用主（main）命名空间：

```
$loader->addPath($templateDir, 'admin');
```

命名空间模板可以用通过特定的 `@namespace_name/template_path`符号访问：

```
$twig->render('@admin/index.html', array());
```

\####`Twig_Loader_Array`

使用`Twig_Loader_Array`从PHP数组加载模板。它被传递一个绑定模板名称的字符串数组：

```
    $loader = new Twig_Loader_Array(array(
        'index.html' => 'Hello {{ name }}!',
    ));
    $twig = new Twig_Environment($loader);
    
    echo $twig->render('index.html', array('name' => 'Fabien'));
```

这个加载器对于单元测试非常有用。它还可以用于将所有模板存放在单个PHP文件内的小型项目，的确可以这样做。

提示：

> 在使用带有缓存极致的 `Array` 或 `String` 加载器时，你应当明白新的缓存键时在每次模板内容改变时生成的（缓存键是指模板的源代码）。如果不希望缓存失控地增加，你需要注意自行清除旧的缓存。

\####`Twig_Loader_Chain`

`Twig_Loader_Chain` 将模板的加载工作委派给其他加载器：

```
$loader1 = new Twig_Loader_Array(array(
    'base.html' => '{% block content %}{% endblock %}',
));
$loader2 = new Twig_Loader_Array(array(
    'index.html' => '{% extends "base.html" %}{% block content %}Hello {{ name }}{% endblock %}',
    'base.html'  => 'Will never be loaded',
));

$loader = new Twig_Loader_Chain(array($loader1, $loader2));

$twig = new Twig_Environment($loader);
```

在查找模板时，Twig会轮流尝试每个加载器，并在找到模板时立即返回。前面的例子中，在渲染 `index.html`模板时，Twig 会使用`$loader2`来加载它，但`base.html`模板会从`$loader1`中被加载。

`Twig_Loader_Chain` 能接收任意实现了 `Twig_LoaderInterface`接口的加载器。

注意：

> 你还可以使用`addLoader()` 方法来添加加载器。

\###创建你自己的加载器

所有的加载器都实现了 `Twig_LoaderInterface`：

```
interface Twig_LoaderInterface
{
    /**
     * Gets the source code of a template, given its name.
     *
     * @param  string $name string The name of the template to load
     *
     * @return string The template source code
     */
    function getSource($name);

    /**
     * Gets the cache key to use for the cache for a given template name.
     *
     * @param  string $name string The name of the template to load
     *
     * @return string The cache key
     */
    function getCacheKey($name);

    /**
     * Returns true if the template is still fresh.
     *
     * @param string    $name The template name
     * @param timestamp $time The last modification time of the cached template
     */
    function isFresh($name, $time);
}
```

考虑到最后修改的时间，如果当前被缓存的模板仍然是最新的，则 `isFresh()` 方法必须返回 `true`，否则返回`false`。

提示：

> 从Twig 1.11.0 开始，你还可以实现 `Twig_ExistsLoaderInterface`，让你的加载器在使用链式加载器时更快速。

\##使用扩展

Twig的扩展程序其实是为Twig添加新特性的包（package）。使用扩展跟使用 `addExtension()` 方法一样简单：

```
$twig->addExtension(new Twig_Extension_Sandbox());
```

Twig comes bundled with the following extensions:

- *Twig_Extension_Core*: 定义Twig的所有核心特性。
- *Twig_Extension_Escaper*: 为代码块添加自动化输出转义以及是否转义的可能性。
- *Twig_Extension_Sandbox*: 为默认的Twig环境添加沙河模式，使其能安全地评估未受信任的代码。
- *Twig_Extension_Profiler*: 启用内置的Twig分析器（Twig 1.18版本以上可用）。
- *Twig_Extension_Optimizer*: 在编译前优化节点树。

上面的 core, escaper, 以及 optimizer 扩展不是必须要添加到 Twig环境中，因为它们都是默认已被注册的。

\##内置的扩展

这一节介绍由内置扩展添加的特性

提示：

> 这一章是关于扩展Twig的，阅读这一章学习如何创建你自己的扩展。

\###核心扩展`core`

`core`扩展定义Twig的所有核心特性：

- Tags
- Filters
- Functions
- Tests

\###转义扩展`Escaper`

使用 `escaper` 扩展，为Twig添加动态输出转义。它定义了`autoescape`标签和 `raw`过滤器。

在创建转义扩展时，你可以打开或者关闭全局输出转义策略：

```
$escaper = new Twig_Extension_Escaper('html');
$twig->addExtension($escaper);
```

如果将其设置为 `html`，模板中的所有变量都会被转义（使用`html`转义策略）, 除非是用了 `raw` 过滤器。：

```
{{ article.to_html|raw }}
```

还可以使用 `autoescape` 标签来局部地改变转义模式（查看 autoescape 文档，了解Twig 1.8以上的语法）：

```
{% autoescape 'html' %}
    {{ var }}
    {{ var|raw }}      {# var won't be escaped #}
    {{ var|escape }}   {# var won't be double-escaped #}
{% endautoescape %}
```

警告：

> `autoescape` 对引入的文件没有影响。

像下面这样实现转义规则：

- 在模板中直接用作变量或过滤器参数的字面值（包括整型数、布尔值、数组等）**从不**被自动转义：

  ```
    {{ "Twig<br />" }} {# won't be escaped #}
  
    {% set text = "Twig<br />" %}
    {{ text }} {# will be escaped #}
  ```

- 被标注为安全的，其结果总是一个字面值或变量的表达式，**从不**被自动转义：

  ```
    {{ foo ? "Twig<br />" : "<br />Twig" }} {# won't be escaped #}
  
    {% set text = "Twig<br />" %}
    {{ foo ? text : "<br />Twig" }} {# will be escaped #}
  
    {% set text = "Twig<br />" %}
    {{ foo ? text|raw : "<br />Twig" }} {# won't be escaped #}
  
    {% set text = "Twig<br />" %}
    {{ foo ? text|escape : "<br />Twig" }} {# the result of the expression won't be escaped #}
  ```

- 转义应用于打印之前，和其他过滤器应用之后：

  ```
    {{ var|upper }} {# is equivalent to {{ var|upper|escape }} #}
  ```

- `raw`过滤器只能用在过滤器链的结尾：

  ```
    {{ var|raw|upper }} {# will be escaped #}
  
    {{ var|upper|raw }} {# won't be escaped #}
  ```

- 如果当前上下文（context，例如`html`或`js`）的过滤器链中最后一个过滤器被标注为安全，那么自动转义不会被应用。`escape`和 `escape('html')` 用于将 HTML标注为安全，`escape('js')` 用于将JavaScript标注为安全，`raw`可以将任意内容标注为安全：

  ```
    {% autoescape 'js' %}
        {{ var|escape('html') }} {# will be escaped for HTML and JavaScript #}
        {{ var }} {# will be escaped for JavaScript #}
        {{ var|escape('js') }} {# won't be double-escaped #}
    {% endautoescape %}
  ```

注意：

> 自动转义有一些局限性，因为针对表达式的转义是在评估之后才应用的。举个例子，在处理连接时，`{{ foo|raw ~ bar }}` 不会给出预期结果，因为转义是应用于连接的结果上的，而不是应用在单个变量上（所以，`raw`过滤器此时不会生效）。

\###沙盒扩展`sandbox`

`sandbox`扩展用于评估未被信任的代码。禁止访问不安全的属性和方法。沙盒的安全性由一个policy实例进行管理。默认地，Twig带有一个policy类： `Twig_Sandbox_SecurityPolicy`。这个类允许你为标签、属性、方法添加白名单：

```
$tags = array('if');
$filters = array('upper');
$methods = array(
    'Article' => array('getTitle', 'getBody'),
);
$properties = array(
    'Article' => array('title', 'body'),
);
$functions = array('range');
$policy = new Twig_Sandbox_SecurityPolicy($tags, $filters, $methods, $properties, $functions);
```

基于上述配置，安全策略仅允许使用`if`标签和`upper`过滤器。而且，模板只能用`Article`对象调用 `getTitle()` 和 `getBody()`方法。其它的用法都被禁止，并会生成一个 `Twig_Sandbox_SecurityError` 异常。

策略对象 policy 是沙盒构造函数的第一个参数：

```
$sandbox = new Twig_Extension_Sandbox($policy);
$twig->addExtension($sandbox);
```

默认地，沙盒模式是被禁用了的。但在使用`sandbox`标签引入未被信任的模板代码时，应当启用沙盒模式：

```
{% sandbox %}
    {% include 'user.html' %}
{% endsandbox %}
```

可以通过将 extension 构造函数的第二个参数设置为`true`，将所有模板放入沙盒中：

```
$sandbox = new Twig_Extension_Sandbox($policy, true);
```

\###分析器扩展`profiler`

> 在 Twig 1.18 添加了分析器扩展

分析器扩展`profiler`为Twig模板启用了分析器；由于它增加了一些开销，所以只能在开发环境中使用它：

```
$profile = new Twig_Profiler_Profile();
$twig->addExtension(new Twig_Extension_Profiler($profile));

$dumper = new Twig_Profiler_Dumper_Text();
echo $dumper->dump($profile);
```

一份分析结果包含了模板、代码块、以及指令执行的时间和内存消耗等信息

可以将分析数据转换成 [Blackfire.io](https://blackfire.io/) 兼容的格式：

```
$dumper = new Twig_Profiler_Dumper_Blackfire();
file_put_contents('/path/to/profile.prof', $dumper->dump($profile));
```

将分析结果上传，使其可视化（需要先[创建一个账号](https://blackfire.io/signup)）：

```
blackfire --slot=7 upload /path/to/profile.prof
```

\###优化器扩展`potimizer`

优化器扩展 `optimizer` 在编译前优化节点树：

```
$twig->addExtension(new Twig_Extension_Optimizer());
```

默认地，所有优化项都是开启了的。你可以通过将某些你想要启用的优化项传递给构造函数，以开启它们：

```
$optimizer = new Twig_Extension_Optimizer(Twig_NodeVisitor_Optimizer::OPTIMIZE_FOR);

$twig->addExtension($optimizer);
```

Twig 支持以下优化项：

- `Twig_NodeVisitor_Optimizer::OPTIMIZE_ALL`, 启用所有优化项（这是默认值）
- `Twig_NodeVisitor_Optimizer::OPTIMIZE_NONE`, 禁用所有优化项。这会减少编译时间，但会增加执行时间和内存消耗。
- `Twig_NodeVisitor_Optimizer::OPTIMIZE_FOR`, 如果可以，则通过移除`loop`变量的创建来优化`for`标签。
- `Twig_NodeVisitor_Optimizer::OPTIMIZE_RAW_FILTER`, 如果可能，则移除 `raw` 过滤器。
- `Twig_NodeVisitor_Optimizer::OPTIMIZE_VAR_ACCESS`, 如果可能，则简化已编译模板中变量的创建和访问。

### 3.3 异常

Twig 可以抛出异常：

- `Twig_Error`: 所有错误的基础异常。
- `Twig_Error_Syntax`: 抛出此异常，表示模板语法存在问题。
- `Twig_Error_Runtime`: 在运行时刻（runtime）重现了某个错误，则抛出这个异常（比如某个过滤器并不存在）
- `Twig_Error_Loader`: 在模板加载过程中重现了某个错误，则抛出这个异常。
- `Twig_Sandbox_SecurityError`: 在沙盒模式模板中调用了某个未被允许的标签、过滤器或方法时，抛出这个异常。

---

## 4.标签 tags

### 4.1 autoescape

无论自动转义是否启用，都可以使用`autoescape`标签，标注是否将模版的某一节进行转义：

```twig
{# 以下语法在 Twig 1.8 以上可用 #}
{% autoescape %}
  此处内容以HTML转义策略进行自动转义
{% endautoescape %}
{% autoescape 'html' %}
  此处内容以HTML转义策略进行自动转义
{% endautoescape %}
{% autoescape 'js' %}
  此处内容以JS转义策略进行自动转义
{% endautoescape %}
{% autoescape false %}
  此处的内容以原本的样子输出，不转义
{% endautoescape %}
```

> <font color=orange>Reminder：</font>在Twig v1.8前，语法有所不同。

> ```twig
> {% autoescape true %}
> 	此处内容以HTML转义策略进行自动转义
> {% endautoescape %}
> {% autoescape false %}
> 	此处的内容以原本的样子输出，不转义
> {% endautoescape %}
> {% autoescape true js %}
> 	此处内容以JS转义策略进行自动转义
> {% endautoescape %}
> ```

如果自动转义已被弃用，除了明确标注为安全的值外，所有东西都会默认被转义。使用`raw`过滤器在模板中标注：

```twig
{% autoescape %}
	{{ safe_value|raw }}
{% endautoescape %}
```

函数返回的模板数据（比如`macro`和`parent`函数）总是会返回安全标签。

> <font color=orange>Notice：</font>
>
> - Twig不会再次转义已由`escape`过滤器转义过的值。
>
> - Twig 不会转义静态表达式：
>
>   ```twig
>   {% set hello = "<strong>Hello</strong>" %}
>   {{ hello }}
>   {{ "<strong>world</strong>" }}
>   ```
>
>   将会被渲染为`<strong>Hello</strong><strong>world</strong>`。

### 4.2 do

> Twig v1.5 加入`do`标签。

`do`标签的工作方式很适合常规的变量表达式(`{{ ... }}`)，只是它并不打印任何东西：

```twig
{% do 1 + 2 %}
```

### 4.3 embed

`embed`标签：包含模板并扩展模板的内容。结合了 include 和 extends 的行为。

将嵌入式模板视为“微型布局骨架”。

```twig
{% embed "teasers_skeleton.twig" %}
	{# 这些block在"teasers_skeleton.twig"文件中定义，在这里覆盖 #}
  {% block left_teaser %}
  	Some content for the left teaser box
  {% endblock %}
  {% block right_teaser %}
  	Some content for the right teaser box
  {% endblock %}
{% endembed %}
```

`embed`标签将模板继承的想法带到内容片段层面。虽然模板继承支持由子模板填充生命的“文档架构”，但`embed`标签允许创建较小单元的“架构”，并在任何需要的地方重用或覆盖。

想象一下由多个HTML页面共享的基本模板，定义一个名为“content”的块：

```
┌─── page layout ─────────────────────┐
│           ┌── block "content" ──┐   │
│           │                     │   │
│           │ (child template to  │   │
│           │  put content here)  │   │
│           │                     │   │
│           └─────────────────────┘   │
└─────────────────────────────────────┘
```

某些页面（“foo”和“bar”）共享相同的内容结构 - 两个垂直堆叠的框：

```
┌─── page layout ─────────────────────┐
│           ┌── block "content" ──┐   │
│           │ ┌─ block "top" ───┐ │   │
│           │ │                 │ │   │
│           │ └─────────────────┘ │   │
│           │ ┌─ block "bottom" ┐ │   │
│           │ │                 │ │   │
│           │ └─────────────────┘ │   │
│           └─────────────────────┘   │
└─────────────────────────────────────┘
```

而其他页面（“boom”和“baz”）共享不同的内容结构 - 两个并排的框：

```
┌─── page layout ─────────────────────┐
│           ┌── block "content" ──┐   │
│           │                     │   │    
│           │ ┌ block ┐ ┌ block ┐ │   │
│           │ │"left" │ │"right"│ │   │
│           │ │       │ │       │ │   │
│           │ └───────┘ └───────┘ │   │
│           └─────────────────────┘   │
└─────────────────────────────────────┘
```

如果没有`embed`标签，有两种方法来设计模板：

1. 创建两个继承主布局模板的“中级”基础模板：一个具有垂直堆叠的框，用于“foo”和“bar”页面，另一个用于“boom”和“baz ”的并排框“页面。
2. 将 top/bottom 和 left/right block 的标记直接嵌入到每个页面模板中。

这两种方案都不利于扩展，因为它们各自都有一个主要缺点：

- 第一种解决方案确实适用于这个简化的例子。但如果添加了一个侧边栏，它可能会再次包含不同的，重复出现的内容结构，而此时就需要为所有出现的内容结构和侧边栏结构组合创建中间基础模板......等。
- 第二种解决方案涉及复制公共代码及其所有负面后果：任何更改都涉及查找和编辑所有受影响的结构副本，必须验证每个副本的正确性，副本可能会因粗心修改等而不同步。

在这种情况下，`embed`标签就派上用场了。公共布局的代码可以存在于单个基本模板中，而这两种不同的内容结构，我们称之为“微布局”，它们进入到需要嵌入的单独的模板中：

```twig
{# foo.twig #}
{% extends "layout_skeleton.twig" %}
{% block content %}
	{% embed "vertical_boxes_skeleton.twig" %}
  	{% block top %}
    	Some content for the top box
    {% endblock %}
    {% block bottom %}
    	Some content for the bottom box
    {% endblock %}
  {% endembed %}
{% endblock %}

{# vertical_boxes_skeleton.twig, 目的是提出框的HTML标记 #}
<div class="top_box">
	{% block top %}
  	Top box default content
  {% endblock %}
</div>
<div class="bottom_box">
  {% block bottom %}
  	Bottom box default content
  {% endblock %}
</div>
```

`embed`标签带有和`include`标签完全相同的参数：

```twig
{% embed "base" with {'foo': 'bar'} %}
  ...
{% endembed %}
{% embed "base" with {'foo': 'bar'} only %}
  ...
{% endembed %}
{% embed "base" ignore missing %}
	...
{% endembed %}
```

> <font color=red>Warning：</font>由于嵌入式模板没有名称，如果更改了上下文（例如，在一个HTML中嵌入 CSS / JavaScript 模板），基于模板`filename`的自动转义策略将无法正常工作。这种情况下，使用`autoescape`标签显式设置默认自动转义策略。

### 4.4 extends & block

`block`标签被用于继承，同时也用于充当占位符和替换件。block 的名称包含字母、数字、下划线，不允许使用破折号。

`extends`标签可以用于继承另一个模板。

```twig
{# base.html——一个基本模板 #}
<!DOCTYPE html>
<html>
  <head>
  	{% block head %}
    	<link rel="stylesheet" href="style.css" />
      <title>{% block title %}{% endblock %} - My Webpage</title>
    {% endblock %}
  </head>
  <body>
  	<div id="content">{% block content %}{% endblock %}</div>
    <div id="footer">
    	{% block footer %}
  			&copy; Copyright 2011 by <a href="http://domain.invalid/">you</a>.
      {% endblock %}
    </div>
  </body>
</html>
```

所有`block`标签都是告诉模板引擎子模板可以覆盖模板的部分。

**Child Template**

```twig
{# 子模板 #}
{% extends "base.html" %}
{% block title %}Index{% endblock %}
{% block head %}
	{{ parent() }}
  <style type="text/css">
  	.important { color: #336699; }
  </style>
{% endblock %}
{% block content %}
  <h1>Index</h1>
  <p class="important">
  	Welcome on my awesome homepage.
  </p>
{% endblock %}
```

> <font color=orange>Notice：</font>
>
> `extends`标签是关键——告诉模板引擎该模板“扩展”另一个模板，当模板引擎解析此模板时，它会先找父模板。
>
> `extends`标签必须是子模板中的第一个标签。
>
> 与 PHP 一样，Twig 不支持多重继承，因此每次渲染都只能调用一个`extends` 标签。
>
> 子模板未设置的 block，就用来自父模板的值替代使用。

不能在同一模板中定义多个同名`block`，因为块标签在“两个” 方向上工作。也就是说，块标记不仅提供了要填充的孔，它还定义了填充父级孔的内容。如果模板中有两个同名`block`，则该模板的父级就不知道要使用哪个块的内容。 

如果要多次打印块，则可以使用以下`block`功能：

```twig
<title>{% block title %}{% endblock %}</title>
<h1>{{ block('title') }}</h1>
{% block body %}{% endblock %}
```

**Parent Block**

可以使用`parent()`呈现 Parent Block 的内容。它返回 Parent Block 的结果：

```twig
{% block sidebar %}
	<h3>Table Of Contents</h3>
  ...
  {{ parent() }}
{% endblock %}
```

**命名块结束标签**

Twig 允许在结束标记后放置当前 Block 的名称来提高可读性：

```twig
{% block sidebar %}
	{% block inner_sidebar %}
  	...
  {% endblock inner_sidebar %}
{% endblock sidebar %}
```

**Block 嵌套和扩展**

可以嵌套 Block 用于更复杂的布局。默认情况下，Block 可以访问外部作用域中的变量：

```twig
{% for item in seq %}
	<li>{% block loop_item %}{{ item }}{% endblock %}</li>
{% endfor %}
```

**阻止快捷方式**

对于内容很少的块，可以使用快捷语法。

```twig
{% block title %}
	{{ page_title|title }}
{% endblock %}
{# 使用快捷语法达到同样效果 #}
{% block title page_title|title %}
```

**动态继承**

Twig 通过使用变量作为基础模板来支持动态继承：如果变量的值为一个 `Twig_Template`对象，Twig 将它作为 Parent Template。

```twig
{% extends some_var %}
$layout = $twig->loadTemplate('some_layout_template.twig');
$twig->display('template.twig', array('layout' => $layout));
```

Twig v1.2 支持传递模板数组。还可以提供已检查存在的模板列表，存在的第一个模板将用作父级：

```twig
{% extends ['layout.html', 'base_layout.html'] %}
```

**条件继承**

由于父模板的名称可以是任何有效的Twig表达式，因此可以使继承机制成为条件。

```twig
{% extends standalone ? "minimum.html" : "base.html" %}
```

**Block 如何工作**

Block 提供了一种方法来更改模板的某个部分的呈现方式，但它不会以任何方式干扰模板周围的逻辑。

```twig
{# base.twig #}
{% for post in posts %}
	{% block post %}
  	<h1>{{ post.title }}</h1>
    <p>{{ post.body }}</p>
  {% endblock %}
{% endfor %}
{# 如果渲染此模板，无论是否使用block标签，结果都一样 #}
{# block内的for循环只是一种可以让它被子模板覆盖的方法 #}
{# child.twig #}
{% extends "base.twig" %}
{% block post %}
  <article>
  	<header>{{ post.title }}</header>
    <section>{{ post.text }}</section>
  </article>
{% endblock %}
{# 在渲染子模板时，循环将使用子模板中定义的Block而不是基础模板中定义的Block #}
{# 执行的模板等效于以下模板 #}
{% for post in posts %}
  <article>
    <header>{{ post.title }}</header>
    <section>{{ post.text }}</section>
  </article>
{% endfor %}
```

`if`声明中包含的一个块：

```twig
{% if posts is empty %}
	{% block head %}
    {{ parent() }}
    <meta name="robots" content="noindex, follow">
  {% endblock head %}
{% endif %}
```

此模板不会条件地定义 Block，它只是使子模板可覆盖条件为`true`时呈现的内容输出。

如果要条件地显示输出，使用以下代码：

```twig
{% block head %}
	{{ parent() }}
  {% if posts is empty %}
  	<meta name="robots" content="noindex, follow">
  {% endif %}
{% endblock head %}
```

### 4.5 use

> <font color=orange>Notice：</font>水平重用是 Twig 的一个高级特性，几乎不需要常规模板。它主要用于那些不用继承而需要模板可重用的项目。

模板继承仅限于单继承，模板只能扩展另一个模板。

水平重用是实现与多重继承相同目标的一种方法，但没有那么复杂。

`use`语句告诉 Twig 将指定的 block 导入当前模板。

```twig
{% extends "base.html" %}
{# 告诉Twig将sidebar块导入当前模板 #}
{% use "blocks.html" %}
{% block title %}{% endblock %}
{% block content %}{% endblock %}
{# blocks.html #}
{% block sidebar %}{% endblock %}

{# 等同于以下代码(导入的block不会自动输出) #}
{% extends "base.html" %}
{% block sidebar %}{% endblock %}
{% block title %}{% endblock %}
{% block content %}{% endblock %}
```

> <font color=orange>Notices：</font>
>
> 如果没有继承其他的模板、没有定义 macro、并且主体为空，`use`标签只导入一个模板，但它可以使用其他的模板。
>
> 因为`use`是独立于传递给模板上下文解析的，所以模板引用不能是表达式。

主模板也可以覆盖任何导入的 block。如果模板定义的 block 和导入的 block 名称重复，则会忽略导入的 block。为避免名称冲突，可以重命名导入的块：

```twig
{% extends "base.html" %}
{# 如果模板已经定义了sidebar块，则blocks.html忽略定义的block #}
{# 为避免冲突，重命名导入的block #}
{% use "blocks.html" with sidebar as base_sidebar, title as base_title %}
{% block sidebar %}{% endblock %}
{% block title %}{% endblock %}
{% block content %}{% endblock %}
```

`parent()`函数自动确定正确的继承树，因此可以在覆盖导入模板中定义的块时使用它：

```twig
{% extends "base.html" %}
{% use "blocks.html" %}
{% block sidebar %}
	{{ parent() }}
{% endblock %}
{% block title %}{% endblock %}
{% block content %}{% endblock %}
```

> <font color=pink>Tip：</font>重命名允许模拟继承调用 Parent block。
>
> ```twig
> {% extends "base.html" %}
> 
> {% use "blocks.html" with sidebar as parent_sidebar %}
> 
> {% block sidebar %}
> {{ block('parent_sidebar') }}
> {% endblock %}
> ```
>
> <font color=orange>Notice：</font>可以在任何给定的模板中使用任意数量的`use`语句，如果两个导入的模板定义了相同的块，则会用最后的模板。

### 4.6 filter

Filter 模板数据块可以应用常规的 Twig 过滤器。只需将代码包装在`filter`标签内：

```twig
{% filter upper %}
	This text becomes uppercase
{% endfilter %}
```

链接多个过滤器：

```twig
{% filter lower|escape %}
	<strong>SOME TEXT</strong>
{% endfilter %}
{# => "&lt;strong&gt;some text&lt;/strong&gt;" #}
```

### 4.7 flush

> `flush`标签：Twig v1.5 新增。

`flush`标签告诉 Twig 刷新输出缓冲区。

```twig
{% flush %}
```

> Twig 内部使用的是 PHP `flush()`函数。

### 4.8 for

`for`用于循环遍历序列（序列可以是实现`Traversable`接口的数组或对象）中的每个项目。

```twig
<h1>Members</h1>
<ul>
	{% for user in users %}
  	<li>{{ user.username|e }}</li>
  {% endfor %}
</ul>
```

如果确实需要遍历一系列数字，则可以使用`..`运算符：

```twig
{# 打印0~10间的所有数字，当然它也可以用于字母 #}
{% for i in 0..10 %}
	* {{ i }}
{% endfor %}
```

`..`操作符可以在双方采取任何表达：

```twig
{% for letter in 'a'|upper..'z'|upper %}
  * {{ letter }}
{% endfor %}
```

**loop 变量**

在`for`循环块内部，可以访问一些特殊变量：

- `loop.first`：当前循环为第一次循环时返回`true`。
- `loop.last`：当钱循环为最后一次循环时返回`true`。

- `loop.index`：循环的当前次数（从1开始）。
- `loop.index0`：循环的当前次数（从0开始）。
- `loop.revindex`：循环剩余次数（最小值为1）。
- `loop.revindex0`：循环剩余次数（最小值为0）。
- `loop.length`：循环总数。
- `loop.parent`：被循环的数组或对象。

```twig
{% for user in users %}
	{{ loop.index }} - {{ user.username }}
{% endfor %}
```

> <font color=orange>Notice：</font>`loop.length`、`loop.revindex`、`loop.revindex0`、`loop.last`只对 PHP 数组或实现了`Countable`接口的对象可用。当使用条件循环时，它们也不可用。

在 Twig v1.2 中添加了`if`修饰符的支持。

**添加条件**

与PHP不同，它不可以在循环中`break`或`continue`。不过可以在循环期间过滤序列来跳过某项。

```twig
{# 跳过所有未处于活动状态的用户 #}
<ul>
	{% for user in users if user.active %}
  	<li>{{ user.username|e }}</li>
  {% endfor %}
</ul>
```

优点：特殊循环变量会正确计数，不会计算未遍历的用户。

> <font color=blue>Remenber：</font>使用循环条件时，不会定义像`loop.last`这样的属性。
>
> <font color=orange>Notice：</font>不建议在条件中使用`loop`变量，因为它可能不会如你所望地执行（如添加`loop.index > 4`这样的条件不起作用，因为只有当条件为`true`时索引才会递增，因此条件永远不会匹配）。

**else 语句**

如果由于序列为空而没有发生遍历，则可以使用`else`。

```twig
<ul>
	{% for user in users %}
  	<li>{{ user.username|e }}</li>
  {% else %}
  	<li><em>no user found</em></li>
  {% endfor %}
</ul>
```

**遍历键**

默认情况下，循环遍历序列的值。可以通过使用`keys`过滤器遍历序列的键：

```twig
<h1>Members</h1>
<ul>
	{% for key in users|keys %}
  	<li>{{ key }}</li>
  {% endfor %}
</ul>
```

**遍历键和值**

```twig
<h1>Members</h1>
<ul>
	{% for key, user in users %}
  	<li>{{ key }}: {{ user.username|e }}</li>
  {% endfor %}
</ul>
```

**遍历子集**

`slice`过滤器：遍历值的子集。

```twig
<h1>Top Ten Members</h1>
<ul>
	{% for user in users|slice(0, 10) %}
  	<li>{{ user.username|e }}</li>
  {% endfor %}
</ul>
```

### 4.9 if

`if`语句在 Twig 语句相当于 PHP 的`if`语句。它的基本功能：

1. 测试表达式的计算结果是否为真：

   ```twig
   {% if online == false %}
   	<p>Our website is in maintenance mode. Please come back later.</p>
   {% endif %}
   ```

2. 测试数组是否为空：

   ```twig
   {% if users %}
   	<ul>
     	{% for user in users %}
       	<li>{{ user.username|e }}</li>
       {% endfor %}
     </ul>
   {% endif %}
   ```

> <font color=orange>Notice：</font>如果要测试变量是否定义，用`if variable is defined`。

`not`用来测试表达式计算结果为`false`的值：

```twig
{% if not user.subscribed %}
	<p>You are not subscribed to our mailing list.</p>
{% endif %}
```

多条件判断，可以使用`and`和`or`：

```twig
{% if temperature > 18 and temperature < 27 %}
	<p>It's a nice day for a walk in the park.</p>
{% endif %}
```

多分支条件判断，可以像 PHP 一样使用`elseif`、`else`。

```twig
{% if kenny.sick %}
	Kenny is sick.
{% elseif kenny.dead %}
	You killed Kenny! You bastard!!!
{% else %}
	Kenny looks okay --- so far
{% endif %}
```

> <font color=orange>Notice：</font>判断表达式为真还是为假的规则和 PHP 一样。下面是边缘情况规则：
>
> ```
> ====================== ====================
> 值                      Boolean值
> ====================== ====================
> 空字符串                 false
> 空格字符串               true
> 空数组                   false
> 非空数组                 true
> 数字零                   false
> null                    false
> 对象                    true
> ====================== =====================
> ```



### 4.10  macro & import & from

`macro`相当于常规编程语言中的函数。它们可用于将经常使用的 HTML 惯用语法放入可重用的元素中，以免重复。

```twig
{# macro呈现表单元素示例 #}
{% macro input(name, value, type, size) %}
	<input type="{{ type|default('text') }}" name="{{ name }}" value="{{ value|e }}" size="{{ size|default(20) }}" />
{% endmacro %}
```

`macro`与原生 PHP 函数的区别体现在几个方面：

- 默认参数值是通过使用 macro 体中的`default`过滤器定义的；
- macro 的参数始终是可选的；
- 如果将额外的位置参数传递给 macro，它们最终会在特殊`varargs`变量中作为值列表。

与 PHP 函数一样，macro 无法访问当前模板的变量。

> <font color=blue>Tip：</font>通过使用特殊的`_context`变量将整个上下文环境作为参数传递

macro 可以在任何模板中定义，在使用之前需要用`import`导入。

macro 导入到模板有两种方法：

1. 将完整模板导入变量；
2. 将模板中的名称导入当前命名空间。

```twig
{# forms.html => 渲染表单的辅助模块 #}
{% macro input(name, value, type, size) %}
    <input type="{{ type|default('text') }}" name="{{ name }}" value="{{ value|e }}" size="{{ size|default(20) }}" />
{% endmacro %}
{% macro textarea(name, value, rows, cols) %}
    <textarea name="{{ name }}" rows="{{ rows|default(10) }}" cols="{{ cols|default(40) }}">{{ value|e }}</textarea>
{% endmacro %}
{# 1.在其他模板中将渲染表单辅助的整个模块作为一个变量导入 #}
{% import 'forms.html' as forms %}
<dl>
	<dt>Username</dt>
	{# 将模块中的macro作为变量的属性使用 #}
  <dd>{{ forms.input('username') }}</dd>
  <dt>Password</dt>
  <dd>{{ forms.input('password', null, 'password') }}</dd>
</dl>
<p>{{ forms.textarea('comment') }}</p>
{# 2.将模板中的名称导入当前命名空间 #}
{% from 'forms.html' import input as input_field, textarea %}
<dl>
	<dt>Username</dt>
  {# 以别名形式调用 #}
  <dd>{{ input_field('username') }}</dd>
  <dt>Password</dt>
  <dd>{{ input_field('password', '', 'password') }}</dd>
</dl>
<p>{{ textarea('comment') }}</p>
```

> <font color=pink>Tip：</font>要在当前文件的一个 macro 导入当前文件的另一个 macro，可以对源文件使用`_self`变量，在本地导入。
>
> ```twig
> {% macro input(name, value, type, size) %}
> 	<input type="{{ type|default('text') }}" name="{{ name }}" value="{{ value|e }}" size="{{ size|default(20) }}" />
> {% endmacro %}
> {% macro wrapped_input(name, value, type, size) %}
> 	{% import _self as forms %}
>   <div class="field">
>   	{{ forms.input(name, value, type, size) }}
>   </div>
> {% endmacro %}
> ```

### 4.11 include

`include`语句 include 一个模板文件，并将该文件的呈现内容返回到当前命名空间：

```twig
{% include 'header.html' %}
Body
{% include 'footer.html' %}
```

include 的模板可以访问当前上下文的变量。如果使用的是文件系统加载程序，则会在其定义的路径中查找模板。可以通过在`with`关键字后传递其他变量来添加：

```twig
{# template.html能够访问当前上下文以及提供的其他变量 #}
{% include 'template.html' with {'foo': 'bar'} %}
{% set vars = {'foo': 'bar'} %}
{% include 'template.html' with vars %}
```

通过附加`only`关键字，禁用对上下文的访问：

```twig
{# 只有foo变量是可访问的 #}
{% include 'template.html' with {'foo': 'bar'} only %}
{# 没有变量可访问 #}
{% include 'template.html' only %}
```

> <font color=pink>Tip：</font>当 include 由最终用户创建的模板时，应该考虑沙箱化它。

模板名称可以是任何有效的Twig表达式：

```twig
{% include some_var %}
{% include ajax ? 'ajax.html' : 'not_ajax.html' %}
```

如果表达式求值结果为一个`Twig_Template`对象，Twig 将直接使用它：

```twig
{% include template %}
$template = $twig->loadTemplate('some_template.twig');
$twig->loadTemplate('template.twig')->display(array('template' => $template));
```

如果要包含的模板不存在，Twig会忽略用`ignore missing`标记 的include 语句。这个标记必须放在模板名称后面。

```twig
{% include 'sidebar.html' ignore missing %}
{% include 'sidebar.html' ignore missing with {'foo': 'bar'} %}
{% include 'sidebar.html' ignore missing only %}
```

可以 include 已检查存在的模板列表。

```twig
{# 将include第一个存在的模板 #}
{% include ['page_detailed.html', 'page.html'] %}
```

当没有模板存在时，如果有`ignore missing`标记，它将回退到什么都不呈现，否则它将抛出异常。

### 4.12 sandbox

当没有为 Twig 环境全局启用沙盒时，`sandbox`标签可用于为 include 的模板启用沙盒模式：

```twig
{% sandbox %}
	{% include 'user.html' %}
{% endsandbox %}
```

> <font color=red>Warning：</font>`sandbox`标签仅在启用沙箱扩展时可用。
>
> <font color=orange>Notice：</font>`sandbox`标签只能用于包裹一个`include`标签在沙箱中，不能用于包裹模板的部分内容在沙箱中。
>
> ```twig
> {% sandbox %}
> 	{% for i in 1..2 %}
>   	{{ i }}
>   {% endfor %}
> {% endsandbox %}
> ```

### 4.13 set

在代码块内，可以使用`set`标签为变量赋值，在一个 set 块内可以给多个变量赋值。在`set`调用后，`set`内定义的变量可以跟模板其他的变量一样使用。

```twig
{% set foo = 'bar' %}
{{ foo }}
{# 一个set块中定义多个变量 #}
{% set foo, bar = 'foo', 'bar' %}
{# 相当于 #}
{% set foo = 'foo' %}
{% set bar = 'bar' %}
```

指定的值可以是任何有效的 Twig 表达式。

```twig
{% set foo = [1, 2] %}
{% set foo = {'foo': 'bar'} %}
{% set foo = 'foo' ~ 'bar' %}
```

`set`标签也可以用于文本的“捕捉”块：

```twig
{% set foo %}
	<div id="pagination">
  	...
  </div>
{% endset %}
```

> <font color=red>Warning：</font>如果启用自动输出转义，Twig 只会在捕获文本块时认为内容是安全的。
>
> <font color=orange>Notice：</font>注意在 Twig 中循环的作用域，在`for`循环中声明的变量在循环之外是不可访问的。
>
> ```twig
> {% for item in list %}
> 	{% set foo = item %}
> {% endfor %}
> {# 在for循环外的区域foo是不可访问的 #}
> {# 如果要在外部访问这个变量，就要在for循环之前定义这个变量 #}
> {% set foo = "" %}
> {% for item in list %}
> 	{% set foo = item %}
> {% endfor %}
> {# 现在在外部可访问foo了 #}
> ```

### 4.14 spaceless

`spaceless`标签可以删除*HTML元素*之间的空格，而不是HTML标记中的空格或纯文本中的空格：

```twig
{% spaceless %}
	<div>
  	<strong>foo</strong>
  </div>
{% endspaceless %}
{# 输出=> <div><strong>foo</strong></div> #}
```

这个标签并不是“优化”生成的 HTML 内容大小，只是为了避免HTML标记之间的额外空格而导致浏览器某些情况下的渲染问题。

> <font color=pink>Tips：</font>
>
> 如果想优化生成的 HTML 内容，可以使用 gzip 压缩输出。
>
> 如果想要创建一个标签来真正删除 HTML 字符串中的额外空格，请注意这并不像看上去那么简单。建议使用像 Tidy 这样的第三方库。

### 4.15 verbatim

>  Twig v1.12 之前`verbatim`叫做`raw`，工作方式完全相同，重命名是为了避免与`raw`过滤器混淆。

`verbatim`标签的功能是：阻止模板引擎的编译。

```twig
{% verbatim %}
	<ul>
  	{% for item in seq %}
    	<li>{{ item }}</li>
    {% endfor %}
  </ul>
{% endverbatim %}
```

---

## 5.过滤器

Twig 过滤器分为2种：借用自PHP自带函数的过滤器、内建过滤器。

### 5.1 借用 PHP 自带函数的过滤器

借用 PHP 自带函数的过滤器有：[abs](#abs)、[capitalize](#capitalize)、[title](#title)、[lower](#lower)、[upper](#upper)、[nl2br](#nl2br)、[format](#format)、[trim](#trim)、[join](#join)、[split](#split)、[striptags](#striptags)、[convert_encoding](#convertencoding)、[json_encode](#jsonencode)、[url_encode](#urlencode)、[date](#date)、[number_format](#numberformat)、[keys](#keys)、[length](#length)、[merge](#merge)、[reverse](#reverse)、[slice](#slice)、[sort](#sort)。

1. **abs：**<a id="abs"> </a>返回绝对值。

   > 实际上`abs`过滤器用的是 PHP 的`abs()`函数。

2. **capitalize：**<a id="capitalize"> </a>将字符串的首字母大写。

3. **title：**<a id="title"> </a>将字符串中每个单词的首字母大写，其他字母全部小写。

   ```twig
   {{ 'my first car'|title }}
   {# => 'My First Car' #}
   ```

4. **lower：**<a id="lower"> </a>将字符串所有字母全部变成小写。

5. **upper：**<a id="upper"> </a>将字符串所有字母全部变成大写。

6. **nl2br：**<a id="nl2br"> </a>将字符串里的`\n`替换成`<br/>`。

7. **format：**<a id="format"> </a>通过替换占位符来格式化一个字符串，近似于`printf` 。

   ```twig
   {% set foo = 'foo' %}
   {{ "I like %s and %s."|format(foo, "bar") }}
   {# => I like foo and bar. #}
   ```

8. **trim：**<a id="trim"> </a>去除字符串首尾的指定字符，默认为空格。

9. **join：**<a id="join"> </a>将数组的各个元素按指定分隔符组成字符串。

   元素分隔符默认为空字符串，第一个参数(可选)用于定义分隔符。

   ```twig
   {{ [1, 2, 3]|join(' | ') }}
   {# => 1 | 2 | 3 #}
   ```

10. **split：**将字符串分割成数组。

    第一个参数指定分隔符。第二个参数指定分割的每组字符串长度，如果这个值为正，则返回的数组最大长度为这个值，数组最后一个元素包含剩余所有字符串；如果这个值为负，除最后一个限制数的元素其他都返回；如果这个值为0，将其视为1。

    ```twig
    {% set foo = "one,two,three,four,five"|split(',', 3) %}
    {# foo contains ['one', 'two', 'three,four,five'] #}
    ```

    如果分隔符是一个空字符串，则字符串会被相等的块分割，长度由`limit`参数设置（默认为1个字符）。

    ```twig
    {% set foo = "123"|split('') %}
    {# foo = ['1', '2', '3'] #}
    {% set bar = "aabbcc"|split('', 2) %}
    {# bar = ['aa', 'bb', 'cc'] #}
    ```

11. **striptags：**<a id="striptags"> </a>去除字符串中的 HTML/PHP 标记。

12. **convert_encoding：**<a id="convertencoding"> </a>编码转换。第一个参数指定转换后的编码类型，第二个参数指定转换前的编码类型。

    ```twig
    {{ data|convert_encoding('UTF-8', 'iso-2022-jp') }}
    ```

    > 这个过滤器依赖于`iconv_×× / mbstring_××`，所以必须安装其中一个，如果两个都安装了，则默认使用`mbstring_××`。

13. **json_encode：**<a id="joinencode"> </a>编码 JSON 格式。

    > 实际上`json_encode`过滤器用的是 PHP 的`json_encode()`函数。

    ```twig
    {{ data|json_encode() }}
    ```

    参数可传递 `json_encode`选项的位掩码。	

14. **url_encode：**<a id=urlencode> </a>编码链接字符串。

15. **date：**<a id="date"> </a>格式化时间，可处理与`strtotime` 兼容的字符串，或 `DateTime`/`DateInterval`实例。

    第一个参数指定时间格式，格式说明符与 date 支持的格式说明符相同。要转义日期格式中的单词和字符，要在每个字符前使用`\\`。

    ```twig
    {{ post.published_at|date("F jS \\a\\t g:ia") }}
    ```

    第二个参数（可选）指定时区，如果所修饰的数据为空则默认为当前时间。默认情况下通过应用默认时区（在 php.ini 中指定的或 Twig 中声明的时区）来显示日期，也可以显式指定时区来覆盖它：

    ```twig
    {{ post.published_at|date("m/d/Y", "Europe/Paris") }}
    ```

    如果日期已经是 DateTime 对象，并且如果要保留其当前时区，则传递`false`为时区值。

    也可以通过调用全局设置默认时区`setTimezone()`：

    ```php
    $twig = new Twig_Environment($loader);
    $twig->getExtension('core')->setTimezone('Europe/Paris');
    ```

16. **number_format：**<a id="numberformat"> </a>格式化数值。

    > 它是 PHP `number_format()`函数的包装。

    第1个参数指定要显示的小数位数，默认为0；第2个参数指定表示小数点的字符，默认为`.`；第3个参数指定表示千分位的字符，默认为`,`。

17. **keys：**<a id="keys"> </a>将对象的全部键名提取成一个数组。常用于遍历对象的键名。

    ```twig
    {% for key in array|keys %}
    	...
    {% endfor %}
    ```

18. **length：**<a id="length"> </a>返回数组元素的个数或字符串的长度，等同于 `count` 和 `strlen` 的结合体。

19. **merge：**<a id="merge"> </a>合并两数组` {{ 数组1|merge(数组2) }}`）。

20. **reverse：**<a id="reverse"> </a>反转一个数组或字符串。

    ```twig
    {% for user in users|reverse %}
    	...
    {% endfor %}
    {{ '1234'|reverse }}
    {# => '4321' #}
    ```

    > <font color=pink>Tip：</font>对于类数组对象，不会保留它的数字键，可以通过传入`true`来保留数字键和值的对应。
    >
    > ```twig
    > {% for key, value in {1: "a", 2: "b", 3: "c"}|reverse %}
    > 	{{ key }}: {{ value }}
    > {%- endfor %}
    > {# => {1: 'c', 2: 'b', 3: 'a'} #}
    > {% for key, value in {1: "a", 2: "b", 3: "c"}|reverse(true) %}
    > 	{{ key }}: {{ value }}
    > {%- endfor %}
    > {# => {3: 'c', 2: 'b', 1: 'a'} #}
    > ```
    >
    > 它也适用于实现对象。

21. **slice：**<a id="slice"> </a>截取数组或字符串的一部分，在 array_slice 的基础上增加了对字符串的处理。

22. **sort：**<a id="sort"> </a>对数组排序。使用的是 PHP 的`asort`函数。

### 5.2 内建过滤器

Twig 内建的过滤器有：[batch](#batch)、[date_modify](#datemodify)、[default](#default)、[escape](#escape)、[first](#first)、[last](#last)、[replace](#replace)、[raw](#raw)。

1. **batch：**<a id="batch"> </a>将数组按指定的个数分割成更小的数组。第2个参数(可选)用来在元素不够的情况下进行填充。

   ```twig
   {% set items = ['a', 'b', 'c', 'd', 'e', 'f', 'g'] %}
   <table>
   {% for row in items|batch(3, 'No item') %}
   	<tr>
   	{% for column in row %}
   		<td>{{ column }}</td>
   	{% endfor %}
   	</tr>
   {% endfor %}
   </table>
   ```

   最后实现的 html：

   ```html
   <table>
     <tr>
       <td>a</td>
       <td>b</td>
       <td>c</td>
     </tr>
     <tr>
       <td>d</td>
       <td>e</td>
       <td>f</td>
     </tr>
     <tr>
       <td>g</td>
       <td>No item</td>
       <td>No item</td>
     </tr>
   </table>
   ```

2. **date_modify：**<a id="datemodify"> </a>修改时间，常与 date 联用。

   ```twig
   {{ post.published_at|date_modify("+1 day")|date("m/d/Y") }}
   ```

   传入的参数必须是`strtotime`函数支持格式的字符串或`DateTime`实例。

3. **default：**当所修饰的数据不存在或为空时，提供默认值。

   ```twig
   {{ var|default('var is not defined') }}
   {{ var.foo|default('foo item on var is not defined') }}
   {{ var['foo']|default('foo item on var is not defined') }}
   {{ ''|default('passed var is empty')  }}
   ```

4. **escape：**<a id="escape"> </a>将字符串安全地处理成合法的指定数据，可简写为 e，支持多种转换模式，默认模式为 `html`，其他可选模式有`html_attr`、`js`、`css`、`url`。

   ```twig
   {{ user.username|e }}
   {{ user.username|e('js') }}
   ```

5. **first：**<a id="first"> </a>返回数组的第一个元素或字符串的第一个字符。

   ```twig
   {{ [1, 2, 3, 4]|first }}
   {# => 1 #}
   {{ { a: 1, b: 2, c: 3, d: 4 }|first }}
   {# => 1 #}
   {{ '1234'|first }}
   {# => 1 #}
   ```

   > 它也适用于实现了`Traversable`接口的对象。

6. **last：**<a id="last"> </a>返回数组的最后一个元素或字符串的最后一个字符。

   ```twig
   {{ [1, 2, 3, 4]|last }}
   {# => 4 #}
   {{ { a: 1, b: 2, c: 3, d: 4 }|last }}
   {# => 4 #}
   {{ '1234'|last }}
   {# => 4 #}
   ```

   > 它也适用于实现了`Traversable`接口的对象。

7. **replace：**<a id="replace"> </a>替换一个字符串中的指定内容。

   ```twig
   {{ "I like %this% and %that%."|replace({'%this%': foo, '%that%': "bar"}) }}
   ```

8. **raw：**让数据在 `autoescape` 过滤器里失效。

   ```twig
   {% autoescape %}
   	{{ var|raw }} {# var不会被转义 #}
   {% endautoescape %}
   ```

   > <font color=blue>Notice：</font>在表达式中使用`raw`过滤器时要小心
   >
   > ```twig
   > {% autoescape %}
   >   {% set hello = '<strong>Hello</strong>' %}
   >   {% set hola = '<strong>Hola</strong>' %}
   >   {{ false ? '<strong>Hola</strong>' : hello|raw }}
   >   does not render the same as
   >   {{ false ? hola : hello|raw }}
   >   but renders the same as
   >   {{ (false ? hola : hello)|raw }}
   > {% endautoescape %}
   > ```
   >
   > 上面第一个三目运算表达式没有转义：hello 被标记为安全，Twig 不会转义它。
   >
   > 第二个三目运算表达式中，即使 hello 被标记，hola 仍是不安全的。
   >
   > 第三个三目运算表达式整个被标记，最后的结果不会转义。

## 6.函数

Twig 的函数有：[attribute](#attribute)、[block](#block)、[constant](#constant)、[cycle](#cycle)、[date](#datef)、[dump](#dump)、[max](#max)、[min](#min)、[parent](#parent)、[random](#random)、[source](#source)、[template_from_string](#templatefromstring)。

1. **attribute：**<a id="attribute"> </a>访问变量的动态属性。

   ```twig
   {{ attribute(object, method) }}
   {{ attribute(object, method, arguments) }}
   {{ attribute(array, item) }}
   ```

   另外加上`defined`可以测试是否存在动态属性。

   ```twig
   {{ attribute(object, method) is defined ? 'exists' : 'not exist' }}
   ```

2. **block：**<a id="block"> </a>如果模板使用了`extends`或`use`，且要多次打印一个代码块，可以使用`block()`。

   ```twig
   <title>{% block title %}{% endblock %}</title>
   <h1>{{ block('title') }}</h1>
   {% block body %}{% endblock %}
   ```

3. **constant：**<a id="constant"> </a>从字符串或对象取得常量值。

   ```twig
   {{ some_date|date(constant('DATE_W3C')) }}
   {{ constant('Namespace\\Classname::CONSTANT_NAME') }}
   ```

4. **cycle：**<a id="cycle"> </a>循环显示一个数组的元素。

   ```twig
   {% set start_year = date() | date('Y') %}
   {% set end_year = start_year + 5 %}
   {% for year in start_year..end_year %}
   	{{ cycle(['odd', 'even'], loop.index0) }}
   {% endfor %}
   ```

   调用形式：`cycle(数组, 循环变量)`。

5. **date：**<a id="datef"> </a>格式化时间。

   ```twig
   {# 将参数转换成日期格式进行日期比较 #}
   {% if date(user.created_at) < date('-2days') %}
   	......
   {% endif %}
   ```

   参数必须是PHP支持的`date and time formats`之一。如果没有传递参数，则该函数返回当前日期。

   第二个可选参数指定时区。

   ```twig
   {% if date(user.created_at) < date('-2days', 'Europe/Paris') %}
   	...
   {% endif %}
   ```

   > <font color=blue>Notice：</font>可以通过调用全局设置默认的时区`setTimezone()`上`core`延伸的实例
   >
   > ```php
   > $twig = new Twig_Environment($loader);
   > $twig->getExtension('core')->setTimezone('Europe/Paris');
   > ```

6. **dump：**<a id="dump"> </a>在开启调试模式的情况下显示详细的变量信息。

   > <font color=blue>Notice：</font>`dump`函数在默认情况下不可用。在创建分支时必须显式添加`Twig_Extension_Debug`扩展名。
   >
   > ```php
   > $twig = new Twig_Environment($loader, array(
   >   'debug' => true,
   >   // ...
   > ));
   > $twig->addExtension(new Twig_Extension_Debug());
   > ```
   >
   > 即使启用了`dump`函数，如果不启用环境中的`debug`选项，`dump`函数也不会显示任何内容(避免在生产服务器上泄漏调试信息)。

   在HTML上下文中，使用`pre`标签包装输出以使其更易读：

   ```twig
   <pre>
   	{{ dump(user) }}
   </pre>
   ```

   可以通过将变量作为附加参数传递来调试多个变量：

   ```twig
   {{ dump(user, categories) }}
   ```

   如果不传递任何值，则将 dump 当前上下文中的所有变量。

7. **include：**包含其他模板文件。

   依次的参数：

   - template：要呈现的模板。

   - variables：传递给模板的变量。

   - with_context：是否传递当前上下文变量。

     通过设置`with_context=false`禁用对上下文的访问。

     ```twig
     {{ include('template.html', {foo: 'bar'}, with_context = false) }}
     ```

   - ignore_missing：是否忽略丢失的模板。

     如果设置了它，将会回到没有任何模板存在的情况下，否则会抛出异常。

   - sandboxed：是否沙箱化模板。包含最终创建的模板时考虑使用这个。

8. **max：**<a id="max"> </a>返回序列或一组数值的最大值。

   ```twig
   {{ max(1, 3, 2) }}
   {{ max([1, 3, 2]) }}
   ```

9. **min：**<a id="max"> </a>返回序列或一组数值的最小值。

10. **parent：**在覆盖代码片段时用于引用父片段的内容。

11. **random：**根据提供的参数类型返回一个随机数。

    - 序列中的随机项;
    - 字符串中的随机字符;
    - 0和整数参数（包括）之间的随机整数。

    ```twig
    {{ random(['apple', 'orange', 'citrus']) }}
    {{ random('ABC') }}
    {{ random() }}
    {{ random(5) }}
    ```

12. **range：**<a id="range"> </a>返回一个指定区间的数组，可指定步长，Twig 使用 `..` 作为其简用法。

    参数：

    - `low`：序列的第一个值。
    - `high`：序列的最高可能值。
    - `step`：序列元素之间的增量。

    ```twig
    {% for i in range(0, 3) %}
    	{{ i }},
    {% endfor %}
    {# 等同于 #}
    {% for i in 0..3 %}
    	{{ i }},
    {% endfor %}
    ```

13. **source：**<a id="source"> </a>返回模板的内容而不呈现它。

    参数：

    - `name`：要读取的模板的名称。
    - `ignore_missing`：是否忽略丢失的模板。

    设置`ignore_missing`时，如果模板不存在，Twig将返回一个空字符串：

    ```twig
    {{ source('template.html', ignore_missing = true) }}
    ```

14. **template_from_string：**<a id="templatefromstring">根据字符串加载模板

## 7. 测试方法

测试方法有：[constant](#constant)、[defined](#defined)、[divisible by](#divisibleby)、[empty](#empty)、[null](#null)、[even](#even)、[odd](#odd)、[iterable](#iterable)、[sameas](#sameas)。

1. **constant：**和`constant`函数一样。

   ```twig
   {% if post.status is constant('Post::PUBLISHED') %}
   	the status attribute is exactly the same as Post::PUBLISHED
   {% endif %}
   ```

2. **defined：**<a id="defined"> </a>在当前作用域是否已定义。

   ```twig
   {% if foo.bar is defined %}
   ```

3. **divisible by：**<a id="divisibleby"> </a>目标数值是否能够被指定值整除，其中指定值不能为 0。

   ```twig
   {% if loop.index is divisible by(3) %}
   	...
   {% endif %}
   ```

4. **empty：**<a id="empty"> </a>是否为空。

   ```twig
   {% if foo is empty %}
   	...
   {% endif %}
   ```

5. **null：**<a id="null"> </a>是否为`null`。
6. **even：**<a id="even"> </a>是否为偶数。
7. **odd：**<a id="odd"> </a>是否为奇数。
8. **iterable**：<a id="iterable"> </a>是否为数组或可遍历对象。
9. **same as：**目标变量与指定值是否指向的是内存中的同一个地址。





