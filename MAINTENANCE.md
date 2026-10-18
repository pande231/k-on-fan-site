# 维护手册 — 轻音少女粉丝站

> **不想碰代码？** 直接双击 `可视化编辑器.html`，或看 `新手教程.md`。
> 本手册是给「要改数据结构、加页面、调逻辑」的人看的。

面向「以后想自己改点东西」的场景。按 **我想改 X → 改哪里** 组织，不需要通读全部代码。

---

## 0. 先记住这几件事

- **只有一个文件**：`轻音少女粉丝站.html`（约 5000 行 / 443 KB，HTML 外壳 + 内联 JS 全在里面）
- **没有构建步骤**：改完保存 → 浏览器刷新（建议 `Ctrl+F5` 强刷，避免缓存）
- **所有页面都是 JS 渲染的**：`<main id="view">` 在 HTML 里是空的，路由走 hash
  （`#/music`、`#/gallery/wall`、`#/character/yui`、`#/goods`）
- **编辑器请存成 UTF-8（无 BOM）**。存成 GBK 会让所有日文/中文变成乱码

代码被分成 13 个编号区块，用 `Ctrl+F` 搜 `⑨` 这种圈码就能跳过去：

| 区块 | 内容 | 起始行约 |
|---|---|---|
| ① | `MEDIA` 图片/音频统一入口 | 506 |
| ①-b | `CHAR_SRC` 角色立绘原图 | 555 |
| ② | `ICONS` 内联 SVG 图标 | 565 |
| — | `SCENE_MAP` 名场面关键词配图 | 617 |
| — | `EP_COVER` 每集封面映射 | 647 |
| ③ | `CHARACTERS` 角色数据 | 725 |
| ④ | `SONGS` 曲目数据 | 1132 |
| — | `MUSIC_CATS` 音乐分类 | 1343 |
| ⑤ | `EPISODES` 分集数据 | 1356 |
| ⑥ | `GOODS_DATA` 周边数据 | 1562 |
| ⑦ | 渲染函数（各页模板） | 1609 |
| — | `GALLERY_META` 图库逐图标注 | 2983 |
| — | `GALLERY_CATS` 图库分类 | 3049 |
| ⑧ | `Router` / `PostRender` 路由 | 3415 |
| ⑨ | `PREVIEWS` 试听 + `Player` 播放器 | 3879 |
| ⑩ | `Playlists` 歌单/收藏 + `LyricStore` | 4473 |
| ⑪ | `GAMES` 趣味互动 | 4685 |
| ⑫ | 社区 / API 层（`KON_API` / `ACHV` / `DAILY` / `WALL`） | 4949 |
| ⑬ | `init()` 启动 | 5431 |

> 行号会随编辑漂移，`Ctrl+F` 搜圈码最可靠；改动后请顺手更新本表。

页面主题色在 `<head>`：Tailwind 配置约 **15 行**，CSS 变量 `:root` 约 **52 行**。

---

## 1. 速查表

| 我想…… | 改哪里 |
|---|---|
| 换/加一张图片 | 图片丢进 `assets/`，路径填进 `MEDIA`（512 行） |
| 加一首歌 | `SONGS` 数组（1150 行），用 `S(...)` 助手 |
| 给歌配真实音源 | 加进 `PREVIEWS`（3866 行）或填 `song.audio` |
| 加图库图片 | `MEDIA.gallery_xxx` **+ 必须**补 `GALLERY_META`（2980 行） |
| 换角色头像 | 覆盖 `assets/char_<id>_face.jpg` |
| 换角色立绘 | `CHAR_SRC`（558 行） |
| 加周边条目 | `GOODS_DATA`（1571 行） |
| 换周边分类代表图 | 覆盖 `assets/goods_<分类>.jpg` |
| 改主题配色 | `:root`（52 行）+ Tailwind 配置（15 行） |
| 改导航菜单 | `initNav`（3614 行）附近 + `<nav>` 结构 |
| 改建站日期 | `SITE_BIRTH`（1667 行） |
| 改更新日志 | `viewHome` 里的 `news` 数组（约 2406 行） |
| 改分集剧情 | `EPISODES`（1389 行） |

---

## 2. 常见修改

### 2.1 加 / 换一张图片

1. 把图片放进 `assets/`
2. 想全站统一替换的，登记进 `MEDIA`：

```js
const MEDIA = {
  banner1: 'assets/banner1.jpg',
  my_new_pic: 'assets/my_new_pic.jpg',   // ← 新增
```

3. 模板里用 `ph()` 输出（它会自动处理「有图显示图 / 没图显示风格化占位块」）：

```js
${ph('my_new_pic', '', '图片说明', {ic:'star'})}
```

