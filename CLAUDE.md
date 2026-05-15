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
| 8111 Saint Martins Ln, Philadelphia, PA | `czVDUnbDue6` | ✅ 已验证（私有模型）|
| 240 Library Place, Princeton, NJ | `3Yzq4Bq71BP` | ✅ 已验证 |
| 2222 Wyoming Ave NW, Washington, DC | `KNSj7socXtz` | ✅ 已验证（私有模型，Showcase 26.4.5 首例）|
| 2388 W Lake of the Isles Pkwy, Minneapolis, MN | `MjDCcEUapar` | ✅ 已验证（MLS 公开模型，免 token，Showcase 26.4.5）|
| 1 W 72nd St Unit 84, New York, NY | `utaPFuhkSB5` | ✅ 已验证（私有模型，Showcase 26.5.1 首例，overlay 补丁首次自动生效）|
| 136 Crest Rd, Ridgewood, NJ | `PUTBuppB2bC` | ✅ 已验证（无 auth 无 key URL 模式但仍需 applicationKey 环境变量）|
| 173 Macdougal St Unit 5, New York, NY | `tVkP3reU4q6` | ✅ 已验证（私有模型，Showcase 26.5.1，in-place overlay 自动救 9180 tile）|
| 91 Algonquin Rd, Fairfield, CT — 主屋 | `mUT8ygkB5Tc` | ✅ 已验证（私有模型，Showcase **26.5.2** 首例，159 sweep / 29 GB） |
| 91 Algonquin Rd, Fairfield, CT — 附属 1 | `YuuxsvSH9S3` | ✅ 已验证（私有，13 sweep / 2.6 GB，同房产第 2 个 model）|
| 91 Algonquin Rd, Fairfield, CT — 附属 2 | `cw4ASygjtAC` | ✅ 已验证（私有，1.5 GB，同房产第 3 个 model，无 overlay）|
| 91 Algonquin Rd, Fairfield, CT — 鸟瞰 | `91-algonquin-aerial-splat/` | ✅ 已验证（**Gaussian Splatting 航拍** —— CoStar Matterport Exterior，57 MB） |
| 340 S Plymouth Blvd, Los Angeles, CA | `JuV9VNgJzob` | ❌ 未验证（手动添加，需要验证）|
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

**czVDUnbDue6（8111 Saint Martins Ln, Philadelphia, PA）— 私有模型**
- 全景瓦片：46332 张 JPG | 3D 网格：218 个 `.glb`
- 总下载资源：39402（Success 39291，100%）| 生成裁剪图：8036
- **必须先从浏览器抓 auth token 才能下载**（看下方"私有模型下载"章节）
- **带 defurnished view**（家具抹除/切换功能），触发踩坑 #5 overlay tile 漏下载；修复见 `showcase-latest-upgrade` 分支

**3Yzq4Bq71BP（240 Library Place, Princeton, NJ）**
- 全景瓦片：76907 张 JPG | 3D 网格：244 个 `.glb`
- Total fetches: 69230 | Success: 69082 (100%) | 生成裁剪图: 8928
- 公开模型，含 Draco/Basis 解码器

**KNSj7socXtz（2222 Wyoming Ave NW, Washington, DC）— 私有模型**
- 总下载资源：59154（Success 55611，94%）| 生成裁剪图：7840 | 18 GB
- **Showcase 26.4.5_webgl-632 首例 + Three.js 0.184.0**（之前都是 26.4.3 / 0.176.0）
- **必须先从浏览器抓 auth token 才能下载**（看下方"私有模型下载"章节）
- **defurnished 模型**（companion id `0m52t3rzmeqydn21h9ezerzid`，未下载）—— 但实现机制与 czVDUnbDue6 不同：**整模型替换**而非 overlay 层叠加，所以踩坑 #5 不适用，没漏 overlay tile
- 触发了 3 个新坑（详见踩坑 #6/#7/#8）：插件下载 400 致命、`/api/mp/accounts/graph` 端点、新 image/font 资源缺失

