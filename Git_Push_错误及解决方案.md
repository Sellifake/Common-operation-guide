# Git 推送被拒绝错误及解决方案

Generated with the assistance of ChatGPT

## 错误说明
当你执行 `git push -u origin main` 时，如果远端的 `main` 分支已经存在提交且与你本地的历史不一致，Git 会拒绝推送并提示：
```
Updates were rejected because the remote contains work that you do
not have locally. This is usually caused by another repository pushing
to the same ref.
```
这是为了防止意外覆盖远端已有的内容。

## 解决方案

### 方案 A：先拉取并合并再推送
1. **拉取远端并合并**  
   ```bash
   git pull origin main --allow-unrelated-histories
   ```
2. **解决冲突**（如有）  
   - 编辑冲突文件，完成修改后：  
     ```bash
     git add <已解决文件>
     ```
3. **提交合并结果**（如 `git pull` 未自动提交）：  
   ```bash
   git commit
   ```
4. **再次推送**  
   ```bash
   git push -u origin main
   ```

### 方案 B：强制覆盖远端
> 警告：此操作会丢弃远端原有历史，应谨慎使用。
```bash
git push -u origin main --force
```

- `--force` 会将本地分支的提交直接替换远端分支的历史。

## 以后推送的小贴士
完成上述任一方案并设置上游分支后，以后在 `main` 分支上只需执行：
```bash
git push
```
即可将本地更新推送到远端，无需再使用 `-u` 参数。
