---
{"更新時間":"2026-06-03","創建時間":"2026-06-03","tags":["aws","lambda"],"dg-created_time":"2026-06-03","dg-updated_time":"2026-06-03","dg-publish":true,"permalink":"/技術文件/AWS/lambda/Lambda asynchronous-synchronous/","dgPassFrontmatter":true,"created":"2026-06-03","updated":"2026-06-03","dg-note-properties":{"更新時間":"2026-06-03","創建時間":"2026-06-03","tags":["aws","lambda"]}}
---

# 目的
---
本文目的是建立一個 lambda function，並以 sync 或 async 的方式運行這個 lambda func
並且藉由 cloudwatch 工具，確認運行結果

## Code
---
建立 lambda function 的過程就跳過，以下是 lambda 程式碼
```
import json

print('Loading function')

def lambda_handler(event, context):
    print("Received event: " + json.dumps(event, indent=2))
    print("value1 = " + event['key1'])
    print("value2 = " + event['key2'])
    print("value3 = " + event['key3'])
    print(1/0) # 這裡故意讓他出錯
    #return event['key1']  # Echo back the first key value
    #raise Exception('Something went wrong')
```
![Pasted image 20260607233107.png](/img/user/images/Pasted%20image%2020260607233107.png)

# lambda synchronous
---
trigger lambda
```
export REGION=us-east-1
aws lambda invoke --function-name helloworld --cli-binary-format raw-in-base64-out --payload '{"key1": "value1", "key2": "value2", "key3": "value3" }' --region ${REGION} response.json
```

記得要確認 lambda region，因為 lambda region 是 regional resource
![Pasted image 20260603075401.png](/img/user/images/Pasted%20image%2020260603075401.png)

# lambda asynchronous
---
加 async 參數到 command
 `--invocation-type Event `

會取得 statuscode 202
![Pasted image 20260603075817.png](/img/user/images/Pasted%20image%2020260603075817.png)

## 如何看執行結果？
---
到 Functions > helloworld > Monitor > View CloudWatch logs
![Pasted image 20260603080543.png](/img/user/images/Pasted%20image%2020260603080543.png)

找到最新的 Log stream
![Pasted image 20260603080757.png](/img/user/images/Pasted%20image%2020260603080757.png)

執行 log 結果
![Pasted image 20260603080832.png](/img/user/images/Pasted%20image%2020260603080832.png)
