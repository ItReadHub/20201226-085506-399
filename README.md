# 抖音短视频爬取实战

<a name="6VZ69"></a>
### 需求
爬取用户的抖音号、粉丝数、关注数、点赞数
<a name="7smGu"></a>
### 目的：
某公司做生鲜电商平台，这个公司平台想在流量平台上投放广告，发现抖音这个大平台，<br />抖音拥有不错的用户流量，通过数据分析预测在抖音投放公司广告，看他是否会对公司利益带来收益，这样就要分析抖音数据，抖音的用户画像，判断用户群体与公司的匹配度。<br />那么分析昵称，粉丝书，关注书，和公司有什么关系，通过分析可以知道谁的粉丝突然就增多了，谁的点赞数增多，那么这个时候就可以和广告合作，拍广告或者做营销宣传，或者根据点赞趋势，可以知道用户喜欢的视频，，这样就可以想办法把公司产品融入到用户喜欢的视频中，公关公司根据数据，找到网红黑马，进行营销包装。<br />![](https://cdn.nlark.com/yuque/0/2020/png/97322/1606533182484-b86dbe1b-5511-43d0-bcce-8ff366c3b784.png#align=left&display=inline&height=449&margin=%5Bobject%20Object%5D&originHeight=449&originWidth=983&size=0&status=done&style=none&width=983)<br /> <br />
<a name="xUUd2"></a>
## 三 粉丝数据接口
使用fiddler抓包<br />[https://aweme.snssdk.com/aweme/v1/user/follower/list/?user_id=105178085411&max_time=1556944953&count=20&offset=0&source_type=1&address_book_access=1&gps_access=1&retry_type=no_retry&iid=70850033498&device_id=67587886667&ac=wifi&channel=tengxun_new&aid=1128&app_name=aweme&version_code=600&version_name=6.0.0&device_platform=android&ssmix=a&device_type=SM-G955F&device_brand=samsung&language=zh&os_api=22&os_version=5.1.1&uuid=354730011196544&openudid=1c45444f5846a419&manifest_version_code=600&resolution=720*1280&dpi=240&update_version_code=6002&_rticket=1556944951543&mcc_mnc=46007&ts=1556944953&as=a115515c29b3bc480d4611&cp=1c3bc6579cd1ca83e1%5DcKg&mas=01f0b2caf2e5cf4c82c18f011cc8e22af58c8c6c2c260c1c2cc646](https://aweme.snssdk.com/aweme/v1/user/follower/list/?user_id=105178085411&max_time=1556944953&count=20&offset=0&source_type=1&address_book_access=1&gps_access=1&retry_type=no_retry&iid=70850033498&device_id=67587886667&ac=wifi&channel=tengxun_new&aid=1128&app_name=aweme&version_code=600&version_name=6.0.0&device_platform=android&ssmix=a&device_type=SM-G955F&device_brand=samsung&language=zh&os_api=22&os_version=5.1.1&uuid=354730011196544&openudid=1c45444f5846a419&manifest_version_code=600&resolution=720*1280&dpi=240&update_version_code=6002&_rticket=1556944951543&mcc_mnc=46007&ts=1556944953&as=a115515c29b3bc480d4611&cp=1c3bc6579cd1ca83e1%255DcKg&mas=01f0b2caf2e5cf4c82c18f011cc8e22af58c8c6c2c260c1c2cc646)<br />![](https://cdn.nlark.com/yuque/0/2020/png/97322/1606533182579-dd0ab01e-1b87-4533-8839-7d650760a9b5.png#align=left&display=inline&height=403&margin=%5Bobject%20Object%5D&originHeight=403&originWidth=1857&size=0&status=done&style=none&width=1857)<br /> 

 
<a name="CECQg"></a>
### mitmproxy中间人代理
　　发现请求头中的加密数据太多，我们换成了mitmproxy中间人代理的方式抓取数据包，并保存我们需要的数据。<br /> <br />粉丝请求url<br />https://aweme.snssdk.com/**aweme/v1/user/follower/list/**<br /> <br /> <br />编写mitproxy处理响应体数据，代码如下<br />  def response(flow):<br />      if 'aweme/v1/user/follower/list/' in flow.request.url:<br />          with open('user.txt','w') as f:<br />              f.write(flow.response.text)<br /> <br /> <br />启动 mitmdump<br /> mitmdump -s decode_douyin_fans.py -p 8889<br /> <br /> <br />手机端或者模拟器端，设置好代理，启动抖音刷粉丝页面，生成txt文件，验证，获取的数据为粉丝列表响应数据。

![](https://cdn.nlark.com/yuque/0/2020/png/97322/1606533182502-76089294-c28e-4542-a74f-bbed49c6d780.png#align=left&display=inline&height=191&margin=%5Bobject%20Object%5D&originHeight=191&originWidth=1109&size=0&status=done&style=none&width=1109)<br /> <br />数据清洗过程

![](https://cdn.nlark.com/yuque/0/2020/png/97322/1606533182613-67ba3340-aa96-41af-9f7d-c6a48d482ad2.png#align=left&display=inline&height=727&margin=%5Bobject%20Object%5D&originHeight=727&originWidth=1036&size=0&status=done&style=none&width=1036)<br /> <br />入库<br />mitmdump response拦截代码<br />import json<br />  from handle_db import mongo_info<br />  <br />  <br />  def response(flow):<br />      if 'aweme/v1/user/follower/list/' in flow.request.url:<br />          for user in json.loads(flow.response.text)['followers']:<br />              douyin_info={}<br />              douyin_info['share_id']=user['uid']<br />              douyin_info['douyin_id']=user['short_id']<br />              douyin_info['nickname']=user['nickname']<br />              mongo_info.save_task(douyin_info)<br />              # print(douyin_info)<br /> <br /> <br />`handle_db.py`数据库操作代码<br />import pymongo<br />  <br />  class Connect_mongo(object):<br />      def __init__(self):<br />          #连接数据库,如果没有数据,则创建数据库<br />          self.client=pymongo.MongoClient(host='106.12.108.236',port=27017)<br />          self.db=self.client['douyindb']<br />  <br />  <br />  <br />      def save_task(self,task):<br />          '保存粉丝信息'<br />          db_collection=self.db['douyin']<br />          #找到就更新，没找到就新增<br />          db_collection.update({'share_id':task['share_id']},task,True)<br />  <br />      def handle_get_task(self):<br />          #获取到数据，并删除数据库中的文档<br />          return self.db['douyin'].find_one_and_delete({})<br />  <br />  <br />  mongo_info=Connect_mongo()<br /> <br /> 
<a name="CGtRw"></a>
## appium实现抖音app自动滑动实现粉丝数据包的处理
<a name="5OgoU"></a>
### 1 测试是否连通,代码如下
from appium import webdriver<br />  #等待元素加载<br />  from selenium.webdriver.support.ui import WebDriverWait<br />  <br />  #定义设备参数<br />  cap={<br />      "platformName": "Android",<br />      "platformVersion": "5.1.1",<br />      "deviceName": "127.0.0.1:62025",<br />      "appPackage": "com.ss.android.ugc.aweme",<br />      "appActivity": "com.ss.android.ugc.aweme.main.MainActivity",<br />      "noReset": True,<br />      'unicodekeyboard':True, #使用unicode输入法<br />      'resetkeyboard':True, #还原输入法<br />  }<br />  <br />  driver =webdriver.Remote('http://localhost:4723/wd/hub',cap)<br /> <br /> <br />2 连通后使用uiautomator viewer,获取空间的id,xpath...表达式<br />注释事项:抓取截屏数据的时候,必须等待数据加载完毕,或者把视频暂停,否则报错<br />此过程使用了appium和uiautomator viewer交替定位元素.<br />![](https://cdn.nlark.com/yuque/0/2020/png/97322/1606533182471-3e9f6199-3219-487c-827d-d40c71b19324.png#align=left&display=inline&height=808&margin=%5Bobject%20Object%5D&originHeight=808&originWidth=1062&size=0&status=done&style=none&width=1062)<br /> 

 <br />最终appium代码,如下<br />import time<br />  <br />  from appium import webdriver<br />  #等待元素加载<br />  from selenium.webdriver.support.ui import WebDriverWait<br />  <br />  #定义设备参数<br />  cap={<br />      "platformName": "Android",<br />      "platformVersion": "6.0.1",<br />      "deviceName": "758ca8ba",<br />      "appPackage": "com.ss.android.ugc.aweme",<br />      "appActivity": "com.ss.android.ugc.aweme.main.MainActivity",<br />      "noReset": True,<br />      'unicodekeyboard':True, #使用unicode输入法<br />      'resetkeyboard':True, #还原输入法<br />  }<br />  <br />  driver =webdriver.Remote('http://localhost:4723/wd/hub',cap)<br />  <br />  def get_size():<br />      #获取设备屏幕大小<br />      x=driver.get_window_size()['width']<br />      y=driver.get_window_size()['height']<br />      return x,y<br />  <br />  #点击搜索框<br />  try:<br />      if WebDriverWait(driver, 10).until(lambda x: x.find_element_by_id("com.ss.android.ugc.aweme:id/aqz")):<br />          driver.find_element_by_id("com.ss.android.ugc.aweme:id/aqz").click()<br />  except:<br />      pass<br />  <br />  #定位搜索框<br />  try:<br />      if WebDriverWait(driver, 10).until(lambda x: x.find_element_by_xpath(<br />              "//android.widget.EditText[@resource-id='com.ss.android.ugc.aweme:id/agq']")):<br />          driver.find_element_by_xpath(<br />              "//android.widget.EditText[@resource-id='com.ss.android.ugc.aweme:id/agq']").send_keys('191433445')<br />  except:<br />      pass<br />  <br />  #点击搜索<br />  if WebDriverWait(driver,10).until(lambda x:x.find_element_by_xpath("//android.widget.TextView[@resource-id='com.ss.android.ugc.aweme:id/agt']")):<br />      driver.find_element_by_xpath("//android.widget.TextView[@resource-id='com.ss.android.ugc.aweme:id/agt']").click()<br />  <br />  #点击用户,进入个人主页<br />  if WebDriverWait(driver,10).until(lambda x:x.find_element_by_xpath("/hierarchy/android.widget.FrameLayout/android.widget.LinearLayout/android.widget.FrameLayout/android.widget.FrameLayout/android.widget.FrameLayout[2]/android.widget.RelativeLayout/android.widget.FrameLayout/android.widget.RelativeLayout/android.support.v4.view.ViewPager/android.widget.LinearLayout/android.widget.FrameLayout/android.view.ViewGroup/android.support.v7.widget.RecyclerView/android.widget.LinearLayout[1]/android.widget.LinearLayout[1]/android.support.v7.widget.RecyclerView/android.widget.RelativeLayout/android.widget.FrameLayout/android.widget.ImageView[2]")):<br />      el4 = driver.find_element_by_xpath("/hierarchy/android.widget.FrameLayout/android.widget.LinearLayout/android.widget.FrameLayout/android.widget.FrameLayout/android.widget.FrameLayout[2]/android.widget.RelativeLayout/android.widget.FrameLayout/android.widget.RelativeLayout/android.support.v4.view.ViewPager/android.widget.LinearLayout/android.widget.FrameLayout/android.view.ViewGroup/android.support.v7.widget.RecyclerView/android.widget.LinearLayout[1]/android.widget.LinearLayout[1]/android.support.v7.widget.RecyclerView/android.widget.RelativeLayout/android.widget.FrameLayout/android.widget.ImageView[2]")<br />      el4.click()<br />  <br />  # 点击粉丝<br />  if WebDriverWait(driver,10).until(lambda x:x.find_element_by_xpath("//android.widget.TextView[@text='粉丝']")):<br />      driver.find_element_by_xpath("//android.widget.TextView[@text='粉丝']").click()<br />  <br />  # 等待页面刷新<br />  time.sleep(3)<br />  <br />  #开始滑动<br />  l =get_size()<br />  x1=int(l[0]*0.5)<br />  y1=int(l[1]*0.9)<br />  y2=int(l[1]*0.25)<br />  <br />  while True:<br />      if '没有更多了' in driver.page_source:<br />          break<br />      else:<br />          #初始位置x1,y1   结束位置为:x1,y2<br />          driver.swipe(x1,y1,x1,y2)<br />          time.sleep(0.2)<br /> <br /> <br /> <br />**电脑开wifi给手机用,使用mitmdump开代理,有点坑,ip找这里,我用的是网线!!!**

![](https://cdn.nlark.com/yuque/0/2020/png/97322/1606533182572-61a11449-b988-4695-9de4-22a3fc1450df.png#align=left&display=inline&height=140&margin=%5Bobject%20Object%5D&originHeight=140&originWidth=599&size=0&status=done&style=none&width=599)<br /> 
<a name="SwJCb"></a>
### 代码执行顺序
1 启动弄mitmproxy<br />mitmdump -p 8889 -s decode_douyin_fans.py<br /> <br />2 启动appnium程序,把粉丝信息入库<br />python handle_appium_douyin.py<br /> <br />3 执行采集分享id对应用户的详细信息,并入库<br />python handle_share_web.py<br /> 
<a name="vWida"></a>
### 多任务端
<br />![](https://cdn.nlark.com/yuque/0/2020/png/97322/1606533182609-71325596-b5d3-4c2c-a331-1566f884f2dd.png#align=left&display=inline&height=146&margin=%5Bobject%20Object%5D&originHeight=146&originWidth=414&size=0&status=done&style=none&width=414)<br /> <br />注意事项<br />查找模拟器对应端口,找到对应的设备.<br />![](https://cdn.nlark.com/yuque/0/2020/png/97322/1606533182590-25441a82-b9bb-43e7-8001-b8a3419fae34.png#align=left&display=inline&height=168&margin=%5Bobject%20Object%5D&originHeight=168&originWidth=625&size=0&status=done&style=none&width=625)<br />![](https://cdn.nlark.com/yuque/0/2020/png/97322/1606533182467-20a637ad-5e86-4066-ae23-ede7a5cd50ff.png#align=left&display=inline&height=141&margin=%5Bobject%20Object%5D&originHeight=141&originWidth=394&size=0&status=done&style=none&width=394)<br /> <br />![](https://cdn.nlark.com/yuque/0/2020/png/97322/1606533182491-4ac4ef1e-f212-4012-89d3-d4783f65ae01.png#align=left&display=inline&height=266&margin=%5Bobject%20Object%5D&originHeight=266&originWidth=753&size=0&status=done&style=none&width=753)<br />![](https://cdn.nlark.com/yuque/0/2020/png/97322/1606533182696-146fdbd1-eb4a-492d-9cf6-78453a1d582c.png#align=left&display=inline&height=541&margin=%5Bobject%20Object%5D&originHeight=541&originWidth=1388&size=0&status=done&style=none&width=1388)<br /> 

没有粉丝处理<br />![](https://cdn.nlark.com/yuque/0/2020/png/97322/1606533182568-51865688-477b-425a-9f26-ba4054d7cad0.png#align=left&display=inline&height=247&margin=%5Bobject%20Object%5D&originHeight=247&originWidth=666&size=0&status=done&style=none&width=666)<br /> 

返回,清空,重新输入<br /> <br /> <br />封装兼容不同设备函数<br />![](https://cdn.nlark.com/yuque/0/2020/png/97322/1606533182495-4d597380-79a0-4927-b6f7-07ee832a70c3.png#align=left&display=inline&height=309&margin=%5Bobject%20Object%5D&originHeight=309&originWidth=810&size=0&status=done&style=none&width=810)<br /> 

 <br />启动多进程

![](https://cdn.nlark.com/yuque/0/2020/png/97322/1606533182617-26ab7a9d-646d-4dd5-bedd-07af0ea9cae7.png#align=left&display=inline&height=330&margin=%5Bobject%20Object%5D&originHeight=330&originWidth=968&size=0&status=done&style=none&width=968)<br /> <br /> <br />伪装爬虫<br />![](https://cdn.nlark.com/yuque/0/2020/png/97322/1606533182949-b59914b4-be27-4296-bee7-1bcbcd17724b.png#align=left&display=inline&height=54&margin=%5Bobject%20Object%5D&originHeight=54&originWidth=662&size=0&status=done&style=none&width=662)<br /> 

 
<a name="Mjzs8"></a>
## 抖音视频下载


网页源码中找到加密js

![](https://cdn.nlark.com/yuque/0/2020/png/97322/1606533182562-fc64c8da-3c7f-49c8-8d62-02aeaed79835.png#align=left&display=inline&height=357&margin=%5Bobject%20Object%5D&originHeight=357&originWidth=960&size=0&status=done&style=none&width=960)<br /> <br />请求参数分析 max_cursor<br />![](https://cdn.nlark.com/yuque/0/2020/png/97322/1606533182470-d55cd9e1-a5eb-43d9-90a5-b942180c9b01.png#align=left&display=inline&height=236&margin=%5Bobject%20Object%5D&originHeight=236&originWidth=591&size=0&status=done&style=none&width=591)<br />网页中加的dytk<br />![](https://cdn.nlark.com/yuque/0/2020/png/97322/1606533182462-fc096628-ce97-450e-b708-b8aa9128ee44.png#align=left&display=inline&height=212&margin=%5Bobject%20Object%5D&originHeight=212&originWidth=877&size=0&status=done&style=none&width=877)<br /> <br /> 
<a name="xqN9K"></a>
### 解密signature找寻流程
网页张找到js加密文件,全局搜索(点击右侧三个点,或者ctrl +f)<br />![](https://cdn.nlark.com/yuque/0/2020/png/97322/1606533182510-1bc4d5e8-499d-4a72-8b25-437dc9e44ebd.png#align=left&display=inline&height=163&margin=%5Bobject%20Object%5D&originHeight=163&originWidth=1279&size=0&status=done&style=none&width=1279)<br /> <br />继续找<br />![](https://cdn.nlark.com/yuque/0/2020/png/97322/1606533182537-e39e6b0a-b599-4f73-bd03-bed72ab91a0e.png#align=left&display=inline&height=303&margin=%5Bobject%20Object%5D&originHeight=303&originWidth=493&size=0&status=done&style=none&width=493)<br /> <br /> <br />![](https://cdn.nlark.com/yuque/0/2020/png/97322/1606533182601-45666cc8-1f55-4a6a-aec1-c10c25fa38aa.png#align=left&display=inline&height=182&margin=%5Bobject%20Object%5D&originHeight=182&originWidth=476&size=0&status=done&style=none&width=476)<br /> <br />一层一层的找,找到所有的流程<br /> 

<a name="0qfYR"></a>
### 找完之后,开始构建html文件和js文件
html 头部文件

![](https://cdn.nlark.com/yuque/0/2020/png/97322/1606533182528-07a7ffec-3fe4-426e-9fcc-a2ed5208740a.png#align=left&display=inline&height=321&margin=%5Bobject%20Object%5D&originHeight=321&originWidth=541&size=0&status=done&style=none&width=541)<br /> <br />html尾部文件(复制调用的js函数到脚部文件)<br />![](https://cdn.nlark.com/yuque/0/2020/png/97322/1606533182688-20719874-7066-4b01-9ace-55f6585ffb55.png#align=left&display=inline&height=455&margin=%5Bobject%20Object%5D&originHeight=455&originWidth=791&size=0&status=done&style=none&width=791)<br /> <br /> <br />使用文件合并的方法,生成html文件<br />![](https://cdn.nlark.com/yuque/0/2020/png/97322/1606533182598-a0ad20c9-6a1a-4719-af8d-de42c83f8550.png#align=left&display=inline&height=272&margin=%5Bobject%20Object%5D&originHeight=272&originWidth=728&size=0&status=done&style=none&width=728)<br /> <br /> <br />每次打开生成的html 自动生成signature 秘钥<br />![](https://cdn.nlark.com/yuque/0/2020/png/97322/1606533182648-bcc67e7c-eb0b-43a8-a02d-73cf94b8f964.png#align=left&display=inline&height=366&margin=%5Bobject%20Object%5D&originHeight=366&originWidth=869&size=0&status=done&style=none&width=869)<br /> <br /> 

<a name="0gnyg"></a>
### js 破解思路:
　　使用开发者工具,全局搜索加密字段所在的文件,理清楚加密过程,即执行流程,自己构建html文件,模拟加密执行函数,无需自己写,只需要拷贝js代码.<br /> 
<a name="iQaxl"></a>
## 项目总结
<a name="w9KG5"></a>
### 个人数据 --TTF 字体混淆
![](https://cdn.nlark.com/yuque/0/2020/png/97322/1606533182665-a8e3e604-330f-4a9c-a29d-e2c17a04d0c0.png#align=left&display=inline&height=158&margin=%5Bobject%20Object%5D&originHeight=158&originWidth=521&size=0&status=done&style=none&width=521)<br /> 

 <br />fontedit解析字体,找到映射关系表

![](https://cdn.nlark.com/yuque/0/2020/png/97322/1606533182786-98460cdc-e5f0-4538-b1e6-eff7614ca515.png#align=left&display=inline&height=676&margin=%5Bobject%20Object%5D&originHeight=676&originWidth=874&size=0&status=done&style=none&width=874)<br /> <br />映射表,正则替换

![](https://cdn.nlark.com/yuque/0/2020/png/97322/1606533182613-6e72c7b4-431e-4f4c-8cc5-f3366c10cdf1.png#align=left&display=inline&height=339&margin=%5Bobject%20Object%5D&originHeight=339&originWidth=847&size=0&status=done&style=none&width=847)<br /> 
<a name="p7fBk"></a>
### 粉丝数据抓取
<a name="5y0nP"></a>
#### appium模拟滑动+mitmdump解析数据
 <br />使用了uiautomator viewer和appium结合进行元素定位,进行元素的操作,使用mitmdump中间人攻击技术,在请求分数数据的时候,把响应数据进行解析,入库.<br /> 
<a name="in8wf"></a>
### 视频抓取阶段
编写破解文件

![](https://cdn.nlark.com/yuque/0/2020/png/97322/1606533182639-2fa213f6-f3fe-4296-b75a-b3c641adfac1.png#align=left&display=inline&height=226&margin=%5Bobject%20Object%5D&originHeight=226&originWidth=385&size=0&status=done&style=none&width=385)<br /> <br />注意:<br />1 ttf字体数据对应关系,如果关系更改了,爬虫就需要做响应的更改.<br />
<br />2 一般一个名人最多获取5000条粉丝数据<br />
<br />3 移动设备设置代理抓包后,如果无法联网或无法解析https数据时候,需要安装xpose框架+justtrustme组件进行屏蔽证书强校验.<br />
<br />4 在设置多设备、多进程数据抓取时候,需要设置appium服务器端的bootstrap端口,以及客户端的udid字段.<br />
<br />5 视频数据抓取,需要破解signature字段,并使用html文件,解析js,<br />
<br />拼接html文件,逆退出加密过程.<br />
<br />6 数据抓取需要加上代理,伪装爬虫.<br />
<br />7 最好使用真实的移动设备.<br />
<br />


---

<a name="23c4a2b4"></a>
#### TiToData：专业的短视频数据接口服务平台。
<a name="f12e0c02"></a>
#### 更多信息请联系： [TiToData](https://www.titodata.com/about?from=douyinapi)
覆盖主流平台：抖音，快手，小红书，TikTok，Zynn，YouTube，1688，拼多多，淘宝，美团，饿了么，淘宝，微博
