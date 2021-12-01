---
highlight: darcula
theme: smartblue
---

# 从 vue-cli 源码中，我发现了这个27行代码的读取 package.json 的 npm 包
## 1. 前言

>大家好，我是[若川](https://lxchuan12.gitee.io)。最近组织了[源码共读活动](https://juejin.cn/pin/7005372623400435725)，感兴趣的可以加我微信 [ruochuan12](https://juejin.cn/pin/7005372623400435725) 参与，或者关注我的[公众号若川视野](https://lxchuan12.gitee.io)，回复“源码”参与。已进行三个月，大家一起交流学习，共同进步，很多人都表示收获颇丰。

想学源码，极力推荐之前我写的[《学习源码整体架构系列》](https://juejin.cn/column/6960551178908205093) 包含`jQuery`、`underscore`、`lodash`、`vuex`、`sentry`、`axios`、`redux`、`koa`、`vue-devtools`、`vuex4`、`koa-compose`、`vue 3.2 发布`、`vue-this`、`create-vue`、`玩具vite`等20余篇源码文章。

[本文仓库 read-pkg-analysis，求个star^_^](https://github.com/lxchuan12/read-pkg-analysis.git)

最近组织了[源码共读活动](https://juejin.cn/pin/7005372623400435725)，每周大家一起学习200行左右的源码。每周一期，已进行到15期。于是搜寻各种值得我们学习，且代码行数不多的源码。

阅读本文，你将学到：
```bash
1. 如何学习调试源码
2. 学会如何获取 package.json
3. 学到 import.meta 
4. 学到 引入 json 文件的提案
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

### package.json

```js
{
	"name": 
	"scripts": {
		"test": "xo && ava && tsd"
	}
}
```

[xo](https://github.com/xojs/xo)

>JavaScript/TypeScript linter (ESLint wrapper) with great defaults

>JavaScript/TypeScript linter（ESLint 包装器）具有很好的默认值

[tsd](https://github.com/SamVerschueren/tsd)
Check TypeScript type definitions
检查 TypeScript 类型定义

[nodejs 测试工具 ava](https://github.com/avajs/ava)

### 调试
```bash
# 推荐克隆我的项目，保证与文章同步
git clone https://github.com/lxchuan12/read-pkg-analysis.git
# npm i -g yarn
cd read-pkg && yarn
# VSCode 直接打开当前项目
# code .

# 或者克隆官方项目
git clone https://github.com/sindresorhus/read-pkg.git
# npm i -g yarn
cd read-pkg && yarn
# VSCode 直接打开当前项目
# code .
```

用最新的`VSCode` 打开项目，找到 `package.json` 的 `scripts` 属性中的 `test` 命令。鼠标停留在`test`命令上，会出现 `运行命令` 和 `调试命令` 的选项，选择 `调试命令` 即可。

调试如图所示：

更多调试细节可以看我的这篇文章：[新手向：前端程序员必学基本技能——调试JS代码](https://juejin.cn/post/7030584939020042254)

我们跟着调试来看测试用例。
## 测试用例

```js
// read-pkg/test/test.js
import {fileURLToPath} from 'url';
import path from 'path';
import test from 'ava';
import {readPackage, readPackageSync} from '../index.js';

const dirname = path.dirname(fileURLToPath(import.meta.url));
process.chdir(dirname);
const rootCwd = path.join(dirname, '..');

test('async', async t => {
	const package_ = await readPackage();
	t.is(package_.name, 'unicorn');
	t.truthy(package_._id);
});

test('async - cwd option', async t => {
	const package_ = await readPackage({cwd: rootCwd});
	t.is(package_.name, 'read-pkg');
});

test('sync', t => {
	const package_ = readPackageSync();
	t.is(package_.name, 'unicorn');
	t.truthy(package_._id);
});

test('sync - cwd option', t => {
	const package_ = readPackageSync({cwd: rootCwd});
	t.is(package_.name, 'read-pkg');
});
```

### url

url 模块提供用于网址处理和解析的实用工具。

[中文文档](http://nodejs.cn/api/url.html#urlfileurltopathurl)

**url.fileURLToPath(url)**

url <URL> | <string> 要转换为路径的文件网址字符串或网址对象。
返回: <string> 完全解析的特定于平台的 Node.js 文件路径。
此函数可确保正确解码百分比编码字符，并确保跨平台有效的绝对路径字符串。
### import.meta.url

[import.meta.url](https://es6.ruanyifeng.com/?search=import.meta.url&x=10&y=9#docs/proposals#import-meta)

>（1）import.meta.url

>import.meta.url返回当前模块的 URL 路径。举例来说，当前模块主文件的路径是https://foo.com/main.js，import.meta.url就返回这个路径。如果模块里面还有一个数据文件data.txt，那么就可以用下面的代码，获取这个数据文件的路径。

>new URL('data.txt', import.meta.url)
>注意，Node.js 环境中，import.meta.url返回的总是本地路径，即是file:URL协议的字符串，比如file:///home/user/foo.js。

### process.chdir

process.chdir() 方法更改 Node.js 进程的当前工作目录，如果失败则抛出异常（例如，如果指定的 directory 不存在）。

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

### process 进程模块

[process 中文文档](http://nodejs.cn/api/process.html#process)

process 对象提供有关当前 Node.js 进程的信息并对其进行控制。 虽然它作为全局可用，但是建议通过 require 或 import 显式地访问它：

```js
import process from 'node:process';
```

[Node 文档](https://nodejs.org/dist/latest-v16.x/docs/api/modules.html)

也就是说引用 `node` 原生库可以加 `node:` 前缀，比如 `import util from 'node:util'`
### path 路径模块

[path 中文文档](http://nodejs.cn/api/path.html)

path 模块提供了用于处理文件和目录的路径的实用工具。 

### fs 文件模块

[fs 中文文档](http://nodejs.cn/api/fs.html)
### parseJson 解析 JSON

```js
const fallback = require('json-parse-even-better-errors');
const parseJson = (string, reviver, filename) => {
	if (typeof reviver === 'string') {
		filename = reviver;
		reviver = null;
	}

	try {
		try {
			return JSON.parse(string, reviver);
		} catch (error) {
			fallback(string, reviver);
			throw error;
		}
	} catch (error) {
		// 省略
	}
}
```
### normalizePackageData 规范化包元数据

[npm 官方库](https://github.com/npm/normalize-package-data)

>normalizes package metadata, typically found in package.json file.

规范化包元数据

```js
module.exports = normalize
function normalize (data, warn, strict) {
	// 省略若干代码
	data._id = data.name + '@' + data.version
}
```


## 总结

建议读者克隆 [我的仓库](https://github.com/lxchuan12/read-pkg-analysis.git) 动手实践调试源码学习。

最后可以持续关注我@若川。欢迎加我微信 [ruochuan12](https://juejin.cn/pin/7005372623400435725) 交流，参与 [源码共读](https://juejin.cn/pin/7005372623400435725) 活动，每周大家一起学习200行左右的源码，共同进步。


https://stackoverflow.com/questions/34944099/how-to-import-a-json-file-in-ecmascript-6