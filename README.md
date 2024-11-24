## 发布说明
name:  包名，后续在npm中搜索全靠它


version： 版本号，每发布一次npm包就要增加一个版本，每个版本不能重复。


description：描述


main:  本包向外暴露的文件，很重要，一定要和你打包出来的文件名一模一样，我的叫做"dist/index.js"


private: true/false  是否为私有。  一般为false否则只有自己能使用


flies: 暴露的文件夹, 有哪些文件夹提交到npm上面 格式为[ "es", "lib" ]


keywords:  npm检索的关键字


author: 作者


license: ISC


peerDependencies: 代表着当前npm包依赖下面这几种环境。

如果是第一次发布包，执行以下命令，然后输入前面注册好的NPM账号，密码和邮箱，将提示创建成功
npm adduser
如果不是第一次发布包，执行以下命令进行登录，同样输入NPM账号，密码和邮箱
npm login
注意：npm adduser成功的时候默认你已经登陆了，所以不需要再进行npm login了

接着先进入项目文件夹下，然后输入以下命令进行发布
  npm publish
当终端显示如下面的信息时，就代表版本号为1.0.0(你的package.json中的版本号)的包发布成功啦！前往NPM官网就可以查到你的包

报错
 npm ERR! code E403
  npm ERR! 403 403 Forbidden - PUT https://registry.npmjs.org/ghost-watermarkdemo - Forbidden
  npm ERR! 403 In most cases, you or one of your dependencies are requesting
  npm ERR! 403 a package version that is forbidden by your security policy, or
  npm ERR! 403 on a server you do not have access to.

以下几种原因会导致
   账号密码错误   (请检查npm官网的账号密码)
  包重名     (请检查npm官网上是否有同名项目，名字取决于 package.js 的项目名字段)
  网络原因   
  镜像源问题 
  新注册的用户邮箱未激活。  登陆你的邮箱去激活(如下)

更新已经发布的包
npm publish
但是每次更新时，必须修改版本号后才能更新，比如将1.0.0修改为1.0.1后才能更新发布。
这里的包版本管理规则都是一样的，采用的是semver（语义化版本），意思就是版本号：大改.中改.小改

从npm上面卸载自己发布的包
npm unpublish --force

 npm WARN using --force Recommended protections disabled.
-包名@0.1.0
则卸载成功，这时在npm上面就搜索不到了

## main/module/types  不同引入 会去对应的入口文件找
- main: 对应commonjs引入方式的程序入口文件 const { a } = require('a');
- module: 对应esmodule引入方式的程序入口文件 import { a } from 'a';
- types: 描述了程序中所有组件以及变量的类型定义

## exports： exports定义了自定义导出规则，可以理解为路径映射
{
   "exports": {
    ".": {
      "types": "./dist/index.d.ts",
      "import": "./dist/index.mjs",
      "require": "./dist/index.js"
    },
    "./unstyled": {
      "types": "./dist/unstyled.d.ts",
      "import": "./dist/unstyled.mjs",
      "require": "./dist/unstyled.js"
    }
  },
}
如 import { a } from 'a'; 只能从packages 的 module去找
定义了./unstyled import aUnstyled from 'a/unstyled'; 就够可以这样找到
如下配置后glob后 就可以根据文件目录去找 
"./*": [
+      "./*",
+      "./*.d.ts"
+    ]

## type和exports/main/module的关系
- 具体参考 https://juejin.cn/post/7212436135287504954?searchId=20241124132255F6621DA2FBB500958D31#heading-5
- 首先我们需要理解type字段的含义
- 当设置为“module”时，所在项目中（不包含node_modules）所有.js文件将被视为EsModule类型文件。
- 如果省略“type”字段或设置为“commonjs”，则项目中（不包含node_modules）所有.js文件都被视为CommonJS类型文件。
- type: "module" 此时.js文件将被视为esmodule，并且我们需要将commonjs文件显示声明为.cjs
改造配置如下：
{
  ...,
  "type": "module",
  "main": "./dist/index.cjs",
  "module": "./dist/index.js",
  "types": "./dist/index.d.ts",
  "exports": {
    ".": {
      "types": "./dist/index.d.ts",
      "import": "./dist/index.js",
      "require": "./dist/index.cjs"
    },
    "./unstyled": {
      "types": "./dist/unstyled.d.ts", // 可以省略，但不建议
      "import": "./dist/unstyled.js",
      "require": "./dist/unstyled.cjs"
    },
    "./*": "./*"
  },
  ...
}
type: "commonjs" 或不设置
此时.js将被视为commonjs，并且我们需要将esmodule文件显示声明为.mjs/.esm.js(实际上你声明成.xxx.js也可以，甚至.xxx也行，但不建议)
改造配置如下：
json 代码解读复制代码{
  ...,
  "main": "./dist/index.js",
  "module": "./dist/index.mjs",
  "types": "./dist/index.d.ts",
  "exports": {
    ".": {
      "types": "./dist/index.d.ts",
      "import": "./dist/index.mjs",
      "require": "./dist/index.js"
    },
    "./unstyled": {
      "types": "./dist/unstyled.d.ts", // 可以省略，但不建议
      "import": "./dist/unstyled.mjs",
      "require": "./dist/unstyled.js"
    },
    "./*": "./*"
  },
  ...
}

## tsup https://tsup.egoist.dev/#what-can-it-bundle
- 由esbuild提供支持，无需配置即可捆绑您的 TypeScript 库。
- 主要作用打出符合规范的包 具体参考文档