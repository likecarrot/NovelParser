# 《{{title}}》· {{volume}} 常用操作命令速查

请在终端中指定当前卷路径为 `--project` 执行流水线命令：

```powershell
# 1. 查看当前卷分镜进度与体检
& $python -X utf8 $cli status --project '{{root}}'
& $python -X utf8 $cli doctor --project '{{root}}'

# 2. 标记阅读正文与确认来源
& $python -X utf8 $cli mark-read --project '{{root}}' --chapter ch000001 --note '已读完本章核心剧情'
& $python -X utf8 $cli confirm-source --project '{{root}}' --note '已确认全文无缺漏漏读'

# 3. 导入工作剧本与结构检查
& $python -X utf8 $cli set-script --project '{{root}}' --file 'scripts/work-script.json'
& $python -X utf8 $cli check-script --project '{{root}}'
& $python -X utf8 $cli estimate-scale --project '{{root}}'

# 4. 提交三轮审查并锁定分镜
& $python -X utf8 $cli review --project '{{root}}' --kind coverage --file 'reports/coverage.json'
& $python -X utf8 $cli review --project '{{root}}' --kind continuity --file 'reports/continuity.json'
& $python -X utf8 $cli review --project '{{root}}' --kind comic --file 'reports/comic.json'
& $python -X utf8 $cli lock-script --project '{{root}}'
```
