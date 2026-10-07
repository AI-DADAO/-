---

name: hybrid-video-orchestrator
description: 将真人口播原片与文案剪成有导演节拍的竖屏视频，包含去卡壳、口气裁切、内容可视化、字幕、线上许可配乐、封面首帧和质检。适用于完整口播成片制作。

口播剪辑专家 · SkillHub Director Edition v12.1

© 2026 AI大道。原创内容的权利声明与第三方许可范围见 COPYRIGHT.md。

本版保留默认的 Director Mode：一份 director_plan.json 同时驱动人物、原景、信息画面和字幕；director_render.py 统一渲染。它不包含可选的旧版 Three.js PERSON / Remotion VISUAL 三路工程，也不提供该路线的运行命令。默认 Director Mode 的源码、计划契约与画面逻辑来自 v12.1 完整包。

安装与能力边界

目标环境为 Windows x64。由安装此 Skill 的智能体先确认 Python 3.13 可用；若缺失，可经系统包管理器安装并重新检测。随后执行 py -3.13 scripts/bootstrap.py，再执行 py -3.13 scripts/bootstrap.py --check-only 与 py -3.13 scripts/runner.py preflight --mode raw。bootstrap.py 准备独立运行环境、FFmpeg、固定 Python 依赖与基础转写模型；网络或系统权限不足时应报告失败，不能把未完成安装视为可用。若本机用 python 而非 py 启动 3.13，则以下命令也统一替换。SkillHub 拒收 Windows .cmd 启动文件，因此本版使用 Python 入口。完整环境要求见 buyer-readiness.md。

AI 道具图生成、线上音乐下载及逐曲授权核实依赖用户可用工具和曲库访问；这些素材不从上一项目复制。原片粗剪的 EDL、口气、音乐、封面与平台安全区均需审查；本 Skill 不承诺无人审查的一键成片。每条片建立全新空项目，记录原始视频与文案 SHA-256。

默认流程


基础精剪：原始口播与目标文案执行 py -3.13 scripts/runner.py raw_pipeline prepare --project <新项目> --video <原片> --target <文案>，审查报告并导出本片 intermediate/07_reviewed_edl.json，再执行 py -3.13 scripts/runner.py raw_pipeline finalize ...。得到 output/final.mp4 与细粒度 output/final.srt。已精剪视频和对应 SRT 可跳过此步。
口气闸门：执行 py -3.13 scripts/runner.py tighten_pauses --project <新项目>，对照原声审核候选静音，只将确认为多余口气的帧对齐窗口写入 intermediate/pause_cuts_reviewed.json，再用 --cuts 执行。保留自然句间停顿。
导演计划：从本片文案、SRT 和人物原景提炼传播作用、观众认知与视觉任务，再决定每个节拍的路由和构图。写 version:3 的 intermediate/director_plan.json，其中至少两处内容专属的 designed 场景真正进入 scene.elements。节拍数量随内容变化，不固定为 18 个。契约见 director-mode.md 与 composition-v11.md。
统一渲染：执行 py -3.13 scripts/runner.py director_render --project <新项目> --source <精剪片> --srt <字幕> --plan <最终计划> --out <无音乐画面成片>。检查 director_render_trace.json、接触表、重点帧和切点；不支持的布局应报错。
线上曲库配乐：根据本片 music_profile 试听、选择并核实许可，保存曲目与 music_license_evidence.json，再用 py -3.13 scripts/runner.py online_bgm ... 混音。保持音乐低于活跃人声并试听最密集口播、转场和结尾。规则见 online-music.md。
封面首帧：按本片 hook 与结论制作道具图，用本片原视频人物帧，执行 py -3.13 scripts/runner.py cover_intro ... 插入恰好一帧并输出独立封面 PNG。发布时还需在平台预览中检查遮挡。见 creator-collage-cover.md。
QC 与复核：执行 py -3.13 scripts/runner.py online_qc ...，核对声音、封面帧数、计划场景和媒体契约；再人工检查视觉品质、字幕中心轴、口气、音乐授权和平台安全区。自动 QC 通过不等于审美和版权通过。


视觉导演原则

每个节拍先回答“观众应留下什么判断”，再决定视觉任务、人物是否离场、画面布局和动效。连续人物段改变信息层级或景别；数字比较必须有结论，方法段要展示顺序。字幕一次一行，关键词只在一个层级承担焦点。

1080×1920 画布沿中心轴排版，顶部信息可左对齐；平台右侧操作栏通过安全宽度处理，不把所有内容整体左移。人物主要重点词采用左上角三级标题：短色条与小标签、白色粗体主标题、低对比解释句。避免大色块盖住人物胸口。开场保持源画面的自然亮度与肤色。

每条新片从题材和原景推导独立美术身份，至少两处内容专属构图。封面、全屏视觉与结尾若能与上一片直接互换，应重做关键画面。抖音预览安全区以实际发布界面复核；静态规则见 composition-v11.md。口气检查见 director-mode.md，音乐混音见 online-music.md。

交付

交付成片、导演计划、封面、接触表和 QC 报告，并用 provenance.json 记录本片原素材哈希。不能因自动检查通过而忽略明显的遮挡、同景别疲劳或风格断层。
