title: 上传jar包到nexus私服
catalog: true
author: wangmj
tags: []
categories: []
date: 2019-03-18 17:03:00
subtitle:
header-img:
---
#### 上传jar包
主要有两种方式，一个是网页上传，一个是命令行上传
#### 网页上传
![在这里插入图片描述](https://img-blog.csdnimg.cn/20190318161357654.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dtajc2NQ==,size_16,color_FFFFFF,t_70)
上传之后，将私服jar包同步到本地。在idea执行-U idea:idea。
#### 通过命令上传
首先修改maven的settings文件，添加用私服户名及密码，

```
	<server>
		<id>releases</id>
		<username>admin</username>
		<password>admin</password>
	</server>

	<server>
		<id>snapshots</id>
		<username>admin</username>
		<password>admin</password>
	</server>
  </servers>
```
之后在jar包所在目录执行以下命令
```
mvn deploy:deploy-file -DgroupId=com.qianhai -DartifactId=chainsdk -Dversion=1.3.4 -Dpackaging=jar -Dfile=chainsdk1.3.4.jar -Durl=http://10.144.137.112:8081/nexus/content/repositories/releases -DrepositoryId=releases
```
执行成功后同步到本地即可
