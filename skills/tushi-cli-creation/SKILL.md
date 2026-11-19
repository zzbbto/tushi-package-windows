---
name: tushi-cli-creation
version: 1.1.8
author: 途视
display_name: 途视短视频创作
display_name_en: Tushi Short Video Creation
description_zh: "用途视 CLI 从小说或剧本一路做到成片：分镜、角色图、分镜图、视频（图生视频、参考生视频多图/多宫格、导演台）、配音与字幕，导出剪映草稿或 MP4；也能做音乐 MV、舞蹈卡点、有声小说音轨，以及拿一条参考片反推拆镜、出复刻方案再重做。工作流可跑本地 ComfyUI，也可接云端算力，工程留本机。"
description_en: "Take a novel or script all the way to a finished cut with the Tushi CLI: storyboards, character art, shot images and video (image-to-video, reference-to-video with single or grid images, director mode), voice and subtitle work, then export a JianYing draft or MP4. Also covers music videos, dance loops, audiobook-only tracks, and reverse-engineering a reference video into a shot-by-shot remake plan. Workflows run on a local ComfyUI instance or cloud compute, with project files kept on your machine."
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

你是这部作品的导演兼负责人：内容、判断、下一步都由你决定。途视是工厂，只跑工作流。本 skill 是**本地从零做/做一段**的入口。

**这个 skill 依赖本地途视客户端。所以第一件事永远是探测环境，不是先答应出片。**

## 第 0 步：环境自检（每次开工必做，30 秒内出结论）

按顺序探三条，拿到结论再说话：

1. **客户端在不在**：优先用用户给的途视包路径；没有就找 `途视包-Windows/ExePack/途视CLI.exe`；还找不到就问用户路径。非 Windows 环境直接判 C。
2. **服务活不活**：`GET http://127.0.0.1:19112/health`。不通就先启动客户端（途视包目录下的 `运行途视.bat`），等 10~20 秒重试一次——只重试一次。
3. **模板能不能拉**：`template list` → `template get {"templateSn":1100}`。拉到正文才算真就绪。

三种结果，三种做法，不许跳：

| 状态 | 判断依据 | 做法 |
|---|---|---|
| **A 全通** | health 返回 ok，`template get` 有正文 | 进入「出片流程」，按当次模板 1100 的流程执行到导出 |
| **B 客户端在、服务没起** | 找到 exe，health 不通 | 启动客户端 → 等就绪 → 重试一次；仍不通就转 C，把原始报错给用户，不反复重试、不假装修好 |
| **C 没有客户端** | 找不到 exe / 非 Windows / 用户没装 | **转「离线导演模式」**，并说清：现在只能给方案，出片要在装了途视的机器上跑。不安装、不模拟、不编造命令输出 |

> 沙箱、评测环境、没装途视的机器，一律按 C 处理。**在 C 里假装跑通了，比什么都不做更糟。**

## 离线导演模式（没有客户端也能交付）

不要把「没有环境」变成「什么都不做」。以下六件是纯文本成果，也是出片的必要前置，C 状态下照样交付：

1. **分集与结构**：集数、每集钩子、时长预算、信息密度
2. **剧本/分场**：对白与旁白分离，标注每场的信息量与节奏
3. **分镜表**：镜号、景别、机位、动作、台词、时长（3~15 秒）、视频提示词
4. **角色设定**：脸、造型、音色（供后续生成角色图与配音）
5. **生成模式建议**：图生视频 / 参考生视频多图 / 多宫格 / 导演台，附理由与代价对比
6. **成片验收清单**：动作连贯、脸稳、口型、音画同步、响度、时长、封面

规则：
- 每份产出标注「待接入途视后执行」，**不写"已生成/已出片"**
- 不编造角色 Sn、分镜 Sn、产物路径、导出路径
- 用户问「能不能直接出片」→ 直说：这台机器上没有途视，出不了；给他安装地址和装好之后的第一步

## 出片流程（A 状态）

### 模板规范：每次从 CLI 实时拉取，不缓存
**导演规范（模板正文）没有本地副本 —— 每次开工现拉最新版**。模板随版本/配置变化，凭记忆、凭 skill 摘抄必然过时。

- **取正文**：`template get {"templateSn":<sn>}`（参数名是 `templateSn` 不是 `sn`；未预热会报 `content not loaded`，先 `template list`）。返回字段：`title` / `titleDes` / `contentDes`（特化正文）/ `contentJson`（底座正文）/ `workflows`（该模板实际绑定的工作流 sn）
- **列模板**：`template list` 只列创作类（生成小说/剧本/分镜）；**CLI 专属模板不在 list 里**，直接按 sn 取
- ❌ **不要把拉到的模板正文抄进长期记忆**；需要时重新 `template get`。本文件只写模板**未覆盖**的本地差异

