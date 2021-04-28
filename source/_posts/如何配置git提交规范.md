---
title: git提交规范及如何配置
date: 2021-01-15 16:31:20
tags: 工程配置
categories: 工程配置
---

`Git` 是现在比较流行的版本控制工具，在开发的过程中，`Git `每次提交代码，都需要写`Commit message `（即提交说明）。如果没有对 `Commit message `进行规范，会造成很多的麻烦，比如：

* 每个人的 `Commit message `风格不同，格式凌乱，查看就换个不方便
* 有一些commit  没有写 `message`，事后就很难知道对应修改的作用。



 所以说规范的 Commit `message `是很有必要的。也是有很多的好处的，比如：

* 可以统一团队的`Git commit `日志风格
* 方便日后查阅， `Reviewing Code`等
* 可以帮助我们写好 `Changelog`
* 能提升项目的整体质量

 <!--more-->

我们要配置`git`提交规范的话， 肯定需要知道它的规范是什么？先来看` Git Commit ` 规范 是什么？

# Git Commit  规范

目前规范使用较多的是 [Angular 团队得规范](https://www.barretlee.com/blog/2019/10/28/commit-convention/) 我们也叫这种规范做: **Git 约定式提交规范**。

这种规范提供了一中轻量级的的提交历史编写规则，它的内容十分的简单:

它包含了三个部分：`Heade`r，`Body`，`Footer`

```javascript
// 注意冒号 : 后有空格
<type>(<scope>): <subject>
//  空一行
<body>
// 空一行
<footer>
```

其中，`Header` 是必需的，`Body` 和` Footer` 可以省略。

> **注意：** 不管是哪一个部分，任何一行都不得操作72个字符（或者100个字符）。 当然， 这只是避免自动换行影响美观而已。

## Header 

`Header` 部分 只有一行，包括三个字段：**`type(必填`)**，**`scope(可选)`**，**`subject(必填)`**

### type

`type` 字段用于说明` commit` 的类别，它允许使用下面的标识

* `feat`：新功能（`feature`）

* `fix`： 修改`bug`

* `docs`：文档（`documentation`）

* `style`：格式（不影响代码运行的变动）

* `refactor`：重构（即不是新增功能，也不是修改`bug`的代码变动）

* `test`：增加测试

* `chore`：构建过程或者辅助工具的变动

### scope

`scope `用于说明  `commit `的影响范围，比如： 数据层，控制层，视图层等等。因具体项目而定。

### subject

`subject `用于说明 `commit` 目的的简短描述，最好不要操作50 个字符。我们在写`ubject`的时候需要注意：

* 以动词开头，使用第一人称现在时，比如：`change`，而不是`changed` 或者 `changes`
* 第一个字母小写
* 结尾不加句号

## Body

`Body` 部分是对本次 `commit` 的详细描述，可以分成多行。写Body 部分的时候也需要注意：

- 使用第一人称现在时候， 比如使用 `change` 而不是 `changed` 或 `changes`
- 应该说明代码变动的动机，以及与以前行为的对比



## Footer

Footer 部分只用于这两种情况：

### 不兼容变动

如果当前的代码与上一版本不兼容，则`Footer` 部分以` BREAKING CHANGE` 开头， 后面是对变动的描述，以及变动的理由和迁移的办法。

```javascript
BREAKING CHANGE: isolate scope bindings definition has changed.

    To migrate the code follow the example below:

    Before:

    scope: {
      myAttr: 'attribute',
    }

    After:

    scope: {
      myAttr: '@',
    }

    The removed `inject` wasn't generaly useful for directives so there should be no code using it.
```

### 关闭Issue

如果当前的`commit` 是针对某个` issue`。那么在`footer`部分关闭这个`iissue`。可以一次性关系多个`issue`

```javascript
closes #123, #245, #992
```



## Revert

 有一种特殊情况， 如果当前的`commit` 是用于撤销 以前的``commit `的，则必须以`Revert `开头。 后面紧跟着被撤销的`Commit` 的 `Header`。

`Body`部分的格式是固定的，必须写成`This reverts commit <hash>.`，其中的`hash`是被撤销` commit `的 `SHA` 标识符。

如果当前` commit` 与被撤销的`commit`，在同一个发布（`release`）里面，那么它们都不会出现在 `Change log` 里面。如果两者在不同的发布，那么当前 `commit`，会出现在 `Change log` 的`Reverts`小标题下面。

  ## 举个例子

我们新建一个文件夹:

```javascript
mkdir  gitCommit
```

然后进入到这个文件夹:

```javascript
cd gitCommit
```

执行：

```javascript
git init
```

新建一个文件: 

```javascript
touch index.js
```

执行:

```javascript
git add
```



这时候我们开始写提交的`commit message`，命令行输入

```
git commit
```

这时候会跳出编辑器让我们编写` message`， 我们则可以写成：

```javascript
 feat:(*): 添加index.js文件
 
 在根目录添加了indexjs文件
 
