# spider_python
# learning
# dytt_spider


#!/bin/env/python
from lxml import etree
import requests

BASE_DOMAIN = 'http://www.dytt8.net'
#url = 'http://www.dytt8.net/html/gndy/dyzz/list_23_1.html'
HEADERS = {
    'User-Agent': 'Mozilla/5.0 (Windows NT 6.1; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/65.0.3325.146 Safari/537.36'
}

def get_detail_urls(url):
    response = requests.get(url,headers=HEADERS)
    # response.content返回直接抓下来的bytes流，text是经过requests库进行解码的
    #requests库，默认会使用自己猜测的编码方式将内容进行解码，然后存储在text属性上
    #在电影天堂的网页中，因为编码方式猜错了，所以就乱码了
    #text  = response.content.decode('gbk')
    text = response.text
    html = etree.HTML(text)
    detail_urls = html.xpath("//table[@class='tbspan']//a/@href")
    #for detail_urls in detail_urls:
    #    print (BASE_DOMAIN+detail_urls)
    detail_urls = map(lambda url:BASE_DOMAIN+url,detail_urls)
    return detail_urls

def parse_detail_page(url):
    movie = {}
    response = requests.get(url,headers=HEADERS)
    text = response.content.decode('gbk')
    html = etree.HTML(text)
    title = html.xpath("//div[@class='title_all']//font[@color='#07519a']/text()")[0]
    #上面调用了xpath中的text()属性，也就是以纯文本的形式返回了，所以就不用在用tostring去转换了
    #print (title)
    #如果没有以text的形式返回，则打印的时候需要用tostring去转换
    #for x in title:
    #    print(etree.tostring(x, encoding='utf-8').decode("utf-8"))
    movie['title'] = title

    zoome = html.xpath("//div[@id='Zoom']")[0]
    #zoome中放的是element对象,这个对象中包含了两个src的东西，想一想
    imgs = zoome.xpath(".//img/@src")
    cover = imgs[0]
    screeshot = imgs[1]
    movie['cover'] = cover
    movie['screeshot'] = screeshot

    def parse_info(info,rule):
        return info.replace(rule,"").strip()

    infos = zoome.xpath(".//text()")

    for index,info in  enumerate(infos):
        if info.startswith("◎年　　代"):
            info = parse_info(info,"◎年　　代")
            movie['year']  = info
        elif info.startswith("◎产　　地"):
            info = parse_info(info,"◎产　　地")
            movie['country'] = info
        elif info.startswith("◎类　　别"):
            info = parse_info(info,"◎类　　别")
            movie['category'] = info
        elif info.startswith("◎豆瓣评分"):
            info = parse_info(info,"◎豆瓣评分")
            movie['douban_tating'] = info
        elif info.startswith("◎片　　长"):
            info = parse_info(info,"◎片　　长")
            movie['duration'] = info
        elif info.startswith("◎导　　演"):
            info = parse_info(info,"◎导　　演")
            movie['director'] = info
        elif info.startswith("◎主　　演"):
            print("=" * 30)
            info = parse_info(info,"◎主　　演")
            actors = [info]
            for x in range(index+1,len(infos)):
                if infos[x].startswith("◎"):
                    break
                else:
                    actors.append(infos[x].strip())

            movie['actors'] = actors
        elif info.startswith("◎简　　介"):
            info = parse_info(info,"◎简　　介")
            movie['Introduction'] = info
    download_url = html.xpath("//td[@bgcolor='#fdfddf']/a/@href")[0]
    movie['download_url'] = download_url
    return movie


def spider():
    base_url = 'http://www.dytt8.net/html/gndy/dyzz/list_23_{}.html'
    # 第一个for循环控制总共有7页，
    movies = []
    for x in range(2,8):
        url = base_url.format(x)
        detail_urls = get_detail_urls(url)
     #   第二个for循环是用来拿到每一页中的所有电影的url
        for detail_url in detail_urls:
            movie = parse_detail_page(detail_url)
            movies.append(movie)
            print(movies)
            #因为太多所以就只打印一个
            break
        break



if __name__ == '__main__':
    spider()
