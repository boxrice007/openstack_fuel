问题：在OpenStack平台上使用中文时就会出现编码问题，问题描述如下：
ERROR (UnicodeEncodeError): 'ascii' codec can't encode characters in position 0-3: ordinal not in range(128)
解决方法：进入/usr/lib/python2.7目录，把系统原有的sitecustomzie.py进行备份，编辑此文件：
# encoding=utf8   
import sys   
 
reload(sys)   
sys.setdefaultencoding('utf8')
重启python解释器，验证编码：
>>> import sys
>>> sys.getdefaultencoding()
'utf8

此时OpenStack就可以识别中文
