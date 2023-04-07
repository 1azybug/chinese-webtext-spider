# chinese-webtext-spider


> class Spider(database_path, sleep_time=35)
> > 参数 
> > * database_path(str) - 数据库路径，spider将会通过该参数与数据库交互
> > * sleep_time(int) - 网站被爬过后的冷却时间
>
> > 属性
> > * database_path(str) - 数据库路径，spider将会通过该参数与数据库交互
> > * sleep_time(int) - 网站被爬过后的冷却时间
> > * urls(Dict[str,List[str]]) - 记录每个网站待爬取的url列表，从数据库中加载
> > * visited_urls(Dict[str,Set[str]]) - 记录每个网站已经爬取过的url，从数据库中加载
> > * docs(Dict[str,List[str]]) - 记录每个网站爬到的文档，从数据库中加载
>
> > 方法
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

