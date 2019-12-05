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
1. 为啥没有很少有人说csslint和lesslint？难道只需要格式化不需要检查？？
2. 大厂的less、css 在检查（lint命令）的时候是不检查的？



# 参考
- [一步一步，统一项目中的编码规范](https://juejin.im/post/5cbfde7c5188250a7d6ddcd1)
- [Vscode-prettier](https://marketplace.visualstudio.com/items?itemName=esbenp.prettier-vscode)
- [vscode+vetur+eslint+prettier实现团队代码风格统一](https://trainspott.in/2018/12/07/vscode+vetur+eslint+prettier%E5%AE%9E%E7%8E%B0%E5%9B%A2%E9%98%9F%E4%BB%A3%E7%A0%81%E9%A3%8E%E6%A0%BC%E7%BB%9F%E4%B8%80/)
- [eslint-plugin-vue](https://eslint.vuejs.org/)
- [Vetur](https://vuejs.github.io/vetur/linting-error.html#error-checking)
- [Prettier](https://prettier.io/docs/en/comparison.html)
- [prettier-vscode](https://marketplace.visualstudio.com/items?itemName=esbenp.prettier-vscode)

