# 智能歌词样式切换

这是一个 Folium `folium: 1` 客户端模组。

功能：

- 在“歌词动画”按钮组最后添加“随机”；
- 随机池读取 Folia 的实时歌词动画注册表，包含官方动画和已经加载的第三方模组动画；
- “每首歌都随机样式”：每次换歌重新抽取；已经用爱心固定的歌曲优先使用本地记忆；
- “固定随机”：第一次为歌曲抽取后写入本地记忆，之后这首歌继续使用同一个动画；
- 播放页右下角爱心上方提供“刷新当前随机动画”按钮：立即为当前播放重新抽取；在“固定随机”模式下会同步更新这首歌的记忆，在“每首歌都随机样式”模式下只刷新当前播放；
- 播放页右下角提供爱心按钮，点击固定当前动画，再次点击取消固定；
- 本地记忆保存在该模组的 `song-style-map` 数据项中，最多保留最近 2000 首歌曲；歌曲键包含来源、歌曲名、歌手和专辑；
- 不联网，不上传歌曲信息，不需要 API Key。

## 宿主设置行为

Folia 自带的“每首歌随机模式”和本模组的随机动画会同时决定动画。只有用户真正进入本模组的“随机”动画时，模组才会检查宿主设置；如果宿主的 `randomVisualizerModePerSong` 已开启，模组会关闭它并显示提示。这是为了避免两个随机器互相覆盖。启用模组本身、停用模组或选择其他歌词动画时不会修改该设置。

## 权限和内部接口

模组清单中的权限为 `filesystem.data`，用于保存每首歌对应的动画模式。为了调用完整的官方和第三方动画，还使用了 Folia 0.7.x 的内部接口：

- `folium.internals.React` 和 `folium.internals.ReactDOMClient`：在模组自己的 React 根节点中挂载被选中的动画渲染器；
- `folium.internals.visualizerRegistry`：读取所有已经注册的动画和它们的真实 `render` 函数；
- `folium.internals.stores.visualizerSettings`：读取各模式的 tuning 和背景配置；仅在用户选择本模组“随机”且宿主旧随机开关开启时调用 `handleToggleRandomVisualizerModePerSong(false)`；
- `folium.internals.stores.themeSettings`：读取宿主的封面取色背景开关，用于组成与宿主一致的背景配置。

这些接口属于 Folia 内部实现，升级 Folia 后可能需要适配。模组没有使用签名文件，也不会修改其他主题设置。

## 渲染限制

模组会从 `stores.visualizerSettings` 读取各模式的 tuning，并按宿主渲染器实际使用的字段名（例如 `classicTuning`、`cadenzaTuning`）传给被抽中的渲染器；同时传递背景模式、通用背景开关和各背景 tuning。Folia 0.7.18 的公开内部导出没有暴露视觉器资源 store，所以自定义 Monet 背景图片不能从模组内部完整恢复。

Folia 的模组上下文只提供 `currentTime` 时钟和 `audio.getPower()`/`audio.getBands()` getter；音频 getter 的值是 0–1 快照，不提供真正的 MotionValue。模组保留这个数值范围，并用本地 `.get()`/`.on('change')` 适配器转换成渲染器可读的信号；`currentTime` 也只是具有 `.get()`/`.on()` 的 FoliumClock，不是 MotionValue。依赖这些对象接口的动画通常可以渲染，但像使用 `useSpring(audioBands.bass)` 的动画可能无法随音乐连续响应，这需要在实际播放器中验证。渲染器挂在模组自己创建的 React 根节点，因此依赖宿主 React Context 的第三方动画可能无法完整工作。

Folia 0.7.18 源码确认包含 14 个内置动画（cadenza、cappella、claddagh、classic、diorama、fume、lumiere、monet、partita、pendolo、sonnet、still、tempera、tilt）。当前工作区没有可启动的 Folia 0.7.18 播放器和第三方模组，无法声称已经逐一实机运行这些动画；发布前应在 Folia 0.7.18 中逐个切换，并至少加载一个第三方视觉器确认。已完成 JavaScript 语法检查，并对注册表为空、缺少内部接口、预览模式和存储失败路径做了保护。

## 本地安装

将整个 `lyric-style-switcher` 目录打包，确保 ZIP 顶层仍然是该目录，然后拖到 Folium 的模组面板。首次启用时请允许文件数据权限。

提交到 `folium-compound` 前，请添加一张 PNG/JPG/WebP 预览图，并在 `mod.json` 中设置 `preview`。源码仓库中不要放 `folium.sig.json`，签名由维护者审查后生成。
