# chinese-webtext-spider

# Quick Start

## 安装

    pip install Chinese-webtext-spider==1.0.4

## RobustSpider使用

* RobustSpider可以爬取任意网站的文本，但是质量较低

给定存储路径创建RobustSpider实例。

通过add_url方法添加爬取的url。

通过crawl方法爬取url和中文网络文本。

中文文本保存在给定的存储路径目录下，以docs.json命名。
```python
from zhspider.spiders import RobustSpider

save_path = 'Webtext_database/'
spider = RobustSpider(save_path, 0) #给定存储路径创建RobustSpider实例。

# 通过add_url方法添加爬取的url。
url='https://www.bilibili.com/read/home'
spider.add_url(url) 

url='https://blog.csdn.net/'
spider.add_url(url)

url='https://www.zhihu.com/'
spider.add_url(url)

# url='https://hao.360.com/'
# spider.add_url(url)

# 通过crawl方法爬取url和中文网络文本。
spider.crawl()
```

## 获取文本数据
文本数据以json形式保存,类型为Dict[str(网站),Dict[str(网页),List[str(文本)]]]

用户可以从docs.json中获取数据，也可以直接从Spider实例中的docs属性里获取:
```python
for site, docs_dict in spider.docs.items():
    for link, docs in docs_dict.items():
        for doc in docs:
            print('*'*10)
            print(doc)
```
以下是一部分输出:
```
**********
1.puts函数
功能：输出字符串，里面传入数组名，也可以直接传入字符串（用双引号包围起来）。
char str[5]=“haha”; puts(str); puts(“haha”);
2.gets函数
功能：输入字符串，里面同样传入字符数组名，不能传入一个未定义的数组名
char str[5]; gets(str);
3.strcat函数
功能：连接两个字符串，strcat(字符数组1，字符数组2)，把字符数组2连接上字符数组1的后面，其中字符数组1的大小要能容纳字符数组1和字符数组2的和长。
char str1[10]="wo";  char str2[5]="aini";  strcat(str1,str2);
打印：woaini
4.strcpy和strncpy
功能：复制字符串，strcpy(字符数组1，字符数组2)，将字符串2复制到字符串1中去。
要点：赋值后，字符数组1中原内容不存在。
strncpy可以将字符数组2中前n个字符复制到字符数组1中去，strncpy(字符数组1，字符数组2，n),赋值后根据数组的长度，原数组的值可能存在。
char str1[5]="woheni":  char str2[5]="ni";
strcpy(str1,str2);  printf("%s",str1);//打印:ni
strncpy(str1,str2,1);  printf("%s",str1);//打印：noheni
5.strcmp
功能：比较两个字符串的大小，strcmp(字符数组1，字符数组2)，当1=2，返回值为0；当1>2,返回值大于0;当1<2，返回值小于0.
char str1[5]="woheni":  char str2[5]="ni":
if(strcmp(str1,str2))  printf("yes");//打印:yes
```

## 其他Spiders
爬取训练数据时，我们新建了一些类可以爬取到更优质的数据，但是只能在固定网站中爬取。

使用方法如下：
mode:一共有两种模式 
mode='inner'时只爬取网站内链和该网站下的文本
mode='external'时会爬取外链的其他网站的文本
```
from zhspider.spiders import CSDNSpider
spider = CSDNSpider('CSDN_database/', 5, mode='inner')
url='https://blog.csdn.net/'
spider.add_url(url)
spider.crawl()
```

```
from zhspider.spiders import BilibiliSpider
spider = BilibiliSpider('Bilibili_database/', 5, mode='inner')
url='https://www.bilibili.com/read/home'
spider.add_url(url)
spider.crawl()
```
其他专门爬取的类可以看spiders里的类

## 高阶技巧

如果用户想要爬取某个网站，我们提供了Spider类供用户继承。

用户继承后只需重写pick_webtext(self,html:str)->List[str]方法即可针对某个网站进行爬虫。

如果用户只想筛选某些url中的文本，用户可以重写other_rule(self,url：str)->bool

只有满足other_rule条件的url中的文本才会被爬取

注意：不满足other-rule条件的url中的链接仍会爬取

如果用户不想访问站内的某些url，用户可以重写ban(self,url:str)->bool

满足ban条件的url不会被访问（文本和链接都不会被爬取）

