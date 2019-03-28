**简介**
^^^^^^^^^^

本文档主要针对需要集成HTTP API的技术研发工程师。

| 高重AI开放平台HTTP API使用签名机制对每个接口请求进行权限校验，对于校验不通过的请求，API将拒绝处理，并返回鉴权失败错误。

| 接口调用者在调用API时必须带上接口请求签名，其中签名信息由接口`请求参数`和`应用密钥`根据本文提供的签名算法生成。

| 本文档提供 getReqSign（计算签名） 、doHttpPost（执行HTTP POST调用）两个PHP实现的工具函数方便开发者参考。

**签名算法**
^^^^^^^^^^^^^^

**1. 计算步骤**

用于计算签名的参数在不同接口之间会有差异，但算法过程固定如下4个步骤。

    | 1. 将<key, value>请求参数对按key进行字典升序排序，得到有序的参数对列表N
    | 2. 将列表N中的参数对按URL键值对的格式拼接成字符串，得到字符串T（如：key1=value1&key2=value2），URL键值拼接过程value部分需要URL编码，URL编码算法用大写字母，例如%E8，而不是小写%e8
    | 3. 将应用密钥以app_key为键名，组成URL键值拼接到字符串T末尾，得到字符串S（如：key1=value1&key2=value2&app_key=密钥)
    | 4. 对字符串S进行MD5运算，将得到的MD5值所有字符转换成大写，得到接口请求签名


**2. 注意事项**

- 不同接口要求的参数对不一样，计算签名使用的参数对也不一样

- 参数名区分大小写，参数值为空不参与签名

- URL键值拼接过程value部分需要URL编码

- 签名有效期5分钟，需要请求接口时刻实时计算签名信息

- 更多注意事项，请查看常见问题

**3. 参考代码(PHP)**

以下是使用PHP实现的签名算法代码，开发者可以参考实现其他语言的版本。

::

    // getReqSign ：根据 接口请求参数 和 应用密钥 计算 请求签名
    // 参数说明
    //   - $params：接口请求参数（特别注意：不同的接口，参数对一般不一样，请以具体接口要求为准）
    //   - $appkey：应用密钥
    // 返回数据
    //   - 签名结果
    function getReqSign($params /* 关联数组 */, $appkey /* 字符串*/)
    {
        // 1. 字典升序排序
        ksort($params);

        // 2. 拼按URL键值对
        $str = '';
        foreach ($params as $key => $value)
        {
            if ($value !== '')
            {
                $str .= $key . '=' . urlencode($value) . '&';
            }
        }

        // 3. 拼接app_key
        $str .= 'app_key=' . $appkey;

        // 4. MD5运算+转换大写，得到请求签名
        $sign = strtoupper(md5($str));
        return $sign;
    }

**接口调用实示例**
^^^^^^^^^^^^^^^^^^^

本示例仅供开发者参考，学习如何生成签名以及调用API。

**1. 示例数据**

假设接口请求参数如下，应用密钥为：a95eceb1ac8c24ee28b70f7dbba912bf，其中sign参数用于存储接口请求签名。

============ ==================== ==========================
  参数名称     参数数据              描述
============ ==================== ==========================
  app_id	       1000001	           仅供参考
  image		                           实时计算base64
  time_st	                           实时计算
  nonce_str		                       实时计算
  sign		                           实时计算   
============ ==================== ==========================

关于如何获取应用密钥，请查阅 `新手指引`_ 。

.. _新手指引: https://docsunny1.readthedocs.io/en/latest/guide.html#id2

**2. 计算请求签名**

上述接口请求参数除了sign参数，其他参数均已确定参数值。

| 此时按照签名算法要求，sign的计算代码如下。

::

    // 接口请求参数
    $params = array(
        'app_id'     => '10000',
        'time_stamp' => '1493449657',
        'nonce_str'  => '20e3408a79',
        'key1'       => '腾讯AI开放平台',
        'key2'       => '示例仅供参考',
        'sign'       => '',
    );

    // 应用密钥
    $appkey = 'a95eceb1ac8c24ee28b70f7dbba912bf';

    // 计算sign参数（接口请求签名）
    $params['sign'] = getReqSign($params, $appkey);

    // 得到所有请求参数
    var_dump($params);