**MjDCcEUapar（2388 W Lake of the Isles Pkwy, Minneapolis, MN）— MLS 公开模型**
- 总下载资源：42368（Success 42174，99.4%，21 个 403 都是 Matterport CDN 死链） | 生成裁剪图：1861 | 12 GB
- Showcase 26.4.5_webgl-632（与 KNSj7socXtz 同版本）
- **不需要 auth token** —— iframe URL 含 `mls=1` 但无 `auth=Bearer`，只用 applicationKey + brand=0 + mls=1 即可（首个免 token 的 homes.com 模型，看下方"MLS 公开模型下载"小节）
- 无 defurnished view，无 overlay 层
- 26.4.5 新静态资源里 5 个被 matterport-dl 自动下了（scope/vert_arrows/surface_grid/2 个 mp-font），4 个走踩坑 #8 流程手动 curl 补回（atlas/logo-white/logo-white-r/nav_help_gesture）

**utaPFuhkSB5（1 W 72nd St Unit 84, New York, NY）— 私有模型**
- 总下载资源：138170（Success 85483，62%，Failed404 52532 全是 overlay 试探正常 404）| 生成裁剪图：5588 | 23 GB
- **Showcase 26.5.1_webgl-654 首例**（之前是 26.4.5，Matterport 又升级了）
- **必须从浏览器抓 auth token**（私有模型）
- **defurnished 模型 + in-place overlay 风格**（companion id `ugip3puc3ck781ushhnwzi0ed`）—— **首次让我们的 overlay 补丁在新下载时自动生效**：日志显示 `Found 1 overlay layer(s); queuing overlay tile downloads for 132 sweeps`，自动救出 14790 个 overlay tile（czVDUnbDue6 当时手动 curl 4590）
- 26.5.1 新静态资源比 26.4.5 多缺：4 个 webgl-vendors（draco_decoder.wasm + basis_transcoder.wasm + 2 个 js）+ 9 个 26.4.5 同款 + 4 个 cursors + 4 个 css + 26 个 IBM Plex Sans 字体变体（cosmetic 跳过）。手动 curl 18/19 成功（`css/scene.css` 仍是 403 死链，与早期同步）

**PUTBuppB2bC（136 Crest Rd, Ridgewood, NJ）— 无 auth 无 key URL 模式**
- 总下载资源：36049（Success 35988（合并两次跑），99.8%）| 生成裁剪图：1504 | 9.6 GB
- Showcase 26.5.1_webgl-654（同 utaPFuhkSB5）
- **第三种 URL 模式**：iframe URL 既无 `auth=Bearer` 也无 `mls=1` 也无 `applicationKey`（只 `m=...&play=1&qs=1`）—— **但 matterport-dl 仍必须设 `MATTERPORT_APPLICATION_KEY` 环境变量**，否则 main CDN 给所有资产请求 400（看下方"无 auth 无 key URL 模式"小节）
- **defurnished 模型 + 独立模型风格**（companion id `34mhcckaafix5hhezer547qta`，未下载）—— 不需 overlay 补丁，与 KNSj7socXtz 同款

**tVkP3reU4q6（173 Macdougal St Unit 5, New York, NY）— 私有模型**
- 总下载资源：73722（Success 47128，64%；Failed404 26523 全是 overlay 试探正常 404）| 生成裁剪图：5072 | 14 GB
- Showcase 26.5.1_webgl-654（同 utaPFuhkSB5/PUTBuppB2bC）
- **必须从浏览器抓 auth token**（私有模型，URL 含 `auth=Bearer` + `qs=1`，无 applicationKey 但脚本要加）
- **defurnished 模型 + in-place overlay 风格**（companion id `uyfiabwsc0md34gi1q0cawcmc`，未下载）—— **overlay 补丁第二次自动生效**：日志显示 `Found 1 overlay layer(s); queuing overlay tile downloads for 70 sweeps`，自动救 9180 个 overlay tile
- 26.5.1 必需文件大部分齐全（cursors / fonts / 4 个 webgl-vendors 都自动下了），仅 atlas.png + logo-white.svg + 2 个 logo svg 需手动 curl（4 个，比 utaPFuhkSB5 当时少很多）

**91 Algonquin Rd, Fairfield, CT — 多模型房产（首例，3 个 Matterport + 1 个航拍 splat）**

