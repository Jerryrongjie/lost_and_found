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
南苑寻物是一款立足于中山大学南方学院的校园服务APP，可以帮助用户进行失物招领。通过Azure认知服务和百度智能云，为寻物降低困难。

拾遗者可以通过APP的拍照上传物品功能，识别物品的分类，填写简单的物品描述、位置信息和联系方式，将物品上传至遗失物数据库。
而失主则可以在APP数据库中根据不同的分类寻找自己的物品。失主还可以通过上传物品的照片、淘宝图片或者日常照片的细节截图对数据库中的物品进行比对，快速地找到相似度高的相关物品，减少搜索的时间。

## 角色分析
- 数据库管理员
    - 数据库管理员负责对后台物品标签进行人工检验，解决自定义视觉、计算机视觉等API出现的识别错误导致的问题，人工更新错误标签。
    - 对普通用户的身份进行二次确认，减少冒领遗失物品的情况。
- 普通用户
    - 遗失者：用户可以通过网购时的商品图片、日常照片截图等上传至服务器，对数据库中的物品进行相似度分析，快速找到自己的物品。
    - 拾遗者：用户可以对捡到的遗失物进行拍照，并上传至数据库，自定义视觉服务会将其进行科学分类，贴上对应标签，供遗失者进行寻找。
    
    
## 产品迭代
1. 建立失物数据库，对导入大学生常用物品的图片集至Azure应用中，构建自定义视觉图片库，对不同物品人工贴上标签。
2. 完善图片上传服务，对用户上传的图片进行分析，生成物品类别标签、同时与数据库中的图片进行相似度比对。
3. 建立沟通交流平台，为遗失者和拾遗者上传文字描述提供帮助，同时帮助双方快速建立联系。
4. 完成APP的整体设计，产出正式版APP产品。


## 使用场景
＃ | 标题 | 用户案例 | 重要程度
---|---|---|---
1 | 个人物品遗失 | 用户A突然发现自己的物品丢失了，上传物品信息和图片前来挂失 | 非常重要
2 | 捡到遗失物品 | 用户B在路边捡到一个钥匙，拍照上传到APP等待失主 | 非常重要



## API使用对比

＃ | API商家 | API类别 | 描述 | 不同点
---|---|---|---|---
1 | Azure | 自定义视觉 |Azure控制台可以很清晰的看到图片对比的相似度，对于上传图片、优化模型有很大的便捷性，可以通过可视化界面进行操作，不需要写代码。 | Azure 生成的标签均为英文，需要额外的翻译步骤，对于国内用户不友好。
2 | Baidu | 图像搜索 | Baidu图像搜索功能也可以通过可视化界面上传图片库，不需要使用代码。但是图片搜索与数据库中的图片进行对比时，仍需要编写代码，同时没有明确的相似匹配度数据 | 数据不够清晰，对图片拍摄角度光线要求比较高，没有提供相似度的数据

## [百度 图像搜索代码示例](https://github.com/Jerryrongjie/lost_and_found/blob/master/_baidu.ipynb)

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

## [Azure 计算机视觉代码示例](https://github.com/Jerryrongjie/lost_and_found/blob/master/_Azure.ipynb)

```
import http.client, urllib.request, urllib.parse, urllib.error, base64
import requests
import json

headers = {
    # Request headers
    'Content-Type': 'application/json',
    'Ocp-Apim-Subscription-Key': 'xxx', # 改成自己的key
}

params = urllib.parse.urlencode({
    # Request parameters
    'visualFeatures': 'Categories,Tags',
    'language': 'zh',
})

url = 'https://xxx/vision/v2.0/analyze' # 改成自己的站点
image_url = 'http://inews.gtimg.com/newsapp_ls/0/10966517164_294195/0.jpg' # 可以修改图片的地址

response = requests.post(url, params=params, headers=headers, json={"url": image_url})
vison = response.json()

print(vison)
```

## PRD 价值主张设计
### PRD1.加值宣言
- 加值宣言：
  - Azure 自定义视觉：可以通过拍照帮助用户生成物品的信息，减少人工输入描述的时间成本，也减少了模糊描述、错误描述产生的不良影响。
  - Azure 计算机视觉：失主可以通过上传照片，在数据库中找到匹配度较高的物品，减少翻失物库的时间。


