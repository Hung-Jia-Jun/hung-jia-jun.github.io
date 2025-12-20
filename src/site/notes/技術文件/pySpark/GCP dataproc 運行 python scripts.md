---
{"dg-publish":true,"permalink":"/技術文件/pySpark/GCP dataproc 運行 python scripts/","tags":["pyspark"],"created":"2024-11-17T23:14:33.805+08:00","updated":"2025-12-17T23:04:33.424+08:00"}
---


建立 dataproc cluster
```
gcloud dataproc clusters create jason-test-spark-job \
    --region=us-central1 \
    --properties='^#^dataproc:conda.packages=google-cloud-storage==2.18.2#yarn:yarn.scheduler.maximum-allocation-mb=256928#yarn:yarn.nodemanager.resource.memory-mb=256928'
```

## properties
在 package 裡面安裝 python package
`dataproc:conda.packages=google-cloud-storage==2.18.2`

指定 memory 使用量
`yarn:yarn.scheduler.maximum-allocation-mb=256928#yarn:yarn.nodemanager.resource.memory-mb=256928`

# pyspark pip install

1. 建立
requirements.txt
```
pytest==6.2.5
pyspark==3.2.0
google-cloud-storage==1.43.0
mlflow==1.23.0
```
2. 寫 pip init 腳本
pip_init.py
```
#!/bin/bash
# 1. 確認 requirements.txt 文件是否已經上傳到 GCS 並下載到本地
GCS_BUCKET_PATH="gs://dataproc-staging-us-central1-473678078038-tw1bdolx/requirements.txt"

LOCAL_PATH="/tmp/requirements.txt"

# 下載 requirements.txt
gsutil cp ${GCS_BUCKET_PATH} ${LOCAL_PATH}

# 使用 pip 安裝依賴
pip install -r ${LOCAL_PATH}
```

3. 上傳檔案
```
gsutil cp requirements.txt gs://dataproc-staging-us-central1-473678078038-tw1bdolx/
gsutil cp pip_init.sh gs://dataproc-staging-us-central1-473678078038-tw1bdolx/
```

4. 刪除原本的 cluster
`gcloud dataproc clusters delete jason-test-spark-job --region=us-central1`

5. 建立 cluster 時運行腳本
```
gcloud dataproc clusters create jason-test-spark-job \
--region=us-central1 \
--initialization-actions=gs://dataproc-staging-us-central1-473678078038-tw1bdolx/pip_init.sh
```


# Delete cluster
`gcloud dataproc clusters delete jason-test-spark-job --region=us-central1`

## Submit job
`gcloud dataproc jobs submit pyspark test.py --region=us-central1 --cluster jason-test-spark-job`

> [!IMPORTANT]  
> 記得要切換 GCP 環境 (SIT/UAT/PROD)

#python #pyspark #spark #cluster