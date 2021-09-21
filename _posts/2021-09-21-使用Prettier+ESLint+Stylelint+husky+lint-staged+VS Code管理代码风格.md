---
layout: mypost
title: 使用Prettier+ESLint+Stylelint+husky+lint-staged+VS Code管理代码风格
categories: [代码规范]
---

本文主要介绍如何在团队中定制出一套统一的代码风格规则并且较好的执行下去。

## 背景

在前端项目中，代码格式规则分为两部分，格式规则和质量规则。格式规则主要是规定一些如每行长度和单双引号等习惯导致不统一的规则；质量规则主要是规定一些如全等使用等可能会引起bug的规则。在团队中，这两部分的规则本来就很难被完全规定出来，且不同人员也无法说服对方哪一种方案更好。这时，就需要一个外部声音说，这些规则我来帮你们统一检查和修正，你们无需再讨论了。这些工具就是Prettier、ESLint、Stylelint。

现在前端开发几乎都是在各个IDE中进行，相比文本编辑器。IDE提供了更加强大的功能，如环境集成，调试集成，git集成等等。有了上述工具，我们还可以将其整合进各个IDE中，利用IDE帮助我们进行风格管理和自动化操作。

## Lint插件

要进行风格管理，往往需要多个插件配合使用，各个插件各司其职，相互整合，达到较好的管理效果。

### [Prettier](https://prettier.io/)

Prettier 是一个 Opinionated 的代码格式化工具。支持前端几乎所有语言和扩展文件。它制定了一套格式规则，并且只提供少量的可修改的规则。使用它就意味着你要接受它提供的规则，这也解决了前面提到的无法统一规则的问题。并且它提供了很好的IDE支持，使得在开发中能够及时检查和修改代码。

-   安装使用

    Prettier对大部分IDE都提供了插件，利用IDE的特性可以在保持文件时自动执行格式化操作，但如果你需要手动格式化，或初始格式化一个。就需要在项目中安装依赖配置脚本了。

    1.  运行 `npm install -D prettier` 即可安装到项目中。
    1.  添加 `"lint": "prettier --write **/*.{html,js,jsx,vue}"` 命令，运行 `npm run lint` 即可手动格式化代码

-   配置修改

    Prettier提供了文件 `.prettierrc` 修改默认配置规则，提供文件 `.prettierignore` 设置忽略文件

    ```yml
    # .prettierignore

    dist
    node_modules
    public
    src/assets
    ```

### [ESLint](https://eslint.org/)

前面介绍的Prettier主要规范了格式规则，而ESLint主要负责JS的代码质量规则。

ESLint是一个强大且灵活的Lint工具，它提供了大量的规则，内部通过 **extends** , **plugins**, **rules** 等机制去定制。在这里ESLint和Prettier的规则是有重复的。下面分别介绍怎么安装配置ESLint以及和Prettier整合

-   安装使用

    ESLint同样对大部分IDE提供了插件，利用IDE的特性可以在保持文件时自动执行格式化操作。但是一般项目中是需要根据项目框架等指定ESLint的，所以最好还会安装依赖。

    1.  运行 `npm install -D eslint` 即可安装到项目中。根据具体项目框架等，可以安装对应的ESLint插件

        ```json
        // package.json
        {
            "devDependencies": {
                "eslint": "^7.32.0",
                "eslint-config-prettier": "^8.3.0",
                "eslint-plugin-prettier": "^4.0.0",
                "eslint-plugin-vue": "^7.17.0",
            }
        }
        ```

    1.  添加 `"lint": "eslint --fix **/*.{js,jsx,vue}"` 命令，运行 `npm run lint` 即可手动格式化代码

-   修改配置

    ESLint提供了文件 `.eslintrc` 修改默认配置规则，提供文件 `.eslintignore` 设置忽略文件

    ```json
    // .eslintrc
    {
      "root": true,
      "env": {
        "browser": true,
        "es2021": true,
        "node": true
      },
      "extends": [
        "eslint:recommended",
      ],
      "globals": {
        // vue3
        "defineProps": "readonly",
        "defineEmits": "readonly",
        "defineExpose": "readonly",
        "withDefaults": "readonly"
      },
      "rules": {
        "no-unused-vars": "warn"
      }
    }
    ```

