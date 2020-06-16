一、Python+unittest+requests+HTMLTestRunner 完整的接口自动化测试框架搭建_00——框架结构简解
common：

——configDb.py：这个文件主要编写数据库连接池的相关内容，本项目暂未考虑使用数据库来存储读取数据，
               此文件可忽略，或者不创建。本人是留着以后如果有相关操作时，方便使用。

——configEmail.py：这个文件主要是配置发送邮件的主题、正文等，将测试报告发送并抄送到相关人邮箱的逻辑。

——configHttp.py：这个文件主要来通过get、post、put、delete等方法来进行http请求，并拿到请求响应。

——HTMLTestRunner.py：主要是生成测试报告相关

——Log.py：调用该类的方法，用来打印生成日志

result:

——logs：生成的日志文件

——report.html：生成的测试报告

testCase:

——test01case.py：读取userCase.xlsx中的用例，使用unittest来进行断言校验

testFile/case:

——userCase.xlsx：对下面test_api.py接口服务里的接口，设计了三条简单的测试用例，如参数为null，参数不正确等

caselist.txt：配置将要执行testCase目录下的哪些用例文件，前加#代表不进行执行。当项目过于庞大，
             用例足够多的时候，我们可以通过这个开关，来确定本次执行哪些接口的哪些用例。

config.ini：数据库、邮箱、接口等的配置项，用于方便的调用读取。

getpathInfo.py：获取项目绝对路径

geturlParams.py：获取接口的URL、参数、method等

readConfig.py：读取配置文件的方法，并返回文件中内容

readExcel.py：读取Excel的方法

runAll.py：开始执行接口自动化，项目工程部署完毕后直接运行该文件即可

test_api.py：自己写的提供本地测试的接口服务

test_sql.py：测试数据库连接池的文件，本次项目未用到数据库，可以忽略

二、Python+unittest+requests+HTMLTestRunner完整的接口自动化测试框架搭建_01——测试接口服务

首先，我们想搭建一个接口自动化测试框架，前提我们必须要有一个可支持测试的接口服务。有人可能会说，
现在我们的环境不管测试环境，还是生产环境有现成的接口。但是，一般工作环境中的接口，不太满足我们框架的各种条件。
举例如，接口a可能是get接口b可能又是post，等等等等。因此我决定自己写一个简单的接口！用于我们这个框架的测试！

按第一讲的目录创建好文件，打开test_api.py，写入如下代码
"""
import flask
import json
from flask import request
 
'''
flask： web框架，通过flask提供的装饰器@server.route()将普通函数转换为服
'''
# 创建一个服务，把当前这个python文件当做一个服务
server = flask.Flask(__name__)
# @server.route()可以将普通函数转变为服务 登录接口的路径、请求方式
@server.route('/login', methods=['get', 'post'])
def login():
    # 获取通过url请求传参的数据
    username = request.values.get('name')
    # 获取url请求传的密码，明文
    pwd = request.values.get('pwd')
    # 判断用户名、密码都不为空
    if username and pwd:
        if username == 'xiaoming' and pwd == '111':
            resu = {'code': 200, 'message': '登录成功'}
            return json.dumps(resu, ensure_ascii=False)  # 将字典转换字符串
        else:
            resu = {'code': -1, 'message': '账号密码错误'}
            return json.dumps(resu, ensure_ascii=False)
    else:
        resu = {'code': 10001, 'message': '参数不能为空！'}
        return json.dumps(resu, ensure_ascii=False)
 
if __name__ == '__main__':
    server.run(debug=True, port=8888, host='127.0.0.1')
 """
 执行test_api.py，在浏览器中输入http://127.0.0.1:8888/login?name=xiaoming&pwd=11199回车，
 验证我们的接口服务是否正常~
 https://img-blog.csdnimg.cn/201811211613525.png
 变更我们的参数，查看不同的响应结果确认接口服务一切正常
 https://img-blog.csdnimg.cn/20181121161516925.png
 三、Python+unittest+requests+HTMLTestRunner完整的接口自动化测试框架搭建_02——配置文件读取

    在我们第二讲中，我们已经通过flask这个web框架创建好了我们用于测试的接口服务，
    因此我们可以把这个接口抽出来一些参数放到配置文件，然后通过一个读取配置文件的方法，
    方便后续的使用。同样还有邮件的相关配置~

 按第一讲的目录创建好config.ini文件，打开该文件写入如下：
 # -*- coding: utf-8 -*-
[HTTP]
scheme = http
baseurl = 127.0.0.1
port = 8888
timeout = 10.0
 
 
 
[EMAIL]
on_off = on;
subject = 接口自动化测试报告
app = Outlook
addressee = songxiaobao@qq.com
cc = zhaobenshan@qq.com
在HTTP中，协议http，baseURL，端口，超时时间。

在邮件中on_off是设置的一个开关，=on打开，发送邮件，=其他不发送邮件。subject邮件主题，addressee收件人，cc抄送人。

在我们编写readConfig.py文件前，我们先写一个获取项目某路径下某文件绝对路径的一个方法。
按第一讲的目录结构创建好getpathInfo.py，打开该文件
import os
 
def get_Path():
    path = os.path.split(os.path.realpath(__file__))[0]
    return path
 
if __name__ == '__main__':# 执行该文件，测试下是否OK
    print('测试路径是否OK,路径为：', get_Path())

填写如上代码并执行后，查看输出结果，打印出了该项目的绝对路径：
 https://img-blog.csdnimg.cn/20181121171447135.png
 继续往下走，同理，按第一讲目录创建好readConfig.py文件，打开该文件，以后的章节不在累赘
import os
import configparser
import getpathInfo#引入我们自己的写的获取路径的类
 
