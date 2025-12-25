---
{"tags":null,"dg-publish":true,"dg-draft":false,"dg-created_time":"2025-12-25","dg-updated_time":"2025-12-25","permalink":"/技術文件/Apache Beam/Apache Beam 研究/","dgPassFrontmatter":true,"created":"2025-12-25","updated":"2025-12-25"}
---

#etl #apache-beam #pipeline
Example code


```
import sys

import apache_beam as beam
"""

假設這是個用戶從什麼來源進來的資料

"""

sample_dataset = [

	{'id': '123',  'source': 'A'}, 
	
	{'id': '456',  'source': 'A'}, 
	
	{'id': '456',  'source': 'B'}, 
	
	{'id': '789',  'source': 'B'}, 
	
	{'id': '789',  'source': 'C'}, 
	
	{'id': '789',  'source': 'D'}, 

]

  

if __name__ == '__main__':

	with beam.Pipeline(argv=sys.argv) as pipeline:

		(
		
		pipeline
		
		| '資料初始化(轉為 pcollection)' >> beam.Create(sample_dataset)
		
		| '每行資料轉為 (source,  1)' >> beam.Map(lambda row: (row['source'],  1))
		
		| '同樣的 source 加總起來' >> beam.CombinePerKey(sum)
		
		| '轉換輸出的格式' >> beam.Map(lambda row: {'source': row[0],  'count': row[1]})
		
		| '寫入檔案' >> beam.io.WriteToText('sample-output')
		
		)

```