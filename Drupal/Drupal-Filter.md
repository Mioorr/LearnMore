# Drupal Twig Filters & Functions

## 1.Drupal-specific Twig functions



---

## 2.Drupal-specific Filters

### 2.1 `t`

`t`过滤器将字符串转换为当前语言或给定语言。此过滤器应该用于手动放置在将为用户显示的模板中的任何接口字符串。变量中包含的字符串可以被认为已经被翻译（这在设置变量时发生）并且应该直接输出。

`t`可以接收3个参数：

1. String-string(must) - 要翻译的文本字符串。

2. Args-array(optional) - 由占位符名称及其对应值键控的用于替换的关联数组。

3. Options-array(optional) - 附加选项的关联数组，包含以下元素。
   - langcode(默认为当前语言)：语言代码，用于转换为用于显示页面的语言之外的语言。
   - context(默认为空的上下文)：源字符所属的上下文。

```twig
{{ 'In the @key'|t({'@key': 'News'}) }}
{# 当前网站为英文版,则=> 'In the News' #}
{# 当前网站为中文版,则=> '在新闻中' #}
{# 翻译通过Drupal在网站中配置 #}
```

### 2.2 `safe_join`

`safe_join`过滤器将几个字符串连接在一起。可传入连接符。

```twig
{% set items = ['Hello!', 'How are you~', 'Nice to meet you!'] %}
{{items|safe_join(' | ')}}
{# => 'Hello! | How are you~ | Nice to meet you!' #}
```

### 2.3 `without`

`without`过滤器创建一个渲染数组的副本，并删除指定的子元素。它会保留原始可渲染数组，子元素仍然可以完整地从原始元素打印。

```twig
{{ content|without('links') }}
{# 打印content变量中除了content.links外的所有内容 #}
```

### 2.4 `drupal_escape`

`drupal_escape`过滤器是 Twig 中`escape`过滤器的替身，用于在显示 HTML 字符串前转义它们。

### 2.5 `clean_class`

`clean_class`过滤器准备一个字符串来用作有效的 HTML 类名。

### 2.6 `clean_id`

`clean_id`过滤器准备一个字符串来用作有效的 HTML ID。



























