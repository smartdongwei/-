# commons-lang3的api学习

## 1: 基础内容

1. 相关依赖的信息为

```xml
 <dependency>
    <groupId>org.apache.commons</groupId>
    <artifactId>commons-lang3</artifactId>
    <version>3.8</version>
</dependency>
```

2. 相关结构为

   ![image-20220226010731958](E:\study\images\image-20220226010731958.png)

## 2: 相关接口

### 2.1 org.apache.commons.io.IOUtils

```java
closeQuietly：关闭一个IO流、socket、或者selector且不抛出异常，通常放在finally块
toString：转换IO流、 Uri、 byte[]为String
copy：IO流数据复制，从输入流写到输出流中，最大支持2GB
toByteArray：从输入流、URI获取byte[]
write：把字节. 字符等写入输出流
toInputStream：把字符转换为输入流
readLines：从输入流中读取多行数据，返回List<String>
copyLarge：同copy，支持2GB以上数据的复制
lineIterator：从输入流返回一个迭代器，根据参数要求读取的数据量，全部读取，如果数据不够，则失败复制代码 
```

### 2.2 org.apache.commons.io.FileUtils



### 2.3 org.apache.commons.lang.StringUtils

```
isBlank：字符串是否为空 (trim后判断)
isEmpty：字符串是否为空 (不trim并判断)
equals：字符串是否相等
join：合并数组为单一字符串，可传分隔符
split：分割字符串
EMPTY：返回空字符串
trimToNull：trim后为空字符串则转换为null
replace：替换字符串复制代码 
```



### 2.4 org.apache.http.util.EntityUtils

```
toString：把Entity转换为字符串
consume：确保Entity中的内容全部被消费。可以看到源码里又一次消费了Entity的内容，假如用户没有消费，那调用Entity时候将会把它消费掉
toByteArray：把Entity转换为字节流
consumeQuietly：和consume一样，但不抛异常
getContentCharset：获取内容的编码复制代码 
```



### 2.5 org.apache.commons.lang3.StringUtils

```
isBlank：字符串是否为空 (trim后判断)
isEmpty：字符串是否为空 (不trim并判断)
equals：字符串是否相等
join：合并数组为单一字符串，可传分隔符
split：分割字符串
EMPTY：返回空字符串
replace：替换字符串
capitalize：首字符大写复制代码 
```



### 2.6 org.apache.commons.io.FilenameUtils

```
getExtension：返回文件后缀名
getBaseName：返回文件名，不包含后缀名
getName：返回文件全名
concat：按命令行风格组合文件路径(详见方法注释)
removeExtension：删除后缀名
normalize：使路径正常化
wildcardMatch：匹配通配符
seperatorToUnix：路径分隔符改成unix系统格式的，即/
getFullPath：获取文件路径，不包括文件名
isExtension：检查文件后缀名是不是传入参数(List<String>)中的一个复制代码 
```



### 2.7 org.springframework.util.StringUtils

```
hasText：检查字符串中是否包含文本
hasLength：检测字符串是否长度大于0
isEmpty：检测字符串是否为空（若传入为对象，则判断对象是否为null）
commaDelimitedStringToArray：逗号分隔的String转换为数组
collectionToDelimitedString：把集合转为CSV格式字符串
replace 替换字符串

delimitedListToStringArray：相当于split
uncapitalize：首字母小写
collectionToDelimitedCommaString：把集合转为CSV格式字符串
tokenizeToStringArray：和split基本一样，但能自动去掉空白的单词复制代码 
```



### 2.8 org.apache.commons.lang.ArrayUtils

```
contains：是否包含某字符串
addAll：添加整个数组
clone：克隆一个数组
isEmpty：是否空数组
add：向数组添加元素
subarray：截取数组
indexOf：查找某个元素的下标
isEquals：比较数组是否相等
toObject：基础类型数据数组转换为对应的Object数组复制代码 
```



### 2.9 org.apache.commons.lang.StringEscapeUtils

```
参考十五：org.apache.commons.lang3.StringEscapeUtils复制代码
```



### 2.10 org.apache.http.client.utils.URLEncodedUtils

```
format：格式化参数，返回一个HTTP POST或者HTTP PUT可用application/x-www-form-urlencoded字符串
parse：把String或者URI等转换为List<NameValuePair>复制代码 
```



### 2.11 org.apache.commons.codec.digest.DigestUtils

```
md5Hex：MD5加密，返回32位字符串
sha1Hex：SHA-1加密
sha256Hex：SHA-256加密
sha512Hex：SHA-512加密
md5：MD5加密，返回16位字符串复制代码 
```



### 2.12 org.apache.commons.collections.CollectionUtils

```
isEmpty：是否为空
select：根据条件筛选集合元素
transform：根据指定方法处理集合元素，类似List的map()
filter：过滤元素，雷瑟List的filter()
find：基本和select一样
collect：和transform 差不多一样，但是返回新数组
forAllDo：调用每个元素的指定方法
isEqualCollection：判断两个集合是否一致复制代码 
```



### 2.13 org.apache.commons.lang3.ArrayUtils

```
contains：是否包含某个字符串
addAll：添加整个数组
clone：克隆一个数组
isEmpty：是否空数组
add：向数组添加元素
subarray：截取数组
indexOf：查找某个元素的下标
isEquals：比较数组是否相等
toObject：基础类型数据数组转换为对应的Object数组复制代码 
```



### 2.14 org.apache.commons.beanutils.PropertyUtils

```java
getProperty：获取对象属性值
setProperty：设置对象属性值
getPropertyDiscriptor：获取属性描述器
isReadable：检查属性是否可访问
copyProperties：复制属性值，从一个对象到另一个对象
getPropertyDiscriptors：获取所有属性描述器
isWriteable：检查属性是否可写
getPropertyType：获取对象属性类型复制代码 
```



### 2.15 org.apache.commons.lang3.StringEscapeUtils

```java
unescapeHtml4：转义html
escapeHtml4：反转义html
escapeXml：转义xml
unescapeXml：反转义xml
escapeJava：转义unicode编码
escapeEcmaScript：转义EcmaScript字符
unescapeJava：反转义unicode编码
escapeJson：转义json字符
escapeXml10：转义Xml10复制代码 
这个现在已经废弃了，建议使用commons-text包里面的方法。
```



### 2.16 org.apache.commons.beanutils.BeanUtils

```java
copyPeoperties：复制属性值，从一个对象到另一个对象
getProperty：获取对象属性值
setProperty：设置对象属性值
populate：根据Map给属性复制
copyPeoperty：复制单个值，从一个对象到另一个对象
cloneBean：克隆bean实例复制代码

```

