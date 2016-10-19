---
title: 使用hexo搭建git博客
date: 2016-10-19 10:46:20
categories: tools               
tags: hexo
---

##### 1. 环境
安装好node和git,注册好github账号. 注意:用户名一定不能有大写.
最新的git: http://msysgit.github.io/

##### 2.安装hexo:
执行: npm install -g hexo

##### 3.创建hexo文件夹:
创建hexo文件夹,如 E:\myblogs\hexoRita
cmd到对应目录下执行: hexo init 

##### 4.安装依赖:
npm install

##### 5.本地测试:
hexo g
hexo s
在打开浏览器 localhost:4000，可以看到一个样例helloworld.

##### 6.在github上创建博客仓库
ritali.github.io(与用户名一致，不能有大写)
选择主题，发布

##### 7. 配置_config.yml
先执行 npm install hexo-deployer-git --save
deploy:
- type: git
  repo: git@github.com:Ritali/ritali.github.io
  branch: master
- type: git
  repo: git@git.coding.net:RitaLi/ritali.git
  branch: coding-pages

再次执行hexo g, hexo s

##### 8. 访问
coding 访问链接： http://ritali.coding.me/ritali/
git 访问链接： ritali.github.io

##### 附：
1、在github上创建项目
2、使用git clone https://github.com/xxxxxxx/xxxxx.git克隆到本地
3、编辑项目
4、git add . （将改动添加到暂存区）
5、git commit -m "提交说明"
	第一次提交建立关联：
	git remote add origin https://github.com/findingsea/myRepoForBlog.git
6、git push origin master 将本地更改推送到远程master分支。
	在执行git push origin master时，报错：
　　error:failed to push som refs to.......
	git pull origin master


hexo new page “tags”#新建目录
hexo new “blog”#新建文章

问题：
1. npm的node_modules路径太长，无法删除更无法上传到github
robocopy empty_dir will_delete_dir /purge