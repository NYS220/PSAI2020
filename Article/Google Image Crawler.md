# How to Crawl Image from Google by using icrawler?

In order to learn an image model, a variety of images must be obtained.  
But downloading images by hand is too hard and troublesome.  
There are a variety of crawling methods to solve this problem, and today I'm going to introduce you to how to use an icrawler.  

*urls: https://icrawler.readthedocs.io/en/latest/builtin.html*

### 1. Confirm Current Work Directory
```python
import os

work_dir = os.getcwd()
print(work_dir)
```

### 2. Make Data Directory
```python
try:
    os.chdir(work_dir)
    
    # make directory by mkdir cmd
    os.mkdir('./data')
    
except Exception as err:
    # error occurs if there are already data folder exists
    print(err)

data_dir = work_dir + '/data'

print(data_dir)
```
If there are data directory already exists, error occurs. 

![already exists](Article/Images/already exists.jpg)