> 这套 homes.com 房产用了**多个独立 Matterport model + 1 个 Gaussian Splatting 鸟瞰图**。Player 内部 thumbnail 切换不改 iframe src，是 Matterport SDK 用 JS 在同一 iframe 内换 model。每个 model 独立 auth token。判断方式：在 homes.com 页面 JS 跑 `for (const m of document.documentElement.outerHTML.matchAll(/[\"']m[\"']\s*:\s*[\"']([A-Za-z0-9]{11})[\"']/g)) ids.add(m[1]);` 列出所有 model ID。3 个建筑 + 鸟瞰：

- **`mUT8ygkB5Tc`（主屋）** — Showcase **26.5.2_webgl-687_g2028078226 首例**（之前都是 26.4.5 / 26.5.1，Matterport 又升级了）。159 sweep / 29 GB / 19848 crops / 81090 main tasks。defurnished + in-place overlay → 6630 overlay tile 自动救出。Three.js 仍是 0.184.0 路径。
- **`YuuxsvSH9S3`（附属建筑 1）** — 13 sweep / 2.6 GB。同样 in-place overlay defurnished。
- **`cw4ASygjtAC`（附属建筑 2）** — 5142 任务 / 99% 成功 / 1.5 GB。无 overlay layer，最简结构。
- **航拍 Gaussian Splatting** — 见下方专门章节。

**26.5.2 新缺失文件**（matterport-dl 又漏了一批）：`scope.svg` `atlas.png` `logo-white.svg` `mp-font.woff/.woff2` `cursors/grab.png` 等 16 个，都从 `static.matterport.com/showcase/26.5.2_webgl-687-g2028078226/` 直接 curl 即可（无 origin/referer 限制）。

**91 Algonquin Rd 鸟瞰图 — Gaussian Splatting / CoStar Matterport Exterior**
- 总大小 **57 MB / 18 文件**（独立目录 `downloads/91-algonquin-aerial-splat/`，**不在 Matterport model 目录下**）
- **完全不同于 Matterport Showcase** —— 用 3D Gaussian Splatting 技术（2023 新算法，渲染 outdoor/aerial 比 photogrammetry mesh 自然得多）
- **域名/路径**：`https://www.homes.com/matterport-exterior-viewer/?t=<token>&a=<asset_id>&e=<entity_id>&edn=<edn>&at=131&mi=true&mc=true`，资产路径 `https://www.homes.com/matterport-community/<UUID>/{means_l,means_u,quats,scales,sh0}.webp`
- **5 个 splat 数据 webp**（共 50 MB）：
  - `means_l.webp` / `means_u.webp` — XYZ 位置（high/low 字节拆分编码精度）
  - `quats.webp` — 旋转四元数
  - `scales.webp` — 各向异性缩放
  - `sh0.webp` — 球谐光照系数（基色，仅 0 阶 → 不带 view-dependent 反射，简化版）
- **下载方式见专门章节"Gaussian Splatting 航拍场景下载"**

## Commands

### 启动本地离线服务（切换模型只需改 ID）
```bash
cd "C:\Users\Larkl\Claude Project\matterport-dl"

venv\Scripts\python.exe run.py Kfa2BHFroVn 127.0.0.1 8081   # 111 S Van Ness Ave, Los Angeles
venv\Scripts\python.exe run.py Bmz3NvZfvx6 127.0.0.1 8081   # 528 Roslyn Rd
venv\Scripts\python.exe run.py pVAbacdvn9h 127.0.0.1 8081   # 8951 St Ives Dr, LA
venv\Scripts\python.exe run.py DHNjTb7tAw5 127.0.0.1 8081   # 535 Park Avenue_4AB, nyc
venv\Scripts\python.exe run.py czVDUnbDue6 127.0.0.1 8081   # 8111 Saint Martins Ln, Philadelphia, PA
venv\Scripts\python.exe run.py 3Yzq4Bq71BP 127.0.0.1 8081   # 240 Library Place, Princeton NJ
venv\Scripts\python.exe run.py KNSj7socXtz 127.0.0.1 8081   # 2222 Wyoming Ave NW, Washington DC
venv\Scripts\python.exe run.py MjDCcEUapar 127.0.0.1 8081   # 2388 W Lake of the Isles Pkwy, Minneapolis MN
venv\Scripts\python.exe run.py utaPFuhkSB5 127.0.0.1 8081   # 1 W 72nd St Unit 84, NYC
venv\Scripts\python.exe run.py PUTBuppB2bC 127.0.0.1 8081   # 136 Crest Rd, Ridgewood NJ
venv\Scripts\python.exe run.py tVkP3reU4q6 127.0.0.1 8081   # 173 Macdougal St Unit 5, NYC
venv\Scripts\python.exe run.py mUT8ygkB5Tc 127.0.0.1 8081   # 91 Algonquin Rd 主屋 (Fairfield CT)
venv\Scripts\python.exe run.py YuuxsvSH9S3 127.0.0.1 8081   # 91 Algonquin Rd 附属建筑 1
venv\Scripts\python.exe run.py cw4ASygjtAC 127.0.0.1 8081   # 91 Algonquin Rd 附属建筑 2
venv\Scripts\python.exe run.py JuV9VNgJzob 127.0.0.1 8081   # 340 S Plymouth Blvd, Los Angeles, CA 

```
浏览器打开 `http://127.0.0.1:8081`

