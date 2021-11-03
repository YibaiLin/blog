---
title: python 数据分析
date: 2020-12-01 23:00:04
categories: 
- [python]
- [economics]
tags: [pandas]

---

**人生苦短，让我们学一点python**

希望大家不要在意我取这样一个自媒体吸引眼球式的题目。言归正传。

python是一门简单的编程语言，千万不要被“编程”这个词语唬住，所谓“编程”，其实就是把你想让电脑帮你做的事情，通过计算机听得懂的语言，即一些非常简单的英文单词按照固定的语法格式表述出来即可，所以千万不要看到代码就“头大”。从零基础到初步掌握这个小工具，一周时间足矣。No kidding!

我不打算介绍python的来历，也不打算介绍python的语法。我想通过给大家分享四个具体的例子和使用场景，展示下python对于我们经济学专业的学生而言可能有什么帮助。

<!--more-->

##### 一、数据处理：以中国全球投资追踪数据库为例

在我们研究一个专业问题时，数据往往是我们了解问题特征的依据和展开研究的第一步程序。对于一些数据量极大、类别极多的数据，处理起来可能会比较费时费力，这种时候，我们可以使用python来编写程序，帮助我们灵活地操作这些数据以得到我们想要的统计结果。

以中国全球投资追踪数据库（China Global Investment Tracker ）为例，CGIT数据库是美国企业研究所发布的记录中国海外投资和对外承包工程的微观数据库，该数据库统计了自2005年至今共3400多笔金额在1亿美元以上的投资记录，包括每笔投资的投资总额、中国的母公司、东道国、投资所属行业等信息。数据的excel是这样的：

