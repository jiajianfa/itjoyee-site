# itjoyee-site
使用hexo系统和Pacman主题搭建github博客源码
## 说明
测试一下markdown怎么使用

# 加减法个人blog

本库是基于Hexo开发个人blog的源码. 你可以通过 [jiajianfa/jiajianfa.github.io](https://github.com/jiajianfa/jiajianfa.github.io) 链接查看生成的静态文件.
访问[itjoyee.com](http://itjoyee.com)可以进入我的blog,欢迎各位光临.

## Hexo起步教程

Install dependencies:

``` bash
$ git clone https://github.com/hexojs/site.git
$ cd site
$ npm install
```

Generate:

``` bash
$ gulp
$ hexo generate
```

Run server:

``` bash
$ hexo server
```

## Requirements

- [Hexo](http://hexo.io/)

	``` bash
  $ npm install hexo -g
  ```

- [Gulp](http://gulpjs.com/)

	``` bash
	$ npm install gulp -g
	```
## 安装 Pacman主题的方法

- 安装
git clone https://github.com/A-limon/pacman.git themes/pacman
Pacman需要安装Hexo 2.4.5 或以上版本 请先升级您的Hexo程序，再启用此主题。

- 启用
修改你的博客根目录下的config.yml配置文件中的theme属性，将其设置为pacman。同时请设置stylus属性中的compress值为true。

- 更新
cd themes/pacman
git pull

## 更多
更多配置查看[pacman](http://yangjian.me/workspace/introducing-pacman-theme/)官网的说明