### 启动本地 Gaussian Splatting viewer 服务（91 Algonquin 鸟瞰）

跟 Matterport Showcase 的 8081 不同（避免冲突），用普通 Python 静态 HTTP server 起在 **8083**：

```cmd
cd "C:\Users\Larkl\Claude Project\matterport-dl\downloads\91-algonquin-aerial-splat"
python -m http.server 8083 --bind 127.0.0.1
```

或用项目 venv 的 Python：
```cmd
cd "C:\Users\Larkl\Claude Project\matterport-dl"
venv\Scripts\python.exe -m http.server 8083 --bind 127.0.0.1 --directory downloads\91-algonquin-aerial-splat
```

浏览器打开 `http://127.0.0.1:8083/`，**直接进 viewer，无需任何 query 参数**（HTML 里 `window.sse.contents` 已经被 ROT-3 重写指向本地路径）。

**接收 splat 数据的临时 receiver 脚本**（用于重新抓数据时）：
```python
import http.server, os, sys
SAVE = r'E:\matterport-dl\downloads\<dir>\matterport-community\<UUID>'
os.makedirs(SAVE, exist_ok=True)
class H(http.server.BaseHTTPRequestHandler):
    def do_OPTIONS(self):
        self.send_response(204); self.send_header('Access-Control-Allow-Origin','*')
        self.send_header('Access-Control-Allow-Methods','POST,OPTIONS')
        self.send_header('Access-Control-Allow-Headers','*'); self.end_headers()
    def do_POST(self):
        n = int(self.headers.get('Content-Length',0)); data = self.rfile.read(n)
        with open(os.path.join(SAVE, self.path.lstrip('/')), 'wb') as f: f.write(data)
        self.send_response(200); self.send_header('Access-Control-Allow-Origin','*'); self.end_headers()
        self.wfile.write(b'ok'); sys.stderr.write(f'Saved {self.path} ({n}B)\n')
    def log_message(self,*a): pass
http.server.HTTPServer(('127.0.0.1', 8082), H).serve_forever()
```

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

### MLS 公开模型下载（homes.com 列表里的房源，免 token）

部分 homes.com 房源是 MLS 公开数据，iframe URL 里**没有** `auth=Bearer` 参数，只带 `mls=1` 标记，免认证可下载。判断方式：抓 iframe `src` 看有没有 `auth=Bearer`。

例（`MjDCcEUapar` = 2388 W Lake of the Isles Pkwy）：
```
https://my.matterport.com/show/?m=MjDCcEUapar&brand=0&mls=1&play=1&search=0&vr=0&applicationKey=249b3f19a520439a9f49173073cb49f4
```

**下载命令**（环境变量不要设 `MATTERPORT_AUTH_TOKEN`）：
```bash
cd "C:\Users\Larkl\Claude Project\matterport-dl"
set MATTERPORT_APPLICATION_KEY=249b3f19a520439a9f49173073cb49f4
set MATTERPORT_EXTRA_PARAMS=applicationKey=249b3f19a520439a9f49173073cb49f4&brand=0&mls=1
venv\Scripts\python.exe run.py "https://my.matterport.com/show/?m=<model_id>"
```

