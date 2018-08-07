翻译源

http://docs.grafana.org/reference/templating/#advanced-formatting-options

## Variables(变量)

变量允许有更多的仪表盘动态交互。

## Interpolation(插值)

面板标题和度量标准查询可以使用两种不同的语法来引用变量:

- `$<varname>` Example: apps.frontend.$server.requests.count
- `[[varname]]` Example: apps.frontend.[[server]].requests.count

为什么两种方式？ 第一种语法更易于读写，但不允许在单词中间使用变量。 在my.server [[serverNumber]]等表达式中使用第二种语法。

在将查询发送到数据源之前，将对查询进行插值，这意味着将变量替换为其当前值。 在插值期间，可以对变量值进行转义，以符合查询语言的语法和使用它的位置。 例如，InfluxDB或Prometheus查询中的正则表达式中使用的变量将进行正则表达式转义。 有关插值期间值转义的详细信息，请阅读数据源特定文档文章。

## Advanced Formatting Options(高级格式化选项)

> 仅适用于`Grafana`5.1以上版本

变量插值的格式取决于数据源，但在某些情况下您可能希望更改默认格式。 例如，MySql数据源的默认值是将多个值连接为逗号分隔的引号：'server01'，'server02'。 在某些情况下，您可能希望使用逗号分隔的字符串，不带引号：server01，server02。 现在可以使用高级格式化选项。

Syntax: `${var_name:option}` 

| Filter Option | Example                | Raw                | Interpolated          | Description                                                 |
| ------------- | ---------------------- | ------------------ | --------------------- | ----------------------------------------------------------- |
| `glob`        | ${servers:glob}        | `'test1', 'test2'` | `{test1,test2}`       | （默认）将多值变量格式化为glob（用于Graphite查询）          |
| `regex`       | ${servers:regex}       | `'test.', 'test2'` | `(test.|test2)`       | 将多值变量格式化为正则表达式字符串                          |
| `pipe`        | ${servers:pipe}        | `'test.', 'test2'` | `test.|test2`         | Formats multi-value variable into a pipe-separated string   |
| `csv`         | ${servers:csv}         | `'test1', 'test2'` | `test1,test2`         | Formats multi-value variable as a comma-separated string    |
| `distributed` | ${servers:distributed} | `'test1', 'test2'` | `test1,servers=test2` | Formats multi-value variable in custom format for OpenTSDB. |
| `lucene`      | ${servers:lucene}      | `'test', 'test2'`  | `("test" OR "test2")` | Formats multi-value variable as a lucene expression.        |

