聊一聊H5应用缓存-Manifest
=======================

#### 导读

> Manifest是H5提供的一种应用缓存机制，基于它web应用可以实现离线访问（offline cache）。为此，浏览器还提供了应用缓存的api-applicationCache。虽然manifest的技术已被web标砖飞起，但这不影响我们尝试了解他，也正是因为manifest的应用缓存及时如此诱人，饿了吗和office 365邮箱都还在使用它。

#### 描述

> 鉴于manifest 应用缓存的技术，我们可以做到：
> 
> * 离线访问：即使服务器挂了，或者没有网络，用户依然可以正常浏览网页。
> 
> * 访问更快：数据存在于本地，省去了浏览器发起http请求的时间，因此访问更快，移动端效果更明显。
> 
> * 降低负载：浏览器指在manifest文件该懂事采取服务器下载需要缓存的资源，大大降低了服务器负载。

### manifest缓存的过程（来自网络）

![manifest缓存的过程](http://ww2.sinaimg.cn/large/006y8lVagw1fa479nwxhmj30pm08hdgj.jpg)

### 支持性

> 主流浏览器都支持manifest应用缓存技术，如下表格：

<table>
	<tr>
		<td>IE</td>
		<td>Edge</td>
		<td>Firefox</td>
		<td>Chrome</td>
		<td>Safari</td>
		<td>Opera</td>
		<td>ios</td>
		<td>Android</td>
	</tr>
	<tr>
		<td>10+	</td>
		<td>12+</td>
		<td>3.5+</td>
		<td>4+</td>
		<td>4+</td>
		<td>11.5+</td>
		<td>7.1+</td>
		<td>2.3+</td>
	</tr>
</table>

> H5标准中，[Offline Web applications](https://html.spec.whatwg.org/#offline) 部分有如下描述：

	This feature is in the process of being removed from the Web platform. (This is a long process that takes many years.) Using any of the offline Web application features at this time is highly discouraged. Use service workers instead. [SW](https://html.spec.whatwg.org/#refsSW)

### 如何开启应用缓存

> manifest使用缓存清单进行管理，缓存清单需要与html标签进行关联，如下：

```html
	
	<html manifest="test.appcache">
	...
	</html>

```

> 在html标签中指定manifest文件，便表示该网页使用manifest进行离线缓存，该网页徐亚魂村的文件需要在test。appcache文本文件中指定。

### manifest缓存清单

> 就像写作文一样，manifest采用经典的三段式，分别是CACHE,NETWORK 和FALLBACK.如下，先看一个例子：

```js

	CACHE MANIFEST
	# v1.0.0
	content.css	

	NETWORK
	app.js

	FALLBACK
	/other 404.html
	
``

> 其中第一行必须以CACHE MANIFEST开头，后面可跟若干字符注释，注释从#号开始，更在CACHE MANIFEST 行后的文件，每行留出一个，这些文件是需要缓存的文件。一次content.css会被缓存，不需要访问网络。
>
> 第二段内容以NETWORK:开始，更在该行后的文件表示需要访问网络，如app.js将直接从网络上下载，并不走manifest cache，如果出了第一段中缓存的文件意外，其他文件都从网络上获取，那么此时可将app.js改为*（通配符）。
>
> 第三段内容以：FALLBACK:kaishi1,跟在该行后的文件表示会有一个替代方案。如：当访问/other路径是，如果访问失败，那么将会自动加载404.html作为替代

### manifest缓存状态

> 每个manifest缓存都有一个状态，标示这缓存的情况，一份缓存清单只有一个缓存状态，及时它被多个页面引用，提下是哥哥缓存状态：
> 
> * UNCACHED(为缓存)：表明应用缓存对象还没有初始化完成
> * IDLE(空闲)：应用缓存并未处于更新状态
> * CHECKING(检查)：正在检查是否存在更新
> * DOWNLOADING(下载)：清单更新后，重新下载全部资源到零食缓存中
> * UPDATEREADY(更新就绪)：新版本的缓存下载完成，全部就绪，随机除法时间updayeready
> * OBSOLETE(废弃)：应用缓存已被废弃
> * 上述缓存状态变量一次取值 0,1,2,3,4,5

### applicationCache

> applicationCache是操作应用缓存的瑞士军刀，也是唯一的一把刀
> 
> 首先我们来获取该对象

```js

	// webview下
	var cache = window.applicationCache;
	// shared worker中
	var cache = self.applicationCache;

``` 



