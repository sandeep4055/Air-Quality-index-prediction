
# <p align=center>Air Quality Index Prediction for BangloreCity city</p>

## Problem Statement:


## Steps Involved in Machine Learning Project
<p align="center">
	<img src="https://editor.analyticsvidhya.com/uploads/80329Roadmap.PNG" alt="Logo">
</p>
<p>
## Data Collection:
<p>The Data collection is done from different sources, we collected data of independent features from  https://en.tutiempo.net/ website using python builtin library 'Requests' which sends html requests to extract data. Then downloaded dependent feature data from "https://github.com/krishnaik06/AQI-Project/tree/master/Data/AQI".</p>
</p>
 
### First let us import modules os, time , requests , sys :

-     import os
-     import time
-     import requests
-     import sys   
-     import matplotlib.pyplot as plt 
-     import pandas as pd
-     from bs4 import BeautifulSoup    

*	Now we create a function retive_html to extract data from website using url 

```
  def retrieve_html():
    #we take data from year 2013 to 2018 so we are iterating
    for year in range(2013,2019):
        #Then we iterate over months
        for month in range(1,13):
            if(month<10):
                url='http://en.tutiempo.net/climate/0{}-{}/ws-421820.html'.format(month,year)
            else:
                url='http://en.tutiempo.net/climate/{}-{}/ws-421820.html'.format(month,year)
            #the requests package goes to the url and collects the data
            texts=requests.get(url)
            #then encode the data collected
            text_utf=texts.text.encode('utf=8')
            #Create the directory & stores in file
            if not os.path.exists("Data/Html_Data/{}".format(year)):
                os.makedirs("Data/Html_Data/{}".format(year))
            with open("Data/Html_Data/{}/{}.html".format(year,month),"wb") as output:
                output.write(text_utf)
            #gets out of the file
        sys.stdout.flush()
```
## Data Preprocessing
1. Data Cleaning & combinig:
 Since we extract the data using requests library it collects all the html content that is available on the requested url , but we don't need all that data, we need only data that is stored in tabular form, so we need to use beautiful library to scrap the data, for we created a function callled met_data.
 #### What is Beautiful Soup?
 <p>Beautiful Soup is a Python library for pulling data out of HTML and XML files. It works with your favorite parser to provide idiomatic ways of navigating, searching, and modifying the parse tree. It commonly saves programmers hours or days of work.</p>
 
```
  def met_data(month, year):
    
    # we open the file in binary format for reading by providing month & year 
    file_html = open('Data/Html_Data/{}/{}.html'.format(year,month), 'rb')
    plain_text = file_html.read()

    tempD = []
    finalD = []
    
    soup = BeautifulSoup(plain_text, "lxml")
    # we need the data that stores in table 
    for table in soup.findAll('table', {'class': 'medias mensuales numspan'}):
        #iterating through table body and rows and storing the content in list
        for tbody in table:
            for tr in tbody:
                a = tr.get_text()
                tempD.append(a)
                
# To find the no.of rows we divide length of list using no.of columns bcz we know the no.of columns present from website as 15, they are static but rows vary because of 
no.of days in months vary.
    rows = len(tempD) / 15
    
# We wanted to store each row as single element in list so we create nested list 
    for times in range(round(rows)):
        newtempD = []
        for i in range(15):
            newtempD.append(tempD[0])
            tempD.pop(0)
        finalD.append(newtempD)

    length = len(finalD)

# Deleted the unwanted columns beacuse there is no data available in those columns 
    finalD.pop(length - 1)
    finalD.pop(0)

    for a in range(len(finalD)):
        finalD[a].pop(6)
        finalD[a].pop(13)
        finalD[a].pop(12)
        finalD[a].pop(11)
        finalD[a].pop(10)
        finalD[a].pop(9)
        finalD[a].pop(0)

    return finalD
```
*	Now lets Deal with dependet features data , the data of dependent features has hourly data where as the data of independent features is in day wise data, so now we need to transform the hourly data into day wise data by averaging the data for every 24 hours.
```
  def avg_data_2013():
    temp_i=0
    average=[]
    
    #lets open the csv file of the dependent feature with chunksize =24 , means the take data of every 24hours
    
    for rows in pd.read_csv('Data/AQI/aqi2013.csv',chunksize=24):
        add_var=0
        avg=0.0
        data=[]
        df=pd.DataFrame(data=rows)
        
#iterrows() is used to iterate over a pandas Data frame rows in the form of (index, series) pair. 
#This function iterates over the data frame column,it will return a tuple with the column name and content in form of series.
        for index,row in df.iterrows():
        # appending PM2 column into list
            data.append(row['PM2.5'])
        for i in data:
            if type(i) is float or type(i) is int:
                add_var=add_var+i
            elif type(i) is str:
                if i!='NoData' and i!='PwrFail' and i!='---' and i!='InVld':
                    temp=float(i)
                    add_var=add_var+temp
        # Caluclating Average
        avg=add_var/24
        temp_i=temp_i+1
        
        average.append(avg)
    return average
```
* Similarly use the above function for all the years.

* Now lets combine the independent and dependent feautres into Real_combine.csv
```
if __name__ == "__main__":
    # Check for paths exists if not create
    if not os.path.exists("Data/Real-Data"):
        os.makedirs("Data/Real-Data")
    for year in range(2013, 2017):
        final_data = []
        # Create the csv file and insert data into it by iterating over month & year
        with open('Data/Real-Data/real_' + str(year) + '.csv', 'w') as csvfile:
            wr = csv.writer(csvfile, dialect='excel')
            wr.writerow(
                ['T', 'TM', 'Tm', 'SLP', 'H', 'VV', 'V', 'VM', 'PM 2.5'])
        for month in range(1, 13):
            temp = met_data(month, year)
            final_data = final_data + temp
            
        pm = getattr(sys.modules[__name__], 'avg_data_{}'.format(year))()

        if len(pm) == 364:
            pm.insert(364, '-')

        for i in range(len(final_data)-1):
            # final[i].insert(0, i + 1)
            final_data[i].insert(8, pm[i])
            
    #Delete the rows that are having no values or values with "-"

        with open('Data/Real-Data/real_' + str(year) + '.csv', 'a') as csvfile:
            wr = csv.writer(csvfile, dialect='excel')
            for row in final_data:
                flag = 0
                for elem in row:
                    if elem == "" or elem == "-":
                        flag = 1
                if flag != 1:
                    wr.writerow(row)
     #now combine the data using data_combine function that we created               
    data_2013 = data_combine(2013, 600)
    data_2014 = data_combine(2014, 600)
    data_2015 = data_combine(2015, 600)
    data_2016 = data_combine(2016, 600)
     
    total=data_2013+data_2014+data_2015+data_2016
    
    with open('Data/Real-Data/Real_Combine.csv', 'w') as csvfile:
        wr = csv.writer(csvfile, dialect='excel')
        wr.writerow(
            ['T', 'TM', 'Tm', 'SLP', 'H', 'VV', 'V', 'VM', 'PM 2.5'])
        wr.writerows(total)
        
        
df=pd.read_csv('Data/Real-Data/Real_Combine.csv')
```

 

