link-code
=========
Including Pagging


import requests
import req_proxy
from bs4 import BeautifulSoup
from lxml import html
import re
import time
import logging
import os
import glob
from selenium import webdriver
from Queue import Queue
from threading import Thread
import multiprocessing
import csv
import fab_info
import sys
logging.basicConfig(level=logging.DEBUG, format='[%(levelname)s] (%(threadName)-10s) %(message)s',)


num_fetch_threads = 20
enclosure_queue = Queue()

def queue_to_print_link(f2,i, q):
    for link in iter(q.get ,None):
        try:
            #print link.strip()
            fab_info.main(f2,link.strip())
            print "inserted..............."

        except:
            pass

        q.task_done()
    q.task_done()

def main3(filename):
    f = open(filename)
    filename2 ="%s.csv"%(filename[:-4])
    f2 =open( filename2,"a+")

    proc = []
    
    for i in range(num_fetch_threads):
        proc.append(Thread(name = str(i), target=queue_to_print_link, args=(f2,i, enclosure_queue,)))       
        proc[-1].start()

    for link in f:
        enclosure_queue.put(link)
   
    enclosure_queue.join()

    for p in proc:
        enclosure_queue.put(None)

    enclosure_queue.join()

    for p in proc:
        p.join(120)

    print "done........."

    f.close() 
    f2.close()   


def main2(directory):
    filepattern = "%s/*.doc" %(directory)
    file_list = glob.glob(filepattern)
    print  file_list

    for l in file_list:
        main3(l)



def main(link):
    page = req_proxy.main(link)
    soup = BeautifulSoup(page, "html.parser")
  
    link_block = soup.find("section", attrs={"class":"catalog-products-section box-bd pan mtm"})
    link_block_ul = link_block.find("ul", attrs={"id":"productsCatalog"})
   
    link_block_ul_li = link_block_ul.find_all("li", attrs={"class":"itm hasOverlay unit onhover  last"})
  
    link_block_ul_li2 = link_block_ul.find_all("li", attrs={"class":"itm hasOverlay unit onhover  "})
    link_block_ul_li.extend(link_block_ul_li2)

    div_block_a = []

    for li in link_block_ul_li:
        div_block_a = soup.find_all("a", attrs={"class":"itm-link"})           

    file_name = soup.find("ul", attrs={"class":"lfloat"})
    file_name1 = file_name.find("li", attrs={"class":"brcItm last-child"})

    file_name2 = file_name1.find("span").get_text().strip()
    f = file_name2.replace(" & ","_")
    f1 = f.replace(" ","_")
    directory = "pro_Fachion_Files%s" %(time.strftime("%d%m%Y"))

    try:
        os.makedirs(directory)
    except:
        pass

    filename = "%s/%s.doc" %(directory,f1)
    f = open(filename, "a+")

    for link in div_block_a:
        all_href = link.get('href')
        f.write(str(all_href) + "\n")
    logging.debug("inserted...............................................")
    f.close()

    return directory
    

  


def supermain(link_list):
    str1 = "?page="

    for link in link_list:
        start = 1
        loop = True
        while (start < 15) and (loop is  True):
            str2 = link + str1 + str(start)
            try:
                directory = main(str2)
            except:
                loop = False
            start = start + 1
        

    return directory



   
if __name__=="__main__":
       
#    directory = "pro_Fachion_Files14072014"
      # directory = supermain(link_list)
    directory = supermain(["http://www.fabfurnish.com/home-decor/photo-frames-albums/new-products"])
     # directory = "pro_Fachion_Files31052014"
    #main2(directory)
