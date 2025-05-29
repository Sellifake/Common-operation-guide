# 问题一

在 VS Code 里，当我打开 project 文件夹后，Powershell 终端的路径就在该目录，但是想运行子文件夹 `github` 里的 Python 文件时，每次都需要 `cd` 过去才能运行，有没有更方便的方式，就像 PyCharm 那样能一键运行？

## 1. 开启 “在文件目录执行” 设置

1. 打开命令面板（`Ctrl+Shift+P`），输入并选择 **Preferences: Open Settings (UI)**  
2. 搜索 `Execute In File Dir`，勾选 **Python › Terminal: Execute In File Dir**  
3. 此后，打开任何 `.py` 文件，点击右上角的 **Run Python File in Terminal** 按钮，VS Code 会先 `cd` 到该文件所在目录再执行  

## 2. 配置调试（launch.json）

1. 在侧边栏点击 **Run and Debug**（或 `Ctrl+Shift+D`），选择 **create a launch.json file**  
2. 选 “Python File” 作为模板，VS Code 会生成一个 `.vscode/launch.json`，确认或补充：  
   ```json
   {
     "version": "0.2.0",
     "configurations": [
       {
         "name": "Run Current File",
         "type": "python",
         "request": "launch",
         "program": "${file}",
         "console": "integratedTerminal",
         "cwd": "${fileDirname}"
       }
     ]
   }
   ```

## （可选）使用 Code Runner 插件

1. 打开设置（`Ctrl+,`)，搜索 `code-runner.runInTerminal`，勾选它  
2. 搜索 `code-runner.executorMap.python`，在 JSON 中修改：  
   ```json
   "python": "cd $dir && python $fileName"
   ```  
3. 在任意 `.py` 文件中，点击右上角的 **Run Code** 即可自动切换目录并运行脚本  

## 小结

- **最简单**：在 Settings 中开启 **Python: Terminal: Execute In File Dir**，用右上角的“Run Python File in Terminal”按钮  
- **调试时**：在 `.vscode/launch.json` 中把 `cwd` 设为 `${fileDirname}`  
- **使用 Code Runner**：在它的设置里加 `cd $dir`  
