# CLI (zume-cli)

一个命令行伴侣程序 `zume-cli` 随应用一同发布（在 Windows 上位于可执行文件
旁边，在 Linux 上位于 `bin/` 中，在 macOS 上位于应用文件夹中）。它复用编辑器
自身的引擎，因此结果与图形界面一致。

!!! warning "Microsoft Store (MSIX) 版本"
    **Microsoft Store** 版本**不**支持从命令行使用 `zume-cli`。它的文件安装
    在受保护的 `C:\Program Files\WindowsApps\…` 文件夹下（一个随每次更新而
    改变、且不在你的 `PATH` 中的版本专属路径），因此 `zume-cli` 无法直接运行
    或编写脚本。要在 Windows 上使用 CLI——或
    [MCP 服务器](mcp.md)——请改为安装**便携版 ZIP**（或安装程序版本），将其
    解压到一个稳定的文件夹，并从那里运行 `zume-cli.exe`。

```text
zume-cli <command> [arguments] [flags]
```

下列命令是**完整**集合。`zume-cli` 是编辑器刻意精简的无界面子集（完整工具就是
应用本身）；它没有 `--version`，不带参数或以未知命令运行时会打印用法并以代码
`2` 退出。

## 全局标志

| 标志 | 作用 | 适用于 |
| --- | --- | --- |
| `--regex` | 将模式视为正则表达式（默认：字面量） | `find`、`search`、`replace` |
| `--ignore-case` | 不区分大小写匹配（默认：区分大小写） | `find`、`search`、`replace` |

## 退出代码

| 代码 | 含义 |
| --- | --- |
| `0` | 成功 |
| `1` | 运行时错误（向 stderr 打印 `error: <message>`——例如文件未找到、正则表达式错误） |
| `2` | 用法错误（参数错误；打印用法） |

## 文本命令

### `find` - 搜索单个文件

```text
zume-cli find <pattern> <file> [--regex] [--ignore-case]
```
每个匹配打印一行，格式为 `file:line:column`（从 1 开始），然后是汇总
`N match(es)`。

### `search` - 在多个文件中递归查找

```text
zume-cli search <pattern> <dir> [glob] [--regex] [--ignore-case]
```
递归搜索 `dir`。`glob` 可选地限制文件名（例如 `*.cpp`）。
`.git`、`build` 和 `node_modules` 始终被跳过。每个匹配打印
`file:line:column: preview`，然后是 `N match(es) in M file(s)`（如果达到结果
上限则带 `(truncated)`）。

### `replace` - 在单个文件中全部替换

```text
zume-cli replace <pattern> <replacement> <file> [--regex] [--ignore-case]
```
替换每个匹配并就地保存文件（在 `--regex` 模式下，像 `$1` 这样的正则反向引用
有效）。打印 `N replacement(s)`。仅当至少进行了一次替换时才会写入文件。

## `identify` - 文件类型与创建者 {#identify}