> `ph()` 也支持数组索引 key，例如 `ph('gallery_wall[3]', ...)`。

### 2.2 加一首歌

在 `SONGS` 里按 `S()` 的签名加一行：

```js
S('唯一id','日文标题','中文标题','罗马音','演唱单位','分类',时长秒,
  {ly:'作词',comp:'作曲',arr:'编曲',rel:'发售日',album:'所属专辑',ep:'出现场景',
   tempo:170, mood:'元气', chars:['yui','mio'],
   lyrics:[LYR('日文','中文','罗马音')]}),
```

- **`id` 必须唯一**，它同时是 `PREVIEWS` / 歌单 / 收藏 / URL 的键
- `分类` 只能取 `MUSIC_CATS` 里的 id：`op` `ed` `insert` `chara` `ost` `movie` `karaoke` `live`
- `chars` 填角色 id（`yui`/`mio`/`ritsu`/`tsumugi`/`azusa`…），会影响「智能乱序」的同角色不连播
- `tempo` 用于快慢穿插，`charSongs` 会自动按 `chars` 归类到角色页

### 2.3 音源：怎么让它能播 / 怎么标「暂无音源」

播放器的音源优先级（`songAudioUrl()`）：

```
song.audio 自填  >  MEDIA.audio 映射  >  PREVIEWS iTunes 官方 30 秒试听
```

**三者都没有 → `songPlayable()` 返回 false**，UI 会自动：
- 列表/详情页显示「暂无音源」灰标
- 点击时跳到下一首可播放的，而不是静默失败

所以想加试听，只要往 `PREVIEWS`（3866 行）加一行：

```js
"你的song_id": 'https://audio-ssl.itunes.apple.com/....m4a',
```

想用自备音频（完整版），改 `S()` 返回里的 `audio` 字段或 `MEDIA.audio`。

### 2.4 给图库加图片（**两步，少一步就出错**）

图库分类固定 6 个：`official` / `scene` / `cd` / `wall` / `q`（头像Q版）/ `cut`。

**第一步**：把图片加进 `MEDIA` 对应数组的**末尾**：

```js
gallery_wall:[ ...原有..., 'assets/我的新壁纸.jpg' ],
```

**第二步（关键）**：在 `GALLERY_META`（2980 行）里给它写标注：

```js
'assets/我的新壁纸.jpg': {
  label:'唯与梓部室合影',      // 标题，14 字内
  scene:'部室·HTT黑板',        // 场景，10 字内
  kind:'动画截图',             // 插画 / 动画截图 / 实景照片 / 商品图 / 头像裁切
  chars:['yui','azusa'],       // 图中**真实出现**的角色 id，按左到右顺序
  note:'补充说明'
},
```

`chars` 的规矩：
- 能认出具体是谁 → 写具体 id（一张图可以有多个）
- 3 人以上确实认不出 → 写 `['group']`（界面显示「全员/多人」）
- 纯场景/静物/实景照片 → 写 `[]`（界面显示「无角色」）
- **认不出就不要写**，宁可空着

**没登记的图不会报错**，但只会显示「角色壁纸 N」这种通用标题，而且不参与角色筛选。

> 分类张数 `n` 会按数组实际长度**自动同步**，不要手工改。
> 角色筛选按钮由 `galleryCharsOf()` 生成，只列出该分类里真实出现过的角色。

### 2.5 换角色头像 / 立绘

- **头像**：覆盖 `assets/char_<id>_face.jpg`（<id> 见 `CHARACTERS`，如 `yui`、`nodoka`）。
  建议 **480×480 正方形**。全站的头像格子是 56×56 圆角方，用 `object-fit:cover`，
  所以**头像必须正方形且人脸居中**，否则一排格子里会显得歪。
- **立绘**（角色详情页左侧大图）：改 `CHAR_SRC`（558 行）。
  这里**没有条目的话不会显示占位块，而是回退去用头像**（`MEDIA.<id>`）顶上，
  所以漏配立绘时页面不会崩，只是大图变成了头像。
- **5 位配角**（`megumi`/`sumire`/`yukari`/`ichigo`/`riko`）用的是风格化 SVG：
  `assets/char_<id>.svg`。它们和其余 9 张动画风的截图风格不同，这是已知的。

**统一头像构图口径**（本站现在的标准，新裁头像请沿用）：
头部高度占画面 **70%**、头顶留 **10%** 余量、脸部**水平居中**、不留大面积背景。

### 2.6 加周边条目 / 换分类代表图

- 条目：`GOODS_DATA`（1571 行），5 个分组 `cds` / `figures` / `merch` / `books` / `collab`，
  每条是 `{t:'名称', d:'发售信息', note:'备注'}`
