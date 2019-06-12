### 合併txt：

-----


This should do it

**For large files:**

```py
filenames = ['file1.txt', 'file2.txt', ...]
with open('path/to/output/file', 'w') as outfile:
    for fname in filenames:
        with open(fname) as infile:
            for line in infile:
                outfile.write(line)
```

**For small files:**

```py
filenames = ['file1.txt', 'file2.txt', ...]
with open('path/to/output/file', 'w') as outfile:
    for fname in filenames:
        with open(fname) as infile:
            outfile.write(infile.read())
```

**… and another interesting one that I thought of**:

```py
filenames = ['file1.txt', 'file2.txt', ...]
with open('path/to/output/file', 'w') as outfile:
    for line in itertools.chain.from_iterable(itertools.imap(open, filnames)):
        outfile.write(line)
```





### Crawler

---------

####  "HTTP Error 403: Forbidden" 

##### 分析：

之所以出現上面的異常,是因為如果用urllib.request.urlopen 方式打開一個URL，服務器端只會收到一個單純的對於該頁面訪問的請求，但是服務器並不知道發送這個請求使用的瀏覽器、操作系統、硬件平台等信息。而缺失這些信息的請求往往都是非正常的訪問。例如爬蟲。

有些網站為了防止這種非正常的訪問，會驗證請求信息中的UserAgent(它的信息包括硬件平台、系統軟件、應用軟件和用戶個人偏好)，如果UserAgent存在異常或者是不存在，那麼這次請求將會被拒絕(如上錯誤信息所示) 所以可以嘗試在請求中加入UserAgent的信息

抓Billboard排行時遇到此問題，用以下方法解決，

```python
def url_to_soup(url):
    #for act like a browser , not crawler
    headers = {'User-Agent':'Mozilla/5.0 (Windows NT 6.1; WOW64; rv:23.0) Gecko/20100101 Firefox/23.0'}
    req = urllib.request.Request(url=url, headers=headers)
    html = urllib.request.urlopen(req).read()
#     html = urlopen(url).read()
    
    soup = BeautifulSoup( html , "html.parser")
    return soup
```



##### soup抓取tag ＆ class：

要分清楚find_all() & find() 的差別，fina_all 出來可能會是一個list，這裡Result type = bs4 object，要印出find_all找的東西，就必須用FOR迴圈遍歷一次，而剛好需要的歌名就在title這個屬性內，所以可以直接`tab['title']`，以此得到title的text。

關於這邊取text值部分，每次都會不一樣，要分清楚。

若是正常html，直接find_all給他爆下去也可以。

```python
# kkbox
''' html
<a class="song-title" href="/tw/tc/song/mdtJsGs0F-Y-X3XKd3XKd0P4-index.html" title="快樂天堂">
                            快樂天堂
                    </a>
'''
url = "https://www.kkbox.com/tw/tc/playlist/8tnKNwashJlCdD1JTb"
soup = url_to_soup(url)
Result = soup.find_all("a", class_= "song-title")
for tab in Result:
    with open('song_title.txt','a+') as f:
        f.write(tab['title']+'\n')
```

另一個爬billboard:

一開始不知道怎麼`get_text()`，一直.text都不能。用`strip('\n')` 有些換行沒辦法去除乾淨，用`replace('a','b') #use b to replace a`才能完全去除。

這裡是因為歌名、歌手都在同一個div下，先找此div的class，然後每一個div物件進去看，分別用FIND來找，因為只有一個目標物件。

```python
#billboard
url = "https://www.billboard.com/charts/hot-100/1970-06-27"
soup = url_to_soup(url)
song = soup.find_all("div",class_="chart-list-item__text ")
dic = {}
for line in song:
    # use get_text 取得純文字， strip去除換行
    title = line.find('span',class_="chart-list-item__title-text").get_text().replace('\n','')
    artist = line.find('div', class_="chart-list-item__artist").get_text().replace('\n','')
    dic.setdefault(title,artist)
#     dic.append({'song':song_title,'artist':artist})
print(dic)
for i,v in dic.items():
    print(i+' : ',v)
```



