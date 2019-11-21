---
layout: post
title: "I’m sorry I annoyed you with my efficiency"
date:   2019-11-20
categories: NLP
author: "Leo Dai"
---

While I have always heard of clean and efficient codes, I did not realize its importance until I started the NLP project. Working on project means that you must communicate with other people and let other people understand what your code is doing. Being able to communicate your logic effectively vital for the group productivity. I come across script that have such nested logics and ill-defined variable and random numbers that I have absolutely no idea on what the code is doing; worst still, the code had bug, and the author just threw me the code and asked me to debug that. With the painful experience of reading anti-patterns, I started reflecting on my own codes. Such reflection prompted me to search for a better way to write clean and tidy code. 

## Clean Code & PEP8
A significant number of programmers think that being able to write some nested logic and sophisticated one-liner shows their knowledge in the programming language. While, those codes might appear sophisticated, they are also extremely difficult for anyone (including the author, at some later time) to read, use and maintain. 
After some searching, I come across the idea of clean code, that is a reader-focused development style. The main idea are: never do too much (usually not more than 1) operations in a line, and always label everything clearly. At first, following those rules seems slow and inefficient because you might end up typing more lines and longer names. E.g. it forces you to change `df1, df2, mydf` and `myawesomedf` to `raw_dataframe, cleaned_dataframe, preprocessed_dataframe` and `output_dataframe`.  But in the long run, I find it extremely time saving no having to look at my previous code and guess what does `myawesomedf` means. Although, I must admit that, at times, I might be lazy and name something as `temp` inside a function. Nevertheless, writing clean code proved to be extremely beneficial for me and anyone who wants to read my codes because they are clear and easy to follow. 
Another great tool is PEP8. Just like any language, grammars restrict your thought such that you can express more. PEP8 is analogical to grammar for python. It specifies, along with other things, the naming conventions for variables and how to write readable code. PEP8 was written with collaboration in mind, so it indicates how end user or collaborator of source code can modify code to obtain desired result. That philosophy fits the situation of the project in that large data project often requires collaborators to input various data and variable specific to their computer. For me, going PEP8 is a natural and logical decision. It just simplifies things. 
Everything has a learning curve. Even though the principles in clean coding and PEP8 are simple and I was not asking team members in the group to follow strictly to those rules, not everyone were as keen in the idea as I was. I mean why change if the status quo works. It was difficult to force them to follow those rules. Well….I can not just throw them back the unstructured code whenever the code is not following the clean code principle or PEP8 . I can only convince them to follow by showing them the great benefit of those ideas.

## Efficiency? Or Obsession? 
Working with bigdata takes time. I run a data preprocessing programme that took 8 hours to complete. The amount of data we have collected is huge, and the processing required is also quite substantial. A rough estimation for applying tokenization function only could take up to half (not including the tweets). Therefore, like many other programmers, I was desperate to find ways to speed up the codes. For data wrangling, there are three major reasons for slow code: 1. Nested logics 2. Checking data type 3. CPU speed limit. For the first one, it’s requires the author to think creatively on what the most efficient way is to do things: whether writing many if and elif is more efficient or trying to notice and leveraging on the special structure of data. It requires thinking and planning rather than just trying to compile a working code. Second issue is data type. Python is sometimes slow because it must consistently check the datatype to ensure inputs are valid. While that prevented great number of errors, it also slows down the process. This is when numpy comes to light. Numpy create array of specific type, such that much of the checking is bypassed. However, having a tool doesn’t mean knowing how to use that tool. If a person tries to append integer to a numpy array with a for loop, the console still must process many data and try to figure out the data type. So, whatever tools one has, the most import thing is still to think before you type. 

