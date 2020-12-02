# JSONP解决前端跨域问题

### JSONP即JSON with Padding

由于同源策略的限制，XmlHttpRequest只允许请求当前源（域名、协议、端口）的资源。如果要进行跨域请求，我们可以通过使用 html的script标记来进行跨域请求，并在响应中返回要执行的script代码，其中可以直接使用JSON传递javascript对象。这种跨域的通讯方式称为JSONP。

ajax只能下载同源的数据，跨源的数据禁止下载。

### 同源策略

1.同协议

2.同域名/同IP

3.同端口

举个简单的例子：

1. http://www.abc.com:3000到https://www.abc.com:3000的请求会出现跨域（协议不同）
2. http://www.abc.com:3000到http://www.def.com:3000的请求会出现跨域（域名不同）
3. http://www.abc.com:3000到http://www.abc.com:3001的请求会出现跨域（端口不同）

### 跨源需求

1.修改ajax的同源协议（安全起见不建议）

2.委托php文件件进行跨域

3.JSONP



#### JSONP跨源

首先，不知道大家有没有注意，不管是我们的script标签的src还是img标签的src，或者说link标签的href他们没有被同源策略所限制，比如我们有可能使用一个网络上的图片，就可以请求得到

 src 这个属性是可以完成跨源。

```JavaScript
<img src="https://ss0.bdstatic.com/70cFuHSh_Q1YnxGkpoWK1HF6hhy/it/u=188173295,510375359&fm=26&gp=0.jpg" >
```


然后我们来试试script中src完成跨域



##### 错误示范

在HTML文件中调用js文件

```JavaScript
<script src="demo.js" type="text/javascript" charset="utf-8"></script>
```

在demo.js直接输入函数

```JavaScript
"i am String!"
```

这样是不行的，没有任何用。这种方式相对于把js里边的代码写在HTML上，而且是个常量。



##### 正确的做法

在js文件中，写个函数，用函数调用的方式

```
download("i am String!");
```

HTML文件中，先声明这个函数在加个参数，然后再引进js文件。

```JavaScript
	<script type="text/javascript" charset="utf-8">
		function download(data)
		{
			alert("下载的数据:"+data);
		}
		
	</script>
	<script src="demo.js" type="text/javascript" charset="utf-8"></script>
```


但是这种做法也是有点小问题。

1.如何在需要的时候加载数据

2.能否引入除js文件以外的其他路径



给按钮添加一个点击事件，点击之后下载数据

```JavaScript
    <script>
        window.onload = function(){
            var oBtn = document.getElementById("btn1");
            oBtn.onclick = function(){
                var oScript = document.createElement("script");
                oScript.src = 'demo.txt';
                document.body.appendChild(oScript);
            }
        }
    </script>
```
对于计算机来说，文件后缀是没有任何用处的。后缀作用：用于给计算机上的软件，快速识别应该用哪个软件打开。



### JSONP跨域的使用流程：

​    1、先去声明一个函数，这个函数有一个形参，这个形参会拿到我们想要下载的数据，使用这个
​    2、在需要下载数据的时候，动态创建script标签，将标签src属性设置成，下载数据的链接
​    3、当script插入到页面上的时候，就会，调用已经封装好的函数，将我们需要的数据传过来。



#### 前后端如何交互

用前端举个例子

```JavaScript
var oScrpipt=document.createElement("script");
oScrpipt.src=`https://query.asilu.com/weather/baidu?city=${oCity.value}&callback=download`;
document.body.appendChild(oScrpipt);
```

主要说下callback这个参数，callback参数就是核心所在。**为什么要定义callback呢？首先我们知道，这个get请求已经被发出去了，那么我们如何接口请求回来的数据呢，callback=download则可以帮我们做这件事。**

**需要注意的是，callback参数定义的方法是需要前后端定义好的，具体什么名字，商讨好就可以了。其实jsonp的整个过程就类似于前端声明好一个函数，后端返回执行函数。执行函数参数中携带所需的数据，整个过程实际非常简单易懂。**



**这里针对ajax与jsonp的异同再做一些补充说明：**

1、ajax和jsonp这两种技术在调用方式上”看起来”很像，目的也一样，都是请求一个url，然后把服务器返回的数据进行处理，因此jquery和ext等框架都把jsonp作为ajax的一种形式进行了封装。

2、但ajax和jsonp其实本质上是不同的东西。ajax的核心是通过XmlHttpRequest获取非本页内容，而jsonp的核心则是动态添加