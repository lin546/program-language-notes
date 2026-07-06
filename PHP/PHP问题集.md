# PHP 问题集

### 1、PHP 单引号和双引号
在PHP中，单引号（''）和双引号（""）都可以用来表示字符串。它们之间有一些区别：

+ 解析变量：双引号可以解析变量并将其替换为其对应的值，而单引号不会解析变量，会将其作为普通的字符串。

```plain
$name = "John";
echo "My name is $name"; // 输出：My name is John
echo 'My name is $name'; // 输出：My name is $name
```

+ 转义字符：双引号允许使用转义字符（例如：\n、\t等）来表示特殊字符，而单引号不会解析转义字符，将其作为普通字符串。

```plain
$name = "John";
echo "My name is $name"; // 输出：My name is John
echo 'My name is $name'; // 输出：My name is $name
```

+ 性能：由于双引号需要解析变量和转义字符，因此在性能上稍微比单引号慢一些。对于没有包含变量或转义字符的简单字符串，使用单引号可能更高效。

综上所述，选择使用单引号还是双引号取决于您的需求。如果需要解析变量或转义字符，建议使用双引号；如果字符串是静态的且不需要解析变量或转义字符，使用单引号可以提高性能。



> 更新: 2024-10-03 13:46:59  
> 原文: <https://www.yuque.com/thinkspace/du51gc/xko1w9lz5k9rew66>