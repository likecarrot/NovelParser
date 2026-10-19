# 辅助脚本与分镜交付命令

使用本技能 `scripts/comic_pipeline.py` 与可用 Python。全部命令走标准库；DOCX、PDF 提取按相应依赖。不生图，不需要 Pillow、reportlab、pypdf。

```powershell
$comicPython = '实际Python绝对路径'
$comicCli = '本技能scripts/comic_pipeline.py绝对路径'
& $comicPython -X utf8 $comicCli status --project '作品绝对路径/书名/第1卷'
```

全部命令中的中文/含空格路径作为单独参数，JSON 使用 UTF-8。程序只执行机械步骤，不调用绘图模型、不代写剧情或通过报告。

## 项目目录

书名根目录维护 README、docs/、source_texts/ 归档原稿与 split_texts/ 切分正文；各卷目录独立维护 `project.json`（Schema v6）、`source/originals`、`scripts/chapters`、`docs/adaptation`、`reports`。不建立 `art/`、`pages/`、`prompts/`、`exports/`。

`init-book` 默认复制原稿并核验 SHA256；只有用户明确 `--action move` 时移动。`init` 仅用于新建或空卷目录。续做使用 `status`。原文重提取见 [recovery.md](recovery.md)。

## 命令契约

| 命令 | 主要参数 | 行为 |
|---|---|---|
| init-book | --book-dir [--title] [--source] [--action copy/move] | 书名项目与原稿归档 |
| split-source | --book-dir --file [--output-dir] [--pattern] | 按真实卷章标题切分文本 |
| init | --project --source [--title] [--volume] | 初始化独立卷 |
| confirm-source | --note [--scope] | 确认来源完整性与边界 |
| chapter | [--chapter] | 读取真实来源 unit ID |
| mark-read | --chapter --note | 记录实际精读 |
| check-adaptation | --chapter [--file] | 校验本章细节契约；不代替语义审查 |
| set-script | --file | 导入完整稿，失效旧锁 |
| script-chapter | --chapter | 读取本章实际分镜与交接 |
| set-script-chapter | --chapter --file | 在已有完整结构稿中替换本章 |
| impact | --file | 比较真实修订影响 |
| check-script | — | 原著引用与结构检查 |
| review | --kind coverage/continuity/comic --file | 登记当前卷实际审查 |
| check-stage-review | [--file] | 机械校验阶段审查报告：批次覆盖、章节指纹与整改闭环 |
| lock-script | — | 三轮全卷审查与各阶段审查全覆盖后锁定，写出 full-script.md |
| assert-art | — | 检查锁仍然有效 |
| status / preflight | — | 只读显示锁定状态、章节、分镜与页数、缺失审查 |
| doctor | — | 检查环境与项目状态 |
| estimate-scale | — | 估算分镜规模与阶段审查批次 |
| resolve-issue | --id --evidence | 记录原文提取问题的真实解决证据 |

正常交付：`init` → `mark-read` → `confirm-source` → `set-script` → `check-adaptation` → `check-script` → `review` ×3 → `check-stage-review` → `lock-script`。锁定产物是 `full-script.md`、`project.json`、`scripts/chapters/*.json`、`docs/adaptation/*.md`、`reports/`。

剧本锁定后再修改，旧锁与旧审查立即失效，必须重跑审查。字段定义见 [schemas.md](schemas.md)。
