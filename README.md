## v2ray-heroku
[![](https://www.herokucdn.com/deploy/button.png)](https://heroku.com/deploy?template=https://github.com/anthey2/cypher) 

### heroku上部署v2ray
- [x] 支持VMess和VLESS两种协议
- [x] 支持自定义websocket路径
- [x] 伪装首页（3D元素周期表）
- [x] HTML5测速
- [x] 使用v2ray最新版构建

请求`/`，返回3D元素周期表

![image](https://cdn.jsdelivr.net/gh/DaoChen6/Heroku-v2ray/doc/1.png)

请求`/speedtest/`，html5-speedtest测速页面

![image](https://cdn.jsdelivr.net/gh/DaoChen6/Heroku-v2ray/doc/2.png)

请求`/test/`，文件下载速度测试

![image](https://cdn.jsdelivr.net/gh/DaoChen6/Heroku-v2ray/doc/3.png)

请求`/ray`（可配置）v2ray websocket路径
### 提示：UUID请使用UUID生成器，推荐[UUID Generator](https://www.uuidgenerator.net/)

### 提醒：滥用可能导致账户被删除！！！

### 提醒：配置连接方式时请仔细阅读带粗体的注意事项! ! !

### 请之前已经fork过的用户删除项目后重新fork本项目! ! !

### 已恢复Vmess、VLESS、Trojan、Shadowsocks的所有连接! ! !

### 想改其他传输协议的请参考[HTTP路由/HTTP支持版本](https://devcenter.heroku.com/articles/http-routing#http-versions-supported)然后酌情修改，仅限有经验用户修改，因修改传输协议出现连接错误的本项目不承担任何责任！！！

## 服务端创建操作流程

0.给本项目个stars

1.将本项目fork至自己仓库修改`Deploy to Heroku`按键指向地址为自己仓库地址："deploy?template="后面改为自己的仓库地址就可以了

[![Deploy](https://www.herokucdn.com/deploy/button.png)](https://dashboard.heroku.com/new?template=https://github.com/anthey2/cypher) 

2.点击上面紫色`Deploy to Heroku`，会跳转到heroku app创建页面，应用程序名无需填写也能创建，名字会由heroku随机生成，选择节点（美国或者欧洲），新用户只需要自定义UUID码和CADDYIndexPage（参考：[Caddy主页配置](https://github.com/DaoChen6/IF-XTW/blob/master/README.md#5caddy%E4%B8%BB%E9%A1%B5%E9%85%8D%E7%BD%AE))，其他建议保持默认，点击下面deploy，几秒后搞定！   

3.若出现`We couldn't deploy your app because the source code violates the Salesforce Acceptable Use and External-Facing Services Policy.`提示，则返回仓库，>`Setting`>`Repository name`修改仓库名。

4.若执行了第3步修改仓库名的操作，则必须修改app.json中的name和description，十分重要，切记！！！！

5.注意`repository`必须留空以免项目被禁

6.再修改`Deploy to Heroku`按键指向地址为自己仓库地址，重复`2`的操作

7.带有删除线的部分表示已经废弃或不适用

### 环境变量说明

|  名称 | 值  | 说明  |
| ------------ | ------------ | ------------ |
|  PROTOCOL |  vmess<br>vless（可选） |  协议：nginx+vmess+ws+tls或是nginx+vless+ws+tls |
|  UUID |  [uuid在线生成器](https://www.uuidgenerator.net "uuid在线生成器") | 用户主ID  |
|  WS_PATH | 默认为`/ray` |  路径，请勿使用`/speedtest`，`/`，`/test` 等已经被占用的请求路径 |

### 进阶
heorku可以绑卡（应用一直在线，不扣费），绑定域名，套cf，[uptimerobot](https://uptimerobot.com/) 定时访问防止休眠（只监控CF Workers反代地址好了，不然几个账户一起监控没几天就把时间耗完了）

### CloudFlare Workers反代代码（可分别用两个账号的应用程序名（`PROTOCOL`、`UUID`、`WS_PATH`保持一致），单双号天分别执行，那一个月就有550+550小时）
<details>
<summary>CloudFlare Workers单账户反代代码</summary>

```js
addEventListener(
    "fetch",event => {
        let url=new URL(event.request.url);
        url.hostname="appname.herokuapp.com";
        let request=new Request(url,event.request);
        event. respondWith(
            fetch(request)
        )
    }
)
```
</details>

<details>
<summary>CloudFlare Workers单双日轮换反代代码</summary>

```js
const SingleDay = 'app0.herokuapp.com'
const DoubleDay = 'app1.herokuapp.com'
addEventListener(
    "fetch",event => {
    
        let nd = new Date();
        if (nd.getDate()%2) {
            host = SingleDay
        } else {
            host = DoubleDay
        }
        
        let url=new URL(event.request.url);
        url.hostname=host;
        let request=new Request(url,event.request);
        event. respondWith(
            fetch(request)
        )
    }
)
```
</details>

<details>
<summary>CloudFlare Workers每五天轮换一遍式反代代码</summary>

```js
const Day0 = 'app0.herokuapp.com'
const Day1 = 'app1.herokuapp.com'
const Day2 = 'app2.herokuapp.com'
const Day3 = 'app3.herokuapp.com'
const Day4 = 'app4.herokuapp.com'
addEventListener(
    "fetch",event => {
    
        let nd = new Date();
        let day = nd.getDate() % 5;
        if (day === 0) {
            host = Day0
        } else if (day === 1) {
            host = Day1
        } else if (day === 2) {
            host = Day2
        } else if (day === 3){
            host = Day3
        } else if (day === 4){
            host = Day4
        } else {
            host = Day1
        }
        
        let url=new URL(event.request.url);
        url.hostname=host;
        let request=new Request(url,event.request);
        event. respondWith(
            fetch(request)
        )
    }
)
```
</details>

<details>
<summary>CloudFlare Workers一周轮换反代代码</summary>

```js
const Day0 = 'app0.herokuapp.com'
const Day1 = 'app1.herokuapp.com'
const Day2 = 'app2.herokuapp.com'
const Day3 = 'app3.herokuapp.com'
const Day4 = 'app4.herokuapp.com'
const Day5 = 'app5.herokuapp.com'
const Day6 = 'app6.herokuapp.com'
addEventListener(
    "fetch",event => {
    
        let nd = new Date();
        let day = nd.getDay();
        if (day === 0) {
            host = Day0
        } else if (day === 1) {
            host = Day1
        } else if (day === 2) {
            host = Day2
        } else if (day === 3){
            host = Day3
        } else if (day === 4) {
            host = Day4
        } else if (day === 5) {
            host = Day5
        } else if (day === 6) {
            host = Day6
        } else {
            host = Day1
        }
        
        let url=new URL(event.request.url);
        url.hostname=host;
        let request=new Request(url,event.request);
        event. respondWith(
            fetch(request)
        )
    }
)
```
</details>

### 客户端配置

```
  - name: "yourName"
    type: vmess
    server: yourName.workers.dev
    port: 443
    uuid: yourUuid
    alterId: 0
    cipher: auto
    udp: true
    tls: true
    #skip-cert-verify: true
    servername: yourName.workers.dev
    network: ws
    ws-path: /ray
```