Finally, which is also the most exciting, frustrating, and confusing: CPU limit. There are processes that are CPU bound and those that are I/O bound. Data processing are usually the former. Because python run code line by line, a process would not run until the previous line has been completed. However, that can sometimes be inefficient. What if you just want to sum the value of the columns and you created a for loop. Each process in the loop is independent of the previous process, but the next process would not start until the previous process has ended. In this case you can assign independent task to multiple cpu and combine the results when each cpu has completed the tasks. It is easier said than done. Trying to run multiprocessing package in Windows in painful and requires a lot of trial and error. Also, because tasks have been assigned to new python sessions it’s almost impossible to step into a process and debug. It took me about 5 hours of google and trial&error to get the script to work. For most programmes, you can often copy and paste existing code to ‘solve’ an issue, multiprocessing is entirely different. If you don’t understand the mechanism and execution details your script is bound to throw you errors. It is not for the faint hearted. Nonetheless, I was able to complete the data cleaning 9 times faster at the end. 
![image](/myghpage/assets/images/blog/Efficiency/Multiprocessing.png)
Also, a word of caution, don’t be too aggressive with multiprocessing. I have blown up my cpu and memory, and it’s not fun.

If you are interested, below is an example for multiprocessing. Have fun reading. 

```python
#%% Init
import pandas as pd
import numpy as np
import pickle
import os
import sys
import _LocalVariable
import multiprocessing
from projectpackage.data_preprocessing import TextProcessing
from projectpackage.data_preprocessing import DataTransformation
import time


cores_allocated = int(multiprocessing.cpu_count())
start_time = time.time()
#%% Load data and trained model
DATA_PATH = _LocalVariable._DATA_DIRECTORY + "\\raw_data-opinion-combined.pkl"
#data = pd.read_csv(DATA_PATH, sep='\t')
data = pd.read_pickle(DATA_PATH)

os.chdir(_LocalVariable._OBJECT_DIRECTORY)

file_list = os.listdir()
for item in file_list:
    object_name = item[:-4]  
    f = open(item, 'rb')
    vars()[object_name] = pickle.load(f)

os.chdir(_LocalVariable._WORKING_DIRECTORY)

#%% functions

def text_processing(df):
    df['combined_text'] = df['title'] + " " + df['text']
    df['combined_text'] = df['combined_text'].apply(lambda x: x.lower())
    df['combined_text'] = df['combined_text'].apply(TextProcessing.combine_specialwords)
    
    return df

def tokenization(df):
    df['tokens'] = df['combined_text'].apply(DataTransformation.get_tokens)
    return df

def data_transformation(df):
    df['bow_vector'] = df['tokens'].apply(\
        lambda x: DataTransformation.get_bow_vector(x, word_index_map))
    DTM = DataTransformation.get_DTM_predict(df, word_index_map)
    
    return DTM
#%% main
if __name__ == '__main__':

    chunks = np.array_split(data, cores_allocated)
    
    with multiprocessing.Pool(processes=cores_allocated) as pool:
        results = pool.map(text_processing, chunks)
        data = pd.concat(results)
    
    print("Preprocessing--- %s seconds ---" % (time.time() - start_time))
    
    chunks = np.array_split(data, cores_allocated)

    with multiprocessing.Pool(processes=cores_allocated) as pool:
        results = pool.map(tokenization, chunks)
        data = pd.concat(results)
        
    print("tokenization--- %s seconds ---" % (time.time() - start_time))
    
    chunks = np.array_split(data, cores_allocated)
    with multiprocessing.Pool(processes=cores_allocated) as pool:
        results = pool.map(data_transformation, chunks)
        DTM = pd.concat(results)

    print("DTM--- %s seconds ---" % (time.time() - start_time))
    
    #%% Prediction 
    prediction = model_ab.predict(DTM)
    prediction = pd.DataFrame(prediction)
    prediction.columns = ['brexit']
    
    data_predicted = pd.concat([data.reset_index(drop=True), prediction], axis=1)
    
    data_predicted = data_predicted.loc[:,['Date', 'combined_text',\
                                           'news_paper', 'brexit']]
    
    print("Predict--- %s seconds ---" % (time.time() - start_time))
        
    #%% save the prediction result
    os.chdir(_LocalVariable._DATA_DIRECTORY)
    FILENAME = "prediction_result-combined.pkl"
    pd.to_pickle(data_predicted, FILENAME)
    
    print("Save--- %s seconds ---" % (time.time() - start_time))

```