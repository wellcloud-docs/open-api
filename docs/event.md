﻿# 事件订阅与推送

最近更新日期：{docsify-updated}

# 1. 挂断事件订阅

**请求示例**

请求头部字段`Authorization`字段值即申请到的`token`值

```
// general
POST http://tpisdk.wellcloud.cc/api/operation/tenant/createSubscription

// request headers
Authorization: 12345678
Content-Type: application/json;charset=utf-8

// request body
{
	"subscriber": "test-domain.cc",
	"topics": "event.call.test-domain_cc",
	"callbackUrl": "http://test-domain.cc/api",
	"token": "iosdfiajlsdfksdf"
}

// response
{
  "subscriptionId": "http://test-domain.cc/api"
}
```

**路径与查询字符串参数模型**

`POST http://tpisdk.wellcloud.cc/api/operation/tenant/createSubscription`

**请求体说明**

名称 | 是否必须 | 说明
---|---|---
subscriber | 是 | 订阅租户的域名
topics | 是 | 订阅规则。形式必须是：event.call.domain。注意：`domian(域名)中的点必须全部替换成下划线`
callbackUrl | 是 | 回调地址
token | 是 | 用于配置回调地址的验证。并非是目录1中申请的token
id | 否 | 唯一订阅标识


**响应体说明**

名称 |  说明
---|---
subscriptionId | 订阅成功后返回订阅id

# 2. 坐席状态事件订阅

**请求示例**

请求头部字段`Authorization`字段值即申请到的`token`值

```
// general
POST http://tpisdk.wellcloud.cc/api/operation/tenant/createSubscription

// request headers
Authorization: 12345678
Content-Type: application/json;charset=utf-8

// request body
{
    "subscriber ":"008.cc",
    "topics": "event.agent-status.5001.008_cc",
    "callbackUrl": "https://callbackUrl"
    "token": "iosdfiajlsdfksdf"
}
说明：当需要订阅008.cc这一租户下所有坐席的坐席状态事件的时候，topics需要写event.agent-status.*.008_cc


// response
{
  "subscriptionId": "http://test-domain.cc/api"
}
```

**路径与查询字符串参数模型**

`POST http://tpisdk.wellcloud.cc/api/operation/tenant/createSubscription`

**请求体说明**

名称 | 是否必须 | 说明
---|---|---
subscriber | 是 | 订阅租户的域名
topics | 是 | 订阅规则。形式必须是：event.agent-status.<code>.<domain>。注意：`domian(域名)中的点必须全部替换成下划线`
callbackUrl | 是 | 回调地址
token | 是 | 用于配置回调地址的验证。并非是目录1中申请的token
id | 否 | 唯一订阅标识


**响应体说明**

名称 |  说明
---|---
subscriptionId | 订阅成功后返回订阅id


# 3. 回调地址验证

订阅事件过程中，WellCloud平台会发送验证的GET请求到服务器配置的URL上，GET请求携带四个参数：signature（签名）、timestamp（当前时间戳）、nonce（随机数）、echostr（随机字符串）

```
GET http://callbackURL/?signature={{signature}}&timestamp={{timestamp}}&nonce={{nonce}}&echostr={{echostr}}
```

签名/校验流程如下：

1.	将token、timestamp、nonce三个参数进行字典序排序。
2.	将三个参数字符串拼接成一个字符串进行sha1加密。
3.	开发者获得加密后的字符串可与signature对比，如果相同，则说明数据是从我们平台发出的。
4.	返回echostr（随机字符串），返回示例：`{"result":"abxfewsvt23v7sxw"}`。如果无需校验，可在收到get请求后，跳过1，2，3步骤，直接按照示例格式返回echostr（随机字符串）

**sha1 算法 （java）**

```
public final class SHA1 {    
    private static final char[] HEX_DIGITS = {'0', '1', '2', '3', '4', '5',  
                           '6', '7', '8', '9', 'a', 'b', 'c', 'd', 'e', 'f'};    
    /** 
     * Takes the raw bytes from the digest and formats them correct. 
     * 
     * @param bytes the raw bytes from the digest. 
     * @return the formatted bytes. 
     */  
    private static String getFormattedText(byte[] bytes) {  
        int len = bytes.length;  
        StringBuilder buf = new StringBuilder(len * 2);  
        for (int j = 0; j < len; j++) {  
            buf.append(HEX_DIGITS[(bytes[j] >> 4) & 0x0f]);  
            buf.append(HEX_DIGITS[bytes[j] & 0x0f]);  
        }  
        return buf.toString();  
    }    
    public static String encode(String str) {  
        if (str == null) {  
            return null;  
        }  
        try {  
            MessageDigest messageDigest = MessageDigest.getInstance("SHA1");  
            messageDigest.update(str.getBytes());  
            return getFormattedText(messageDigest.digest());  
        } catch (Exception e) {  
            throw new RuntimeException(e);  
        }  
    }  
}  
```

**接口验证示例 （java）**


```

@GET
public String checkSignature(@Context HttpServletRequest request,
		@Context HttpServletResponse response) {
	// 验证URL真实性
	String signature = request.getParameter("signature");// 签名
	String timestamp = request.getParameter("timestamp");// 时间戳
	String nonce = request.getParameter("nonce");// 随机数
	String echostr = request.getParameter("echostr");// 随机字符串

	List<String> params = new ArrayList<String>();
	params.add(Token);
	params.add(timestamp);
	params.add(nonce);
	// 1. 将token、timestamp、nonce三个参数进行字典序排序
	Collections.sort(params, new Comparator<String>() {
		public int compare(String o1, String o2) {
			return o1.compareTo(o2);
		}
	});
	// 2. 将三个参数字符串拼接成一个字符串进行sha1加密
	String temp = SHA1.encode(params.get(0) + params.get(1) + params.get(2));
	if (temp.equals(signature)) { // 3.对比signature
		LogUtil.trace(this, "########## signature successful ##########");
		return echostr; // 4. 返回随机字符串
	} else {
		LogUtil.trace(this, "########## signature error ##########");
	}
	     return "error";
}

```


