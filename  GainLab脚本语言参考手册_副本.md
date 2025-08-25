# GainLab Script 脚本语言参考手册

### GainLab Script 是什么

GainLab Script 是在GainLab图表中执行的指标脚本。

脚本基于​**JavaScript**​，支持变量、函数、表达式与回调等。

建议具备JS或其它编程语言常识（如：变量/数组/对象/方法/回调等），也可借助 AI 辅助快速编写与校对。

示例：

```
// @name = MA
// @position = main
len = I.int(10, title='周期')
style = S.line('#5b8ff9', 1, 'solid')
ma = F.ma(dataList, len, 'close')
D.line(ma, style)
```

以上内容就可以在图表中绘制出一条均线。

并可以在设置面板中修改周期，均线样式，从而实现动态调整均线。

### 脚本结构

脚本由​**元数据**​、​**参数定义**​、​**样式定义**​、​**计算**​、​**绘图**​、**输出**几部分组成。

脚本中以//@x的方式定义元数据。

脚本中提供了以下方法与数据用于编写脚本：

**input** 方法定义可设置参数。可简写为大写​**I**​，（input.int与I.int功能是相同的，后面不再赘述简写功能）

**style** 方法定义可设置绘图样式。可简写为大写**S**

**draw** 方法绘制内容。可简写为大写**D**

**output** 方法输出结果。可简写为大写**O**

**formula** 公式库，用于计算指标。可简写为大写**F**

**util** 工具库，用于辅助脚本编写。可简写为大写**U**

**Math** 方法，用于数学计算。

以上内容详细说明与使用方法，请在脚本参考手册中相应内容中查看。

### 内置数据与方法

​**dataList**​，K线数据数组，为当前图表中所有k线数据。数组内容为对象格式：

* timestamp：时间戳
* open：开盘价
* high：最高价
* low：最低价
* close：收盘价
* volume：成交量
* turnover：成交额

​**visibleList**​，当前窗口内的可见K线数据数组。数组内容为对象格式与dataList相同。

​**chartSymbol**​，脚本所在图表交易对名称。如：BTCUSDT

​**chartPeriod**​，脚本所在图表周期。如：1m

​**setPrecision()**​，用于设置显示内容值与Y轴坐标的精度。值有3种形式：

* price：价格（基于当前商品的价格精度）
* volume：成交量（基于当前商品的成交量精度）
* number：数字（精度位数），如：4，表示小数点后4位

​**setMax()**​，副图方法，用于设置Y轴坐标的最大值。如不设置，则根据数据自动计算。

​**setMin()**​，副图方法，用于设置Y轴坐标的最小值。如不设置，则根据数据自动计算。

​**SOURCE**​，常量，用于设置数据源的取值常见方式，内容为数组:

* close：收盘价
* open：开盘价
* high：最高价
* low：最低价
* volume：成交量
* hl2：(最高价+最低价) / 2
* oc2：(开盘价+收盘价) / 2
* hlc3：(最高价+最低价+收盘价) / 3
* ohl3：(最高价+最低价+开盘价) / 3
* ohlc4：(最高价+最低价+开盘价+收盘价) / 4

#### 示例：

```
//@name=MA
//@title=MovingAverage 移动平均线
//@desc = ## MovingAverage 移动平均线
//作为衡量主力成本重要的参照指标，用以观察价格变动趋势和起到测试压力和支撑的作用。
//当作为趋势判断的时候，周期越长、越有效。
//另一方面，需要注意的是，当均线形成多头排列后，短期均线才是作为持仓的依据。
//@position=main
//@version=1
period = I.int(5, title="周期", min=1) 
source = I.select('close', title="数据源", tip='用来计算的数据', options=SOURCE)
maStyle = S.line('#F00',1, 'solid', title="MA样式") 
ma = F.ma(dataList, period, source)
D.line(ma,maStyle)
setPrecision('price') 
O.tools('MA'+period,ma,maStyle)
```

以上是一个比较完整的MA脚本示例。

脚本中用到的函数和方法，可以在脚本参考手册对应内容中查看。

### 脚本语法

脚本语法与JavaScript基本相同

支持变量、数组、对象等数据类型

支持if、for、foreach、while等判断循环语句

支持函数定义与调用

支持表达式与回调

支持常量

支持注释

支持模块化

支持数组、字符串等原生操作方法

支持三元运算符

支持箭头函数

支持解构赋值

支持模板字符串

如果有JS或其它编程基础，可以参考JS语法，快速上手脚本编写。

```
//@name=BBI
//@title=BBI 多空指数
//@desc=BBI（Bull and Bear Index）多空指数
//是一种将不同周期移动平均线加权平均之后的综合指标，属于均线型指标，一般选用3、6、12、24等4个参数。
//它是针对普通移动平均线MA指标的一种改进，既有短期移动平均线的灵敏，又有明显的中期趋势特征。
//@position=main
//@version=1

// 输入参数
N1 = I.int(3, title="N1", tip="第一个周期参数")
N2 = I.int(6, title="N2", tip="第二个周期参数") 
N3 = I.int(12, title="N3", tip="第三个周期参数")
N4 = I.int(24, title="N4", tip="第四个周期参数")

// 样式设置
bbiStyle = S.line('#9933FF', 1, 'solid', title="BBI样式")

// 计算BBI
// BBI = (MA(CLOSE, N1) + MA(CLOSE, N2) + MA(CLOSE, N3) + MA(CLOSE, N4)) / 4

// 计算四个周期的移动平均线
ma1 = F.ma(dataList, N1, 'close')
ma2 = F.ma(dataList, N2, 'close')
ma3 = F.ma(dataList, N3, 'close')
ma4 = F.ma(dataList, N4, 'close')

// 计算BBI = (MA1 + MA2 + MA3 + MA4) / 4
bbi = ma1.map((value, index) => {
  if (value !== null && ma2[index] !== null && ma3[index] !== null && ma4[index] !== null) {
    return (value + ma2[index] + ma3[index] + ma4[index]) / 4
  }
  return null
})

// 绘制BBI线
D.line(bbi, bbiStyle)

// 添加工具提示
O.tools('BBI', bbi, bbiStyle) 
// 设置精度
setPrecision('price')
```

### 元数据：