##### 爬html表格 `<tr> <td>`

如下表，要找每列，也就是每個 tr，所以`find_all('tr')` 是找出每個row的東西，而`find_all('td')` 則是此row 裡面的每一col，在看是第幾行來抓取需要資料，ex `tds[1].content[0]` 則是抓第一行的資料

```python
'''
<table>
	<tbody>
		<tr>
			<td> song </td>
			<td> singer </td>
		</tr>
		<tr> .... </tr>
'''
# for hit fm 2001-now
url = " https://tw-pop-chart.blogspot.com/2014/01/hit-fm-2011"
data_list = []
soup = url_to_soup(url)
Result = soup.find_all('tr')
for idx, tr in enumerate(soup.find_all('tr')):
    tds = tr.find_all('td')
    data_list.append(tds[1].contents[0])

print(data_list)
```

後面這是爬billboard榜，找很久的href及每列不一定都有相同行數，也就是不規則表格時要怎麼爬。

一樣先找tr來看每個row，但每個row的td不一定有固定數量的td，也就是col數。所以設定為 `len(col)>1`時再來抓。

一樣在取得資料時，務必注意是第幾行，我在col[0] ,col[1]找錯很久，一直col[1]找不到有href，想說怎麼那個怪，一直不能`get('href')`，結果發現是在col[0]，改對了後就馬上可以了。

更新：

發現可以用find，但是要先找到 `'a'` 才能用 `get('href')`，因為這邊html 為 `<a href = "/charts/hot-100/2010-01">`

> 然後這邊不知道為何`find('href')` 不能，明明只有第一個col有一個href。但用find_all後再用get就可以了



```python
# billboard
head = " https://www.billboard.com/archive/charts/"
# year = 2001
data_list = []
singer_list = []
dic = {}
for year in range(2010,2019):
#     url= "https://www.billboard.com/archive/charts/1975/hot-100"
    url = head + str(year) + "/hot-100"
    print(url)
    soup = url_to_soup(url)
    Result = soup.find_all('tr')

    for idx, tr in enumerate(soup.find_all('tr')):
        col = tr.find_all('td')
#         print(col)
        if(len(col)>1): #會有少行，所以檢查後再find
#             print(col[0])
#====================發現以下這樣也可以，可以註解掉後面迴圈＝＝ ===========
						test = col[0].find('a').get('href')
            print(test)
#===================================================================
    
            for link in col[0].find_all('a',href = True): 
    						#搞很久，結果發現是col[0], not col[1] QQ
                href = link.get('href')
                getdetail(href , dic)
                time.sleep(1)
                print(href)
# print(dic)
```





### 網頁參考 bs4 find / fina_all

---

<https://zhuanlan.zhihu.com/p/35354532>

本文分为如下几个部分

- 解析html流程说明
- 识别标签
- 提取内容
- 查看标签信息
- 其他提取细节

## **解析html流程说明**

爬取网页信息，可以理解成从html代码中抽取我们需要的信息，而html代码由众多标签组成，我们要做的就分为两个部分

- 精确定位标签
- 从标签中提取内容

先说第二个，在找到对应标签位置后，我们要的信息一般会存储在标签的两个位置中

- 标签中的内容
- 属性值

比如下面这行标签

```text
<p><a href='www.wenzi.com'>文字</a></p>
```

一般要提取“文字”部分，或者那个链接`www.wenzi.com`部分

那么如何精确定位到标签呢？只能依靠标签名和标签属性来识别。下面一个例子提供识别的大致思路，之后会具体总结

```html
<title>标题</title>
<body>
    <ul class='list1'>
        <li>列表1第1项</li>
        <li>列表1第2项</li>
    </ul>
    <p class='first'>文字1</p>
    <p class='second'>文字2</p>
    <ul class='list2'>
        <li>列表2第1项</li>
        <li>列表2第2项</li>
    </ul>
</body>
```

