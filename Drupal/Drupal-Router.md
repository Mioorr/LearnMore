# Drupal Theme

## 1. Drupal Router

**The function  generate a link to the appropriate path: ** 

1. `url($router_name, $parameters=array(), $options=array())`：给定路径名和参数，生成绝对 URL。
2. `path($router_name, $parameters=arry(), $options=arry())`：给定路径名和参数，生成相对 URL。

```Twig
{# Link to the home page #}
<a href="{{ path('<front>') }}">{{ 'Home'|t }}</a>
{# Link to a specific node #}
<a href="{{ path('entity.node.canonical', {'node': node.id}) }}">{{ 'Read more'|t }}</a>
```

在写入路径时，当需要从其他主题或模块引用模板时，都要以`@namespace`的方式使用路径，如果在自己的主题中，则无需使用它。在这里，`@`的功能差不多就是相当于一个变量的功能。

```twig
{% extends "@classy/block/block.html.twig" %}
```

---

## 2. Placeholders

Drupal 中的占位符有3种：

1. `@variable`：（多数用例）将文本中的特殊字符转换为 HTML 实体。

   ```twig
   {{ t('Hello @name, welcome back!', array('@name' => $user->getDisplayName())); }}
   {# => Hello Dries, welcome back! #}
   ```

2. `%variable`：这个占位符通过`drupal_placeholder()`传递文本，文本被 HTML 转义，然后用`<em>`标签包装。

   ```twig
   {{ t('The file was saved to %path.', array('%path' => $path_to_file)); }}
   {# => The file was saved to <em class="placeholder">sites/default/files/myfile.txt</em>. #}
   ```

3. `:variable`：在替换`href`属性的值时使用这种占位符样式，值将被 HTML 转义并过滤危险协议。

   ```twig
   {{ t('Hello <a href=":url">@name</a>', array(':url' => 'http://example.com', '@name' => $name)); }}
   {# => Hello <a href="http://example.com">Dries</a> #}
   ```

---

## 3. AJAX

### 3.1 Drupal.ajax()

没有必要使用虚拟 DOM 元素来触发 AJAX 请求。AJAX对象有一个`execute()`方法，以编程方式触发使用 Drupal API 的 AJAX 请求。

```javascript
var ajaxObject = Drupal.ajax({url: 'some/url.json' /* ajax settings */});
ajaxObject.execute();
```

所有创建的 AJAX 对象都自动存储在`Drupal.ajax.instances`数组中

### 3.2 jQuery.ajax()



---

## 4. Drupal Object

### 4.1 Drupal.theme

`Drupal.theme`：用于在 JS 中输出 DOM 元素，这是在 JS 中输出 DOM 的最好方式。

可以在 *core/misc/drupal.js* 文件中找到`Drupal.theme`函数的定义。可以在自定义的 Theme 中自定义方法覆盖`Drupal.theme`中的方法。

```js
(function (Drupal) {
  // 覆盖Drupal.theme中的placeholder
  Drupal.theme.placeholder = function(str) {
    return `<em class="friendly-placeholder">${Drupal.checkPlain(str)}</em>`;
  }
  if (drupalSettings.friendly.name) {
    var siteName = document.getElementsByClassName('site-name')[0];
    siteName.getElementsByTagName('a')[0].innerHTML = `<h1>Howdy,${Drupal.theme('placeholder', drupalSettings.friendly.name)}!</h1>`;
  }
})(Drupal);
```

### 4.2 Drupal.checkPlain & Drupal.formatPlural

这两个方法的定义都可以在 *core/misc/drupal.js* 文件中找到。

`Drupal.checkPlain`：确保字符串可以安全地输出到 DOM 中，用于确保用户提供的 HTML 实体在显示之前都已正确编码。

```js
 Drupal.checkPlain = function (str) {
   str = str.toString()
     .replace(/&/g, '&amp;')
     .replace(/"/g, '&quot;')
     .replace(/</g, '&lt;')
     .replace(/>/g, '&gt;');
   return str;
  };
```

`Drupal.formatPlural`：确保包含项目计数的字符串以正确的复数表示。

### 4.3 Drupal.t & Drupal.formatString

`Drupal.t()`：将特定字符串转换为当前或给定的语言。

`Drupal.t(str, args, options)`：

- **str** ：(必须) 要翻译的字符串。
- **args** ：(可选) 占位符变量值。
- **options** ：(可选) 指定转换语言，以及使用该字符串的上下文。

`Drupal.formatString()`：处理原始字符串中的任何动态值的替换，`Drupal.t()`函数定义中用到它。

`Drupal.t()`第二参数有3种可以使用的占位符类型：

- `!variable`：不做任何处理。

- `@variable`：(默认，也是最常用的类型) 这种变量会在显示之前用`Drupal.checkPlain()`转义。

- `%variable`：这种变量会用`Drupal.checkPlain()`转义，再通过`Drupal.placeholder()`主题函数传递，它默认将文本包装在`<em>`标签中。

  ```html
  <em class="placeholder">text output<em>
  ```

---

## drupalSettings

Drupal Core 在 *core/core.libraries.yml* 文件中定义了一个名为 *drupalSettings* 的 JavaScript 库。定义这个库的目的是提供`drupalSettings`变量，其他模块和主题可以通过`drupalSettings`变量将 PHP 计算的的变量用于前端 JS 代码。

需要在要用到这个变量的 Module 或 Theme 的资产库(Asset Libaray)中声明依赖于 drupalSettings。 

```yaml
# friendly.libraries.yml
friendly-greeting:
  version: 1.0
  license: GPL
  js:
  	# 这里声明了整个库将包含的JS文件
    js/friendly-greeting.js: { }
  dependencies:
  	# 声明依赖
    - core/drupalSettings
```

通过 hooks 函数将资产库和自定义变量添加到页面中。

```php
/* friendly.theme, 由Drupal Console自动生成 */
function friendly_page_attachments_alter(array &$page) {
  // 传递用户的显示名称到前端代码
  $account = \Drupal::currentUser();
  if ($account->isAuthenticated()) {
    // 1.将资产库添加到页面
    $page['#attached']['library'][] = 'friendly/friendly-greeting';
    // 2.传递动态值,这在JS中以drupalSettings.friendly.name形式提供
    $page['#attached']['drupalSettings']['friendly']['name'] = $account->getDisplayName();
  }
}
```

首先，代码加载当前的 Drupal 用户，检查此请求是否来自经过身份验证的用户。如果用户已通过身份验证，会将 *friendly-greeting* 资产库加到该页面，还会将一个变量（用户的显示名称）加到`drupalSettings`对象中。数组的格式决定自定义变量的命名空间。在这种情况下，资产库中的 JS 文件才能用`drupalSettings.friendly.name`。

然后，在资产库定义的 JS 文件中使用这个变量。

```js
(function(Drupal) {
  Drupal.theme.placeholder = function(str) {
    return `<em class="friendly-placeholder">${Drupal.checkPlain(str)}</em>`;
  };
  if (drupalSettings.friendly.name) {
    const siteName = document.getElementsByClassName("site-branding__name")[0];
    siteName.getElementsByTagName("a")[0].innerHTML = 
      `<h1>Howdy, ${Drupal.theme("placeholder", drupalSettings.friendly.name)}!</h1>`;
  }
})(Drupal);
```































