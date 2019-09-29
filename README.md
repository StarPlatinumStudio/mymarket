# 电子商务系统

[TOC]



## 概述

前端编程思想以Vue的***数据驱动视图***为核心

后端使用Spring Boot操作MySQL和Redis数据库提供RESTful接口 

# Spring Boot

### Redis

#### 什么是Redis？

Redis是 一个开源的内存中键值数据存储，用作数据库，缓存和消息代理。在实现方面，键值存储代表NoSQL空间中最大和最旧的成员之一。Redis支持带范围查询的数据结构，例如字符串，哈希，列表，集合和排序集合。

在  Spring Redis的框架, 可以很容易地编写，通过提供一个抽象的数据存储使用Redis的键值存储的Spring应用程序。

Spring Boot驱动操作Redis:

#### Redis配置

我们需要将我们的应用程序与Redis服务器连接。为了建立这种连接，我们使用  Redis客户端实现[Jedis](https://github.com/xetorthio/jedis)。

将  `JedisConnectionFactory` 制成一个bean，以便我们可以创建一个  `RedisTemplate` 查询数据。

#### 使用讯息发布者(Message Publisher)

#### 为了程序的健壮性（solid）创建接口类RedisRepository

#### 增删改查

使用`HashOperations` Spring Data Redis提供的  模板：

```
public void add(final Good good) {
        hashOperations.put(KEY, good.getId(), good.getName());
    }

    public void delete(final String id) {
        hashOperations.delete(KEY, id);
    }
    
    public good findgood(final String id){
        return (good) hashOperations.get(KEY, id);
    }
    
    public Map<Object, Object> findAllgoods(){
        return hashOperations.entries(KEY);
    }

   public String findAddressByID(String id){
        return hashOperations.get("ADDRESS", id).toString();
    }

   public void addAddress(R_Address address){
        hashOperations.put("ADDRESS", address.getId(), address.getName());
    }
}
```

#### 创建Rest Controllers并映射API请求。

```
   @RequestMapping("/keys")
    public @ResponseBody Map<Object, Object> keys() {
        return redisRepository.findAllMovies();
    }

    @RequestMapping("/values")
    public @ResponseBody Map<String, String> findAll() {
        Map<Object, Object> aa = redisRepository.findAllMovies();
        Map<String, String> map = new HashMap<String, String>();
        for(Map.Entry<Object, Object> entry : aa.entrySet()){
            String key = (String) entry.getKey();
            map.put(key, aa.get(key).toString());
        }
        return map;
    }

    @RequestMapping(value = "/add", method = RequestMethod.POST)
    public ResponseEntity<String> add(
        @RequestParam String key,
        @RequestParam String value) {
        
        Movie movie = new Movie(key, value);
        
        redisRepository.add(movie);
        return new ResponseEntity<>(HttpStatus.OK);
    }

    @RequestMapping(value = "/delete", method = RequestMethod.POST)
    public ResponseEntity<String> delete(@RequestParam String key) {
        redisRepository.delete(key);
        return new ResponseEntity<>(HttpStatus.OK);
    }

    @RequestMapping(value = "/addaddress", method = RequestMethod.POST)
    public ResponseEntity<String> address(
            @RequestParam String key,
            @RequestParam String value) {
        R_Address address = new R_Address(key, value);
        redisRepository.addAddress(address);
        return new ResponseEntity<>(HttpStatus.OK);
    }
    @RequestMapping(value = "/getaddress",method = RequestMethod.GET)
    public @ResponseBody String getAddress(@RequestParam String phonenumber){
       return redisRepository.findAddressByID(phonenumber);
    }
```



### MySQL

使用Spring Boot，JPA，Hibernate和MySQL创建REST API

1. 创建Spring Boot项目。

2. 定义数据库配置。

3. 创建一个实体类。

4. 创建JPA数据存储库层。

5. 创建Rest Controllers并映射API请求。

   #### 增删改查示例：

   ```
   @GetMapping("/carts")
   public List<Cart> getCarts() throws Exception {
       return cartRepository.findAll();
   }
   
       @RequestMapping("/addcart")
       public ResponseMsg addCart(@Valid @RequestBody Cart cart){
           List<Cart> carts=cartRepository.findAll();
           for (Cart item:carts
                ) {
               if (item.getItemid().equals(cart.getItemid())) {
                   cart.setNum(cart.getNum() + item.getNum());
                   cart.setId(item.getId());
               continue;
               }
           }
       cartRepository.save(cart);
       return new ResponseMsg("Success",0);
       }
       @RequestMapping("/deletecart")
       public ResponseMsg deleteCart(@Valid @RequestBody Cart cart){
           cartRepository.delete(cart);
           return new ResponseMsg("Success",0);
       }
   ```

   

# Vue.js

### 概述

> Vue (读音 /vjuː/，类似于 **view**) 是一套用于构建用户界面的**渐进式框架**。与其它大型框架不同的是，Vue 被设计为可以自底向上逐层应用。Vue 的核心库只关注视图层，不仅易于上手，还便于与第三方库或既有项目整合。另一方面，当与[现代化的工具链](https://cn.vuejs.org/v2/guide/single-file-components.html)以及各种[支持类库](https://github.com/vuejs/awesome-vue#libraries--plugins)结合使用时，Vue 也完全能够为复杂的单页应用提供驱动。

语言：编写Vue程序语言使用 ECMAScript 6 标准的 JavaScript

思想：面对数据编写JavaScript

AJAX：使用axios进行http通信刷新局部数据

跨域：后端接口统一允许前端Node.js端口开启CORS跨域

### 商城主页

#### 账户

使用了***阿里云短信API***

思想：手机短信/验证码形式登录

登录信息存储至Cookie

去注册，去账户

以手机号为参数调用接口拉取所有数据

可以不登录浏览商品和分类

但是初始状态进入购物车、订单、商品详情需要 ***手机号为参数调用接口*** 的操作时触发登录窗口

登录完成触发axios载入数据



#### 会员购

调用接口从Redis拉取商品信息显示在页面中

使用$route跳转

```
   this.$router.push({ path: "Goods", query: { item: item,phonenumber: username} });
```



#### 商品展示界面

##### 页头滚动渐变效果

```
 // 监听滚动事件
    window.addEventListener(
      "scroll",
      () => {
        this.scrollTop =
          document.documentElement.scrollTop ||
          window.pageYOffset ||
          document.body.scrollTop ||
          document.querySelector(this.el).scrollTop;
        //console.log(document.documentElement.scrollTop);
        this.$data.opacity.opacity = this.scrollTop * 0.001;
        // 控制滚动按钮的隐藏或显示
        if (this.scrollTop > 150) {
          this.isScrollTop = true;
        } else {
          this.isScrollTop = false;
        }
      },
      true
    );
```

##### 添加购物车

```
addCart(phonenumber){
      let data={"fastshot":this.item.ip+this.item.name+this.item.figure,"phonenumber":phonenumber,"itemid":this.item.itemid,"num":this.num,"price":this.item.price}
      this.$http.post(this.serverSrc+"/api/addcart",data).then(res=>{
        if(res.data.ret==0){
           this.$notify({
          title: '添加成功',
          message: '宝贝已经成功添加到您的购物车中',
          type: 'success'
        });
        this.dialogVisible=false
        }
        else{
          this.$notify.error({
          title: '错误',
          message: '发生错误：'+ret.msg
        });
        }
      })
    }
  }
};
```



#### 购物车

![图片](https://i.loli.net/2019/09/16/QS3cfYyaBLmrtnO.png)

使用<code>el-table</code>表格实现购物车整体，



 <template slot="header" slot-scope="scope">......
 <template slot="header" slot-scope="groupscope">......

使用Vue的‘插槽’实现表格内搜索功能和增减删下单操作作用域、数据分离 **插槽显示的位置由子组件自身决定，slot写在组件template的什么位置，父组件传过来的模板将来就显示在什么位置**。



实现搜索功能代码：

```
data="this.mycart.filter(data => !search || data.fastshot.toLowerCase().includes(search.toLowerCase()))"
```



实现购物车的所有方法函数：

-  <code>@selection-change="handleSelectionChange"</code>  在选择器更变时实时改变选择数据的数组

- <code> @click="add(groupscope.$index,mycart)"</code>增加订单数量

- <code> @click="sub(groupscope.$index,mycart)"</code>减少物品数量

- <code>@click="deleteOrder(groupscope.$index,mycart)"</code>删除订单商品

- <code>@click="check(groupscope.$index)"</code>立即下单 单选商品

- <code> @click="toggleSelection(mycart)"</code>反选按钮

- <code>@click="checkArray"</code>立即下单 多选商品

  ![图片](https://i.loli.net/2019/09/16/eJb42GEhnjsvOW3.png)

  多选下单：<code>@selection-change="handleSelectionChange"</code>表格selection更变时记录选择的数据数组，用选择的数据数组在dialog内生成表格，并且用forEach计算订单总金额

#### 订单

MySQL建立order表

```sql
CREATE TABLE `order` (
	`id` int(11) NOT NULL AUTO_INCREMENT,
	`phonenumber` varchar(32) NOT NULL,
	`name` varchar(32) NOT NULL,
	`address` varchar(32) NOT NULL,
	`price` int(11) NOT NULL,
	`data` text NOT NULL,
	PRIMARY KEY (`id`)
) ENGINE=InnoDB
DEFAULT CHARACTER SET=utf8 COLLATE=utf8_general_ci;
```

读取订单

```
  getOrders(){
  this.$http.get("http://localhost:8080/api/orders?phonenumber="+this.username).then(response=>{
  let datas=[]
    response.data.forEach(element=> {
     element.data=JSON.parse(element.data)
      datas.push(element)
    });
      this.orders=datas
    })
},
```

#### 我的

+ 退出登录：恢复data数据为初始值，删除cookie

  删除 cookie 非常简单。您只需要设置 expires 参数为以前的时间即可

  ##### 可以添加/移除收货地址

  ##### 使用全国省市区选择器插件选择地区

  ``` 
   npm install v-distpicker --save
  ```

  以key/value的数据结构将收货地址数组存在Redis


### 控制台 

#### 商品管理

使用<code>el-table</code>暂时商品数据，设置一个<code>slot-scope</code>编写【编辑】、【下架】按钮

#### 上架商品

以key/value的数据结构将收货地址数组存在Redis

##### 上传图片

```
 addImage() {
    let self = this;
    if (self.$refs.new_image.files.length !== 0) {
        var formData = new FormData()
        formData.append('image_data', self.$refs.new_image.files[0]);
        //单个文件进行上传
        self.$http.post(this.serverSrc+'/addimage', formData, {
            headers: {
                "Content-Type": "multipart/form-data"
            }
        }).then(response => {
          if(response.data.msg==0){
            this.iconbtndisabled=true
             this.subiconbtndisabled=false
            this.form.icon=this.serverSrc+'/img/'+response.data.name
             this.$message('封面图片上传成功');
          }
        })
    }
```



# 微信开发

### 接入微信公众平台开发

### 内网穿透

不用花生壳：花生壳个人版无法使用80端口，微信要求80/443端口

使用natapp穿透内网



### 创建Spring Boot项目

配置监听80端口

```
server.port=80
```

编写用SignUtil工具类

```
package com.tlc.eduweb.modles.util;

import java.security.MessageDigest;
import java.security.NoSuchAlgorithmException;
import java.util.Arrays;

public class SignUtil {
    // 与开发模式接口配置信息中的Token保持一致.
    private static String token = "weixinCourse";

    /**
     * 校验签名
     * @param signature 微信加密签名.
     * @param timestamp 时间戳.
     * @param nonce 随机数.
     * @return
     */
    public static boolean checkSignature(String signature, String timestamp, String nonce) {
        // 对token、timestamp、和nonce按字典排序.
        String[] paramArr = new String[] {token, timestamp, nonce};
        Arrays.sort(paramArr);

        // 将排序后的结果拼接成一个字符串.
        String content  = paramArr[0].concat(paramArr[1]).concat(paramArr[2]);

        String ciphertext = null;
        try {
            MessageDigest md = MessageDigest.getInstance("SHA-1");
            // 对拼接后的字符串进行sha1加密.
            byte[] digest = md.digest(content.toString().getBytes());
            ciphertext = byteToStr(digest);
        } catch (NoSuchAlgorithmException e) {
            e.printStackTrace();
        }

        // 将sha1加密后的字符串与signature进行对比.
        return ciphertext != null ? ciphertext.equals(signature.toUpperCase()) : false;
    }

    /**
     * 将字节数组转换为十六进制字符串.
     * @param byteArray
     * @return
     */
    private static String byteToStr(byte[] byteArray) {
        String strDigest = "";
        for (int i = 0; i < byteArray.length; i++) {
            strDigest += byteToHexStr(byteArray[i]);
        }
        return strDigest;
    }

    /**
     * 将字节转换为十六进制字符串.
     * @param mByte
     * @return
     */
    private static String byteToHexStr(byte mByte) {
        char[] Digit = { '0', '1' , '2', '3', '4' , '5', '6', '7' , '8', '9', 'A' , 'B', 'C', 'D' , 'E', 'F'};
        char[] tempArr = new char[2];
        tempArr[0] = Digit[(mByte >>> 4) & 0X0F];
        tempArr[1] = Digit[mByte & 0X0F];

        String s = new String(tempArr);
        return s;
    }

}

```

编码RestController

```
package com.tlc.eduweb.controller;

import com.tlc.eduweb.modles.util.SignUtil;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RestController;

@RestController
@RequestMapping("/weixin")
public class WeixinController {
    @GetMapping("/sign")
    public String Sign(String signature,String timestamp,String nonce,String echostr){
        System.out.println("=======开始请求校验======");
        // 微信加密签名.
        System.out.println("signature====" + signature);
        // 时间戳.
        System.out.println("timestamp====" + timestamp);
        // 随机数.
        System.out.println("nonce====" + nonce);
        // 随机字符串.
        System.out.println("echostr====" + echostr);
        // 请求校验，若校验成功则原样返回echostr，表示接入成功，否则接入失败.
        if (SignUtil.checkSignature(signature, timestamp, nonce)) {
            System.out.println("=======请求校验成功======" + echostr);
           return echostr;
        }
        System.out.println("=======请求校验失败======" + echostr);
        return null;
    }
}

```

### 提交接口配置信息

请填写接口配置信息，此信息需要你有自己的服务器资源，填写的URL需要正确响应微信发送的Token验证，请阅读[消息接口使用指南](http://mp.weixin.qq.com/wiki/index.php?title=消息接口指南)。

URL

http://6t665s.natappfree.cc/weixin/sign

Token

weixinCourse

### ACCESS TOKEN

微信开发工具集:

```
 <dependency>
            <groupId>com.github.liyiorg</groupId>
            <artifactId>weixin-popular</artifactId>
            <version>2.8.17</version>
        </dependency>
```



#### HttpClientUtils

在Oauth网页授权中依然用得到

```
package com.tlc.eduweb.modles.util;
import com.alibaba.fastjson.JSONObject;
import org.apache.http.HttpEntity;
import org.apache.http.HttpResponse;
import org.apache.http.client.methods.HttpGet;
import org.apache.http.impl.client.CloseableHttpClient;
import org.apache.http.impl.client.HttpClients;
import org.apache.http.util.EntityUtils;
import java.io.IOException;
public class HttpClientUtils {
    /**
     * 发起一个GET请求, 返回数据是以JSON格式返回
     * @param url
     * @return
     * @throws IOException
     */
    public static JSONObject doGet(String url) throws IOException {
        JSONObject jsonObject = null;
        CloseableHttpClient client = HttpClients.createDefault();
        HttpGet httpGet = new HttpGet(url);
        HttpResponse response = client.execute(httpGet);
        HttpEntity entity = response.getEntity();
        if (entity != null) {
            String result = EntityUtils.toString(entity, "UTF-8");
            jsonObject = JSONObject.parseObject(result);
        }
        httpGet.releaseConnection();
        return jsonObject;
    }
}

```

##　使用SDK进行高效微信开发

### weixin-popular 微信开发工具集：

```
 <dependency>
            <groupId>com.github.liyiorg</groupId>
            <artifactId>weixin-popular</artifactId>
            <version>2.8.17</version>
        </dependency>
```

WIKI 传送门：https://github.com/liyiorg/weixin-popular/wiki

### 定时任务获取ASSCESS TOKEN

先在main方法启用定时任务

```
@EnableScheduling
@SpringBootApplication
```

```
  @Scheduled(fixedRate= 1000*60*90, initialDelay = 2000)//项目启动2秒中之后执行一次，然后每90min执行一次，单位都为ms
    public void getToken(){
        token = TokenAPI.token(appId,appSecret);
        logger.info("获取Access_token:" + token.getAccess_token());
    }

```

### 自定义菜单

做成接口

```
    @GetMapping("/menuinit")
    public void menuInit(){
        Button sub1 = new Button();
        sub1.setType("view");
        sub1.setName("教育资源");
        sub1.setUrl("http://www.baidu.com/");

        Button sub2 = new Button();
        sub2.setType("click");
        sub2.setName("关于言桥");
        sub2.setKey("click-01");

        Button sub3_1 = new Button();
        sub3_1.setType("scancode_push");
        sub3_1.setName("扫码带提示");
        sub3_1.setKey("rselfmenu_0_0");
        Button sub3_2 = new Button();
        sub3_2.setType("location_select");
        sub3_2.setName("发送位置");
        sub3_2.setKey("rselfmenu_2_0");
        Button group = new Button();
        group.setName("更多功能");
        List<Button> grouplist=new ArrayList<>();
        grouplist.add(sub3_1);
        grouplist.add(sub3_2);
        group.setSub_button(grouplist);

        MenuButtons btn1 = new MenuButtons();
        btn1.setButton(new Button[]{sub1, sub2, group});

        MenuAPI.menuCreate(token.getAccess_token(), btn1);
    }
```

### Oauth网页授权

https://developers.weixin.qq.com/doc/offiaccount/OA_Web_Apps/Wechat_webpage_authorization.html

处理好页面跳转URL逻辑，前端通过路由参数获取用户信息即可

```
/**
     * 网页授权登录
     * @param response
     * @throws IOException
     */
    @RequestMapping("/login")
    public void wxlogin(HttpServletResponse response)throws IOException {
        // 第一步：用户同意授权，获取code
        String url = "https://open.weixin.qq.com/connect/oauth2/authorize?appid=" + appId +
                "&redirect_uri=" + springUrl+"%2Fweixin%2Fwxcallback" +
                "&response_type=code" +
                "&scope=snsapi_userinfo" +
                "&state=STATE#wechat_redirect";
        logger.info("login in"+url);
        response.sendRedirect(url);
    }
    @RequestMapping("/wxcallback")
    public void wxcallback(HttpServletResponse response,String code) throws IOException {
        String url = "https://api.weixin.qq.com/sns/oauth2/access_token?appid="+appId+"&secret="+appSecret+"&code="+code+"&grant_type=authorization_code";
        JSONObject userInfoJson = HttpClientUtils.doGet(url);
        System.out.println("UserInfo:" + userInfoJson);
            response.sendRedirect(vueUrl+"?openid="+userInfoJson.getString("openid")+"&access_token="+userInfoJson.getString("access_token"));
        }

    @GetMapping("/getinfo")
    public JSONObject getInfo(String access_token,String openid) throws IOException{
        String url="https://api.weixin.qq.com/sns/userinfo?access_token="+access_token+"&openid="+openid+"&lang=zh_CN";
        return HttpClientUtils.doGet(url);
    }
```



在拼接的时候需要对地址进行encode转义，进入以下这个地址

https://meyerweb.com/eric/tools/dencoder/

这次需要将我们刚刚设置的网页授权获取用户信息回调域名加上http://进行一下Encode转义，得到转义后的地址，将其拼接到微信获取授权code后的redirect_uri中，这次就不会出现redirect_uri域名还是与后台配置不一致的问题了。

### 微信前端开发

#### 环境准备

host监听

```
 "dev": "webpack-dev-server --inline --progress --config build/webpack.dev.conf.js --host 192.168.124.53",

```

#### 解决Vue invalid host header

```
devServer: {
    disableHostCheck = true,
```

#### 从路由获取id和token，换取用户信息

```
  data() {
    return {
      msg: "Welcome to My Vue.js App",
      openid: this.$route.query.openid,
      access_token: this.$route.query.access_token,
      userinfo:{}
    };
  },
    mounted() {
    this.$http.get("http://7ubcw3.natappfree.cc/weixin/getinfo?openid="+this.openid+"&access_token="+this.access_token).then(response => {
      this.userinfo=response.data
      console.log(response.data);
    });
  },
```

# 线上部署

使用Maven将Spring Boot打包为Jar包：

```
$ clean package -Dmaven.test.skip=true
```



使用Vue.cli默认配置打包Vue项目

```
$ npm run build
```
>  Tip: built files are meant to be served over an HTTP server.
  Opening index.html over file:// won't work.

Vue项目需要使用HTTP服务托管



```
$ java -jar com.michaelcgood-0.0.1.jar
```

在命令行执行jar包（默认8080端口）



"# -" 