上述var_dump($params)的输出结果：

::

    array(6) {
    ["app_id"]=>
    string(5) "10000"
    ["time_stamp"]=>
    string(10) "1493449657"
    ["nonce_str"]=>
    string(10) "20e3408a79"
    ["key1"]=>
    string(20) "腾讯AI开放平台"
    ["key2"]=>
    string(18) "示例仅供参考"
    ["sign"]=>
    string(32) "BE918C28827E0783D1E5F8E6D7C37A61"
    }

可知，sign的计算结果为BE918C28827E0783D1E5F8E6D7C37A61。

**3. 最终请求数据**

在完成sign计算后，即可得到所有接口请求数据，开发者可进入下一步完成API的调用（构造HTTP请求）。

============ ==================== ==========================
  参数名称     参数数据              描述
============ ==================== ==========================
  app_id	       1000001	           仅供参考
  image		                           实时计算base64
  time_st	                           实时计算
  nonce_str		                       实时计算
  sign		                           实时计算   
============ ==================== ==========================

**4. 执行API调用（PHP）**

假设该示例的接口API地址为：https://api.ai.qq.com/path/to/api，请求方式要求为：POST。

| 此时通过PHP实现API的调用代码如下，开发者可以参考实现其他语言的版本。

::

    // doHttpPost ：执行POST请求，并取回响应结果
    // 参数说明
    //   - $url   ：接口请求地址
    //   - $params：完整接口请求参数（特别注意：不同的接口，参数对一般不一样，请以具体接口要求为准）
    // 返回数据
    //   - 返回false表示失败，否则表示API成功返回的HTTP BODY部分
    function doHttpPost($url, $params)
    {
        $curl = curl_init();

        $response = false;
        do
        {
            // 1. 设置HTTP URL (API地址)
            curl_setopt($curl, CURLOPT_URL, $url);

            // 2. 设置HTTP HEADER (表单POST)
            $head = array(
                'Content-Type: application/x-www-form-urlencoded'
            );
            curl_setopt($curl, CURLOPT_HTTPHEADER, $head);

            // 3. 设置HTTP BODY (URL键值对)
            $body = http_build_query($params);
            curl_setopt($curl, CURLOPT_POST, true);
            curl_setopt($curl, CURLOPT_POSTFIELDS, $body);

            // 4. 调用API，获取响应结果
            curl_setopt($curl, CURLOPT_HEADER, false);
            curl_setopt($curl, CURLOPT_NOBODY, false);
            curl_setopt($curl, CURLOPT_RETURNTRANSFER, true);
            curl_setopt($curl, CURLOPT_SSL_VERIFYHOST, true);
            curl_setopt($curl, CURLOPT_SSL_VERIFYPEER, false);
            $response = curl_exec($curl);
            if ($response === false)
            {
                $response = false;
                break;
            }

            $code = curl_getinfo($curl, CURLINFO_HTTP_CODE);
            if ($code != 200)
            {
                $response = false;
                break;
            }
        } while (0);

        curl_close($curl);
        return $response;
    }

    // 设置请求数据（应用密钥、接口请求参数）
    $appkey = 'a95eceb1ac8c24ee28b70f7dbba912bf';
    $params = array(
        'app_id'     => '10000',
        'time_stamp' => '1493449657',
        'nonce_str'  => '20e3408a79',
        'key1'       => '腾讯AI开放平台',
        'key2'       => '示例仅供参考',
    );
    $params['sign'] = getReqSign($params, $appkey);

    // 执行API调用
    $url = 'https://api.ai.qq.com/path/to/api';
    $response = doHttpPost($url, $params);
    echo $response;

上述echo $response的输出结果即API的响应结果（注意不同API返回数据结构是不一样的）：

:: 

    {
    "ret": 0,
    "msg": "ok",
    "data": {
    }
    }