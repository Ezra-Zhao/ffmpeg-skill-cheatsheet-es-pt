# ffmpeg-skill 速查表 | ffmpeg-skill Cheatsheet | Guía rápida de ffmpeg-skill

**给编程 agent 装一个本地剪辑师——42 件工具全在本机跑，先量后剪再自验，不用把素材传上云**

> 一页看懂 kajisho5/ffmpeg-skill：给 Claude Code、Cursor、Codex 与任何 MCP 客户端的本地视频剪辑 skill。42 件包住 ffmpeg 与 ffprobe 的小工具，全在本机跑、无云、无 API key、只用 Python 标准库。这张卖工作流与平台模板，不卖参数表：固定工作流先量真实文件、能无损就不重编码、剪完自己复量抽帧亲眼看过才报完成；外加一张 doctor 坑卡——总检的 ok 与单件工具的 usable 是两回事。

- 上游项目：[kajisho5/ffmpeg-skill](https://github.com/kajisho5/ffmpeg-skill)（2026-11-17 实测 7,640 ★ / 703 forks，205 个 release，MIT，根目录只有英文 README）
- 平台：Claude Code、Cursor、Codex 与任何读 SKILL.md 的 agent；每件工具也可作 MCP 工具调用；本机要有 ffmpeg 与 python3 在 PATH 里
- 本页结构：中文主文 → English → Español → Português
- 同系列：用 HTML 写视频的生成路看 [hyperframes 速查表](https://github.com/Ezra-Zhao/hyperframes-cheatsheet-es-pt)；这张是拿真素材在本机剪辑与交付，两张是生成与剪辑两联

---

Apoyo y más: [GitHub Sponsors](https://github.com/sponsors/Ezra-Zhao) · [Gumroad — plantillas y cuadernos que uso](https://ezraflow28.gumroad.com) · [Ko-fi](https://ko-fi.com/ezra777zhao)

## 第一屏·固定工作流卡

| 步 | 做什么（原仓口径） |
|---|---|
| probe 先量 | 每件活先用 `probe.py` 量真实文件：时长、帧率、分辨率、色彩、音轨布局都以实测为准，不猜文件名、不假设参数 |
| edit 再剪 | 用带类型参数的结构化工具动手，不拼 shell 字符串；能无损（stream copy）就不重编码 |
| check 对规格 | 产物按目标平台的规格表逐项核：画幅、时长、响度、文件大小，超限项逐条给修法 |
| look 亲眼看 | 画面变了的，还要抽接触表（contact sheet）亲眼看过，才报完成——量对了不等于看对了 |

为什么要这套流程（原仓自述）：agent 就算「会」ffmpeg 也会猜——假设帧率、挑容器装不下的编码、把只需流复制的文件重编码一遍、报完成时压根没打开过产物。这套 skill 就是把猜的环节逐个拿掉。

## 42 件工具分组卡

| 组 | 干什么活（原仓口径，一组一行） |
|---|---|
| 剪接与去静音 | 剪、接、按阈值去静音（`silence.py --threshold -35` 这类），口播素材先粗剪就靠这组 |
| 定时长与改画幅 | 凑时长、改画幅（`fit.py --aspect 9:16 --fit crop` 把 16:9 裁成 9:16），横屏素材发竖屏平台的主力 |
| 字幕与卡拉 OK 高亮 | 烧字幕、逐词弹出与卡拉 OK 高亮（`caption.py --animate pop --karaoke`），位置绕开各平台界面遮挡 |
| 叠加与动态图形 | 下三分之一标题、章节片、进度条、倒计时、台标、贴纸这类图形层 |
| HDR 转 SDR 与 LUT | HDR 素材转 SDR、套 LUT，混素材时先统一到一条线上 |
| 音频清理与同步 | 音频清理、带类型的动态处理、同步漂移修正、多机位对齐 |
| 响度归一 | 按目标口径归一（`loudness.py -I -14 --tp -1` 这类），交付前必走 |
| 平台导出与交付检查 | 按平台模板出片（`render.py --template tiktok` 这类）、整片项目渲染、批量文件夹、交付前逐项检查 |

## 平台模板一行版

仓库自带交付模板：tiktok、reels、shorts、youtube、x、linkedin、facebook 七个视频平台，`--template all` 一次把七个平台从同一份剪辑全出齐；交付检查的规格表还覆盖广播与播客（播客含章节标记与声道数检查）。模板里把字幕位置按各家界面安全区摆好、响度按平台口径设好——一条命令出一个平台，不用逐个平台手调参数。

## 三条命令起步卡

| 命令 | 做什么（原仓口径） |
|---|---|
| `npx ffmpeg-skill` | 安装本体；已装的再跑一次是刷新 `~/.claude/skills/ffmpeg-skill`，副本不会自动更新 |
| `npx ffmpeg-skill doctor` | 体检本机组件：读 ffmpeg 的编码器、解码器、滤镜清单，报每件工具要的组件在不在，先于任务在 ffmpeg 里失败之前发现 |
| `npx ffmpeg-skill contract --json` | 读机器可读契约：每件工具收什么、写什么、要哪些 ffmpeg 组件、产物怎么验，都在里面 |

MCP 口径：`tools/list` 默认只列核心 12 件，省客户端上下文；设 `FFMPEG_SKILL_MCP_FULL=1` 才列全 42 件。没列出来的工具按名字照样能调，列不列只影响清单大小，不影响可用性。

## 坑卡（本张的命门）

| 坑 | 实际情况（原仓自述） |
|---|---|
| doctor 的 ok 与单件 usable 是两回事 | `ok` 是「只要有任何一件工具要的组件缺了就为假」：Homebrew 版 ffmpeg 缺 subtitles 滤镜、或缺 libvidstab，都会把总检转红，但多数工具照跑。要逐件看 `doctor --json` 的 `tools` 字段，别只看总 ok |
| 字幕转写要另装本地语音引擎 | `caption.py --transcribe` 要单独装一个本地语音引擎（最简单是 `pip install faster-whisper`）；首次运行下模型权重，音频全程不出本机。没装引擎时其余功能照常，doctor 会把引擎标为缺失的可选项，转写命令会直接给出各引擎的装法而不是瞎猜 |
| 它只当手不当脑 | 剪哪里、成片过不过审、什么算亮点、字幕写什么，都是上层的事，不在这套工具里——它只负责量、剪、导出与自验，喂给它的字幕文本按原文烧，一字不改 |
| 平台上限是参考不是保证 | 交付检查里时长与文件大小超平台表上限的，标 WARN 并注明依据是参考口径：平台会改这些数，上传前以平台当下口径为准 |

## 诚实标注区

- **势头数据**：7,640 ★ / 703 forks、205 个 release、590 个 commit（2026-11-17 原仓页与 gh API 实测，证据 A）；今日第三方榜位在列、第三方实测文称其登上当日 trending 页为第三方口径（证据 B）。2026-09-03 建仓，两个半月到 7.6K 星，版本已到 3.10.1。
- **兼容口径**：ffmpeg 6.1、7.1、8、9 在原仓 CI 里实测过；42 件工具全是 Python 3.10 以上标准库，无第三方依赖。
- **先量后剪再验**：这套的价值不在参数多全，在流程固定——量错了后面全错，所以 probe 永远第一步，look 永远最后一步。

---

## English (summary)

ffmpeg-skill (7,640 stars / 703 forks as of 2026-11-17, 205 releases, MIT) is a local video-editing skill for Claude Code, Cursor, Codex and any MCP client: 42 small tools wrapping ffmpeg and ffprobe, all running on your machine — no cloud, no API keys, nothing but the Python standard library. It teaches the agent a fixed workflow instead of a parameter table: probe first (every job starts by measuring the real file — duration, frame rate, resolution, colour, audio layout — never guessing from the file name), edit losslessly where possible (structured tools with typed arguments, no shell strings, stream copy instead of re-encoding when that suffices), then check the output against the destination's spec, and look at it as a contact sheet before reporting done. The 42 tools group into cutting/joining and silence removal, fitting duration and aspect (16:9 to 9:16 reframing), captions with karaoke highlighting, overlays and motion graphics, HDR-to-SDR and LUTs, audio cleanup with drift-corrected sync and multicam, loudness normalisation, and platform export with delivery checks. Delivery templates ship for TikTok, Reels, Shorts, YouTube, X, LinkedIn and Facebook — `--template all` renders all seven video destinations from one edit, with caption placement respecting each platform's safe zones — and the delivery-check table also covers broadcast and podcast specs. Three commands to start: `npx ffmpeg-skill` installs (re-running refreshes the copy; copies do not update themselves), `npx ffmpeg-skill doctor` audits your ffmpeg build's components, and `npx ffmpeg-skill contract --json` prints the machine-readable contract. Under MCP, `tools/list` advertises a core 12 by default to keep client context small (`FFMPEG_SKILL_MCP_FULL=1` lists all 42); every tool stays callable by name either way. The trap card matters most: doctor's overall `ok` and a single tool's `usable` are different signals — `ok` goes false when anything any tool needs is missing, so a Homebrew ffmpeg without the subtitles filter turns the whole check red while most tools still run; read the per-tool `tools` field in `doctor --json`. Speech transcription for captions needs a separate local speech engine (faster-whisper is the simplest install), downloads model weights on first run, and the audio never leaves the machine. And the honest boundary: this repo is the hands, not the brain — what to cut, whether a deliverable is approvable and what a caption should say are decided above it; caption text you feed in is burned exactly as written. FFmpeg 6.1, 7.1, 8 and 9 are tested in its CI.

---

## Español (guía rápida — traducción de estudio, pendiente de revisión por un hablante nativo)

**Un editor de video local para tu agente de programación: 42 herramientas en tu máquina, medir primero, cortar después y verificar al final, sin subir tu material a la nube.**

ffmpeg-skill (7.640 estrellas / 703 forks a 17 de noviembre de 2026, 205 releases, MIT) es una skill de edición de video local para Claude Code, Cursor, Codex y cualquier cliente MCP: 42 herramientas pequeñas que envuelven ffmpeg y ffprobe, todas en tu máquina — sin nube, sin claves API, solo la biblioteca estándar de Python. No vende una tabla de parámetros: enseña un flujo fijo. Primero medir (cada trabajo empieza midiendo el archivo real — duración, fotogramas por segundo, resolución, color, disposición del audio — nunca adivinando por el nombre), después editar sin pérdidas cuando se pueda (herramientas estructuradas con argumentos tipados, nada de cadenas de shell; copia de flujo en vez de recodificar cuando basta), luego comprobar la salida contra la especificación del destino y mirarla en una hoja de contactos antes de darla por terminada.

Las 42 herramientas se agrupan en: corte, unión y eliminación de silencios; ajuste de duración y de formato (pasar de 16:9 a 9:16); subtítulos con resaltado karaoke; superposiciones y gráficos en movimiento; HDR a SDR y LUT; limpieza de audio con sincronía corregida por deriva y multicámara; normalización de sonoridad; y exportación por plataforma con controles de entrega. Trae plantillas de entrega para TikTok, Reels, Shorts, YouTube, X, LinkedIn y Facebook — `--template all` genera los siete destinos de video desde una misma edición, con los subtítulos colocados fuera de las zonas que tapa cada interfaz — y su tabla de controles cubre también broadcast y pódcast. Tres comandos para empezar: `npx ffmpeg-skill` instala (al repetirlo refresca la copia; las copias no se actualizan solas), `npx ffmpeg-skill doctor` revisa los componentes de tu ffmpeg, y `npx ffmpeg-skill contract --json` imprime el contrato legible por máquina. Por MCP, `tools/list` anuncia por defecto un núcleo de 12 herramientas para ahorrar contexto (`FFMPEG_SKILL_MCP_FULL=1` lista las 42); todas se pueden llamar por nombre en cualquier caso.

La tarjeta de trampas es la pantalla clave: el `ok` global de doctor y el `usable` de cada herramienta son señales distintas — el `ok` se apaga en cuanto falta un componente que alguna herramienta necesita, así que un ffmpeg de Homebrew sin el filtro subtitles pone todo en rojo aunque la mayoría de herramientas siga funcionando; hay que leer el campo `tools` de `doctor --json` herramienta por herramienta. La transcripción para subtítulos pide un motor de voz local aparte (lo más simple: `pip install faster-whisper`), descarga los pesos la primera vez y el audio nunca sale de tu máquina. Y el límite honesto: este repositorio son las manos, no el cerebro — qué cortar, si una entrega se aprueba y qué dice un subtítulo se decide por encima de él; el texto que le des se quema tal cual, sin reescribir. Su CI prueba ffmpeg 6.1, 7.1, 8 y 9. Esta página es una síntesis con mis propias palabras a partir de la documentación pública del proyecto, no una traducción de su README.

---

## Português (guia rápido — tradução de estudo, aguardando revisão de um falante nativo)

**Um editor de vídeo local para o seu agente de programação: 42 ferramentas na sua máquina, medir primeiro, cortar depois e conferir no fim, sem subir o seu material para a nuvem.**

O ffmpeg-skill (7.640 estrelas / 703 forks em 17 de novembro de 2026, 205 releases, MIT) é uma skill de edição de vídeo local para Claude Code, Cursor, Codex e qualquer cliente MCP: 42 ferramentas pequenas que embrulham o ffmpeg e o ffprobe, todas na sua máquina — sem nuvem, sem chaves de API, só a biblioteca padrão do Python. Não vende uma tabela de parâmetros: ensina um fluxo fixo. Primeiro medir (cada trabalho começa medindo o arquivo real — duração, quadros por segundo, resolução, cor, disposição do áudio — nunca chutando pelo nome), depois editar sem perdas quando der (ferramentas estruturadas com argumentos tipados, nada de strings de shell; cópia de fluxo em vez de recodificar quando basta), em seguida conferir a saída contra a especificação do destino e olhar para ela numa folha de contatos antes de dar o trabalho por pronto.

As 42 ferramentas se agrupam em: corte, junção e remoção de silêncios; ajuste de duração e de formato (passar de 16:9 para 9:16); legendas com destaque karaokê; sobreposições e gráficos em movimento; HDR para SDR e LUT; limpeza de áudio com sincronia corrigida por deriva e multicâmera; normalização de loudness; e exportação por plataforma com checagens de entrega. Traz modelos de entrega para TikTok, Reels, Shorts, YouTube, X, LinkedIn e Facebook — `--template all` gera os sete destinos de vídeo a partir da mesma edição, com as legendas fora das zonas que cada interface cobre — e a tabela de checagens cobre também broadcast e podcast. Três comandos para começar: `npx ffmpeg-skill` instala (rodar de novo atualiza a cópia; as cópias não se atualizam sozinhas), `npx ffmpeg-skill doctor` revisa os componentes do seu ffmpeg, e `npx ffmpeg-skill contract --json` imprime o contrato legível por máquina. Por MCP, o `tools/list` anuncia por padrão um núcleo de 12 ferramentas para economizar contexto (`FFMPEG_SKILL_MCP_FULL=1` lista as 42); todas podem ser chamadas pelo nome de qualquer jeito.

O cartão de armadilhas é a tela-chave: o `ok` global do doctor e o `usable` de cada ferramenta são sinais diferentes — o `ok` apaga assim que falta um componente de que alguma ferramenta precisa, então um ffmpeg do Homebrew sem o filtro subtitles deixa tudo vermelho embora a maioria das ferramentas continue rodando; é preciso ler o campo `tools` do `doctor --json` ferramenta por ferramenta. A transcrição para legendas pede um motor de voz local à parte (o mais simples: `pip install faster-whisper`), baixa os pesos na primeira vez e o áudio nunca sai da sua máquina. E o limite honesto: este repositório são as mãos, não o cérebro — o que cortar, se uma entrega é aprovada e o que diz uma legenda se decide acima dele; o texto que você der é queimado exatamente como foi escrito. O CI dele testa ffmpeg 6.1, 7.1, 8 e 9. Esta página é uma síntese com as minhas próprias palavras a partir da documentação pública do projeto, não uma tradução do README dele.

---

## 来源与声明

- 灵感来源 / Source / Fuente / Fonte: [kajisho5/ffmpeg-skill](https://github.com/kajisho5/ffmpeg-skill)（2026-11-17 实测 7,640 ★ / 703 forks，205 个 release，许可证 MIT；星数为同日 GitHub 实测）
- 本仓是个人学习整理：内容用我自己的话总结自上游公开文档，不搬运原文，不冒充官方；工作流、工具分组与命令以 2026-11-17 的上游 README 为准，上游迭代后请以官方文档为最终依据。
- 西语、葡语部分为学习译稿，待母语校对。Las secciones en español y portugués son traducciones de estudio pendientes de revisión nativa.
- License: MIT © 2026 Guangyi Zhao（本仓自有内容；上游项目为 MIT）
