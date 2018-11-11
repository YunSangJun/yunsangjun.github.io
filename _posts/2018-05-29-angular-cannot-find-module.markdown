---
layout: post
title:  "Cannot find module 'webpack/lib/node/NodeTemplatePlugin'"
author: 윤상준
date: 2018-05-29
categories: angular
tags:
- angular
- npm
---

Angular build 시 아래와 같은 에러가 나는 경우가 있습니다.
npm5에 도입된 `package-lock.json`을 삭제하지 않고 `npm install`을 한 경우 발생합니다.

```
yarn run xxx
yarn run v1.7.0
$ ng build --prod --aot --progress=false --output-path rootfs/dist
Cannot find module 'webpack/lib/node/NodeTemplatePlugin'
Error: Cannot find module 'webpack/lib/node/NodeTemplatePlugin'
    at Function.Module._resolveFilename (module.js:547:15)
    at Function.Module._load (module.js:474:25)
    at Module.require (module.js:596:17)
    at require (internal/module.js:11:18)
    at Object.<anonymous> (/Users/sangjunyun/git/zcp-catalog-ui/node_modules/html-webpack-plugin/lib/compiler.js:11:26)
    at Module._compile (module.js:652:30)
    at Object.Module._extensions..js (module.js:663:10)
    at Module.load (module.js:565:32)
    at tryModuleLoad (module.js:505:12)
    at Function.Module._load (module.js:497:3)
    at Module.require (module.js:596:17)
    at require (internal/module.js:11:18)
    at Object.<anonymous> (/Users/sangjunyun/git/zcp-catalog-ui/node_modules/html-webpack-plugin/index.js:7:21)
    at Module._compile (module.js:652:30)
    at Object.Module._extensions..js (module.js:663:10)
    at Module.load (module.js:565:32)
error Command failed with exit code 1.
info Visit https://yarnpkg.com/en/docs/cli/run for documentation about this command.
make: *** [compile-aot] Error 1
```

아래 명령어를 실행해 `node_modules`, `dist` 그리고 `package-lock.json`을 삭제합니다.

```
$ rm -rf node_modules dist package-lock.json
$ npm cache clean
```

이제 `npm install` 명령을 실행해 node_modules을 다시 설치합니다.
이후 Angular build를 다시하면 정상적으로 실행됩니다.
