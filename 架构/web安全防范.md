# SQL注入
## 定义
利用后端程序的漏洞，针对数据库（主要是关系型数据库）进行攻击。攻击者通常通过输入精心构造的参数，绕过服务器端的验证，来执行恶意 SQL 语句，SQL注入会造成拖库（原始数据表被攻击者获取），绕过权限验证，或者篡改、破坏、删除数据
## 例子
对于一个登陆验证的请求一般需要通过用户输入的用户名和密码进行查询验证，查询SQL 语句如下：
```
SELECT * FROM users WHERE username = "$username" AND password = "$password";
```
攻击者已知一个用户名archer2017，可以构造密码为 anywords" OR 1=1，此时，此时后端程序的执行的SQL语句为
```
SELECT * FROM users WHERE username = "archer2017" AND password = "anywords" OR 1=1;
```
## 防范
大多数情况都是拼装 SQL 语句造成的，防范 SQL 注入，主要的方式就是避免直接使用用户输入的数据。
- 使用预编译语句（PreparedStatement）：一方面可以加速 SQL 的执行，一方面可以防止SQL注入。
    - SQL 语句的编译分为如下几个阶段：
    - 基本解析：包括SQL语句的语法、语义解析，以及对应的表和列是否存在等等。
    - 编译：将 SQL 语句编译成机器理解客理解的中间代码格式。
    - 查询优化：编译器在所有的执行方案中选择一个最优的。
    - 缓存：缓存优化后的执行方案。(使用占位符来替代参数，从而实现预编译)
    - 执行阶段：执行最终查询方案并返回给用户数据。
- 对用户的输入进行过滤或者转义
- 注.在微软的数据技术LINQ向.NET开发人员彻底清除了SQL注入的问题，LINQ是以多个子表达式组成的，这些表达式会根据映射来产生安全的SQL代码，这个过程是自动内置的，无法干预
# CSRF跨站点请求伪造
## 定义
CSRF攻击者通过构造的第三方页面诱导受害者完成加载或者点击，利用受害者的权限（通常是某已登录网站的Cookie），以其身份向合法网站发起恶意请求，来进行比如虚拟货币的转账，账号信息修改，发恶意邮件等
## 例子
CSRF 攻击有一个前提条件，是用户具有某个网站正常访问的访问权限。   
一般网站的访问都具备一定的有效期（cookie属性设置expires），在此期间权限信息会保留在用户浏览器的cookie中   
![CSRF](https://coderxing.gitbooks.io/architecture-evolution/assets/4D0E55F2-2330-4A5E-BB7D-2B379C35BBC2.png)   
- 攻击者利用正常网站A的CSRF漏洞，构造一个恶意网站B
- 恶意网站中包含对网站A的请求
- 用户访问A以后恰好访问了B（假定此时身份验证信息未过期）
- 在用户访问B时，会触发攻击请求，修改用户信息等等比如
```
<img widht=0 height=0 src="http://normal-site.com/transfer.do?from=rommel&to=attacker&amount=100" />
```
## 防范
- 添加HTTP请求首部中的Referer白名单
    - 若请求的域名不在白名单内则拒绝请求
- 令牌Token验证
    - 用户在提交表单或者修改数据之前，给用户生成一个随机token，提交数据时需要验证token
    - 一般作为Post字段或者ajax头信息
```
<form>
    <input name="from" value="rommel" />
    <input name="to" value="" />
    <input name="csrf_token" value="QMYjiBlZ9V9mGnap" />
</form>
```
```
$.ajax({
    headers: {
    "X-CSRF-TOKEN": "QMYjiBlZ9V9mGnap"
    }
});
```
- 二次验证：在修改敏感信息或操作时，增加验证码
- 设置Set-cookie的 SameSite属性，在SameSite 的限制下，Cookie 的传输必须在同一个域名下。SameSite 属性有两种限制模式 strict 和 lax，默认为 lax。
# XSS跨站脚本攻击
## 定义
恶意攻击者利用网站没有对用户提交数据进行转义处理或者过滤不足的缺点，进而添加一些代码，嵌入到web页面中去。使别的用户访问都会执行相应的嵌入代码。
## 例子
1. 留言、发帖  
    1. 假若用户填写数据为：
    ```
    <script>alert('foolish!')</script>
    ```
    2. 提交后将会弹出一个foolish警告窗口，接着将数据存入数据库   
    3. 等到别的客户端请求这个留言的时候，将数据取出显示留言时将执行攻击代码，将会显示一个foolish警告窗口。
2. 盗取cookie
    1. 填写数据时插入读取cookie的JS到图片或者y二面
    2. 每次用户访问时，都会将浏览器的cookie传到攻击者网站
## 防范
- 在表单提交或者url参数传递前，对内容进行过滤转义
- 指定HTTP头的内容类型：content-type
- 注.net的解决方法
    - Razor语法输出的内容已经被编码，可以不做任何其他处理
    - 对输入内容进行编码来阻止：Html.Encode，Html.AttributeEncode，Url.Encode
# 上传文件漏洞
## 定义
攻击者可以用利用服务器端的上传文件漏洞绕过安全验证将代码提交到服务器端，并想办法让代码文件被执行。一单可执行的代码上传成功，会造成比较严重的安全问题
## 举例
- 将脚本伪造成图片之类的上传限制文件

- 使用 curl 命令上传文件，并将 Content-Type 改成 image/jpeg。
- 在图片的comment添加恶意代码
- 构造文件名比如xxx.php%00.jpg，使用%00截断，并欺骗上传类型检测
- 当运行用户修改已上传文件时，用户修改文件后缀
## 防范
- 对文件类型做校验：通过白名单方式判断文件类型和扩展名是否是程序需要的（不可靠）

- 文件和程序分开存储：文件单独存放在一个服务器上,并设置目录不可执行。比如上传到cdn服务器上，用户保存文件的服务器只提供基本的文件存取功能，不提供脚本执行能力。
- 对上传的图片进行重绘：压缩函数或者resize函数或者直接进行重绘,破坏里面可能包含的代码
- 合理设置服务器权限：对于应用服务器程序，如果不在本机存储文件，可以去掉应用程序所在目录的写权限。对于文件服务器，可以去掉可执行权限。
- 对上传文件重命名：对上传文件重新命名，最好具有一定的随机性，提高攻击成本。
- 改写文件路径
- 单独设置文件服务器的域名:由于浏览器同源策略的关系，一系列客户端攻击将失效
# 参考资料
[ASP.NET MVC编程——验证、授权与安全](https://www.cnblogs.com/hdwgxz/p/8637421.html)   
[跨網站指令碼](https://zh.wikipedia.org/wiki/%E8%B7%A8%E7%B6%B2%E7%AB%99%E6%8C%87%E4%BB%A4%E7%A2%BC)   
[攻防：文件上传漏洞的攻击与防御](http://www.h3c.com/cn/d_201408/839582_30008_0.htm)   
[上传文件过滤](https://coderxing.gitbooks.io/architecture-evolution/di-san-pian-ff1a-bu-luo/641-web-an-quan-fang-fan/6414-shang-chuan-wen-jian-guo-lv.html)   
[SQL 注入](https://coderxing.gitbooks.io/architecture-evolution/di-san-pian-ff1a-bu-luo/641-web-an-quan-fang-fan/6413-sql-zhu-ru.html)   
[CSRF](https://coderxing.gitbooks.io/architecture-evolution/di-san-pian-ff1a-bu-luo/641-web-an-quan-fang-fan/6412-csrf.html)   

    