```text
zume-cli identify <file>
```
根据文件内容（魔数签名，而非扩展名）识别其类型，并打印 `format`、
`description`、`category`、`confidence`、`mime`、`extensions`、`container` 和
`detail` 行。对于文档或图像，它还会打印 `producer` / `creator`——即写入该文件
的软件——读取自 PDF（`/Producer`、`/Creator`）、EXIF（Software / Make / Model …）、
PNG（`Software`）或 OOXML / ODF（`<Application>` / `<meta:generator>`）的元数据。
只读取有限的前缀（对于 PDF，还包括其尾部）。与编辑器的
**Code → Identify File…** 以及 MCP [`identify`](mcp.md#tools) 工具使用相同的引擎。

```text
$ zume-cli identify report.pdf
format: pdf
description: PDF document
category: document
mime: application/pdf
producer: Microsoft: Print To PDF
creator: Microsoft Word
```

## `aikey` - 生成式 AI API 密钥（用于 MCP 枢纽） {#aikey}

管理 [MCP](mcp.md) `http_request` 工具注入的命名 API 密钥。密钥保存在操作系统的
凭据存储中，绝不存入文件。

```text
zume-cli aikey set <name> [value]     # 存储一个密钥（打印：stored '<name>'）
zume-cli aikey list                   # 列出密钥名称，每行一个
zume-cli aikey remove <name>          # 删除一个密钥（打印：removed '<name>'）
```
如果 `set` 时省略 `value`，密钥将从标准输入读取，这样它就不会留在你的 shell
历史记录中。

## `cloud` - 云端网盘 {#cloud}

操作 **Dropbox、Google Drive、OneDrive 和 Box** 上的文件。刷新令牌保存在操作
系统的凭据存储中（绝不存入文件），并与编辑器共享——在此处或在应用中登录，两边
都生效。

```text
zume-cli cloud <provider> login ...          # 授权并保存令牌
zume-cli cloud <provider> ls [path]          # 列出一个文件夹（默认为根目录）
zume-cli cloud <provider> get <remote> <local>   # 下载一个文件
zume-cli cloud <provider> put <local> <remote>   # 上传一个文件
zume-cli cloud <provider> logout             # 遗忘已保存的令牌
```

`<provider>` 为 `dropbox`、`googledrive`、`onedrive` 或 `box`。`login`
参数因提供商而异（你需要提供自己的 OAuth 应用凭据）：

```text
zume-cli cloud dropbox     login <app_key>
zume-cli cloud googledrive login <client_id> <client_secret>
zume-cli cloud onedrive    login <client_id>
zume-cli cloud box         login <client_id> <client_secret>
```

Dropbox 会显示一个供你粘贴回来的代码；Google / OneDrive / Box 会打开浏览器并在
本地端口上捕获重定向（请在你的 OAuth 应用中将
`http://localhost:53682/callback` 注册为重定向 URI）。`zume-cli dropbox ...`
仍可作为 `zume-cli cloud dropbox ...` 的别名使用。

## `geo` - 地图 / 地理空间数据

复用编辑器的地理引擎（参见 [MAP](map.md)）。文件转换会读取一个文件并将结果打印
到 **stdout**；坐标可以给出为十进制度数或 DMS（例如 `35°41'N`）。

### 格式转换

```text
zume-cli geo geojson2wkt <file>
zume-cli geo wkt2geojson <file>
zume-cli geo geojson2kml <file>
zume-cli geo kml2geojson <file>
zume-cli geo geojson2gpx <file>
zume-cli geo gpx2geojson <file>
zume-cli geo geojson2csv <file>
zume-cli geo csv2geojson <file> [lonCol latCol]
```
`csv2geojson` 会自动检测经度/纬度表头列；如果无法检测，则传入两个**从 0 开始**
的列索引（`lonCol latCol`）。

### 测量

```text
zume-cli geo length <file>     # 测地线长度，单位米（输入为 GeoJSON 或 WKT）
zume-cli geo area   <file>     # 测地线面积，单位平方米
```

### 坐标运算

| 命令 | 参数 | 输出 |
| --- | --- | --- |
| `geohash` | `<lat> <lon> [precision=9]` | geohash 字符串 |
| `unhash` | `<geohash>` | `lat, lon` |
| `jpr` | `<lat> <lon> <zone>` | `northing, easting`（日本平面直角坐标系，带 1-19 号） |
| `unjpr` | `<X> <Y> <zone>` | `lat, lon` |
| `tile` | `<lat> <lon> <zoom>` | `z/x/y  quadkey=…  bbox=west,south,east,north` |
| `dms` | `<lat> <lon>` | 十进制 `lat, lon` |
| `reproject` | `--from <epsg> --to <epsg> <file.geojson>` | 重投影后的 GeoJSON（通过 PROJ） |

## `convert` - 文件转换

```text
zume-cli convert                      # 列出每个转换器 id
zume-cli convert <id> <in> [out]      # 转换一个文件（省略 out 时输出到 stdout）
```

`<id>` 是一个转换器，例如 `base64.encode`、`csv.toJson`、`json.toYaml`、
`gzip.compress`、`bzip2.compress`、`html.toMarkdown`。Markdown / HTML 转 PDF
有其自己的 id，并且总是需要一个输出文件：

```text
zume-cli convert md.toPdf   notes.md   notes.pdf
zume-cli convert html.toPdf page.html  page.pdf
```

完整列表及在编辑器内的对应功能，请参见
[文件转换与归档](convert.md)。

## `archive` - zip / tar / 7z 归档

```text
zume-cli archive list    <archive>                 # 列出条目
zume-cli archive extract <archive> [dest-dir]      # 解压（默认：当前目录）
zume-cli archive create  <archive> <path>...       # 从文件 / 文件夹创建
```

`create` 根据扩展名选择格式：`.zip`、`.tar`、`.tar.gz` / `.tgz`、
`.tar.bz2` / `.tbz2`、`.tar.xz` / `.txz`、`.7z`。`extract` 还可读取 `.rar` 和
`.xz`（只读）。不支持加密归档。

## 示例

```text
zume-cli find TODO src/main.cpp
zume-cli search "fn\s+\w+" src --regex --ignore-case
zume-cli replace http:// https:// config.ini
zume-cli aikey set openai            # 然后在提示符处粘贴密钥
zume-cli geo csv2geojson points.csv
zume-cli geo geohash 35.68 139.76 8
zume-cli geo reproject --from 4326 --to 3857 area.geojson
zume-cli convert json.toYaml config.json config.yaml
zume-cli convert md.toPdf README.md README.pdf
zume-cli archive create site.tar.gz public/
zume-cli archive extract bundle.7z out/
```
