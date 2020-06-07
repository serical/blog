### API参数签名算法

基本参考微信支付的签名算法, [微信支付参数签名算法详情](https://pay.weixin.qq.com/wiki/doc/api/app/app.php?chapter=4_3)

### 客户端使用相关参数

| 参数名    | 解释                                | 来源                           |
| --------- | ----------------------------------- | ------------------------------ |
| appId     | 客户端应用ID                        | 后端提供                       |
| appKey    | 客户端应用秘钥                      | 后端提供                       |
| timestamp | 调用接口时间戳(秒)                  | 前端生成(标准时间戳前后10分钟) |
| nonceStr  | 该次请求随机字符串, 长度在16~32之间 | 前端生成                       |
| signType  | 签名算法,只支持 HMAC-SHA256         | HMAC-SHA256                    |
| sign      | 所有参与请求参数的签名              | 本次请求所有参数计算得出       |

### 签名算法详情

调用一次使用`API参数签名`的接口详细过程

**appKey**

```java
String appKey = "DEMZeWYzDDUvX7EOzEgYS00WObyrOniaAm5gVe0KFdL6vA";
```

**公共参数**

```javascript
{
	"appId":"Vl5gbYRrQ8IDbAEpX2jviVy2Yy84",
	"timestamp":"1591501212",
	"nonceStr":"prni9m312nenw5i0d3tr9t1j77x6chty",
	"signType":"HMAC-SHA256",
}
```

**接口参数**

```javascript
{
	"a":"aaa",
	"b":"1"
}
```

**第一步：对参数按照key=value的格式，并按照参数名ASCII字典序排序如下：**

参与这一步的只有`公共参数`+`接口参数`一共6个, `appKey`不参与这一步

```javascript
a=aaa&appId=Vl5gbYRrQ8IDbAEpX2jviVy2Yy84&b=1&nonceStr=prni9m312nenw5i0d3tr9t1j77x6chty&signType=HMAC-SHA256&timestamp=1591501212
```

**第二步：拼接API密钥：**

`上一步结果`+`&key=appKey`

```javascript
a=aaa&appId=Vl5gbYRrQ8IDbAEpX2jviVy2Yy84&b=1&nonceStr=prni9m312nenw5i0d3tr9t1j77x6chty&signType=HMAC-SHA256&timestamp=1591501212&key=DEMZeWYzDDUvX7EOzEgYS00WObyrOniaAm5gVe0KFdL6vA
```

**第三步：HMAC-SHA256签名, 然后转化为十六进制字符：**

将上一步计算结果进行`HMAC-SHA256`编码, 然后将编码后的`字节数组`转为`16进制`字符串, 即可得到本次签名结果

```java
CA401D1FBD5F514E80763ACD046A8AA9F1E465149BE9705EEF2C599AEE5B3AFB
```

**本次请求最终参数为: **

`公共参数`+`接口参数`+`参数签名`

```javascript
{
	"appId":"Vl5gbYRrQ8IDbAEpX2jviVy2Yy84",
	"timestamp":"1591501212",
	"nonceStr":"prni9m312nenw5i0d3tr9t1j77x6chty",
	"signType":"HMAC-SHA256",
	"a":"aaa",
	"b":"1",
	"sign":"CA401D1FBD5F514E80763ACD046A8AA9F1E465149BE9705EEF2C599AEE5B3AFB"
}
```

**CURL请求例子为: **

```bash
curl --location --request GET 'localhost:8080/api/v1/file/testSign' \
--header 'terminalType: 1' \
--header 'Content-Type: application/json' \
--data-raw '{
	"appId":"Vl5gbYRrQ8IDbAEpX2jviVy2Yy84",
	"timestamp":"1591501212",
	"nonceStr":"prni9m312nenw5i0d3tr9t1j77x6chty",
	"signType":"HMAC-SHA256",
	"a":"aaa",
	"b":"1",
	"sign":"CA401D1FBD5F514E80763ACD046A8AA9F1E465149BE9705EEF2C599AEE5B3AFB"
}'
```

**调用此类接口应该使用Post方式, 参数使用body raw json方式传递最为稳妥, 不会出现参数被URLENCODED的情况**

### Android端可以参考的Java签名代码

**使用示例**

```java
@Test
    public void testSignParam() {
      	// 此处需要注意使用TreeMap直接可以得到ASCII排序参数
        Map<String, String> param = Maps.newTreeMap();
        String appId = "Vl5gbYRrQ8IDbAEpX2jviVy2Yy84";
        String appKey = "DEMZeWYzDDUvX7EOzEgYS00WObyrOniaAm5gVe0KFdL6vA";

        param.put("appId", appId);
        param.put("timestamp", "1591501212");
        param.put("nonceStr", "prni9m312nenw5i0d3tr9t1j77x6chty");
        param.put("signType", "HMAC-SHA256");
        param.put("a", "aaa");
        param.put("b", "1");
        param.put("sign", SignUtil.sign(param, appKey));

        System.out.println(param);
    }
```

**签名代码**

```java
/**
 * api签名工具类
 * 算法参考：https://pay.weixin.qq.com/wiki/doc/api/app/app.php?chapter=4_3
 *
 * @author serical 2019-12-28 03:56:47
 */
public class SignUtil {

    /**
     * 签名算法
     *
     * @param map         参数map
     * @param appKey      appKey
     * @param ignoreField 忽略的属性,如sign|appKey
     * @return 签名
     */
    public static String sign(Map<String, String> map, String appKey, String... ignoreField) {
        // 移除忽略的属性
        if (null != ignoreField && ignoreField.length > 0) {
            for (String field : ignoreField) {
                map.remove(field);
            }
        }

        // 拼接stringA
        StringBuilder stringA = new StringBuilder();
        for (String key : map.keySet()) {
            if (StringUtils.isNotBlank(map.get(key))) {
                stringA.append("&").append(key).append("=").append(map.get(key));
            }
        }

        // 拼接秘钥key
        stringA.append("&").append("key=").append(appKey);
        String text = stringA.substring(1);

        try {
            final String algorithm = HmacAlgorithm.HmacSHA256.getValue();
            final Mac instance = Mac.getInstance(algorithm);
            final SecretKeySpec keySpec = new SecretKeySpec(appKey.getBytes(StandardCharsets.UTF_8), algorithm);
            instance.init(keySpec);
            final byte[] bytes = instance.doFinal(text.getBytes(StandardCharsets.UTF_8));
            return HexUtil.encodeHexStr(bytes).toUpperCase();
        } catch (NoSuchAlgorithmException | InvalidKeyException e) {
            e.printStackTrace();
        }
        return null;
    }
}
```

