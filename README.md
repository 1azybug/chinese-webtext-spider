# chinese-webtext-spider

# 说明文档
> ## class Spider(database_path, sleep_time=35)
> > #### 说明
> > 所有spider的基类
> > #### 参数 
> > * database_path(str) - 数据库路径，spider将会通过该参数与数据库交互
> > * sleep_time(int) - 网站被爬过后的冷却时间
>
> > #### 属性
> > * database_path(str) - 数据库路径，spider将会通过该参数与数据库交互
> > * sleep_time(int) - 网站被爬过后的冷却时间
> > * urls(Dict[str,List[str]]) - 记录每个网站待爬取的url列表，从数据库中加载
> > * visited_urls(Dict[str,Set[str]]) - 记录每个网站已经爬取过的url，从数据库中加载
> > * docs(Dict[str,List[str]]) - 记录每个网站爬到的文档，从数据库中加载
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
>
> ## class RobustSpider(database_path, sleep_time=35, extractor=Extractor(), evaluator=Evaluator())
> > #### 说明
> > Spider的子类，能从任意类型的网站中爬取高质量的文本
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
> ## class Evaluator(evaluator_model=BERT(), threshold = 0.8)
> > #### 说明
> > 能判断一个文本是否优质，以及两个文本是否连续。
> > #### 参数 
> > * evaluator_model(torch.nn.Module) - 用于判断文本是否优质和连续的模型，能给出一个文本优质的概率和两个文本连续的概率。
> > * threshold(float) - 判断文本优质和连续的阈值，文本优质和连续的概率大于等于该阈值才被视为优质和连续。提高该阈值虽然能提高文本质量，但是会减少文本的获取效率且使文本和训练evaluator_model的正例（高质量文本）趋同；降低该阈值可以提高文本的多样性。
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