示例如下：
```
from zhspider.spider import Spider
class BilibiliSpider(Spider):
    def __init__(self, database_path, sleep_time=15, mode="external", debug=True, load_old=False):
        super().__init__(database_path, sleep_time, mode, debug, load_old)
        
        # 构造初始url
        # https://www.bilibili.com/read/cv22990000 从2023-04-11 16:27开始往前爬
        # https://www.bilibili.com/read/cv20000000 爬到2022-11-24 13:57 共5个月

        if 'www.bilibili.com' not in self.urls:
            self.urls['www.bilibili.com']=[]

        # 重置
        if len(self.urls['www.bilibili.com'])<10:
            self.urls['www.bilibili.com']=['https://www.bilibili.com/read/cv'+str(i) for i in range(20000000,22990001)]
            self.visited_urls={}
        
    # 只对满足other_rule的url的HTML进行提取文本(pick_webtext)
    def other_rule(self,url):
        return 'read' in url and "cv" in url
    
    # 被ban的url将不会访问
    def ban(self,url):
        return 'read' not in url
    
    def pick_webtext(self,html):
        threshold = 5
        if '页面不存在或已被删除' in html:
            return []
        
        if not html:
            return []

        # 获取点赞数
        upvote_re = r'like":[\s\S]*?"dislike"'
        upvote = re.findall(upvote_re,html)[0]
        upvote = re.findall(r'\d+',upvote)[0]           
        upvote = int(upvote)   
        # print(upvote)
        # print(html)

        if upvote < threshold:
            return []

        # 获取正文
        content_re = r'"content":"[\s\S]*?","keywords":"'
        content = re.findall(content_re,html)[0]

        content = re.sub(r'"content":"','',content)
        content = re.sub(r'","keywords":"','',content)
        
        if r'\"insert\":\"' in content:
            reg = r'{\\"insert\\":\\"[\s\S]*?\\"}'
            content = re.findall(reg,content)
            str_list = []
            for c in content:
                print(c)
                print(c[14:-3])
                str_list.append(c[14:-3])
            content = '\n'.join(str_list)

        # print([content])
        

        # 删标签
        content = re.sub(r'</h\d>|</p>','\n',content)
        
        content = re.sub(r'\\u003C\\u002Fp\\u003E','\n',content)
        content = re.sub(r'\\u003c\\u002fp\\u003e','\n',content)
        content = re.sub(r'\\u003C\\u002Fh\d\\u003E','\n',content)
        content = re.sub(r'\\u003c\\u002fh\d\\u003e','\n',content)

        content = re.sub(r'\\u003Ch\d\\u003E','\n',content)
        content = re.sub(r'\\u003ch\d\\u003e','\n',content)
        content = re.sub(r'\\u003Cp\\u003E','\n',content)
        content = re.sub(r'\\u003cp\\u003e','\n',content)

        content = re.sub(r'<[^>]+>','',content)
        content = re.sub(r'\\u003C[\s\S]*?\\u003E','',content)
        content = re.sub(r'\\u003c[\s\S]*?\\u003e','',content)
        # print([content])
        # 把unicode转回str
        content = re.sub(r'\\u002F',r'/',content)
        content = re.sub(r'\\u002f',r'/',content)
        content = re.sub(r'\\u002C',r'<',content)
        content = re.sub(r'\\u002c',r'<',content)
        content = re.sub(r'\\u002E',r'>',content)
        content = re.sub(r'\\u002e',r'>',content)
        content = re.sub(r'\\\\n','\n',content)
        content = re.sub(r'\\\\t','\t',content)
        content = re.sub(r'\\\\\\"',r'"',content)
        content = re.sub(r'\\\\',r'\\',content)
        content = re.sub(r'\\n',r'\n',content)
        content = re.sub(r'\\t',r'\t',content)
        
        
        # html实体转回字符
        import html
        content = html.unescape(content)
        del html 

        # 删空行
        content = re.sub(r'\s*\n\s*','\n',content) 
        return [content.lstrip('\n')]
```


