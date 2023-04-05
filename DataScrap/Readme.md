# Web Scraping with Python
In this project, we demonstrate how to scrape data from a webpage using Python's requests and BeautifulSoup libraries. Specifically, we extract information about food items and their prices from the following webpage: https://happymoonsemaar.jacca.com/Menu?C=view_tr_happymoonsemaar_99

# Libraries
<p>We start by importing the necessary libraries:<p>

```
import requests
from bs4 import BeautifulSoup
```
# Retrieving the Webpage
<p>We use the requests library to retrieve the webpage:<p>

```
header = {
    "user-agent": "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/111.0.0.0 Safari/537.36"
}
url = "https://happymoonsemaar.jacca.com/Menu?C=view_tr_happymoonsemaar_99"
parser = BeautifulSoup(requests.get(url, headers=header).content, "html.parser")
```
<p>Note that we set a user-agent in the header to mimic a web browser and avoid being blocked by the server.<p>

# Scraping the Data
<p>We extract the following information from the webpage:
Category names
Product names
Product contents
Product prices
We define four functions, each of which retrieves one of the above pieces of information using the BeautifulSoup library: <p>

```
catagoryList = list()
productNameList = list()
productNameContentsList = list()
productPriceList = list()
```

```
# Veriler Html taglarından çekilirken kirli bir şekilde geldi temizlenmesi gerek  
# Eksik Verilerde mevcut ancak doldurmaya gerek duymadım
# html taglarından Katagori ismini alıp categoryList'e ekledik
def catagory():
    catagory = parser.find_all("li", {"class": "product bubbleImageSideAligned"})
    for i in catagory:
        i = i.text
        cleaned_text = i.strip().replace('\n', ' ')
        cleaned_text = cleaned_text.split("    ")[0]
        catagoryList.append(cleaned_text)
        
# html taglarından Ürün ismini alıp productNameList'e ekledik
def productName():
    productCatagory = parser.find_all("div", {"class": "itemName alignment bubbleImageSideAligned"})
    for i in productCatagory:
        i = i.text
        cleaned_text = i.strip().replace('\n', ' ')
        productNameList.append(cleaned_text)

# html taglarından Ürün içeriğini  ismini productNameContentsList'e ekledik
def productContents():
    productCatagory = parser.find_all("div", {"class": "itemDescription"})
    for i in productCatagory:
        i = i.text
        cleaned_text = i.strip().replace('\n', ' ')
        productNameContentsList.append(cleaned_text)
# html taglarından Ürün fiyatını alıp productPriceList'e ekledik
def productPrice():
    productCatagory = parser.find_all("div", {"class": "itemPrice alignment bubbleImageSideAligned"})
    for i in productCatagory:
        i = i.text
        cleaned_text = i.strip().replace('\n', ' ')
        cleaned_text = cleaned_text.replace(",", ".").replace(" ₺", "")
        cleaned_text = float(cleaned_text)
        productPriceList.append(cleaned_text)
```

<p>Note that we also perform some data cleaning in the above functions to remove extraneous characters and convert the price data to a float format.

# Conclusion
<p>In this project, we demonstrated how to scrape data from a webpage using Python's requests and BeautifulSoup libraries. We extracted information about food items and their prices from a particular webpage, and performed some data cleaning to make the results more usable.<p>



# Saving Web Scraped Data to MySQL Database in Python

<p>In this project, we will save the data scraped from a webpage using Python's requests and BeautifulSoup libraries to a MySQL database.<p>

# Libraries

<p>We will use the following libraries:<p>

```
import pymysql as mysql
import pandas as pd
```
# Establishing Connection

<p>We start by establishing a connection to our MySQL database. We will use the pymysql library to connect to the database.<p>

```
# mysql bağlantısı yapıldı
try:
    connectDb=mysql.connect(host='localhost',
                      user='root',
                      password='password',
                      db='database_name',
                      charset='utf8mb4',
                      cursorclass=mysql.cursors.DictCursor
                      )
except Error as error:    
     print("Failed to connect: {}".format(error))

newCursor=connectDb.cursor()
```
# Creating a Table

<p>We will create a new table named HAPPYMOONS3 in the database to store the data we scraped.<p>

```
newCursor.execute("Create Table HAPPYMOONS3(id INT(5) UNSIGNED AUTO_INCREMENT PRIMARY KEY,Kategori VARCHAR(75),Ürün VARCHAR(500),Aciklama VARCHAR(700),FIYAT VARCHAR(100))")
```
# Converting Data to DataFrame and CSV

# Next, we will convert the scraped data into a pandas DataFrame and save it to a CSV file.

```
# database kayıtı için hazır hale getirdik
t=0
newProduct=list()
for i in productNameList:
    newProductt="("+"'"+catagoryList[t]+"'"+","+"'"+productNameList[t]+"'"+","+"'"+productNameContentsList[t]+"'"+","+"'"+str(productPriceList[t])+"'"+")"
    newProduct.append(newProductt)
    t+=1

#Dataframe haline getirdim daha sonra csv formatında kayıt edip veritabanına kayıta için hazır halegetireceğim
newProduct=pd.DataFrame(newProduct)

newProduct.to_csv("product.csv",index=False)
```

# Inserting Data into Database
<p>Finally, we will insert the data into the HAPPYMOONS3 table using the executemany method.<p>

```
# read the csv file
data=pd.read_csv("product.csv")

query=("INSERT INTO MENU.HAPPYMOONS3(Kategori,Ürün,Aciklama,Fiyat) VALUES(%s,%s,%s,%s)")

# birden çok kayıt eklemek için bu yöntemi kullandık
newCursor.executemany(query,data)
connectDb.commit()
```

<p>The data is now successfully inserted into the MySQL database.

