```note
文件类型：l 为软链接，d 为目录，- 为二进制文件

操作命令：
	1. ll -alh
		-a 所有文件展示；-l 展示每个详情；-h 展示容易理解的格式
	2. mkdir -p
		-p 递归的创建文件夹
	3. cd 打开文件夹
	4. rmdir 删除文件夹，但是文件夹里非空的话删除不了
	5. cp -rp old new
		-r 复制目录；-p 文件或或文件夹属性一并复制
	6. mv old new 剪切，可实现文件改名
	7. pwd 显示当前的绝对路径
	8. rm -rf
		-r 删除文件夹；-f 强制删除；
	
	9. touch 创建文件
	10. cat -n 打开文件
		-n 显示行号
	11. tac 倒序查看文件，cat英文反着写就是tac
	12. more 查看文件，分页查看，不能回到上一页
	13. less 分页查看文件，可以回到上一页
	14. head -nX 查看文件前X行
		-nX X为数字，不写-n的话默认前10行
	15. tail -nX 查看文件倒数X行
		-f 动态获取倒数多少行，文件改变了会动态获取最新的
		
	16. ln [原文件] [目标文件] 创建链接文件，默认硬链接，硬链接是一个节点对应两个文件
		-s  创建软链接，相当于创建快捷方式
		硬链接：可做备份操作，但不能跨区
		软链接：可跨区，权限始终是rwxrwxrwx

			文件所有者user-用户组group-其他人other
	17. chmod 修改文件权限
		-R 递归修改
		修改方式： chmod {ugoa}{+-=}{rwx} [文件/目录]
				如：chmod g+r /etc/Test/demo1   当前用户组增加读的权限
				
		一般采用数字表示权限：
			r --- 4          换种方式理解： r --- 100  二进制表示为4
			w --- 2						  r --- 010  二进制表示为2
			x --- 1					      r --- 001  二进制表示为1
			rwxrwxrwx --> 可以表示 777
		
		chmod 777 /etc/Test/demo1
		
		chmod -R 777 /etc/Test   将Test权限及文件夹内所有文件改为777
		
	18. 对目录设置权限操作
		
```

| r    | 读权限   | 查看文件 | 可以列出目录中的内容       |
| ---- | -------- | -------- | -------------------------- |
| w    | 写权限   | 修改文件 | 可以在目录中创建、删除文件 |
| x    | 执行权限 | 执行文件 | 可以进入目录               |

场景：root创建的 -rx-------  /etc/demo1 普通用户可执行删除操作。很明显在用户组与其他用户来说都没有任何权限，但是普通用户却可以删除。原因：root对 /etc 目录设置了其他用户的‘写权限’，所有普通用户拥有目录的 '写权限'，就可以对该目录内所有文件进行删除。



