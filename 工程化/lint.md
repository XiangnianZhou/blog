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


# 参考
- [一步一步，统一项目中的编码规范](https://juejin.im/post/5cbfde7c5188250a7d6ddcd1)
- [Vscode-prettier](https://marketplace.visualstudio.com/items?itemName=esbenp.prettier-vscode)

