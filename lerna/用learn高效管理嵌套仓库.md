> lerna多包管理实践之路

### 潜移默化

> 前面一片文章介绍了[lerna的一些用法](https://algesthesiahunter.top/articles/5f47d6f6fb6bb6f5fb8fb5df)，自己呢也有个多包仓库需求，再出一篇实践文章岂不是顺手拈来的事

### 开干

现有资源是主仓库是对子包组件的一个展示页，传统嵌套包管理模式，子包大约有5个包每次需要切到相应目录分别 `构建发布`和 `push git仓库`，及其费劲和费时。我都忘了我为何当初搞成这样了，貌似是特殊的依赖问题，今天我们就用`lerna`重构下

#### 先初始化下

``` bash
# 确保安装了lerna
npm install lerna -g

#  在仓库初始化lerna配置
lerna init
```

#### 使用 workspaces模式

每个子 `package` 都有自己的 `node_modules` ，通过这样设置后，只有顶层有一个 `node_modules`，修改顶层 `package.json` and `lerna.json`

  package.json

``` json
{
  "private": true,
  "workspaces": [
    "packages/*"
  ],
}

```

 lerna.json

``` json
{
  "packages": ["packages/*"],
  "useWorkspaces": true,
  "npmClient": "yarn",
  "version": "0.0.0"
}
```

#### 生成一个npm包

``` bash
lerna create @algesthesiah/vue-lazy-view

```

##### 配置package.json 交互式基础信息

``` bash
package name: (@algesthesiah/vue-lazy-view) y
version: (0.0.1) y
description:Vue.js 2.x component level lazy loading solution
keywords:vue
homepage:https://github.com/vuejs/vue-cli/tree/dev/packages/@vue/babel-preset-app#readme
repository:https://github.com/Algesthesiahunter/base.git
```

##### 补全老包信息到上面的配置文件中

将老包里面的依赖和脚本copy进上面的配置文件中最后得到如下配置

``` json
{
  "name": "@algesthesiah/vue-lazy-view",
  "version": "0.0.12",
  "description": "Vue.js 2.x component level lazy loading solution",
  "keywords": [
    "vue"
  ],
  "author": "liulu <541877028@qq.com>",
  "homepage": "https://github.com/Algesthesiahunter/base/packages/vue-lazy-view#readme",
  "license": "MIT",
  "main": "dist/index.js",
  "directories": {
    "lib": "lib",
    "test": "__tests__"
  },
  "files": [
    "lib"
  ],
  "publishConfig": {
    "access": "public"
  },
  "devDependencies": {
    "babel-core": "^6.26.0",
    "babel-loader": "^7.1.2",
    "babel-preset-env": "^1.6.0",
    "babel-preset-stage-3": "^6.24.1",
    "cross-env": "^7.0.2",
    "css-loader": "^3.2.1",
    "extract-text-webpack-plugin": "^3.0.2",
    "mini-css-extract-plugin": "^0.8.0",
    "node-sass": "npm:dart-sass",
    "optimize-css-assets-webpack-plugin": "^3.2.0",
    "postcss-loader": "^3.0.0",
    "postcss-preset-env": "^6.7.0",
    "sass-loader": "^8.0.0",
    "style-loader": "^1.0.1",
    "vue-loader": "^15.7.2",
    "vue-style-loader": "^4.1.2",
    "vue-template-compiler": "^2.6.10",
    "webpack": "^4.41.2",
    "webpack-cli": "^3.3.10"
  },
  "repository": {
    "type": "git",
    "url": "https://github.com/Algesthesiahunter/base.git",
    "directory": "packages/vue-lazy-view"
  },
  "scripts": {
    "test": "echo \"Error: run tests from root\" && exit 1",
    "build": "cross-env sb=1 NODE_ENV=production webpack  --progress"
  },
  "gitHead": "f2c659911ece8d3a1c048c8d97ad30ad8ee69ede"
}
```

#### 优化release changelog

限制优雅的`git commit`提交，并自动生成直观的`release changelog`

``` bash
yarn add -W -D @commitlint/config-conventional @commitlint/cli lint-staged husky commitizen cz-conventional-changelog lint-staged standard-version
```

``` json
// package.json
{
  "husky": {
    "hooks": {
      "commit-msg": "commitlint -E HUSKY_GIT_PARAMS"
    }
  },
  "config": {
    "commitizen": {
      "path": "cz-conventional-changelog"
    }
  }
}
```

``` js
// commitlint.config.js
module.exports = {
  extends: [
    "@commitlint/config-conventional"
  ]
};
```

``` json
// lerna.json
{
  "npmClient": "yarn",
  "useWorkspaces": true,
  "version": "0.0.12",
  "packages": [
    "packages/*"
  ],
  "command": {
    "publish": {
      "ignoreChanges": [
        "*.md"
      ],
      "verifyAccess": false,
      "verifyRegistry": false,
      "message": "chore: publish"
    }
  },
  "changelog": {
    "repo": "base",
    "labels": {
      "PR: New Feature": ":rocket: New Features",
      "PR: Breaking Change": ":boom: Breaking Changes",
      "PR: Bug Fix": ":bug: Bug Fix",
      "PR: Documentation": ":memo: Documentation",
      "PR: Internal": ":house: Internal",
      "PR: Underlying Tools": ":hammer: Underlying Tools"
    },
    "cacheDir": ".changelog"
  }
}
```

#### 提交发布

git备注上面的更改信息

``` bash
gaa
gcmsg 'fix(lerna) add vue-lazy-view' -n
gp origin master
```

生成并提交`changelog`

``` bash
npx standard-version
```

万事俱备，发布把

``` bash
lerna publish
```

### 缺陷

不提供单独发包的形式，单独发包得 `npm publish`
