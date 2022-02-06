## Redis使用之Lua脚本

在Lua脚本中调用Redis命令，可以使用redis.call函数调用。比如我们调用string类型的命令。
 `redis.call(‘set’,’hello’,’world’)`
 redis.call 函数的返回值就是redis命令的执行结果。redis.call函数会将这5种类型的返回值转化对应的Lua的数据类型。

- 从Lua脚本中获得返回值
   在很多情况下我们都需要脚本可以有返回值，在脚本中可以使用return 语句将值返回给redis客户端，通过return语句来执行，如果没有执行return，默认返回为nil。
- 如何在redis中执行lua脚本
   Redis提供了EVAL命令可以使开发者像调用其他Redis内置命令一样调用脚本。
   `[EVAL] [脚本内容] [key参数的数量] [key …] [arg …]`
   可以通过key和arg这两个参数向脚本中传递数据，他们的值可以在脚本中分别使用KEYS和ARGV 这两个类型的全局变量访问。比如我们通过脚本实现一个set命令，通过在redis客户端中调用，那么执行的语句是：
   lua脚本的内容为： `return redis.call(‘set’,KEYS[1],ARGV[1])`        //KEYS和ARGV必须大写
   `eval "return redis.call('set',KEYS[1],ARGV[1])" 1 hello world`
   EVAL命令是根据 key参数的数量-也就是上面例子中的1来将后面所有参数分别存入脚本中KEYS和ARGV两个表类型的全局变量。当脚本不需要任何参数时也不能省略这个参数。如果没有参数则为0
   `eval "return redis.call(‘get’,’hello’)" 0`
- EVALSHA命令
   考虑到我们通过eval执行lua脚本，脚本比较长的情况下，每次调用脚本都需要把整个脚本传给redis，比较占用带宽。为了解决这个问题，redis提供了EVALSHA命令允许开发者通过脚本内容的SHA1摘要来执行脚本。该命令的用法和EVAL一样，只不过是将脚本内容替换成脚本内容的SHA1摘要。

1. Redis在执行EVAL命令时会计算脚本的SHA1摘要并记录在脚本缓存中。
2. 执行EVALSHA命令时Redis会根据提供的摘要从脚本缓存中查找对应的脚本内容，如果找到了就执行脚本，否则返回“NOSCRIPT No matching script,Please use EVAL”。

