title: Travis CI 持续部署Hexo博客到GitHub Page和VPS服务器
date: 2018-08-28 12:00:00
tags: [Travis CI, Hexo]
---

### 写在前面 
>N久之前在GitHub Page上部署了Hexo，本地生成文章的时候需要`hexo g&&hexo d`等步骤才能上传到GitHub，最近又把博客折腾到了VPS上，手动deploy到VPS的同时又不能更新GitHub上，因此搜索了一下解决方案，发现可以实现git flow push到source后，使用Travis CI 持续集成自动deploy到GitHub Page，同时推送到VPS服务器。

### 构建流程

#### 基本准备：

* GitHub Source 即博客源码仓库，以下称为Source
* GitHub Page 静态页仓库
* 配置好Nginx，git等环境的VPS
      
#### 构建流程如下：

1. 新增或修改了文章，push到GitHub的source
2. Travis CI 发现source改变后自动执行配置好的脚本，生成静态页面
3. Travis 推送静态页面文件到VPS服务器
4. Travis push静态页面到GitHub

### GitHub 和 Travis 设置

因为之前已经建好Blog的仓库和GitHub Page的仓库了，这些网上都很多教程，并不复杂，就直接记录如何让Source的仓库和Travis关联的步骤：

1. 进入GitHub的Setting页面
    
    ![](/images/15354229059347.jpg)
    
2. 选择Developer settings
    ![](/images/15354230831184.jpg)

3. 选择Personal access tokens，生成Travis需要的access token
    ![](/images/15354231182717.jpg)

4. 填写一个token的描述，如hexoblog，选择repo，然后Generate token即可

    ![](/images/15354232588232.jpg)

5. 这里要复制下生成的token（只允许看见一次），在Travis那边可以使用
     ![](/images/15354239349022.jpg)

