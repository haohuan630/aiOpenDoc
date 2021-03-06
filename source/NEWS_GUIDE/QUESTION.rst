**基础技术**
^^^^^^^^^^

**Q：什么是字典升序排序？**

A：字典升序排序是一种关联数组排序方式，开发者可参考PHP中的ksort内置函数实现。

例如：以PHP为例，假设关联数组如下。

::

  $list = array(
    'aaa' => 'aaa',
    'abc' => 'abc',
    'abb' => 'abb',
    '1aa' => '1aa',
    'abd' => 'abd'
  );

那么字典升序排序结果如下（使用ksort实现）。

::

  $list = array(
    '1aa' => '1aa',
    'aaa' => 'aaa',
    'abb' => 'abb',
    'abc' => 'abc',
    'abd' => 'abd'
  );

**Q：什么是base64编码？平台什么限制？**

A：base64编码是一种基于64个可打印字符对二进制数据进行编码存储的方式，方便在HTTP请求/响应正文中以明文字符串形式传输图片、语音等类型数据（二进制数据）。

base64编码存在多个变种实现，请开发者注意以下细节。

  -编码结果只会包含大小写字母、数字、+、/共64种可打印字符，不会包含回车、换行等特殊控制字符
  -对于图片base64编码，编码结果不包含图片头data:image/jpg;base64,

**Q：base64编码有哪些参考实现？怎么判断我的base64编码是正确的？**

A：标准base64可参考wiki文档实现。对于使用PHP语言的开发者，可以直接使用base64_encode/base64_decode内置函数实现。

判断base64是否符合要求，可以参考下述2种方式进行。

  -对比PHP的base64_encode内置函数的输出结果，如果一致，说明是正确的。

::
<?php
  // 输出/path/to/data文件的base64编码结果
  $data = file_get_contents('/path/to/data');
  echo base64_encode($data);
  ?>

  -对比Linux的base64内置工具的输出结果，如果一致，说明是正确的。

::

  $ ## 输出/path/to/data的base64编码结果
  $ base64 -w0 /path/to/data

**Q：什么是URL编码？平台什么限制？**

A：URL编码是一种基于百分号编码对HTTP非保留字符数据进行编码的方式，保证HTTP请求/响应报文的合法性（能够正常解析）。

URL编码存在多个变种实现，请开发者注意以下细节。

  -保留编码的字符： -、.、_、数字、大小写字母（这些字符在URL编码结果后不会变化）

  -特殊编码的字符：空格（这个字符在URL编码后变成+符号）

  -其他字符：除了保留编码和特殊编码之外的字符（含多字节字符），一律使用%XX方式（百分号）编码，其中XX是该字符（每个字节值）的十六进制字母表示，字母一律使用大写形式。例如：GBK编码的高重共有2个字符，每个字符由2个字节组成，十六进制分别为：腾 => 0xCCDA、讯 => 0xD1B6，那么高重的URL编码是：%CC%DA%D1%B6。

  -URL编码受字符编码影响，即不同字符编码的字符串，在URL编码后得到的结果也不一样（具体编码方式请以具体接口要求为准）。例如：GBK编码的高重的URL编码结果是：%CC%DA%D1%B6，而UTF-8编码的高重的URL编码结果是：%E8%85%BE%E8%AE%AF。

  -对于使用PHP语言的开发者，可以直接使用urlencode内置函数实现。

**ORC识别**
^^^^^^^^^^^^

**Q：能否批量识别车牌号或者营业执照？**

A：目前只能单张图进行识别，单个接口只能串行不能同时并行批量识别。

**Q：为什么图片识别的结果不是很准确？**

A：请开发者优先按下述方式自检。

  -图片必须清晰，采样的图片尽量对正对齐，不要有大量的阴影和色块，及反光的情况
  -识别一系列证件时请只将证件信息进行采样，如有其它文字信息随同证件一起采样，识别会产生误差