```

这`commit message`的 `Header` 为` feat:(*): 添加index.js文件`），表示：我们加了一个新功能（`type = feat`），它的影响范围是*（`scope = `*）, 它的简短描述(`subject`)是：添加`index.js`文件

这`commit message` 的` Body` 为： ` 在根目录添加了indexjs文件`。表示这次的`commit` 的简单描述是： ` 在根目录添加了indexjs文件`



如果我们要撤销上面的 `commit` 。 我们这可以 使用 `git revert <commitId>`

我们可以下先使用 `git log`, 找出要 revert 的 commitId 0（f6c37576c793b2e2f4e66a87f96c2e825e073d35）

然后执行：`git revert f6c37576c793b2e2f4e66a87f96c2e825e073d35`。

```javascript
  Revert "feat:(*): 添加index.js文件"

  This reverts commit f6c37576c793b2e2f4e66a87f96c2e825e073d35.
```

这是的提交规范是这样的，已经默认帮我们写好了。当然我们也是可以修改的。

# 配置 git 提交规范

当然，在我们的日常开发当中， 需要记住上面的规范， 有点繁琐。我们的目标还是要通过工具生成和约束。 那么我们现在就来配置吧！

## Commitizen

那么有什么 工具可以 做到生成规范并且约束呢？ [Commitizen](https://github.com/commitizen/cz-cli)就是一个很不错的， 很合格的工具，

`Commitizen/cz-cli`: 是一个格式化 `commit message` 的工具，可以约束提交者按照制定的规范一步一步的填写 `commit message`。

**安装**

```
 npm install commitizen --save
```

然后在项目的根目录里， 执行以下的命令，使其支持`Angular` 的 `commit message` 格式。

```
 commitizen init cz-conventional-changelog --save --save-exact
```

**配置**

我们打开` packge.json`。 可以看到配置为：

```
"config": {
  "commitizen": {
    "path": "./node_modules/cz-conventional-changelog"
  }
}
```

之后，只要是用到 `git commit `命令，一律改为使用 `git cz`， 然后就会出现选项， 用来生成符合格式的 `commit message`

```javascript
git cz

cz-cli@4.2.3, cz-conventional-changelog@3.3.0

? Select the type of change that you're committing: (Use arrow keys)
❯ feat:     A new feature 
  fix:      A bug fix 
  docs:     Documentation only changes 
  style:    Changes that do not affect the meaning of the code (white-space, formatting, missing semi-colons, etc) 
  refactor: A code change that neither fixes a bug nor adds a feature 
  perf:     A code change that improves performance 
  test:     Adding missing tests or correcting existing tests 
```

## cz-customizable

上面是直接使用 `cz-conventional-changelog` 作为 Adapter。但是如果需要自定义` Adapter`， 比如：默认提交的`types `非常多；或者有些使用我们可能只需要其中的某些`type`。或者自定义一些`type`，那么就可以通过 `cz-customizable` 来自定义了。



**安装**

```javascript
 npm i cz-customizable --save
```

将之前符合`Angular`规范的**`cz-conventional-changelog`**适配器路径改成**`cz-customizable`**适配器路径：  **"`path`": "`./node_modules/cz-conventional-changelog`" **改成  **"`path`": "`./node_modules/cz-customizable`"**

````javascript
 "config": {
    "commitizen": {
      "path": "./node_modules/cz-customizable"
    }
  },
````

**配置**

在跟目录下新建.`cz-config.js`。 内容的示例文件如：[cz-config-EXAMPLE.js](https://github.com/leonardoanalista/cz-customizable/blob/master/cz-config-EXAMPLE.js)

```javascript
'use strict';

