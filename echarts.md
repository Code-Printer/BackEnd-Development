# Echart技术  
Echarts是基于Canvas画布，底层依赖矢量图形库ZRender的JavaScript开源可视化库，提供个性化的数据可视化图表。
项目中的应用：网站的活跃度分析  
## Echarts操作步骤  
1、下载并在项目中引入echarts.min.js文件  
2、创建一个容器,用于将图表在容器中显示  
3、初始化echarts实例对象(echarts.init(document.querySelector(".容器名")))  
4、指定配置项和数据(option)  
5、将配置项设置给echarts实例对象(echarts对象名.setOption(option)) 

## option中主要的配置关键字  
1、series：系列列表。每个系列通过 type 决定自己的图表类型  
2、xAxis：直角坐标系grid中的x轴。  
3、xAxis中的boundaryGap属性: 默认为true，坐标轴两边留白，这时候刻度只是作为分隔线，标签和数据点都会在两个刻度之间的带(band)中间。  
4、yAxis：直角坐标系grid中的y轴。  
5、grid：在直角坐标系内绘图网格的样式。  
6、title：数据表的标题组件。  
7、tooltip：提示框组件。    
8、legend：图例组件。  
9、color：调色盘颜色列表。  
10、stack：数据堆叠，同个类目轴上系列配置相同的stack值后，后一个系列的值会在前一个系列的值上相加。  

代码演示：
```html
option = {
    //设置线条的颜色，后面是个数组
    color:['pink','red','green','blue','gray'],
    //设置图表标题
    title: {
        text: '折线图堆叠1233标题'
    },
    //图表的提示框组件
    tooltip: {
        //触发方式 - 坐标轴
        trigger: 'axis'
    },
    //图例组件
    legend: {
        //series有name了，这里的data可以删除掉
        data: ['邮件营销', '联盟广告', '视频广告', '直接访问', '搜索引擎']
    },
    //网格配置 grid可以控制线形图 柱状图 图标大小
    grid: {
        left: '3%',
        right: '4%',
        bottom: '3%',
        //是否显示刻度标签
        containLabel: true
    },
    //工具箱组件，可以另存为图片等功能
    toolbox: {
        feature: {
            saveAsImage: {}
        }
    },
    //设置x轴的相关配置
    xAxis: {
        type: 'category',
        //线条和y轴是否有缝隙
        boundaryGap: false,
        data: ['周一', '周二', '周三', '周四', '周五', '周六', '周日']
    },
    //设置y轴的相关配置
    yAxis: {
        type: 'value'
    },
    //系列图表配置，决定显示那种类型的图表
    series: [
        {
            name: '邮件营销',
            type: 'line',
            //总量，后面的会堆叠前面的累加起来，删除掉就会折叠了，一般不需要
            //stack: '总量',
            data: [120, 132, 101, 134, 90, 230, 210]
        },
        {
            name: '联盟广告',
            type: 'line',
            //stack: '总量',
            data: [220, 182, 191, 234, 290, 330, 310]
        },
        {
            name: '视频广告',
            type: 'line',
            //stack: '总量',
            data: [150, 232, 201, 154, 190, 330, 410]
        },
        {
            name: '直接访问',
            type: 'line',
            //stack: '总量',
            data: [320, 332, 301, 334, 390, 330, 320]
        },
        {
            name: '搜索引擎',
            type: 'line',
            //stack: '总量',
            data: [520, 932, 901, 934, 1290, 1330, 1320]
        }
    ]
};
``` 
图片说明：  
![result](https://static01.imgkr.com/temp/cd3d7e8d9119479fb540221de58f5037.png)

## 立即执行函数  
为了减少命名冲突，我们可以采用立即执行函数的写法，立即执行函数修饰的变量都是局部变量。  
代码演示：  
```html
// 基础折线图
(function() {

  // 实例化对象
  var myChart = echarts.init(document.querySelector(".bar .chart"));
  // 指定配置和数据
  var option = {
    xAxis: {
        type: 'category',
        data: ['Mon', 'Tue', 'Wed', 'Thu', 'Fri', 'Sat', 'Sun']
    },
    yAxis: {
        type: 'value'
    },
    series: [{
        data: [150, 230, 224, 218, 135, 147, 260],
        type: 'line'
    }]
  };
  
  // 配置项和数据给我们的实例化对象
  myChart.setOption(option);
  
  // 当我们浏览器缩放的时候，图表也等比例缩放
  window.addEventListener("resize", function() {
    // 让我们的图表调用 resize这个方法
    myChart.resize();
  });

})();

// 饼图
(function() {

  // 基于准备好的dom，初始化echarts实例
  var myChart = echarts.init(document.querySelector(".line .chart"));

  // 指定配置和数据
  var option = {
    title: {
        text: '某站点用户访问来源',
        subtext: '纯属虚构',
        left: 'center'
    },
    tooltip: {
        trigger: 'item'
    },
    legend: {
        orient: 'vertical',
        left: 'left',
    },
    series: [
        {
            name: '访问来源',
            type: 'pie',
            radius: '50%',
            data: [
                {value: 1048, name: '搜索引擎'},
                {value: 735, name: '直接访问'},
                {value: 580, name: '邮件营销'},
                {value: 484, name: '联盟广告'},
                {value: 300, name: '视频广告'}
            ],
            emphasis: {
                itemStyle: {
                    shadowBlur: 10,
                    shadowOffsetX: 0,
                    shadowColor: 'rgba(0, 0, 0, 0.5)'
                }
            }
        }
    ]
  };
  
  // 配置项和数据给我们的实例化对象
  myChart.setOption(option);
  
  // 当我们浏览器缩放的时候，图表也等比例缩放
  window.addEventListener("resize", function() {
    // 让我们的图表调用 resize这个方法
    myChart.resize();
  });
  
})();
```   
## 图表随屏幕自适应  
```html
// 当我们浏览器缩放的时候，图表也等比例缩放
  window.addEventListener("resize", function() {
    // 让我们的图表调用 resize这个方法
    myChart.resize();
  });
```

