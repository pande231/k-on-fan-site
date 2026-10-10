# 放課後ティータイム 応援站 — 轻音少女 K-ON! 粉丝站

单文件静态站点。**双击 `轻音少女粉丝站.html` 即可打开**，不需要服务器、不需要联网、不需要构建。

## 结构

```
轻音少女粉丝站.html     站点本体（约 4400 行：HTML 外壳 + 内联 JS）
assets/
  ├─ *.jpg / *.svg      角色立绘、场景图、壁纸、CD 封面
  ├─ fonts/             自托管字体（560 个 woff2 切片 + fonts.css）
  └─ vendor/            Tailwind Play CDN 运行时的本地副本
_shots/                 布局截图（开发产物，未纳入版本控制）
_*.py / _*.js           开发期临时脚本（未纳入版本控制）
```

**站点是一个 SPA**：`<main id="view">` 在 HTML 里是空的，所有页面都由文件底部那段内联 JS 用模板字符串渲染，路由走 hash（`#/music`、`#/gallery/wall` 等）。

## 本地自托管依赖

站点原本依赖两个外部地址，已全部落盘为本地文件，因此断网也能完整显示：

| 原本 | 现在 |
|---|---|
| `cdn.tailwindcss.com` | `assets/vendor/tailwind-3.4.17.js` |
| `miaoda.feishu.cn/fonts/...` | `assets/fonts/fonts.css` + 560 个 woff2 |

保留的远程地址只有三类，均为有意为之：

- **47 个 iTunes 30 秒试听**（`audio-ssl.itunes.apple.com`）——音乐的「真实音源」层，取不到时自动回退到 WebAudio 合成音
- **站外收听链接**（Apple Music / Spotify / 网易云 / uta-net）——`<a href>` 外链
- SVG 命名空间 `www.w3.org`——不是网络请求

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

**`GALLERY_CATS` 的 `n` 不需要手工改。** 它会被一行代码按 `MEDIA.gallery_*` 的实际数组长度覆盖——
因为 `galleryItems()` 是按 `n` 截断的，两处手工维护一旦脱节就会渲染出空白占位块。

**`initStatsCounter()` 只能在首页渲染之后调用**（现由 `PostRender.init()` 负责）。放在 `init()` 里会在
`Router.render()` 之前执行，此时 `#view` 还是空的，函数会静默失效——数据统计六项就永远停在 0。

**`Player.next(auto)` 的参数不能省。** `true` 表示「本曲自然播完，往后走」，只有两条自动路径传它
（`_audio.onended`、合成演示音的虚拟进度）；手动切歌（⏭ 按钮、`→` 键）不传参。单曲循环下两种
情况必须分开：自动播完重播本曲，手动点下一首要真的换歌。把两条合并回 `next()` 的话，单曲循环时
⏭ 会退化成「重头播放」，而 ⏮ 仍然真的翻歌，两个方向行为不一致；`_verify.mjs` 有对应用例覆盖。

**内容填充原则**：剧情简介、名场面、制作名单等，只写有确切出处的。核实不到的一律留 `待补充`
由渲染层显示，**不用推测填充**——部分填充会让验证过的和没验证的看起来一样权威，比留白更糟。

## 已知缺口

- **31 首角色歌的制作名单**（作词/作曲/编曲）为空，渲染显示「待补充」。公开资料对这批曲目
  的记载零散且互相矛盾，未做填充
- 恵 / 菫 / 紫 / いちご / 莉子 五位配角的乐器与台词为「待核实」
- 图库部分条目的下载链接为「待补充」

## 版权

本站为非营利粉丝向资讯站。所有角色、图片、音乐版权归原作者 **kakifly**、**芳文社**、**京都动画** 所有。
站内不提供完整歌词与音频文件下载。
