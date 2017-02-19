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
	
```

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

> 以下是其他属性和方法介绍：
> 
> * status：返回当前页面的应用缓存状态，通常开始应用化黁的页面可能返回1，其他页面返回0
> * update()：手动触发缓存更新
> 
> (1) 若有更新，则依次触发①检查事件（Checking event）②下载事件（Downloading event）③下载进度事件（Progress event）④更新完成事件（UpdateReady event）
> 
> (2) 若无更新，则依次触发①检查事件（Checking event），②无更新事件（NoUpdate event）
> 
> 在未开启应用缓存的页面调用将抛出 Uncaught DOMException 错误
> 
> update()：方法通常在长时间不关闭的页面使用，比如说邮箱应用，用于定期检测可能的更新
> * abort()：取消应用缓存的更新，可用于节省有限的网络速度
> * swapCache()：如果存在一个更新版本的应用缓存，那么它将切换过去，否则将抛出 Uncaught DOMException 错误，通常，我们会在uodateready事件触发一周手动调用 swapCache() 方法，swapCache的切换只对后续加载的缓存文件有效，已经加载成功额资源并不会重新加载。
> 
> 那么如何利用好上述api更新一个页面的应用缓存呢？[Beginner’s Guide to Using the Application Cache](https://www.html5rocks.com/en/tutorials/appcache/beginner/) -文中提供了如下的样板方法：

```js

	// Check if a new cache is available on page load.
	widow.addEventListener('load', function(e) {
		window.applicationCache.addEventListener('updateready', function(e) {
			if (window.applicationCache.status == window.applicationCache.UPDATEREADY) {
				// Brower download a new app cache
				// Swap it in and reload the page to get the new hotness
				window.applicationCache.swapCache();
				if(confirm('A new version of this site is available. Load it?')){
					window.location.reload();
				}
			} else {
				// Mainfest didn`t change. Nothing new to sever.
			}
			
		}, false);
	}, false);

```

### mainfest缓存独立性

> 1. manifest的花村和浏览器默认缓存是两套机制，相互独立，并且不受浏览器缓存大小限制（Chrome 下测试结果）
> 
> 2. 各个manifest文件的缓存相互独立，各自在独立的区域进行缓存，即使是缓存同一个文件，也可能由于缓存的版本不一致，而造成各个页面资源不一致。

### manifest缓存规则

> 1. 遵循全量缓存的规则，即：manifest文件改动后，将重新缓存一边所有文件（包括html本身和动态添加的需要缓存的文件，即使缓存列表中没有该html），第一次缓存过程中，如果出现缓存失败，那么地位低访问，又将重新缓存一边所有文件，以此类推。
> 
> 2. manifest文件本身不能写进缓存清单，否则联通连同html和资源在其缓存失效之前，将永远不能获得更新
> 
> 3. 即使manifest文件丢失，缓存依然有效，不过从此以后引入的manifest的html，将永远不能获得更新。

### webview的缓存现象

> 通常，webview的缓存有如下三种现象：
> *普通网页（无manifest文件），不受manifest缓存影响，缓存只走http cache
> * 包含manifest文件的网页，缓存文件只受manifest影响（只有manifest文件改变时才会更新缓存资源），缓存资源完全与http cache无关，但是NETWORK 段落后需要访问网络的文件，将继续走 http cache
> * webview直接加载manifest缓存过的文件时，有限加载第一个manifest缓存的该文件，如果没找到manifest缓存，那么它将自动虚招http cache 或者 在线加载

### 最佳实践

> * 通常只是用一个manifest文件，并保证缓存文件尽可能的少，以减小manifest每次更新清单中文件所消耗的时间和流量
> * 如果一定要是两个及两个以上的manifest文件，缓存温江尽量不要相同
> * 如果以上两条都不能保证，那么请确保尽可能在manifest缓存的状态更新时，主动刷新网页（此时并不能保证不同网页之间同一个缓存文件版本一致）

###　具体落地步骤

> 1. 如果缓存的文件需要加参数运行，建议将参数内容加到hash中，如cached-page.html#parameterName=value
> 
> 2.　manifest的引入可以使用绝对路径，如果你使用的是绝对路径，那么你的manifest文件必须和你的站点处于同一个域名下
>
> 3. manifest文件可以保存为任意的扩展名，但是响应头中以下字段须取以下定值，以保证manifest'文件被正确解析，并且它没有http缓存

```js
	
	Content-Type: text/cache-manifest
	Cache-Control: max-page=0
	Expries: [CURRENT TIME]
	
```

### 如何更新缓存

> 1. 更新骂你粉丝团文件后，webview将自动更新缓存
> 
> 2. js更新缓存（手动触发manifest更新）：window.applicationCache.update();

### 其他

> chrome浏览器下通过访问[chrome://appcache-internals/](chrome://appcache-internals/)可以查看缓存在本地的资源文件
> 
> 本文参考一篇MDN的文章以及HTML5 Rocks的[Beginner’s Guide to Using the Application Cache](http://www.html5rocks.com/en/tutorials/appcache/beginner/)一文，还有两个个连接可供大家比较阅读。
> * [Cache manifest in HTML5](https://en.wikipedia.org/wiki/Cache_manifest_in_HTML5)on Wikipedia
> * [Offline Web Applications](http://www.w3.org/TR/offline-webapps/)W3C Working Group Note





