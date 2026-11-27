---
name: tushi-cli-creation
version: 1.2.3
author: 途视
display_name: 途视短视频创作
display_name_en: Tushi Short Video Creation
description_zh: "把小说或剧本做成分集、剧本、分镜表与视频提示词，装了途视客户端还能逐镜出图出片：四种生成模式、配音与字幕、导出剪映草稿或 MP4；也能做 MV、舞蹈卡点、有声小说音轨，以及按参考片反推复刻。工作流可跑本地 ComfyUI，也可接云端算力。"
description_en: "Turn a novel or script into episodes, scripts, storyboards and shot prompts; with the Tushi CLI installed it also renders each shot - four generation modes, voice work and subtitles, then export a JianYing draft or MP4. Also covers music videos, dance loops, audiobook-only tracks, and reverse-engineering a reference video into a remake plan. Workflows run on local ComfyUI or cloud compute."
description: >-
  用途视CLI驱动途视.exe做短视频/短剧并导出剪映。用户说「用途视出片」「做一条短剧」「导出剪映草稿」「生成角色图/分镜/参考生视频」「做MV/舞蹈视频」「有声小说出音轨」「照这条片子复刻」「接着做/继续上回的」时用。
examples_zh:
  - "用这个短故事做一条三分钟短剧，人物和场景要对得上，最后导出剪映草稿"
  - "把这个剧本拆成分镜，出角色图和参考生视频，每镜控制在15秒以内"
  - "照这条参考片反推它是怎么拍的，给我复刻方案，再换成我的人物重做一版"
examples_en:
  - "Turn this short story into a three-minute drama with consistent characters, then export a JianYing draft"
  - "Break this script into shots, generate character art and reference-to-video clips, each shot under 15 seconds"
  - "Reverse-engineer how this reference film was shot, then remake it with my own cast"
---
如果本地不存在应用，就用：https://gitee.com/zttbb/tushi-package-windows.git
# 途视CLI 短视频创作（本地）

你是这部作品的导演兼负责人：内容、判断、下一步都由你决定。途视是工厂，只跑工作流。本 skill 是本地从零做、或接手做一段的入口。

## 你能交付什么

| 产物 | 内容 | 需要客户端 |
|---|---|---|
| 分集结构 | 集数、每集钩子、时长预算、信息密度 | 否 |
| 剧本 / 分场 | 对白与旁白分离，标注节奏与信息量 | 否 |
| 分镜表 | 镜号、景别机位、画面动作、台词、时长、视频提示词 | 否 |
| 角色设定 | 脸、造型、音色（供后续出图与配音） | 否 |
| 生成模式建议 | 图生视频 / 参考生视频多图 / 多宫格 / 导演台，附代价对比 | 否 |
| 角色图 / 分镜图 / 视频 | 按分镜逐镜生成、跨镜衔接 | **是** |
| 配音 / 字幕 / 导出 | 克隆音色配音、字幕校准、剪映草稿或 MP4 | **是** |

**前五项现在就能做**，不依赖任何安装；后两项要途视客户端，见「要出片时」。

## 默认动作：先把可拍的方案给全

收到故事、剧本或一句想法，**不要先问环境、不要先说"需要安装"**，直接产出：

1. **分集**：按目标时长切集，每集给冷开场 + 转折 + 落点
2. **分镜表**：一镜一动作，逐镜给齐六列——镜号 / 景别与机位 / 画面动作 / 台词或旁白 / 时长（3~15 秒，单镜不超 15 秒）/ 视频提示词
3. **冷读自检**：装第一次看的观众，逐镜自答"谁、在哪、在干嘛"，答不出的镜先改文本再往下走
4. **生成模式建议**：一句话说清该用哪种模式、为什么、代价是什么

分镜表能不能用，就看这几条：**动作打头、氛围在后**（否则只会出慢推）；画面里一个字都不写；时长按台词量估（约 4 字/秒）；旁白和对白分开写。

交付完再说环境：要真出片，见下一节。

## 要出片时：先探测环境（30 秒）

1. **客户端在不在**：优先用用户给的途视包路径；没有就找 `途视包-Windows/ExePack/途视CLI.exe`；还找不到就问用户路径。非 Windows 环境直接判 C。
2. **服务活不活**：`GET http://127.0.0.1:19112/health`。不通就先启动客户端（途视包目录下的 `运行途视.bat`），等 10~20 秒重试一次——只重试一次。
3. **模板能不能拉**：`template list` → `template get {"templateSn":1100}`。拉到正文才算真就绪。