元数据以注释方法(//)开头，@前缀来定义脚本的各种信息。

元数据用于描述脚本的基本信息，如：名称、标题、描述、显示位置等。

```
// @name = MA
// @title = MovingAverage
// @desc = MA（MovingAverage）作为衡量主力成本重要的参照指标，是主力成本的平均值。
// 第二行描述
// ...第N行描述
// @position = main
// @version = 1
```

​**@name**​(必填) 为脚本名称，在图表工具栏、脚本列表等位置显示。

**@title** 为脚本标题，在图表设置面板、脚本列表等位置显示。如省略则使用@name的内容。

**@desc** 为脚本描述内容，用来说明脚本用途、用法等详细信息。在图表设置面板中显示。

* 以//@desc 开始第一行
* 第二行及以后为//
* 直到下一个元数据设置或//结束为止
* 支持简单markdown格式：
* # 1-6级标题
* \*\*加粗\*\*
* - 无序列表
* n. 有序列表

**@position** 为脚本显示位置，用来确定此脚本在图表中的显示位置，只有两个选项，main(主图)/vice(副图)，如不设置则默认vice。

**@version** 为脚本引擎版本号，用来确定脚本解析与运行的版本号，目前只有1。

### input 方法说明

input 是定义**可设置参数**的方法。

以input方法定义的参数，会显示在设置面板，供用户修改参数，并把最终值注入变量。

input方法可直接简写为大写​**I**​，两个方法是相同的

#### 示例：

```
period1 = I.int(10, title='周期1', min=1, max=500)
period2 = input.int(20, title='周期2', min=1, max=500)
```

以上示例中，len变量会显示在设置面板，显示为周期设置。修改面板的值后，最新值会自动注入到len中。

input方法中设置的**初始值**为​**默认值**​，用户修改后再点击恢复默认，会恢复为此值

如果不想参数显示在面板中调整，用普通变量定义即可。如:

```
period = 14
```

将不会显示在设置面板

### input.int()

定义**整数**可设置变量。

#### 语法

```
val = I.int(default, title, min, max, tip)
```

#### 参数

* default：初始值（必填,整型数值），方式:直接写值
* title：名称文本（显示在面板中的名称），如不设置，则显示变量名，方式：title='名称'
* min：最小值（可选），方式：min=1
* max：最大值（可选），方式：max=500
* tip：提示说明（可选），方式：tip='提示说明'

#### 返回值：

整数。如：14

#### 示例

```
len = I.int(14, title='周期', min=1, max=500)
```

### input.float()

定义**小数**可设置变量。

#### 语法

```
val = I.float(default, title, min, max, tip)
```

#### 参数

* default：初始值（必填，数值型），方式:直接写值
* title：名称文本（可选），方式：title='名称'
* min：最小值（可选），方式：min=1
* max：最大值（可选），方式：max=500
* tip：提示说明（可选），方式：tip='提示说明'

#### 返回值：

小数。如：0.5

#### 示例

```
ratio = I.float(0.5, title='系数', min=0, max=5, )
```

### input.bool()

定义**布尔**可设置变量。

#### 语法

```
val = I.bool(default, title, tip)
```

#### 参数

* default：初始布尔值（必填，true或false），方式:直接写值
* title：名称文本（可选），方式：title='名称'
* tip：提示说明（可选），方式：tip='提示说明'

#### 返回值：

布尔值。如：true

#### 示例

```
smooth = I.bool(true, title='是否平滑计算')
```

### input.select()

定义**下拉选择**可设置变量。

#### 语法

```
val = I.select(default, options, title, tip)
```

#### 参数

* default：初始值（必填，需在 options 列表内），方式:直接写值
* options：可选项来源（必填），方式:options=数组列表，如['MA','SMA']
  * `SOURCE` （常量，内置数据中有说明）
  * 数组字面量，如 `['MA','SMA']`
* title：名称文本（可选），方式：title='名称'
* tip：提示说明（可选），方式：tip='提示说明'

#### 返回值：

字符串。如：'close'

#### 示例

```
src = I.select('close', options=SOURCE, title='数据源')
maType = I.select('MA', options=['MA','EMA','SMA','WMA'], title='均线计算方式')
```

说明：暂不支持对象数组选项。

### style 方法说明

style 是定义**可设置样式**的方法。

以style方法定义的参数，会显示在设置面板，供用户修改样式，并把最终值注入变量。

style方法可直接简写为大写​**S**​，两个方法是相同的

#### 示例：

```
color1 = S.color('#5b8ff9', title='颜色1')
color2 = style.color('rgb(255,0,0)', title='颜色2')
```

以上示例中，变量会显示在设置面板，显示为颜色设置。修改面板的值后，最新值会自动注入到对应的变量中。

style方法中设置的**初始值**为​**默认值**​，用户修改后再点击恢复默认，会恢复为此值

如果不想样式显示在面板中调整，用普通变量定义即可。如:

```
color = '#5b8ff9'
```

将不会显示在设置面板

### style方法有两类，单独样式与组合样式

### 单独样式：

* 有color、width、size、style、full、icon方法
* 返回单独属性，可在draw方法绘图时自由组合为样式对象使用。
* 设置面板中显示单个样式设置

### 组合样式：

* 有line、bar、label、labelbg、shape、rect、srect、sshape等多个方法
* 为系统预设对应绘图方法所需的样式组合，返回样式对象，可直接使用
* 设置面板中显示多个样式设置

### 通用参数：

* title：名称文本。字符串，（可选）
* tip：提示说明。字符串，（可选）

### 其它公用参数说明：

#### position 位置（标签、图形中使用）：

* value：值（数值计算的坐标位置）
* high：最高价（最高价坐标位置）
* low：最低价（最低价坐标位置）
* open：开盘价（开盘价坐标位置）
* close：收盘价（收盘价坐标位置）
* highUp：最高价上（最高价坐标-字号\*1.5位置）
* lowDown：最低价下（最低价坐标+字号\*1.5位置）
* top：顶部（图表顶部坐标位置）
* bottom：底部（图表底部坐标位置）
* center：中心（图表中心坐标位置）

#### show 显示（组合样式中使用）

* true：显示
* false：不显示

#### 示例

```
color = S.color('#5b8ff9', title='颜色')
lineStyle = S.line('#5b8ff9', 1, 'solid',title='主线')
```

上面示例中，

"颜色":显示为一个颜色选择器

"主线:"显示为颜色选择器、线宽选择器、线型选择器组合。

### style.color()

定义**颜色**可设置变量。

在设置面板中表现为一个颜色选择器。

#### 语法

```
val = S.color(default, title, tip)
```

\*\*支持 `#RRGGBB`、`rgb()`、`rgba()`等网页标准颜色。

#### 参数

* default：初始值。字符串，（必填，颜色值），方式:直接写值
* title：名称文本。字符串，（可选），方式:title='名称'
* tip：提示说明。字符串，（可选），方式:tip='提示说明'

#### 返回值：颜色值字符串。

#### 示例

```
color = S.color('#00F',title='颜色')
```

示例中返回值为：

```
color = '#00F';
```

### style.width()

定义**线宽**可设置变量。

在设置面板中表现为一个线宽选择器。

#### 语法

```
width = S.width(default, title, tip)
```

#### 参数

* default：初始值。整数，（必填，1-6），方式:直接写值
* title：名称文本。字符串，（可选），方式:title='名称'
* tip：提示说明。字符串，（可选），方式:tip='提示说明'

#### 返回值：整型数值。

#### 示例

```
width = S.width(2,title='线宽')
```

示例中返回值为：

```
width = 2;
```

### style.size()

定义**尺寸**可设置变量。

在设置面板中表现为一个数字选择器。

#### 语法

```
size = S.size(default, title, tip)
```

#### 参数

* default：初始值。整数，（必填），方式:直接写值
* title：名称文本。字符串，（可选），方式:title='名称'
* tip：提示说明。字符串，（可选），方式:tip='提示说明'

#### 返回值：整型数值。

#### 示例

```
size = S.size(20,title='尺寸')
```

示例中返回值为：

```
size = 20;
```

### style.style()

定义**线型**可设置变量。

在设置面板中表现为一个线型选择器。

#### 语法

```
style = S.style(default, title, tip)
```

#### 参数

* default：初始值（必填，值：solid(实线)、dashed(虚线)、dotted(点线)），方式:直接写值
* title：名称文本（可选），方式:title='名称'
* tip：提示说明（可选），方式:tip='提示说明'

#### 返回值：字符串。

#### 示例

```
style = S.style('solid',title='线型')
```

示例中返回值为：

```
style = 'solid';
```

### style.full()

定义**填充/描边**可设置变量。

在设置面板中表现为一个填充类型选择器。

#### 语法

```
full = S.full(default, title, tip)
```

#### 参数

* default：初始值。字符串，（必填，值：fill(填充)、stroke(描边)），方式:直接写值
* title：名称文本。字符串，（可选），方式:title='名称'
* tip：提示说明。字符串，（可选），方式:tip='提示说明'

#### 返回值：字符串。

#### 示例

```
full = S.full('fill',title='填充')
```

示例中返回值为：

```
full = 'fill';
```

### style.icon()

定义**图形**可设置变量。

在设置面板中表现为一个图形选择器。

#### 语法

```
icon = S.icon(default, title, tip)
```

#### 图形列表

**arrowUp**上箭头

**arrowDown**下箭头

**arrowLeft**左箭头

**arrowRight**右箭头

**triangleUp**上三角

**triangleDown**下三角

**triangleLeft**左三角

**triangleRight**右三角

**circle**圆形

**square**方形

**diamond**菱形

**star**星

**cross**十字

**flag**旗子

**heart**心形

**x**x

**check**勾(对号)

#### 参数

* default：初始值。字符串，（必填，图形名称如上表），方式:直接写值
* title：名称文本。字符串，（可选），方式:title='名称'
* tip：提示说明。字符串，（可选），方式:tip='提示说明'

#### 返回值：字符串。

#### 示例

```
icon = S.icon('arrowDown',title='图形')
```

示例中返回值为：

```
icon = 'arrowDown';
```

### style.line()

定义**线条**样式可设置变量。

在设置面板中表现为：显示选择器，颜色选择器、线宽选择器、线型选择器。

#### 语法

```
style = S.line(color, size, style, show, title, tip)
```

#### 参数

* color：颜色。字符串，（必填，值同color），方式:直接写值
* size：线宽。整数，（可选，值同width，默认：1），方式:直接写值
* style：线型。字符串，（可选，值同style，默认：'solid'），方式:直接写值
* show：是否显示。布尔，（可选，默认：true），方式:直接写值
* title：名称文本。字符串，（可选），方式:title='名称'
* tip：提示说明。字符串，（可选），方式:tip='提示说明'

#### 返回值：

线型样式对象。可直接用于绘图方法中

#### 示例

```
ln1 = S.line('#5b8ff9', 1, 'solid', title='线条1', tip='线条样式')
ln2 = S.line('rgb(255,255,255)',title='线条2')
```

示例中返回值为：

```
ln1 = {"color":"#5b8ff9","size":1,"style":"solid","show":true};
ln2 = {"color":"rgb(255,255,255)","size":1,"style":"solid","show":true};
```

### style.bar()

定义**柱状**样式可设置变量。

在设置面板中表现为：显示选择器，颜色选择器、填充选择器、边框线宽选择器、边框线型选择器。

#### 语法

```
style = S.bar(color, full, size, style, show, title, tip)
```

#### 参数

* color：颜色。字符串，（必填，值同color），方式:直接写值
* full：填充类型。字符串，（可选，值同full，默认：'fill'），方式:直接写值
* size：边框线宽。整数，（可选，值同width，默认：1，只在full为stroke时有效），方式:直接写值
* style：边框线型。字符串，（可选，值同style，默认：'solid'，只在full为stroke时有效），方式:直接写值
* show：是否显示。布尔，（可选，默认：true），方式:直接写值
* title：名称文本。字符串，（可选），方式:title='名称'
* tip：提示说明。字符串，（可选），方式:tip='提示说明'

#### 返回值：

柱状样式对象。可直接用于绘图方法中

#### 示例

```
buyStyle = S.bar('#2DC08E', 'stroke', title="多柱",tip="多柱显示样式")
sellStyle = S.bar('#F92855', 'fill', title="空柱",tip="空柱显示样式")
```

示例中返回值为：

```
buyStyle = {"color":"#2DC08E","size":1,"style":"solid","full":"stroke","show":true};
sellStyle = {"color":"#F92855","size":1,"style":"solid","full":"fill","show":true};
```

### style.label()

定义**标签/文字**样式可设置变量。

在设置面板中表现为：显示选择器，颜色选择器、字号选择器、对齐选择器、位置选择器、加粗选择器、斜体选择器。

可与style.labelbg()配合使用，设置标签背景

#### 语法

```
style = S.label(color, size, align,position, bold, italic, show, title, tip)
```

#### 参数

* color：颜色。字符串，（必填，值同color），方式:直接写值
* size：字号。整数，（可选，值同size，默认：12），方式:直接写值
* align：对齐方式。字符串，（可选，值:left/center/right，默认：'left'），方式:直接写值
* position：位置。字符串，（可选，值见style方法说明，默认：'value'），方式:直接写值
* bold：加粗。字符串，（可选，值:'bold'，默认：空值为不选），方式:直接写值
* italic：斜体。字符串，（可选，值:'italic'，默认：空值为不选），方式:直接写值
* show：是否显示。布尔，（可选，默认：true），方式:直接写值
* title：名称文本。字符串，（可选），方式:title='名称'
* tip：提示说明。字符串，（可选），方式:tip='提示说明'

#### 返回值：

标签样式对象。可直接用于绘图方法中

#### 示例

```
avg = S.label('#fff', 14, 'right','bold', title='平均价')
price = S.label('#fff','bottom', title='成交量')
```

示例中返回值为：

```
avg = {color:"#fff",size:14,align:"right",bold:true,italic:false,position:"value",show:true};
price = {color:"#fff",size:12,align:"left",bold:false,italic:false,position:"bottom",show:true};
```

### style.labelbg()

定义**标签背景**样式可设置变量。

在设置面板中表现为：显示选择器，颜色选择器、填充选择器、边框线宽选择器、边框线型选择器。

#### 语法

```
style = S.labelbg(color, full, size, style, show, title, tip)
```

#### 参数

* color：颜色。字符串，（必填，值同color），方式:直接写值
* full：填充类型。字符串，（可选，值同full，默认：'fill'），方式:直接写值
* size：边框线宽。整数，（可选，值同width，默认：1，只在full为stroke时有效），方式:直接写值
* style：边框线型。字符串，（可选，值同style，默认：'solid'，只在full为stroke时有效），方式:直接写值
* show：是否显示。布尔，（可选，默认：true），方式:直接写值
* title：名称文本。字符串，（可选），方式:title='名称'
* tip：提示说明。字符串，（可选），方式:tip='提示说明'

#### 返回值：

标签背景样式对象。可与标签样式直接用于绘制标签方法中

#### 示例

```
bg = S.labelbg('rgba(0, 0, 0,0.6)','fill',title='标签背景')
```

示例中返回值为：

```
bg = {"color":"rgba(0, 0, 0,0.6)","full":"fill","size":1,"style":"solid","padding":4,"radius":0,"show":true};
```

\* 关于padding、radius在设置中没有显示的属性，在绘图中介绍。

### style.shape()

定义**图形**样式可设置变量。

在设置面板中表现为：显示选择器，颜色选择器、图形选择器、位置选择器、尺寸选择器、填充选择器。

#### 语法

```
style = S.shape(icon, color, size, position, full, show, title, tip)
```

#### 参数

* icon：图形名称。字符串，（必填，值同icon），方式:直接写值
* color：颜色。字符串，（必填，值同color），方式:直接写值
* size：尺寸。整数，（可选，值同size，默认：12），方式:直接写值
* position：位置。字符串，（可选，值见style方法说明，默认：'value'），方式:直接写值
* full：填充类型。字符串，（可选，值同full，默认：'fill'），方式:直接写值
* show：是否显示。布尔，（可选，默认：true），方式:直接写值
* title：名称文本。字符串，（可选），方式:title='名称'
* tip：提示说明。字符串，（可选），方式:tip='提示说明'

**​注意：​**在脚本中还可以设置 `x` 和 `y` 偏移参数来微调图形位置。

#### 返回值：

图形样式对象。可直接用于绘制图形方法中

#### 示例

```
buySignal = S.shape('arrowUp','#00FF00',12,'fill','lowDown',title='买入信号')
sellSignal = S.shape('arrowDown','#FF0000',12,'fill','highUp',title='卖出信号')
```

示例中返回值为：

```
buySignal = {"icon":"arrowUp","color":"#00FF00","size":12,"full":"fill","position":"lowDown","show":true};
sellSignal = {"icon":"arrowDown","color":"#FF0000","size":12,"full":"fill","position":"highUp","show":true};
```

### style.rect()

定义**矩形**样式可设置变量。

在设置面板中表现为：显示选择器，颜色选择器、填充选择器、边框线宽选择器、边框线型选择器。

#### 语法

```
style = S.rect(color, full, size, style, show, title, tip)
```

#### 参数

* color：颜色。字符串，（必填，值同color），方式:直接写值
* full：填充类型。字符串，（可选，值同full，默认：'fill'），方式:直接写值
* size：边框线宽。整数，（可选，值同width，默认：1，只在full为stroke时有效），方式:直接写值
* style：边框线型。字符串，（可选，值同style，默认：'solid'，只在full为stroke时有效），方式:直接写值
* show：是否显示。布尔，（可选，默认：true），方式:直接写值
* title：名称文本。字符串，（可选），方式:title='名称'
* tip：提示说明。字符串，（可选），方式:tip='提示说明'

#### 返回值：

图形样式对象。可直接用于绘制图形方法中

#### 示例

```
bullishFVG = S.rect('rgba(0, 255, 0, 0.3)','fill',title='看涨FVG')
bearishFVG = S.rect('rgba(255, 0, 0, 0.3)',1,'fill','solid',title='看跌FVG')
```

示例中返回值为：

```
bullishFVG = {"color":"rgba(0, 255, 0, 0.3)","size":1,"style":"solid","full":"fill","show":true};
bearishFVG = {"color":"rgba(255, 0, 0, 0.3)","size":1,"style":"solid","full":"fill","show":true};
```

### style.area()

定义**区域填充**样式可设置变量。

在设置面板中表现为：显示选择器，颜色选择器。

#### 语法

```
style = S.area(color, show, title, tip)
```

#### 参数

* color：颜色。字符串，（必填，值同color），方式:直接写值
* show：是否显示。布尔，（可选，默认：true），方式:直接写值
* title：名称文本。字符串，（可选），方式:title='名称'
* tip：提示说明。字符串，（可选），方式:tip='提示说明'

#### 返回值：

区域填充样式对象。可直接用于绘制图形方法中

#### 示例

```
trendUpFill = S.area('rgba(0, 255, 0, 0.1)', title="上涨趋势填充")
trendDownFill = S.area('rgba(255, 0, 0, 0.1)', title="下跌趋势填充")
```

示例中返回值为：

```
trendUpFill = {"color":"rgba(0, 255, 0, 0.1)","show":true};
trendDownFill = {"color":"rgba(255, 0, 0, 0.1)","show":true};
```

### style.candle()

定义**蜡烛图**样式可设置变量。

在设置面板中表现为：显示选择器，颜色选择器、填充选择器、线宽选择器。

#### 语法

```
style = S.candle(color, full, size, show, title, tip)
```

#### 参数

* color：颜色。字符串，（必填，值同color），方式:直接写值
* full：填充类型。字符串，（可选，值同full，默认：'fill'），方式:直接写值
* size：线宽。整数，（可选，值同width，默认：1），方式:直接写值
* show：是否显示。布尔，（可选，默认：true），方式:直接写值
* title：名称文本。字符串，（可选），方式:title='名称'
* tip：提示说明。字符串，（可选），方式:tip='提示说明'

#### 返回值：

蜡烛图样式对象。可直接用于绘制图形方法中

#### 示例

```
upCandle = S.candle('#00FF41', 'stroke', 1, title="上升")
downCandle = S.candle('#FF3333', 'fill', 1, title="下降")
```

示例中返回值为：

```
upCandle = {"color":"#00FF41","size":1,"style":"solid","full":"stroke","show":true};
downCandle = {"color":"#FF3333","size":1,"style":"solid","full":"fill","show":true};
```

### style.slabel()

定义**屏幕相对标签/文字**样式可设置变量。

与style.label()基本一致，但**没有 position**属性，针对于draw.slabel()方法的样式对象。

可与style.labelbg()配合使用，设置标签背景

在设置面板中表现为：显示选择器，颜色选择器、字号选择器、对齐选择器、加粗选择器、斜体选择器。

#### 语法

```
style = S.slabel(color, size, align, bold, italic, show, title, tip)
```

#### 参数

* color：颜色。字符串，（必填，值同color），方式:直接写值
* size：字号。整数，（可选，值同size，默认：12），方式:直接写值
* align：对齐方式。字符串，（可选，值:left/center/right，默认：'left'），方式:直接写值
* bold：加粗。字符串，（可选，值:'bold'，默认：空值为不选），方式:直接写值
* italic：斜体。字符串，（可选，值:'italic'，默认：空值为不选），方式:直接写值
* show：是否显示。布尔，（可选，默认：true），方式:直接写值
* title：名称文本。字符串，（可选），方式:title='名称'
* tip：提示说明。字符串，（可选），方式:tip='提示说明'

#### 返回值：

标签样式对象。可直接用于绘图方法中

#### 注意：

样式对象创建后，可在脚本中手动添加以下属性：

* text：文本内容
* x：X轴偏移
* y：Y轴偏移

#### 示例

```
labelStyle = S.slabel('#fff', 14, 'left', title='标签样式')
labelStyle.text = '提示文本'
labelStyle.x = 10
labelStyle.y = -5
```

示例中返回值为：

```
labelStyle = {"color":"#fff","size":14,"align":"left","bold":false,"italic":false,"show":true};
```

### style.sarea()

定义**屏幕相对多边形/区域**样式可设置变量。

与style.area()基本一致，但**添加 border**等属性，针对于draw.sarea()方法的样式对象。

在设置面板中表现为：显示选择器、颜色选择器、边框颜色选择器、边框线宽选择器、边框线型选择器。

#### 语法

```
style = S.sarea(color, border, size, style, show, title, tip)
```

#### 参数

* color：颜色。字符串，（可选，值同color），方式:直接写值
* border：边框颜色。字符串，（可选，值同color），方式:border='颜色值'
* size：边框线宽。整数，（可选，值同width，默认：1，只在有border时有效），方式:直接写值
* style：边框线型。字符串，（可选，值同style，默认：'solid'，只在有border时有效），方式:直接写值
* show：是否显示。布尔，（可选，默认：true），方式:直接写值
* title：名称文本。字符串，（可选），方式:title='名称'
* tip：提示说明。字符串，（可选），方式:tip='提示说明'

\*\* sarea中的**color**为填充颜色，不设置为不显示填充，只显示边框

\*\* sarea中的**border**设置格式为​**border = 颜色值**​，如不设置为不显示边框

#### 返回值：

标签样式对象。可直接用于绘图方法中

#### 示例

```
sarea1 = S.sarea('rgba(255,0,0,0.1)', 1, title='多边形1')
sarea2 = S.sarea('rgba(255,0,0,0.1)', border='#F00',1, title='多边形2')
sarea3 = S.sarea(border='#F00', 1, title='多边形3')
```

示例中返回值为：

```
sarea1 = {"color":"rgba(255,0,0,0.1)","border":null,"size":1,"style":"solid","show":true};
sarea2 = {"color":"rgba(255,0,0,0.1)","border":"#F00","size":1,"style":"solid","show":true};
sarea3 = {"color":null,"border":"#F00","size":1,"style":"solid","show":true};
```

### style.srect()

定义**屏幕相对矩形**样式可设置变量。

与style.rect()基本一致，但**添加 border**等属性，针对于draw.srect()方法的样式对象。

在设置面板中表现为：显示选择器、颜色选择器、边框颜色选择器、边框线宽选择器、边框线型选择器。

#### 语法

```
style = S.srect(color, border, size, style, show, title, tip)
```

#### 参数

* color：颜色。字符串，（可选，值同color），方式:直接写值
* border：边框颜色。字符串，（可选，值同color），方式:border='颜色值'
* size：边框线宽。整数，（可选，值同width，默认：1，只在有border时有效），方式:直接写值
* style：边框线型。字符串，（可选，值同style，默认：'solid'，只在有border时有效），方式:直接写值
* show：是否显示。布尔，（可选，默认：true），方式:直接写值
* title：名称文本。字符串，（可选），方式:title='名称'
* tip：提示说明。字符串，（可选），方式:tip='提示说明'

\*\* srect中的**color**为填充颜色，不设置为不显示填充，只显示边框

\*\* srect中的**border**设置格式为​**border = 颜色值**​，如不设置为不显示边框

#### 返回值：

标签样式对象。可直接用于绘图方法中

#### 示例

```
srect1 = S.srect('rgba(255,0,0,0.1)', 1, title='矩形1')
srect2 = S.srect('rgba(255,0,0,0.1)', border='#F00',1, title='矩形2')
srect3 = S.srect(border='#F00', 1, title='矩形3')
```

示例中返回值为：

```
srect1 = {"color":"rgba(255,0,0,0.1)","border":null,"size":1,"style":"solid","show":true};
srect2 = {"color":"rgba(255,0,0,0.1)","border":"#F00","size":1,"style":"solid","show":true};
srect3 = {"color":null,"border":"#F00","size":1,"style":"solid","show":true};
```

### style.scircle()

定义**屏幕相对圆形**样式可设置变量。

在设置面板中表现为：显示选择器、颜色选择器。

#### 语法

```
style = S.scircle(color, border, size, style, show, title, tip)
```

#### 参数

* color：颜色。字符串，（可选，值同color），方式:直接写值
* border：边框颜色。字符串，（可选，值同color），方式:border='颜色值'
* size：边框线宽。整数，（可选，值同width，默认：1，只在有border时有效）
* style：边框线型。字符串，（可选，值同style，默认：'solid'，只在有border时有效）
* show：是否显示。布尔，（可选，默认：true），方式:直接写值
* title：名称文本。字符串，（可选），方式:title='名称'
* tip：提示说明。字符串，（可选），方式:tip='提示说明'

\*\* scircle中的**color**为填充颜色，不设置为不显示填充，只显示边框

\*\* scircle中的**border**设置格式为​**border = 颜色值**​，如不设置为不显示边框

#### 返回值：

标签样式对象。可直接用于绘图方法中

#### 示例

```
scircle1 = S.scircle('rgba(255,0,0,0.1)', 1, title='圆形1')
scircle2 = S.scircle('rgba(255,0,0,0.1)', border='#F00',1, title='圆形2')
scircle3 = S.scircle(border='#F00', 1, title='圆形3')
```

示例中返回值为：

```
scircle1 = {"color":"rgba(255,0,0,0.1)","border":null,"size":1,"style":"solid","show":true};
scircle2 = {"color":"rgba(255,0,0,0.1)","border":"#F00","size":1,"style":"solid","show":true};
scircle3 = {"color":null,"border":"#F00","size":1,"style":"solid","show":true};
```

### style.sshape()

定义**屏幕相对图形**样式可设置变量。

与style.shape()基本一致，但**减少 position**属性，针对于draw.sshape()方法的样式对象。

在设置面板中表现为：显示选择器，颜色选择器、图形选择器、尺寸选择器、填充选择器。

#### 语法

```
style = S.sshape(icon, color, size, full, show, title, tip)
```

#### 参数

* icon：图形名称。字符串，（必填，值同icon），方式:直接写值
* color：颜色。字符串，（必填，值同color），方式:直接写值
* size：尺寸。整数，（可选，值同size，默认：12），方式:直接写值
* full：填充类型。字符串，（可选，值同full，默认：'fill'），方式:直接写值
* show：是否显示。布尔，（可选，默认：true），方式:直接写值
* title：名称文本。字符串，（可选），方式:title='名称'
* tip：提示说明。字符串，（可选），方式:tip='提示说明'

#### 返回值：

图形样式对象。可直接用于绘制图形方法中

#### 示例

```
buySignal = S.sshape('arrowUp','#00FF00',12,'fill',title='支撑区')
sellSignal = S.sshape('arrowDown','#FF0000',12,'fill',title='压力区')
```

示例中返回值为：

```
buySignal = {"icon":"arrowUp","color":"#00FF00","size":12,"full":"fill","show":true};
sellSignal = {"icon":"arrowDown","color":"#FF0000","size":12,"full":"fill","show":true};
```

### draw 方法说明

draw **绘制图形**的方法。

draw方法可直接简写为大写​**D**​，两个方法是相同的

### draw方法有两种对应方式，相对于K线坐标系，与相对于图表坐标系

### 相对于K线坐标系：

* 每个图形都是相对于对应的k线，图形会跟随对应k线位置自动调整
* 有line、bar、area、candle、rect、shape、label方法

### 相对图表坐标系：

* 每个图形都是相对于整个图表，不会跟随k线变化
* 有hline、vline、sshape、srect、sarea、scircle等方法

#### 示例：

```
rsiPeriod = I.int(14, title="RSI周期", min=1) 
maPperiod = I.int(10, title="MA周期", min=1) 
buyLine = I.int(70, title="超买线") 
sellLine = I.int(30, title="超卖线") 
rsiStyle = S.line('#FF6D00',title='RSI')
maStyle = S.line('#F00',title='MA')
buyLineStyle = S.line('#888','dotted',title='超买线')
sellLineStyle = S.line('#888','dotted',title='超卖线')
zero = S.line('#888','dashed',title='中轴线')
full = S.area('rgba(129, 29, 160, 0.1)',title='填充')
rsiData = F.rsi(dataList, rsiPeriod)
maData = F.ma(rsiData, maPperiod)
D.line(rsiData,rsiStyle)
D.line(maData,maStyle)
D.hline(50, zero)
D.hline(buyLine, buyLineStyle)
D.hline(sellLine, sellLineStyle)
D.srect({x1:'left',y1:buyLine,x2:'right',y2:sellLine},full)
```

以上示例中：

* 使用了相对于K线坐标系的D.line()方法，绘制了rsi线与rsi的ma线。两条线会跟随K线移动变化。
* 使用了相对于图表坐标系的D.hline()与D.srect()方法，绘制了中轴线、超买线、超卖线、填充区域。这些内容不会跟随K线变化而变化。

### 坐标设置说明：

#### K线坐标系：

* X轴为K线索引：每个绘制的内容的X坐标为对应K线索引
* Y轴为对应的数值计算出来的位置或指定位置：
* 数值计算，如：画ma线时，ma的y轴坐标为当前ma的值计算出来的y轴对应坐标
* 指定位置，如：S.shape()设置样式时，按选中的position显示位置计算y轴坐标绘制

#### 图表坐标系：

图表坐标系的x与y轴均支持多种设置方法。

#### X轴：

* 数值（正负值）：正值表示与图表左侧的距离，负值表示与图表右侧的距离，如:100表示与图表左侧距离100px，-100表示与图表右侧距离100px
* 百分比(字符串)：表示与图表宽度的百分比，如:'30%'表示与图表左侧距离30%的位置
* 字符串:'left'/'center'/'right：表示x坐标为图表左侧/中间/右侧
* 字符串(index:下标):表示为以图表中的k线位置为基准，有正数/负数/end三种形式
* 如：'index:10'/'index:-5'/'index:end'：表示x坐标对应第10根K线/倒数第5根K线/最后根K线的位置

#### Y轴：

* 数值：表示用此数值转换为y轴坐标对应的位置，如：100表示y轴对应值为100的位置
* 像素(字符串,'数值px')：表示为数值对应y轴的位置，如：'100px'/'-100px'，表示距离y轴顶部100px/距离y轴底部100px
* 百分比(字符串,'数值%')：表示为数值对应y轴的位置，如：'30%'，表示距离y轴顶部30%
* 字符串:'top'/'center'/'bottom'：表示y坐标为图表顶部/中间/底部

### 样式：

所有绘制方法都需要传递样式

#### 样式对象：

* 可以自己设置对应绘制方法中的样式对象
* 也可以使用style中的组合方法直接创建样式对象
* 各方法中有对应的样式对象结构说明

#### 样式回调函数：

k线坐标系绘制方法的样式分为样式对象与样式回调函数两种方式。

* 有时会有一组数据显示不同样式的需要，可以用回调方法来添加判断所显示的样式
* 回调方法的统一上下文入参为 `({ value, index, prev })`，value为当前值，index为当前索引，prev为前一个值
* 回调方法必须以**return**返回**样式对象**

#### 示例：

```
D.line(ma, ({ value, index, prev }) => {
  return value >= prev ? style1 : style2
})
```

以上示例中，使用了样式回调函数，根据ma线的值与前一个值的大小关系，来判断绘制线的不同样式。

### 渐变颜色：

* 绘制方法中支持渐变颜色
* 需设置color为​**数组方式**​，如：`{color:['#FF0000', '#00FF00', '#0000FF']}`
* 颜色数组时，会自动创建渐变

### draw.line()

在K线坐标系中绘制一条线。

#### 语法

```
D.line(array, style)
```

#### 参数

* array：数值数组（对应每根K线）。
* style：样式对象或样式回调函数。

#### 样式：

样式对象：{color, size, style, show, continuous}

* color：颜色。字符串，（必填）
* size：线宽。整数，（可选，默认：1）
* style：线型。字符串，（可选，默认：'solid'）
* show：是否显示。布尔，（可选，默认：true）
* continuous：是否连续。布尔，（可选，默认：false）

默认若数据为非有效数据 `null/undefined/NaN` 时将跳过该点绘制（形成​**断点**​）。

即只有相邻的两个数据都有效时才会绘制。

#### 方法示例

```
period = I.int(5, title="周期", min=1)
source = I.select('close', title="数据源", tip='用来计算的数据', options=SOURCE)
emaStyle = S.line('#F00',1, 'solid', title="EMA样式")
ema = F.ema(dataList, period, source)
D.line(ema,emaStyle)
```

以上示例为普通ema线绘制，样式可直接使用style.line()方法返回的样式对象。

#### 回调样式示例

```
period = I.int(5, title="周期", min=1)
source = I.select('close', title="数据源", tip='用来计算的数据', options=SOURCE)
maStyle1 = S.line('#F00',2, 'solid', title="MA样式1")
maStyle2 = S.line('#0FF',1, 'solid', title="MA样式2")
ma = F.ma(dataList, period, source)
D.line(ma,({value,index}){
  return value >= dataList[index].close ? maStyle1 : maStyle2
})
```

以上示例为如果ma的值大于等于对应k线数据的收盘价，则使用MA样式1，否则使用MA样式2。

### 连续线

如需绘制​**连续线**​，需设置`continuous:true`，此时会在前后两个有效值之间绘制。

如`continuous:true`时，样式回调时会添加`prevValid`（上一有效值），提供使用。

#### 连续线示例

```
buyStyle = S.line('#00FF00', 2, 'solid', true, title="多头样式")
sellStyle = S.line('#FF0000', 2, 'solid', true, title="空头样式")
buyStyle.continuous = true
sellStyle.continuous = true
D.line(zigzagData, ({ value, prevValid }) => {
  if (prevValid !== null) {
    if (value > prevValid) {
      return buyStyle
    } else if (value < prevValid) {
      return sellStyle
    }
  }
  return buyStyle
})
```

### draw.bar()

在K线坐标系中绘制柱状图。

#### 语法

```
D.bar(array, baseline, style)
```

#### 参数

* array：数值数组（对应每根K线）。
* baseline：基线值，支持多种格式：
  * 数值：如 `0`，所有柱体都以此值为基线
  * 数组：如 `[0, 10, 20, ...]`，每个数据点使用对应的基线值
  * 字符串关键字：
    * `'top'`：使用Y轴最大值作为基线
    * `'bottom'`：使用Y轴最小值作为基线
    * `'center'`：使用Y轴中间值作为基线
* style：样式对象或样式回调函数。

#### 样式：

样式对象：{color, full, size, style, show}

* color：颜色。字符串，（必填）
* full：填充方式。字符串，（可选，默认：'fill'）
* size：线宽/边框宽度。整数，（可选，默认：1）
* style：线型。字符串，（可选，默认：'solid'）
* show：是否显示。布尔，（可选，默认：true）

#### 方法示例

```
// 基础柱状图，基线为0
barStyle = S.bar('#5B8FF9', 'fill', 1, 'solid', title="柱状图样式")
D.bar(volume, 0, barStyle)

// 使用数组基线，每个柱体有不同的基线
baselineArray = [10, 20, 15, 25, 30]
D.bar(data, baselineArray, barStyle)

// 使用关键字基线
D.bar(data, 'center', barStyle)  // 以Y轴中心为基线
D.bar(data, 'bottom', barStyle)  // 以Y轴底部为基线
```

#### 回调样式示例

```
// 根据数据值动态调整柱状图颜色
D.bar(data, 0, ({value, index}) => {
  return {
    color: value > 0 ? '#00FF00' : '#FF0000',
    full: 'fill',
    size: Math.abs(value) / 10,
    show: value !== 0
  }
})

// 结合数组基线的回调样式
baselineArray = [0, 10, 20, 30]
D.bar(data, baselineArray, ({value, index}) => {
  return {
    color: value > baselineArray[index] ? '#00FF00' : '#FF0000',
    full: 'stroke',
    size: 2,
    style: 'dashed'
  }
})
```

柱状图会根据数据值与基线值的差异，从基线向上或向下延伸绘制。正值向上延伸，负值向下延伸。

### draw.area()

在K线坐标系中填充两条数据曲线围成的区域。

#### 语法

```
D.area(dataArray1, dataArray2, style)
```

#### 参数

* dataArray1：上/下边界数据（数组）
* dataArray2：另一条边界数据（数组）
* style：样式对象或样式回调函数。

#### 样式：

样式对象：{color, border, size, style, show}

* color：填充颜色。字符串，（可选）
* show：是否显示。布尔，（可选，默认：true）

### 样式回调参数

回调函数中回传参数有`{value,value2,index,prev,prev2}`提供选择

* value:第一组数据的当前值
* value2:第二组数据的当前值
* index:当前下标
* prev:第一组数据的前一个值
* prev2:第二组数据的前一个值

#### 方法示例

```
// 基础区域填充
areaStyle = S.area('#5B8FF9', title="区域样式")
D.area(F.ma(dataList, 20), F.ma(dataList, 60), areaStyle)

// 使用回调样式与自定义样式对象
D.area(data1, data2, ({value,value2}) => {
  return {
    color: value > value2 ? 'rgba(0,255,0,0.2)' : 'rgba(255,0,0,0.2)'    
  }
})
```

若两边界在某些索引无效（null/undefined/NaN），该段不填充。

### draw.candle()

绘制蜡烛K线。

#### 语法

```
D.candle(klineArray, style?)
```

#### 参数

* klineArray：K线数据数组（如系统提供的dataList），数组元素为k线数据对象
  * open:开盘价(必填)
  * close:收盘价(必填)
  * high:最高价(可选)
  * low:最低价(可选)
* style：样式对象或样式回调函数。

#### 样式：

样式对象：{color, size, style, full, show}

* color：颜色。字符串，（可选，默认：'#00FF41'）
* size：线宽。整数，（可选，默认：1）
* style：线型。字符串，（可选，默认：'solid'）
* full：填充方式。字符串，（可选，默认：'fill'）
* show：是否显示。布尔，（可选，默认：true）

#### 方法示例

```
upCandle = S.candle('#00FF41', 'fill', 1, title="上升")
downCandle = S.candle('#FF3333', 'fill', 1, title="下降")
D.candle(dataList, ({value})=>{
	return value.close < value.open ? downCandle : upCandle
})
```

以上示例为根据收盘价与开盘价判断上升或下降，使用回调样式设置蜡烛颜色。

### draw.rect()

绘制K线相对矩形，内部会根据可见区间优化渲染。

#### 语法

```
D.rect(rectData, style)
```

#### 参数

* rectData：矩形数据，形如 `{ start, end, startIndex, endIndex }`
* style：样式对象或样式回调函数。

#### 样式：

样式对象：{color, full, size, style, show}

* color：颜色。字符串，（可选）
* full： 填充方式。字符串，（可选，默认：'fill'）
* size：边框宽度。整数，（可选，默认：1，full为'stroke'时有效）
* style：边框线型。字符串，（可选，默认：'solid'，full为'stroke'时有效）
* show：是否显示。布尔，（可选，默认：true）

#### 方法示例

```
rectStyle = S.rect('#5B8FF9', 'fill', title="矩形样式")
rectData = dataList.map((kData,index)=>{
  if(kData.close < kData.open) return null
  return {
    start: kData.open,
    end : kData.close,
    startIndex: index,
    endIndex:index + 1
  }  
})
D.rect(rectData, rectStyle)
```

### draw.shape()

在K线坐标系中绘制图形标记（点状）。

#### 语法

```
D.shape(dataArray, style)
```

#### 参数

* dataArray：数组，数组元素为`有效值，非null、undefined、NAN、''（空字符串）、{}（空对象）`的元素
* style：样式对象或样式回调函数。

#### 样式：

样式对象：{color, icon, size, position, full, show, x, y}

* color：颜色。字符串，（必填）
* icon：图形类型。字符串，（可选，默认：'circle'）
* size：图形大小。整数，（可选，默认：5）
* position：位置。字符串或数值，（可选，默认：'value'）
  
  * 字符串值：'value'、'high'、'low'、'open'、'close'、'top'、'bottom'、'center'、'highUp'、'lowDown'
  * 数值：正或负值，表示为距离顶/底部的像素值
  * `position:100` 表示距离顶部100px
  * `position:-100` 表示距离底部100px
  * `position:'highUp'` 表示最高价上方
  * `position:'lowDown'` 表示最低价下方
* full：填充方式。字符串，（可选，默认：'fill'）
* show：是否显示。布尔，（可选，默认：true）
* x：X轴偏移。数值（正负值），（可选，默认：0）
* 须在脚本中设置，设置面板中没有此项
* `x:10` 表示向右偏移10px
* `x:-10` 表示向左偏移10px
* y：Y轴偏移。数值（正负值），（可选，默认：0）
* 须在脚本中设置，设置面板中没有此项
* `y:10` 表示向下偏移10px
* `y:-10` 表示向上偏移10px

#### 方法示例

```
upStyle = S.shape('circle','#339933',6,title='多方')
downStyle = S.shape('circle','#FF0033',6,title='空方')
sar = F.sar(dataList,startAf,step,maxAf)
D.shape(sar,({value,index})=>{
    return (value < (dataList[index].high + dataList[index].low) / 2) ? downStyle : upStyle
})
```

以上示例为根据SAR值判断上升或下降，使用回调样式设置图形颜色。

### draw.label()

在K线坐标系中绘制文本标签。

#### 语法

```
D.label(dataArray, style, backgroundStyle?)
```

#### 参数

* dataArray：数组，数组元素为`有效值，非null、undefined、NAN、''（空字符串）、{}（空对象）`的元素
* style：样式对象或样式回调函数。
* backgroundStyle：背景样式对象或回调函数。（可选）

#### 样式：

标签样式对象：{color,size,align,bold,italic,text,position,x,y,show}

* color：文本颜色。字符串，（必填）
* size：字体大小。整数，（可选，默认：12）
* align：对齐方式。字符串，（可选，默认：'left'）
* bold：是否粗体。布尔，（可选，默认：false）
* italic：是否斜体。布尔，（可选，默认：false）
* text：文本内容。字符串，（可选）
* position：位置。字符串或数值，（可选，默认：'value'）
  
  * 字符串值：'value'、'high'、'low'、'open'、'close'、'top'、'bottom'、'center'、'highUp'、'lowDown'
  * 数值：正或负值，表示为距离顶/底部的像素值
  * `position:100` 表示距离顶部100px
  * `position:-100` 表示距离底部100px
  * `position:'highUp'` 表示最高价上方
  * `position:'lowDown'` 表示最低价下方
* x：X轴偏移。数值（正负值），（可选，默认：0）
* 须在脚本中设置，设置面板中没有此项
* `x:10` 表示向右偏移10px
* `x:-10` 表示向左偏移10px
* y：Y轴偏移。数值（正负值），（可选，默认：0）
* 须在脚本中设置，设置面板中没有此项
* `y:10` 表示向下偏移10px
* `y:-10` 表示向上偏移10px
* show：是否显示。布尔，（可选，默认：true）

背景样式对象：{color,full,size,style,padding,radius,show}

* color：背景颜色。字符串，（可选）
* full：填充方式。字符串，（可选，默认：'fill'）
* size：边框宽度。整数，（可选，默认：1）
* style：边框线型。字符串，（可选，默认：'solid'）
* padding：内边距。整数，（可选，默认：4）
  
  * 须在脚本中设置，设置面板中没有此项
  * `padding:10` 表示内边距为10px
* radius：圆角。整数，（可选，默认：0）
* 须在脚本中设置，设置面板中没有此项
* `radius:2` 表示圆角为2px
* show：是否显示。布尔，（可选，默认：true）

### 标签内容：

* 优先级1：回调style中text对应值
* 优先级2：style中text对应值
* 优先级3：dataArray中text对应值
* 优先级4：如果style与array中都没有text字段，如果array对应值为非对象，则使用array对应值
* 如果都没有，则不绘制

#### 示例

```
labelStyle1 = S.label('rgb(0, 255, 255)',12,'left','lowDown',title='看多')
labelStyle1.y = -10
labelStyle1.x = 5
labelStyle2 = S.label('rgb(255, 0, 0)',12,'left','highUp',title='看空')
labelStyle2.y = 10
labelStyle2.x = -5
labelBg = S.labelbg('rgba(0, 0, 0,0.6)','fill',title='标签背景')
labelBg.padding = 5
labelBg.radius = 2
textData = dataList.map(item=>{
	if(item.close > item.open){
		return {text:'涨'}
	}else{
		return {text:'跌'}
	}
})
D.label(textData,({value})=>{
	return value.text === '涨'?labelStyle1:labelStyle2
},labelBg)
```

#### 示例2，使用emoji

```
labelStyle1 = S.label('rgb(0, 255, 255)',12,'left','lowDown',title='看多')
labelStyle1.y = -10
labelStyle1.x = 5
labelStyle1.text = '✔️'
labelStyle2 = S.label('rgb(255, 0, 0)',12,'left','highUp',title='看空')
labelStyle2.y = 10
labelStyle2.x = -5
labelStyle2.text = '❌'
D.label(dataList,({value})=>{
	return value.open > value.close ?labelStyle1:labelStyle2
})
```

### draw.hline()

绘制屏幕坐标系水平线。

#### 语法

```
D.hline(y, style, x1, x2)
```

#### 参数

* y：Y坐标，支持多种格式：
  * 支持单个值与数组（多条线）
  * 可参见draw方法说明中图表坐标系Y轴说明
* style：样式对象。
* x1：起始X坐标。（可选，默认：'left'），可参见draw方法说明中图表坐标系X轴说明
* x2：结束X坐标。（可选，默认：'right'），可参见draw方法说明中图表坐标系X轴说明

#### 样式：

样式对象：{color, size, style, show}

* color：颜色。字符串，（必填）
* size：线宽。整数，（可选，默认：1）
* style：线型。字符串，（可选，默认：'solid'）
* show：是否显示。布尔，（可选，默认：true）

#### 方法示例

```
// 基础水平线
lineStyle = S.line('#5B8FF9', 1, 'solid', title="水平线样式")
D.hline('center', lineStyle)

// 多条水平线
D.hline(['30%', '80%'], lineStyle)

// 指定起止位置
D.hline(100, lineStyle, 'index:-10', 'index:end')

// 使用Y轴对应值
D.hline(50, lineStyle, 'left', 'right')
```

### draw.vline()

绘制屏幕坐标系垂直线。

#### 语法

```
D.vline(x, style, y1, y2)
```

#### 参数

* x：X坐标，支持多种格式：
  * 支持单个值与数组（多条线）
  * 可参见draw方法说明中图表坐标系X轴说明
* style：样式对象。
* y1：起始Y坐标。（可选，默认：'top'），可参见draw方法说明中图表坐标系Y轴说明
* y2：结束Y坐标。（可选，默认：'bottom'），可参见draw方法说明中图表坐标系Y轴说明

#### 样式：

样式对象：{color, size, style, show}

* color：颜色。字符串，（必填）
* size：线宽。整数，（可选，默认：1）
* style：线型。字符串，（可选，默认：'solid'）
* show：是否显示。布尔，（可选，默认：true）

#### 方法示例

```
// 基础垂直线
lineStyle = S.line('#F5222D', 1, 'solid', title="垂直线样式")
D.vline('center', lineStyle)

// 多条垂直线
D.vline(['index:-20', 'index:-10', 'index:-5'], lineStyle)

// 指定起止位置
D.vline('index:end', lineStyle, 0, '100px')

// 使用像素坐标
D.vline('30%', lineStyle, 'top', 'bottom')
```

### draw.sline()

绘制屏坐标系直线段。

根据任意两点（x1,y1）与（x2,y2）绘制直线

#### 语法

```
D.sline(data, style)
```

#### 参数

* data：直线数据，对象与数组：
  * 对象：`{x1, y1, x2, y2}`
  * 数组：`[{x1, y1, x2, y2}, ...]` 绘制多条线段
  * 可参见draw方法说明中图表坐标系X轴与Y轴说明
* style：样式对象。

#### 样式：

样式对象：{color, size, style, show}

* color：颜色。字符串，（必填）
* size：线宽。整数，（可选，默认：1）
* style：线型。字符串，（可选，默认：'solid'）
* show：是否显示。布尔，（可选，默认：true）

#### 方法示例

```
// 基础直线段
lineStyle = S.line('#5B8FF9', 1, 'solid', title="直线样式")
D.sline({x1: 'center', y1: 'center', x2: 'right', y2: 'center'}, lineStyle)

// 多条直线段
D.sline([
  {x1: 100, y1: '100px', x2: 200, y2: '200px'},
  {x1: 'index:-10', y1: '30%', x2: 'index:end', y2: '70%'}
], lineStyle)
```

### draw.sarea()

绘制屏幕坐标系多边形。

#### 语法

```
D.sarea(data, style)
```

#### 参数

* data：多边形数据，支持多种格式：
  * 单个多边形：每个多边形至少有3个坐标点
    
    * `[{x:x1, y:y1}, {x:x2, y:y2},{x:x3, y:y3}, ...]`：对象形式
    * `[[x1,y1],[x2,y2],[x3,y3],...]`：数组形式
  * 多个多边形：
  * 多个多边形的数组
  * `[[{x:x1, y:y1}, {x:x2, y:y2}, ...], [{x:x1, y:y1}, {x:x2, y:y2}, ...]]`：对象形式
  * `[[[x1,y1],[x2,y2],...], [[x1,y1],[x2,y2],...]]`：数组形式
  * 可参见draw方法说明中图表坐标系X轴与Y轴说明
* style：样式对象。

#### 样式：

样式对象：{color, border, size, style, show}

* color：填充颜色。字符串，（可选）
* border：边框颜色。字符串，（可选）
* size：边框宽度。整数，（可选，默认：1）
* style：边框线型。字符串，（可选，默认：'solid'）
* show：是否显示。布尔，（可选，默认：true）

#### 方法示例

```
//带边框的多边形
areaStyle1 = S.sarea('#5B8FF9',border='#F00',1,'solid', title="多边形样式1")
//不带边框的多边形
areaStyle2 = S.sarea('#5B8FF9', title="多边形样式2")
//不带填充
areaStyle3 = S.sarea(border='#F00', title="多边形样式3")

D.sarea([
  {x: 100, y: '100px'},
  {x: 300, y: '100px'},
  {x: 300, y: '500px'}
], areaStyle2)
D.sarea([
  {x: 200, y: '200px'},
  {x: 300, y: '300px'},
  {x: 300, y: '200px'}
], areaStyle3)
// 多个多边形
D.sarea([
  [{x: 'left', y: 'top'}, {x: 'center', y: 'center'}, {x: 'right', y: 'top'}],
  [{x: 'index:-10', y: '100px'}, {x: 'index:end', y: 200}, {x: 'index:end', y: '-100px'}]
], areaStyle1)
```

### draw.srect()

绘制屏幕坐标系矩形。

#### 语法

```
D.srect(data, style)
```

#### 参数

* data：矩形数据，支持多种格式：
  * 单个矩形：`{x1, y1, x2, y2}`
  * 多个矩形：`[{x1, y1, x2, y2}, ...]`
  * 坐标支持：数值、像素、百分比、关键字、K线索引等
* style：样式对象。

#### 样式：

样式对象：{color, border, size, style, show}

* color：填充颜色。字符串，（可选）
* border：边框颜色。字符串，（可选）
* size：边框宽度。整数，（可选，默认：1）
* style：边框线型。字符串，（可选，默认：'solid'）
* show：是否显示。布尔，（可选，默认：true）

#### 方法示例

```
// 基础矩形
rectStyle = S.srect('#5B8FF9', border='#5B8FF9', 1, 'solid', title="矩形样式")
D.srect({x1: 100, y1: '100px', x2: 200, y2: '200px'}, rectStyle)

// 多个矩形
D.srect([
  {x1: 'left', y1: 'top', x2: 100, y2: '100px'},
  {x1: 'index:-10', y1: '100px', x2: 'index:end', y2: '200px'}
], rectStyle)
```

### draw.scircle()

绘制屏幕坐标系圆形。

#### 语法

```
D.scircle(data, style)
```

#### 参数

* data：圆形数据，支持多种格式：
  * 单个圆形：`{x1, y1, x2, y2}`（两点确定直径）
  * 多个圆形：`[{x1, y1, x2, y2}, ...]`
  * 坐标支持：数值、像素、百分比、关键字、K线索引等
* style：样式对象。

#### 样式：

样式对象：{color, border, size, style, show}

* color：填充颜色。字符串，（可选）
* border：边框颜色。字符串，（可选）
* size：边框宽度。整数，（可选，默认：1）
* style：边框线型。字符串，（可选，默认：'solid'）
* show：是否显示。布尔，（可选，默认：true）

#### 方法示例

```
// 基础圆形
circleStyle = S.scircle('rgba(255,0,0,0.1)', border='#F5222D', 1, 'solid', title="圆形样式")
D.scircle({x1: 200, y1: '150px', x2: 300, y2: '200px'}, circleStyle)

// 多个圆形
D.scircle([
  {x1: 50, y1: 'center', x2: 80, y2: '60%'},
  {x1: 'index:-10', y1: '100px', x2: 'index:end', y2: '120px'}
], circleStyle)
```

### draw.sshape()

绘制屏幕坐标系图形标记。

#### 语法

```
D.sshape({x,y}, shapeStyle)
```

#### 参数

* data：对象或数组；
  * 单个图形：`{x, y}`
  * 多个图形：`[{x, y}, ...]`

#### 样式：

样式对象：{color, icon, size, full, show, x, y}

* color：颜色。字符串，（必填）
* icon：图形类型。字符串，（可选，默认：'circle'）
* size：图形大小。整数，（可选，默认：10）
* full：填充方式。字符串，（可选，默认：'fill'）
* show：是否显示。布尔，（可选，默认：true）
* x：X轴偏移。数值（正负值），（可选，默认：0）
  * 须在脚本中设置，设置面板中没有此项
  * `x:10` 表示向右偏移10px
  * `x:-10` 表示向左偏移10px
* y：Y轴偏移。数值（正负值），（可选，默认：0）
* 须在脚本中设置，设置面板中没有此项
* `y:10` 表示向下偏移10px
* `y:-10` 表示向上偏移10px

#### 示例

```
shape = S.sshape('star','#52c41a', 12,title='图形1')
// 单个图形
D.sshape({x:'right',y:'bottom'}, shape)
// 多个图形
D.sshape([{x:'index:-10',y:'center'},{x:'index:end',y:'top'}], shape)
```

### draw.slabel()

绘制屏幕坐标系文本标签。

#### 语法

```
D.slabel({x,y,text?} | array, labelStyle, backgroundStyle?)
```

#### 参数

* data：对象或数组；
  * 单个标签：`{x, y, text?}`
  * 多个标签：`[{x, y, text?}, ...]`
* labelStyle：文字样式对象
* backgroundStyle：背景样式对象

#### 文字样式：

样式对象：{color, size, align, bold, italic, show, text, x, y}

* color：文本颜色。字符串，（可选，默认：'#333333'）
* size：字体大小。整数，（可选，默认：12）
* align：文本对齐。字符串，（可选，默认：'left'）
* bold：是否加粗。布尔，（可选，默认：false）
* italic：是否斜体。布尔，（可选，默认：false）
* show：是否显示。布尔，（可选，默认：true）
* text：文本内容。字符串，（可选）
* x：X轴偏移。数值（正负值），（可选，默认：0）
  * 须在脚本中设置，设置面板中没有此项
  * `x:10` 表示向右偏移10px
  * `x:-10` 表示向左偏移10px
* y：Y轴偏移。数值（正负值），（可选，默认：0）
* 须在脚本中设置，设置面板中没有此项
* `y:10` 表示向下偏移10px
* `y:-10` 表示向上偏移10px

#### 背景样式：

样式对象：{color, full, size, style, padding, radius, show}

* color：背景颜色。字符串，（必填）
* full：填充方式。字符串，（可选，默认：'fill'）
* size：边框宽度。整数，（可选，默认：1）
* style：边框线型。字符串，（可选，默认：'solid'）
* padding：内边距。整数，（可选，默认：4）
  
  * 须在脚本中设置，设置面板中没有此项
  * `padding:10` 表示内边距为10px
* radius：圆角。整数，（可选，默认：0）
* 须在脚本中设置，设置面板中没有此项
* `radius:2` 表示圆角为2px
* show：是否显示。布尔，（可选，默认：true）

#### 文本优先级：

* 优先级1：样式对象中的 text
* 优先级2：数据对象中的 text
* 如果都没有，则不绘制

#### 示例

```
labelStyle = S.slabel('#fff', 14, 'left', title='标签样式')
labelStyle.text = '提示文本'
backgroundStyle = S.labelbg('rgba(255, 0, 0, 0.6)', 'fill', title='标签背景')
backgroundStyle.padding = 8
backgroundStyle.radius = 2
D.slabel({x:'right',y:'top',text:'提示'}, labelStyle, backgroundStyle)
```

以上示例为在图表右上角显示"提示文本"，因style和数据中都有text内容，style中text显示的优先级大于数据中的text，所以显示为"提示文本"。

### maindraw 方法

**副图指标在主图绘制图形**的方法。

maindraw方法可直接简写为大写​**MD**​，两个方法是相同的。

与draw内支持的方法相同。

#### 使用条件

脚本为​**副图位置**​：`//@position: 'vice'`

#### 示例

```
n = I.int(12, title="快线周期", tip="计算的快线周期", min=1)
k = I.int(26, title="慢线周期", tip="计算的慢线周期", min=1)
m = I.int(9, title="DEA周期", tip="计算的DEA周期", min=1)
// 定义可设置样式
difStyle = S.line('#FF9900', 1, 'solid', title="DIF样式")
deaStyle = S.line('#0066CC', 1, 'solid', title="DEA样式")
zeroStyle = S.line('#999', 1, 'solid', title="0轴样式")
buyMacdUp=S.bar('#2DC08E','stroke',title='多头增加')
buyMacdDown=S.bar('#2DC08E','fill',title='多头减少')
sellMacdUp=S.bar('#F92855','stroke',title='多头增加')
sellMacdDown=S.bar('#F92855','fill',title='多头减少')
buyStyle =  S.shape('arrowup', 12, '#00ff00', 'lowDown',title='买入信号')
sellStyle = S.shape('arrowdown', 12, '#ff0000', 'highUp',title='卖出信号')
// 计算数据
macdResult = F.macd(dataList, n, k, m)
difData = F.attr(macdResult,'dif')
deaData = F.attr(macdResult,'dea')
macdData = F.attr(macdResult,'macd')
// 绘制0轴
D.hline(0, zeroStyle)
// 绘制MACD柱状图（动态颜色）
D.bar(macdData, 0, ({value, prev}) => {
  if (value >= 0) {
    // 多头
    return prev < value ? buyMacdUp : buyMacdDown
  } else {
    // 空头
    return prev < value ? sellMacdUp : sellMacdDown
  }
})
// 绘制DIF线
D.line(difData, difStyle)
// 绘制DEA线
D.line(deaData, deaStyle)

// 主图绘制买入信号
buySignals = F.throughUp(difData, deaData)
MD.shape(buySignals,buyStyle)

// 主图绘制卖出信号  
sellSignals = F.throughDown(difData, deaData)
MD.shape(sellSignals,sellStyle )

// 设置精度
setPrecision(4)
// 输出工具栏提示数据
O.tools('DIF',difData,difStyle)
O.tools('DEA',deaData,deaStyle)
O.tools('MACD',macdData,{color: '#666666'})
```

以上示例为，MACD在副图显示，同时会在主图绘制信号图形。

### output 方法说明

output **输出**的方法。

output方法可直接简写为大写​**O**​，两个方法是相同的

output方法可以将指定数据输出

* 输出到工具栏：脚本左上角，名称与操作按钮后面
* 输出到调试面板：帮助编写脚本进行时调试。
* 信号提醒：当满足条件时，弹出提醒等

### output.tools()

输出到工具栏。

显示在脚本左上角名称与操作按钮后

#### 语法

```
O.tools(name, data, style)
```

#### 参数

* name：字符串，工具栏中显示的文字
* data：数据，支持以下类型：
  * 数组：与K线数组对应，显示当前或鼠标所选K线对应的值
  * 字符串：直接显示字符串内容
  * 数值：直接显示数值内容
* style：样式对象

#### 样式：

样式对象：{color}

* color：文本颜色。字符串

\*注 output.tools()中**没有样式回调方法**

#### 返回

无返回。

#### 示例

```
style1 = S.line('#F00',1,title='ma1样式')
ma2Color = S.color('#00F',title='ma2颜色')
ma1 = F.ma(dataList, 20)
ma2 = F.ma(dataList, 50)
setPrecision('price')
O.tools('MA20', ma1, style1)
O.tools('MA50', ma2, {color:ma2Color})

// 显示字符串或数值
O.tools('POC', '123.45', pocTextStyle)
O.tools('Status', 'Active', statusStyle)
O.tools('Price', 100.50, priceStyle)
```

以上示例为在工具栏中依次显示MA20和MA50内容以及对应的值。

显示格式为：MA20:值 MA50:值 POC:123.45 Status:Active Price:100.50

可以自己设置样式对象也可直接用S方法设置的样式对象

### output.print()

输出到调试面板。

#### 语法

```
O.print(message)
```

#### 参数

* message：字符串

#### 返回

无返回。

#### 示例

```
ma = F.ma(dataList, 20)
O.print('第一个ma的值：' + ma[0])
O.print('脚本已加载')
```

以上示例为在调试面板中依次显示第一个ma的值和脚本已加载。

### output.warn()

输出到调试面板。

#### 语法

```
O.warn(message)
```

#### 参数

* message：字符串

#### 返回

无返回。

#### 示例

```
O.warn('警告信息')
```

以上示例为在调试面板中显示警告信息。

### output.error()

输出到调试面板。

#### 语法

```
O.error(message)
```

#### 参数

* message：字符串

#### 返回

无返回。

#### 示例

```
O.error('错误信息')
```

以上示例为在调试面板中显示错误信息。

### output.signal()

输出到信号提醒。

#### 语法

```
O.signal(dataArray, callback?)
```

#### 参数

* dataArray：信号数据数组
* callback：回调函数（可选）

#### 使用方式

* O.signal(dataArray) - 默认输出
* O.signal(dataArray, (value) => '买入信号') - 使用回调函数
* O.signal(dataArray, (value) => ({type: 'buy', name: 'MA金叉'})) - 返回对象

#### 示例

```
ma1 = F.ma(dataList, 5, 'close')
ma2 = F.ma(dataList, 20, 'close')
buys = F.throughUp(ma1, ma2)
sells = F.throughDown(ma1, ma2)

O.signal(buys, (value) => ({type: 'buy', name: 'ma5上穿ma20'}))
O.signal(sells, (value) => ({type: 'sell', name: 'ma5下破ma20'}))
```

#### 返回

无返回。

#### 示例

```
ma1 = F.ma(dataList, 5, 'close')
ma2 = F.ma(dataList, 20, 'close')
buys = F.throughUp(ma1, ma2)
sells = F.throughDown(ma1, ma2)
O.signal(buys, (value) => {
  return {type: 'buy', name: 'ma5上穿ma20'}
})
O.signal(sells, (value) => {
  return {type: 'sell', name: 'ma5下破ma20'}
})
```

### formula 方法说明

GainLab Script 提供大量常用公式算法，方便在脚本中使用。

* 减少编写时须写大量代码实现计算。
* 减少常用公式反复重写。

formula方法可直接简写为大写​**F**​，两个方法是相同的

### F.attr(list, attr)

功能：获取对象数组属性值。

#### 参数

* list：数组（必须），对象数组
* attr：属性名称（必须）
  * 当对应对象不存在时，返回 null
  * 当对应属性不存在时，返回 NaN
  * 当对象是k线数据对象时可使用以下计算属性
    * hl2: 最高价和最低价的平均值
    * oc2: 开盘价和收盘价的平均值
    * hlc3: 最高价、最低价和收盘价的平均值
    * ohl3: 最高价、最低价和开盘价的平均值
    * ohlc4: 最高价、最低价、开盘价和收盘价的平均值
    * percentage: 涨跌幅%
  * \* **​注：​**F方法中其它支持**attr**属性的方法，都支持以上计算属性

#### 返回

数组，无效值处返回 null或NaN。

#### 示例

```
// 获取所有k线的收盘价数组
closeArr = F.attr(dataList, 'close')
// 获取所有k线的涨跌幅%数组
percentageArr = F.attr(dataList, 'percentage')
// 获取所有k线的最高价和最低价的平均值
hl2Arr = F.attr(dataList, 'hl2')
// 获取k线的macd值
macdArr = F.attr(F.macd(dataList, 12,26,9),'macd')
```

### formula.ma()

移动平均（Moving Average）。

#### 语法

```
F.ma(list, period, attr)
```

#### 参数

* list：数组（必须），数值数组或对象数组
* period：周期（必须），整数，≥1
* attr：属性名称（可选）
  * 当 list 为数值数组时，attr 无效
  * 当 list 为对象数组时，attr 为取值属性名

#### 返回

数组，不足周期或无效值处返回NAN。

### formula.ema()

指数移动平均（Exponential Moving Average）。

#### 语法

```
F.ema(list, period, attr)
```

#### 参数

* list：数组（必须），数值数组或对象数组
* period：周期（必须），整数，≥1
* attr：属性名称（可选）
  * 当 list 为数值数组时，attr 无效
  * 当 list 为对象数组时，attr 为取值属性名

#### 返回

数组，不足周期或无效值处返回NAN。

### formula.wma()

加权移动平均（Weighted Moving Average）。

#### 语法

```
F.wma(list, period, attr)
```

#### 参数

* list：数组（必须），数值数组或对象数组
* period：周期（必须），整数，≥1
* attr：属性名称（可选）
  * 当 list 为数值数组时，attr 无效
  * 当 list 为对象数组时，attr 为取值属性名

#### 返回

数组，不足周期或无效值处返回NAN。

### formula.sma()

平滑移动平均（Smoothed Moving Average）。

#### 语法

```
F.sma(list, period, attr)
```

#### 参数

* list：数组（必须），数值数组或对象数组
* period：周期（必须），整数，≥1
* attr：属性名称（可选）
  * 当 list 为数值数组时，attr 无效
  * 当 list 为对象数组时，attr 为取值属性名

#### 返回

数组，不足周期或无效值处返回NAN。

### formula.rma()

威尔德移动平均（Running Moving Average / Wilder's Moving Average）

#### 语法

```
F.rma(list, period, attr)
```

#### 参数

* list：数组（必须），数值数组或对象数组
* period：周期（必须），整数，≥1
* attr：属性名称（可选）
  * 当 list 为数值数组时，attr 无效
  * 当 list 为对象数组时，attr 为取值属性名

#### 返回

数组，不足周期或无效值处返回NAN。

### formula.rmaWithAlpha()

自定义平滑因子的 F.rma()。

#### 语法

```
F.rmaWithAlpha(list, period, attr)
```

#### 参数

* list：数组（必须），数值数组或对象数组
* period：周期（必须），整数，≥1
* alpha：平滑因子（可选），浮点数
  * 如没未传alpha，alpha = 1/period
* attr：属性名称（可选）
* 当 list 为数值数组时，attr 无效
* 当 list 为对象数组时，attr 为取值属性名

#### 返回

数组，不足周期或无效值处返回NAN。

### formula.trima()

三角移动平均线。

#### 语法

```
F.trima(list, period, attr)
```

#### 参数

* list：数组（必须），数值数组或对象数组
* period：周期（必须），整数，≥1
* alpha：平滑因子（可选），浮点数
  * 如没未传alpha，alpha = 1/period
* attr：属性名称（可选）
* 当 list 为数值数组时，attr 无效
* 当 list 为对象数组时，attr 为取值属性名

#### 返回

数组，不足周期或无效值处返回NAN。

### formula.kama()

自适应移动平均线

#### 语法

```
F.kama(list, period, attr)
```

#### 参数

* list：数组（必须），数值数组或对象数组
* period：周期（必须），整数，≥1
* alpha：平滑因子（可选），浮点数
  * 如没未传alpha，alpha = 1/period
* attr：属性名称（可选）
* 当 list 为数值数组时，attr 无效
* 当 list 为对象数组时，attr 为取值属性名

#### 返回

数组，不足周期或无效值处返回NAN。

### formula.t3()

T3是一种多重指数移动平均线，它通过多次应用EMA（指数移动平均线）来减少滞后性，同时保持平滑性。

#### 语法

```
F.t3(list, period, attr)
```

#### 参数

* list：数组（必须），数值数组或对象数组
* period：周期（必须），整数，≥1
* attr：属性名称（可选）
  * 当 list 为数值数组时，attr 无效
  * 当 list 为对象数组时，attr 为取值属性名

#### 返回

数组，不足周期或无效值处返回NAN。

### formula.dema()

双重指数移动平均。

#### 语法

```
F.dema(list, period, attr)
```

#### 参数

* list：数组（必须），数值数组或对象数组
* period：周期（必须），整数，≥1
* attr：属性名称（可选）
  * 当 list 为数值数组时，attr 无效
  * 当 list 为对象数组时，attr 为取值属性名

#### 返回

数组，不足周期或无效值处返回NAN。

### formula.tema()

三重指数移动平均（TEMA）。

#### 语法

```
F.tema(list, period, attr)
```

#### 参数

* list：数组（必须），数值数组或对象数组
* period：周期（必须），整数，≥1
* attr：属性名称（可选）
  * 当 list 为数值数组时，attr 无效
  * 当 list 为对象数组时，attr 为取值属性名

#### 返回

数组，不足周期或无效值处返回NAN。

### formula.hma()

赫尔移动平均（HMA）。

#### 语法

```
F.hma(list, period, attr)
```

#### 参数

* list：数组（必须），数值数组或对象数组
* period：周期（必须），整数，≥1
* attr：属性名称（可选）
  * 当 list 为数值数组时，attr 无效
  * 当 list 为对象数组时，attr 为取值属性名

#### 返回

数组，不足周期或无效值处返回NAN。

### formula.tma()

三角移动平均（SMA of SMA）。

#### 语法

```
F.tma(list, period, attr)
```

#### 参数

* list：数组（必须），数值数组或对象数组
* period：周期（必须），整数，≥1
* attr：属性名称（可选）
  * 当 list 为数值数组时，attr 无效
  * 当 list 为对象数组时，attr 为取值属性名

#### 返回

数组，不足周期或无效值处返回NAN。

### formula.lsma()

最小二乘移动平均（线性回归预测）。

#### 语法

```
F.lsma(list, period, attr)
```

#### 参数

* list：数组（必须），数值数组或对象数组
* period：周期（必须），整数，≥1
* attr：属性名称（可选）
  * 当 list 为数值数组时，attr 无效
  * 当 list 为对象数组时，attr 为取值属性名

#### 返回

数组，不足周期或无效值处返回NAN。

### formula.jma()

Jurik 移动平均。

#### 语法

```
F.jma(list, period, phase=0, attr)
```

#### 参数

* list：数组（必须），数值数组或对象数组
* period：周期（必须），整数，≥1
* phase：[-100,100] 调整响应
* attr：属性名称（可选）
  * 当 list 为数值数组时，attr 无效
  * 当 list 为对象数组时，attr 为取值属性名

#### 返回

数组，不足周期或无效值处返回NAN。

### formula.mcginley()

McGinley Dynamic 动态平均。

#### 语法

```
F.mcginley(list, period, attr)
```

#### 参数

* list：数组（必须），数值数组或对象数组
* period：周期（必须），整数，≥1

#### 返回

数组，不足周期或无效值处返回NAN。

### formula.edsma()

Ehlers 动态平滑移动平均。

#### 语法

```
F.edsma(list, period, attr)
```

#### 参数

* list：数组（必须），数值数组或对象数组
* period：周期（必须），整数，≥1

#### 返回

数组，不足周期或无效值处返回NAN。

### formula.vama()

成交量加权的平均（Volume Adjusted MA）。

#### 语法

```
F.vama(list, period, attr)
```

#### 参数

* list：数组（必须），数值数组或对象数组
* period：周期（必须），整数，≥1

#### 返回

数组，不足周期或无效值处返回null。

### formula.std()

标准差。

#### 语法

```
F.std(list, period, attr)
```

#### 参数

* list：数组（必须），数值数组或对象数组
* period：周期（必须），整数，≥1

#### 返回

数组，不足周期或无效值处返回null。

### formula.slope()

线性回归斜率。

#### 语法

```
F.slope(list, period, attr)
```

#### 参数

* list：数组（必须），数值数组或对象数组
* period：周期（必须），整数，≥1

#### 返回

数组，不足周期或无效值处返回null。

### formula.linreg()

线性回归值（支持偏移预测）。

#### 语法

```
F.linreg(list, period, offset=0, attr)
```

#### 参数

* list：数组（必须），数值数组或对象数组
* period：周期（必须），整数，≥1
* offset：偏移（可选），整数，默认0
* source：属性名称（可选），当 list 为对象数组时取值属性名

#### 返回

数组，不足周期或无效值处返回null。

### formula.calcVolatility()

年化波动率（%）。

#### 语法

```
F.calcVolatility(list, period)
```

#### 参数

* list：数组（必须），数值数组或对象数组
* period：周期（必须），整数，≥1

#### 返回

数组，不足周期或无效值处返回null。

### formula.avg()

计算数组的平均值。

#### 语法

```
F.avg(list, attr, precision)
```

#### 参数

* list：数组（必须），数值数组或对象数组
* attr：属性名称（可选），当 list 为对象数组时取值属性名
* precision：精度（可选），整数，默认4

#### 返回

数值，数组的平均值；不足周期或无效值处返回null。

### formula.hhv()

计算周期内最高值。

#### 语法

```
F.hhv(list, period, attr)
```

#### 参数

* list：数组（必须），数值数组或对象数组
* period：周期（必须），整数，≥1
* attr：属性名称（可选），当 list 为对象数组时取值属性名

#### 返回

数组，不足周期或无效值处返回 null。

### formula.llv()

计算周期内最低值。

#### 语法

```
F.llv(list, period, attr)
```

#### 参数

* list：数组（必须），数值数组或对象数组
* period：周期（必须），整数，≥1
* attr：属性名称（可选），当 list 为对象数组时取值属性名

#### 返回

数组，不足周期或无效值处返回 null。

### formula.cross()

检测两条线的交叉。

#### 语法

```
F.cross(list1, list2 | number, attr)
```

#### 参数

* list1：数组（必须），第一条线的数据
* list2：数组或数值（必须），第二条线的数据
  * 当 list2 为数值时，每个list1中的数据与其对比
  * 当 list2 为数组时，每个list1中的数据与对应的list2中的数据对比
* attr：属性名称（可选），当数据为对象数组时取值属性名

#### 返回

数组，长度与 list1 相同；交叉时返回对象 {val1, val2, type('up'/'down')}，否则返回 null。

#### 示例

```
ma = F.ma(dataList, 5, 'close')
// 获取ma5与收盘价的交叉
cross1 = F.cross(dataList, ma, 'close')
macdArr = F.attr(F.macd(dataList, 12,26,9),'macd')
// 获取macd与0值的交叉
cross2 = F.cross(macdArr, 0)
```

以上示例会返回与dataList长度相同的数组，如果有交叉会返回对象 {val1, val2, type('up'/'down')}，否则返回 null。

### formula.throughUp()

检测数据上穿另一条数据。

#### 语法

```
F.throughUp(list1, list2, attr)
```

#### 参数

* list1：数组（必须），第一条线的数据
* list2：数组或数值（必须），第二条线的数据
  * 当 list2 为数值时，每个list1中的数据与其对比
  * 当 list2 为数组时，每个list1中的数据与对应的list2中的数据对比
* attr：属性名称（可选），当数据为对象数组时取值属性名

#### 返回

数组，长度与 list1 相同；上穿时返回对象 {val1, val2}，否则返回 null。

#### 示例

```
ma1 = F.ma(dataList, 5, 'close')
ma2 = F.ma(dataList, 20, 'close')
// ma5上穿ma20的信号
through1 = F.throughUp(ma1, ma2)
rsi = F.rsi(dataList, 12)
// 获取rsi上穿20的信号
through2 = F.throughUp(rsi, 20)
```

以上示例会返回与dataList长度相同的数组，返回对象上穿时 {val1, val2}，否则返回 null。

### formula.throughDown()

检测下破信号。

#### 语法

```
F.throughDown(list1, list2, attr)
```

#### 参数

* list1：数组（必须），第一条线的数据
* list2：数组或数值（必须），第二条线的数据
  * 当 list2 为数值时，每个list1中的数据与其对比
  * 当 list2 为数组时，每个list1中的数据与对应的list2中的数据对比
* attr：属性名称（可选），当数据为对象数组时取值属性名

#### 返回

数组，长度与 list1 相同；下穿时返回对象 {val1, val2}，否则返回 null。

#### 示例

```
rsi = F.rsi(dataList, 12)
// 获取rsi下破80的信号
through = F.throughDown(rsi, 80)
```

以上示例会返回与dataList长度相同的数组，返回对象下破时 {val1, val2}，否则返回 null。

### formula.kdj()

计算KDJ指标（Stochastic Oscillator）。

#### 语法

```
F.kdj(list, k_period, d_period, j_period)
```

#### 参数

* list：数组（必须），k线数据数组
* k\_period：K值周期（必须），整数，≥1
* d\_period：D值周期（必须），整数，≥1
* j\_period：J值周期（必须），整数，≥1

#### 返回

对象数组。对象结构：`{k,d,j}`。不足周期或无效值处返回空对象{}。

### formula.boll()

计算布林带（Bollinger Bands）。

#### 语法

```
F.boll(list, period, multiplier)
```

#### 参数

* list：数组（必须），k线数据数组
* period：周期（必须），整数，≥1
* multiplier：标准差倍数（必须），数值，≥1

#### 返回

对象数组，对象结构：`{mid,ub,lb}`。不足周期或无效值处返回 null。

### formula.macd()

MACD指标（Moving Average Convergence Divergence）。

#### 语法

```
F.macd(list, fast, slow, signal)
```

#### 参数

* list：数组（必须），k线数据数组
* fast：快线周期（必须），整数，≥1
* slow：慢线周期（必须），整数，≥1
* signal：信号线周期（必须），整数，≥1

#### 返回

对象数组，对象结构：`{dif, dea, macd}`。不足周期或无效值处返回 null。

### formula.tr()

真实波幅 TR。

#### 语法

```
F.tr(list)
```

#### 参数

* list：数组（必须），k线数据数组

#### 返回

数组，不足周期或无效值处返回 null。

### formula.atr()

平均真实波幅（Average True Range）。

#### 语法

```
F.atr(list, period)
```

#### 参数

* list：数组（必须），k线数据数组
* period：周期（必须），整数，≥1

#### 返回

数组，不足周期或无效值处返回 null。

### formula.atrWithRMA()

使用 RMA 平滑的 ATR。

#### 语法

```
F.atrWithRMA(list, period)
```

#### 参数

* list：数组（必须），k线数据数组
* period：周期（必须），整数，≥1

#### 返回

数组，不足周期或无效值处返回 null。

### formula.adxWithRMA()

平均方向性指数 ADX（基于 RMA）。

#### 语法

```
F.adxWithRMA(list, period)
```

#### 参数

* list：数组（必须），k线数据数组
* period：周期（必须），整数，≥1

#### 返回

数组，不足周期或无效值处返回 null。

### formula.rsi()

相对强弱指数（Relative Strength Index）。

#### 语法

```
F.rsi(list, period)
```

#### 参数

* list：数组（必须），k线数据数组
* period：周期（必须），整数，≥1

#### 返回

数组，不足周期或无效值处返回 null。

### formula.cci()

顺势指标（Commodity Channel Index）。

#### 语法

```
F.cci(list, period)
```

#### 参数

* list：数组（必须），k线数据数组
* period：周期（必须），整数，≥1

#### 返回

数组，不足周期或无效值处返回 null。

### formula.mtm()

动量指标（Momentum）。

#### 语法

```
F.mtm(list, period, attr)
```

#### 参数

* list：数组（必须），数值数组或对象数组
* period：周期（必须），整数，≥1
* attr：属性名称（可选），当数据为对象数组时取值属性名

#### 返回

数组，不足周期或无效值处返回 null。

### formula.roc()

变动率指标（%）。

#### 语法

```
F.roc(list, period)
```

#### 参数

* list：数组（必须），k线数据数组
* period：周期（必须），整数，≥1

#### 返回

数组，不足周期或无效值处返回 null。

### formula.wr()

威廉指标。

#### 语法

```
F.wr(list, period)
```

#### 参数

* list：数组（必须），k线数据数组
* period：周期（必须），整数，≥1

#### 返回

数组，不足周期或无效值处返回 null。

### formula.kijunV2()

改进版基准线（一目均衡表变体）。

#### 语法

```
F.kijunV2(list, period, factor)
```

#### 参数

* list：数组（必须），k线数据数组
* period：周期（必须），整数，≥1
* factor：因子（必须），数值，≥1

#### 返回

数组，不足周期或无效值处返回 null。

### formula.trix()

三重指数平滑指标。

#### 语法

```
F.trix(list, period)
```

#### 参数

* list：数组（必须），k线数据数组
* period：周期（必须），整数，≥1

#### 返回

数组，不足周期或无效值处返回 null。

### formula.brar()

BRAR 人气意愿指标。

#### 语法

```
F.brar(list, period)
```

#### 参数

* list：数组（必须），k线数据数组
* period：周期（必须），整数，≥1

#### 返回

对象数组，对象结构：`{ br, ar }`。不足周期或无效值处返回 null。

### formula.vr()

成交量变异率。

#### 语法

```
F.vr(list, period)
```

#### 参数

* list：数组（必须），k线数据数组
* period：周期（必须），整数，≥1

#### 返回

数组，不足周期或无效值处返回 null。

### formula.obv()

能量潮指标。

#### 语法

```
F.obv(list)
```

#### 参数

* list：数组（必须），k线数据数组

#### 返回

数组，不足周期或无效值处返回 null。

### formula.emv()

简易波动指标。

#### 语法

```
F.emv(list, period)
```

#### 参数

* list：数组（必须），k线数据数组
* period：周期（必须），整数，≥1

#### 返回

数组，不足周期或无效值处返回 null。

### formula.psy()

心理线。

#### 语法

```
F.psy(list, period)
```

#### 参数

* list：数组（必须），k线数据数组
* period：周期（必须），整数，≥1

#### 返回

数组，不足周期或无效值处返回 null。

### formula.vo()

量价指标（短长周期上涨占比差）。

#### 语法

```
F.vo(list, short, long)
```

#### 参数

* list：数组（必须），k线数据数组
* short：短周期（必须），整数，≥1
* long：长周期（必须），整数，≥1

#### 返回

对象数组，对象结构：`{ up, down, val }`。不足周期或无效值处返回 null。

### formula.dmi()

动向指数 DMI。

#### 语法

```
F.dmi(list, period, adxPeriod)
```

#### 参数

* list：数组（必须），k线数据数组
* period：周期（必须），整数，≥1
* adxPeriod：ADX周期（必须），整数，≥1

#### 返回

返回对象数组，长度与 list 相同。对象格式`{pdi, mdi, adx, adxr}`；不足周期或无效值处返回 null。

### formula.dma()

平均线差（DMA）。

#### 语法

```
F.dma(list, fast, slow)
```

#### 参数

* list：数组（必须），k线数据数组
* fast：快线周期（必须），整数，≥1
* slow：慢线周期（必须），整数，≥1

#### 返回

数组，不足周期或无效值处返回 null。

### formula.asi()

振动升降指标（Accumulation Swing Index）。

#### 语法

```
F.asi(list, period)
```

#### 参数

* list：数组（必须），k线数据数组
* period：周期（必须），整数，≥1

#### 返回

数组，不足周期或无效值处返回 null。

### formula.stochastic()

随机指标（Stochastic Oscillator）。

#### 语法

```
F.stochastic(list, k_period, d_period)
```

#### 参数

* list：数组（必须），k线数据数组
* k\_period：K值周期（必须），整数，≥1
* d\_period：D值周期（必须），整数，≥1

#### 返回

对象数组，对象结构：`{ k, d }`。不足周期或无效值处返回 null。

### formula.donchianChannel()

唐奇安通道（Donchian Channel）。

#### 语法

```
F.donchianChannel(list, period)
```

#### 参数

* list：数组（必须），k线数据数组
* period：周期（必须），整数，≥1

#### 返回

对象数组，对象结构：`{ upper, lower, middle }`。不足周期或无效值处返回 null。

### formula.mfi()

资金流量指标（Money Flow Index）。

#### 语法

```
F.mfi(list, period)
```

#### 参数

* list：数组（必须），k线数据数组
* period：周期（必须），整数，≥1

#### 返回

数组，不足周期或无效值处返回 null。

### formula.moneyFlow()

资金流量（Money Flow）。

#### 语法

```
F.moneyFlow(list)
```

#### 参数

* list：数组（必须），k线数据数组

#### 返回

数组，无效值处返回 null。

### formula.mf()

市场促进指数/资金流（与 moneyFlow 类似）。

#### 语法

```
F.mf(list)
```

#### 参数

* list：数组（必须），k线数据数组

#### 返回

数组；首根返回 null。

### formula.getTrend()

趋势方向。

#### 语法

```
F.getTrend(list, short, long, source)
```

#### 参数

* list：数组（必须），k线数据数组
* short：短周期（必须），整数，≥1
* long：长周期（必须），整数，≥1
* source：源数据（可选），字符串，默认 'close'

#### 返回

字符串数组，元素为 'up' | 'down' | 'sideways' | null。

### formula.fibonacciRetracement()

斐波那契回调位序列。

#### 语法

```
F.fibonacciRetracement(high, low)
```

#### 参数

* high：最高价（必须），数值
* low：最低价（必须），数值

#### 返回

数组，对象结构：`{ level, price, percentage }`。

### formula.sar()

抛物线转向（Parabolic SAR）。

#### 语法

```
F.sar(list, acceleration, maximum, period)
```

#### 参数

* list：数组（必须），k线数据数组
* acceleration：加速因子（必须），数值，≥1
* maximum：最大加速因子（必须），数值，≥1
* period：周期（必须），整数，≥1

#### 返回

数组，不足周期或无效值处返回 null。

### formula.high()

获取数组中的最高值。

#### 语法

```
F.high(list, attr)
```

#### 参数

* list：数组（必须），数值数组或对象数组
* attr：属性名称（可选），当 list 为对象数组时取对应属性计算

#### 返回

对象 `{index, val}`，包含最高值的索引和数值。

### formula.low()

获取数组中的最低值。

#### 语法

```
F.low(list, attr)
```

#### 参数

* list：数组（必须），数值数组或对象数组
* attr：属性名称（可选），当 list 为对象数组时取对应属性计算

#### 返回

对象 `{index, val}`，包含最高值的索引和数值。

### formula.bias()

乖离率（Bias）。

#### 语法

```
F.bias(list, period, attr)
```

#### 参数

* list：数组（必须），数值数组或对象数组
* period：周期（必须），整数，≥1
* attr：属性名称（可选），当 list 为对象数组时取值属性名

#### 返回

数组，不足周期或无效值处返回 null。

### formula.call()

动态调用公式库函数方法。

按**方法名字符串**动态调用公式库函数，便于通过 `I.select` 让用户选择不同算法，再统一用一套调用逻辑。

#### 语法

```
F.call(methodName, ...args)
```

#### 使用场景

* 让用户选择均线类型（MA/EMA/WMA/RMA/...），脚本用一处 `F.call` 调用
* 根据不同配置切换动量/波动率/趋势等算法

#### 参数

* methodName：字符串（大小写不敏感），例如 'ma'、'ema'、'rsi'、'macd'、'tr'、'atr'、'kama'、'trima'、't3'、'linreg' 等
* ...args：对应方法的参数序列，顺序与各方法一致（见上文 F.\* 定义）

#### 返回

等同于被调用方法的返回类型。

#### 示例

```
// 用户在面板选择
maType = I.select('MA', options=['MA','EMA','WMA','RMA'], title='均线类型')
len = I.int(20, title='周期')

// 统一调用
avg = F.call(maType, dataList, len, 'close')
D.line(avg, S.line('#5b8ff9',1))

// 切换动量类
oscType = I.select('RSI', options=['RSI','CCI'], title='振荡指标')
osc = F.call(oscType, dataList, 14)
D.line(osc, S.line('#faad14',1))
```

#### 说明

* 请确保参数数量与顺序正确；传错将返回 null 或产生意外结果

### util 方法说明

GainLab Script 提供大量常用辅助函数，方便在脚本中使用。

* 减少编写时须写大量代码实现计算。
* 减少常用辅助函数反复重写。

util方法可直接简写为大写​**U**​，两个方法是相同的

### util.formatNumber()

格式化数字，可选千分位。

#### 语法

```
U.formatNumber(value, precision=2, useThousandSeparator=false)
```

#### 参数

* value：数值
* precision：小数位数（默认 2）
* useThousandSeparator：是否使用千分位（默认 false）

#### 返回

字符串。

#### 示例

```
// "123,456.79
U.formatNumber(123456.789, 2, true) 
// "12.3000"
U.formatNumber(12.3, 4)
```

### util.formatPercent()

格式化为百分比（value 为小数）。

#### 语法

```
U.formatPercent(value, precision=2)
```

#### 参数

* value：小数（如 0.1234）
* precision：小数位数（默认 2）

#### 返回

字符串。

#### 示例

```
U.formatPercent(0.1267, 1) // "12.7%"
```

### util.formatPrice()

格式化价格到固定位数。

#### 语法

```
U.formatPrice(value, precision=2)
```

#### 参数

* value：数值
* precision：小数位数（默认 2）

#### 返回

字符串。

#### 示例

```
U.formatPrice(1234.5, 2) // "1234.50"
```

### util.toNumber()

处理科学计数法到普通数字。

#### 语法

```
U.toNumber(val)
```

#### 参数

* val：字符串/数值，允许科学计数法

#### 返回

数值。

#### 示例

```
U.toNumber('1e-8')   // 0.00000001
U.toNumber('123.45') // 123.45
```

### util.mod()

取模。

#### 语法

```
U.mod(a, b)
```

#### 参数

* a：被除数
* b：模

#### 返回

数值。

#### 示例

```
U.mod(7, 3) // 1
```

### util.average()

数组平均值。

#### 语法

```
U.average(array)
```

#### 参数

* array：数值数组

#### 返回

数值；空数组返回 0。

#### 示例

```
U.average([1,2,3]) // 2
```

### util.sum()

数组求和。

#### 语法

```
U.sum(array)
```

#### 参数

* array：数值数组

#### 返回

数值。

#### 示例

```
U.sum([1,2,3]) // 6
```

### util.unique()

数组去重（浅层）。

#### 语法

```
U.unique(array)
```

#### 参数

* array：任意数组

#### 返回

新数组。

#### 示例

```
U.unique([1,1,2,2,3]) // [1,2,3]
```

### util.sort()

数值数组排序。

#### 语法

```
U.sort(array, ascending=true)
```

#### 参数

* array：数值数组
* ascending：是否升序（默认 true）

#### 返回

新数组。

#### 示例

```
U.sort([3,1,2])        // [1,2,3]
U.sort([3,1,2], false) // [3,2,1]
```

### util.getValue()

安全读取数组指定索引。

#### 语法

```
U.getValue(array, index, defaultValue=null)
```

#### 参数

* array：数组
* index：索引
* defaultValue：缺省返回值（默认 null）

#### 返回

任意类型。

#### 示例

```
U.getValue([10], 3, 0) // 0
```

### util.isValidNumber()

是否为有限的有效数字。

#### 语法

```
U.isValidNumber(value)
```

#### 参数

* value：任意值

#### 返回

布尔。

#### 示例

```
U.isValidNumber(1/0)     // false
U.isValidNumber('123')   // false
U.isValidNumber(123.456) // true
```

### util.isEmpty()

是否为 null/undefined/''。

#### 语法

```
U.isEmpty(value)
```

#### 参数

* value：任意值

#### 返回

布尔。

#### 示例

```
U.isEmpty('')      // true
U.isEmpty(null)    // true
U.isEmpty('text')  // false
```

### util.random()

返回 [min,max) 区间随机数。

#### 语法

```
U.random(min=0, max=1)
```

#### 参数

* min/max：范围

#### 返回

数值。

#### 示例

```
U.random(10, 20) // [10,20) 内随机小数
```

### util.randomInt()

返回 [min,max] 随机整数。

#### 语法

```
U.randomInt(min, max)
```

#### 参数

* min/max：范围（含端点）

#### 返回

整数。

#### 示例

```
U.randomInt(1, 6) // 1..6
```

### util.clamp()

将数值限制在区间内。

#### 语法

```
U.clamp(value, min, max)
```

#### 参数

* value：当前值
* min/max：限制区间

#### 返回

数值。

#### 示例

```
U.clamp(120, 0, 100) // 100
```

### util.lerp()

线性插值。

#### 语法

```
U.lerp(start, end, t)
```

#### 参数

* start/end：起止值
* t：插值系数 [0..1]

#### 返回

数值。

#### 示例

```
U.lerp(0, 10, 0.3) // 3
```

### util.toRadians()

角度转弧度。

#### 语法

```
U.toRadians(degrees)
```

#### 参数

* degrees：角度

#### 返回

数值。

#### 示例

```
U.toRadians(180) // 3.1415926...
```

### util.toDegrees()

弧度转角度。

#### 语法

```
U.toDegrees(radians)
```

#### 参数

* radians：弧度

#### 返回

数值。

#### 示例

```
U.toDegrees(Math.PI) // 180
```

### util.distance()

两点距离。

#### 语法

```
U.distance(x1, y1, x2, y2)
```

#### 参数

* x1/y1：起点
* x2/y2：终点

#### 返回

数值。

#### 示例

```
U.distance(0,0,3,4) // 5
```

### U.formatDate(timestamp, format='YYYY-MM-DD')

功能：时间戳格式化（支持 YYYY/MM/DD/HH/mm/ss）。

#### 参数

* timestamp：毫秒时间戳
* format：格式模板（默认 'YYYY-MM-DD'）

#### 返回

字符串。

#### 示例

```
U.formatDate(1700000000000, 'YYYY-MM-DD HH:mm:ss')
```

### U.now()

功能：当前毫秒时间戳。

#### 返回

数值。

#### 示例

```
ts = U.now()
```

### util.deepClone()

深拷贝（Date/Array/Object 处理）。

#### 语法

```
U.deepClone(obj)
```

#### 参数

* obj：对象/数组/Date

#### 返回

新对象/数组。

#### 示例

```
copy = U.deepClone({a:1,b:[2,3]})
```

### util.merge()

浅/深合并（递归对象字段）。

#### 语法

```
U.merge(target, source)
```

#### 参数

* target：目标对象
* source：来源对象

#### 返回

合并后的对象。

#### 示例

```
U.merge({a:1, b:{x:1}}, {b:{y:2}, c:3}) // {a:1, b:{x:1,y:2}, c:3}
```

### util.colorToRgb()

颜色字符串转RGB对象。

#### 语法

```
U.colorToRgb(color)
```

#### 参数

* color：颜色字符串

#### 返回

RGB对象。`{r, g, b}`

#### 示例

```
U.colorToRgb('#ff0000') // {r:255, g:0, b:0}
```

### util.colorToRgba()

颜色字符串转RGBA字符串。

传入颜色值和透明度，返回rgba(r,g,b,a)格式字符串。

#### 语法

```
U.colorToRgba(color, alpha=1)
```

#### 参数

* color：颜色字符串（必须）
* alpha：透明度（可选），数值，范围0-1，默认1

#### 返回

字符串，rgba(r,g,b,a)格式；无效颜色返回null。

#### 示例

```
U.colorToRgba('#ff0000') // "rgba(255,0,0,1)"
U.colorToRgba('#ff0000', 0.5) // "rgba(255,0,0,0.5)"
U.colorToRgba('blue', 0.8) // "rgba(0,0,255,0.8)"
```

### util.isValid()

检查值是否为有效值，用于判断数据是否可用于计算。

#### 语法

```
U.isValid(value)
```

#### 参数

* value：任意值

#### 返回

布尔。

#### 说明

以下情况返回false：

* null 或 undefined
* NaN 数值
* 空字符串或只包含空格的字符串
* 空对象（无属性）

#### 示例

```
U.isValid(null)      // false
U.isValid(undefined) // false
U.isValid(NaN)       // false
U.isValid('')        // false
U.isValid('   ')     // false
U.isValid({})        // false
U.isValid(123)       // true
U.isValid('text')    // true
U.isValid({a:1})     // true
```

