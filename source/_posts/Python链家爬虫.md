title: Python链家爬虫
date: 2017-03-21 10:23:17
tags: [Python,Python爬虫,链家]
---

业余时间做了个链家的爬虫，爬取数据写入sqlite，方便浏览和对比。
具体参考了冰蓝大牛的博客<http://lanbing510.info/2016/03/15/Lianjia-Spider.html?utm_source=tuicool&utm_medium=referral>, 根据链家最新的web样式做了修改（2017-03）爬在售的二手房的数据。

链家对短时间内同一个IP的流量有监控，所以如果用多线程去爬，太快可能会被要求输入验证码。试了以下用代理ip池去爬，因为可用的免费代理ip不多也不稳定，就放弃了。后面想数据爬的也不多，就加个延时模拟人为慢慢爬了，另外还有个问题可能是单位时间内Request太快可能解析不到数据，就把延时放在BeautifulSoup解析后面，并加了如果没有数据则重复5次请求的验证过程，才成功抓全数据：
	
	time.sleep(np.random.rand()*3+3)

这里贴出关键的匹配代码，代码好粗糙，仅供发参考：）

	def onsell_spider(mydb,url_page=u"http://gz.lianjia.com/ershoufang/pg1rs越秀/",area=u"越秀"):
	    # time.sleep(np.random.rand()*1)
	    print url_page
	    counts = 0
	    trytime = 0
	    while counts==0 & trytime<=5:
	        try:
	            req = urllib2.Request(url_page,headers=hds[random.randint(0,len(hds)-1)])
	            source_code = urllib2.urlopen(req,timeout=10).read()
	            plain_text=unicode(source_code)#,errors='ignore')   
	            soup = BeautifulSoup(plain_text, "lxml")
	        except (urllib2.HTTPError, urllib2.URLError), e:
	            print e
	            exception_write('onsell_spider',url_page)
	            return
	        except Exception,e:
	            print e
	            exception_write('onsell_spider',url_page)
	            return
	        time.sleep(np.random.rand()*3+3)
	        cj_list=soup.findAll('div',{'class':'info clear'})
	        
	        print len(cj_list)
	        counts = len(cj_list)
	        trytime = trytime + 1
	        for cj in cj_list:
	            info_dict={}
	            href=cj.find('a')
	            if not href:
	                continue
	            info_dict.update({u'链接':href.attrs['href']})
	            name=cj.find('a').text
	            info_dict.update({u'标题':name})

	     #href TEXT primary key UNIQUE, name TEXT, community TEXT, style TEXT, area TEXT, orientation TEXT,decoration TEXT,haslift TEXT,floor TEXT, year TEXT, bplace TEXT,splace TEXT, unit_price TEXT, total_price TEXT, subway TEXT, other TEXT
	       
	            content=unicode(cj.find('div',{'class':'houseInfo'}).renderContents().strip())
	            info=re.match(r"<span .*></span><a .*>(.*)</a>(.*)", content)

	            # print info
	            if info:
	                info=info.groups()
	                info_dict.update({u'小区':info[0]})

	                str = info[1].strip().split('|')
	                # print str[1]

	                try:
	                    info_dict.update({u'户型':str[1].strip()})
	                except Exception,e:
	                    info_dict.update({u'户型':''})
	                try:
	                    info_dict.update({u'面积':str[2].strip()})
	                except Exception,e:
	                    info_dict.update({u'面积':''})
	                try:
	                    info_dict.update({u'朝向':str[3].strip()})
	                except Exception,e:
	                    info_dict.update({u'朝向':''})
	                try:
	                    info_dict.update({u'装修':str[4].strip()})
	                except Exception,e:
	                    info_dict.update({u'装修':''})
	                try:
	                    info_dict.update({u'有无电梯':str[5].strip()})
	                except Exception,e:
	                    info_dict.update({u'有无电梯':''})

	            content=unicode(cj.find('div',{'class':'positionInfo'}).renderContents().strip())
	            info=re.match(r"<span .*></span>(.*)\)(.*)<a .*>(.*)</a>", content)
	            if info:
	                info=info.groups()
	                # print info
	                info_dict.update({u'楼层':info[0]})
	                info_dict.update({u'建造时间':info[1]})
	                info_dict.update({u'大区域':area})
	                try:
	                    info_dict.update({u'小区域':info[2]})
	                except Exception,e:
	                    info_dict.update({u'小区域':info[2]})
	            
	            
	            content=cj.find('div',{'class':'unitPrice'}).find('span').text
	            if content:
	                info_dict.update({u'单价':content})
	            content=cj.find('div',{'class':'totalPrice'}).find('span').text
	            if content:
	                info_dict.update({u'总价':content})

	            content=cj.find('span',{'class':'subway'})
	            # print content
	            if content:
	                try:
	                    info_dict.update({u'地铁':content.text})
	                except Exception,e:
	                    info_dict.update({u'地铁':''})

	            content=cj.find('div',{'class':'followInfo'}).text
	            if  content:
	                info_dict.update({u'其他':content})

	            command=sql_onsell_insert_command(info_dict)
	            mydb.execute(command,1)


	def do_onsell_spider(mydb,area=u"越秀"):
	    
	    url=u"http://gz.lianjia.com/ershoufang/pg%drs%s/" % (1,area)
	    
	    try:
	        req = urllib2.Request(url,headers=hds[random.randint(0,len(hds)-1)])
	        source_code = urllib2.urlopen(req,timeout=10).read()
	        plain_text=unicode(source_code)#,errors='ignore')   
	        soup = BeautifulSoup(plain_text, "lxml")
	    except (urllib2.HTTPError, urllib2.URLError), e:
	        print e
	        exception_write('do_onsell_spider',area)
	        return
	    except Exception,e:
	        print e
	        exception_write('do_onsell_spider',area)
	        return

	    time.sleep(np.random.rand()*1+1)
	    content=soup.find('div',{'class':'page-box house-lst-page-box'})
	    # print soup
	    
	    if content:
	        d="d="+content.get('page-data')
	        exec(d)
	        total_pages=d['totalPage']

	    print total_pages

	    for i in range(total_pages):
	        time.sleep(np.random.rand()*1)
	        url_page=u"http://gz.lianjia.com/ershoufang/pg%drs%s/" % (i+1,area)
	        onsell_spider(mydb,url_page,area)
        
对BeautifulSoup的方法还不太熟悉，用的都是简单粗暴的方法，后续再去细看了。

爬下来的数据：
	![imgdata](images/imgdata.png)