![CGIT](https://image.xiuwujinda.cn/2020/12/01/CGIT.png)

可以看到，总计有3444条记录数据。假如我们需要分析2005-2019年中国对外投资和承包工程每一年每个行业的投资额是多少，以观察行业投资趋势。即，我们期望得到下面这样格式的汇总数据：

![model](https://image.xiuwujinda.cn/2020/12/01/model.png)

来看下python如何实现，完整代码如下：

```python
import pandas as pd

# 从excel提取数据
data = pd.read_excel('./data/test.xlsx')
data.drop(columns=['BRI'], inplace=True)

# 按年份、行业归总数据
df = data.groupby(['Year', 'Sector']).sum().unstack(-1, 0)

# 将结果保存至新的excel表
df.to_excel('./data/分行业CGIT.xlsx')

```

核心代码只需一行：

```python
data.groupby(['Year', 'Sector']).sum().unstack(-1, 0)

```

便可实现我们需要的结果，即将3444条数据按年份、行业分类加总（如下图所示）。是不是看着单词就明白什么意思了？这条指令告诉计算机去处理“data”对象，先依据“Year”和“Sector”进行“groupby”即分组处理，再“sum”分组加总后，最后“unstack”展开。

![sector](https://image.xiuwujinda.cn/2020/12/01/sector.png)

##### 二、画图

金刚和沈坤荣（2019）的研究驳斥了“一带一路”是债务陷阱的说法，文中利用CGIT数据库分析了中国海外投资各行业的时间趋势，并展示如下图所示的结果：

![sp](https://image.xiuwujinda.cn/2020/12/01/sp.png)

金刚和沈坤荣是使用stata代码制作的上图，同样的，利用前面处理过的数据，我们也可以使用python制作类似的图。绘图代码如下：

```python
# 开始画图
fig, axs = plt.subplots(2, 4, figsize=(15, 7.5)) # 指定画布布局，2行4列，共8幅子图
axs_flat = axs.flat

for i, ax in enumerate(axs_flat):
    sector = sectors[i]
    label = '(' + char[i] + ') ' + sector
    ax.set_xlim(2004, 2020) # 设置x轴范围
    ax.set_xticks(np.arange(2006, 2022, 4)) # 设置x轴刻度
    ax.set_xlabel(label) # 设置x轴标题
    ax.set_ylim(0, 1000) # 设置y轴标题
    ax.set_yticks(np.arange(0, 1000, 200)) # 设置y轴刻度
    ax.annotate('(100Million $)', xy=(-0.2, 1.02), xycoords='axes fraction') # 标注单位
    ax.plot('Year', sector, data=df, marker='s', color='black', markersize=2) # 绘图命令

# 保存图片
fig.savefig('../pictures/分行业CGIT.png', dpi=120)

```

示例效果图如下所示：

![分行业CGIT](https://image.xiuwujinda.cn/2020/12/01/分行业CGIT.png)

当然，还可以继续个性化调整该图片的显示效果。对图片上的所有可视元素均可以进行操作调整，这正是python作图的灵活性的体现。

##### 三、爬取数据

自动化是计算机程序最大的魅力所在。`https://outlook.gihub.org/` 这个网站可以下载《全球基础设施展望报告》中的国别数据，但是每次只能下载一个国家的数据。比如下载中国的基建数据（如下图所示），可以得到包含七个基建行业的自2007年至2040年的数据，有三个子表：Investment，Needs 和 Gap。其中，Investment子表包含按当前中国实际投资额的趋势进行预测的年度投资额；Needs指按最佳效益测算中国每年需要多少基础设施投资；Gap指Needs和Investment的差额。

![china](https://image.xiuwujinda.cn/2020/12/01/china.png)

但是，该报告共有56个国家的数据，如果每次下载一个国家的数据，汇总起来是比较麻烦的，仅仅是进行56次下载这个过程本身就很麻烦。这个时候，python便很适合用来完成这些重复性的操作，核心伪代码如下：

```python
for i, country in enumerate(countries): # 遍历国家列表，循环56次下载
    response = requests.get(url, params={'countries': country})  # 发送http请求，进行一次下载
    data.append(process(response)) # 从下载得到的数据中提取需要的数据并处理

data.to_excel(filename) # 将结果保存至excel
```

上面伪代码的思路很直白，即发起一次下载网络请求，收到数据后进行处理，然后循环56次。运行程序，得到最终的结果如下：

![coutries](https://image.xiuwujinda.cn/2020/12/01/coutries.png)

事实上，越是有规律且重复次数越多的数据获取或数据操作的工作，越适合用python来处理。比如，查询城市的经纬度，可以通过Google Earth 人工一个个地查询，如下图所示查询西安市的经纬度：

![google earth](https://image.xiuwujinda.cn/2020/12/01/google-earth.png)

但当查询城市数量是1000个时，人工查询是极其枯燥且浪费时间的，而通过python程序，只需要将一次查询的步骤用程序来表示，剩下的999次，通过循环，程序自己便可以完成。核心代码如下：

```python
for j, city in enumerate(cities_cn): # 循环1000次，遍历每个城市
    r = requests.get(url, params={'address': city}) # 一次查询，访问百度地图api查询
    result = r.json()['result']
    record = [cities[j], city, result['location']['lng'], result['location']['lat'], result['confidence']] # 从查询结果提取经纬度信息
    poi.append(record) # 将信息保存至缓存中
    if j in holdon:
        print('sleelp for 2 seconds...')
        time.sleep(2) # 每10次查询后，暂停2秒再继续查询
    
poi_df = pd.DataFrame(poi, columns=['city', 'city_cn', 'lng', 'lat', 'confidence'])
poi_df.to_excel(dest, index=False) # 将最终结果保存至excel表中
```

结果如下：

![gis](https://image.xiuwujinda.cn/2020/12/01/gis.png)

相较于用1000次重复的人工操作而言，一个python脚本可谓是 lifesaver 了。

##### 总结

学习使用python，对于我们学术研究来说当然不是必须的，一方面可能数据量根本不值当通过编写一段程序去处理，另一方面从零基础开始学习还是需要一定的时间成本。不过，python及其背后的开源世界所提供的各种科学分析工具，在大批量数据的处理、灵活多变的作图、通过互联网爬取大量数据上是很有帮助的，此外，python也可以代替stata用于计量分析。因此，我认为python值得我们花一点时间去探索和了解。希望通过本文这几个具体且实际的例子，可以激发各位了解这个小工具的兴趣，建议感兴趣的同学可以尝试去学习下这门编程语言。

##### 学习资源

python语言学习：https://www.liaoxuefeng.com/wiki/1016959663602400/1018138095494592

数据处理包 pandas：https://www.pypandas.cn/docs/getting_started/overview.html

数据处理包 numpy：https://www.numpy.org.cn/article/

画图包 matplotlib：https://matplotlib.org/index.html

##### 参考文献

[1] 金刚,沈坤荣.中国企业对“一带一路”沿线国家的交通投资效应:发展效应还是债务陷阱[J].中国工业经济,2019(09):79-97.