# 前端常说的优化之工作流优化

## vscode 格式化

## git flow

## git commit message

参考另一篇文章。

```shell
npm install --save-dev commitizen
```

## lint

```shell
npm i -D husky lint-staged
```

webpack eslint-loader，因为提示会在页面里，代码里不直观，建议用 eslnt 插件，会直接在代码里显示出错误。

## 其他

vscode 支持 webpack 别名

# vue 项目的格式化

```
npm install --save-dev babel-eslint
npm install --save-dev eslint eslint-plugin-vue


npx install-peerdeps --dev eslint-config-airbnb-base


```

’‘2525

```

{
    "eslint.autoFixOnSave": true,
    "eslint.validate": [
        "javascript",
        "javascriptreact",
        {
            "language": "vue",
            "autoFix": true
        }
    ]
}

```

editor.defaultFormatter

prettier.eslintIntegration
prettier.tslintIntegration
prettier.stylelintIntegration

问题：

1. 为啥没有很少有人说 csslint 和 lesslint？难道只需要格式化不需要检查？？
2. 大厂的 less、css 在检查（lint 命令）的时候是不检查的？

# 参考

- [一步一步，统一项目中的编码规范](https://juejin.im/post/5cbfde7c5188250a7d6ddcd1)
- [Vscode-prettier](https://marketplace.visualstudio.com/items?itemName=esbenp.prettier-vscode)
- [vscode+vetur+eslint+prettier 实现团队代码风格统一](https://trainspott.in/2018/12/07/vscode+vetur+eslint+prettier%E5%AE%9E%E7%8E%B0%E5%9B%A2%E9%98%9F%E4%BB%A3%E7%A0%81%E9%A3%8E%E6%A0%BC%E7%BB%9F%E4%B8%80/)
- [eslint-plugin-vue](https://eslint.vuejs.org/)
- [Vetur](https://vuejs.github.io/vetur/linting-error.html#error-checking)
- [Prettier](https://prettier.io/docs/en/comparison.html)
- [prettier-vscode](https://marketplace.visualstudio.com/items?itemName=esbenp.prettier-vscode)
