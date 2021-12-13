---
theme: default
background: https://pt-starimg.didistatic.com/static/starimg/img/tvbJynvHBP1639379103598.jpeg
class: text-center
highlighter: shiki
lineNumbers: false
info: |
  ## Slidev Starter Template
  Presentation slides for developers.

  Learn more at [Sli.dev](https://sli.dev)
drawings:
  persist: false
title: unplugin
---

# 使用 unplugin 开发构建插件

vite / webpack / rollup

<div class="pt-12">
  <span @click="$slidev.nav.next" class="px-2 py-1 rounded cursor-pointer" hover="bg-white bg-opacity-10">
    <carbon:arrow-right class="inline"/>
  </span>
</div>

<!--
The last comment block of each slide will be treated as slide notes. It will be visible and editable in Presenter Mode along with the slide. [Read more in the docs](https://sli.dev/guide/syntax.html#notes)
-->

---

# What is unplugin?

用于开发构建工具插件的聚合系统

- 🛠 **省事** - 1 次开发，多处使用
- 📝 **可靠** - 社区已有大量实践，如 *nuxt*
- 🎨 **简单** - 基于 rollup plugin API 拓展，上手方便

<br>
<br>

**已支持**

- Rollup
- Vite
- Webpack4
- Webpack5

<p style="text-align: right">

[了解更多](https://github.com/unjs/unplugin) 

</p>

<!--
You can have `style` tag in markdown to override the style for the current page.
Learn more: https://sli.dev/guide/syntax#embedded-styles
-->

<style>
h1 {
  background-color: #2B90B6;
  background-image: linear-gradient(45deg, #4EC5D4 10%,  20%);
  background-size: 100%;
  -webkit-background-clip: text;
  -moz-background-clip: text;
  -webkit-text-fill-color: transparent;
  -moz-text-fill-color: transparent;
}
</style>

---

# 快速开始

```ts
import { createUnplugin } from 'unplugin'

export const unplugin = createUnplugin((options: UserOptions) => {
  return {
    name: 'my-first-unplugin',
    transformInclude (id) {
      return id.endsWith('.vue')
    },
    transform (code) {
      return code.replace(
        /<template>/,
        `<template><div>Injected</div>`
      )
    },
  }
})

export const vitePlugin = unplugin.vite
export const rollupPlugin = unplugin.rollup
export const webpackPlugin = unplugin.webpack
```

<style>
.footnotes-sep {
  @apply mt-20 opacity-10;
}
.footnotes {
  @apply text-sm opacity-75;
}
.footnote-backref {
  display: none;
}
</style>

---
layout: two-cols
image: https://pt-starimg.didistatic.com/static/starimg/img/i5q24yIGyX1639379237798.png
class: small
---

# 插件 hooks

插件的 *生命周期*

- transform <sup>🌶🌶🌶</sup>

  <span class="text-xs">最常用的钩子，可读取并转换代码了；一般的操作是字符串替换。</span>

- buildStart <sup>🌶🌶</sup>

  <span class="text-xs">开始构建钩子，可读取并改写配置等。</span>

- transformInclude <sup>*🌶🌶</sup>

  <span class="text-xs">类似过滤器，可以根据 id（文件名）来决定处理与否</span>

- resolveId <sup>🌶</sup>

  <span class="text-xs">解析到文件资源</span>

- load <sup>🌶</sup>

  <span class="text-xs">加载文件资源，可以返回代码字符串</span>

::right::

#

<div
  class="h-full bg-contain bg-no-repeat"
  style="
    backgroundImage: url('https://pt-starimg.didistatic.com/static/starimg/img/i5q24yIGyX1639379237798.png');
  "
/>

<style>
.small li p {
  margin: .5rem 0;
}
</style>

---

# 实战
实现一个 babel-plugin-import，达到按需加载的目的

- 🤔 需求分析

把组件的导入，替换为精确导入；同时导入对应的样式文件 *style.js*

```ts
import { Button } from 'mand-mobile-next';

      ↓ ↓ ↓ ↓ ↓ ↓ 

import Button from 'mand-mobile-next/dist/es/button'
import 'mand-mobile-next/dist/es/button/style.js'
```
- 🚀 环境准备

  <div></div>
  编辑器: <file-icons:vscode color="#2a8ace"/>

---

# 技术选型
做好技术选型事半功倍

- 构建工具：[tsup](https://github.com/egoist/tsup)
  
  <span class="text-xs">基于 esbuild 的极速构建工具 🚁</span>

- 词法解析器：[es-module-lexer](https://github.com/guybedford/es-module-lexer)

  <span class="text-xs">我比 babel 快 100 倍 ⚡️ </span>

- 字符串处理：[magic-string](https://github.com/Rich-Harris/magic-string)

  <span class="text-xs"> 哎，就是玩儿字符串 😁</span>

---
class: code-pre
---

# 编码
怎么优雅怎么来

<div grid="~ cols-2 gap-2" m="-t-2">

  ```ts
  import { init, parse, ImportSpecifier } from 'es-module-lexer'
  import MagicString from 'magic-string'
  import { createFilter } from '@rollup/pluginutils'

  const unplugin = createUnplugin((options: UserOptions) => {
    const include = ['**/*.vue', '**/*.ts', '**/*.js', '**/*.tsx', '**/*.jsx']
    const exclude = 'node_modules/**'

    return {
      name: 'md-import',
      async transform (code, id) {
        if (!code || !filter(id) || !needTransform(code)) return null
        // transformer
      },
    }
  })

  function needTransform(code) {
    return /("|')mand-mobile-next("|')/.test(code)
  }
  ```
  <v-click>

  ```ts
  // transformer
  await init

  let imports: ImportSpecifier[] = [];

  try {
    imports = parse(code)[0]
  } catch (e) {
    console.error(id, e)
    return
  }
  if (!imports.length) {
    return null
  }
  ```

  </v-click>

</div>

<style>
.code-pre h1:last-child {
  opacity: 0;
}
</style>

---
class: code-pre
---

# 编码 <sup>2</sup>
只要能实现，代码难看一点也问题不大

<div grid="~ cols-2 gap-2" m="-t-2">

  ```ts
  let s: MagicString | undefined
  const str = () => s || (s = new MagicString(code))

  for (let index = 0; index < imports.length; index++) {
    const { s: start, e: end, se, ss } = imports[index]
    const name = code.slice(start, end)
    if (!name) {
      continue
    }
    const importStr = code.slice(ss, se)
    const exportVariables = transformImportVar(importStr)
    // overwrite
  }

  function transformImportVar(importStr: string) {
    if (!importStr) return []
    const exportStr = importStr.replace('import', 'export').replace(/\s+as\s+\w+,?/g, ',')
    let exportVariables: string[] = []
    return exportVariables = parse(exportStr)[1]
  }
  ```
  <v-click>

  ```ts
  // overwrite
  str()
    .overwrite(
      ss,
      se,
      exportVariables
        .map(m => {
          const path = `mand-mobile/components/${hyphenate(m)}/index.vue`
          if (fs.existsSync(path)) {
            return `import ${m} from "${path}"`
          } else {
            return `import ${m} from "mand-mobile/components/${hyphenate(m)}"`
          }
        })
        .join('\n')
    )

  const hyphenateRE = /\B([A-Z])/g;
  const hyphenate = function (str) {
    return str.replace(hyphenateRE, '-$1').toLowerCase()
  }
  ```

  </v-click>

</div>

<style>
.code-pre h1:last-child {
  opacity: 0;
}
</style>

---

# 拓展
一些别的库

- [unbuild](https://github.com/unjs/unbuild)
- [consola](https://github.com/unjs/consola)

<span
  style="color: #4EC5D4;"
  class="
    block absolute bottom-8 left-1/2
    -translate-x-1/2
    transform
    text-xs
  ">
  power by @<a target="_black" href="https://sli.dev/">slidev</a>
</span>
