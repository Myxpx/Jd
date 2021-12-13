import requests
import time
import datetime
import os,re
import sys
from ping3 import ping


def env(key):
    return os.environ.get(key)



if env("CFD_NUM"):
    
    num=int((env("CFD_NUM")))
else:
    num=0

if env("CFD_VALUE"):
    
    value=str((env("CFD_VALUE")))
else:
    value=str(100)


if env("JD_COOKIE"):
    
    jd_cookies=(env("JD_COOKIE").split('&'))




try:
    from sendNotify import send
    print('加载了senNotify')
except:
    try:
        from notify import send
        print('加载了notify')
    except:
        print('找不到通知文件，没有通知')
        send=None




   
print('默认第一个,变量CFD_NUM选择ck，0位第一个，1为第二个。完整ck列表：')
print(jd_cookies)
print('****************************************************') 
print('当前选用cookie')
print(jd_cookies[num])




def head(cookies):

    headers={"Host":"m.jingxi.com",
                        "Accept":"*/*",
                        "Accept-Encoding":"gzip, deflate, br",
                        "User-Agent":"jdpingou;iPhone;5.11.0;15.1;4305736ed7317d2e8cbbcd0959427edb8d84a4c8;network/wifi;model/iPhone12,1;appBuild/100755;ADID/;supportApplePay/1;hasUPPay/0;pushNoticeIsOpen/0;hasOCPay/0;supportBestPay/0;session/86;pap/JA2019_3111789;brand/apple;supportJDSHWK/1;Mozilla/5.0 (iPhone; CPU iPhone OS 14_6 like Mac OS X) AppleWebKit/605.1.15 (KHTML, like Gecko) Mobile/15E148",
                        "Accept-Language":"zh-CN,zh-Hans;q=0.9",
                        "Referer":"https://st.jingxi.com/",
                        "Cookie":cookies[num]}
    return headers


def req(url,headers=None):
    response=requests.get(url,headers=headers)
    return response.text



def url_update():
    print('当前选择红包金额为{}'.format(value))
    url='https://m.jingxi.com/jxbfd/user/ExchangeState?strZone=jxbfd&bizCode=jxbfd&source=jxbfd&dwEnv=7&_cfd_t=1638279541920&ptag=7155.9.47&dwType=2&_stk=_cfd_t%2CbizCode%2CdwEnv%2CdwType%2Cptag%2Csource%2CstrZone&_ste=1&h5st=20211130213901920%3B8205428831590161%3B10032%3Btk01w8b491ba830n56srACb2UmAmafi%2BLyc%2FOSihe790t88cNuM5e2y4IEXcCbgbaEEqsyDJVwTKyZDhRwmZU4L8e86e%3B8e5c6737a403773238fbcf60196e8b43960a00efe57bbbfcfc2f7ea582a55cbf&_=1638279541921&sceneval=2&g_login_type=1&callback=jsonpCBKT&g_ty=ls'
    com=req(url,head(jd_cookies))
    dwLvl=re.compile(r'(?<="dwLvl":)\d+')
    ddwPaperMoney=re.compile(r'(?<="ddwPaperMoney":)\d+')
    poolname=re.compile(r'jxcfd2_exchange_hb_\d+')
    strPrizeName=re.compile(r'(\d+[\.]*\d+)元')
    dwlv1_lis=dwLvl.findall(com)
    ddw_lis=ddwPaperMoney.findall(com)
    pool_lis=poolname.findall(com)
    strPrizeName_lis=strPrizeName.findall(com)
    position=strPrizeName_lis.index(value)

    return dwlv1_lis[position],ddw_lis[position],pool_lis[0]





def yanchi():

    time_ping=ping("m.jingxi.com")/2
  
    url_jd='https://api.m.jd.com/client.action?functionId=queryMaterialProducts&client=wh5'

    time_jd=req(url_jd)
    time_jd_re=re.compile(r'(?<="currentTime2":")\d+')
    time_jd_jg=time_jd_re.findall(time_jd)
   

    time_now=float(time_jd_jg[0])/1000
 
    time_sleep=3600-time_now%3600
    return time_sleep-time_ping



def yfc():    

        response=str(time.time())#每次通知不一样，防止不通知
        
        dwl,ddw,strPoolName=url_update()

        url='https://m.jingxi.com/jxbfd/user/ExchangePrize?strZone=jxbfd&bizCode=jxbfd&source=jxbfd&dwEnv=7&_cfd_t=1638278745387&ptag=7155.9.47&dwType=3&dwLvl={}&ddwPaperMoney={}&strPoolName={}&strPgtimestamp=1638278745370&strPhoneID=4305736ed7317d2e8cbbcd0959427edb8d84a4c8&strPgUUNum=abeb8f04e71ab624faed67a15fbb1013&_stk=_cfd_t%2CbizCode%2CddwPaperMoney%2CdwEnv%2CdwLvl%2CdwType%2Cptag%2Csource%2CstrPgUUNum%2CstrPgtimestamp%2CstrPhoneID%2CstrPoolName%2CstrZone&_ste=1&h5st=20211130212545387%3B8205428831590161%3B10032%3Btk01w8b491ba830n56srACb2UmAmafi%2BLyc%2FOSihe790t88cNuM5e2y4IEXcCbgbaEEqsyDJVwTKyZDhRwmZU4L8e86e%3B1281f1dea57d0b48c4f93830c7a7d9ae8437aff159efea0e5b9ae7c23f64c2c3&_=1638278745389&sceneval=2&g_login_type=1&callback=jsonpCBKV&g_ty=ls'.format(dwl,ddw,strPoolName)
        print(url)
        sleep=yanchi()
        
      
        print('延迟{}后执行'.format(sleep))
        time.sleep(sleep)
        print('发包')
        response+=str(req(url,head(jd_cookies)))
        print('结束')                 
        send('财富岛抢购通知',response)
       
yfc()

