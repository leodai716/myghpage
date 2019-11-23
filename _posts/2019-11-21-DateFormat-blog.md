---
layout: post
title: "Date Preprocessing – Changing the Date Format"
date:   2019-11-21
categories: NLP
author: "Darius Choi"
---

Previously we have scraped pieces of opinion articles from multiple newspapers. The corpus contains 10 editorials each day in 2019 from 5 newspaper websites. Assuming that data is collected until the last day of October, the number of editorial entries already totals 15200. The date format is in MMMD or MMMDD originally, for example, JAN8 for 8th January and JAN10 for 10th January. To allow easier handling and reading of the corpus, I would like to change the date format into a more common and manageable style which is YYYY-MM-DD, for example, 2019-01-08. Doing it manually and using the “replace all” function in excel day by day are time-consuming. Or, scraping the data all again in a new date format is an option but it is not optimal as it might take days for the robot to finish the task. Thus, I decide to use Python to read all the data and amend the format with dictionary. With a surprise, Python can do it in a split second.  

## Deep Dive into the Code
What the code does at first is to open the existing tsv file which stores the data and to create a new tsv file to store the amended version. I avoid directly overriding the existing file with the new content because both versions might come in handy someday in the future. It is better to have both at hand and not to spend time reversing the process when necessary. Then, I create a dictionary to match all the values to the keys. Values here are the date format that I prefer (YYYY-MM-), and the keys are the first 3 characters of the existing format (MMM) that sufficiently indicate the month concerned. By using dictionary, Python can, on one hand, read the keys in the corpus and on the other hand refers to the associated values for replacement. Next, it gets to reading the text. Since I a newbie to Python, I struggled with reading a tsv file with Python. I had heard of how the pandas package manages data with proficiency, so I did have a try but with no success. Hilariously, I only realised that Python was powerful enough to manipulate a tsv file without external packages had I done a long time of trial and error. So I simply use the for loop to read the file line by line. You can notice there are 2 if statements within the for function. It is due to the expression of the original date format. The corpus technically contains 2 types of date: single digit (MMMD\t) and double digit (MMMDD\t). “\t” here represents the tab that delimits text in a tsv file. For the dates that are single-digit, they go to the first if statement because the position of “\t” meets the condition. A “0” is supplemented after the matching value to make all the dates double-digit in order to maintain consistency. On the contrary, all the dates that are double-digit go to the elif statement. You can notice that there is another requirement in both if statements: entry[0:3]!=“Dat”. This condition is to instruct the computer to skip the header line which is “Date\ttitle\ttext”. After the elif statement, I instruct Python to write the entry into the new tsv file and the loop continues until all the entries are read and processed.

## The Power of Python  
To all the new beginners of Python, don’t underestimate the power of this programming language. Sometimes, the tasks can be simply performed by Python in an intuitive manner. You do not have to overthink. Have faith in Python and it will definitely save you time.   



```python
tsvfile = open(r"C:\Users\Darius Choi\Downloads\independentpastopinion.tsv","r+",encoding="UTF-8")
newtsvfile = open(r"C:\Users\Darius Choi\Downloads\independentpastopinio(dateamended).tsv","w+",encoding="UTF-8")
monthdictionary = {"JAN":"2019-01-", 
                   "FEB":"2019-02-", 
                   "MAR":"2019-03-", 
                   "APR":"2019-04-", 
                   "MAY":"2019-05-", 
                   "JUN":"2019-06-", 
                   "JUL":"2019-07-", 
                   "AUG":"2019-08-", 
                   "SEP":"2019-09-", 
                   "OCT":"2019-10-",
                   "NOV":"2019-11-",
                   "DEC":"2019-12-"}
for entry in tsvfile:
    if entry[4] == "\t" and entry[0:3] != "Dat":
        entry = entry.replace(entry[0:3],monthdictionary[entry[0:3]]+"0")
    elif entry[0:3] != "Dat":
        entry = entry.replace(entry[0:3],monthdictionary[entry[0:3]])
    newtsvfile.write(entry)
tsvfile.close()
newtsvfile.close()

```