- 分类代表图：覆盖 `assets/goods_<分类>.jpg`（即 `goods_cd.jpg` / `goods_figure.jpg` /
  `goods_merch.jpg` / `goods_book.jpg` / `goods_collab.jpg`）。
  在 `MEDIA` 里删掉对应的 `goods_xx:` 那一行，该分类会退回图标占位。

> 代表图是按**缩略图**显示的（`object-fit:contain`），长宽比不限，不用特意裁成正方形。

### 2.7 改主题配色

两处要一起改，否则 Tailwind 的类名（`text-sakura2`、`bg-cream` 等）和 CSS 变量会对不上：

1. `<head>` 里的 Tailwind 配置（约 15 行）：`colors: { sakura:'#FFB7C5', ... }`
2. `<head>` 的 `:root`（约 52 行）：`--sakura:#FFB7C5; --sakura2:#FF8FAA; ...`

### 2.8 改导航 / 加一个页面

1. 写渲染函数 `function viewXxx(){ return \`...\`; }`（放在 ⑦ 区块）
2. 在 `Router.render()`（约 3400 行）的 `if / else if` 链里加一条路由
3. 需要页面级交互的，在 `PostRender.init()`（3482 行）里按 base 挂初始化
4. 加进顶部导航和 `initNav`（3614 行）

---

## 3. 七条铁律（违反会「静默」出错，不会报错）

1. **图库角色名只能来自 `GALLERY_META`**，不能按序号生成。
   旧版用 `CHARACTERS[i % 长度]` 轮流贴名字，导致图片和文字完全对不上。
2. **`GALLERY_CATS` 的 `n` 不要手改**，它按 `MEDIA.gallery_*` 的实际长度自动覆盖。
3. **`Player.next(auto)` 的参数不能省**。`true` 表示「本曲自然播完往后走」（只有
   `_audio.onended` 传），手动切歌不传参。合并两条会让「单曲循环」下 ⏭ 退化成重播。
4. **`next()` 里「顺序播放到队尾就停」和「智能乱序去全曲库补推荐」是两条分支，不能合并。**
   前者调 `pause()`，后者调 `_replenishFromLibrary()`。
5. **没有音源的判定统一走 `songPlayable()`。** 另外两道闸不能拆：`_deadAudio`（记本次
   会话加载失败的曲目）、`_audioFailStreak`（连续失败 6 次判定断网并停止续跳）——
   没有它们，断网时会一路跳完整个曲库。
6. **内容只写有确切出处的。** 核实不到就留 `待补充` 由渲染层显示，**不要推测填充**——
   部分填充会让验证过的和没验证的看起来一样权威，比留白更糟。
7. **`initStatsCounter()` 只能在首页渲染之后调用**（现由 `PostRender.init()` 负责）。
   放进 `init()` 会因为它跑在 `Router.render()` 之前而静默失效，六项统计永远停在 0。

---

## 4. 改完怎么验证

别只靠肉眼看。仓库里有两个现成脚本（`_*.js` 已被 `.gitignore` 忽略，不会进版本库）。

**注意：这台机器的 shell 是 Windows PowerShell 5.1，没有 `pwsh` 命令。**

```powershell
# 1) 起一个带调试端口的无头 Edge
$edge = 'C:\Program Files (x86)\Microsoft\Edge\Application\msedge.exe'
Start-Process $edge -ArgumentList '--headless=new','--remote-debugging-port=9333',
  '--remote-allow-origins=*',"--user-data-dir=$env:TEMP\kon-verify",
  '--autoplay-policy=no-user-gesture-required','--mute-audio','about:blank'
Start-Sleep 4

# 2) 跑回归：合成音 / ⏭ 行为 / 无音源跳过 / 图库图文对应 / 每日壁纸 / 猜歌 / 周边 / 角色头像 / 全站报错
node _verify.js '轻音少女粉丝站.html'
# 结果写入 _verify_out.json，全部步骤应为 ok（看 _verify_log.txt）

# 3) 资源引用审计：有没有引用了不存在的文件、有没有闲置素材
node _assets_audit.js '轻音少女粉丝站.html'
```

看图库时要注意：名场面/场景图带 `loading="lazy"`，**不滚动到视口就不会加载**，
直接查 `img.complete` 会误判成「加载失败」——验证脚本里已经做了滚动处理。

### 改过 `可视化编辑器.html` 之后必跑

编辑器是「读源码 → 改字符串 → 写回文件」，一旦解析逻辑出错就可能**损坏主站**。改它之后必须跑：

```powershell
# 41 项安全测试：往返一致性（什么都不改必须逐字节相同）、单点修改、新增/删除条目、语法合法性
node _editor_test.js '轻音少女粉丝站.html' '可视化编辑器.html'

# 浏览器端到端（需先起上面的无头 Edge）
node _editor_browser_test.js
```

