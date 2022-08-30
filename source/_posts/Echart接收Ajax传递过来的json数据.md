---
title: Echart接收Ajax传递过来的json数据
top: false
cover: false
toc: true
mathjax: true
date: 2020-09-22 23:37:29
password:
summary: 本文介绍如何利用Echart插件通过ajax接受json数据
tags:
- 原创
- web前端
categories:
- json
---
## 效果图
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200918194249836.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzQzNDE3Mjc2,size_16,color_FFFFFF,t_70#pic_center)

## 源码
前端代码
```
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>图形输出</title>
    <script src="js/jquery-1.9.1.min.js"></script>
    <script src="js/echarts.min.js"></script>
    <script src="js/echarts.helper.js"></script>
</head>
<body onresize="myFunction()">
<!-- 为 ECharts 准备一个具备大小（宽高）的 DOM -->
<div id="main" style="width: 100%;height:400px;"></div>
</body>
<script type="text/javascript">
    function myFunction(){
        var w = window.outerHeight;
        var h =window.outerHeight;
    }
    var myChart = echarts.init(document.getElementById('main'));
    // 显示标题，图例和空的坐标轴
    myChart.setOption({
        title: {
            text: '贪夜蛾自动检测'
        },
        tooltip: {
            trigger: 'axis', //触发类型；轴触发，axis则鼠标hover到一条柱状图显示全部数据，item则鼠标hover到折线点显示相应数据，
            axisPointer: {  //坐标轴指示器，坐标轴触发有效，
                type: 'line', //默认为line，line直线，cross十字准星，shadow阴影
                crossStyle: {
                    color: '#fff'
                }
            }
        },
        legend: {
            data:['红外计数','PM2.5','温度','湿度']
        },
        xAxis: {
            boundaryGap: false,
            data: []
        },
        grid: {
            left: '3%',
            right: '4%',
            bottom: '3%',
            containLabel: true
        },
        toolbox: {
            feature: {
                saveAsImage: {}
            }
        },
        yAxis: {},
        series: [{
            name: '红外计数',
            type: 'line',
            data: []
        },{
            name: 'PM2.5',
            type: 'line',
            data: []
        },{
            name: '温度',
            type: 'line',
            data: []
        },{
            name: '湿度',
            type: 'line',
            data: []
        }]
    });
    myChart.showLoading();    //数据加载完之前先显示一段简单的loading动画
    var names=[];    //类别数组（实际用来盛放X轴坐标值）
    var nums=[];    //销量数组（实际用来盛放Y坐标值）
    var nums1=[];
    var nums2=[];
    var nums3=[];
    
    $.ajax({
        type : "post",
        async : true,            //异步请求（同步请求将会锁住浏览器，用户其他操作必须等待请求完成才可以执行）
        url : "/datass/selectAll",    //请求发送到TestServlet处
        data : {},
        dataType : "json",        //返回数据形式为json
        success : function(result) {
            //请求成功时执行该函数内容，result即为服务器返回的json对象
            if (result) {
                for(var i=0;i<result.length;i++){
                    names.push(result[i].nowtime);    //挨个取出类别并填入类别数组
                }
                for(var i=0;i<result.length;i++){
                    nums.push(result[i].hongwai);    //挨个取出销量并填入销量数组
                }
                for(var i=0;i<result.length;i++){
                    nums1.push(result[i].pm);
                }
                for(var i=0;i<result.length;i++){
                    nums2.push(result[i].wendu);
                    nums3.push(result[i].shidu);
                }
                myChart.hideLoading();    //隐藏加载动画
                myChart.setOption({        //加载数据图表
                    xAxis: {
                        data: names
                    },
                    series: [{
                        // 根据名字对应到相应的系列
                        name: '红外计数',
                        data: nums
                    },{
                        name: 'PM2.5',
                        data: nums1
                    },{
                        name: '温度',
                        data: nums2
                    },{
                        name: '湿度',
                        data: nums3
                    }]
                });
                myChart.setOption(option);
                setTimeout(function (){
                    window.onresize = function (){
                        myChart.resize();
                    }
                },200)

            }

        },
        error : function(errorMsg) {
            //请求失败时执行该函数
            alert("图表请求数据失败!");
            myChart.hideLoading();
        }
    })
    // window.onresize = myChart.resize();
    // $("#main").resize(myChart.resize);
    // myChart.setOption(option);
    myChart.setOption(option);
    setTimeout(function (){
        window.onresize = function (){
            myChart.resize();
        }
    },200)


</script>


</html>
```
后台传送json数据

```
 @RequestMapping("/selectAll")
    @ApiOperation("查询所有")
    public List<Datas> selectAll(Map map){

        return datasService.selectAll(map);
    }
```
前端接收的json数据

```
[{"id":1,"nowtime":"2020-09-16 21:05:15","hongwai":12,"fengxiang":"南偏北风","fengsu":"20.3","wendu":36.5,"shidu":45.1,"pm":52.3,"tWendu10":34.2,"tShidu10":32.1,"tWendu20":30.5,"tShidu20":24.1,"tWendu30":20.4,"tShidu30":40.5},{"id":2,"nowtime":"2020-09-02 17:49:11","hongwai":23,"fengxiang":"北风","fengsu":"21.4","wendu":45.6,"shidu":48.4,"pm":54.7,"tWendu10":45.8,"tShidu10":10.2,"tWendu20":23.5,"tShidu20":24.5,"tWendu30":27.1,"tShidu30":16.4}]
```