path = getpathInfo.get_Path()
#调用实例化，还记得这个类返回的路径为C:\Users\songlihui\PycharmProjects\dkxinterfaceTest
config_path = os.path.join(path, 'config.ini')
#这句话是在path路径下再加一级，最后变成C:\Users\songlihui\PycharmProjects\dkxinterfaceTest\config.ini
config = configparser.ConfigParser()#调用外部的读取配置文件的方法
config.read(config_path, encoding='utf-8')
 
class ReadConfig():
 
    def get_http(self, name):
        value = config.get('HTTP', name)
        return value
    def get_email(self, name):
        value = config.get('EMAIL', name)
        return value
    def get_mysql(self, name):#写好，留以后备用。但是因为我们没有对数据库的操作，所以这个可以屏蔽掉
        value = config.get('DATABASE', name)
        return value
 
 
if __name__ == '__main__':#测试一下，我们读取配置文件的方法是否可用
    print('HTTP中的baseurl值为：', ReadConfig().get_http('baseurl'))
    print('EMAIL中的开关on_off值为：', ReadConfig().get_email('on_off'))

执行下readConfig.py，查看数据是否正确
https://img-blog.csdnimg.cn/2018112117235997.png
一切OK

四、Python+unittest+requests+HTMLTestRunner完整的接口自动化测试框架搭建_03——读取Excel中的case

配置文件写好了，接口我们也有了，然后我们来根据我们的接口设计我们简单的几条用例。
首先在前两讲中我们写了一个我们测试的接口服务，针对这个接口服务存在三种情况的校验。
正确的用户名和密码，账号密码错误和账号密码为空
https://img-blog.csdnimg.cn/20181122204909459.png

https://img-blog.csdnimg.cn/20181122204921287.png

https://img-blog.csdnimg.cn/20181122204935574.png

我们根据上面的三种情况，将对这个接口的用例写在一个对应的单独文件中
testFile\case\userCase.xlsx ，userCase.xlsx内容如下：
https://img-blog.csdnimg.cn/20181122205235624.png

紧接着，我们有了用例设计的Excel了，我们要对这个Excel进行数据的读取操作，继续往下，我们创建readExcel.py文件

五、Python+unittest+requests+HTMLTestRunner完整的接口自动化测试框架搭建_04——requests请求

配置文件有了，读取配置文件有了，用例有了，读取用例有了，我们的接口服务有了，我们是不是该写对某个接口进行http请求了
这时候我们需要使用pip install requests来安装第三方库，在common下configHttp.py
执行该文件，验证结果正确性
我们发现和浏览器中进行请求该接口，得到的结果一致，说明没有问题，一切OK
六、Python+unittest+requests+HTMLTestRunner完整的接口自动化测试框架搭建_05——参数动态化

在上一讲中，我们写了针对我们的接口服务，设计的三种测试用例，使用写死的参数
（result = RunMain().run_main('post', 'http://127.0.0.1:8888/login', 'name=xiaoming&pwd=')）
来进行requests请求。本讲中我们写一个类，来用于分别获取这些参数，
来第一讲的目录创建geturlParams.py
通过将配置文件中的进行拼接，拼接后的结果：http://127.0.0.1:8888/login?和我们请求的一致
七、Python+unittest+requests+HTMLTestRunner完整的接口自动化测试框架搭建_06——unittest断言

以上的我们都准备好了，剩下的该写我们的unittest断言测试case了，在testCase下创建test01case.py文件
八、Python+unittest+requests+HTMLTestRunner完整的接口自动化测试框架搭建_07——HTMLTestRunner

按我的目录结构，在common下创建HTMLTestRunner.py文件

九、Python+unittest+requests+HTMLTestRunner完整的接口自动化测试框架搭建_08——调用生成测试报告

先别急着创建runAll.py文件（所有工作做完，最后我们运行runAll.py文件来执行接口自动化的测试工作
并生成测试报告发送报告到相关人邮箱），但是我们在创建此文件前，还缺少点东东。按我的目录结构创建caselist.txt文件
这个文件的作用是，我们通过这个文件来控制，执行哪些模块下的哪些unittest用例文件。
如在实际的项目中：user模块下的test01case.py，店铺shop模块下的我的店铺my_shop，
如果本轮无需执行哪些模块的用例的话，就在前面添加#。我们继续往下走，还缺少一个发送邮件的文件。
在common下创建configEmail.py文件
运行configEmail.py验证邮件发送是否正确
邮件已发送成功，我们进入到邮箱中进行查看，一切OK~~不过这我要说明一下，我写的send_email是调用的outlook，
如果您的电脑本地是使用的其他邮件服务器的话，这块的代码需要修改为您想使用的邮箱调用代码
继续往下走，这下我们该创建我们的runAll.py文件了
执行runAll.py，进到邮箱中查看发送的测试结果报告，打开查看
然后继续，我们框架到这里就算基本搭建好了，但是缺少日志的输出，在一些关键的参数调用的地方我们来输出一些日志。
从而更方便的来维护和查找问题。
按目录结构继续在common下创建Log.py

然后我们在需要我们输出日志的地方添加日志：

我们修改runAll.py文件，在顶部增加import common.Log，然后增加标红框的代码
https://img-blog.csdnimg.cn/20181129112445380.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3NvbmdsaDEyMzQ=,size_16,color_FFFFFF,t_70
让我们再来运行一下runAll.py文件，发现在result下多了一个logs文件，我们打开看一下有没有我们打印的日志
OK，至此我们的接口自动化测试的框架就搭建完了，后续我们可以将此框架进行进一步优化改造，使用我们真实项目的接口，
结合持续集成定时任务等，让这个项目每天定时的来跑啦~~~











 
 