| 状态 | 判断依据 | 做法 |
|---|---|---|
| **A 全通** | health 返回 ok，`template get` 有正文 | 进入「出片流程」，按当次模板 1100 的流程执行到导出 |
| **B 客户端在、服务没起** | 找到 exe，health 不通 | 启动 → 等就绪 → 重试一次；仍不通就转 C，把原始报错给用户，不反复重试 |
| **C 没有客户端** | 找不到 exe / 非 Windows / 用户没装 | 按「默认动作」交方案，说清出片要在装了途视的机器上跑；不安装、不模拟、不编造命令输出与产物路径 |

> 沙箱、评测环境、别人的机器，一律按 C 走：**方案照给，只是不出片**。在 C 里假装跑通了，比什么都不做更糟。
- 用户问「能不能直接出片」→ 直说：这台机器上没有途视，出不了；给他安装地址和装好之后的第一步

## 出片流程（A 状态）

### 模板规范：每次从 CLI 实时拉取，不缓存、不写入长期记忆
**导演规范（模板正文）没有本地副本 —— 每次开工现拉最新版**。模板随版本/配置变化，凭记忆、凭 skill 摘抄必然过时。

- **取正文**：`template get {"templateSn":<sn>}`（参数名是 `templateSn` 不是 `sn`；未预热会报 `content not loaded`，先 `template list`）。返回字段：`title` / `titleDes` / `contentDes`（特化正文）/ `contentJson`（底座正文）/ `workflows`（该模板实际绑定的工作流 sn）
- **列模板**：`template list` 只列创作类（生成小说/剧本/分镜）；**CLI 专属模板不在 list 里**，直接按 sn 取
- ❌ **不要把拉到的模板正文抄进长期记忆**；需要时重新 `template get`。本文件只写模板**未覆盖**的本地差异

> **开场先拉规范**：`template get {"templateSn":1100}`（CLI导演全自动）→ 拉进上下文 → 按它的「流程」执行到导出。
> 命令名、参数名**禁止凭本文档记忆**，以当次 `help` / `schema` / `template get` 返回为准。

> **接手别人做到一半的项目？** 先盘点真实状态（哪个项目、几集、角色与分镜产物到哪一步），从断点续做，别新建项目。

### 开场四件事
1. **启动途视**：跑途视包目录下的 `运行途视.bat`（已在跑就跳过）
2. **拿 CLI 绝对路径**（skill 可能被拷到别处，相对路径失效）：优先用户给的路径，否则找 `途视包-Windows/ExePack/途视CLI.exe`
3. **`help` → `template list`（预热）→ `template get {"templateSn":1100}`**，按它的「流程」执行到导出
4. **剪映草稿目录**：`schema` 查设置命令 → list → get。**先信本机**：`source=setting` 用已有设置，空则扫本机剪映/CapCut 常见目录（`source=local` 会写回设置），`source=missing` 才请用户在途视设置里填，不要写死盘符路径

