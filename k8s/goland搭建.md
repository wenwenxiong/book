参考网址： https://www.newasp.net/soft/423786.html

1、下载并安装```JetBrains GoLand 2018 Linux```

2、 复制 ```JetbrainsCrack.jar``` 到 安装目录 ```/bin```文件夹中

3、 使用记事本编辑文件```GoLand.vmoptions```或```GoLand64.vmoptions```，在安装文件```/bin```:中

4、将下面的代码复制到文件中 (新的一行中添加):``` -javaagent: {InstallDir} /bin/JetbrainsCrack.jar```

例如：```-javaagent:C:\Program Files\JetBrains\GoLand 2018.1.1\bin\JetbrainsCrack.jar```

5、运行```GoLand 2018 Linux```，注册时输入任意数字激活即可。

更改goland的配置,一般需要改一下字体和主题,

字体的更改方法: File -> Settings -> Editor -> Font -> Size, 推荐选18或者20

主题的更改方法: File -> Settings -> Editor -> Color Scheme -> Scheme, 推荐选Colorful Darcula

如果没有喜欢的主题,也可以到plugin里安装,搜索material,install后,重启goland再次选择主题.