### PRD2.核心价值
- 核心价值宣言：通过自定义视觉API识别物品类型，简化描述物品的步骤；通过计算机视觉API，使失主可以通过自己物品的照片或图片与数据库中的物品进行快速匹配。

### PRD3.核心价值与用户痛点
- 用户痛点宣言：丢失物数据库解决了寻找失物具有延迟性的痛点，数据匹配和物品识别让上传物品和寻找物品的时间大大减少。

### PRD4.人工智能概率性与用户痛点 3%
- AI概率性考量：自定义图像识别存在对物品分类错误的情况，捡到遗失物的用户可以根据识别结果判断是否自行修改物品类别。


### PRD5.需求列表与人工智能API加值 3%

需求列表 | API加值
---|---
用户描述物品时存在模糊性 | 计算机视觉API识别物品的分类和标签
失物招领东西太多太难找 | 图像搜索API将不同物品以相似度进行排序
  
## 原型

### 原型页面

![原型页面](https://github.com/Jerryrongjie/lost_and_found/blob/master/%E5%8E%9F%E5%9E%8B.jpg)

### 产品架构图

![产品架构图](https://github.com/Jerryrongjie/lost_and_found/blob/master/%E4%BA%A7%E5%93%81%E6%9E%B6%E6%9E%84.jpg)

### 流程图

![流程图](https://github.com/Jerryrongjie/lost_and_found/blob/master/%E6%B5%81%E7%A8%8B%E5%9B%BE.jpg)

### 原型1.交互及界面设计

#### 核心交互人工智能加值的体现

- [1.4 发布成功页](https://jerryrongjie.github.io/lost_and_found/#g=1&p=1_4_%E5%8F%91%E5%B8%83%E6%88%90%E5%8A%9F%E9%A1%B5)

![1.4 发布成功页](https://github.com/Jerryrongjie/lost_and_found/blob/master/1_4_%E5%8F%91%E5%B8%83%E6%88%90%E5%8A%9F%E9%A1%B5.jpg)

- 通过图像搜索API，快速的匹配数据库中的物品信息，将发布物品相似度较高的物品详情列出，快速的找到匹配度最高的物品。
- 当数据库中存在遗失者所丢失的物品时，能够在第一时间帮助遗失者找回物品。

### 原型2.信息设计

#### 信息设计人工智能的加值

- [1.2 遗失发布](https://jerryrongjie.github.io/lost_and_found/#g=1&p=1_2_%E9%81%97%E5%A4%B1%E5%8F%91%E5%B8%83)

![1.2 遗失发布](https://github.com/Jerryrongjie/lost_and_found/blob/master/1_2_%E9%81%97%E5%A4%B1%E5%8F%91%E5%B8%83.jpg)
- 通过计算机视觉API，分析现实中的物品图片，生成物品的分类和标签，减少人工输入物品描述的时间成本。

### 原型3.原型文档

[南苑寻物原型链接](https://jerryrongjie.github.io/lost_and_found/)

### 原型4.口头操作说明

口头操作说明：多少程度上在2-3分钟时间限制内，对听众留下了「的确这是个可行丶可用的人工智能加值产品」的印象

## API 产品使用关键AI或机器学习之API的输出入展示
### API1.使用水平

- Azure计算机视觉：
    - 输入：书包、钥匙、耳机等大学生常用物品的图片
    - 输出：物品的类别和标签

- 百度：图像搜索API
    - 输入：物品图片的入库
    - 输出：物品图片的检索


### [API2.使用比较分析](## API使用对比)



### API3.使用后风险报告

- 使用后风险报告：对于物主身份的识别是目前遇到的主要问题，在线图片具有广泛的传播性，无法通过照片识别物主的身份，只能通过学生证、饭卡等信息记录用户数据，当出现纠纷时，及时将用户信息进行比对。

- 在产品原型设计的过程中，加入了个人信息验证的功能，当新用户使用APP时，需要登录并填写个人信息。在通过验证之后，系统后台将会将该用户标记为“已通过验证”用户。每个用户都可以看到这个验证通过的标识，并以此证明用户的可信度。

### API4.加分项
使用复杂度：用了2个以上机器学习与人工智能的API之输入及输出

- 百度：图像搜素API
- Azure：计算机视觉API
