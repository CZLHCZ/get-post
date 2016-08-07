# get-post

写在前面的话

我们在使用Ajax时,当我们向服务器发送数据时,我们可以采用Get方式请求服务器,也可以使用Post方式请求服务器.那么,我们什么时候该采用Get方式,什么时候该采用Post方式呢?

Get请求和Post请求的区别

1.使用Get请求时,参数在URL中显示,而使用Post方式,则不会显示出来

2.使用Get请求发送数据量小,Post请求发送数据量大

例子

页面的HTML代码:

<html xmlns="http://www.w3.org/1999/xhtml">
<head>
    <title></title>
    <style type="text/css">
        *
        {
            margin:8px;
        }
    </style>
</head>
<body>
    <label for="txt_username">
        姓名:</label>
    <input type="text" id="txt_username" />
    <br />
    <label for="txt_age">
        年龄:</label>
    <input type="text" id="txt_age" />
    <br />
    <input type="button" value="GET" id="btn_get" onclick="btn_get_click();" />
    <input type="button" value="POST" id="btn_post" onclick="btn_post_click();" />
    <div id="result">
    </div>
</body>
</html>
区别:

 	Get请求	Post请求
客户端脚
本
代
码	
function btn_get_click() {
    var xmlHttp = window.XMLHttpRequest ? 
        new XMLHttpRequest() : new ActiveXObject("Microsoft.XMLHTTP");

    var username = document.getElementById("txt_username").value;
    var age = document.getElementById("txt_age").value;

    //添加参数,以求每次访问不同的url,以避免缓存问题
    xmlHttp.open("get", "Server.aspx?username=" + encodeURIComponent(username)
        + "&age=" + encodeURIComponent(age) + "&random=" + Math.random());

    xmlHttp.onreadystatechange = function () {
        if (xmlHttp.readyState == 4 && xmlHttp.status == 200) {
            document.getElementById("result").innerHTML = xmlHttp.responseText;
        }
    }

    //发送请求,参数为null
    xmlHttp.send(null);
}
function btn_post_click() {
    var xmlHttp = window.XMLHttpRequest ?
        new XMLHttpRequest() : new ActiveXObject("Microsoft.XMLHTTP");

    var username = document.getElementById("txt_username").value;
    var age = document.getElementById("txt_age").value;
            
    var data = "username=" + encodeURIComponent(username)
        + "&age=" + encodeURIComponent(age);

    //不用担心缓存问题
    xmlHttp.open("post", "Server.aspx", true);

    //必须设置,否则服务器端收不到参数
    xmlHttp.setRequestHeader("Content-Type", "application/x-www-form-urlencoded");

    xmlHttp.onreadystatechange = function () {
        if (xmlHttp.readyState == 4 && xmlHttp.status == 200) {
            document.getElementById("result").innerHTML = xmlHttp.responseText;
        }
    }

    //发送请求,要data数据
    xmlHttp.send(data);
}
区别:

1.get请求需注意缓存问题,post请求不需担心这个问题

2.post请求必须设置Content-Type值为application/x-form-www-urlencoded

3.发送请求时,因为get请求的参数都在url里,所以send函数发送的参数为null,而post请求在使用send方法时,却需赋予其参数

对于客户端代码中都请求的server.aspx,我们来看server.aspx.cs中的代码:

protected void Page_Load(object sender, EventArgs e)
{
    string username = string.Empty;
    int age = 0;

    if (Request.HttpMethod.ToUpper().Equals("GET"))
    {
        username = Request.QueryString["username"];

        age = int.Parse(Request.QueryString["age"]);
    }
    else
    {
        username = Request.Form["username"];

        age = int.Parse(Request.Form["age"]);
    }

    Response.Clear();

    Response.Write("姓名:'" + username + "'<br/>年龄:" + age + "<br/>时间:'" + DateTime.Now.ToString() + "'");

    Response.End();
}
此处,我们发现了get请求和post请求在服务器端的区别:

在客户端使用get请求时,服务器端使用Request.QueryString来获取参数,而客户端使用post请求时,服务器端使用Request.Form来获取参数.

关于服务器端获取数据,我们还可以使用一个通用的获取参数的方式即Request["username"],但是此方法存在一个问题,我们随后来讲.

下面,我们使用HttpWatch来看下,当使用get和post方式发送请求时,客户端究竟发送了什么,收到了什么.

对于get请求和post请求中的时间差,请不要在意,因为是在不同时间按下的get按钮和post按钮.

 	OverView
Get请求	image
Post请求	image
从请求的url可以看出,get请求是带着参数的,post请求的url则不带.

 	Header
Get请求	image
Post请求	image
因为访问的是同一个服务器,所以从服务器获取的信息都是一致的.但是客户端发送的却不一样了.

 	Header
Get请求	image
Post请求	image
从cache可以看出,get请求在发送后,即被缓存,而post请求时 never cached.

 	Query String
Get请求	image
Post请求	image
因为get请求的参数是在url中的,所以Query String中是有值的.而post请求则没有.

 	POST Data
Get请求	image
Post请求	image
在Post Data里,因为get请求的字符串是在url中附带的,所以Post Data中无数据.

 	Content(从服务器获取的数据)