6. 打开 [Travis](https://travis-ci.org) 网站(https://travis-ci.org) ,org后缀的才是对GitHub Public仓库免费的，com的针对是付费的私有仓库的。使用GitHub账号登录。
    
    ![](/images/15354244967581.jpg)

7. 网上很多教程说需要打开`Build only if .travis.yml is present`选项（.travis.yml存在才生成），但最新的设置好像没有这个选项了，这里也不会有影响，然后需要Add一个Token, 新增一个HEXO_TOKEN，ValuE那里填写刚刚GitHub复制的access token即可。
iv和key是后面VPS那边自动生成的，这里可以先忽略。配置完以后就需要到博客仓库`.travis.yml`文件了。
![](/images/15354245505828.jpg)


### 本地配置

1. 在本地Blog根目录下新建.travis.yml文件，配置如下：
    其中`github.com/redanula/redanula.github.com.git`为GitHub Page的仓库，`HEXO_TOKEN`为Travis刚刚Travis配置的token的标识。
    
        language: node_js
        node_js: stable
        cache:
          apt: true
          directories:
          - node_modules
        before_install:
        - export TZ='Asia/Shanghai'
        install:
        - npm install
        script:
        - hexo clean
        - hexo g && gulp
        after_script:
        - git clone https://${GH_REF} .deploy_git
        - cd .deploy_git
        - git checkout master
        - cd ../
        - mv .deploy_git/.git/ ./public/
        - cd ./public
        - git config user.name "redanula"
        - git config user.email "yanglangjing@hotmail.com"
        - git add .
        - git commit -m "Travis CI Auto Builder at `date +"%Y-%m-%d %H:%M"`"
        - git push --force --quiet "https://${HEXO_TOKEN}@${GH_REF}" master:master
        branches:
          only:
          - master
        env:
          global:
          - GH_REF: github.com/redanula/redanula.github.com.git
        notifications:
          email:
          - yanglangjing@hotmail.com
          on_success: change
          on_failure: always
    
2. 提交`.travis.yml`文件到GitHub，这时候GitHub Page就会自动生成了。可以打开[Travis](https://travis-ci.org)查看对应的Current构建过程。首次提交的之后，发现生成的 [redanula.github.io](https://redanula.github.io) 是空白的，找了下原因是因为使用的next主题没有在Source仓库的hexo theme的目录，add & push对应的主题到Source仓库之后就成功了。接下来就是服务器的工作了。
    ![](/images/15354264456856.jpg)


### VPS服务器配置

1. ssh登录VPS服务器，安装Travis服务
先安装ruby 如果提示如下：
current directory: /var/lib/gems/2.3.0/gems/ffi-1.9.25/ext/ffi_c
/usr/bin/ruby2.3 -r ./siteconf20180821-29696-1onq3fa.rb extconf.rb
mkmf.rb can't find header files for ruby at /usr/lib/ruby/include/ruby.h
需要安装对应的ruby包:

        # 如果是在centos等系统下面，执行命令：
        yum install ruby-devel 

        # 如果是在Ubuntu等系统下面，执行命令:
        apt-get install ruby-dev  

        # 安装travis命令行工具，gem指令需要先安装ruby
        gem install travis
        
2. 新增一个Git账号

        adduser git
        
3. 赋予git用户sudo权限
        
        chmod 740 /etc/sudoers
        vim /etc/sudoers
        
        # User privilege specification 在root行后面增加
        git    ALL=(ALL:ALL) ALL
        
        # 保存退出后，修改回文件权限
        chmod 440 /etc/sudoers
        
4. 配置SSH
        
        # 切换到git用户下
        su git
        
        # 生成ssh密钥对 注意密码要为空，Travis自动过程中才不会被输入密码步骤卡住，生成后目录 /home/git/.ssh/id_rsa
        ssh-keygen -t rsa
        
        # 设置.ssh目录为700
        chmod 700 ~/.ssh/
        
        # 设置.ssh目录下的文件为600
        chmod 600 ~/.ssh/*
        
        # 切换到.ssh/目录
        cd .ssh/
        
        # 将公钥内容添加到authorized_keys
        cat id_rsa.pub >> authorized_keys
        
5. Travis配置

        # 在home目录下拉取Source仓库
        cd /home
        git clone 你的仓库.git 

        # cd到仓库根目录
        # 登录github帐号
        travis login --auto
        
        # 生成加密公钥文件id_rsa.enc并自动增加解密行到.travis.yml文件；-r 指向GitHub的Source仓库
        travis encrypt-file ~/.ssh/id_rsa --add -r redanula/blog-hexo-source
        
    修改.travis.yml，在before_install钩子后面添加权限处理:
    
        before_install:
    
        - chmod 600 ~/.ssh/id_rsa
        - echo -e "Host 【配置名】\n\tStrictHostKeyChecking no\n" >> ~/.ssh/config
        
    修改.travis.yml，在after_success钩子后面添加推送行为：
        
        #/var/www/blog为博客目录
        after_script:   
         
        - rsync -rv --delete -e 'ssh -o stricthostkeychecking=no -p 对应ssh端口' public/ git@对应的IP或域名:/var/www/blog
        
    这里注意`~/.ssh/id_rsa`路径需要去掉travis命令自动添加的转义 \\ 这里的key和iv文件就是上文图中Travis后台自动出现的两个值
    
        openssl aes-256-cbc -K $encrypted_598a070685f0_key -iv $encrypted_598a070685f0_iv -in id_rsa.enc -out ~/.ssh/id_rsa -d
        
    推送.travis.yml文件到Source（也可以vps先推送上去本地拉下来做上述几步的修改）
        
        git add ./
        git commit -m "Travis CI"
        git push
 
    接下来就可以随意只管提交文章，Travis会自动持续集成到GitHub Page 和 服务器了：）。
    
        Done. Your build exited with 0.
        
### 参考文章

[Hexo - 使用 Travis CI 自動佈署 Blog](https://skychang.github.io/2017/01/01/Hexo-Use_Travis_CI_Auto_Deploy_Blog/)
[使用 Travis 自动部署 Hexo 到 Github 与 自己的服务器](https://segmentfault.com/a/1190000009054888)
[用TravisCI持续集成自动部署Hexo博客的个人实践](https://blog.csdn.net/qq_23079443/article/details/79015225)
[使用travis-ci自动部署hexo博客](https://www.noonme.com/post/2016/03/travisci-hexo-deploy/)
[Deploying Pharo builds from Travis over ssh](https://www.peteruhnak.com/blog/2016/06/06/deploying-pharo-builds-from-travis-over-ssh/)
[Travis-CI自动化测试并部署至自己的CentOS服务器](https://juejin.im/post/5a9e1a5751882555712bd8e1)
[Hexo搭建个人博客并使用Git部署到VPS](https://www.jianshu.com/p/b926ecf1c6f6)

