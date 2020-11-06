# IR-Evaluation
实验三的实验报告及代码实现
## 实验内容
实现以下指标评价，并对HW1.2检索结果进行评价：
Mean Reciprocal Rank (MRR);
Normalized Discounted Cumulative Gain (NDCG)；
## 实验过程
### 得到qrels_dict和test_dict
qrels_dict和test_dict分别从qrels.txt中和result.txt中得到
```javascript
#get qrels_dict
      for line in f:
          ele = line.strip().split(' ')
          if ele[0] not in qrels_dict:
              qrels_dict[ele[0]] = {}
          if int(ele[3]) > 0:
              qrels_dict[ele[0]][ele[2]] = int(ele[3])
```
```javascript
#get test_dict
      for line in f:
          ele = line.strip().split(' ')
          if ele[0] not in test_dict:
              test_dict[ele[0]] = []
          test_dict[ele[0]].append(ele[1])
```
简单来说，可以认为test_dict是测试的idset，qrels_dict是真实的idset,即
```javascript
      test_result = test_dict[query]
      true_list = set(qrels_dict[query].keys())
```
### MAP评价
MAP在Precision@K的基础上进行，主要步骤为：
1.考虑每个相关docid在测试结果中的位置，K 1 , K 2 , … K R；
2.为K 1 , K 2 , … K R计算Precision@K；
3.求这R个P@K的平均值AvgPrec，得到AP；
4.MAP极为AP的均值；
```javascript
#MAP
      for doc_id in test_result[0: length_use]:
          i += 1
          if doc_id in true_list:
              i_retrieval_true += 1
              P_result.append(i_retrieval_true / i)
              #print(i_retrieval_true / i)
      if P_result:
          AP = np.sum(P_result) / len(true_list)
          print('query:', query, ',AP:', AP)
          AP_result.append(AP)
      MAP=np.mean(AP_result)
```
可以得到MAP评价结果如下：（部分结果）
```javascript
query: 171 ,AP: 0.9498040597601832

query: 172 ,AP: 0.3412969283276451

query: 173 ,AP: 0.9978136200716846

query: 174 ,AP: 0.5675347800347801

…

query: 222 ,AP: 0.30126376980342995

query: 223 ,AP: 0.9940746736049804

query: 224 ,AP: 0.5178732378732379

query: 225 ,AP: 0.9920063553263518

MAP = 0.6148422817122279
```
### MRR评价
MRR相比其他两个较为简单，只需考虑第一个相关文档出现的位置就可以，步骤为：
1.考虑第一个相关文档的名次位置
2.计算排名分数为1/k
3.MRR即为RR的均值
```javascript
#get MRR
      for doc_id in test_result[0: length_use]:
          i += 1
          if doc_id in true_list:
              i_retrieval_true = 1
              P_result.append(i_retrieval_true / i)
              break
              #print(i_retrieval_true / i)
      if P_result:
          RR = np.sum(P_result)/1.0
          print('query:', query, ',RR:', RR)
          RR_result.append(RR)
      MRR=np.mean(RP_result)
```
可以得到MRR评价结果如下：（部分结果）
```javascript
query: 171 ,RR: 0.5

query: 172 ,RR: 1.0

query: 173 ,RR: 1.0

query: 174 ,RR: 0.2

…

query: 222 ,RR: 0.3333333333333333

query: 223 ,RR: 1.0

query: 224 ,RR: 0.2

query: 225 ,RR: 1.0

MRR = 0.79737012987013
```
### NDCG评价
NDCG基于两个假设：
1.高度相关的文档比边缘相关的文档更加有用
2.文档的排名越低，对用户越无用
具体步骤为：
1.给每一个真实相关的doc，附一个gain
2.计算第n级的CG
3.做一个discount的log运算，意为对测试结果的排名做一个惩罚（高rel，但rank不够靠前也很拉低评分），得到DCG
4.标准化，得到IDCG,进而计算NDCG
5.对每个query的NDCG求均值，得到最后的NDCG
```javascript
#getNDCG
        if length_use <= 0:
            print('query ', query, ' not found test list')
            return []
        for doc_id in test_result[0: length_use]:
            i += 1
            rel = qrels_dict[query].get(doc_id, 0)
            DCG += (pow(2, rel) - 1) / math.log(i, 2)
            IDCG += (pow(2, true_list[i - 2]) - 1) / math.log(i, 2)
        NDCG = DCG / IDCG
        print('query', query, ', NDCG: ', NDCG)

```
可以得到NDCG评价结果如下：（部分结果）
```javascript
query 171 , NDCG: 0.9398543518229351

query 172 , NDCG: 0.9522319284335552

query 173 , NDCG: 0.8787194969898994

query 174 , NDCG: 0.4307012038436227

…

query 222 , NDCG: 0.5087328728028815

query 223 , NDCG: 0.9063275712084274

query 224 , NDCG: 0.3773185814513307

query 225 , NDCG: 0.9706077927297266

NDCG = 0.756819929645465
```
## 实验
