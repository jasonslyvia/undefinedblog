---
title: Micolog自动摘要及keywords、description输出代码
tags: 

  - micolog
  - python
permalink: micolog-keywords-description
id: 112
updated: '2012-11-22 23:37:47'
date: 2012-11-22 23:37:47
---

<p>Micolog 的主题默认是不输出任何 keywords 和 description 的 meta 信息的，了解 SEO 的人都知道搜索引擎很看重这些。所以为了让 Micolog 更加 SEO 友好，我们应该手动添加这些内容。</p>
<p>一般来说，对于文章页，我们的输出思路是：</p>
<ul>
<li>keywords：所有标签</li>
<li>description：文章的前若干字，一般为100</li>
</ul>
<p>下面就贴上本人收集整理的一些代码，用于自动生成文章摘要及输出 keywords 和 description 信息：</p>
<p><strong>model.py</strong></p>
<p>在这个文件中找到 <strong>get_content_excerpt(self, more='..more'): &nbsp;</strong>这个函数，并将其完整替换为如下内容</p>
<pre class="brush: python;fontsize: 100; first-line: 1; ">	def get_content_excerpt(self,more='..more'):
		if g_blog.show_excerpt:
			if self.excerpt:
				return self.excerpt+' &lt;a href="/%s"&gt;%s&lt;/a&gt;'%(self.link,more)
			else:
				sc=self.content.split('&lt;!--more--&gt;')
				if len(sc)&gt;1:
					return sc[0]+u' &lt;a href="/%s"&gt;%s&lt;/a&gt;'%(self.link,more)
				else:
					temp = self.truncate_html((sc[0]),'120 html')
					return self.filter_tags(temp)
		else:
			return self.content

	def truncate_html(self,content, args):
	   maxlen=120       #显示字数
	   format = 'text' #html：保留html格式截取 text:纯文本取
	   showimg = ''    #空　表示不显示图片 img 显示图片
	   end='...'         #省略显示符号
	   if args:
		   bits = args.split(' ')
		   if len(bits) == 3:
			   maxlen, format, showimg = bits
		   elif len(bits) == 2:
			   maxlen, format = bits
		   else:
			   maxlen = args
	   maxlen = int(maxlen)
	   #过虑段落标记
	   content = re.sub('&lt;/?(P|p)[^&lt;&gt;]*/?&gt;', '', content)
	   #截取纯文本
	   if format == 'text':
		   content = re.sub('(&lt;)[^&lt;&gt;]*(&gt;?)', '', content)
		   content = re.sub('&amp;nbsp;', '', content)#过虑掉&amp;nbsp;

		   if len(content) &lt;= maxlen:
			   return content

		   return '%s%s' % (content[:maxlen],end)
	   #过虑图片标记
	   if showimg != 'img':
		   content = re.sub('&lt;/?(IMG|img)[^&lt;&gt;]*/?&gt;', '', content)
	   n =0
	   result = ''
	   temp = ''
	   isCode = False #是不是HTML代码
	   isHtml = False #是不是HTML特殊字符,如&amp;nbsp;
	   #获取指定长度的内容
	   for i in range(len(content)):
		   temp = content[i]
		   if temp == '&lt;':
			   isCode = True
		   elif temp == '&amp;':
			   isHtml = True
		   elif temp == '&gt;' and isCode:
			   n = n -1
			   isCode = False
		   elif temp == ';' and isHtml:
			   isHtml = False

		   if not isCode and not isHtml:
			   n = n + 1

		   result += temp

		   if n &gt;= maxlen:
			   break

	   #取出所有html标记
	   temp_result = re.sub('(&gt;)[^&lt;&gt;]*(&lt;?)','&gt;&lt;', result)
	   temp_result = temp_result.lower()

	   if len(content) - len(temp_result) &lt; maxlen:
		   return content

	   result += str(end)

	   #去掉不需要结束标记html标记
	   rg = "&lt;/?(AREA|BASE|BASEFONT|BODY|BR|COL|COLGROUP|DD|DT|FRAME|HEAD|HR|HTML|IMG|INPUT|ISINDEX|LI|LINK|META|OPTION|P|PARAM|TBODY|TD|TFOOT|TH|THEAD|TR|area|base|basefont|body|br|col|colgroup|dd|dt|frame|head|hr|html|img|input|isindex|li|link|meta|option|p|param|tbody|td|tfoot|th|thead|tr)[^&lt;&gt;]*/?&gt;"

	   temp_result = re.sub(rg, '',temp_result)

	   #去掉成对的html标记
	   temp_result = re.sub('&lt;([a-zA-Z]+)[^&lt;&gt;]*&gt;(.*?)&lt;/\1&gt;', '',temp_result)

	   #取出html标记符号
	   arr = re.findall('&lt;([a-zA-Z]+)[^&lt;&gt;]*&gt;', temp_result)

	   #补全html标记
	   for i in range(len(arr)):
		   result += '&lt;/%s&gt;' % arr[len(arr)-i-1]
	   return result

	def filter_tags(self, htmlstr):
		re_cdata=re.compile('//&lt;![CDATA[[^&gt;]*//]]&gt;',re.I) #匹配CDATA
		re_script=re.compile('&lt;s*script[^&gt;]*&gt;[^&lt;]*&lt;s*/s*scripts*&gt;',re.I)#Script
		re_style=re.compile('&lt;s*style[^&gt;]*&gt;[^&lt;]*&lt;s*/s*styles*&gt;',re.I)#style
		re_br=re.compile('&lt;brs*?/?&gt;')#处理换行
		re_h=re.compile('&lt;/?w+[^&gt;]*&gt;')#HTML标签
		re_comment=re.compile('&lt;!--[^&gt;]*--&gt;')#HTML注释
		s=re_cdata.sub('',htmlstr)#去掉CDATA
		s=re_script.sub('',s) #去掉SCRIPT
		s=re_style.sub('',s)#去掉style
		s=re_br.sub('n',s)#将br转换为换行
		s=re_h.sub('',s) #去掉HTML 标签
		s=re_comment.sub('',s)#去掉HTML注释
		#去掉多余的空行
		blank_line=re.compile('n+')
		s=blank_line.sub('n',s)
		s=self.replaceCharEntity(s)#替换实体
		return s

	def repalce(s,re_exp,repl_string):
		return re_exp.sub(repl_string,s)

	##替换常用HTML字符实体.
	#使用正常的字符替换HTML中特殊的字符实体.
	#你可以添加新的实体字符到CHAR_ENTITIES中,处理更多HTML字符实体.
	#@param htmlstr HTML字符串.
	def replaceCharEntity(self, htmlstr):
		CHAR_ENTITIES={'nbsp':' ','160':' ',
					'lt':'&lt;','60':'&lt;',
					'gt':'&gt;','62':'&gt;',
					'amp':'&amp;','38':'&amp;',
					'quot':'"','34':'"',}

		re_charEntity=re.compile(r'&amp;#?(?P&lt;name&gt;w+);')
		sz=re_charEntity.search(htmlstr)
		while sz:
			entity=sz.group()#entity全称，如&gt;
			key=sz.group('name')#去除&amp;;后entity,如&gt;为gt
			try:
				htmlstr=re_charEntity.sub(CHAR_ENTITIES[key],htmlstr,1)
				sz=re_charEntity.search(htmlstr)
			except KeyError:
				#以空串代替
				htmlstr=re_charEntity.sub('',htmlstr,1)
				sz=re_charEntity.search(htmlstr)
		return htmlstr</pre>
<p>在主题模板的 <strong>base.html</strong> 中，添加如下代码：</p>
<pre class="brush: xml;fontsize: 100; first-line: 1; ">{% if entry %}
&lt;meta name="description" content="{{entry|excerpt_more}}" /&gt;
    {%if entry.strtags%}
	&lt;meta name="keywords" content="{{entry.strtags}}" /&gt;
{%endif%}
{%else%}
&lt;meta name="keywords" content="ios开发,wordpress,php,objective-c,苹果教程网" /&gt;
&lt;meta name="description" content="iOS开发博客，主要记录iOS开发、PHP开发相关技术内容" /&gt;
{% endif %}</pre>
<p>确保其它的模板中没有另外的 keywords 和 description 输出语句即可</p>
<p>&nbsp;</p>
<p>&nbsp;</p>