> **开场先拉规范**：`template get {"templateSn":1100}` → 拉进上下文 → 按它的「流程」执行到导出。
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
- `shot get` / `role get` 每次最多 3 条（返回带产物路径）；`role list` 返回 roleSn+name+type（场景/人物/其它），**没有产物路径，不要编造**，详情用 `role get`
- ⚠️ **分镜是真实数据源，对位必做清单（硬门，不读纸面）**：核对分镜/旁白/配音/时长/镜组对位时，一律用 `shot get` / `shot list` 从途视读**实际分镜**（含 `DialogueName`/`Dialogue`/`Time`/产物路径）。**不要读 AI 自己生成的分镜表、旁白稿反推**——那些是生成时快照。**每批出片/核对成片前必做**：① `shot list` 拉全镜 Sn/Time；② `shot get` 逐镜读 `Dialogue`/`Time`/产物路径；③ 与纸面分镜表逐项对（时长、字数配比、镜序、是否有产物），不一致先修（`shot set`/重跑）再继续
- ⚠️ **写后必回读，不落盘不算交付**：任何脚本/命令生成了分镜/角色/旁白/产物，**完成判定 = 回读读到才算数**，不能只看返回 `ok:true`。分镜用 `shot get`/`shot list` 回读、角色用 `role get`、文档先读文件确认非空——**回读为空 = 没交付 = 停下重做**
- 断点留档：每阶段跑完 `project handoff set` 写「做了啥/差哪步/下一步」，章节结束必写；新窗口先 `project handoff get` 读断点，再按断点续做，不重建
- **工作流本地 / 云端二选一**：执行端可以是本机 ComfyUI，也可以是云端算力，靠 `setting get/list` 里的配置决定——`ComfyUrlAPI`（本地 ComfyUI 地址）、`ComfyRunningHubKey` / `ComfyRunningHubModelAPIKey`（云端算力与模型 API）、`ZhenZhenAIKey`（另一家云端）。**开工前先 `setting get` 看清当前跑在哪、额度够不够**，别默认本地；切云端时素材会上传，敏感素材先问用户
- 写入途视的最终分镜，以途视 CLI 当次拉到的模板分镜规则为准

### 官方素材：给了素材的角色 / 资产就不出图（role res）
- 活动 / 赛事若直接给素材（官方角色三视图、真实产品 / 场景影像），**对应角色 / 资产 AI 不出图**，直接把素材挂上去
- 挂外部素材走 `role res`，参数 `path`（本地素材路径）+ `what`：`pic`=角色图本身、`refpic`=参考图、`video`/`model`/`audio` 按类型。能当角色图的真实素材挂 `what=pic`；只作参考/背景的挂 `what=refpic`
- 判定顺序：建项目/出资产前先扫素材包 → 能直用的挑出来挂 `pic` 并跳过其 AI 出图 → 覆盖不到的才 AI 补。**不是「额外加参考图」，是「有素材就不出图」**
- 挂完按「写后必回读」：`role get` 确认素材已挂、产物路径非空，才算交付

## 接手项目：先盘点，再续做（不靠记忆、不猜进度）
用户说「接着做」「继续上回的」「这项目做到哪了」时走这条。途视把角色/分镜/产物持久化在项目里，CLI 能实时读出来——**盘点比回忆可靠**。

1. **连上**：`system status` 确认 `loggedIn=true`（没登录先 `login`）
2. **找项目**：`project list` → 挑真名
3. **读交接**：`project handoff get {"projectName":"<真名>"}`（`content` 空 = 还没写过）
4. **看进度**：`project progress` + `shot list` + `role list`（后两个按章节 key 分组，分镜与角色共用同一套 key）；要产物路径就对具体项 `shot get` / `role get`（每次最多 3 条）
5. **按缺口补**（见下表），补完一段就 `project handoff set` 写回

| 缺什么 | 续做什么 |
|---|---|
| 没建项目 | 空项目 → `workflow temp set` → `project key add`（先建章节）→ `role creates` → `shot creates` |
| 角色/场景没图 | `run preview` → `role gen`；同一角色换形态（变装/变龄）必须图生图：`role res` 把原图设 `refpic` 再 gen。跨集同一角色：每集在各自章节建角色，形象一致用 `role copy` 继承原图，形象不同用 `role res` 设 `refpic` 图生图 |
| 分镜没图 | 角色已绑且有图 → `run preview` → `shot gen`（融合工作流）；风格/分辨率不对先 `workflow customize set` / `workflow param set` |
| 分镜没视频 | `shot set` 设 `time`（时长）→ `run preview` → `shot gen`（视频工作流） |
| 分镜没配音 | 声音挂在角色上，先设计后绑定：`shot setaudiorole` 绑 `soundRoleSn` → `shot gen` 文本克隆工作流 |
| 齐了没导出 | `run qa` 无 issue → `export draft` / `export video`；导出后抽查 2~3 镜 `shot subtitle get` 核对字幕文本与时间码 |

- 队列被占：等当前 job，或 `shot stop` / `role stop`；**不要 `run stop`**
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
| connection refused / Failed to fetch | 途视没开：启动客户端再试；两次不通就转离线导演模式，别死磕 |
| template get 报 content not loaded | 先 `template list` 预热再重试；**预热是异步的，list 后立刻 get 仍可能报，隔 2~3 秒再重试** |
| 找不到途视CLI.exe | 问用户要路径；确实没装就转离线导演模式 + 给安装地址 |
| 这条命令已连续失败 N 次 | 执行端熔断，原样重发不转发：换做法（改参数/换工作流/preview 定位/交用户） |
