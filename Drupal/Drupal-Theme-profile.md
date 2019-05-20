# Drupal Theme 相关配置数据文件

> 这里假设以 cityu 为根目录，THEMENAME = cityu 。

**信息文件**（`/cityu.info.yml`）

定义主题所需的元数据，并提供Drupal主题图层使用的其他可选设置。这是主题唯一需要的文件。此文件的名称确定 THEMENAME 的值。



**主题逻辑文件**（`/cityu.theme`）

包含条件逻辑的PHP文件，在通过模板文件输出变量之前处理变量的预处理。里面都是些预处理函数。



**模板文件**（in `/templates`，`*.html.yml` as suffix）

使用 Twig 和 HTML 编写的模板文件提供标记和基本表示逻辑。主题中的模板文件通常遵循特定的命名约定，并用于覆盖 Drupal 的默认标记输出。模板文件需要放在`templates/`子目录中，并且可以从那里组织成任意数量的子目录。



**库文件**（`/cityu.libraries.yml`）

定义可以由主题加载的 CSS 和 JavaScript 库。应通过资产库将所有 CSS 和 JavaScript 添加到页面中。



**断点文件**（`/cityu.breakpoints.yml`）

定义主题为 Drupal 使用的响应式设计断点。



**主题设置**（`/config/install/cityu.settings.yml`）

某些主题可能会提供默认配置或主题设置。要创建新的默认设置，通过主题中的“外观”管理UI进行更改，然后导出配置。将新的*THEMENAME.settings.yml*移动到主题的*config / install*目录中。



**CSS 、JS、图像文件**

主题使用的任何CSS，JS和图像资源，不是特定于Drupal的。在大多数情况下，在`css/`，`js/`和`images/`下创建子目录。



