### 本地执行差异（模板 1100 未覆盖的）
- Windows 终端（PowerShell/cmd）**用单引号把整个 JSON 参数包住**，否则引号被吞；Git Bash 乱码先 `chcp 65001`
- ⚠️ **agent 环境里 CLI exe 的 stdout 会被吞（跑了但无输出）** → 改打它的 HTTP 服务：`POST http://127.0.0.1:19112/command`，body 为 `{"cmd":"<命令>","args":{<参数>}}`（如 `{"cmd":"template get","args":{"templateSn":1100}}`）；`GET /health` 探活。**注意 HTTP 的 `cmd` 是字符串、参数要嵌在 `args` 下**，与 exe 的 `cmd 位置参数 + JSON` 写法不同，别混用
- 产物经 HTTP 接口获取：`http://127.0.0.1:19112/file?path=<产物路径>`（图片/视频/音频都走这个接口，视频大文件也适用），不要直接读本地大文件
- `shot get` / `role get` 每次最多 3 条（返回带产物路径）；`role list` 返回 roleSn+name+`baseName`+type+`promptBase`+`curPic`+`refPic`（场景/人物/其它），`baseName`=同主体基准名（同一角色不同形态相同，用于同主体识别），`curPic`=完整形象/变体产物、`refPic`=基准图参考（人物=人脸，道具=基准图），可据此判断两步出图进度；路径为空就是没有，**不要编造**，更多详情用 `role get`
- ⚠️ **分镜是真实数据源，对位必做清单（硬门，不读纸面）**：核对分镜/旁白/配音/时长/镜组对位时，一律用 `shot get` / `shot list` 从途视读**实际分镜**（含 `DialogueName`/`Dialogue`/`Time`/产物路径）。**不要读 AI 自己生成的分镜表、旁白稿反推**——那些是生成时快照。**每批出片/核对成片前必做**：① `shot list` 拉全镜 Sn/Time；② `shot get` 逐镜读 `Dialogue`/`Time`/产物路径；③ 与纸面分镜表逐项对（时长、字数配比、镜序、是否有产物），不一致先修（`shot set`/重跑）再继续
- ⚠️ **写后必回读，不落盘不算交付**：任何脚本/命令生成了分镜/角色/旁白/产物，**完成判定 = 回读读到才算数**，不能只看返回 `ok:true`。分镜用 `shot get`/`shot list` 回读、角色用 `role get`、文档先读文件确认非空——**回读为空 = 没交付 = 停下重做**
- 断点留档：每阶段跑完 `project handoff set` 写「做了啥/差哪步/下一步」，章节结束必写；新窗口先 `project handoff get` 读断点，再按断点续做，不重建
- **工作流本地 / 云端二选一**：执行端可以是本机 ComfyUI，也可以是云端算力，靠 `setting get/list` 里的配置决定（key 就是 `ProjectWorkflowPathSetting` 的字段名，`setting list` 会连 `note` 中文说明一起返回）——`ComfyUrlAPI`（本地 ComfyUI 地址）、`ComfyRunningHubKey` / `ComfyRunningHubKeyAbroad`（云端算力，国内/国外两套）、`ComfyRunningHubModelAPIKey`（模型 API）、`ZhenZhenAIKey`（另一家云端）、`IsAbroad`（true=国外平台，false=国内）、`UseRHAPIChannel` / `UseRHWebChannel`（API/网页通道开关，各有国内/国外一份）、`MaxRHRunTime` / `MaxRHAPIRunTime`（并发上限）、`ProjectPath`（素材缓存目录）、`ProjectDraftPath`（剪映草稿目录，返回里带 `resolved`/`source`）。**开工前先 `setting get` 看清当前跑在哪、额度够不够**，别默认本地；切云端时素材会上传，敏感素材先问用户。`setting set` 只收 `key`+`content`：开关填 `true/false`，数值填整数。⚠️ `setting list` 里 Key/Token 是掩码（`e9aa…bb4b(len=32)`，带 `masked:true`），要明文只能 `setting get <key>` 单取，别把整串凭据写进断点/文档；有 CLI 任务在跑时改 `IsAbroad`/通道开关/Key 会被拒（运行中切换会让在跑的 job 用错 key 轮询，先 `shot stop`/`role stop` 或等结束）
- **云端 Key 没配的官方口径看 `system status`**：返回里 `missingSettings[]`（每项 `key`/`name`/`getUrl`）列出未配置的云端 Key（RunningHub 国内/国外站、贞贞AI；两个站各自独立、**都会列出**，跟当前 `IsAbroad` 切在哪一站无关），`note` 里带「打开哪个网址拿 + 用 `setting set` 怎么填」。为空就是都配好了；有值就**把 `getUrl` 原样给用户去网站获取**（别自己拼、别删查询串），拿到后 `setting set` 填入——**不要凭空编 Key，也不要 Key 为空时硬跑生成**
- 写入途视的最终分镜，以途视 CLI 当次拉到的模板分镜规则为准

### 官方素材：给了素材的角色 / 资产就不出图（role res）
- 活动 / 赛事若直接给素材（官方角色三视图、真实产品 / 场景影像），**对应角色 / 资产 AI 不出图**，直接把素材挂上去
- 挂外部素材走 `role res`，参数 `path`（本地素材路径）+ `what`：`pic`=角色图本身、`refpic`=参考图、`video`/`model`/`audio` 按类型。能当角色图的真实素材（如真实产品照直接当角色图）挂 `what=pic`；只作参考/背景的挂 `what=refpic`
- 判定顺序：建项目/出资产前先扫素材包 → 能直用的挑出来挂 `pic` 并跳过其 AI 出图 → 覆盖不到的才 AI 补。**不是「额外加参考图」，是「有素材就不出图」**
- 挂完按「写后必回读」：`role get` 确认素材已挂、产物路径非空，才算交付

## 接手项目：先盘点，再续做（不靠记忆、不猜进度）
用户说「接着做」「继续上回的」「这项目做到哪了」时走这条。途视把角色/分镜/产物持久化在项目里，CLI 能实时读出来——**盘点比回忆可靠**。

1. **连上**：`system status` 确认 `loggedIn=true`
2. **找项目**：`project list` → 挑真名
3. **读交接**：`project handoff get {"projectName":"<真名>"}`（`content` 空 = 还没写过）
4. **看进度**：`project progress` + `shot list` + `role list`（后两个按章节 key 分组，分镜与角色共用同一套 key）；要产物路径就对具体项 `shot get` / `role get`（每次最多 3 条）
5. **按缺口补**（见下表），补完一段就 `project handoff set` 写回

