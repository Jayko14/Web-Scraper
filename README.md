# Web-Scraper
from decimal import Decimal
import requests
import bs4
from re import sub
import xlrd

#ToDo
#dynamic excel range


maxprice=30
special="@"

#SL= open('ShoppingList.txt','w')

#get array for unique words to find here

uniquewords=set()
n64location="C:\\Users\\maxtw\\PycharmProjects\\N64Shopper\\N64.xlsx"
workbook=xlrd.open_workbook(n64location)
sheet=workbook.sheet_by_name("Uniques")
for j in range(270):
    yae=sheet.cell_value(j,28)
    if yae is not 'y':
        word = sheet.cell_value(j, 27)
        uniquewords.add(word)
print(uniquewords)



def urls(pgmax='2'):
    pgnum = 1

    while pgnum <= pgmax:
        url = "https://www.ebay.com.au/b/Video-Games/139973?LH_BIN=1&LH_PrefLoc=0&Platform=Nintendo%252064&Region%2520Code=PAL%7CRegion%2520Free&rt=nc&_dcat=139973&_fcid=15&_pgn="+str(pgnum)+"&_sop=15&_stpos=2000"
        siter = requests.get(url)
        n=bs4.BeautifulSoup(siter.text,"html.parser").findAll('li',{'class':'s-item'})
        for i in n:
            title = i.find('h3', {'class': 's-item__title'}).text
            checker=str(title).lower()
            exceptions={'instruction','booklet','magazine','manual'} #put words in here that you want to always exclude
            checkervariable = "yes"
            for j in exceptions:
                if j in checker:
                    checkervariable = "no"
                else:continue
            if checkervariable is "yes": #"instruction" not in str(title).lower() and "booklet" not in str(title).lower():
                for k in uniquewords:
                    o=str(k).lower()+" "
                    p=" "+str(k).lower()
                    if (o or p) in str(title).lower():
                        price = i.find('span', {'class': 's-item__price'}).text
                        linker = i.find('a').get('href')
                        try:
                            postage=i.find('span',{'class':'s-item__shipping s-item__logisticsCost'}).text
                        except AttributeError:
                            postage='Check Postage'
                        try:
                            pricevalue = Decimal(sub(r'[^\d.]', '', price))
                            postagevalue = Decimal(sub(r'[^\d.]', '', postage))
                            summer=pricevalue+postagevalue
                            if summer > maxprice: pgnum = pgmax + 1
                        except:
                            try:
                                summer=pricevalue
                            except UnboundLocalError:
                                summer=price
                                pass
                        try:
                            if summer > maxprice: pgnum = pgmax + 1
                        except TypeError:
                            pass

                        try:
                            #SL.write(o + str(title) + special + str(price) + special + str(postage) + special + str(summer) + special + linker + '\n')
                            print(o+"| "+title+" "+linker)
                            #print(title)
                        except UnicodeEncodeError:
                            print("Could not write to file "+title)
                    else: continue
                else: continue
        print(pgnum)
        pgnum = pgnum+1



urls(5)