测试格式化选项[演示地址](http://play.grafana.org/d/cJtIfcWiz/template-variable-formatting-options?orgId=1)

如果指定了任何无效的格式化选项，则glob是默认/回退选项。

另一种语法（可能在将来不推荐使用）是[[var_name：option]]。

## Variable Options

变量显示为仪表板顶部的下拉选择框。 它具有当前值和一组选项。 选项是您可以选择的一组值。

## Adding a variable

![img](.\assets\variables_var_list.png) 

您可以通过Dashboard cogs菜单>模板添加变量。 这将打开一个变量列表和一个用于创建新变量的New按钮。

## Basic variable options

| Option  | Description                                                  |
| ------- | ------------------------------------------------------------ |
| *Name*  | 变量的名称，这是在度量标准查询中引用变量时使用的名称。 必须是唯一的，不包含空格。 |
| *Label* | 此变量的下拉列表的名称。                                     |
| *Hide*  | 隐藏下拉选择框的选项。                                       |
| *Type*  | 定义变量类型。                                               |

## Variable types

| Type             | Description                                                  |
| ---------------- | ------------------------------------------------------------ |
| *Query*          | 此变量类型允许您编写数据源查询，该查询通常返回度量标准名称，标记值或键的列表。 例如，返回服务器名称，传感器ID或数据中心列表的查询。 |
| *Interval*       | 此变量可表示时间跨度。 不是按时间或日期直方图间隔对组进行硬编码，而是使用此类型的变量。 |
| *Datasource*     | 此类型允许您快速更改整个仪表板的数据源。 如果您在不同的环境中有多个数据源实例，则非常有用。 |
| *Custom*         | 使用逗号分隔列表手动定义变量选项。                           |
| *Constant*       | 定义隐藏常量。 对于要共享的仪表板的度量标准路径前缀很有用。 在仪表板导出期间，常量变量将变为导入选项。 |
| *Ad hoc filters* | 非常特殊的变量，目前仅适用于某些数据源，InfluxDB和Elasticsearch。 它允许您添加键/值过滤器，这些过滤器将自动添加到使用指定数据源的所有度量标准查询中。 |

## Query Options

 此变量类型是最强大和最复杂的，因为它可以使用数据源查询动态获取其选项。 

| Option        | Description                                                  |
| ------------- | ------------------------------------------------------------ |
| *Data source* | 查询的数据源目标                                             |
| *Refresh*     | 控制何时更新变量选项列表（下拉列表中的值）。 **在仪表板上加载**将减慢仪表板负载，因为在初始化仪表板之前需要完成变量查询。 如果变量选项查询包**含时间范围**过滤器或取决于仪表板时间范围，则仅将此设置为“开启时间范围更改”。 |
| *Query*       | 数据源特定的查询表达式。                                     |
| *Regex*       | 正则表达式用于过滤或捕获数据源查询返回的名称的特定部分。 可选的。 |
| *Sort*        | 在下拉列表中定义选项的排序顺序。 **已禁用**表示将使用数据源查询返回的选项顺序。 |

### 使用 Regex 过滤/修改变量下拉中的值

使用“正则表达式查询选项”，可以筛选“变量”查询返回的选项列表，或修改返回的选项。

过滤以下选项列表的示例：

```
backend_01
backend_02
backend_03
backend_04
```

过滤以便仅返回以`01`或`02`结尾的选项：

Regex:

```
/.*[01|02]/
```

Result: 

```
backend_01
backend_02
```

使用正则表达式捕获组过滤和修改选项以返回部分文本：

Regex: 

```
/.*(01|02)/
```

Result: 

```
01
02
```

### 过滤和修改 - 普罗米修斯示例

List of options:

```
up{instance="demo.robustperception.io:9090",job="prometheus"} 1 1521630638000
up{instance="demo.robustperception.io:9093",job="alertmanager"} 1 1521630638000
up{instance="demo.robustperception.io:9100",job="node"} 1 1521630638000
```

Regex:

```
/.*instance="([^"]*).*/
```

Result:

```
demo.robustperception.io:9090
demo.robustperception.io:9093
demo.robustperception.io:9100
```

## Query expressions

参考网址：http://docs.grafana.org/reference/templating/#query-expressions

- Graphite

- Elasticsearch

- InfluxDB

- Prometheus

- OpenTSDB

  需要注意的一点是，查询表达式可以包含对其他变量的引用，实际上可以创建链接变量。 Grafana 将检测到这一点并在其中一个变量包含变量时自动刷新变量。

  ## Selection Options

| Option               | Description                                                  |
| -------------------- | ------------------------------------------------------------ |
| *Multi-value*        | 如果启用，该变量将支持同时选择多个选项。                     |
| *Include All option* | 添加一个特殊的`All`选项，其值包括所有选项。                  |
| *Custom all value*   | 默认情况下，`All`值将包括组合表达式中的所有选项。 这可能会变得很长并且可能会出现性能问题。 很多时候，最好指定一个自定义的所有值，比如通配符正则表达式。 为了能够在**Custom all value**选项中使用自定义正则表达式，globs或lucene语法，它永远不会被转义，因此您必须考虑数据源的有效值。 |

## Formatting multiple values

使用所选择的多个值对变量进行插值是很棘手的，因为如何将多个值格式化为在使用该变量的给定上下文中有效的字符串。 Grafana试图通过允许每个数据源插件通知模板插值引擎用于多个值的格式来解决这个问题。

请注意，变量的自定义全部值选项必须留空，以便Grafana将所有值格式化为单个字符串。

例如，Graphite使用glob表达式。在这种情况下，如果当前变量值是host1，host2和host3，则具有多个值的变量将被插值为`{host1，host2，host3}`。

InfluxDB和Prometheus使用正则表达式，因此相同的变量将被插值为`（host1 | host2 | host3）`。如果没有，每个值也将是正则表达式转义，具有正则表达式控制字符的值将破坏正则表达式。

Elasticsearch使用lucene查询语法，因此在这种情况下，相同的变量将被格式化为`（"host1"OR"host2"或"host3"）`。在这种情况下，每个值都需要进行转义，以便该值可以包含lucene控制字和引号。

## Formatting troubles

自动转义和格式化可能会导致问题，掌握它背后的逻辑可能会很棘手。 特别是对于InfluxDB和Prometheus，使用正则表达式语法要求在regex运算符上下文中使用该变量。 如果您不希望Grafana执行此操作，则自动正则表达式转义并格式化您唯一的选项是禁用“多值”或“包含所有”选项。

## Value groups/tags

如果您在多值变量的下拉列表中有很多选项。 您可以使用此功能将值分组为可选标记。

| Option             | Description                                                  |
| ------------------ | ------------------------------------------------------------ |
| *Tags query*       | 应返回标记列表的数据源查询                                   |
| *Tag values query* | 应返回指定标记键值的列表的数据源查询。 在查询中使用$ tag来引用当前选定的标记。 |

[![img](.\assets\variable_dropdown_tags.png)

## Interval variables

使用Interval类型创建表示时间跨度的变量（例如，`1m`，`1h`，`1d`）。 还有一个特殊的`auto`选项会根据当前时间范围而改变。 您可以指定当前时间范围的划分次数，以计算当前的`auto`时间跨度。

此变量类型可用作按时间分组的参数（对于InfluxDB），日期直方图间隔（对于Elasticsearch）或作为汇总函数参数（对于Graphite）。

在石墨函数中使用`Interval`类型的模板变量`myinterval`的示例：

## Global Built-in Variables

Grafana 具有全局内置变量，可以在查询编辑器中的表达式中使用。

## The $__interval Variable

这个$ __ interval变量类似于上面描述的`auto` interval变量。 它可以用作按时间分组的参数（对于InfluxDB），日期直方图间隔（对于Elasticsearch）或作为汇总函数参数（对于Graphite）。

Grafana 自动计算可用于在查询中按时间分组的间隔。 当数据点多于图表上显示的数据点时，可以通过更大的间隔分组来提高查询效率。 在查看3个月的数据时，分组比1天比10分更有效，图表看起来相同，查询会更快。 `$ __interval`使用时间范围和图形宽度（像素数）计算。

近似计算:`(from - to）/resolution`

例如，当时间范围为1小时且图形为全屏时，则可以将间隔计算为`2m` - 以2分钟为间隔对点进行分组。 如果时间范围是6个月并且图表是全屏，则间隔可能是`1d`（1天） - 点按天分组。

在InfluxDB数据源中，遗留变量`$interval`是相同的变量。 应该使用`$__interval`。

InfluxDB 和 Elasticsearch 数据源具有**按时间间隔分组**的字段，用于对间隔进行硬编码或设置`$__interval`变量的最小限制（通过使用`>`语法 - >`> 10m`）。

## The $__interval_ms Variable

此变量是$ __ interval变量，以毫秒为单位（而不是格式化字符串的时间间隔）。 例如，如果`$__interval`是`20m`，那么`$__interval_ms`是1200000。

## The $timeFilter or $__timeFilter Variable

`$timeFilter`变量将当前选定的时间范围作为表达式返回。 例如，时间范围间隔最近7天表达式是`time> now（） - 7d`。

这在InfluxDB数据源的WHERE子句中使用。 在查询编辑器模式下，Grafana会自动将其添加到InfluxDB查询中。 必须在文本编辑器模式下手动添加：`WHERE $ timeFilter`。

`$ __ timeFilter`用于MySQL数据源。

## The $__name Variable

此变量仅在Singlestat面板中可用，可以在“选项”选项卡上的前缀或后缀字段中使用。 变量将替换为系列名称或别名。

## Repeating Panels

模板变量对于在整个仪表板中动态更改查询非常有用。 如果您希望Grafana根据您选择的值动态创建新面板或行，则可以使用*Repeat*功能。

如果您有一个变量启用了`Multi-Vlue`或`Include all value`选项，您可以选择一个面板或一行，并让Grafana为每个选定的值重复该行。 您可以在面板编辑模式的“常规”选项卡下找到此选项。 选择要重复的变量和最小跨度。 最小跨度控制Grafana制作面板的小小（如果选择了多个值）。 Grafana将自动调整每个重复面板的宽度，以便填满整行。 目前，您不能将一行中的其他面板与重复面板混合使用。

仅对第一个面板（原始模板）进行更改。 要使更改在所有面板上生效，您需要触发动态仪表板重建。 您可以通过更改变量值（这是重复的基础）或重新加载仪表板来完成此操作。

## Repeating Rows

此选项要求您打开行选项视图。 将鼠标悬停在行左侧以触发行菜单，在此菜单中单击“行选项”。 这将打开行选项视图。 在这里，您可以找到*Repeat*下拉列表，您可以在其中选择要*Repeat*的变量。

## URL state

始终使用语法`var-<varname>=value`将变量值同步到URL。