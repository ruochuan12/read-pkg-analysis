---
highlight: darcula
theme: smartblue
---

# 从 vue-cli 源码中，我发现了这个27行代码的读取package.json的 npm 包
## 1. 前言

>大家好，我是[若川](https://lxchuan12.gitee.io)。最近组织了[源码共读活动](https://juejin.cn/pin/7005372623400435725)，感兴趣的可以加我微信 [ruochuan12](https://juejin.cn/pin/7005372623400435725) 参与，或者关注我的[公众号若川视野](https://lxchuan12.gitee.io)，回复“源码”参与。已进行三个月，大家一起交流学习，共同进步，很多人都表示收获颇丰。

想学源码，极力推荐之前我写的[《学习源码整体架构系列》](https://juejin.cn/column/6960551178908205093) 包含`jQuery`、`underscore`、`lodash`、`vuex`、`sentry`、`axios`、`redux`、`koa`、`vue-devtools`、`vuex4`、`koa-compose`、`vue 3.2 发布`、`vue-this`、`create-vue`、`玩具vite`等10余篇源码文章。

[本文仓库 only-allow-analysis，求个star^_^](https://github.com/lxchuan12/only-allow-analysis.git)

最近组织了[源码共读活动](https://juejin.cn/pin/7005372623400435725)，每周大家一起学习200行左右的源码。每周一期，已进行到14期。于是搜寻各种值得我们学习，且代码行数不多的源码。

阅读本文，你将学到：
```bash
1. 如何学习调试源码
2. 学会 npm 钩子
3. 学会 "preinstall": "npx only-allow pnpm" 一行代码统一规范包管理器
4. 学到 only-allow 原理
5. 等等
```

## 2. 场景

优雅的获取 package.json

[read-pkg](https://npm.im/read-pkg)

https://github.com/vuejs/vue-cli/blob/dev/packages/%40vue/cli-shared-utils/lib/pkg.js

```js
const fs = require('fs')
const path = require('path')
const readPkg = require('read-pkg')

exports.resolvePkg = function (context) {
  if (fs.existsSync(path.join(context, 'package.json'))) {
    return readPkg.sync({ cwd: context })
  }
  return {}
}
```

[](https://github.com/vuejs/vue-cli/commit/eaa2b7341f174260f6ebc2345bd8e838f85a2ea3#diff-eff8c1ce409e9f1aa37372ff81a6563e31d1270822f5bfdba17426c4b1def4de)


## 环境准备


## 源码

```js
import process from 'node:process';
import fs, {promises as fsPromises} from 'node:fs';
import path from 'node:path';
import parseJson from 'parse-json';
import normalizePackageData from 'normalize-package-data';

export async function readPackage({cwd = process.cwd(), normalize = true} = {}) {
	const filePath = path.resolve(cwd, 'package.json');
	const json = parseJson(await fsPromises.readFile(filePath, 'utf8'));

	if (normalize) {
		normalizePackageData(json);
	}

	return json;
}

```


```js
export function readPackageSync({cwd = process.cwd(), normalize = true} = {}) {
	const filePath = path.resolve(cwd, 'package.json');
	const json = parseJson(fs.readFileSync(filePath, 'utf8'));

	if (normalize) {
		normalizePackageData(json);
	}

	return json;
}
```

### process
### path

### fs
### parseJson

### normalizePackageData

标准化



## 总结

建议读者克隆[我的仓库](https://github.com/lxchuan12/only-allow-analysis.git)动手实践调试源码学习。

最后可以持续关注我@若川。欢迎加我微信 [ruochuan12](https://juejin.cn/pin/7005372623400435725) 交流，参与 [源码共读](https://juejin.cn/pin/7005372623400435725) 活动，每周大家一起学习200行左右的源码，共同进步。