**速度优势**：MLS 公开模型 CDN 不限速，全程 ~10-30 分钟（看大小），明显比私有模型快。

### 无 auth 无 key URL 模式（部分 homes.com 房源，仍需 applicationKey）

部分 homes.com 房源的 iframe `src` **既无 `auth=Bearer` 也无 `mls=1` 也无 `applicationKey`**，只有 `m=<id>&play=1&search=0&vr=0&qs=1` 这种最小参数。容易误以为完全免认证 —— **但实际上 matterport-dl 仍必须设 `MATTERPORT_APPLICATION_KEY` 环境变量**，否则 Matterport main 资产 CDN 对所有请求返回 HTTP 400（FailedUnknown），最终模型只下了 19% / 几 GB（小得离谱）。

例（`PUTBuppB2bC` = 136 Crest Rd, Ridgewood NJ）：
```
https://my.matterport.com/show/?m=PUTBuppB2bC&play=1&search=0&vr=0&qs=1
```

**下载命令**（不设 `MATTERPORT_AUTH_TOKEN`，但要设 key）：
```bash
cd "C:\Users\Larkl\Claude Project\matterport-dl"
set MATTERPORT_APPLICATION_KEY=249b3f19a520439a9f49173073cb49f4
set MATTERPORT_EXTRA_PARAMS=applicationKey=249b3f19a520439a9f49173073cb49f4&brand=0&qs=1
venv\Scripts\python.exe run.py "https://my.matterport.com/show/?m=<model_id>"
```

**诊断**：如果 Done 报告显示 `FailedUnknown` 数 > 几千 + 模型大小不到 5 GB，多半是没加 applicationKey，重跑就行（增量已下的会跳过）。

### Gaussian Splatting 航拍场景下载（CoStar Matterport Exterior，homes.com 的 "3D Exterior" tab）

部分 homes.com 房产除了室内 Matterport 还附带 **CoStar Matterport Exterior** 鸟瞰 viewer（用 3D Gaussian Splatting 而非 Matterport mesh）。这套**完全不走 matterport-dl.py**，要单独抓。

**判断**：homes.com 页面 tab 栏看到 "3D Exterior" 或 dollhouse 不够看到所有建筑，且页面有 video 元素（30s 1080p 预览循环）。

**底层 URL 模式**：homes.com property 页面其实只显示视频预览，真正的 viewer 是独立 URL：
```
https://www.homes.com/matterport-exterior-viewer/?t=<obfs_token>&a=<asset_id>&e=<entity_id>&edn=<edn>&at=131&mi=true&mc=true
```
资产路径：`https://www.homes.com/matterport-community/<UUID>/<file>.webp`

**抓取流程**（举例 `91 Algonquin Rd`）：

1. **找到 viewer URL**：homes.com 页面 F12 Console 跑：
   ```js
   [...document.querySelectorAll('iframe')].map(f=>f.src).filter(s=>s.includes('matterport-exterior-viewer'))[0]
   ```
2. **打开该 viewer URL**，F12 → Console 跑：
   ```js
   performance.getEntriesByType('resource').filter(e=>e.name.includes('matterport-community')).map(e=>new URL(e.name).pathname)
   ```
   会列出所有 splat 资产路径（一般 5 个：`means_l.webp` / `means_u.webp` / `quats.webp` / `scales.webp` / `sh0.webp`）。同时记下 asset UUID（`/matterport-community/<UUID>/...`）。
3. **直 curl 会被 403**（需要 cookie），所以用本地 Python POST 桥接：
   - 起 receiver server（端口 8082）见下方"启动本地 Gaussian Splatting viewer 服务"小节的代码模板
   - 在 viewer 页 Console 跑：
     ```js
     (async () => {
       const ASSET = '<UUID>';
       const files = ['means_l.webp', 'means_u.webp', 'quats.webp', 'scales.webp', 'sh0.webp'];
       for (const f of files) {
         const r = await fetch('/matterport-community/' + ASSET + '/' + f);
         const blob = await r.blob();
         await fetch('http://127.0.0.1:8082/' + f, {method: 'POST', body: blob});
       }
     })();
     ```
   - 同样手法抓 viewer 静态资产（`/css/*.css`、`/js/*.js`、`/fonts/*.woff2`、`/index.html`、`/assets/images/pin-selected.png`、`/favicon.ico`）