-   整合Prettier

    Prettier 提供了 `eslint-config-prettier` 和 `eslint-plugin-prettier` 两个针对ESLint的包，前者配置ESLint关闭与Prettier冲突的规则，后者让Prettier作为一个插件运行在ESLint中，通过ESLint统一检查修改两部分规则

    1.  安装两个包 `npm install -D eslint-config-prettier eslint-plugin-prettier`
    1.  在 **.eslintrc** 中添加配置 `{"extends": ["plugin:prettier/recommended"]}`

### [Stylelint](https://stylelint.io/)

前面介绍的Prettier主要规范了格式规则，包括css，而Stylelint主要负责CSS的代码质量规则

Stylelint提供了更强大的样式格式规则

-   安装使用

    1.  运行 `npm install -D stylelint stylelint-config-standard` 即可安装到项目中。根据具体项目框架等，可以安装对应的Stylelint插件

        ```json
        // package.json
        {
            "devDependencies": {
        	"stylelint": "^13.13.1",
                "stylelint-config-rational-order": "^0.1.2",
                "stylelint-config-standard": "^22.0.0",
            }
        }
        ```

    1.  添加 `"lint:css": "stylelint --fix **/*.{css,less,sass,scss,html,vue}"` 命令，运行 `npm run lint` 即可手动格式化代码

-   修改配置

    Stylelint提供了文件 `.stylelintrc` 修改默认配置规则，提供文件 `.stylelintignore` 设置忽略文件

    ```json
    // .stylelintrc
    {
      "extends": [
        "stylelint-config-standard",
        "stylelint-config-rational-order",
      ]
    }
    ```

-   整合Prettier

    Prettier 提供了 `stylelint-config-prettier` 和 `eslint-prettier` 两个针对Stylelint的包，前者配置Stylelint关闭与Prettier冲突的规则，后者让Prettier作为一个插件运行在Stylelint中，通过Stylelint统一检查修改两部分规则

    1.  安装两个包 `npm install -D stylelint-config-prettier stylelint-prettier`
    1.  在 **.stylelintrc** 中添加配置 `{"extends": ["stylelint-prettier/recommended"]}`

## IDE整合

VS Code中对Prettier，ESLint，Stylelint都提供了对应的插件，利用插件和配置，可以做到在编辑时，对对应格式的文件进行检查，在保存时自动格式化。配置如下

```json
{
    //* editor
    "editor.codeActionsOnSave": {
        "source.fixAll.eslint": true,
        "source.fixAll.stylelint": true
    },
    // format
    // "editor.formatOnSave": true,
    "editor.defaultFormatter": "esbenp.prettier-vscode",
}
```

## Git整合

通过husky，link-staged和git hook机制，可以实现提交时进行格式化检测和格式操作，保证代码提交的质量。

1.  运行 `npm install -D husky lint-staged` 安装husky、lint-staged

1.  运行 `npx husky install` 初始化husky配置目录 **.husky**

1.  运行 `npx husky add .husky/pre-commit "npx lint-staged"` 配置pre-commit hook

1.  配置 lint-staged，可在 `.lintstagedrc` 或 `package.json` 中配置

    ```json
    // package.json
    {
        "lint-staged": {
            "*.{html,js,jsx,vue}": [
                "npm run lint:html"
            ],
            "*.{css,less,sass,scss,html,vue}": [
                "npm run lint:css"
            ],
            "*.{js,jsx,vue}": [
                "npm run lint:js"
            ]
        },
    }
    ```

## Reference

[Prettier看这一篇就行了](https://zhuanlan.zhihu.com/p/81764012)

[Git Hooks、husky、lint-staged](https://juejin.cn/post/6955354072656379911)