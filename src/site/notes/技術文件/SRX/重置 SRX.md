---
{"tags":null,"dg-publish":true,"dg-draft":false,"dg-created_time":"2026-01-01","dg-updated_time":"2026-01-01","permalink":"/技術文件/SRX/重置 SRX/","dgPassFrontmatter":true,"created":"2026-01-01","updated":"2026-01-01"}
---

進入 `cli`
```
root@% cli
root>
```

進入 configuration mode
```
root> configure
Entering configuration mode
The configuration has been changed but not committed

[edit]
root#
```

輸入 `delete`，並回答 `yes`
```
root# delete
This will delete the entire configuration
Delete everything under this level? [yes,no] (no) yes


[edit]
root#
```

重設 root 帳號密碼
```
root# set system root-authentication plain-text-password
New password:
Retype new password:

[edit]
root#
```

輸入 `commit` 提交變更
```
root# commit
commit complete

[edit]
root#
```

輸入 `exit` 離開 config 模式
```
root# exit
Exiting configuration mode

root>
```
