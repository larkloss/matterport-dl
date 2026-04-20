# CLAUDE.md

## Project Overview

基于 [matterport-dl](https://github.com/rebane2001/matterport-dl) 的 Matterport 虚拟导览离线归档项目。支持多套房产共存，通过模型 ID 切换。

- **项目根目录**: `C:\Users\Larkl\Claude Project\matterport-dl\`

### 已归档房产

| 地址 | 模型 ID | 状态 |
|------|---------|------|
| 111 S Van Ness Ave, Los Angeles, CA | `Kfa2BHFroVn` | ✅ 已验证 |
| 528 Roslyn Rd, Kenilworth, IL | `Bmz3NvZfvx6` | ✅ 已验证 |
| 8951 St Ives Dr, Los Angeles, CA | `pVAbacdvn9h` | ✅ 已验证 |
| 535 Park Avenue_4AB, nyc | `DHNjTb7tAw5` | ✅ 已验证 |
| 16 Point Rd, Bellport, NY | `zTURYF5wgYr` | ✅ 已验证（私有模型）|
| 8111 Saint Martins Ln, Philadelphia, PA | `czVDUnbDue6` | ✅ 已验证（私有模型）|
| 240 Library Place, Princeton, NJ | `3Yzq4Bq71BP` | ✅ 已验证 |

## 项目状态

### 已完成
1. **Git clone** — 从 `https://github.com/rebane2001/matterport-dl.git` 克隆到项目根目录
2. **PR#196 修复已应用** — `matterport-dl.py` 从 `340-S-Plymouth-Blvd` 项目复制，包含：
   - `parseDynamicImports()` — 适配 Matterport 2026年1月 JS 打包格式变更（`name.key.js` → `name.js`）
   - `getPreloadJson()` — 提取 `window.MP_PREFETCHED_MODELDATA`
   - `getLatestPluginVersion()` — 插件版本策略验证
3. **离线服务修复已应用**（2026-03-26 三个 Bug）：
   - Bug 1: `do_GraphRequest()` 为未知 graph 操作返回 `{"data": {}}` 而非空响应
   - Bug 2: `init.js` 中 `.onStale` → `.onStal` 阻断过期回调
   - Bug 3: `packages-cwf-util.js` 中 `this.validUntil&&this.onStale` → `this.validUntil&&false` 防止永久阻塞
4. **依赖已安装** — `venv/` 目录，`curl-cffi==0.7.1`、`Pillow`、`aiofiles` 等

### 各模型资源统计

**Kfa2BHFroVn（111 S Van Ness Ave）**
- 全景瓦片：1553 张 | 3D 网格：791 个 `.glb` | DAM 网格：2 个
- 总资源：~132090 主要资源，~141730 总请求
- Draco/Basis WASM 解码器从 JuV9VNgJzob 模型复制

**Bmz3NvZfvx6（528 Roslyn Rd）**
- 全景瓦片：60338 张 JPG | 3D 网格：已下载
- 自动下载完成，含 Draco/Basis 解码器

**pVAbacdvn9h（8951 St Ives Dr, LA）**
- 全景瓦片：80486 张 JPG | 3D 网格：459 个 `.glb`
- 自动下载完成，含 Draco/Basis 解码器

**DHNjTb7tAw5（535 Park Avenue_4AB, nyc）**
- 全景瓦片：12360 张 JPG | 3D 网格：19 个 `.glb`
- 含 Draco/Basis 解码器
- **注意**：下载时 `showcase.modified.js` / `vendors-react.modified.js` 自动生成，但 `init.modified.js` 和 `packages-cwf-util.modified.js` 需手动补丁（Python 脚本：替换 `.onStale`→`.onStal`、`validUntil&&this.onStale`→`validUntil&&false`）

**zTURYF5wgYr（16 Point Rd, Bellport, NY）— 私有模型**
- 总下载资源：180875（Skipped 160590，Requested 20285，Success 20239）
- 生成裁剪图：84176 张 | 全景瓦片 + 3D 网格 + Draco/Basis 解码器齐全
- **必须先从浏览器抓 auth token 才能下载**（看下方"私有模型下载"章节）

**czVDUnbDue6（8111 Saint Martins Ln, Philadelphia, PA）— 私有模型**
- 全景瓦片：46332 张 JPG | 3D 网格：218 个 `.glb`
- 总下载资源：39402（Success 39291，100%）| 生成裁剪图：8036
- **必须先从浏览器抓 auth token 才能下载**（看下方"私有模型下载"章节）
- **带 defurnished view**（家具抹除/切换功能），触发踩坑 #6 overlay tile 漏下载；修复见 `showcase-latest-upgrade` 分支

**3Yzq4Bq71BP（240 Library Place, Princeton, NJ）**
- 全景瓦片：76907 张 JPG | 3D 网格：244 个 `.glb`
- Total fetches: 69230 | Success: 69082 (100%) | 生成裁剪图: 8928
- 公开模型，含 Draco/Basis 解码器

## Commands

### 启动本地离线服务（切换模型只需改 ID）
```bash
cd "C:\Users\Larkl\Claude Project\matterport-dl"

venv\Scripts\python.exe run.py Kfa2BHFroVn 127.0.0.1 8081   # 111 S Van Ness Ave, Los Angeles
venv\Scripts\python.exe run.py Bmz3NvZfvx6 127.0.0.1 8081   # 528 Roslyn Rd
venv\Scripts\python.exe run.py pVAbacdvn9h 127.0.0.1 8081   # 8951 St Ives Dr, LA
venv\Scripts\python.exe run.py DHNjTb7tAw5 127.0.0.1 8081   # 535 Park Avenue_4AB, nyc
venv\Scripts\python.exe run.py zTURYF5wgYr 127.0.0.1 8081   # 16 Point Rd, Bellport, NY
venv\Scripts\python.exe run.py czVDUnbDue6 127.0.0.1 8081   # 8111 Saint Martins Ln, Philadelphia, PA
venv\Scripts\python.exe run.py 3Yzq4Bq71BP 127.0.0.1 8081   # 240 Library Place, Princeton NJ
```
浏览器打开 `http://127.0.0.1:8081`

### 下载新房产
```bash
venv\Scripts\python.exe run.py "https://my.matterport.com/show/?ref=pn&m=模型ID&brand=0"
```
> **注意**: 必须加 `&brand=0` 参数，否则可能出现 "模型不可用"。
> 如果只需更新 JS/HTML/CSS（不重新下载大型资源），加 `--no-main-asset-download`。

### 私有模型下载（homes.com 等平台）
某些模型是 Matterport 私有/licensed 的（`"error_code":"not.found"` 直接访问），必须带 `auth` token + `applicationKey` 才能下载。

**步骤**：
1. **从浏览器抓 token** — homes.com 页面点 "Matterport 3D Tour" 按钮，在 iframe `src` 里读取：
   - `m=<model_id>`
   - `auth=Bearer%20<token>` （24 小时有效）
   - `applicationKey=249b3f19a520439a9f49173073cb49f4`（homes.com 固定 key）

2. **下载命令**（需要修改版 `matterport-dl.py`，已注入以下环境变量支持）：
   ```bash
   cd "C:\Users\Larkl\Claude Project\matterport-dl"
   set MATTERPORT_AUTH_TOKEN=<token 裸值，不带 "Bearer ">
   set MATTERPORT_APPLICATION_KEY=249b3f19a520439a9f49173073cb49f4
   set MATTERPORT_EXTRA_PARAMS=auth=Bearer%20<token>&applicationKey=249b3f19a520439a9f49173073cb49f4&brand=0
   venv\Scripts\python.exe run.py "https://my.matterport.com/show/?m=<model_id>"
   ```

**脚本改动记录**（已应用于 `matterport-dl.py`）：
- `SetupSession()`：支持 `MATTERPORT_AUTH_TOKEN`（OAuth Bearer 注入为 `Authorization` header）、`MATTERPORT_APPLICATION_KEY`、`MATTERPORT_REFERER`
- `downloadCapture()` line 972：URL 构造支持 `MATTERPORT_EXTRA_PARAMS` 环境变量追加
- `downloadFile()` line 302：MAIN 请求容忍 HTTP 404 当响应含 `MP_PREFETCHED_MODELDATA`

**速度注意**：私有模型 CDN 对 tileset 阶段会限速（~1.3 it/s），但后续全景瓦片阶段恢复正常（~23 it/s）。全程约 1-2 小时。

### 如何找到模型 ID
在 homes.com 等网站的页面 HTML 中搜索 `api/v1/player/models/[ID]/thumb`，即可提取模型 ID。

## 关键文件
| 文件 | 说明 |
|------|------|
| `matterport-dl.py` | 主脚本（含 PR#196 + 离线修复） |
| `JSNetProxy.js` | 浏览器端网络拦截代理 |
| `run.py` | 推荐入口，自动处理 venv + 依赖 |
| `downloads/[模型ID]/` | 已下载的模型资源 |
| `downloads/[模型ID]/js/*.modified.js` | 离线补丁文件（服务器优先加载） |

## 参考项目
- **340-S-Plymouth-Blvd** (`C:\Users\Larkl\PycharmProjects\340-S-Plymouth-Blvd\`) — 原始开发项目，包含已验证的补丁和模型 `JuV9VNgJzob`

## 踩坑记录（已解决）
1. **expiring-resource 冻结** — 模型加载但完全无法交互。根因：JS 检测资源过期后 `await` 刷新永远阻塞。修复：3 个 `.modified.js` 补丁
2. **Draco/Basis 解码器缺失** — 所有 `.glb` 3D 网格瓦片无法渲染。修复：从旧模型复制 WASM 文件
3. **全景瓦片缺失** — `~/tiles/` 目录不存在，无法进入房间。修复：增量重新下载
4. **JS 版本不匹配** — Matterport 从 26.2.4 升级到 26.3.4，增量下载跳过旧文件导致 webpack chunk 不匹配。修复：清除所有 JS/HTML/CSS 文件后重新下载（加 `--no-main-asset-download` + `&brand=0`）
5. **模型 404** — 某些模型已下线无法下载（如 `X9wJ8LP4LRA`），需先确认在线可访问
6. **Overlay 瓦片缺失（defurnished/家具切换模型专属）** — 带 defurnished view 的模型（`czVDUnbDue6` 是首个案例）点击部分圆盘时转圈卡死或进入后全图模糊。
   - **触发判定（简单直接）**：**模型没有 defurnished view 就不会遇到此 bug**。判断方式：下载日志里有无警告 `#### WARNING: Model has defurnished view ...`（或 `grep "defurnished view" downloads/<ID>/run_report.log`）。已归档 7 个模型中只 `czVDUnbDue6` 触发；另 6 个（Kfa2BHFroVn / Bmz3NvZfvx6 / pVAbacdvn9h / DHNjTb7tAw5 / zTURYF5wgYr / 3Yzq4Bq71BP）均无此层，天然免疫。
   - **根因**：`matterport-dl.py` 只从 `graph_GetShowcaseSweeps.json` 的 `tileUrlTemplate` 抓**主路径**瓦片（`assets/~/tiles/<sweep>/<res>_face_*.jpg`），**不解析** `index.html` 里 `window.MP_PREFETCHED_MODELDATA` 中的 **overlay 路径** tile template（`assets/overlay/<overlay_uuid>/~/tiles/<sweep>/<res>_face_*.jpg`）。这些 overlay tile 是 Matterport 家具抹除/切换功能用的二次渲染纹理，只有部分 sweep 才需要（非 defurnished 模型根本没这层）。
   - **修复**：已在 `showcase-latest-upgrade` 分支新增 `downloadOverlaySweeps()` 函数（`matterport-dl.py`），从 `index.html` 的 `MP_PREFETCHED_MODELDATA` 提取 overlay `tileUrlTemplate`（含**目录级签名** `k=models/<id>/assets/overlay/<ol>`，一签管整个 overlay 子树），按主路径同样逻辑枚举 `res × face × (x,y)` 下载。非 defurnished 模型 regex 零匹配 → 无新开销。
   - **诊断线索**：`server.log` 出现大量 404 路径形如 `.../assets/overlay/<32位 hex>/~/tiles/<sweep>/<res>_face_*.jpg`。
   - **czVDUnbDue6 实战数据**：63 个 sweep 中 9 个用了 overlay `468caae3ed444fabb4b1bce1d272f1ab`，共补回 4590 个 tile（512/1k/2k/4k 四档全齐），全都能用同一个目录级签名批量拉取。

## 已知问题（无需修复）
- 插件 404（compass、minimap 等）— 预期行为，不影响核心导览
- 字体/图片 404（roboto、nav_help 图片等）— 视觉缺失但不影响交互
- 大模型首次加载可能需要几分钟（3D 网格解码），后续浏览器缓存后会快很多
- 本地服务端口：**8081**（非默认 8080）
- 下载结束时 `Curlm already closed` 警告 — curl-cffi 清理时的无害警告，可忽略
