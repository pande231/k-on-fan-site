# 放課後ティータイム 応援站 — 轻音少女 K-ON! 粉丝站

单文件静态站点。**双击 `轻音少女粉丝站.html` 即可打开**，不需要服务器、不需要联网、不需要构建。

## 结构

```
轻音少女粉丝站.html     站点本体（约 5500 行：HTML 外壳 + 内联 JS）
可视化编辑器.html       ⭐ 不懂代码就用它：表单改文字/配色/图库标注/周边/曲目，选图自动裁剪
新手教程.md             ⭐ 零代码操作步骤（换图 / 改字 / 改色）
MAINTENANCE.md          进阶维护手册（数据在哪、铁律、验证方法、故障排查）
BACKEND.md              社区后端手册（接口表 / 公网部署 / 安全 / 排查）

—— 本机工具（不对外） ——
admin.html              管理台前端：改内容 + 留言审核 + 挑战运营
admin_server.py         管理台后端（改主 HTML；只监听 127.0.0.1，无鉴权，切勿公网）
启动管理台.bat          一键起管理台

—— 社区后端（可公网，配合反向代理） ——
fan_server.py           留言墙 / 每日猜歌 / 成就 / 数据 API（Python 标准库，零依赖）
start_community_service.bat   一键起社区后端
deploy/                 Caddyfile.example（HTTPS 反代）、fan-site.service（systemd）
fan_data.db             社区数据（SQLite，运行期生成，未纳入版本控制）

assets/
  ├─ *.jpg / *.svg      角色立绘、场景图、壁纸、CD 封面、周边代表图
  ├─ fonts/             自托管字体（560 个 woff2 切片 + fonts.css）
  └─ vendor/            Tailwind Play CDN 运行时的本地副本
_shots/                 布局截图（开发产物，未纳入版本控制）
_*.py / _*.js           开发期临时脚本（未纳入版本控制）
```

> **不想碰代码**：先看 `新手教程.md`，或直接双击 `可视化编辑器.html`。

## 社区功能（可选，需要后端）

站点本体**不依赖任何后端**——但留言墙、每日猜歌挑战排行榜、成就云同步这三样需要跨访客共享数据，由 `fan_server.py` 提供：

```bash
python fan_server.py          # 或双击 start_community_service.bat
# 浏览器打开 http://127.0.0.1:8790/ 即可看到完整效果
```

- **前端是渐进式的**：在线时自动探测并启用社区功能；离线（含 `file://` 双击）时这几块显示「离线模式」占位，**站点其余部分完全不受影响**
- **粉丝留言墙**：先审后显，管理台「💬 留言审核」过审后才会公开
- **每日猜歌挑战**：全站同一天同一首歌，最多猜 5 次，可复制无剧透 emoji 战报分享
- **成就徽章墙**：13 枚徽章，进度存本机，填过昵称后可云同步
- 公网部署（HTTPS 反向代理、systemd、安全说明）见 **[BACKEND.md](BACKEND.md)**
- **想零成本上线**：`BACKEND.md` 有两条免费路线——
  **纯静态托管**（Cloudflare Pages / GitHub Pages，社区功能自动降级、站点照常可用）
  和 **自家电脑 + Cloudflare Tunnel**（免备案免公网 IP，完整功能，只花电费）

**站点是一个 SPA**：`<main id="view">` 在 HTML 里是空的，所有页面都由文件底部那段内联 JS 用模板字符串渲染，路由走 hash（`#/music`、`#/gallery/wall` 等）。

## 本地自托管依赖

站点原本依赖两个外部地址，已全部落盘为本地文件，因此断网也能完整显示：

| 原本 | 现在 |
|---|---|
| `cdn.tailwindcss.com` | `assets/vendor/tailwind-3.4.17.js` |
| `miaoda.feishu.cn/fonts/...` | `assets/fonts/fonts.css` + 560 个 woff2 |

保留的远程地址只有三类，均为有意为之：

- **47 个 iTunes 30 秒试听**（`audio-ssl.itunes.apple.com`）——音乐的唯一在线音源层。
  77 首曲目里没有试听的 30 首会标注「暂无音源」，点击时自动跳到下一首可播放的。
  **合成音引擎已整体移除**，不会再有任何 WebAudio 伪造旋律
- **站外收听链接**（Apple Music / Spotify / 网易云 / uta-net）——`<a href>` 外链
- SVG 命名空间 `www.w3.org`——不是网络请求

> 唯一的例外是「趣味互动 → 打call模拟器」的提示音 `callTone()`：它是一个 0.24 秒的三角波短音效，
> 不属于歌曲合成音，因此保留。`getCtx()` / `_masterGain` 只为它而存在。

### 想更新字体

`fonts.css` 与那 560 个 woff2 是从 Google Fonts 抓取的，**不要手工编辑**。重新生成的办法：抓取
`https://fonts.googleapis.com/css2?family=ZCOOL+KuaiLe&family=Noto+Sans+SC:wght@400;500;700;900&family=Zen+Maru+Gothic:wght@500;700;900&display=swap`
（带浏览器 UA，否则拿到的是 ttf 而非 woff2），把其中的 woff2 下载到 `assets/fonts/`，再把 CSS 里的
URL 改写成 `./<文件名>`。