# 4. 呼叫数据示例与模型说明

在每通电话结束以后，WellCloud平台会向第三方回调地址推送电话的呼叫数据，数据格式是JSON。

## 4.1. 呼叫数据示例

```
{
    "eventName": "CallEndEvent",
    "eventTime": "2017.09.13 11:15:53",
    "eventType": "call",
    "serial": 1,
    "params": {
        "subscriptionId": "http://callbackUrl”
    },
    "_type": "component.cti.event.model.CallEndEvent",
    "topics": [
        "model"
    ],
    "namespace": "final.cc",
    "call": {
        "id": "517cefd0-65fb-4b8f-bc42-d0919a4a3ffe",
        "callType": "Outbound",
        "ani": "8017",
        "dnis": "915856975436",
        "startTime": "2017.09.13 11:15:06",
        "endTime": "2017.09.13 11:15:52",
        "talkLength": 0,
        "answerLength": 0,
        "ringLength": 0,
        "holdLength": 0,
        "abandLength": 0,
        "queuedLength": 0,
        "length": 0,
        "firstStation": "8017@final.cc",
        "endReason": "UserDrop",
		 "failedCode": "9",
        "conference": 0,
        "transferred": 0,
        "internalTransferCount": 0,
        "tenantId": "34bc437b-69ec-4219-90a1-e672dd9d2a9c",
        "namespace": "final.cc",
        "callPartyList": [],
        "callSegments": [],
        "attchedData": {
            "data": {
                "uui": "10000000,1",
                "originalCallId": "d13a82d0-2892-487a-b55f-9b427cff9fcc",
                "originalANI": "8017@final.cc",
                "originalDNIS": "917621180543@final.cc"
            }
        }
    }
}


```

## 4.2. 呼叫数据字段说明

> 表格中没有说明字段请不要使用

参数 | 类型 | 是否必须 | 描述
---|---|--- | ---
eventName | string | 是 | 事件名(CallEndEvent)
eventTime | string | 是 | 事件推送时间
eventType | string | 是 | 事件类型(Call)
call | object | 是 | 通话详细信息

## 4.3. call对象字段说明

> 所有时长单位都是秒

参数 | 类型 | 是否必须 | 描述
---|---|--- | ---
id | string | 是 | callId。呼叫id
callType | enum string | 是 | 呼叫类型。参加下面的呼叫类型表格
ani | string | 是 | 主叫号码
dnis | string | 是 | 被叫号码
startTime | datetime | 是 | 呼叫开始时间
answerTime | datetime | 否 | 通话开始时间。未接通时为空，不会推送该字段
endTime | datetime | 是 | 呼叫结束时间
talkLength | int | 是 | 通话时长
answerLength | int | 是 | 应答时长
ringLength | int | 是 | 振铃时长
holdLength | int | 是 | 保持时长
queuedLength | int | 是 | 排队时长
length | int | 是 | 总时长
firstStation | string | 是 | 一个呼叫中第一次接听分机号
failedCode | enum string | 是 | 呼叫失败编码。参见下面呼叫失败码表格
endReason | enum string | 是 | 挂机原因。参加下面的挂机原因表格
conference | int | 是 | 会议方数量，如为0则代表此通电话没有使用会议
transferred | int | 是 | 转接次数
internalTransferCount | int | 是 | 内线转接次数
tenantId | string | 是 | 租户唯一标示ID
namespace | string | 是 | 租户域名
firstAgentId | string | 是 | 第一次接听电话的座席号（如果座席没有登录，直接使用分机，不会推送）
attchedData | object | 是 | 随路数据

## 4.4. 呼叫类型表格

名称 | 说明
---|---
Inbound | 呼入
Outbound | 呼出
Internal | 内线
Offhook | 摘机
PreOccupied | 预占式外呼
PreDictive | 预测式外呼

## 4.5. 挂机原因表格

名称 | 说明
---|---
Abandon | 放弃
UserDrop | 用户挂断
Transferred | 转接
ExtDrop | 分机挂断
Redirect | 重定向
Unknown | 未知

## 4.6. 呼叫失败码表格

编码 | 说明
---|---
-4 | 挂断
-3 | 线路异常
-2 | 暂无
-1 | 暂无
0 |  成功
1 | 用户忙
2 | 来电提醒
3 | 无法接通
4 | 呼叫限制
5 | 呼叫转移
6 | 关机
7 | 停机
8 | 空号
9 | 正在通话中
10 | 网络忙
11 | 超时
12 | 短音忙
13 | 长音忙
14 | 其他

# 5.坐席状态说明
状态名|解释
---|---
NotReady|示忙
WorkNotReady|话后处理状态
Idle|就绪
Ready|就绪
OnCallIn|呼入通话
OnCallOut|呼出通话
Logout|登出
Ringing|振铃
Allocated|预占中
OffHook|摘机
CallInternal|内部通话
Dailing|外线已经振铃
Ringback|回铃
Conference|会议
OnHold|保持
Other|其他
OnCallPreOccupied|预占式外呼通话
OnPreDictive|预测式外呼通话
OnAgentAutoCall|自动外呼通话
OnPreView|预览式外呼通话