4. **修 HTML 让它走本地路径**：viewer HTML 里 `window.sse.contents` 是 ROT-3（凯撒+3 字符位移）加密的 JSON，含 5 个 splat 文件的远程 URL。脚本：
   ```python
   def rot3(s, shift): return ''.join(chr(ord(c)+shift) for c in s)
   # decode → str.replace(homes.com → 127.0.0.1:8083) → re-encode
   ```
   **测过的对照**：`kwwsv=22zzz1krphv1frp` 解密就是 `https://www.homes.com`。
5. **本地服务**：用普通 `python -m http.server 8083` 起静态 server 即可

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
4. **模型 404** — 某些模型已下线无法下载（如 `X9wJ8LP4LRA`），需先确认在线可访问
5. **Overlay 瓦片缺失（defurnished/家具切换模型专属，czVDUnbDue6 风格）** — 带 defurnished view 的模型点击部分圆盘时转圈卡死或进入后全图模糊。
   - **重要前提**：defurnished 有**两种实现机制**，本坑只覆盖 **in-place overlay 叠加**（czVDUnbDue6 风格）；不覆盖**整模型替换**风格（KNSj7socXtz 风格 —— defurnished 是另一个独立模型 ID，单独下载即可，无 overlay tile 需求）。判断：grep `index.html` 是否含 `assets/overlay/<uuid>/~/tiles/...` 路径模板，有就是 in-place 风格。
   - **触发判定**：下载日志里有 `#### WARNING: Model has defurnished view ...` + index.html 含 overlay tile 模板。已归档 8 个模型中只 czVDUnbDue6 触发。
   - **根因**：`matterport-dl.py` 只从 `graph_GetShowcaseSweeps.json` 的 `tileUrlTemplate` 抓**主路径**瓦片（`assets/~/tiles/<sweep>/<res>_face_*.jpg`），**不解析** `index.html` 里 `window.MP_PREFETCHED_MODELDATA` 中的 **overlay 路径** tile template（`assets/overlay/<overlay_uuid>/~/tiles/<sweep>/<res>_face_*.jpg`）。
   - **修复**：已在 `showcase-latest-upgrade` 分支新增 `downloadOverlaySweeps()` 函数（`matterport-dl.py`），从 `index.html` 的 `MP_PREFETCHED_MODELDATA` 提取 overlay `tileUrlTemplate`（含**目录级签名** `k=models/<id>/assets/overlay/<ol>`，一签管整个 overlay 子树），按主路径同样逻辑枚举 `res × face × (x,y)` 下载。非 overlay 模型 regex 零匹配 → 无新开销。
   - **诊断线索**：`server.log` 出现大量 404 路径形如 `.../assets/overlay/<32位 hex>/~/tiles/<sweep>/<res>_face_*.jpg`。
   - **czVDUnbDue6 实战数据**：63 个 sweep 中 9 个用了 overlay `468caae3ed444fabb4b1bce1d272f1ab`，共补回 4590 个 tile（512/1k/2k/4k 四档全齐）。

6. **Plugin 400 致命（Showcase 26.4.5+ 暴露）** — 下载阶段 `Downloading plugins...` 报 `HTTP Error 400: salesperson-info/1.0.21/plugin.json`，整个 `matterport-dl.py` 抛异常退出。
   - **根因**：`downloadPlugins()` 调用 `downloadFile("PLUGIN", True, ...)`，遇到 Matterport CDN 对老插件版本返回 400 时直接 raise Exception，下载流程中断（且这一步在最后阶段，前面 1.7+ GB 下载白瞎）。
   - **修复**：`downloadPlugins()` 内的 `downloadFile` 调用包 try/except，失败仅 WARN 日志记录后 continue（插件本身浏览器侧 JS 自带容错，缺失不致命）。已应用在 `showcase-latest-upgrade` 分支。

