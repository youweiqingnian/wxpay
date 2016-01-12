###wxpay

#####精简的NATIVE扫码付款  
`' 微信开发推荐：方倍工作室，微信官方文档，大多数的问题官方文档有答案'`  
>-采用微信支付模式二，官方也推荐使用模式二  
>-模式一：https://pay.weixin.qq.com/wiki/doc/api/native.php?chapter=6_4  
>-业务流程说明：  
（1）商户后台系统根据微信支付规定格式生成二维码（规则见下文），展示给用户扫码。  
（2）用户打开微信“扫一扫”扫描二维码，微信客户端将扫码内容发送到微信支付系统。  
（3）微信支付系统收到客户端请求，发起对商户后台系统支付回调URL的调用。调用请求将带  productid和用户的openid等参数，并要求商户系统返回交数据包,详细请见"本节3.1回调数据输入参数"  
（4）商户后台系统收到微信支付系统的回调请求，根据productid生成商户系统的订单。  
（5）商户系统调用微信支付【统一下单API】请求下单，获取交易会话标识（prepay_id）  
（6）微信支付系统根据商户系统的请求生成预支付交易，并返回交易会话标识（prepay_id）。  
（7）商户后台系统得到交易会话标识prepay_id（2小时内有效）。  
（8）商户后台系统将prepay_id返回给微信支付系统。返回数据见"本节3.2回调数据输出参数"  
（9）微信支付系统根据交易会话标识，发起用户端授权支付流程。  
（10）用户在微信客户端输入密码，确认支付后，微信客户端提交支付授权。  
（11）微信支付系统验证后扣款，完成支付交易。  
（12）微信支付系统完成支付交易后给微信客户端返回交易结果，并将交易结果通过短信、微信消息提示用户。微信客户端展示支付交易结果页面。  
（13）微信支付系统通过发送异步消息通知商户后台系统支付结果。商户后台系统需回复接收情况，通知微信后台系统不再发送该单的支付通知。  
（14）未收到支付通知的情况，商户后台系统调用【查询订单API】。  
（15）商户确认订单已支付后给用户发货。  
>-模式二：https://pay.weixin.qq.com/wiki/doc/api/native.php?chapter=6_5   
>-业务流程说明：  
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

1. curl 60错误：  
http://www.mamicode.com/info-detail-923818.html  这里有解决方案，因为curl采用了严格模式，改一下WxApi.class.php就解决  
2. 微信服务器不回调notify_url  
原因：本地配置的虚拟域名或者ip，微信服务器找不到这个链接，同时`'微信官方限制，只能单独访问的url，不能传任何参数'`，例如我写的index.php?c=email&m=sendemail这个链接就不能回调访问，必须是.php结尾的，新建一个1.php，代码为  
```
$url = '###'; //一个发送邮件的链接
file_get_content($url);
echo 'SUCCESS';  //必须给微信返回成功，否则微信服务器将请求多次此链接
```
3. 微信服务器多次请求notify_url  
解决方案如上的代码，结尾`'echo 'SUCCESS';'`一下就可以的，微信服务器就知道成功了  

4. 微信接收微信支付成功账号的参数
 微信并未提供接受方法，最基础可以使用
```
 // 接收xml格式数据然后处理成数组
$xml = file_get_content("php://input");
libxml_disable_entity_loader(true);
$array = json_decode(json_encode(simplexml_load_string($xml 'SimpleXMLElement', LIBXML_NOCDATA)), true);
```
5. 页面跳转友好性
微信的回调只是回调，对页面没有任何影响，在支付成功之后，想跳转页面，使用js的setInterval函数，隔一段时间带上订单号请求一个状态的接口就可以的。
```
 // 每半秒请求一次数据，然后判断，跳转，增加用户友好性
$(function(){
        orderno = $('###').val();
        start = self.setInterval("checkstatus(orderno)", 500);
    });

    function checkstatus(order_no){
        if(order_no == undefined || order_no == ''){
            window.clearInterval(start);
        }
        else{
            $.ajax({
                url:"#####",
                type:'POST',
                data:{orderno:orderno},
                success:function(msg){
                    if(msg == 4) {
                        alert('支付成功');
                        location.href = '###';
                    }
                }
            });
        }
    }
```
**其他curl原因需要检查版本问题了，或者证书，或者是否金庸curl_exec函数，为了安全性服务器有可能禁用这个函数**
 `' 其他遇到再补充'`  