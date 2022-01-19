### maven run one test
mvn surefire:test -Dtest=Class#method 

------

### maven 打包到其他文件夹：
<plugins>
	<plugin>
		<groupId>org.apache.maven.plugins</groupId>
		<artifactId>maven-war-plugin</artifactId>
		<configuration>
			<outputDirectory>......</outputDirectory>
		</configuration>
	</plugin>
</plugins>

ag - The Silver Searcher. Like ack, but faster.

------

###  Print gc 
jstat -gcutil  12321 1000

------


### od - dump files in octal and other formats in hexdump
od -A x -t x1z -v   

------

### Applications Links in
 .local/share/applications
 /usr/share/applications

------


 ### 添加IP 别名。
 /sbin/ifconfig eth0:1 192.168.8.43 broadcast 192.168.8.255 netmask 255.255.255.0 up
 /sbin/route add -host 192.168.8.43 dev eth0:1

------

### 文件类型转换。
 unoconv -f doc  source.txt

------

### 查看程序网络链接
 lsof -i -nP

------

### Reverse DNS lookup in Linux
 dig -x 157.240.195.35
 host 157.240.195.35
 nslookup 157.240.195.35

------

### DNS :
 172.24.2.11, 172.24.2.12, 223.5.5.5, 223.6.6.6

------

 ### 计算不同编程语言源代码的行数
 cloc file.tar.gz 
 cloc dir/

------

 ### xsel：
 在Unix下与鼠标粘帖功能互动，让粘帖操作变的更加快捷。
* shell teminal 中拷贝字段到粘贴板中
 cat xx.txt |xsel -ib
``` 
 -i 表示 --input，从标准输入的内容拷贝到粘贴板中。-b 表示 --clipboard，如果你要通过鼠标粘帖必须要选择 -b。之后就可以 用鼠标把内容贴到任何地方了。
```
 如果大家想加入一段文字到粘贴板中，可以使用如下命令：
 echo "..." | xsel -ab
```
  -a 表示 --append，若要选择selection, 则需要加入 -p 或者 -s 或者 -b。
```

- zsh 中 **可使用oh-my-zsh的插件copyfile插件就可以了**