Get请求	image
Post请求	image
从服务器获取的内容都是一致的.

 	 	Stream
Get请求	发送给服务器的	
GET /AjaxWeb/Article7/Server.aspx?username=%E5%BC%A0%E4%B8%89&age=33&random=0.34838340505348675 HTTP/1.1
Accept: */*
Accept-Language: zh-cn
Referer: http://localhost/AjaxWeb/Article7/Ajax.htm
Accept-Encoding: gzip, deflate
User-Agent: Mozilla/4.0 (compatible; MSIE 7.0; Windows NT 5.2; Trident/4.0; Mozilla/4.0 (compatible; MSIE 6.0; Windows NT 5.1; SV1) ; .NET CLR 1.1.4322; .NET CLR 2.0.50727; .NET CLR 3.0.04506.648; .NET CLR 3.5.21022; InfoPath.2; .NET4.0C; .NET4.0E)
Host: localhost
Connection: Keep-Alive

从服务器获取的	
HTTP/1.1 200 OK
Date: Sun, 05 Jun 2011 15:36:27 GMT
Server: Microsoft-IIS/6.0
X-Powered-By: ASP.NET
X-AspNet-Version: 4.0.30319
Cache-Control: private
Content-Type: text/html; charset=utf-8
Content-Length: 60

濮撳悕:'寮犱笁'<br/>骞撮緞:33<br/>鏃堕棿:'2011-6-5 23:36:27'

Post请求	发送给服务器的	
POST /AjaxWeb/Article7/Server.aspx HTTP/1.1
Accept: */*
Accept-Language: zh-cn
Referer: http://localhost/AjaxWeb/Article7/Ajax.htm
Content-Type: application/x-www-form-urlencoded
Accept-Encoding: gzip, deflate
User-Agent: Mozilla/4.0 (compatible; MSIE 7.0; Windows NT 5.2; Trident/4.0; Mozilla/4.0 (compatible; MSIE 6.0; Windows NT 5.1; SV1) ; .NET CLR 1.1.4322; .NET CLR 2.0.50727; .NET CLR 3.0.04506.648; .NET CLR 3.5.21022; InfoPath.2; .NET4.0C; .NET4.0E)
Host: localhost
Content-Length: 34
Connection: Keep-Alive
Cache-Control: no-cache

username=%E5%BC%A0%E4%B8%89&age=33

从服务器获取的	
HTTP/1.1 200 OK
Date: Sun, 05 Jun 2011 15:47:39 GMT
Server: Microsoft-IIS/6.0
X-Powered-By: ASP.NET
X-AspNet-Version: 4.0.30319
Cache-Control: private
Content-Type: text/html; charset=utf-8
Content-Length: 60

濮撳悕:'寮犱笁'<br/>骞撮緞:33<br/>鏃堕棿:'2011-6-5 23:47:39'

比较一下,get请求的url带参数,post请求的url不带参数.post请求是不会被缓存的.

现在,我们来思考另一个问题:

刚才我们说过,服务器在接受参数时,可以采用一个通用的方法,即:Request["username"]来接受参数此方式可以接受get和post请求发送的参数,那么,我们做个测试,在get请求中设置Content-Type,并且send方法中也发送了username=zhangsan,我们看看服务器究竟是返回什么值呢?修改服务器代码如下:

protected void Page_Load(object sender, EventArgs e)
{
    string username = string.Empty;
    int age = 0;

    username = Request["username"];

    age = int.Parse(Request["age"]);

    Response.Clear();

    Response.Write("姓名:'" + username + "'<br/>年龄:" + age + "<br/>时间:'" + DateTime.Now.ToString() + "'");

    Response.End();
}
客户端中,修改btn_get_click()方法如下:

//直接输入张三作为username参数的值
xmlHttp.open("get", "Server.aspx?username=" + encodeURIComponent("张三")
    + "&age=" + encodeURIComponent(age) + "&random=" + Math.random());

//在get请求中添加Content-Type信息
xmlHttp.setRequestHeader("Content-Type", "application/x-www-form-urlencoded");

...

//发送请求,并附带username=zhangsan
xmlHttp.send(username = "zhangsan");
测试代码,结果输出的是"张三".

同样,紧接上面的代码,我们再来做另一个测试,修改post请求,给open方法的url加一个username参数,值为zhangsan.

xmlHttp.open("post", "Server.aspx?username=zhangsan", true);
此时,我们再来运行项目,服务器返回的结果是什么呢?此时我们发现出现的结果是zhangsan.

当我们在get和post请求时,同时在url中、send方法的data中都放置了参数,为什么获取的总是url中的参数值呢?

答:在使用Request时,其会从QueryString,Form,ServerVariable中遍历一番,如果发现符合要求的数据,那么就会停止向后搜寻.所以,我们上例中获取的username实际上都是url中的username值.

何时使用Get请求,何时使用Post请求

Get请求的目的是给予服务器一些参数,以便从服务器获取列表.例如:list.aspx?page=1,表示获取第一页的数据

Post请求的目的是向服务器发送一些参数,例如form中的内容.

下面使用实例来表示Get请求和Post请求在发送同一段数据时的区别.