```
	19. chown 改变文件的所有者
	20. chgrp 改变文件的所属组
	21. umask 查看新建文件/文件夹的缺省权限，及更改缺省权限（缺省权限也就是默认权限）
				设置值进行异或处理得到默认权限
			 -S 查看默认创造的后权限
		umask:  0022 --> ----w--w-
		完全权限:         rwxrwxrwx
        进行异或处理:      rwxr-xr-x  --> 这就是创建目录的默认权限
	
	22. 新建的文件(即缺省文件)都没有执行权限即x权限，如：-rw-r--r--
	
	
	23. 文件搜索命令  find
	   find [搜索范围] [搜索条件]
	   选项： -name  根据文件名搜索
	   				find /etc -name init   搜索所有文件名为init的文件
	   				find /etc -name *init* 加通配符*后可匹配任意字符，搜索所有文件名内含有init的文件
	   				find /etc -name init*  搜索所有文件名以init开始的文件
	   				find /etc -name init???  加问号？后可匹配一个字符，这里是搜索文件名以init开始的且后面跟了3个字符的文件
	   		-iname  不区分大小写搜索文件名
	   		
	   		-size 根据文件大小搜索
	   				find /etc -size +204800  查找大于100MB的文件
	   		
	   		-amin 访问时间access	访问了文件
	   		-cmin 文件属性change	改变了文件属性
	   		-mmin 文件内容modify    修改了文件内容
	   		
	   		如： find /etc -cmin -5  查找最近5分钟内改变了文件属性的文件
	   		
	   		-a 两个条件同时满足 指and
	   		-o 两个条件满足一个 指or
	   			find /etc -name init* -a -type d  查找etc文件夹内：文件名以init开始的，且文件类型为目录； -type f 文件类型为文件  d 文件类型为目录
	   			
	   		-exec [命令] {} \;   对查找到的文件执行命令操作
	   		-ok [命令] {} \;   对查找到的文件执行命令操作，但会进行二次确认
	   		
	   		-inum 查找i节点
	   			
	   		
	   		find .  指在当前目录查找
	   		
	 24. 文件搜索  locate
	 		根据文件资料库搜索，会更快；
	 		但是新建的文件会找不到，需先手动updatedb；
	 		但是对 /tmp 下是不放入文件资料库的，locate对tmp内也是找不到的；
	 		
	 25. 搜索命令 which
	 		搜索命令所在目录及别名信息
	 			which cp   得到的是：
	 						alias cp='cp -i'
                            /bin/cp
         搜索命令 whereis   不仅可以找到命令的绝对路径，还可以找到帮助文档的路径
     
     26. 搜索文件内容 grep
     		grep -iv [指定字串] [文件]
     			-i 不区分大小写
     			-v 排除指定字串
     				如： grep multi-user /etc/inittab  在/etc/inittab文件内查找文件内容为 multi-user 的行
     					grep -v ^# /etc/inittab  排除以#开始的行，即去除注释查看
     					
     
     
     27. 帮助命令
     		man 获取帮助信息，英文原意 manual
     			man ls  查看ls命令是做什么的，且获取到所有选项是干嘛的
     			man services 也可以查看配置文件，不需要写绝对路径
     			
     	查看命令后，后缀带有1的：命令的帮助，后缀带有5的：配置文件的帮助
     	
     		whatis 只获取命令是干嘛的
     		apropos 获取配置文件信息
     		info 
     		命令 --help 
     		
     
     28. 用户管理命令
     		useradd  添加用户的命令
     		passwd [用户]  为用户添加密码
     		
     		who  查看登录用户的信息
     		 结果：[登录用户名] [登录终端{tty：本地终端,pts：远程终端}] [登录时间(IP地址)]
     		w  查看更详细的登录用户的信息，可查看：运行多久，负载情况
     		
     		
     
     29. 压缩解压命令
     		gzip [文件名]  压缩，只能压缩文件，且压缩后不保留原文件
     		gunzip [文件名] 解压； gzip -d [文件名] 也可以解压
     		
     		tar [-zcf] [压缩后文件名] [目录]  压缩后保留原文件和目录
     			-c 打包
     			-v 显示详细信息
     			-f 指定文件名
     			-z 打包同时压缩  一般顺序为 -zcf
     			-x 解压 一般用 -zxf 解压
     			
     		zip [压缩后文件名] [目录] 压缩后保留原文件和目录
     			-r 压缩目录
     		unzip [文件]  解压缩
     		
     		bzip2 压缩
     		bunzip2 解压缩
     		
     
     30. 网络命令
     		write [用户]  回车后向其他用户发消息，必须双方在线，可先 w 命令查看当前在线用户，ctrl+D结束
     		wall [消息]  发广播信息 (wall => write all)
     		ping [ip地址] 测试网络连通性  -c [次数] 指定发送次数
     		ifconfig  查看网卡信息
     			eth 当前以太网地址
     			lo  回环网卡，就是指向本地
     		mail [用户名]  查看/发送电子邮件
     		last  查看最近登录的信息
     		traceroute [地址] 显示数据包到主机间的路径
     		netstat  显示网络相关信息
     			-tlun 查看本机监听的端口
     			-an   查看本机所有的网络连接
     			-rn   查看本机路由表
     		setup  配置网络
     		
     		mount [-t 文件系统]  挂载
     		umount [文件挂载点]  卸载	
     		
     
     
     31. 关机重启命令
     		shutdown [选项] [时间]   关机
     			-c 取消前一个关机命令
     			-h 关机
     			-r 重启
     		其余关机命令：
     			halt
     			poweroff
     			init 0
     		其他重启命令：
     			reboot
     			init 6
```
