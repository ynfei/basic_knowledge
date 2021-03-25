# Changelog自动生成配置

[TOC]

#### 1、安装generate-changelog

两种安装方式

```
$ npm i generate-changelog -D # install it as a dev dependency 
# OR 
$ npm i generate-changelog -g # install it globally 
```

这里使用第二种全局安装方式。

#### 2、配置package.json文件

两种配置方式，可以新建文件填写 ，也可以利用npm自动生成

```
npm init
```

进入到所需要的工程目录下，运行上述命令，会自动提醒填写相关配置。

#### 3、生成CHANGELOG.md

运行以下命令就可以再当前目录下生成CHANGELOG.md文件

```
changelog
```

利用参数进行相关生成配置。

#### 4、注意

利用该工具生成changelog文件，commit必须按照一定的格式：feat,fix,perf,docs等。

#### 5、参考

[generate-changelog, 从git提交生成变更日志](https://www.kutu66.com//GitHub/article_135527)

[npm-init](https://docs.npmjs.com/cli/init)