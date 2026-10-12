# codebee-plugins-registry


那个仓库长什么样
codebee-plugins-registry/      <- 你 push 上去的仓库
    registry.json              <- 唯一被读取的文件
    README.md                  <- 可选
registry.json 内容（索引里每条才指向真正的插件仓库）：
{
  "name": "CodeBee Plugins",
  "packages": [
    {
      "id": "com.example.tray",                       // 插件 id（= 其 plugin.json 的 id）
      "name": "System Tray",
      "description": "常驻托盘 + 全局快捷键",
      "repository": "https://github.com/you/codebee-tray.git",  // 真正的插件仓库
      "ref": "v1.2.0",                                // 默认 tag/分支/commit
      "path": "plugin",                               // 可选：插件在仓库里的子目录
      "tags": ["tray"],
      "versions": ["v1.2.0", "v1.0.0"]                // 可选，仅展示
    }
  ]
}
三者关系
packages.json                registry.json                 插件仓库
 registries:[ url ]  ──clone──►  packages[]  ──Install──►  写回 packages.json 的 dependencies
                                                          （repository/ref/path 变成一条依赖）
- registries 里的 URL 只提供可安装列表（面板里的 "Available"）。
- 你点某个包的 Install 时，它的 repository/ref/path 会被写成 packages.json 里的一条 dependencies，然后才真正去 clone 那个插件仓库。
- 所以 registries 可以留空——你完全可以直接在 packages.json 的 dependencies 里手写插件 git 地址，不依赖任何注册表。
最小配置示例
packages.json：
{
  "registries": ["https://github.com/you/codebee-plugins-registry.git"],
  "dependencies": {}
}


## 3. `packages.json` 参考

```json
{
  "registries": [
    "https://github.com/you/codebee-plugins-registry.git"
  ],
  "dependencies": {
    "com.example.tray": {
      "source": "git",
      "url": "https://github.com/you/codebee-tray.git",
      "ref": "v1.2.0",
      "path": "plugin"
    },
    "com.example.hex": {
      "url": "https://github.com/you/codebee-monorepo.git?path=/packages/hex#main"
    }
  }
}
```

| 字段 | 必填 | 说明 |
|---|---|---|
| `registries` | 否 | 注册表仓库 URL 列表（见 §5）。用于发现可安装插件 |
| `dependencies` | 否 | 以**插件 id**为键。id 必须与插件 `plugin.json` 的 `id` 一致 |
| `dependencies.<id>.source` | 否 | 目前只支持 `"git"`（默认） |
| `dependencies.<id>.url` | 是 | git 地址（也支持 Unity 风格，见 §4） |
| `dependencies.<id>.ref` | 否 | tag / 分支 / commit；空 = 默认分支 |
| `dependencies.<id>.path` | 否 | 插件在仓库里的**子目录**；空 = 仓库根 |

> `path` 里的插件（即包含 `plugin.json` 的那一层）会被复制到缓存目录根部，
> 所以 `plugin.json` 放在 `path` 指向的目录里即可。

---

## 4. 指定文件夹加载（Unity 风格）

一个 git 仓库里可以放**多个插件**，用子目录区分。三种写法等价：

**A. 显式字段**
```json
"com.example.hex": { "url": "https://github.com/you/monorepo.git", "ref": "main", "path": "packages/hex" }
```

**B. Unity 式 URL（`?path=` + `#ref`）** — 可以直接从 Unity 项目的写法粘过来：
```json
"com.example.hex": { "url": "https://github.com/you/monorepo.git?path=/packages/hex#main" }
```
`?path=` 指定子目录（前导 `/` 可省），`#` 后面是 ref。CodeBee 会把它拆成
`url` / `path` / `ref` 再用。

**C. 仓库根**（不写 `path`）：
```json
"com.example.simple": { "url": "https://github.com/you/simple.git", "ref": "v0.1.0" }
```

monorepo 示例见 [`monorepo.example.json`](monorepo.example.json)。

> 显式写了 `ref` / `path` 时，以字段为准，URL 里的对应部分会被忽略。

---

## 5. 插件管理仓库（registry）

注册表就是一个 **git 仓库**，根目录放 `registry.json`：

```json
{
  "name": "CodeBee Plugins",
  "packages": [
    {
      "id": "com.example.tray",
      "name": "System Tray",
      "description": "常驻托盘 + 全局快捷键",
      "repository": "https://github.com/you/codebee-tray.git",
      "ref": "v1.2.0",
      "path": "plugin",
      "tags": ["tray"],
      "versions": ["v1.2.0", "v1.0.0"]
    },
    {
      "id": "com.example.hex",
      "name": "Hex View",
      "repository": "https://github.com/you/monorepo.git",
      "ref": "main",
      "path": "packages/hex"
    }
  ]
}
```

| 字段 | 说明 |
|---|---|
| `repository` | 插件 git 地址（别名 `url`）。`Install` 时作为依赖的 `url` |
| `ref` / `path` | 默认 ref 与子目录，同 §4 |
| `tags` / `versions` | 可选，仅用于展示/筛选 |

把注册表 URL 填进 `packages.json` 的 `registries`，点 **Refresh Registries**，
面板就会出现「Available」列表，逐条 `Install`。示例见
[`../samples/registry.example.json`](../samples/registry.example.json)。

---

## 6. 发布你自己的插件仓库

最小仓库结构（`path` 为空时，`plugin.json` 放仓库根）：

```
my-plugin/
    plugin.json      # { "id": "com.you.myplugin", "version": "1.0.0", "runtime": "lua", "main": "main.lua" }
    main.lua
    README.md
```

1. `git init` 并提交，打 tag：`git tag v1.0.0 && git push --tags`。
2. 在依赖里用 `url` + `ref: "v1.0.0"`（tag 固定）或 `ref: "main"`（跟随分支更新）。
3. monorepo：把每个插件放在子目录，依赖里分别写 `path`（§4）。

---