# 说明文档
> ## class Spider(self, database_path, sleep_time=15, mode="external", debug=True, load_old=False)
> > #### 说明
> > 所有spider的基类
> > #### 参数 
> > * database_path(str) - 数据库路径，spider将会通过该参数与数据库交互
> > * sleep_time(int) - 网站被爬过后的冷却时间
> > * mode(str) - 只能取'external'和'inner'，'external'会爬取外链，'inner'只爬取内链
> > * debug(bool) - 没用
> > * load_old(bool) - True时加载备份文件，False时加载当前文件
>
> > #### 属性
> > * database_path(str) - 数据库路径，spider将会通过该参数与数据库交互
> > * sleep_time(int) - 网站被爬过后的冷却时间
> > * urls(Dict[str,List[str]]) - 记录每个网站待爬取的url列表，从数据库中加载
> > * visited_urls(Dict[str,List[str]]) - 记录每个网站已经爬取过的url，从数据库中加载
> > * docs(Dict[str,Dict[str,List[str]]]) - 记录每个网站的网页爬到的文档，从数据库中加载
>
> > #### 方法
> > > add_url(url) - 向urls添加url。
> > > > 参数
> > > > * url(str) - 要添加的url
> > > >
> > > > 返回
> > > >
> > > > * 无。
> > >
> > > get_html(url) - 获取url对应的HTML文本。
> > >
> > > > 参数
> > > >
> > > > * url(str) 
> > > >
> > > > 返回
> > > >
> > > > * 对应的HTML文本(str)
> > >
> > > deduplicate() - 对docs进行模糊去重。
> > >
> > > > 参数
> > > >
> > > > * 无
> > > >
> > > > 返回
> > > >
> > > > * 无
> > >
> > > save() - 用urls、visited_urls、docs对数据库进行更新。
> > >
> > > > 参数
> > > >
> > > > * 无
> > > >
> > > > 返回
> > > >
> > > > * 无
> > >
> > > get_domain(url) - 获取url的域名
> > >
> > > > 参数
> > > >
> > > > * url(str)
> > > >
> > > > 返回
> > > >
> > > > * url的域名(str)
> > >
> > > crawl() - 自动爬取url和文本到数据库里
> > >
> > > > 参数
> > > >
> > > > * 无
> > > >
> > > > 返回
> > > >
> > > > * 无
> > >
> > > pick_webtext(html) - 从html(str)里提取想要的文本
> > >
> > > > 参数
> > > >
> > > > * html(str)
> > > >
> > > > 返回
> > > >
> > > > * List[str] 你想要的文本
> > >
> > > get_urls() - 在urls里获取一批url，这批url的所属的网站互不相同
> > >
> > > > 参数
> > > >
> > > > * 无
> > > >
> > > > 返回
> > > >
> > > > * List[str] url列表
> > >
> > > is_valid(url) - 判断一个url是否有爬取价值
> > >
> > > > 参数
> > > >
> > > > * url(str)
> > > >
> > > > 返回
> > > >
> > > > * bool 如果url正常返回True，否则返回False
> > >
> > > ban(url) - 额外添加不爬取url的条件
> > >
> > > > 参数
> > > >
> > > > * url(str)
> > > >
> > > > 返回
> > > >
> > > > * 默认False
> > >
> > > other_rule(url) - 只有满足other_rule条件的url中的文本才会被(pick_webtext方法)提取
> > >
> > > > 参数
> > > >
> > > > * url(str)
> > > >
> > > > 返回
> > > >
> > > > * 默认True
>
> ## class RobustSpider(database_path, sleep_time=35, extractor=Extractor(), evaluator=Evaluator())
> > #### 说明
> > Spider的子类，能从任意类型的网站中爬取文本
> > #### 参数 
> > * database_path(str) - 数据库路径，spider将会通过该参数与数据库交互
> > * sleep_time(int) - 网站被爬过后的冷却时间
> > * extractor(Extractor) - 提取器，可以从HTML中提取文本和url
> > * evaluator(Evaluator) - 评估器，可以判断一个文本是否高质量，也可以判断两个文本是否连续
>
> > #### 方法
> > > crawl() - 自动执行爬取并保存到数据库。
> > > > 参数
> > > > * 无
> > > >
> > > > 返回
> > > >
> > > > * 无。
> 
> ## class Extractor()
> > #### 说明
> > 能从HTML字符串中提取相应的内容
> > #### 参数 
> > * 无
> > #### 方法
> > > extract_url(html) - 从html中提取出有效的url
> > > > 参数
> > > > * html(str) - 待处理的HTML字符串
> > > >
> > > > 返回
> > > >
> > > > * valid_urls(List[str]) - HTML字符串中有效的url列表。
> > >
> > > extract_text(html) - 从html中提取出有效的中文文本（出现在中文文本中的数字、英文、日文、韩文等也应视作中文）
> > > > 参数
> > > > * html(str) - 待处理的HTML字符串
> > > >
> > > > 返回
> > > >
> > > > * valid_docs(List[str]) - HTML字符串中有效的中文文本列表。
> > >
> 
> ## class Evaluator(tokenizer=None, model=None, threshold=0.7, cuda="cpu")
> > #### 说明
> > 能判断一个文本是否优质，以及两个文本是否连续。
> > #### 参数 
> > * tokenizer(tokenizers.Tokenizer) - 分词器
> > * model(transformers.AlbertModel) - 用于判断文本是否优质和连续的模型，能给出一个文本优质的概率和两个文本连续的概率。
> > * threshold(float) - 判断文本优质和连续的阈值，文本优质和连续的概率大于等于该阈值才被视为优质和连续。提高该阈值虽然能提高文本质量，但是会减少文本的获取效率且使文本和训练evaluator_model的正例（高质量文本）趋同；降低该阈值可以提高文本的多样性。
> > * cuda(str) 模型推理时使用的设备，默认cpu，如果想用GPU可传入"cuda:0"
> > #### 方法
> > > is_good(text) - 判断text是否是高质量文本
> > > > 参数
> > > > * text(str) - 文本
> > > >
> > > > 返回
> > > >
> > > > * 一个布尔值，text若有高于threshold的概率为高质量文本时返回True,否则返回False
> > >
> > > is_continuous(text1, text2) - 判断text1和text2是否连续
> > > > 参数
> > > > * text1(str) - text1文本
> > > > * text2(str) - text1文本后面的一个文本
> > > >
> > > > 返回
> > > >
> > > > * 一个布尔值，text2若有高于threshold的概率能接在text1的后面则返回True,否则返回False
> > >