其中**「往返一致性」是最重要的一条**：如果「什么都不改」却生成了不同的文件，
说明回写把整份文件都重写了——历史上这里踩过两个坑：CRLF 换行被写成 LF、
正则把行首缩进一起吃掉导致每保存一次缩进就多两个空格。

---

## 5. 常见故障排查

| 现象 | 原因 / 处理 |
|---|---|
| 页面一片空白 | 内联 JS 语法错误。浏览器 F12 看 Console；或把 HTML 里的 `<script>` 抽出来 `new Function()` 解析一遍 |
| 首页六项统计恒为 0 | `initStatsCounter()` 执行时机被挪到渲染之前了（见铁律 7） |
| 图库出现空白格子 | `GALLERY_CATS` 的 `n` 被手改过，与 `MEDIA.gallery_*` 长度脱节 |
| 图库角色名和图对不上 | 该图没写进 `GALLERY_META`，或写错了 key（key 必须和 `MEDIA` 里的路径完全一致） |
| 某首歌点了没反应 | 该曲 `songPlayable()` 为 false（无 `song.audio` / `MEDIA.audio` / `PREVIEWS`），属预期行为 |
| 歌播完不自动切下一首 | 检查 `_audio.onended` 是否被改掉；它必须调 `next(true)` |
| 角色头像一排看起来歪 | 头像不是正方形，或人脸没居中（见 2.5 的构图口径） |
| 中文 PowerShell 脚本报 `Unexpected token` | PS 5.1 按 ANSI 读无 BOM 的 `.ps1`，中文注释的字节会吞掉换行。**辅助脚本一律写成纯 ASCII** |
| 改了字体/图片没生效 | 浏览器缓存，`Ctrl+F5` |

---

## 6. 已知缺口

见 `README.md` 的「已知缺口」一节，主要有：31 首角色歌的制作名单为空、5 位配角的资料待核实、
30 首曲目没有官方试听、图库有 3 张图片内容存疑（`scene_live2.jpg` 是真人实景照、
`scene_kyoto.jpg` 与文件名不符、`wc_816904.jpg` 含跨作品角色）。

---

## 7. 社区后端（`fan_server.py`）

完整手册见 **[BACKEND.md](BACKEND.md)**（端点表、部署、安全、排查）。这里只记维护者最常踩的几点。

### 7.1 两个后端，职责完全不同，不要混

| | `admin_server.py` | `fan_server.py` |
|---|---|---|
| 作用 | 改**主 HTML**（加歌、换图、补名单） | 管**社区动态数据**（留言 / 排行榜 / 成就） |
| 监听 | `127.0.0.1:8765-8784` | `127.0.0.1:8790`（可配） |
| 鉴权 | **无** | 管理接口需 `X-Admin-Token` |
| 可否公网 | ❌ 绝对不要 | ✅ 但必须挂反代 + HTTPS |
| 存储 | `_backup/*.html` 备份 | `fan_data.db`（SQLite） |

**铁律：公网服务永远不提供改写主 HTML 的接口。** 想改内容就用本机管理台。

### 7.2 前端接入点（都在 ⑫ 区块）

- `KON_API.base`：同源留空；分离部署填后端域名
- `KON_API.probe()`：`init()` 里异步探活，**不阻塞首屏**；`file://` 协议直接判离线，不发请求
- 离线的正确姿势：所有社区模块走 `KON_API.offlineHtml()` 占位，**不能抛异常**——
  站点必须保持「双击就能用」这条底线
- 新增社区模块时记得在 `PostRender.init()` 里按 base 重挂（首屏渲染时探活还没回来）

### 7.3 改每日题池 / 加成就

- **每日题池**由曲目数据自动推导（有音源、非 `K-ON!` 前缀、无 `lyricNote`），
  与站内猜歌游戏口径一致；新增曲目后无需改后端，重启即生效
- **成就**改两处：主 HTML 的 `ACHV.DEFS`（前端展示）+ `fan_server.py` 的 `ACHV_KEYS`（服务端白名单）。
  只改一处会出现「徽章能解锁但同步不上去」或「服务端拒绝」
- 敏感词、留言开关、榜单条数在管理台「🏆 挑战运营」页改，写进 `kv` 表

### 7.4 改完怎么验证

```bash
python _test_fan_api.py            # 后端 55 项（临时库，不碰真数据）
node _test_community_browser.js    # 站点侧 23 项（需 headless Edge + fan_server 在跑）
node _test_admin_community.js      # 管理台侧 14 项
```

改主 HTML 后**照旧**要跑第 4 节的 `_verify.js` / `_assets_audit.js` 回归。
