---
layout: post
title: "Welcome to Jekyll!"
date: 2017-04-06 13:32:20 +0300
description: You’ll find this post in your `_posts` directory. Go ahead and edit it and re-build the site to see your changes. # Add post description (optional)
img:  # Add image post (optional)
---
You’ll find this post in your `_posts` directory. Go ahead and edit it and re-build the site to see your changes. You can rebuild the site in many different ways, but the most common way is to run `jekyll serve`, which launches a web server and auto-regenerates your site when a file is updated.

To add new posts, simply add a file in the `_posts` directory that follows the convention `YYYY-MM-DD-name-of-post.ext` and includes the necessary front matter. Take a look at the source for this post to get an idea about how it works.

Jekyll also offers powerful support for code snippets:

{% highlight python %}
	# -*- coding:UTF-8 -*-
	from bs4 import BeautifulSoup
	import subprocess as sp
	from lxml import etree
	import requests
	import random
	import re
	from urllib import request
	from selenium import webdriver
	# 引入配置对象DesiredCapabilities
	from selenium.webdriver.common.desired_capabilities import DesiredCapabilities
	class proxyPool:
		def __init__(self):
			pass
		#获取代理信息
		def getProxys(self,page=1):
			s = requests.Session()
			#target_url = 'http://www.xicidaili.com/nn/%d' %page
			target_url = 'http://www.xicidaili.com/wt/%d' %page
			target_hearders = {
				'Upgrade-Insecure-Requests':'1',
				'User-Agent':'Mozilla/5.0 (Windows NT 6.1; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/58.0.3029.110 Safari/537.36',
				'Accept':'text/html,application/xhtml+xml,application/xml;q=0.9,image/webp,*/*;q=0.8',
				'Referer':'http://www.xicidaili.com/nn/',
				'Accept-Encoding':'gzip, deflate, sdch',
				'Accept-Language':'zh-CN,zh;q=0.8',
			}
			target_rsp = s.get(url=target_url,headers = target_hearders)
			target_rsp.encoding = 'utf-8'
			target_html = target_rsp.text
			bf1_ip_list = BeautifulSoup(target_html,'lxml')
			bf2_ip_list = BeautifulSoup(str(bf1_ip_list.find_all(id='ip_list')),'lxml')
			ip_list_info = bf2_ip_list.table.contents
			proxys_list = []
			for index in range(len(ip_list_info)):
				if index % 2 == 1 and index != 1:
					dom = etree.HTML(str(ip_list_info[index]))
					ip = dom.xpath('//td[2]')
					port = dom.xpath('//td[3]')
					protocol = dom.xpath('//td[6]')
					proxys_list.append(protocol[0].text.lower()+'#'+ip[0].text+'#'+port[0].text)
			return proxys_list
		#正则规则
		def init_pattern(self):
			#匹配丢包数
			lose_time = re.compile(u"(\d+) packets received", re.IGNORECASE)
			#匹配平均时间
			waste_time = re.compile(u"min/avg/max/stddev = (.+?) ms", re.IGNORECASE)
			return lose_time, waste_time
		#验证代理信息
		def checkProxy(self,ip,lose_time,waste_time):
			#命令 ping -c 3 -t 3 %s
			cmd = "ping -c 3 -t 3 %s"
			#执行命令
			p = sp.Popen(cmd%ip,stdin=sp.PIPE,stdout=sp.PIPE,stderr=sp.PIPE,shell=True)
			#获得返回结果并解码
			out = p.stdout.read().decode('utf-8')
			#丢包数
			lose = 3 - int(lose_time.findall(out)[0])
			#如果丢包数大于2，则认为链接超时，返回平均耗时1000ms
			if lose >2 :
				return 1000
			#如果丢包数目小于等于2，获取平均耗时
			else :
				#平均时间
				wts = waste_time.findall(out)
				tlis = wts[0].split('/')
				average = float(tlis[1])
				return average
		#使用代理访问
		def useProxy(self,url,proxie):
			s = requests.session()
			header = {
			'User-Agent' : 'Mozilla/5.0 (Macintosh; Intel Mac OS X 10_12_6) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/61.0.3163.100 Safari/537.36',
			}
			try :
				response = s.get(url,headers = header,proxies = proxie,timeout = 5)
				print(response.text)
			except :
				print('%s 超时' % proxie)
		#保存代理信息
		def saveProxy(self,proxys_list):
			fp = open('proxyList.txt','w+')
			for proxy in proxys_list:
				fp.write(proxy+'\n')
			fp.close()
		#取值
		def getProxy(self):
			proxyList = []
			file_object = open('proxyList.txt')
			for line in file_object:
				line = line.strip('\n')
				proxyList.append(line)
			proxy = random.choice(proxyList)
			proxyObj = proxy.split("#")
			proxie = {}
			proxie[proxyObj[0]] = proxyObj[1]+':'+proxyObj[2]
			return proxie
		#更新
		def updateList(self):
			#初始化正则表达式
			lose_time,waste_time = self.init_pattern()
			#获取代理
			proxys_list = self.getProxys()
			#如果平均时间超过200ms重新选择IP
			for proxy in proxys_list:
				split_proxy = proxy.split("#")
				#获取IP
				ip = split_proxy[1]
				#检查IP
				average_time = self.checkProxy(ip,lose_time,waste_time)
				if average_time > 200 :
					#去掉
					proxys_list.remove(proxy)
					print('ip超时 ')
				else :
					print('%s 可用 \n' % proxy)
			self.saveProxy(proxys_list)
		#使用phanthomjs
		def userPhan(self ,url):
			driver = webdriver.PhantomJS(executable_path='/Users/liangqiyao/Downloads/python34/phantomjs-2.1.1-macosx/bin/phantomjs')
			driver.get(url)
			driver.implicitly_wait(1)
			print(driver.get_cookies())
		def test(self,proxyd):
			# browser = webdriver.Chrome()
			# browser.get('http://xyq.cbg.163.com/')
			#phantomjs_driver_path='/Users/liangqiyao/Downloads/python34/phantomjs-2.1.1-macosx/bin/phantomjs'
			dcap = dict(DesiredCapabilities.PHANTOMJS)
			#从USER_AGENTS列表中随机选一个浏览器头，伪装浏览器
			dcap["phantomjs.page.settings.userAgent"] = 'Mozilla/5.0 (Macintosh; Intel Mac OS X 10_12_6) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/61.0.3163.100 Safari/537.36'
			# 不载入图片，爬页面速度会快很多
			dcap["phantomjs.page.settings.loadImages"] = False
			# 设置代理
			service_args = ['--proxy=%s'%proxyd['http'],'--proxy-type=socks5']
			print(service_args)
			#打开带配置信息的phantomJS浏览器
			driver = webdriver.Chrome(desired_capabilities=dcap,service_args=service_args)
			# 隐式等待5秒，可以自己调节
			driver.implicitly_wait(5)
			# 设置10秒页面超时返回，类似于requests.get()的timeout选项，driver.get()没有timeout选项
			# 以前遇到过driver.get(url)一直不返回，但也不报错的问题，这时程序会卡住，设置超时选项能解决这个问题。
			driver.set_page_load_timeout(10)
			# 设置10秒脚本超时时间
			driver.set_script_timeout(10)
			# driver.get('http://ip.cip.cc/')
			driver.get('http://xyq.cbg.163.com/')
			print(driver.get_cookies())
			print(driver.page_source)
	if __name__ == '__main__':
		ppool = proxyPool()
		#刷新代理信息
		# ppool.updateList()
		#使用代理
		proxy = ppool.getProxy()
		ppool.test(proxy)
		# #url = 'http://ip.cip.cc/'
		# url = 'http://xyq.cbg.163.com/'
		# ppool.useProxy(url,proxy)
{% endhighlight %}

Check out the [Jekyll docs][jekyll-docs] for more info on how to get the most out of Jekyll. File all bugs/feature requests at [Jekyll’s GitHub repo][jekyll-gh]. If you have questions, you can ask them on [Jekyll Talk][jekyll-talk].

[jekyll-docs]: https://jekyllrb.com/docs/home
[jekyll-gh]:   https://github.com/jekyll/jekyll
[jekyll-talk]: https://talk.jekyllrb.com/
