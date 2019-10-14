# PHP常见危险函数

## strcmp()

适用版本：5.3前

漏洞：当传入数组或对象时，会报错，程序会return 0，即两者相等。



## get_magic_quotes_gpc()

获得PHP系统中的环境变量 magic_quotes_gpc 的值，若该值为1，则 '(单引号), ” (双引号),  \\(反斜线) 和空字符会自动转为含有反斜线的溢出字符。



## stripslashes()

stripslashes() 函数删除由 [addslashes()](https://www.w3school.com.cn/php/func_string_addslashes.asp) 函数添加的反斜杠(若是连续二个反斜杠，则去掉一个，保留一个；若只有一个反斜杠，就直接去掉)。

```
例：echo stripslashes("Who\'s Bill Gates?");
运行结果：Who's Bill Gates?

echo stripslashes("Who\\'s Bill Gates?");
运行结果：Who's Bill Gates?

echo stripslashes("Who\\\'s Bill Gates?");
运行结果：Who\'s Bill Gates?
```

