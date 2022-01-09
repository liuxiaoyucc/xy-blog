---
title: 十分钟搞定微信H5支付(ThinkPHP5.1)
date: 2021-03-04 09:05:02
categories:
  - - 大熊BIGBEAR
    - PHP
tags:
  - 支付
  - 微信支付
  - 微信H5支付
  - ThinkPHP
  - 大熊BIGBEAR
---

<meta name="referrer" content="no-referrer" />

# 十分钟搞定微信H5支付(ThinkPHP5.1)

之前接过的支付都是jssdk支付, 也就是在微信浏览器环境下的支付, 这次有个项目需要对接微信H5支付, 这里记录下

## 准备
1. 首先微信公众号要开通微信支付功能, 并绑定商户号
2. 开通微信H5支付服务, 审核大概需要1天左右

## 代码部分

#### 配置参数
在`config`目录下, 新建`wechat.php`文件, 写入如下配置
``` php

<?php
/**
 * 微信配置参数
 */
return [
    'auth' => [
        'appid' => '',
        'appsecret'=> '',
    ],
    'pay' => [ // 微信支付用这里的参数
        'appid'=> 'APPID',
        'mch_id'=> 'MCH_ID', // 商户号
        'notify_url'=> '', // 支付结果通知地址, 填写你接收通知的url
        'key'=> 'E918BB87E7BF1B73359AA378550A2598', // 这个key是在商户平台配置的好像, 随便生成一个md5就可以
    ]
];

```

#### 支付类`WechatAppPay.php`

提供订单相关的功能, 例如统一下单(主要), 查询订单信息, 关闭订单等, 这里只贴出了支付需要用到的部分

我将这个库文件, 放在了`application/extra`目录下面, tp5.1 `extra`目录需要自己创建一下就好了, 
如果想放在其他地方, 将代码中的`namespace`改一下再用.