| 缺什么 | 续做什么 |
|---|---|
| 没建项目 | 空项目 → `workflow temp set` → `project key add`（先建章节）→ `role creates`（场景/角色/道具每项都必须带 `baseName`，不能为空；没有形态变化就填 `name`。`promptBase` 人物/道具必填，场景可选——场景要走基准工作流时填环境基准描述）→ `shot creates` |
| 角色/场景没图 | 人物角色两步：① `run preview` → `role gen` 文生图「基准白底图四视图」（读 `promptBase` 提示词基准：脸型/五官/发型/饰品），产物落 `参考图`（`refPic`）→ ② `role gen` 图生图「一键四视图白底图」（输入 `参考图`=人脸 + `prompt`=衣着，输出 `配图`），第二步产物才是角色图；场景走正常文生图（输出配图）；人物和道具（类型 人物/其它）都先跑「基准白底图四视图」（绑 `promptBase`，输出参考图）——人物继续第二步图生图，道具类型自动落 `curPic`，确认 `curPic` 有图后无变体即完成，有变体再图生图改状态。同一角色换装：复用原形态 `refPic` 人脸参考（`role res` 设到新形态的 `refpic`），**不要重跑脸图，也不要用旧全身图**；同主体按 `baseName` 识别（同一主体的所有形态传同一个 `baseName`），纯换装建议 `promptBase` 与原形态一致。跨集同一角色：每集在各自章节建同名同形态角色，形象一致用 `role copy` 继承原图（含 `refPic`） |
| 分镜没图 | 角色已绑且有图 → `run preview` → `shot gen`（融合工作流）；风格/分辨率不对先 `workflow customize set` / `workflow param set` |
| 分镜没视频 | `shot set` 设 `time`（时长）→ `run preview` → `shot gen`（视频工作流） |
| 分镜没配音 | 声音挂在角色上，先设计后绑定：`shot setaudiorole` 绑 `soundRoleSn` → `shot gen` 文本克隆工作流 |
| 齐了没导出 | `run qa` 无 issue → `export draft` / `export video`；qa 默认按视频口径（必须有画面），走剪映草稿时加 `type:"draft"`（纯音频镜也算可导出）。导出后抽查 2~3 镜 `shot subtitle get` 核对字幕文本与时间码 |

- 队列被占：短任务等当前 job；只想停某几镜的生成用 `shot stop` / `role stop`（同一批里其它镜继续跑）；停整条长任务（`export` / `shot subtitle extract`）用 `run stop <jobId>`，jobId 由 `run status`（不带 jobId 会列最近任务，含已结束）拿
- 拿不准一律 `help` → `schema <命令名>`

## 视频反推复刻（参考片 → 复刻方案 → 分镜）
用户说「照这条片子复刻」「反推这条视频」「这镜头怎么做的」时走这条，不是从零编故事。
1. `template list` 里找「视频反推复刻(rea-video)」：有就 `template get` 取正文按它跑；**没有说明客户端还没更新到带这个模板的版本**，按下面手工口径做，并先跟用户说清是手工跑的
2. 手工口径：装 https://github.com/JuneYaooo/rea-video 的 CLI → 用参考视频跑出 `VIDEO.md` / report（逐镜拆解 + 判实拍/AI/代码/录屏 + 逆向提示词 + 复刻方案）
3. 复刻方案按「推理分镜基础」落成途视分镜（同样 `template get` 取正文），再走正常出片链：角色图 → 分镜图 → 视频 → 配音/字幕 → 导出
4. 反推结果里「需自行提供的素材」（录屏/产品照/实拍片段）要单独找用户要，不要 AI 生图硬凑

## 本地落盘：作品目录（本地特有）
作品产物一律放 `途视包-Windows\作品\参赛作品-<赛事简称>\`：
- 根目录：交付物（成片、文档、脚本）
- 子目录：`交件材料/`、`样片/`等

## 文本硬门：不过不 gen（本地加严）
> 剧情/提示词错了是整批返工，视频瑕疵重抽卡就行——能在文本层发现的问题绝不带到生成层。口径：`shot list` + `shot get` 全量拉 → 逐镜过（叠加模板 1100 审查全项，lint 依据以当次 `template get` 为准）→ `shot set` 自修 → 全过才进首次 gen。
- **冷读**（源《一袖成穹》"剧情看不懂"）：装第一次看的观众，逐镜答"谁/在哪/在干嘛"；答不出的镜先改文本再出图

## 本地出错速查（其余报错看模板 1100 的「出错速查」）
| 现象 | 应对 |
|---|---|
| connection refused / Failed to fetch | 途视没开：启动客户端再试；两次不通就按「默认动作」交方案，说清出片需要客户端，别死磕 |
| template get 报 content not loaded | 先 `template list` 预热再重试；**预热是异步的，list 后立刻 get 仍可能报，隔 2~3 秒再重试** |
| 找不到途视CLI.exe | 问用户要路径；确实没装就按「默认动作」交方案 + 给安装地址 |
| 这条命令已连续失败 N 次 | 执行端熔断，原样重发不转发：换做法（改参数/换工作流/preview 定位/交用户） |
