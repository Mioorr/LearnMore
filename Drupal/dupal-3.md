# <center>Drupal行为</center>

Drupal 行为（`Drupal.behaviors`）仍是核心 JS 的一部分。这些行为将在每个请求上执行（包括AJAX请求）。

> <font color=red>Notice：</font>添加依赖项`core/drupal`才能使用行为 。

```javascript
(function ($) {
  'use strict';
  Drupal.behaviors.awesome = {
    attach: function(context, settings) {
      $('main', context).once('awesome').append('<p>Hello world</p>');
    }
  };
}(jQuery));
```

上例中的`Drupal.behaviors`有什么：

- **namespace**：Drupal行为必须具有唯一的命名空间。在上例中，命名空间为awesome（Drupal.behaviors.awesome）。
- **attach**：包含应执行的实际函数。
- **context**：`context` 变量是适用于这个页面的一部分。在页面加载时，这将是整个文档。但是，在 AJAX 请求中，context 变量将只包含所有新加载的元素。
- **settings**：`settings` 变量用于将信息从PHP代码传递给 JS （`core/drupalSettings`）。
- **once**：使用`.once('awesome')`确保代码只运行一次。否则，代码将在每个 AJAX 请求中执行。它processed为maintag（<main role="main" class="awesome-processed">）添加了一个类来完成这个（core/jquery.once）。

```javascript
(function ($) {
    'use strict';
    Drupal.behaviors.awesome = {
        attach: function(context, settings) {
        },
        detach: function (context, settings, trigger) {
        }
    };
}(jQuery))
```

- **detach**：`detach`与`attach`并存。可用于从页面元素中分离已注册的行为。除了从 context 和 settings外，它还期望 trigger。trigger 是一个包含导致分离行为的字符串。可能的原因是：

    - upload（默认）：正在从 DOM 中删除 context 元素。
    - move：元素在 DOM 中移动（例如，在 tabledrag 行交换期间）。移动完成后，调用`Drupal.attachBehaviors()`，以便该行为可以撤销它为响应移动所做的操作。许多行为将不再需要做任何事情，只响应元素移动，但由于 DOM 元素移动的时候 iframe 重新加载了"src"，绑定到 iframe 元素（WYSIWYG 编辑器）的行为可能需要采取一些行动。
    - serialize：提交 AJAX 表单时，将使用表单作为上下文调用。这为表单中的每个行为提供了一个机会，可以在表单序列化之前确保字段元素在其中具有正确的内容。规范的用例是 WYSIWYG 编辑器可以更新它们绑定的隐藏文本区域。
    
