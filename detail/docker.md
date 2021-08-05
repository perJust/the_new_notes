## 基础概念

1. docker里的镜像与容器：镜像就相当于软件的源（独立软件包），容器就是镜像的运行环境，一个镜像可以启动多个容器；



## 基本命令

```shell
docker images #查看镜像  -a查看所有  -q只查看镜像id
docker search 镜像名  #搜索docker hub上的镜像
docker rmi 镜像id  # 删除镜像  -f 强制删除
				  # -f $(docker images -aq) 强制删除所有镜像，$()是个筛选条件
				  
docker pull 镜像名[:tag] # 不加[:tag]就是下载最新的镜像，tag为版本号
docker run [-选项] 镜像名 [进入路径]  # 运行镜像，生成容器
	--name='Name' # 给容器命名
	-it # 以交互的方式进入容器，进行查看并操作
	-d 	# 后台运行容器
	-p  # 指定端口  -p 主机端口:容器端口   或  -p 容器端口
	-P  # 随机指定端口
exit   # 退出当前容器，直接容器停止并退出
快捷键[ctrl + p + q] # 退出当前容器，容器不停止并退出

docker ps [选项]	# 列出正在运行的容器
			-a	 # 列出所有运行过的容器，包含历史运行过的容器
			-n=? # 列出最近创建的容器
			-q	 # 只显示容器编号
docker rm [容器id] # 删除指定的容器
		  -f 	  # 强制删除指定容器
		  		  # -f $(docker images -aq) 强制删除所有的容器
docker cp [来源] [目标] # 主机与容器之间的拷贝，容器内的文件位置需加上容器[id:]，如：docker cp /tmp/demo qwewerw:/tmp/demo
```