``` php

<?php

namespace app\extra;

class WechatAppPay
{
    //接口API URL前缀
    const API_URL_PREFIX = 'https://api.mch.weixin.qq.com';
    //下单地址URL
    const UNIFIEDORDER_URL = "/pay/unifiedorder";
    //查询订单URL
    const ORDERQUERY_URL = "/pay/orderquery";
    //关闭订单URL
    const CLOSEORDER_URL = "/pay/closeorder";
    //公众账号ID
    private $appid;
    //商户号
    private $mch_id;
    //随机字符串
    private $nonce_str;
    //签名
    private $sign;
    //商品描述
    private $body;

    private $scene_info;

    private $attach;
    //商户订单号
    private $out_trade_no;
    //支付总金额
    private $total_fee;
    //终端IP
    private $spbill_create_ip;
    //支付结果回调通知地址
    private $notify_url;
    //交易类型
    private $trade_type;
    //支付密钥
    private $key;
    //证书路径
    private $SSLCERT_PATH;
    private $SSLKEY_PATH;
    //所有参数
    private $params = array();
    public function __construct($appid, $mch_id, $notify_url, $key)
    {
        $this->appid = $appid;
        $this->mch_id = $mch_id;
        $this->notify_url = $notify_url;
        $this->key = $key;
    }
    /**
     * 下单方法
     * @param   $params 下单参数
     */
    public function unifiedOrder( $params ){
        $this->body = $params['body'];
        $this->out_trade_no = $params['out_trade_no'];
        $this->total_fee = $params['total_fee'];
        $this->trade_type = $params['trade_type'];
        $this->scene_info = $params['scene_info'];
        $this->attach = $params['attach'];
        $this->nonce_str = $this->genRandomString();
        $this->spbill_create_ip = $_SERVER['REMOTE_ADDR'];
        $this->params['appid'] = $this->appid;
        $this->params['mch_id'] = $this->mch_id;
        $this->params['nonce_str'] = $this->nonce_str;
        $this->params['body'] = $this->body;
        $this->params['out_trade_no'] = $this->out_trade_no;
        $this->params['total_fee'] = $this->total_fee;
        $this->params['spbill_create_ip'] = $this->spbill_create_ip;
        $this->params['notify_url'] = $this->notify_url;
        $this->params['trade_type'] = $this->trade_type;
        $this->params['scene_info'] = $this->scene_info;
        $this->params['attach'] = $this->attach;
        //获取签名数据
        $this->sign = $this->MakeSign( $this->params );
        $this->params['sign'] = $this->sign;

        $xml = $this->data_to_xml($this->params);
        $response = $this->postXmlCurl($xml, self::API_URL_PREFIX.self::UNIFIEDORDER_URL);
        if( !$response ){
            return false;
        }
        $result = $this->xml_to_data( $response );
        if( !empty($result['result_code']) && !empty($result['err_code']) ){
            $result['err_msg'] = $this->error_code( $result['err_code'] );
        }
        return $result;
    }
    
    /**
     *
     * 获取支付结果通知数据
     * return array
     */
    public function getNotifyData(){
        //获取通知的数据
//        $xml = $GLOBALS['HTTP_RAW_POST_DATA'];
        $xml = file_get_contents('php://input');
        //echo 123;die;
        $data = array();
        if( empty($xml) ){
            c_log('xml为空');
            return false;
        }
        $data = $this->xml_to_data( $xml );
        if( !empty($data['return_code']) ){
            if( $data['return_code'] == 'FAIL' ){
                return false;
            }
        }
        return $data;
    }
    /**
     * 接收通知成功后应答输出XML数据
     * @param string $xml
     */
    public function replyNotify(){
        $data['return_code'] = 'SUCCESS';
        $data['return_msg'] = 'OK';
        $xml = $this->data_to_xml( $data );
        echo $xml;
        die();
    }
    
    /**
     * 生成签名
     *  @return 签名
     */
    public function MakeSign( $params ){
        //签名步骤一：按字典序排序数组参数
        ksort($params);
        $string = $this->ToUrlParams($params);
        //签名步骤二：在string后加入KEY
        $string = $string . "&key=".$this->key;
        //签名步骤三：MD5加密
        $string = md5($string);
        //签名步骤四：所有字符转为大写
        $result = strtoupper($string);
        return $result;
    }
    /**
     * 将参数拼接为url: key=value&key=value
     * @param   $params
     * @return  string
     */
    public function ToUrlParams( $params ){
        $string = '';
        if( !empty($params) ){
            $array = array();
            foreach( $params as $key => $value ){
                $array[] = $key.'='.$value;
            }
            $string = implode("&",$array);
        }
        return $string;
    }
    /**
     * 输出xml字符
     * @param   $params     参数名称
     * return   string      返回组装的xml
     **/
    public function data_to_xml( $params ){
        if(!is_array($params)|| count($params) <= 0)
        {
            return false;
        }
        $xml = "<xml>";
        foreach ($params as $key=>$val)
        {
            if (is_numeric($val)){
                $xml.="<".$key.">".$val."</".$key.">";
            }else{
                $xml.="<".$key."><![CDATA[".$val."]]></".$key.">";
            }
        }
        $xml.="</xml>";
        return $xml;
    }
    /**
     * 将xml转为array
     * @param string $xml
     * return array
     */
    public function xml_to_data($xml){
        if(!$xml){
            return false;
        }
        //将XML转为array
        //禁止引用外部xml实体
        libxml_disable_entity_loader(true);
        $data = json_decode(json_encode(simplexml_load_string($xml, 'SimpleXMLElement', LIBXML_NOCDATA)), true);
        return $data;
    }
    /**
     * 获取毫秒级别的时间戳
     */
    private static function getMillisecond(){
        //获取毫秒的时间戳
        $time = explode ( " ", microtime () );
        $time = $time[1] . ($time[0] * 1000);
        $time2 = explode( ".", $time );
        $time = $time2[0];
        return $time;
    }
    /**
     * 产生一个指定长度的随机字符串,并返回给用户
     * @param type $len 产生字符串的长度
     * @return string 随机字符串
     */
    private function genRandomString($len = 32) {
        $chars = array(
            "a", "b", "c", "d", "e", "f", "g", "h", "i", "j", "k",
            "l", "m", "n", "o", "p", "q", "r", "s", "t", "u", "v",
            "w", "x", "y", "z", "A", "B", "C", "D", "E", "F", "G",
            "H", "I", "J", "K", "L", "M", "N", "O", "P", "Q", "R",
            "S", "T", "U", "V", "W", "X", "Y", "Z", "0", "1", "2",
            "3", "4", "5", "6", "7", "8", "9"
        );
        $charsLen = count($chars) - 1;
        // 将数组打乱
        shuffle($chars);
        $output = "";
        for ($i = 0; $i < $len; $i++) {
            $output .= $chars[mt_rand(0, $charsLen)];
        }
        return $output;
    }
    /**
     * 以post方式提交xml到对应的接口url
     *
     * @param string $xml  需要post的xml数据
     * @param string $url  url
     * @param bool $useCert 是否需要证书，默认不需要
     * @param int $second   url执行超时时间，默认30s
     * @throws WxPayException
     */
    private function postXmlCurl($xml, $url, $useCert = false, $second = 30){
        $ch = curl_init();
        //设置超时
        curl_setopt($ch, CURLOPT_TIMEOUT, $second);
        curl_setopt($ch,CURLOPT_URL, $url);
        curl_setopt($ch,CURLOPT_SSL_VERIFYPEER,FALSE);
        curl_setopt($ch,CURLOPT_SSL_VERIFYHOST,2);
        //设置header
        curl_setopt($ch, CURLOPT_HEADER, FALSE);
        //要求结果为字符串且输出到屏幕上
        curl_setopt($ch, CURLOPT_RETURNTRANSFER, TRUE);
        if($useCert == true){
            //设置证书
            //使用证书：cert 与 key 分别属于两个.pem文件
            curl_setopt($ch,CURLOPT_SSLCERTTYPE,'PEM');
            //curl_setopt($ch,CURLOPT_SSLCERT, WxPayConfig::SSLCERT_PATH);
            curl_setopt($ch,CURLOPT_SSLKEYTYPE,'PEM');
            //curl_setopt($ch,CURLOPT_SSLKEY, WxPayConfig::SSLKEY_PATH);
        }
        //post提交方式
        curl_setopt($ch, CURLOPT_POST, TRUE);
        curl_setopt($ch, CURLOPT_POSTFIELDS, $xml);
        //运行curl
        $data = curl_exec($ch);
        //返回结果
        if($data){
            curl_close($ch);
            return $data;
        } else {
            $error = curl_errno($ch);
            curl_close($ch);
            return false;
        }
    }
    /**
     * 错误代码
     * @param  $code       服务器输出的错误代码
     * return string
     */
    public function error_code( $code ){
        $errList = array(
            'NOAUTH'                =>  '商户未开通此接口权限',
            'NOTENOUGH'             =>  '用户帐号余额不足',
            'ORDERNOTEXIST'         =>  '订单号不存在',
            'ORDERPAID'             =>  '商户订单已支付，无需重复操作',
            'ORDERCLOSED'           =>  '当前订单已关闭，无法支付',
            'SYSTEMERROR'           =>  '系统错误!系统超时',
            'APPID_NOT_EXIST'       =>  '参数中缺少APPID',
            'MCHID_NOT_EXIST'       =>  '参数中缺少MCHID',
            'APPID_MCHID_NOT_MATCH' =>  'appid和mch_id不匹配',
            'LACK_PARAMS'           =>  '缺少必要的请求参数',
            'OUT_TRADE_NO_USED'     =>  '同一笔交易不能多次提交',
            'SIGNERROR'             =>  '参数签名结果不正确',
            'XML_FORMAT_ERROR'      =>  'XML格式错误',
            'REQUIRE_POST_METHOD'   =>  '未使用post传递参数 ',
            'POST_DATA_EMPTY'       =>  'post数据不能为空',
            'NOT_UTF8'              =>  '未使用指定编码格式',
        );
        if( array_key_exists( $code , $errList ) ){
            return $errList[$code];
        }
    }
}

```

