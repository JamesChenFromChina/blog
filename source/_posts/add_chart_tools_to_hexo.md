title: Hexo中增加对Echarts和MathJax的支持
date: 2018-04-25
tags: hexo
categories: Hexo
---

由于最近准备研究下人工智能领域，需要一些绘图工具和LaTeX的支持来记录笔记，所以今天花了些时间在Hexo里面嵌入了对 Echarts和 MathJax的支持，分别可以用来绘制表格关系图和LaTeX。

---

# 对于ECharts的支持

参考 [这里](http://kchen.cc/2016/11/05/echarts-in-hexo/)

可以在blog目录使用命令 `npm install hexo-tag-echarts3 --save` 进行安装,然后在
`node_modules/hexo-tag-echarts/echarts-template.html` 中把原内容修改为

```
<div id="<%- id %>" style="width: <%- width %>;height: <%- height %>px;margin: 0 auto"></div>
<script type="text/javascript">
...
</script>
将上面的部分 1、2 行之间添加一行改成下面的代码：
<div id="<%- id %>" style="width: <%- width %>;height: <%- height %>px;margin: 0 auto"></div>
<script src="http://echarts.baidu.com/dist/echarts.common.min.js"></script>
<script type="text/javascript">
...
</script>
```

之后使用这样的语法生成图表

```
{% echarts 400 '85%' %}
\\TODO option gose here
{% endecharts %}
```

进行编写，例如

```
{% echarts 400 '85%' %}
option = {
    xAxis: { type: 'category', data: ['1','2','3','4','5','6','7']},
    yAxis: { type: 'value' },
    series: [{
        data: [11,22,33,44,55,66,77],
        type: 'line'
    }]

};
{% endecharts %}
```

现实效果如下:

{% echarts 400 '85%' %}
option = {
    xAxis: { type: 'category', data: ['1','2','3','4','5','6','7']},
    yAxis: { type: 'value' },
    series: [{
        data: [11,22,33,44,55,66,77],
        type: 'line'
    }]

};
{% endecharts %}

---

# 对于LaTeX的支持

首先使用了 [hexo-math](https://github.com/hexojs/hexo-math)插件
之后在 _config.yml中增加 hexo-math的配置如下
```
math:
  engine: 'mathjax'
  mathjax:
    src: 'http://cdn.mathjax.org/mathjax/latest/MathJax.js?config=TeX-AMS-MML_HTMLorMML'
    config:
      {
         tex2jax: {
            inlineMath: [ ['$','$'], ["\\(","\\)"] ],
            skipTags: ['script', 'noscript', 'style', 'textarea', 'pre', 'code'],
            processEscapes: true
         }
      }
```


下面的代码展示执行效果
```
{% math %}
$ \sum_{i=0}^{n}i^2 $
{% endmath %}
```
的效果如下

{% math %}
\sum_{i=0}^{n}i^2
{% endmath %}


或者直接使用
```
$ \sum_{i=0}^{n}i^2 $
```

$ \sum_{i=0}^{n}i^2 $




