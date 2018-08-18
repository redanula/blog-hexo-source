title: Python高德爬虫
date: 2017-03-13 14:29:59
tags: [Python,Python爬虫,高德地图API]
---

前段时间因公司需求，需要爬取高德地图的数据，就学习了下Python，并做了个简易的爬虫GUI。

因为高德有api，所以爬数据灰常简单。
整个流程：输入城市编码-》根据api爬取数据-》导出到Excel
使用的库：BeautifulSoup PyQt5 openpyxl urllib2 requests等

具体思路：
### 高德申请api的key，配置好需要爬取的参数：

参数城市、POI分类、查询关键词等,如果不hardcode的部分可以从界面传入

		url = r'http://restapi.amap.com/v3/place/text?&citylimit=true&&output=xml&offset=25&page=1&key=[your key]&extensions=base'
		wherestr = r'&city=440106&types=050301&keywords=肯德基'
		
### 关键抓取代码：

		def store_spider(self, citystr, storetype, wherestr):
	        page_num=1;
	        store_list=[]
	        try_times=0
	        
	        url = r'http://restapi.amap.com/v3/place/text?&citylimit=true&&output=xml&offset=25&page=1&key=[your key]&extensions=base'
	        url = url + wherestr
	        total_record = 1

	        while(total_record>0):
	            url=url.replace('page='+str(page_num-1),'page='+str(page_num))

	            # time.sleep(np.random.rand()*1)
	            
	            try:
	                req = urllib2.Request(url, headers=hds[page_num%len(hds)])
	                source_code = urllib2.urlopen(req).read()
	                plain_text=str(source_code)   
	            except (urllib2.HTTPError, urllib2.URLError), e:
	                print e
	                try_times+=1;
	                print try_times
	                if try_times>10:
	                    date = QDateTime.currentDateTime(); 
	                    self.bigEditor.setPlainText("网络连接异常，抓取【"+ citystr + "】"+ storetype +"结束。" + date.toString(" 时间:yyyy/MM/dd HH:mm:ss") + "\n" + self.bigEditor.toPlainText())
	                    break
	                else:
	                    continue

	            soup = BeautifulSoup(plain_text, "lxml")
	            list_soup = soup.find('pois', {'type': 'list'})
	            
	            # try_times+=1;
	            if list_soup==None and try_times<10:
	                continue
	            elif list_soup==None or len(list_soup)<=1:
	                break 
	            
	            total_record_str = soup.find('count').string.strip()
	            total_record = string.atoi(str(total_record_str))

	            if total_record == 0 : break

	            for storeinfo in list_soup.findAll('poi'):
	                name  = storeinfo.find('name').string.strip()
	                location  = storeinfo.find('location').string.strip()
	                location_list = location.split(',')
	                
	                try:
	                    location_logtitudes = location_list[0]
	                except:
	                    location_logtitudes = '未知'
	                
	                try:
	                    location_autitudes = location_list[1]
	                except:
	                    location_autitudes = '未知'

	                try:
	                    tel  = storeinfo.find('tel').string.strip()
	                except:
	                    tel  = ''

	                try:
	                    adname  = storeinfo.find('adname').string.strip()
	                except:
	                    adname  = ''

	                try:
	                    address  = storeinfo.find('address').string.strip()
	                except:
	                    address  = ''

	                typecode  = storeinfo.find('typecode').string.strip()
	                pname  = storeinfo.find('pname').string.strip()
	                cityname  = storeinfo.find('cityname').string.strip()
	                amaptype  = storeinfo.find('type').string.strip()
	                amapid  = storeinfo.find('id').string.strip()
	                store_list.append([name,pname,cityname,adname,address,location_logtitudes,location_autitudes,tel,storetype,amapid,typecode,amaptype])
	            try_times=0 

	                # print 'Page %d' % page_num

	            #输出到GUI
	            date = QDateTime.currentDateTime(); 
	            self.bigEditor.setPlainText(date.toString("正在抓取【"+ citystr + "】"+ storetype +",页码: " + str(page_num) + "  时间:yyyy/MM/dd HH:mm:ss") + "\n" + self.bigEditor.toPlainText())
	            app.processEvents()
	            page_num+=1
	        return store_list

### 导出到excel：

		#保存到excel
	    def save_storelists_excel(self, storelists, citystr):
	        
	        wb=Workbook()
	        ws=[]

	        # for i in range(len(typelist)):
	        ws.append(wb.create_sheet(title=citystr.decode())) 
	        # for i in range(len(typelist)): 
	        ws[0].append(['序号','商店名称','省份','城市','区/县','商店地址','经度','纬度','联系电话','商店类型','高德ID','高德类型ID','高德类型'])
	        count=1
	        for storelist in storelists:
	            for st in storelist:
	                ws[0].append([count,st[0],st[1],st[2],st[3],st[4],st[5],st[6],st[7],st[8],st[9],st[10],st[11]])
	                count+=1
	        # 保存excel       
	        save_path= 'storelist'
	        save_path+=('-'+citystr.decode())
	        save_path+='.xlsx'
	        wb.remove_sheet(wb['Sheet']); 
	        # save_path = os.path.join(os.path.abspath(os.curdir), save_path)

	        if getattr(sys, 'frozen', False):
	            application_path = os.path.dirname(sys.executable)
	        elif __file__:
	            application_path = os.path.dirname(__file__)

	        save_path = os.path.join(application_path, save_path)

	        wb.save(save_path)

	        date = QDateTime.currentDateTime(); 
	        self.bigEditor.setPlainText("导出到目录文件:【" + save_path + date.toString("】 时间:yyyy/MM/dd HH:mm:ss") + "\n" + self.bigEditor.toPlainText())
	        app.processEvents()

另外UI使用的是PyQt5，PyQt的UI从运行到打包一堆坑要踩，不过很多问题都可以搜到解决方案。UI部分做了个输入框，输入城市以后可以从本地sqlite查询对应该城市区域的全部编码，然后赋值到wherestr上，开始爬取数据并输出到QTextEdit。

总的来说，高德地图API对爬虫很友好，企业认证的调用次数上限都很高，只是貌似每次调用最多就只能返回1000条数据，如果说POI分类多，最好分别进行遍历抓取。这个爬虫都是用到很基础的库，后续有时间继续研究~

效果截图：
	![imgui](/images/imgui.png)