#### 控制器部分

##### 下单接口
我们控制器需要提供接口给前端, 然后将微信的统一下单接口返回的参数返回给前端

首先在控制器顶部引入支付类`WechatAppPay.php`

```php
use app\extra\WechatAppPay;
```
然后新建一个接口函数供前端调用
``` php

    /**
     * 支付
     * @param Request $request
     * @return string
     */
    public function wechat_pay(Request $request)
    {

        $data = $request->post();
        $body = []; // 附加的业务参数, 会在异步通知中原样收到
        $order_no = ''; // 自定义订单号, 根据时间戳或者其他生成一个, 需保证唯一性

        // 初始化
        $lib_wx_pay = new WechatAppPay(config('wechat.pay.appid'), config('wechat.pay.mch_id'), config('wechat.pay.notify_url'), config('wechat.pay.key'));

        $params['body'] = ''; // 商品描述
        $params['out_trade_no'] = $order_no;   //自定义的订单号，不能重复
        $params['total_fee'] = 1; //订单金额 只能为整数 单位为分
        $params['trade_type'] = 'MWEB'; //交易类型, h5支付固定填写 MWEB
        $params['scene_info'] = [ // 支付场景信息
            'h5_info' => [
                'type' => 'Wap', // 固定
                'wap_url' => '', // 这个是调用此接口的页面url, 应该可以忽略
                'wap_name' => '' // 网站名字, 应该也可以忽略
            ]
        ];
        $params['attach'] = json_encode($body); // 自定义附加参数
        $params['scene_info'] = json_encode($params['scene_info']);
        $res = $lib_wx_pay->unifiedOrder($params); // 调用统一下单接口, 接收返回值, 并返回给前端

        $res['order_no'] = $order_no;
        return $res;
    }

```

