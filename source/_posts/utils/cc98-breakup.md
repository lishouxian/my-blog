---
title: '浙江大学校内论坛CC98中分手贴发帖量的分析'
date: 2020-07-15 20:50:18
tags: [python,数据分析]
categories: utils
published: true
hideInList: false
feature: 
isTop: false
---

整天在校内论坛划水，导致了我总想做些什么。

同时最近98中分手贴的数量暴增，导致我想分析下发帖量和时间的关系

<!-- more -->

```python
from bs4 import BeautifulSoup

file = open("./happy.htm", "rb")

html = file.read()

bs = BeautifulSoup(html, "html.parser")

title_list = bs.find_all(class_="focus-topic-title")

data_list = bs.find_all(class_="focus-topic-middle")

save = []

for i in range(len(title_list)):

    if not "分手厨房" in title_list[i].string:

        save.append(data_list[i].div.div.next_sibling.text)

        #save.append([title_list[i].string,data_list[i].div.div.next_sibling.text])

data = [0 for i in range(120)]

month = [0 for i in range(12)]

for i in range(3,len(save)):

    month[int(save[i][5:7])-1] += 1

    timeline = int(save[i][2:4]) * 12 + int(save[i][5:7])-128

    if 0 < timeline < 120:

        data[timeline] += 1

import matplotlib.pyplot as plt

plt.plot(range(1,13),month)

plt.show()

plt.plot(range(-29,1),data[-30:])

plt.show()
```

[文件下载](http://file.cc98.org/v2-upload/ihzvqxp0.htm)

**大家也可以下载到本地分析自己感兴趣的部分**

楼主主要分析了每天各小时，每年各月，以及由现在往前推30个月的发帖量情况

![](https://blog-1251782526.cos.ap-shanghai.myqcloud.com/uPic/007S8ZIlly1ggryid70snj30hs0dc0tb.jpg)

晚上9点开始是“分手贴”爆发的时候

![](https://blog-1251782526.cos.ap-shanghai.myqcloud.com/uPic/007S8ZIlly1ggryidzs4hj30hs0dcmxp.jpg)

提到分手比较多的月份是5月6月12月

![](https://blog-1251782526.cos.ap-shanghai.myqcloud.com/uPic/007S8ZIlly1ggryidleavj30hs0dcwf7.jpg)

过去两个学年。。。疫情真的很厉害

下面是绘图的代码

```python
figure, ax = plt.subplots()

font1 = {'family' : 'Times New Roman',

		 'weight' : 'normal',

		 'size' : 14,

		 }

month = [ "Aug", "Sep", "Oct", "Nov", "Dec","Jan", "Feb", "Mar", "Apr", "May", "Jun", "Jul"]

plt.plot(month,data[-12:],label='2019-2020',linestyle='--',color='r',marker='D')

plt.plot(month,data[-24:-12],label='2018-2019',linestyle='--',color='b',marker='o')


labels = ax.get_xticklabels() + ax.get_yticklabels()

[label.set_fontname('Times New Roman') for label in labels]

plt.xlabel('Month',font1)

plt.ylabel('Posts',font1)

plt.title('balabala',font1)

plt.legend(prop=font1,loc=2)

plt.grid(axis="y")

plt.show()
```

