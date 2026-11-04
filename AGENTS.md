# Repo 的服务器调试约定

REPO_NAME: LIBERO-VPro-Dataset
REPO_RELATIVE_PATH: Benchmark/LIBERO-VPro-Dataset

每次启动必须读取本文件及 /Users/fangzekuan/Desktop/reasearch/Code/AGENTS.md。
本文件开头两项用于定位 repo；其他项目运行约定从本 repo 的 README 读取。
本地根固定为 /Users/fangzekuan/Desktop/reasearch/Code，远程根固定为 /data0/fangzekuan/Code，
两端 repo 均使用 REPO_RELATIVE_PATH 对应的同名相对路径。

GitHub 仓库和版本不从 repo 目录名猜测，也不需要单独填写 GitHub 账号：
- 仓库 URL 自动读取本地 git remote get-url origin，URL 已包含 owner/repo。
- 调试版本自动读取本地 git rev-parse HEAD；服务器获取 origin 各分支更新后切换到这个完整 commit，不默认写死 main。
- 人工切换本地分支并 commit/push 后，后续调试自动跟随新的本地 HEAD。
- 私有仓库要求服务器已有读取 origin 的 Git 凭据；认证只在缺失时处理，不把密码或 token 写进本文件。

## 人工与 Codex 的分工

- 人工负责本地所有 git add、commit、push；本文件也应由人工提交。Codex 不代为提交或推送。
- 接到调试/运行测试任务后，Codex 自动完成以下步骤，无需再问同步 ID、展示上传计划或重复征求 Git 更新许可。
- 先检查本地 Git 干净状态，包含暂存、未暂存、未跟踪及子模块修改。发现修改时停止，只提示人工提交并推送。
- 取得本地 HEAD 的完整 commit。远程获取 origin 更新，切换到这个确切 commit，再校验两端相同。
- 使用共用 remote.sh 执行；它会自动处理检查、fetch、checkout 和执行前复核，不使用 rsync，不运行本地 commit/push。

## 具体调试步骤

1. 先读取本项目 README 中的环境、依赖、数据/模型路径和运行命令。
2. 从当前 repo 调用：

       bash /Users/fangzekuan/Desktop/reasearch/Code/SSH/remote.sh prepare

   此命令自动取得本地 commit 并准备服务器同版本代码；首次没有 Git 副本时自动 clone。
   若服务器还留有旧的非 Git 目录，验证新 clone 后先将旧目录保留为同级 .before-git-... 备份，再替换为 Git 副本。
   若本地 commit 未推送、服务器认证失败、远程 Git 工作区有修改或 origin 不匹配，说明实际错误并停止，不覆盖现场。

3. 选择项目实际使用的远程解释器/环境，然后执行具体命令，例如：

       bash /Users/fangzekuan/Desktop/reasearch/Code/SSH/remote.sh run -- /远程环境/bin/python script.py 参数

   run 自带 prepare 和版本复核，无需每条命令前手动再调用 prepare。
   需要指定 repo 时使用 --project 本地repo绝对路径；命令参数始终放在 -- 后。

4. 检查退出码和日志，定位问题后在 Mac 修改代码；修改完成只汇报修改内容，由人工 commit/push。
   用户说继续调试后，再自动获取新 commit 并运行，不要求重复初始化。
5. 已确认的解释器、资源路径和常用命令保存在项目 README，后续会话复用。
   需要 GPU 时复用已获准的 GPU 配置并检查当前状态；没有分配信息时再询问实际缺失参数，不抢占他人任务。

调试输出放到 CODE_RUN_DIR，缓存和临时文件使用用户数据盘；避免生成未忽略的测试产物污染仓库。
常规版本获取、切换和已要求的调试操作直接执行。只有无法推断的参数、认证/权限问题或确实有破坏性的操作才询问。