##### 异步接收微信支付结果接口
这是异步接收微信支付结果的函数, 在这里处理订单状态和一些其他的支付后的业务逻辑, 并按微信规定的格式返回

``` php

    public function notify_wechat(PayRecord $m_pay_record, TestRecord $m_test_record)
    {
        $lib_wx_pay = new WechatAppPay(config('wechat.pay.appid'), config('wechat.pay.mch_id'), config('wechat.pay.notify_url'), config('wechat.pay.key'));

        $data = $lib_wx_pay->getNotifyData(); // 接收通知的参数

        if ($data) {

            $body = json_decode($data['attach'], true);
            $order_no = $data['out_trade_no']; // 自定义订单号

            // 下面根据业务需求处理支付成功的逻辑

        }

        $lib_wx_pay->replyNotify(); // 通知微信服务器已处理
    }

```

##### 支付后同步跳转接口
支付成功后跳转的页面, 如果需要处理些逻辑可以通过控制器, 也可以直接在前端跳转到结果页

``` php

    public function return_wechat_pay(Request $request)
    {
        $return_url = ''; // 跳转结果页
        header("Location: $return_url");
    }

```

#### 前端部分
前端这里提供的是jquery代码, 其实就一个post请求, 用啥都一样

``` javascript

    $.post('/index.php/api/v1/pay/wechat_pay',post_data, function (res) {
        if (res.return_msg === 'OK') {
            // mweb_url是统一下单接口返回的链接
            // redirect_url就是支付成功后跳转的链接, 需要encode后拼接到mweb_url后面,
            // 如果不需要跳转则忽略redirect_url, 不拼接
            let red_url = 'return_wechat_pay';
            red_url = encodeURI(red_url);
            
            // 这样跳转后, mweb_url就会调起微信支付
            window.location.href = res.mweb_url + '&redirect_url=' + red_url;
        }
    })

```


## Tips
1. 如果使用redirect_url这个参数, 会有一个体验很不好的地方, 就是在支付过程中停留超过5s, 
或者由于某些原因支付失败, 也会进行跳转到结果页, 所以需要在结果页由用户去手动查询支付结果, 
可以给个弹窗提示下用户, 是否支付成功, 然后使用订单号去查询后台, 是否真的支付成功