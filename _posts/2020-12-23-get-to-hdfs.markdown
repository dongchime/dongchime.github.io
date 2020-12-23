---
layout: post
title:  "큰 사이즈의 파일을 바로 HDFS 로 다운받기"
date:   2020-12-23 14:52:22 +0900
categories: jekyll update
---

실험을 위해 아주 큰 데이터를 다운받아야 할 때가 있는데 서버의 용량이 충분하지 않아서 바로 HDFS 로 저장하고 싶을 때가 있다.

나 같은 경우는 ClueWeb12 (84G Compressed; approximately 688G uncompressed) 를 받아야 했다.  

terminal 에서 pipe("|") :을 이용해서 바로 HDFS 로 파일을 저장할 수 있다.

```bash
wget -O- http://www.lemurproject.org/clueweb12/webgraph.php/ClueWeb12_WebGraph_v2_0.txt.bz2 | bzip2 -dc | hdfs dfs -put - cw12.tsv
```

위와 같은 방식으로 다운을 받을 수 있는데,

먼저 wget 에서 -O- 를 사용해서 파일로 다운을 받는대신 stdout 으로 출력하게 한다. 그리고 그 결과를 받아 bzip2 으로 -d 옵션을 이용해 압축을 풀고 -c 옵션으로 stdout 으로 또다시 출력하게 한다. 그리고나서 HDFS 로 put 을 통해 파일을 저장한다.
 
