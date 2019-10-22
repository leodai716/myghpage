---
layout: post
title: "First step for NLP: Getting Textual data (aka. Have fun with HTML)"
date:   2019-10-20
categories: NLP
author: "Leo Dai"
---

Besides, or because of, the daily internet surfing and social media usage, the internet is full of data that, if properly analyzed, could offer invaluable insights that even for the ever changing and evolving financial markets. Of all the data on the world wide web, a large proportion is textual data; thus, being able to scrap and analyze those textual data are highly valuable skill for any analyst. The first step to any textual analysis (TA/NLP) is getting the data. Because of the unstructured nature of most textual data, scraping those data could prove both difficult and tedious. 

Having no experience in web-scraping before nor any knowledge in html, I venture into the abyss of scraping text from html without knowing what I would encounter in the process. In this short (hopefully) blog, I would take you through my journey in becoming a self-proclaimed, complacent web-scraping guru. 

Disclaimer: my knowledge and use of terms may not be 100% accurate. what more can you ask for from an econ major? 

## Learning the basics
When I ask my computer science major friends what to do when they don’t know how to do something, they tell me that the first thing they would do is google. So, that is exactly what I did. At the time I had no idea on what web scraping is and how that worked. (lecture notes included a section on web-scraping, but I started before the lecturer formally introduced that to us) 

After some reading, I understand that html serves as the backbone of any website. More importantly, it contains all the textual information I want to scrap from a web site. That means if I can get the html of a website, I can use various techniques to get the textual data. 
![image](/myghpage/assets/images/blog/NewsScraping/html.jpg)
 
Before scraping news opinion, I started a mini project trying get graphics cards information form newegg.com (an online electronics wholesale website). 

From this mini project, I learned to use the `requests` package and parsing html with `BeautifulSoup`. By this point everything seemed good and I was fairly confident that I will be able to complete the scraping with ease.  

```python
# import libraries
from bs4 import BeautifulSoup as soup 
import requests 

# get the html
gcards_url = "https://www.newegg.com/Video-Cards-Video-Devices/Category/ID-38?Tpk=graphics%20cards"
uClient = requests.get(gcards_url)

# parse html and save output 
page_soup = soup(page_html, "html.parser")
containers = page_soup.findAll("div", {"class":"item-container"})

filename = "g_cards.tsv"
f = open(filename, "w")
headers = "product_name\tproduct_name\tproduct_price\tproduct_ship\tproduct_total_price\n"
f.write(headers)

for container in containers:
    product_brand = container.find("div", {"class":"item-info"}).find("div", {"class":"item-branding"}).a.img["title"]

    product_name = container.find("div", {"class":"item-info"}).find("a", {"class":"item-title"}).text

    product_price_str = container.find("div", {"class":"item-info"}).find("div", {"class":"item-action"}).find("li", {"class":"price-current"}).strong.text
    product_price_sup = container.find("div", {"class":"item-info"}).find("div", {"class":"item-action"}).find("li", {"class":"price-current"}).sup.text
    product_price = float(product_price_str + product_price_sup)

    product_ship = container.find("div", {"class":"item-info"}).find("div", {"class":"item-action"}).find("li", {"class":"price-ship"}).text.strip()
    
    if product_ship == "Free Shipping": 
        product_ship = 0
    else:
        product_ship = product_ship[1:product_ship.find(" ")]
    
    product_ship = float(product_ship)
    
    product_total_price = product_price + product_ship
    
    print(product_brand)
    print(product_name)
    print(product_price)
    print(product_ship)
    print(product_total_price)
    
    f.write(product_brand +"\t"+ product_name +"\t"+ str(product_price) +"\t"+ str(product_ship) +"\t"+ str(product_total_price)+"\n")

f.close()

```

## Oops, today is not yesterday nor day before yesterday
Our Idea was simple, we would like to build a machine learning model that compare the likelihood of different Brexit outcome with performance indicators. My duty was to understand whether news paper’s opinion would give us any lights on the sentiments. That means my task in data scraping is to get opinions from major news website throughout our period of analysis. However, Websites are constantly changing, and one would not be able to find the opinion of 2008, or even if one could find that it would not appear in the opinion page to today’s website. Therefore, we need a way to capture past opinion headlines and their text for analysis. At last I come across the idea of internet archive with WayBack machine. WayBack Machine allows users to jump to a saved screenshot of a website in previous times. That means we could get the past opinion headline and compare those headlines with Brexit events and some financial indicators. 

## Complications with WayBack Machine
After deciding on using WayBack Machine, I ran through the `requests.get()` and `soup.findAll()`. And boom…. Nothing! What an amazing way to kick start the scraping: coding is just full of errors and bugs. I could not figure out why the information was not appearing on the requested html. I double checked the address and the page html. All hopes were gone until I refreshed the website and observed that the page content took some time to load while the python code could actually load way quicker than the time it took the browser to load. From that point on I suspect that the request function directly scraps html that is readily available at the moment of calling the function, but some element might need time to load. Therefore, the initial request function would not be able to handle the task. I remember reading **Automate the Boring Stuff with Python** by AlSweigart. AlSweigart explained using selenium to automate browser action. But the idea that caught my attention is that it opens a real browser instead of requesting from the web directly. That means I might be able to put the code to sleep for, say, 5 seconds to allow everything to load before obtaining the html. (* I did find functions other than `time.sleep()` later, but I stuck with this code that served me well in my R programming career) 
From this point on everything was just tedious and eye damaging reading of html and try to locate the elements and endless trial and error. But things did get smooth. I would say selenium is not the ideal solution considering its speed, yet it is the only solution I could think of. 

> Lecture on 22/10/2019 discussed the possible reasons behind this. Probably there are some javascripts running in the browser that I cannot simply get the $DOM_0$ directly. With my prior experience with the WayBack Machine, that lecture actually makes a lot of sense to me :)  (blog updated on: 22/10/2019)

## Handshake failed 
One interesting warning we encountered is the `handshake failed`. My main role and responsibility is to develop the backbone and blueprint for News Mining, so I actually outsourced the html parsing for few websites to other members of the group. One observation I had is that when the Chrome browser is opened instead of Firefox, the interpreter would prompt a "handshake failed" warning on the screen. While this did not affect the scraping result, my groupmates and I had an interesting conversation. 
![image](/myghpage/assets/images/blog/NewsScraping/handshake.jpg)

## Quick notes on saving the data
Because of the presence of comma in the text and I would like to preserve as much information in the text as possible (I am aware that comma will be removed in later process of data cleaning), I used tsv instead of csv. It’s very similar to writing csv, but I think this is especially useful for dealing with textual data. Also, after talking to several of my friend, I found out that not many people are aware of tsv or the ability to specify separator in `pd.read_csv(, sep = “”)`

## Concluding Remark
The code runs perfectly, but it takes about one day to scrap data form first of January 2019 to mid-October. That means it’s almost impossible to put the code running on a laptop. Luckily, I have a desktop dedicated to running this kind of extra time-consuming codes. 
Also, life is short. Use API when available or simply stay away from web scraping. It will save you a great deal of anger of frustration. Well, now I am addicted so I can only sink deeper in this bottomless abyss.

## Going Forward
We have basically finished developing codes for obtaining data. The next step is data cleansing and preprocessing including removal of stop words, tokenization, LSA, etc. Future blog will have updates on the creation of text preprocessing functions.   
> Look forward to: regular expressions and matrix operation