- 如果要提取“标题”，只需要使用`title`标签名来识别，因为只出现过一次`title`标签
- 如果要提取“文字1”，不能只使用`p`标签，因为“文字2”也对应了`p`标签，所以要用`p`标签且`class`属性是`'second'`来识别
- 如果“文字1”和“文字2”都要，就可以循环获取所有`p`标签提取内容
- 如果想提取列表1中的两项，就不能直接循环获取`li`标签了，因为列表2中也有`li`标签。此时需要先识别其父节点，先定位到`<ul class='list1'>`这个标签上（通过`ul`标签和`class`属性是`list1`定位）。此时在这个标签里，所有`li`都是我们想要的了，直接循环获取就可以了

这里展示一下使用beautifulsoup实现上述提取的代码，先对这个库的提取思路有一个大概的印象

```python3
a = '''<title>标题</title>
<body>
    <ul class='list1'>
        <li>列表1第1项</li>
        <li>列表1第2项</li>
    </ul>
    <p class='first'>文字1</p>
    <p class='second'>文字2</p>
    <ul class='list2'>
        <li>列表2第1项</li>
        <li>列表2第2项</li>
    </ul>
</body>'''

from bs4 import BeautifulSoup
soup = BeautifulSoup(a, "html.parser")
soup.title.text # '标题'
soup.find('p', attrs={'class':'first'}).text # '文字1'
soup.find_all('p') # [<p class="first">文字1</p>, <p class="second">文字2</p>], 再分别从中提取文字
soup.find('ul', attrs={'class':'list1'}).find_all('li') # [<li>列表1第1项</li>, <li>列表1第2项</li>]
```

识别的大致思路就是这样。

从细节上看，一个完整解析库需要实现三个部分的功能

**从提取内容上看**

- 提取标签内容
- 提取标签属性值

**从识别标签上看**

- 只根据标签来识别

- - 找到名为a的标签（只有一个a标签时）
  - 找到所有名为a的标签
  - 找到名为a或b的标签
  - 根据正则表达式提取标签

- 同时根据标签和属性识别

- - （要求在能识别标签的前提下，能相似地识别属性）比如
  - 找到 标签名为a且B属性的值是b 的标签
  - 找到 标签名为a且B属性的值是b且C属性是c 的标签
  - 找到 标签名为a且B属性的值是b或c 的标签
  - 找到 标签名为a且（不）包含B属性 的标签
  - 找到 标签名为a且B属性的值包含b字符串（各种正则式匹配） 的标签

- 根据标签内内容来识别，也是类似如上划分，最好能用正则表达式匹配内容

- 根据位置识别

- - 找到 第i个a标签
  - 找到 第i个和第j个a标签

**提供一些查看当前标签信息的方法，方便调试**

- 获得标签名
- 获得标签所有属性及值
- 检查标签是否有某属性

**除此之外这个库还有一些提取细节**

- 多层嵌套标签问题
- `find_all`的简写方法
- `find_all`的其他参数

下面按照上述实现的功能顺序来讲解

(其实到最后会发现这个库唯一一个要重点掌握的方法是`find_all`)

首先加载库

```text
from bs4 import BeautifulSoup
```

## **识别标签**

本节分为如下几个部分

- 只根据标签来识别
- 根据标签和属性识别
- 根据标签内内容来识别
- 根据位置识别

**只根据标签来识别**

这部分分为如下几种情况

- 找到名为a的标签（查找唯一标签）
- 找到所有名为a的标签
- 找到名为a或b的标签
- 根据正则表达式提取标签

查找唯一标签时，beautifulsoup库中提供三种方法

- 对象的属性调用，直接提取该名字的标签，但是如果有很多该标签只能找到第一个
- `find`方法，也只能找到第一个
- `find_all`方法，找到所有该标签，返回一个list，如果只找到一个也是返回list，用`[0]`提取

```python3
a = '''
<h1>标题1</h1>
<h2>标题2</h2>
<h2>标题3</h2>
'''

# 将提取到的字符串转化为beautifulsoup的对象
soup = BeautifulSoup(a, "html.parser")

# 提取唯一标签
soup.h1
soup.find('h1')
soup.find_all('h1')[0]
# 上面三条结果都是
# <h1>标题1</h1>
```