module.exports = {

  types: [
    {value: 'feat',     name: 'feat:     A new feature'},
    {value: 'fix',      name: 'fix:      A bug fix'},
    {value: 'docs',     name: 'docs:     Documentation only changes'},
    {value: 'style',    name: 'style:    Changes that do not affect the meaning of the code\n            (white-space, formatting, missing semi-colons, etc)'},
    {value: 'refactor', name: 'refactor: A code change that neither fixes a bug nor adds a feature'},
    {value: 'perf',     name: 'perf:     A code change that improves performance'},
    {value: 'test',     name: 'test:     Adding missing tests'},
    {value: 'chore',    name: 'chore:    Changes to the build process or auxiliary tools\n            and libraries such as documentation generation'},
    {value: 'revert',   name: 'revert:   Revert to a commit'},
    {value: 'WIP',      name: 'WIP:      Work in progress'}
  ],

  scopes: [
    {name: 'accounts'},
    {name: 'admin'},
    {name: 'exampleScope'},
    {name: 'changeMe'}
  ],

  // it needs to match the value for field type. Eg.: 'fix'
  /*
  scopeOverrides: {
    fix: [
      {name: 'merge'},
      {name: 'style'},
      {name: 'e2eTest'},
      {name: 'unitTest'}
    ]
  },
  */
  // override the messages, defaults are as follows
  messages: {
    type: 'Select the type of change that you\'re committing:',
    scope: '\nDenote the SCOPE of this change (optional):',
    // used if allowCustomScopes is true
    customScope: 'Denote the SCOPE of this change:',
    subject: 'Write a SHORT, IMPERATIVE tense description of the change:\n',
    body: 'Provide a LONGER description of the change (optional). Use "|" to break new line:\n',
    breaking: 'List any BREAKING CHANGES (optional):\n',
    footer: 'List any ISSUES CLOSED by this change (optional). E.g.: #31, #34:\n',
    confirmCommit: 'Are you sure you want to proceed with the commit above?'
  },

  allowCustomScopes: true,
  allowBreakingChanges: ['feat', 'fix'],

  // limit subject length
  subjectLimit: 100
  
};
```

我们可以对这个配置进行汉化处理：

```javascript
module.exports = {
  // type 类型
  types: [
    { value: 'feat', name: 'feat:     新增产品功能' },
    { value: 'fix', name: 'fix:      修复 bug' },
    { value: 'docs', name: 'docs:     文档的变更' },
    {
      value: 'style',
      name:
        'style:    不改变代码功能的变动(如删除空格、格式化、去掉末尾分号等)',
    },
    {
      value: 'refactor',
      name: 'refactor: 重构代码。不包括 bug 修复、功能新增',
    },
    {
      value: 'perf',
      name: 'perf:     性能优化',
    },
    { value: 'test', name: 'test:     添加、修改测试用例' },
    {
      value: 'build',
      name: 'build:    构建流程、外部依赖变更，比如升级 npm 包、修改 webpack 配置'
    },
    { value: 'ci', name: 'ci:       修改了 CI 配置、脚本' },
    {
      value: 'chore',
      name: 'chore:    对构建过程或辅助工具和库的更改,不影响源文件、测试用例的其他操作',
    },
    { value: 'revert', name: 'revert:   回滚 commit' },

  ],

  // scope 类型，针对 React 项目
  scopes: [
    ['components', '组件相关'],
    ['hooks', 'hook 相关'],
    ['hoc', 'HOC'],
    ['utils', 'utils 相关'],
    ['antd', '对 antd 主题的调整'],
    ['element-ui', '对 element-ui 主题的调整'],
    ['styles', '样式相关'],
    ['deps', '项目依赖'],
    ['auth', '对 auth 修改'],
    ['other', '其他修改'],
    // 如果选择 custom ,后面会让你再输入一个自定义的 scope , 也可以不设置此项， 把后面的 allowCustomScopes 设置为 true
    ['custom', '以上都不是？我要自定义'],
  ].map(([value, description]) => {
    return {
      value,
      name: `${value.padEnd(30)} (${description})`
    };
  }),

  // allowTicketNumber: false,
  // isTicketNumberRequired: false,
  // ticketNumberPrefix: 'TICKET-',
  // ticketNumberRegExp: '\\d{1,5}',

  // 可以设置 scope 的类型跟 type 的类型匹配项，例如: 'fix'
  /*
    scopeOverrides: {
      fix: [
        { name: 'merge' },
        { name: 'style' },
        { name: 'e2eTest' },
        { name: 'unitTest' }
      ]
    },
   */
  // 覆写提示的信息
  messages: {
    type: "请确保你的提交遵循了原子提交规范！\n选择你要提交的类型:",
    scope: '\n选择一个 scope (可选):',
    // 选择 scope: custom 时会出下面的提示
    customScope: '请输入自定义的 scope:',
    subject: '填写一个简短精炼的描述语句:\n',
    body: '添加一个更加详细的描述，可以附上新增功能的描述或 bug 链接、截图链接 (可选)。使用 "|" 换行:\n',
    breaking: '列举非兼容性重大的变更 (可选):\n',
    footer: '列举出所有变更的 ISSUES CLOSED  (可选)。 例如.: #31, #34:\n',
    confirmCommit: '确认提交?',
  },

  // 是否允许自定义填写 scope ，设置为 true ，会自动添加两个 scope 类型 [{ name: 'empty', value: false },{ name: 'custom', value: 'custom' }]
  // allowCustomScopes: true,
  allowBreakingChanges: ['feat', 'fix'],
  // skip any questions you want
  // skipQuestions: [],

  // subject 限制长度
  subjectLimit: 100,
  // breaklineChar: '|', // 支持 body 和 footer
  // footerPrefix : 'ISSUES CLOSED:'
  // askForBreakingChangeFirst : true,
};

