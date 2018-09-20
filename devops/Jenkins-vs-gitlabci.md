持续集成环境选择：Jenkins VS gitlab-ci

Jenkins

Jenkins作为老牌的持续集成框架，在这么多年的发展中，积累很多优秀的plugin工具，对进行持续集成工作带来很大的便利。

gitlab-ci

gitlab-ci作为gitlab提供的一个持续集成的套件，完美和gitlab进行集成，gitlab-ci已经集成进gitlab服务器中，在使用的时候只需要安装配置gitlab-runner即可。 
gitlab-runner基本上提供了一个可以进行编译的环境，负责从gitlab中拉取代码，根据工程中配置的gitlab-ci.yml，执行相应的命令进行编译。

jenkins VS gitlab-runner

gitlab-runner配置简单，很容易与gitlab集成。当新建一个项目的时候，不需要配置webhook回调地址，也不需要同时在jenkins新建这个项目的编译配置，只需在工程中配置gitlab-ci.yml文件，就可以让这个工程可以进行编译。
gitlab-runner没有web页面，但编译的过程直接就在gitlab中可以看到，不需要像jenkins进入web控制台查看编译过程。
gitlab-runner仅仅是提供了一个编译的环境而已，全部的编译都通过shell脚本命令进行。当然，jenkins也可以是全部的编译都通过shell脚本命令进行。
jenkins的好处就是编译服务和代码仓库分离，而且编译配置文件不需要在工程中配置，如果团队有开发、测试、配置管理员、运维、实施等完整的人员配置，那就采用jenkins，这样职责分明。不仅仅如此，jenkins依靠它丰富的插件，可以配置很多gitlab-ci不存在的功能，比如说看编译状况统计等。如果团队是互联网类型，讲究的是敏捷开发，那么开发=devOps，肯定是采用最便捷的开发方式，推荐gitlab-ci。
如果有些敏感的配置文件不方便存放在工程中（例如nexus上传jar的账户和密码或者是其他配置的账户密码）,都可以在服务器中配置即可。
gitlab-ci对于编译需要的环境，比如jdk，maven都需要自行配置。在jenkins中，对于编译需要的环境，比如jdk，maven都可以在Web控制台安装即可。当然，jenkins也是可以自行配置的（有时候通过控制台配置下载不下来）。
-
总结

在使用过两者后，个人觉得gitlab-ci更简单易用，如果有gitlab-ci达不到的要求，可以考虑使用jenkins。