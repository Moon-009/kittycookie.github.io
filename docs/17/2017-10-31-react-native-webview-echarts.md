---
title: React Native 使用ECharts库展示数据的一次尝试
date: 2017-10-31
layout: post
---
&emsp;&emsp;ECharts是百度开源的一个图表库，图表类型全面且维护也比较及时。工作上需要在移动端以多种图表的形式展示业务数据，考虑到展示效果并参考其他公司的实现，决定使用ECharts作为图表库在移动端使用。
&emsp;&emsp;首先实现的是iOS版，实现这个功能需要使用React Native提供WebVIew组件引入ECharts页面。React Native iOS版的WebView组件是对原生UIWebView的一个封装。由于在以往的开发中，没有涉及很多React Native页面中通过WebView展示复杂页面的场景，这里考虑了几种方案，打算做一个比较。

1. 把基础HTML页面代码（不包含渲染逻辑的空壳页面，下同）与JS代码放在移动端本地，动态从服务器获取需要的数据进行渲染。这种方案主要是想要减少JS代码拉取占用的网络延时，提高渲染速度；
2. 把整个网页作为一个站点放在服务端，移动端WebView读取这个站点。这种方案维护简单，但拉取代码的网络延时将占用一部分渲染时间，到底会对页面渲染有多大的影响，有待在这次尝试中做一个验证；
3. 把基础HTML页面放在服务端，JS放在本地。这种方案的好处是如果页面需要有所改动可以随时更改。此次的方案中的基础HTML页面非常简单，只有一个空的div节点，对这种方案并没有进行验证。考虑到WebView渲染的性能低于React Native的渲染性能（根据官方说法），使用WebView渲染的还只是图表部分，其他页面部分还由React Native来渲染。

&emsp;&emsp;这样需要比较的方案主要是动态获取数据来渲染页面和直接从服务端拉取页面。在实现过程中，第一种方案又衍生了两种实现方案，第一种是使用WebView的JS与React Native代码通信的机制，使用React Native的网络通信模块将数据拉取，传给WebView内部，WebView引用的HTML页面中的JS根据数据进行渲染；第二种是在WebView引用的HTML页面中的JS中拉取数据并渲染。

&emsp;&emsp;这里依次介绍一下几种方案的实现过程。

> 使用WebView的JS与React Native通信机制

&emsp;&emsp;这里首先遇到一个问题，WebView中引用的JS代码无法根据相对路径引用ECharts的库文件。通过上网搜索解决方案，找到一个解决方法。将ECharts的库文件放在XCode的项目中，具体方法是在工程文件上右键选择“Add Files to …”，将库文件引入工程中，也可以先建一个文件夹（工程文件上右键选择New Group）），再把文件放到文件夹中。在React Native引用WebView组件是设置baseUrl属性，路径与放置库文件的路径一致。如：
```
<WebView source=\{\{html:"XXX",baseUrl:"web/"\}\}/>
```
&emsp;&emsp;相应的Xcode的路径为：
&emsp;&emsp;![](/assets/images/17/xcode.png)
&emsp;&emsp;此时在WebView中引用的html文件中就可引用此库文件，代码如下：
```
<script type="javascript" src="echarts.common.min.js" />
```
&emsp;&emsp;接下来的问题是何时触发React Native端获取数据的代码。如果在React Native的componentDidMount方法中触发，WebView中JS代码的库文件可能还没有引用完，这时如果数据返回要求渲染，WebView中的JS会因为找不到库文件而不渲染。这里需要找到一种知道WebView何时完成加载的方法，经过尝试WebView提供的接口onLoadEnd与onLoad等均不能正常工作，只有onNavigationStateChange可被触发，经过尝试可以实现想要的功能，但是onNavigationStateChange是每当WebView渲染状态改变时就会触发，这会造成多次请求数据。另一种解决方法是在HTML页面的JS代码中发送消息给React Native，React Native端在接收到消息后再触发获取数据方法，此种方案可保证数据获取后需要渲染时图表库已加载，且获取数据的方法只触发一次。

> 在WebView引用的JS中获取数据并渲染

&emsp;&emsp;这种方案比较简单，只需要在JS中按照上面的方法引用JS库即可，通过XMLHttpRequest请求数据，待数据返回后渲染图表。

> 搭建服务端站点

&emsp;&emsp;这里使用了Spring MVC搭建了简单的站点，提供图表页面。

&emsp;&emsp;在实现过程中，由于React Native自带的WebView组件无法实现JS与React Native代码的双向通信，这里使用了第三方组件。为了对比不同的第三方组件的性能，这里使用了两种WebView组件，一个是react-native-wkwebview-reborn，一个是react-native-webview-bridge。结合使用的React Native的版本，react-native-wkwebview-reborn使用的版本是0.5.0，react-native-webview-bridge使用的是最新版本。在一个页面中每种组件每个方案都渲染了同一个图表，作为对比每种组件增加了一个渲染完全在移动端静态HTML页面的方案作为对比。经过实验得到的渲染速度的结果是react-native-webview-bridge渲染的优于react-native-wkwebview-reborn。在单个组件的不同方案的对比中，静态页面的渲染速度最快，WebView引用的HTML页面中的JS获取数据并渲染的方案其次，利用WebView与React Native的通信机制的使用React Native获取数据的方案再次，直接拉取服务端页面的方案最慢。其中WebView引用的HTML页面中的JS获取数据并渲染的方案与利用WebView与React Native的通信机制的方案时间上相差不多，均在100ms以内，直接拉取服务端页面的方案需要300-500ms不等。

&emsp;&emsp;由于一开始陷入了WebView与React Native通信的思考中，没有考虑到直接在JS中获取数据这种方案，一直使用的是第三方组件。通过实验得出的结果表明不需要通信也可达到想要的效果，接下来考虑对第三方组件与React Native自带的WebView组件再进行一组对比。

&emsp;&emsp;此次验证只针对了简单数据、简单图表的验证，在实际应用场景中，复杂页面的渲染速度还有待进一步验证。

&emsp;&emsp;另外在这次开发中，在react-native-webview-messaging这个组件中，发现使用webpack似乎可以解决引入本地JS的问题，这也有待探索，毕竟将JS库放入Xcode项目中每次更新都需要App重新打包更新。

&emsp;&emsp;另外这次开发中还发现npm5.x似乎有个bug，安装新包的过程中会删除项目中已有的其他库的文件。通过降级npm并安装相应较低版本的包解决。
