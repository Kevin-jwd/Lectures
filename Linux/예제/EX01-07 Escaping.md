___
# EX-07 Escaping

```shell
#!/bin/bash

var=hello
echo \$var
echo \\$var
echo \"$var\"
echo \'$var\'
echo \`pwd\`
echo \a\b\c\d
echo "\a\b\c\d"

echo 111\n222\t333
echo "111\n222\t333"
echo -e "111\n222\t333"
echo -e "\x31\x32\x33"
echo -e '\x31\x32\x33'

echo $'\x31\x32\x33'
```

![](../../assets/Pasted%20image%2020260918103928.png)

- `\`를 특수 기능 문자 앞에 붙이면 특수 기능이 제거됨