7. **`/api/mp/accounts/graph` 端点未处理（Showcase 26.4.5+）** — 模型加载到 0.5s 第一帧后立刻雪崩 `NetworkError: A network error occurred`，浏览器显示"模型不可用"全黑。
   - **根因**：26.4.5 新增 `/api/mp/accounts/graph` POST 端点（账号/auth 类查询），原本的本地 server `do_POST()` 只识别 `/api/mp/models/graph`，新端点 404 → JS 抛 NetworkError → loadApplication 链式崩溃。
   - **修复**：`do_POST()` 同时识别 `/api/mp/models/graph` 和 `/api/mp/accounts/graph`，都路由到 `do_GraphRequest()`（返回 `{"data": {}}` 兜底）。已应用在 `showcase-latest-upgrade` 分支。

8. **新静态资源 404 致 GLTFLoader 同步抛 NetworkError（Showcase 26.4.5 / 26.5.1+）** — 修完 #7 后又崩溃，触发链是模型 0.6s 加载后 `Failed to load .../images/scope.svg` 等导致 GLTFLoader 抛 NetworkError → loadApplication 崩。
   - **根因**：26.4.5 / 26.5.1 引入了 `matterport-dl.py` 静态资源清单（`image_files`、`font_files`）里没列出的新文件。
   - **必修文件清单**（缺则 NetworkError 崩）：`images/scope.svg` `images/vert_arrows.png` `images/surface_grid_planar_256.png` `images/atlas.png` `images/logo-white.svg` `images/logo-white-r.svg` `images/nav_help_gesture_drag_two_finger_rotate.svg` `fonts/mp-font.woff` `fonts/mp-font.woff2`。**26.5.1 还多缺**：`webgl-vendors/three/0.184.0/libs/draco/gltf/draco_decoder.wasm` + `webgl-vendors/three/0.184.0/libs/basis/basis_transcoder.wasm` + 4 个 cursors（`cursors/grab.png` `grabbing.png` `zoom-in.png` `zoom-out.png`）+ 3 个 css（`unsupported_browser.css` `ws-autotours.css` `ws-blur.css`）。
   - **修复（手动）**：从 `static.matterport.com/showcase/<showcase版本>/<path>` 或 `static.matterport.com/webgl-vendors/three/<three版本>/libs/<path>` 直接 curl 下载。**这些路径无 origin/referer 限制**，curl 即可。
   - **永远拿不到的死链**（持续 403，无救，纯 cosmetic）：`css/scene.css`、`images/matterport-logo.svg`、`matterport-logo-light.svg`、`matterport-app-icon.svg`。
   - **后续**：应该把这些文件加进 `matterport-dl.py` 的 `image_files` / `font_files` 列表（line 539）。当前手动 curl 即可，已下 5 个 26.4.5/26.5.1 模型都用这流程修过。

9. **第三种 URL 模式仍需 applicationKey** — 部分 homes.com 房源 iframe URL 极简（无 `auth=Bearer` / 无 `mls=1` / 无 `applicationKey`），只有 `m=<id>&play=1&qs=1`。容易误以为可裸跑下载 —— 但实际 matterport-dl **必须设 `MATTERPORT_APPLICATION_KEY=249b3f19a520439a9f49173073cb49f4`** 否则 Matterport main CDN 给主资产请求全 400（统计 FailedUnknown 几万、最终模型 < 5 GB 太小）。
   - **诊断**：Done 报告 `FailedUnknown` > 几千 + 模型异常小（<5 GB）= 没加 applicationKey。
   - **修复**：补 applicationKey 重跑（增量跳过已下的，几分钟搞定剩余主资产）。
   - **首例**：PUTBuppB2bC（136 Crest Rd, Ridgewood NJ）。详见 Commands 章节 "无 auth 无 key URL 模式" 小节。

## 已知问题（无需修复）
- 插件 404（compass、minimap 等）— 预期行为，不影响核心导览
- 字体/图片 404（roboto、nav_help 图片等）— 视觉缺失但不影响交互
- 大模型首次加载可能需要几分钟（3D 网格解码），后续浏览器缓存后会快很多
- 本地服务端口：**8081**（非默认 8080）
- 下载结束时 `Curlm already closed` 警告 — curl-cffi 清理时的无害警告，可忽略
