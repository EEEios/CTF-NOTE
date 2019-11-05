# PHP函数参考手册

## array_walk($a,"myfunction");

对$a**数组**中的值应用myfunction函数。

$a中的键名和键值都是参数。

```php
<?php
function myfunction($value,$key)
{
echo "The key $key has the value $value<br>";
}
$a=array("a"=>"red","b"=>"green","c"=>"blue");
array_walk($a,"myfunction");
?>
```

会依次运行 myfunction("a","red"); myfunction("b","green"); myfunction("c","blue");