```

到此， 我们使用 `cz-customizable`  就自定义配置完了。

最后我们使用 `git cz`命令进行提交说明：

   {% asset_img 1.png %}

上图我们就可以看出此时的提交说明已经汉化， 我们继续写提交说明：

   {% asset_img 2.png %}

最后我们提交到远端看到效果如下：

  {% asset_img 3.png %}



## Commitizen校验

我们前面已经约束了一套代码规范提交说明了， 但是还是有人不按照规范提交代码说明怎么呢？， 那么就需要 [commitlint](https://github.com/marionebl/commitlint)来校验 commit 了。

**安装commitlint**

```javascript
 npm install --save-dev @commitlint/cli
```

**安装@commitlint/config-conventional**

安装符合` Abgular `风格校验规则：

```javascript
npm install --save-dev @commitlint/config-conventional
```

然后在项目中新建 `commitlint.config.js`文件，并且设置校验规则：

```javascript
module.exports = {
  extends: ['@commitlint/config-conventional']
};
```

**安装安装huksy（git钩子工具）**

```javascript
npm install husky --save-dev
```

然后在packge.json 中配置 `git commit`提交时的钩子：

```javascript
"husky": {
  "hooks": {
    "commit-msg": "commitlint -E HUSKY_GIT_PARAMS"
  }  
}
```

**需要注意**，使用该校验规则不能对`.cz-config.js`进行不符合`Angular`规范的定制处理，例如之前的汉化，此时需要将`.cz-config.js`的文件按照官方示例文件[cz-config-EXAMPLE.js](https://github.com/leonardoanalista/cz-customizable/blob/master/cz-config-EXAMPLE.js)进行符合`Angular`风格的改动。

最后我们来试一试：

提交不符合规范的错误提示：

  {% asset_img 4.png %}

提交符合规范的提示：

  {% asset_img 5.png %}



**需要注意**：如果使用了 **cz-customizable**配器做了破坏`Angular`风格的提交说明配置，那么不能使用**@commitlint/config-conventional**规则进行提交说明校验，可以使用[commitlint-config-cz](https://github.com/whizark/commitlint-config-cz)对定制化提交说明进行校验。

**安装：**

```javascript
npm install commitlint-config-cz --save-dev
```

然后加入`commitlint`校验规则配置：

```javascript
module.exports = {
  extends: [
    'cz'
  ]
};

```

> 这里推荐使用**@commitlint/config-conventional**校验规则，如果想使用`cz-customizable`适配器，那么定制化的配置不要破坏Angular规范即可。

# git commit 触发 git cz

在提交的时候，我们都习惯了 `git commit` ，虽然换成 `git cz` 不难，但是如果让开发者在 `git commit` 时无感知的触发 `git cz` 肯定是更好的， 而且也能避免不熟悉项目的人直接 `git commit` 提交一些不符合规范的信息。

我们可以在` husky.config.js `中设置：

```javascript
"hooks": {
  "prepare-commit-msg": "exec < /dev/tty && git cz --hook || true",
}
```











**参考文档：**

[让你的commit更有价值](https://juejin.cn/post/6854573220176068615)

[cz-customizable](https://github.com/leoforfree/cz-customizable)

[cz-cli](https://github.com/commitizen/cz-cli)

[小胡子哥的个人博客](https://www.barretlee.com/blog/2019/10/28/commit-convention/)

[Commit message 和 Change log 编写指南](https://www.ruanyifeng.com/blog/2016/01/commit_message_change_log.html)