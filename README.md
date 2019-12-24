# 南苑寻物 APP

Title | content
---|---
Target release | 2019/12/10
Epic | 南苑寻物
Document status | 进行中
Document owner | [叶荣杰](https://github.com/Jerryrongjie)
Designer | [叶荣杰](https://github.com/Jerryrongjie)
Developer | [叶荣杰](https://github.com/Jerryrongjie)
QA | [叶荣杰](https://github.com/Jerryrongjie)

## 产品介绍
南苑寻物是一款立足于中山大学南方学院的校园服务APP，可以帮助用户进行失物招领。通过Azure认知服务，为寻物降低困难。

拾遗者可以通过APP的拍照上传物品功能，识别物品的分类，填写简单的物品描述、位置信息和联系方式，将物品上传至遗失物数据库。
而失主则可以在APP数据库中根据不同的分类寻找自己的物品。失主还可以通过上传物品的照片、淘宝图片或者日常照片的细节截图对数据库中的物品进行比对，快速地找到相似度高的相关物品，减少搜索的时间。

## 使用场景
＃ | 标题 | 用户案例 | 重要程度
---|---|---|---
1 | 个人物品遗失 | 用户A突然发现自己的物品丢失了，上传物品信息和图片前来挂失 | 非常重要
2 | 捡到遗失物品 | 用户B在路边捡到一个钥匙，拍照上传到APP等待失主 | 非常重要


---

## API使用对比

＃ | API商家 | API类别 | 描述 | 不同点
---|---|---|---|---
1 | Azure | 自定义视觉 |Azure控制台可以很清晰的看到图片对比的相似度，对于上传图片、优化模型有很大的便捷性，可以通过可视化界面进行操作，不需要写代码。 | Azure 生成的标签均为英文，需要额外的翻译步骤，对于国内用户不友好。
2 | Baidu | 图像搜索 | Baidu图像搜索功能也可以通过可视化界面上传图片库，不需要使用代码。但是图片搜索与数据库中的图片进行对比时，仍需要编写代码，同时没有明确的相似匹配度数据 | 数据不够清晰，对图片拍摄角度光线要求比较高，没有提供相似度的数据

## [百度API代码示例](https://github.com/Jerryrongjie/lost_and_found/blob/master/_baidu.ipynb)

- 相似图片搜索—入库
```
# encoding:utf-8
import requests
import base64

'''
相似图检索—入库
'''

request_url = "https://aip.baidubce.com/rest/2.0/image-classify/v1/realtime_search/similar/add"
# 二进制方式打开图片文件
f = open('[本地文件]', 'rb')
img = base64.b64encode(f.read())

params = {"brief":"{\"name\":\"小度\", \"id\":\"1\"}","image":img,"tags":"1,1"}
access_token = '[调用鉴权接口获取的token]'
request_url = request_url + "?access_token=" + access_token
headers = {'content-type': 'application/x-www-form-urlencoded'}
response = requests.post(request_url, data=params, headers=headers)
if response:
    print (response.json())
```

- 相似图片搜索—检索
```
# encoding:utf-8

import requests
import base64

'''
相似图检索—检索
'''

request_url = "https://aip.baidubce.com/rest/2.0/image-classify/v1/realtime_search/similar/search"
# 二进制方式打开图片文件
f = open('C:/Users/kedre/Desktop/API/相似库/校园卡.jpg', 'rb')
img = base64.b64encode(f.read())

params = {"image":img,"pn":200,"rn":100}
access_token = '[调用鉴权接口获取的token]'
request_url = request_url + "?access_token=" + access_token
headers = {'content-type': 'application/x-www-form-urlencoded'}
response = requests.post(request_url, data=params, headers=headers)
if response:
    print (response.json())
```



## PRD 价值主张设计
### PRD1.加值宣言
- 加值宣言：
  - Azure 自定义视觉：可以通过拍照帮助用户生成物品的信息，减少人工输入描述的时间成本，也减少了模糊描述、错误描述产生的不良影响。
  - Azure 计算机视觉：失主可以通过上传照片，在数据库中找到匹配度较高的物品，减少翻失物库的时间。


### PRD2.核心价值 3%
- 核心价值宣言：通过自定义视觉API识别物品类型，简化描述物品的步骤；通过计算机视觉API，使失主可以通过自己物品的照片或图片与数据库中的物品进行快速匹配。

### PRD3.核心价值与用户痛点 3%
- 用户痛点宣言：丢失物数据库解决了寻找失物具有延迟性的痛点，数据匹配和物品识别让上传物品和寻找物品的时间大大减少。

### PRD4.人工智能概率性与用户痛点 3%
- AI概率性考量：自定义图像识别存在对物品分类错误的情况，捡到遗失物的用户可以根据识别结果判断是否自行修改物品类别。


### PRD5.需求列表与人工智能API加值 3%
- 需求列表：
  - 物品丢失到发现具有延迟性
  - 用户对于同一个物品的描述具有模糊性
  - 每日丢失物品过多会使寻找物品变得困难
  - 人们常常苦恼捡到东西怎么办
  - 朋友圈寻物会影响到不相关的人的感受
    
## API 产品使用关键AI或机器学习之API的输出入展示 15%
### API1.使用水平 5%
使用水平：在Azure 自定义视觉API中 输入 钥匙、书包、耳机等大学生常用物品，输出 该物品的类别。在计算机视觉API中，输入 同一个物品的不同照片，进行相似度的分析。


### API3.使用后风险报告 5%
使用后风险报告：对于物主身份的识别是目前遇到的主要问题，在线图片具有广泛的传播性，无法通过照片识别物主的身份，只能通过学生证、饭卡等信息记录用户数据，当出现纠纷时，及时将用户信息进行比对。

### API4.加分项 3%
使用复杂度：用了2个以上机器学习与人工智能的API之输入及输出