> 注意：Noto Sans SC 是**可变字体**，101 个切片同时覆盖 400/500/700/900 四个权重，四个 `@font-face`
> 声明指向同一批文件；Zen Maru Gothic 不是，122 个切片 × 3 个权重各一份。

### 想更新 Tailwind

下载 `https://cdn.tailwindcss.com` 的产物，覆盖 `assets/vendor/` 下的文件，并同步改 HTML `<head>` 里的
文件名。主题色定义在 `<head>` 内联的 `tailwind.config` 里（sakura / teabrown / mizuro / matcha 等）。

## 维护须知

> 想自己改内容（加歌、加图库图片、换头像、改配色…）请看 **[MAINTENANCE.md](MAINTENANCE.md)**，
> 那里是按「我想改 X → 改哪里」组织的操作手册。下面几条是改动时最容易踩的坑。

**`GALLERY_CATS` 的 `n` 不需要手工改。** 它会被一行代码按 `MEDIA.gallery_*` 的实际数组长度覆盖——
因为 `galleryItems()` 是按 `n` 截断的，两处手工维护一旦脱节就会渲染出空白占位块。

**图库的角色名一律来自 `GALLERY_META`，不许按序号生成。** 旧实现用 `CHARACTERS[i % 长度]` 给第 i 张图
轮流贴角色名，图片和文字完全对不上。现在 `galleryItems()` 只从 `GALLERY_META[图片路径]` 取标注；
没登记的图退回通用标题，并且**不写角色**——宁缺毋滥。新增图片时请在看图后把
`label / scene / kind / chars` 补进 `GALLERY_META`：`chars` 用 `'group'` 表示 3 人及以上合影，
纯场景写 `[]`。分类页的筛选按钮由 `galleryCharsOf()` 生成，只列出该分类里真实出现过的角色。

**`initStatsCounter()` 只能在首页渲染之后调用**（现由 `PostRender.init()` 负责）。放在 `init()` 里会在
`Router.render()` 之前执行，此时 `#view` 还是空的，函数会静默失效——数据统计六项就永远停在 0。

**`Player.next(auto)` 的参数不能省。** `true` 表示「本曲自然播完，往后走」，只有 `_audio.onended` 传它；
手动切歌（⏭ 按钮、`→` 键）不传参。单曲循环下两种情况必须分开：自动播完重播本曲，手动点下一首要真的
换歌。把两条合并回 `next()` 的话，单曲循环时 ⏭ 会退化成「重头播放」，而 ⏮ 仍然真的翻歌，两个方向
行为不一致；`_verify.mjs` 有对应用例覆盖。

**⚠️ `next()` 里「顺序播放走到队尾就停」和「智能乱序从全曲库补推荐」是两种模式，不要合并。**
`order` 走到末尾调用 `this.pause()`（符合「顺序」的语义）；`smart` 在队列里找不到下一首时调用
`_replenishFromLibrary()`，从 77 首全曲库补一批可播放的曲目进队列——这是为了修「歌单只加了 1 首时
⏭ 没反应」的 bug。两条路径不能互相污染。

**音源可用性由 `songPlayable()` 统一判定**（`song.audio` → `MEDIA.audio` → `PREVIEWS`）。不可播放的
曲目在 UI 上标注「暂无音源」，点击与切歌时由 `_nextPlayableFrom()` 自动跳过。另外两道闸不能拆：
`_deadAudio` 记录运行期加载失败的 id，`_audioFailStreak` 连续失败 6 次即判定断网并停止续跳——
没有它们，断网时会一路跳完整个曲库。

**内容填充原则**：剧情简介、名场面、制作名单等，只写有确切出处的。核实不到的一律留 `待补充`
由渲染层显示，**不用推测填充**——部分填充会让验证过的和没验证的看起来一样权威，比留白更糟。

## 已知缺口

- **31 首角色歌的制作名单**（作词/作曲/编曲）为空，渲染显示「待补充」。公开资料对这批曲目
  的记载零散且互相矛盾，未做填充
- 恵 / 菫 / 紫 / いちご / 莉子 五位配角的乐器与台词为「待核实」
- 30 首曲目没有 iTunes 官方试听，站内只能标注「暂无音源」而无法试听
- 周边图鉴每个分类只有一张代表图，商品级细图未逐条配齐；代表图取自公开商品图，
  并非官方图库素材，未逐张登记图源

### 逐张识图暴露出来的三处图片问题（未擅自改动，待核实）

- `scene_live2.jpg` 被识别为**真人实景舞台照**（5 名持乐器演奏者 + LED 大屏），
  不是《轻音》动画画面。它现在同时出现在「官方宣传图」和「名场面合集」里，位置可疑，
  需要替换成真正的动画图或移出图库
- `scene_kyoto.jpg` 的画面是「红祭典演出」（律与澪穿红色法被式演出服），
  与文件名暗示的京都修学旅行不符；而考据页的 `SCENE_MAP` 正用
  「京都|修学旅行|和服|御神签」等关键词把它当作京都配图，**两处都要一起核实**
- `wc_816904.jpg` 画面约 9 人，其中持小提琴者与粉发者不像轻音角色（疑为杂志跨作品合影），
  目前只标注了能确认的紬 / 澪 / 唯三位

## 版权

本站为非营利粉丝向资讯站。所有角色、图片、音乐版权归原作者 **kakifly**、**芳文社**、**京都动画** 所有。
站内不提供完整歌词与音频文件下载。
