encrypt
=======
  前几天突发奇想要往数据库里保存一些机密的东西，然后就想着怎么让别人即使进入到了数据库也看不懂存储的是什么，那么只有加密了；可是我们自己还要看呢，那只能找一些对称加密的算法了，我们想看的时候再解密回来。  

  在网上找到了一个不错的PHP方面的对称加密算法；在PHP的语法环境里，有urlencode与urldecode,base64_encode和base64_decode自带的对称算法，不过这些自带的算法不能称之为加密算法，只能说是编码方式而已。可是我们可以利用这些来进行一些加工，来实现简单的加密和解密算法。  

  这次的加密和解密算法是使用base64的方式改编的。通常我们使用base64_encode($str)产生的字符串，不经过任何的加工的话，base64_decoe()就能转回我们之前的字符串；可是如果我们在base64_encode()之后的字符串里插入几个字符呢，那他就转不回来了，即使转过来也不是我们自己的字符串。

```php
<?php
$content = "大家好，我是中国人，你是谁";
 
/**
 * 简单对称加密算法之加密
 * @param String $string 需要加密的字串
 * @param String $skey 加密EKY
 * @author Anyon Zou <zoujingli@qq.com>
 * @date 2013-08-13 19:30
 * @update 2014-10-10 10:10
 * @return String
 */
function encode($string = '', $skey = 'wenzi') {
    $strArr = str_split(base64_encode($string));
    $strCount = count($strArr);
    foreach (str_split($skey) as $key => $value)
        $key < $strCount && $strArr[$key].=$value;
    return str_replace(array('=', '+', '/'), array('O0O0O', 'o000o', 'oo00o'), join('', $strArr));
}
 
/**
 * 简单对称加密算法之解密
 * @param String $string 需要解密的字串
 * @param String $skey 解密KEY
 * @author Anyon Zou <zoujingli@qq.com>
 * @date 2013-08-13 19:30
 * @update 2014-10-10 10:10
 * @return String
 */
function decode($string = '', $skey = 'wenzi') {
    $strArr = str_split(str_replace(array('O0O0O', 'o000o', 'oo00o'), array('=', '+', '/'), $string), 2);
    $strCount = count($strArr);
    foreach (str_split($skey) as $key => $value)
        $key <= $strCount && $strArr[$key][1] === $value && $strArr[$key] = $strArr[$key][0];
    return base64_decode(join('', $strArr));
}
 
echo '<pre>';
echo "string : " . $content . " <br />";
echo "encode : " . ($enstring = encode($content)) . '<br />';
echo "decode : " . decode($enstring);
 
exit();
```

  上面的算法里我们可以看到：我们把base64_encode()产生的字符中插入我们提前设定的密钥，然后再把里面的特殊字符进行替换，即使别人看到这样的字符串也不知道是什么。当然，这里我们再稍微的改进下，比如把密钥倒着插入到字符串里，密钥base64后再进行插入等等，插入密钥后再base64一次。

  当然解密就是加密的反方向了，思考了一会儿才知道解密的原理：之前我们在字符串里插入了一些字符，现在解密时我们就要把他摘出来，首先把加密后的字符串按每个数组里2个元素进行分组，然后判断第二个字符是不是密钥里的，如果是，那么第一个字符就是原来base64里的字符。
