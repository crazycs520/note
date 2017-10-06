# [npm](https://www.npmjs.com/) 

## command line

install latest npm

```shell
npm install npm@latest -g  
```

install npm package locally

```shell
npm install <package_name>
```

create **package.json** in the directory

```shell
npm init
```

ask no question , gennerate a default package.json using information extracted from the current directory.

```shell
npm init --yes
```

To add an entry to your `package.json`'s `dependencies`:

```shell
npm install <package_name> --save
```

o add an entry to your `package.json`'s `devDependencies`:

```shell
npm install <package_name> --save-dev
```

update local packages

```shell
npm update
```

Uninstall local packages , `--save` remove it form the dependencies in **package.json**

```shell
npm uninstall <package_name> [--save]
```

Install npm package globally

```shell
npm install -g <package_name>
```

update global packages

```shell
npm update -g <package_name>
npm update -g	#update all global packages
```

uninstall global packages

```shell
npm uninstall -g <package_name>
```

## cnpm

```shell
npm --registry=https://registry.npm.taobao.org install cnpm -g
```

## nrm

[nrm](https://github.com/Pana/nrm) 是一个管理 npm 源的工具.

```shell
npm i nrm -g
```

查看当前 nrm 内置的几个 npm 源的地址：

```shell
nrm ls
```

切换到 cnpm：

```shell
nrm use cnpm
```



# tool

## nodemon

同 supervisor

```Shell
npm install -g nodemon
```



## supervisor

```Shell
npm install -g supervisor
```

`supervisor` 命令启动 js 文件，会监视代码改动，然后自动重新运行。

# [express](http://expressjs.com/)

install express in a project directory and save it in the dependencies list.

```Shell
npm install express --save
```

Use the application generator tool, **`express-generator`**, to quickly create an application skeleton.

```shell
npm install express-generator -g
```

```shell
express -h #help
```

