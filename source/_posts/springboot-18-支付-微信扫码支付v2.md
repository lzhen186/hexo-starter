---
title: SpringBoot 18.支付 微信扫码支付V2
date: 2022-07-04T01:16:43.608Z
tags: [springboot]
---
## SPRING BOOT集成微信NATIVE扫码支付

标签： [支付](https://www.freesion.com/tag/%E6%94%AF%E4%BB%98/)  [java](https://www.freesion.com/tag/java/)

#### 一.微信支付类型

[微信开放文档](https://pay.weixin.qq.com/wiki/doc/api/index.html)
![在这里插入图片描述](https://www.freesion.com/images/895/3b5fa71bf9dff86435d789a7532fdb47.png)

#### 二. NATIVE扫码支付

##### 2.1. 业务流程

![在这里插入图片描述](https://www.freesion.com/images/786/5cd99bf7420bbdae5c41572645c1187a.png)
（1）商户后台系统根据用户选购的商品生成订单。

（2）用户确认支付后调用微信支付【统一下单API】生成预支付交易；

（3）微信支付系统收到请求后生成预支付交易单，并返回交易会话的二维码链接code_url。

（4）商户后台系统根据返回的code_url生成二维码。

（5）用户打开微信“扫一扫”扫描二维码，微信客户端将扫码内容发送到微信支付系统。

（6）微信支付系统收到客户端请求，验证链接有效性后发起用户支付，要求用户授权。

（7）用户在微信客户端输入密码，确认支付后，微信客户端提交授权。

（8）微信支付系统根据用户授权完成支付交易。

（9）微信支付系统完成支付交易后给微信客户端返回交易结果，并将交易结果通过短信、微信消息提示用户。微信客户端展示支付交易结果页面。

（10）微信支付系统通过发送异步消息通知商户后台系统支付结果。商户后台系统需回复接收情况，通知微信后台系统不再发送该单的支付通知。

（11）未收到支付通知的情况，商户后台系统调用【查询订单API】。

（12）商户确认订单已支付后给用户发货。

##### 2.2. 后端代码编写

###### 2.2.1 引入依赖

```
 		<!--微信支付-->
        <dependency>
            <groupId>com.github.wxpay</groupId>
            <artifactId>wxpay-sdk</artifactId>
            <version>RELEASE</version>
        </dependency>
123456
```

###### 2.2.2 配置类编写

```
/*
 * @className: MyWxPayConfig
 * @description 支付所需相关配置类
 * @since JDK1.8
 * @author ljh
 * @createdAt  2020/9/1 0001 
 * @version 1.0.0
 **/
@Configuration
public class MyWxPayConfig implements WXPayConfig {
    /**
     * 获取 App ID
     *
     * @return App ID
     */
    @Override
    public String getAppID() {
        return "wx8397f8696b538317";
    }

    /**
     * 获取 Mch ID
     *
     * @return Mch ID
     */
    @Override
    public String getMchID() {
        return "1473426802";
    }

    /**
     * 获取 API **
     *
     * @return API**
     */
    @Override
    public String getKey() {
        return "T6m9iK73b0kn9g5v426MKfHQH7X8rKwb";
    }

    /**
     * 异步接收微信支付结果通知的回调地址(此处的值随意填写)
     *
     * @return
     */
    public String getNotifyUrl() {
        return "http://localhost:8080/notify";
    }

    /**
     * 获取商户证书内容
     *
     * @return 商户证书内容
     */
    @Override
    public InputStream getCertStream() {
        return null;
    }

    /**
     * HTTP(S) 连接超时时间，单位毫秒
     *
     * @return
     */
    @Override
    public int getHttpConnectTimeoutMs() {
        return 8000;
    }

    /**
     * HTTP(S) 读数据超时时间，单位毫秒
     *
     * @return
     */
    @Override
    public int getHttpReadTimeoutMs() {
        return 10000;
    }

}
1234567891011121314151617181920212223242526272829303132333435363738394041424344454647484950515253545556575859606162636465666768697071727374757677787980
```

###### 2.2.3 扫码支付(这里调用微信的[统一下单]API)

```
	@Service
public class NativeService{

    @Autowired
    MyWxPayConfig payConfig;

    /**
     * 统一下单接口(用于返回二维码的url)
     *
     * @param money      金额
     * @param outTradeNo 商户订单号
     * @param ip         终端IP
     * @return
     */

    public Map<String, String> unifiedOrder(String money, String outTradeNo, String ip) {
        Map<String, String> paramMap = new LinkedHashMap<>(8);
        WXPay wxPay = new WXPay(payConfig);
        //入参 根据微信开放文档的接口入参规则填写即可
        paramMap.put("device_info", "WEB");
        paramMap.put("body", "腾讯充值中心-QQ会员充值");
        paramMap.put("out_trade_no", outTradeNo);
        paramMap.put("total_fee", money);
        paramMap.put("spbill_create_ip", ip);
        paramMap.put("notify_url", payConfig.getNotifyUrl());
        paramMap.put("trade_type", "NATIVE");
        paramMap.put("product_id", "000000000");

        HashMap<String, String> result = new HashMap<>(2);
        try {
            Map<String, String> map = wxPay.unifiedOrder(paramMap);
            //return_code是通信标识，非交易标识，交易是否成功需要查看result_code来判断
            if ("SUCCESS".equals(map.get("return_code"))) {
                if ("SUCCESS".equals(map.get("result_code"))) {
                    result.put("status", "success");
                    result.put("msg", "请求成功");
                    result.put("code_url", map.get("code_url"));
                    result.put("prepay_id", map.get("prepay_id"));
                    result.put("trade_type", map.get("trade_type"));
                    result.put("money", money);
                    result.put("out_trade_no", outTradeNo);
                    return result;
                } else {
                    result.put("status", "fail");
                    result.put("msg", map.get("err_code_des"));
                    return result;
                }
            } else {
                result.put("status", "fail");
                result.put("msg", map.get("return_msg"));
                return result;
            }

        } catch (Exception e) {
            e.printStackTrace();
        }

        result.put("status", "fail");
        result.put("msg", "未知异常");
        return result;
    }

    /**
     * 查询订单
     *
     * @param outTradeNo 商户订单号
     * @return
     */

    public Map<String, String> orderQuery(String outTradeNo) {
        Map<String, String> paramMap = new LinkedHashMap<>(1);
        WXPay wxPay = new WXPay(payConfig);
        paramMap.put("out_trade_no", outTradeNo);
        Map<String, String> result = new HashMap<>();
        try {
            Map<String, String> map = wxPay.orderQuery(paramMap);
            if ("SUCCESS".equals(map.get("return_code"))) {
                if ("SUCCESS".equals(map.get("result_code"))) {
                    //result.put("status", "success");
                    result.put("out_trade_no", map.get("out_trade_no")); //商户订单号
                    switch (map.get("trade_state")) {

                        case "SUCCESS":
                            result.put("trade_type", map.get("trade_type")); //支付类型
                            result.put("trade_state", map.get("trade_state")); //支付状态
                            result.put("transaction_id", map.get("transaction_id")); //微信支付订单号
                            result.put("total_fee", map.get("total_fee")); //支付金额
                            result.put("time_end", map.get("time_end")); //支付完成时间
                            result.put("msg", "支付成功");
                            break;
                        case "REFUND":
                            result.put("msg", "转入退款");
                            break;
                        case "NOTPAY":
                            map.put("msg", "未支付");
                            break;
                        case "CLOSED":
                            map.put("msg", "已关闭");
                            break;
                        case "REVOKED":
                            map.put("msg", "已撤销");
                            break;
                        case "USERPAYING":
                            map.put("msg", "用户支付中");
                            break;
                        case "PAYERROR":
                            map.put("msg", "支付失败");
                            break;
                        default:
                            map.put("msg", "支付失败");
                            break;
                    }

                    return result;
                } else {
                    result.put("status", "fail");
                    result.put("out_trade_no", map.get("out_trade_no")); //商户订单号
                    result.put("msg", map.get("err_code_des"));
                    return result;
                }
            } else {
                result.put("status", "fail");
                result.put("msg", map.get("return_msg"));
                return result;
            }
        } catch (Exception e) {
            e.printStackTrace();
        }

        result.put("status", "fail");
        result.put("msg", "未知异常");
        return result;
    }

}

123456789101112131415161718192021222324252627282930313233343536373839404142434445464748495051525354555657585960616263646566676869707172737475767778798081828384858687888990919293949596979899100101102103104105106107108109110111112113114115116117118119120121122123124125126127128129130131132133134135136
```

###### 2.2.4 生成终端IP工具类

```
/*
 * @className: IpUtil
 * @description 获取终端IP
 * @since JDK1.8
 * @author ljh
 * @createdAt  2020/8/30 0030
 * @version 1.0.0
 **/
public class IpUtil {
    public static String getIpAddr(HttpServletRequest request) {
        String ipAddress = null;
        try {
            ipAddress = request.getHeader("x-forwarded-for");
            if (ipAddress == null || ipAddress.length() == 0 || "unknown".equalsIgnoreCase(ipAddress)) {
                ipAddress = request.getHeader("Proxy-Client-IP");
            }
            if (ipAddress == null || ipAddress.length() == 0 || "unknown".equalsIgnoreCase(ipAddress)) {
                ipAddress = request.getHeader("WL-Proxy-Client-IP");
            }
            if (ipAddress == null || ipAddress.length() == 0 || "unknown".equalsIgnoreCase(ipAddress)) {
                ipAddress = request.getRemoteAddr();
                if (ipAddress.equals("127.0.0.1")) {
                    // 根据网卡取本机配置的IP
                    InetAddress inet = null;
                    try {
                        inet = InetAddress.getLocalHost();
                    } catch (UnknownHostException e) {
                        e.printStackTrace();
                    }
                    ipAddress = inet.getHostAddress();
                }
            }
            // 对于通过多个代理的情况，第一个IP为客户端真实IP,多个IP按照','分割
            if (ipAddress != null && ipAddress.length() > 15) { // "***.***.***.***".length()
                // = 15
                if (ipAddress.indexOf(",") > 0) {
                    ipAddress = ipAddress.substring(0, ipAddress.indexOf(","));
                }
            }
        } catch (Exception e) {
            ipAddress="";
        }
        // ipAddress = this.getRequest().getRemoteAddr();

        return "0:0:0:0:0:0:0:1".equals(ipAddress) ? "127.0.0.1" : ipAddress;
    }
}
1234567891011121314151617181920212223242526272829303132333435363738394041424344454647
```

###### 2.2.5 测试CONTROLLER

```
@RestController
public class PayController {

    @Autowired
    private NativeService nativeService;

    @RequestMapping("/createNative")
    public Map createNative(HttpServletRequest request) {

        return nativeService.unifiedOrder("1", WXPayUtil.generateNonceStr(), IpUtil.getIpAddr(request));

    }

    @RequestMapping("/orderQuery")
    public Map orderQuery(String outTradeNo) {

        int i=0;

        while (true) {
            Map<String, String> map = nativeService.orderQuery(outTradeNo);
            i++;
            if ("SUCCESS".equals(map.get("trade_state"))) {
                return map;
            }

            try {
                //因为这里是循环调用查询订单,所以如果前台超过5分钟没有支付 就会让刷新页面重新生成二维码
                Thread.sleep(3000);
                if (i>=100){
                    System.out.println(i);
                    map.put("status","fail");
                    map.put("msg","二维码已超时");
                    return map;
                }
            } catch (InterruptedException e) {
                e.printStackTrace();
                map.put("msg", "支付失败");
                return map;
            }
        }

    }

}

123456789101112131415161718192021222324252627282930313233343536373839404142434445
```

##### 2.3 前端代码编写

###### 2.3.1 在STATIC目录下创建PAY.HTML

```
<!DOCTYPE html>
<html>

<head>
  <meta charset="UTF-8">
  <meta http-equiv="X-UA-Compatible" content="IE=9; IE=8; IE=7; IE=EDGE">
  <meta http-equiv="X-UA-Compatible" content="IE=EmulateIE7"/>
  <title>微信支付页</title>
  <link rel="stylesheet" type="text/css" href="css/webbase.css"/>
  <link rel="stylesheet" type="text/css" href="css/pages-weixinpay.css"/>
  <!--引入jQuery和qrious相关类库-->
  <script src="js/jquery.min.js"></script>
  <script src="js/qrious.js"></script>


</head>

<body>
<!--head-->
<div>
  <!--主内容-->
  <div class="pay">
    <div class="checkout-tit">
      <h4 class="fl tit-txt">
        <span class="success-icon"></span><span class="success-info">订单提交成功，请您及时付款！订单号：<span id="out_trade_no"></span></span>
      </h4>
      <span class="fr"><em class="sui-lead">应付金额：</em><em class="orange money">￥</em><span id="money"></span>元</span>
      <div class="clearfix"></div>
    </div>
    <div class="checkout-steps">
      <div class="fl weixin">微信支付</div>
      <div class="fl sao">
        <p class="red"></p>
        <div class="fl code">
          <img id="qrious">
          <div class="saosao">
            <p>请使用微信扫一扫</p>
            <p>扫描二维码支付</p>
          </div>
        </div>
        <div class="fl phone">

        </div>

      </div>
    </div>
  </div>

</div>


<script type="text/javascript">

    //页面一加载就调用
    window.onload = function() {
        createNative()
    }

    /*统一下单接口调用*/
    function createNative() {
        $.ajax({
            type: 'get',
            url: 'http://localhost:8080/createNative',
            success: function(data) {
                console.log(data);
                $("#money").html((data.money / 100).toFixed(2))
                $("#out_trade_no").html(data.out_trade_no)
                /*调用后端统一下单接口根据返回的code_url 用qrious生成二维码*/
                var qr = new QRious({
                    element: document.getElementById('qrious'),
                    size: 250,
                    level: 'H', //二维码容错级别 从小到大 : L M Q H
                    value: data.code_url
                });
                orderQuery(data.out_trade_no)
            }
        })
    }

    /*查询订单接口调用*/
    function orderQuery(out_trade_no) {
        $.ajax({
            type: 'get',
            url: 'http://localhost:8080/orderQuery?outTradeNo=' + out_trade_no,
            success: function(data) {
                console.log(data);
	
				//支付成功后跳转成功页面
                if (data.trade_state == "SUCCESS") {
                    location.href = 'paysuccess.html?money=' + ((data.total_fee) / 100).toFixed(2)
                } else {
                    if (data.msg == "二维码已超时") {
                        console.log(data.msg);
                        //二维码超时后重新生成二维码
                        createNative();
                    } else {
						//跳转失败页面
                        location.href = 'payfail.html'
                    }
                }

            }
        })
    }


</script>
</body>

</html>
123456789101112131415161718192021222324252627282930313233343536373839404142434445464748495051525354555657585960616263646566676869707172737475767778798081828384858687888990919293949596979899100101102103104105106107108109110
```

成功页面paysuccess.html

```
<!DOCTYPE html>
<html>

<head>
  <meta charset="UTF-8">
  <meta http-equiv="X-UA-Compatible" content="IE=9; IE=8; IE=7; IE=EDGE">
  <meta http-equiv="X-UA-Compatible" content="IE=EmulateIE7"/>
  <title>支付页-成功</title>


  <link rel="stylesheet" type="text/css" href="css/webbase.css"/>
  <link rel="stylesheet" type="text/css" href="css/pages-paysuccess.css"/>
  <script src="js/jquery.min.js"></script>

</head>

<body>
<!--head-->

<div class="cart">
  <!--logoArea-->
  <div class="logoArea">
    <div class="fl logo"><span class="title">支付页</span></div>
  </div>
  <!--主内容-->
  <div class="paysuccess">
    <div class="success">
      <h3><img src="img/right.png" width="48" height="48">　恭喜您，支付成功啦！</h3>
      <div class="paydetail">
        <p>支付方式：微信支付</p>
        <p>支付金额：￥<span id="money"></span>元</p>
        <p class="button">
          <a href="myOrder.html" class="sui-btn btn-xlarge btn-danger">查看订单</a>&nbsp;&nbsp;&nbsp;&nbsp;<a href="index.html" class="sui-btn btn-xlarge ">继续购物</a>
        </p>
      </div>
    </div>

  </div>
</div>
</body>
<script>
    window.onload = function() {
        var money = getUrlParam("money");
        $("#money").html(money)
    }

    //解析跳转过来时url携带的参数
    function getUrlParam(name) {
        var reg = new RegExp("(^|&)" + name + "=([^&]*)(&|$)"); //构造一个含有目标参数的正则表达式对象
        var r = window.location.search.substr(1).match(reg);  //匹配目标参数
        if (r != null) return unescape(r[2]);
        return null; //返回参数值
    }
</script>
</html>
12345678910111213141516171819202122232425262728293031323334353637383940414243444546474849505152535455
```

支付失败页面payfail.html

```
<!DOCTYPE html>
<html>

	<head>
		<meta charset="UTF-8">
		<meta http-equiv="X-UA-Compatible" content="IE=9; IE=8; IE=7; IE=EDGE">
		<meta http-equiv="X-UA-Compatible" content="IE=EmulateIE7" />
		<title>支付页-失败</title>

		
	
    <link rel="stylesheet" type="text/css" href="css/webbase.css" />
    <link rel="stylesheet" type="text/css" href="css/pages-payfail.css" />
</head>

	<body>
		<!--head-->

		<div class="cart">
			<!--logoArea-->
			<div class="logoArea">
				<div class="fl logo"><span class="title">支付页</span></div>
			</div>
			<!--主内容-->
			<div class="payfail">
				<div class="fail">
					<h3><img src="img/fail.png" width="48" height="48">　支付失败，请稍后再试</h3>
					<div class="fail-text">
					<p>失败原因：不能使用金币购买！</p>
					<p class="button"><a href="" class="sui-btn btn-xlarge btn-danger">重新支付</a></p>
				    </div>
				</div>
				
			</div>
		</div>

<!--页面底部END-->

</body>

</html>
1234567891011121314151617181920212223242526272829303132333435363738394041
```

![在这里插入图片描述](https://www.freesion.com/images/771/99c54b0006683ab2011de309b3d73f43.png)
相关API调用示例可![在这里插入图片描述](https://www.freesion.com/images/80/823bffc25ad71171640cecdcbb9ecbb0.png)