如果要找到所有某标签，或者某几种标签及根据正则表达式提取，只能用`find_all`，返回一个列表

```python3
soup.find_all('h2')
# [<h2>标题2</h2>, <h2>标题3</h2>]

soup.find_all(['h1','h2'])
# [<h1>标题1</h1>, <h2>标题2</h2>, <h2>标题3</h2>]

# 使用正则表达式
import re
soup.find_all(re.compile('^h'))
# [<h1>标题1</h1>, <h2>标题2</h2>, <h2>标题3</h2>]
```

**根据标签和属性识别**

这部分分为如下几种情况

- （要求在能识别标签的前提下，能用相似的方法识别属性）比如
- 找到 标签名为a且B属性的值是b 的标签
- 找到 标签名为a且B属性的值是b且C属性是c 的标签
- 找到 标签名为a且B属性的值是b或c 的标签
- 找到 标签名为a且（不）包含B属性 的标签
- 找到 标签名为a且B属性的值包含b字符串（各种正则式匹配） 的标签

要参考属性提取标签时，只有`find`和`find_all`两种方法，`find`总是返回找到的第一个，而`find_all`会返回所有，如果想要第一个就提取即可，因此这里全用`find_all`来讲，其实只是加一个`attrs`参数

```python3
a = '''
<p id='p1'>段落1</p>
<p id='p2'>段落2</p>
<p class='p3'>段落3</p>
<p class='p3' id='pp'>段落4</p>
'''

soup = BeautifulSoup(a, "html.parser")

# 第一种，直接将属性名作为参数名，但是有些属性不行，比如像a-b这样的属性
soup.find_all('p', id = 'p1') # 一般情况
soup.find_all('p', class_='p3') # class是保留字比较特殊，需要后面加一个_

# 最通用的方法
soup.find_all('p', attrs={'class':'p3'}) # 包含这个属性就算，而不是只有这个属性
soup.find_all('p', attrs={'class':'p3','id':'pp'}) # 使用多个属性匹配
soup.find_all('p', attrs={'class':'p3','id':False}) # 指定不能有某个属性
soup.find_all('p', attrs={'id':['p1','p2']}) # 属性值是p1或p2

# 正则表达式匹配
import re
soup.find_all('p', attrs={'id':re.compile('^p')}) # 使用正则表达式
soup.find_all('p', attrs={'class':True}) # 含有class属性即可
```

**根据标签内内容来识别**

这部分还是使用`find_all`函数，增加`text`参数

```python3
a = '''
<p id='p1'>段落1</p>
<p class='p3'>段落2</p>
<p class='p3'>文章</p>
<p></p>
'''

soup = BeautifulSoup(a, "html.parser")

soup.find_all('p', text='文章')
soup.find_all('p', text=['段落1','段落2'])

# 正则表达式
import re
soup.find_all('p', text=re.compile('段落'))
soup.find_all('p',text=True)

# 传入函数
def nothing(c):
    return c not in ['段落1','段落2','文章']
soup.find_all('p',text=nothing)

# 同上
def nothing(c):  
    return c is None
soup.find_all('p',text=nothing)
```

最后使用的传入函数的方法，不止在这里识别内容上可以这样用，在识别标签名和属性时都可以使用。都是输入要判断东西返回布尔值，结果是True的才会被选中。只是函数定义时有点不同

- 第一个参数识别标签，函数的参数应该是标签，比如这样用

```python3
def has_class_but_no_id(tag):
    return tag.has_attr('class') and not tag.has_attr('id')
```

- `attrs`参数可以字典中的属性键的值中使用函数，传入字符串
- `text`参数也是传入字符串

**根据位置识别**

- 找到 第i个a标签
- 找到 第i个和第j个a标签

有时三个标签的标签属性全都一样，所有东西都一样（内容可能不一样，但是类型是一样的），但是我们只想要第二个，这时就不能只通过标签属性内容这些方法提取了，可能它的位置是特殊的就可以用位置来提取。这里其实可以用`find_all`提取出列表，然后在列表中根据位置再提取一次

