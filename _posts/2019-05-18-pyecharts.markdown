---
layout:     post
title:      "pyecharts绘制金融股票数据"
date:       2019-05-18
author:     "莫大"
header-img: "img/main_pyecharts.jpg"
tags:
    - 读书
    - 技术
---

> 记录技术成长的点点滴滴。

# pyecharts 1.0 版本使用基本方法

Echarts 是一个由百度开源的数据可视化工具，凭借着良好的交互性，精巧的图表设计，得到了众多开发者的认可，“前辈们”基于python，创造了pyecharts。作图效果非常好。



读者可以查看王圣元大神的文章，[盘一盘 Python 系列 7 - PyEcharts](<https://mp.weixin.qq.com/s/5GJKIt5OkMh7C3xqoTJsTA>) 结合金融股票数据的绘制学习pyecharts。



在上面的文章中使用了pyechartsv0.5.X ，但最新的是V1版本，且0.5.x 版本将不再进行维护，新版本系列将从 v1.0.0 开始。[主要变动有](<https://github.com/pyecharts/pyecharts/issues/892>)：

### API 重构

add() 接口拆分，现在的 add() 接口做了太多的事情了。

1. 对于有 XY 轴的图表，拆分为 add_xaixs()/add_yaxis() 方法。
2. 新增 set_series_opts 方法，用于一次性设置所有 series 配置项。
3. 新增 set_base_opts 方法，用于设置 base 配置项，如 dataZoom, legend, tooltip, toolbox 等。

以上的拆分仍能保持接口的简洁性。

### 图表重构

废除 Overlap 组合图表，改为 chart.overlap() 方法，仅部分图表实现了该方法。

### 插件机制重构

废除现有的插件机制，仅支持两种情况

1. online 模式，使用 pyecharts 官方提供的 assets host，或者部署自己的 remote host。
2. local 模式，使用自己本地开启的文件服务提供 assets host，会提供一键启动的脚本，方便部署。

理由

1. 现在的 pyecharts 插件机制分散，管理/升级并没有想象中的方便，而且分开为 jupyter/local render 两种情况，这就导致了两个要分开管理，虽然我们的 pip 包可以同时 update 这两种情况引用的 assets，但是由于存在缓存等因素，并不能保证每次都到正确的更新。
2. pyecharts 不用再依赖这些包，依赖包和 pyecharts 包版本的管理也是一个容易出问题的地方。
3. 减少维护工作，线上热更新。

### 代码风格重构

1. 停止对 Python2.7 版本的支持，仅支持 Python3.5+，是时候全面拥抱 Python3 了
2. 所有代码使用 TypeHint，增加可读性
3. 所有配置项均 OOP，使用 attrs 重写配置项类。
4. 废除 add() 中那堆长得令人发指的参数项列表。

### 可期待的新特性

1. 对 components 的支持，可以使用 pyecharts 制作简单的报表。
2. 支持更加原生的 javascript 配置项，方便用户自己定制

### 兼容性

本次重构基本上不会考虑任何兼容性的问题，这是一个全新版本的 pyecharts。不想再为它打补丁来容忍糟糕的接口设计，是时候重生了。

因此还是推荐使用V1版本，我基于王生的文章，通过V1版本对于部分图进行了重画，便于了解和使用pyecharts.

# 1.0版本概况

笔者觉得1.0版本主要将之前的add()方法进行了拆分，对于很多配置通过opts配置项进行设置。而数据方面主要通过 add_xaixs()/add_yaxis() 方法设置坐标轴。



##  	全局配置项

基于上图可以看到，基本的配置项有：InitOpts、ToolboxOpts、LegendOpts、VisualMapOpts、TooltipOpt、DataZoomOPts配置。



## 系列配置项

主要是对于基本的颜色、标签、文字、线条样式、标记点等进行配置。主要配置项有：ItemStyleOpts、TextStyleOpts、LabelOpts、LineStyleOpts、MarkPointOpts等。

对于直方图、折线图等基本使用可以参照   [pyecharts主页](https://github.com/pyecharts/pyecharts)进行查看。



## 绘制K线图

基于先前文章中的内容，k线绘制流程如下：

- 基于Candlestick对象；
- 传入的数据中按照“开“、”收“、”高“、”低“进行排序；
- 通过set_global_opts设置不同的展示元素；
- 通过add_xaxis和add_yaxis设置日期、点位数据。

主要代码如下：

```python
    candles = Candlestick(init_opts=opts.InitOpts(width="1440px", height="720px"))
    candles.add_xaxis(xaxis_data=dates)
    candles.add_yaxis(series_name="K线",
                      y_axis=price,
                      tooltip_opts=opts.TooltipOpts(is_show=True, trigger='item'),
                      itemstyle_opts=opts.ItemStyleOpts(color="#ec0000", color0="#00da3c", border_color="#8A0000",border_color0="#008F28")
                      )
    candles.set_global_opts(
                            title_opts=opts.TitleOpts(title="苹果-K线图", pos_top='top'),
                            yaxis_opts=opts.AxisOpts
                            (is_scale=True,
                            splitline_opts=opts.SplitLineOpts(is_show=True, linestyle_opts=opts.LineStyleOpts(width=1)),
                            splitarea_opts=opts.SplitAreaOpts(is_show=True, areastyle_opts=opts.AreaStyleOpts(opacity=1)),
                            name="点位", name_location="middle", name_gap=40,
                             name_textstyle_opts=opts.TextStyleOpts(font_size=15, font_weight="bold")
                            ),
                            xaxis_opts=opts.AxisOpts(name="日期", name_location="middle", name_gap=40,
                            name_textstyle_opts=opts.TextStyleOpts(font_size=15, font_weight="bold")),
                            legend_opts=opts.LegendOpts(is_show=True, pos_right=True, orient='vertical', textstyle_opts=
                                                      opts.TextStyleOpts(font_size=10)),
                            datazoom_opts=[opts.DataZoomOpts()],
                            toolbox_opts=opts.ToolboxOpts()
                            )
    candles.set_series_opts()
    line = Line()
    line.add_xaxis(xaxis_data=dates)
    line.add_yaxis(series_name='最高', y_axis=high_price, is_symbol_show=True,
                   label_opts=opts.LabelOpts(is_show=False))
    line.add_xaxis(xaxis_data=dates)
    line.add_yaxis(series_name='最低', y_axis=low_price, is_symbol_show=True,
                   label_opts=opts.LabelOpts(is_show=False))
    candles.overlap(line)
    candles.render("kline.html")
```

对于其中的标题、图例、字体、数据缩放等概念可以查找王生的文章。在V1版本中，主要通过：

```python
# 标题设置
title_opts=opts.TitleOpts(title="苹果-K线图", pos_top='top')
# y坐标轴设置
yaxis_opts=opts.AxisOpts
# x坐标轴设置
xaxis_opts=opts.AxisOpts
# 图例设置
legend_opts=opts.LegendOpts
# 数据缩放设置
datazoom_opts=[opts.DataZoomOpts()]
```

同时也添加了最高、最低点。

```python
    line = Line()
    line.add_xaxis(xaxis_data=dates)
    line.add_yaxis(series_name='最高', y_axis=high_price, is_symbol_show=True,
                   label_opts=opts.LabelOpts(is_show=False))
    line.add_xaxis(xaxis_data=dates)
    line.add_yaxis(series_name='最低', y_axis=low_price, is_symbol_show=True,
                   label_opts=opts.LabelOpts(is_show=False))
    candles.overlap(line)
    candles.render("kline.html")
```



其中，通过overlap方法将绘图对象进行叠加。

绘制图形如下：



## 绘制移动平均线

对于移动平均线，需要通过talib计算出相应的值。

```python
    line = Line()
    for wp in win_peroid:
        MA_price = ta.MA(aap_perf['Close'].values, timeperiod=wp)

        line.add_xaxis(xaxis_data=dates)
        line.add_yaxis(series_name='MA' + str(wp), y_axis=MA_price, is_symbol_show=True,
                       label_opts=opts.LabelOpts(is_show=False))
    avgline.overlap(line)
    avgline.render("kline.html")
```



基于line对象通过add_yaxis增加y坐标的数据。

绘制图形如下：





## 绘制强弱走势图

绘制价格走势和RSI强弱数据，此时需要用到双Y轴。笔者查阅了一些资料，发现V1版本绘制双Y轴相比之前的版本有些繁琐，部分设置还需要进一步熟悉。

主要代码如下：

```python
    avgline = Line(init_opts=opts.InitOpts(width="1440px", height="720px"))
    avgline.add_xaxis(xaxis_data=dates)
    avgline.add_yaxis(series_name="close",
                      y_axis=price,
                      )
    avgline.extend_axis(yaxis=opts.AxisOpts(
                    name="RSI强弱",
                    name_location="middle",
                    name_gap=40,
                    position='right',
                    axislabel_opts=opts.LabelOpts(formatter="{value}"),
                    interval=10,
                    axisline_opts=opts.AxisLineOpts(linestyle_opts=opts.LineStyleOpts(color="#d14a61")),
                    splitline_opts=opts.SplitLineOpts(
                    is_show=True,
                    linestyle_opts=opts.LineStyleOpts(opacity=1))
    ))
    avgline.set_series_opts(label_opts=opts.LabelOpts(is_show=False))

    avgline.set_global_opts(
        title_opts=opts.TitleOpts(title="苹果-价格走势", pos_top='top'),
        yaxis_opts=opts.AxisOpts
        (
            name="股票价格",
            name_location='middle',
            name_gap=50,
            position='left',
            # offset=80,
            # min_=0, max_=100,
            is_scale=True,
            splitline_opts=opts.SplitLineOpts(
                is_show=True,
                linestyle_opts=opts.LineStyleOpts(width=1)),
            splitarea_opts=opts.SplitAreaOpts(
                is_show=True,
                areastyle_opts=opts.AreaStyleOpts(opacity=1)),
            name_textstyle_opts=opts.TextStyleOpts(font_size=15, font_weight="bold"),
            axislabel_opts=opts.LabelOpts(formatter="{value} ($)"),
         ),
        datazoom_opts=[opts.DataZoomOpts()],
        toolbox_opts=opts.ToolboxOpts(),
        tooltip_opts=opts.TooltipOpts(trigger="axis", axis_pointer_type="cross"),
        legend_opts=opts.LegendOpts(
            is_show=True,
            pos_right=True,
            orient='vertical',
            textstyle_opts=opts.TextStyleOpts(
                font_size=10)),
        xaxis_opts=opts.AxisOpts(
            name="日期",
            name_location="middle",
            name_gap=30,
            name_textstyle_opts=opts.TextStyleOpts(
                font_size=15,
                font_weight="bold")),
    )
    line = Line()
    wp = 14
    RSI = ta.RSI(np.array(aap_perf['Close']), timeperiod=wp)
    line.add_xaxis(xaxis_data=dates)
    line.add_yaxis(series_name='RSI',
                   y_axis=RSI,
                   yaxis_index=1,
                   label_opts=opts.LabelOpts(
                       is_show=False))
    line.add_yaxis(series_name='Support',
                   y_axis=30 * np.ones(np.shape(RSI)),
                   label_opts=opts.LabelOpts(
                       is_show=False),
                   yaxis_index=1,)
    line.add_yaxis(series_name='Resistance',
                   y_axis=70 * np.ones(np.shape(RSI)),
                   label_opts=opts.LabelOpts(
                       is_show=False),
                   yaxis_index=1)

    line.set_series_opts(linestyle_opts=opts.LineStyleOpts(type_='dashed'))

    avgline.overlap(line).render("rsi.html")
```

通过extend_axis和set_global_opts中设置了双Y轴。

绘制图形如下：



其中，通过

```python
tooltip_opts=opts.TooltipOpts(trigger="axis", axis_pointer_type="cross"),
```

设置了浏览图片时的提示方式，基于十字准星指示器。

效果如下所示：