## **提取内容**

本节分为两个部分

- 提取标签内容：使用`.text`
- 提取标签属性值，像字典一样提取

```python3
a = '''
<body>
    <h><a href='www.biaoti.com'>标题</a></h>
    <p>段落1</p>
    <p>段落2</p>
</body>
'''
soup = BeautifulSoup(a, 'html.parser')

# 提取内容
soup.p.text
for p in soup.find_all('p'):
    print(p.text)
soup.h.text # 多层嵌套也可以直接返回
soup.h.a.text # 也可以这样
soup.body.text # 里面有多个内容时 '\n标题\n段落1\n段落2\n'

# 提取属性值，像字典一样提取，以下两种方法等价
soup.h.a['href']
soup.h.a.get('href')
```

## **查看标签信息**

本节分为如下内容

- 获得标签名
- 获得标签所有属性的字典
- 检查标签是否有某属性

```python3
a = '''
<body>
    <h><a href='www.biaoti.com'>标题</a></h>
    <p>段落1</p>
    <p></p>
</body>
'''
soup = BeautifulSoup(a, 'html.parser')
for i in soup.body.find_all(True):
    print(i.name) # 提取标签名
    print(i.attrs) # 提取标签所有属性值
    print(i.has_attr('href')) # 检查标签是否有某属性
```

## **其他提取细节**

本节包括以下几个部分

- 多层嵌套标签时的`find_all`——渗透到所有层次子节点全部提取出来
- `find_all`的简写方法——可以省去`find_all`这8个字母
- `find_all`的其他参数
- 通过css选择器来提取
- 库中的其他函数

示例代码如下

```python3
a = '''
<span>
    <span id='s'>内容1</span>
</span>
<span>内容2</span>
'''
soup = BeautifulSoup(a, 'html.parser')

# 多层嵌套标签
soup.find_all('span') # 3个元素
    # [<span>
    #  <span>内容1</span>
    #  </span>, <span>内容1</span>, <span>内容2</span>]
# find_all的简写，标签直接加括号
soup.span('span',id='s') # 相当于调用find_all返回list

# find_all的其他参数
soup.find_all('span', limit=2) # 限制只返回前两个
soup.find_all('span', recursive=False) # 只查找子节点，不查找孙节点
```

总结下来，Beautifulsoup库其实主要使用`find_all`方法进行网页解析，审查不同元素使用不同参数即可实现，不同参数使用方法都相同，无论是列表、正则pattern还是函数都支持，让提取信息非常得心应手。

除此之外，Beautifulsoup库还提供了css选择器的`select`方法，这里不多讲，用css选择器提取的方法会在后面的pyquery库中讲解。

另外，对于库中的其他方法做如下说明：

- `findAll`是之前版本的`find_all`，如果看到有其他教程用这个不要觉得奇怪
- 库中还提供了一些查询兄弟节点等查询方法，还有对html代码进行修改的方法（有时不要的部分才有特点，就可以把不要的部分替换掉，就可以自由提取想要的了），详情可以见[官网](https://link.zhihu.com/?target=https%3A//www.crummy.com/software/BeautifulSoup/bs4/doc/)
- 解析器的选择

`BeautifulSoup(a, "html.parser")`这里的第二个参数表示使用的解析器，BeautifulSoup提供了三个解析器，它们各自的优缺点如下

- `html.parser`内置不依赖扩展，容错能力强，速度适中
- `lxml`速度最快，容错能力强，但是依赖C扩展
- `html5hib`速度最慢，容错能力最强，依赖扩展

它们不仅在这些性能方面有些许差异，而且在对文档的处理上也有差异，虽然这些差异很多时候不影响我们解析，详情见[官网](https://link.zhihu.com/?target=https%3A//www.crummy.com/software/BeautifulSoup/bs4/doc/%23differences-between-parsers)

使用beautifulsoup抓取网页实战可以看这篇文章 [beautifulsoup+json抓取stackoverflow实战](https://zhuanlan.zhihu.com/p/35355106)