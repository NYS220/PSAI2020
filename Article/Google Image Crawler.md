# How to Crawl Image from Google by using icrawler?

In order to learn an image model, a variety of images must be obtained.  
But downloading images by hand is too hard and troublesome.  
There are a variety of crawling methods to solve this problem, and today I'm going to introduce you to how to use an icrawler.  

*url: https://icrawler.readthedocs.io/en/latest/builtin.html*

The following process was carried out using the 'Jupyter Notebook'

### 1. Confirm Current Work Directory
Before crawling images, by using 'os' library, we have to check 'work directory'.

```python
import os

work_dir = os.getcwd()
print(work_dir)
```
**results**  
![result](./img/1.jpg)

### 2. Make Data Directory
After checking the work directory, create a 'data directory' to download the images.  
To handle the error that occurs if there are already 'data' folder exists, use `try - except`.

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
**results**  
![result](./img/2-1.jpg)

**If exception occurs**  
![exception](./img/2-2.jpg)

### 3. Make Parent Directory
Parent Directory means that the image's class(e.g. Flowers, Cars, Mammels, etc.)  
In this article, I set the name as 'cars'  
As like #2, `try - except` is included.  

```python
try:
    os.chdir(data_dir)
    parent_dirname = input("Input Parent Directory name: ")
    os.mkdir(parent_dirname)

except Exception as err:
    # directory exists
    if os.path.exists(work_dir):
        print("\"" + parent_dirname + "\"" , "directory already exists in : ", os.getcwd())
        
    else:
        print("Wrong Directory path!")
        print("Current Directory path: ", os.getcwd())
```
**results**  
![result](./img/3-1.jpg)

**If exception occurs**  
![exception](./img/3-2.jpg)

### 4. Input Keyword to Download
Set a keywords.  
Keyword's format is as like a `A, B, C, D, ...`  
By using `search_words.replace(',', '')` and `.split()` function, keywords are separated.  
In this article, I set the keywords as 'sedan, truck, suv'.

``` python
# input keywords
search_words = input("Input Keyword to Download: ")

# replace "," to ""
if "," in search_words:
    search_words = search_words.replace(',', '')
    
search_list = search_words.split()
print(search_list)
```
**results**  
![result](./img/4.jpg)

### 5. Select Parameters(Download Limits, size, etc), Download Images
Imports `GoogleImageCrawler` from `icrawler.builtin` library.  
Then, set parameters for set image filter. Filter includes type, color, size, license, date informations.  
* `type` - photo, face, clipart, linedrawing, animated, ...
* `color` - blackandwhite, transparent, red, orange, yellow, green, blue, ...
* `size` - large, medium, icon, or larger than a given size (e.g. >640x480), or exactly is a given size (e.g. =1024x768)
* `license` - noncommercial, commercial, noncommercial modify, commercial modify, ...
* `date` - pastday, pastweek, or a tuple of dates (e.g. ((2016, 1, 1), (2017, 1, 1)))  

For download each keywords, `for img_dir in search_list` used.  


```python
from icrawler.builtin import GoogleImageCrawler

# input max_limits
max_limits = int(input("Max Crawling Limit: "))

# save dir
save_dir = parent_dirname

# default size
size_opt = ">640x480"

# type
# photo_type = 'face'

# date
# date_opt = 'pastyear'

# color
# color_opt = "color"

# default license
# license_opt = 'commercial,modify'

# create save directory
curr_dir = os.getcwd()

if not os.path.exists(save_dir):
    os.mkdir(save_dir)

os.chdir(save_dir)
parent_dir = os.getcwd()

print("Current Directory: {}".format(parent_dir))

for img_dir in search_list:
    if not os.path.exists(img_dir) :
        os.mkdir(img_dir)
        
    os.chdir(parent_dir)
    
    google_crawler = GoogleImageCrawler( 
        feeder_threads = 1,
        parser_threads = 2, 
        downloader_threads = 4,
        storage = {'root_dir': parent_dir + "/" + img_dir})
    filters = dict(
        size = size_opt
        # type = photo_type
        # license = license_opt)
    google_crawler.session.verify = False
    google_crawler.crawl(keyword = img_dir, filters = filters, max_num = max_limits, file_idx_offset = 0)
    
    print("keyword {} {} images have been saved.".format(img_dir, max_limits))
    
# restore to original directory
os.chdir(curr_dir)
```
**results**  
![result](./img/5.jpg)

### 6. Set Image Directory
Set the image directory.
```python
img_dir = data_dir + '/' + parent_dirname
print(img_dir)
```

### 7. Make all of the Images to jpeg format
When training image models, errors occur if the image format is not same.
To eliminate errors, change all the image formats to `.jpeg` format.  

```python
from PIL import Image

img_ext_list = [".jpg", ".JPG", ".gif", ".GIF", ".png", ".PNG", ".jpeg", ".JPEG"]

def verify_image(img_file):
    try:
        v_image = Image.open(img_file)
        v_image.verify()
        v_image.close()
        
        return True
    except OSError:
        return False
    
for root, dirs, files in os.walk(img_dir):
    cnt = 1
    for file in files:
        currentFile = os.path.join(root, file)
        
        file, ext = os.path.splitext(currentFile)
        
        if ext in img_ext_list:
            ext = '.jpeg'
            
            if verify_image(currentFile):
                new_file_name = os.path.dirname(currentFile)
                try:
                    final_dir = os.path.dirname(currentFile) + "/"
                    new_file_name = final_dir + os.path.basename(os.path.dirname(currentFile)) + str(cnt) + ext
                    os.rename(currentFile, new_file_name)
                    
                    cnt += 1
                    print("file saved: ", new_file_name, "\n")
                    
                except Exception as err:
                    print(err)
```

### 8. Directory List
Check the image directory list.
```python
dir_list = os.listdir(img_dir)

final_dir_list = []

for dirname in dir_list:
    final_dir_list.append(img_dir + '/' + dirname)
    
for dirname in final_dir_list:
    print(dirname)
```

### 9. Check Images
By using 'tensorflow' library, check if the images are valid and remove if it is illegal or not valid file. 
```python
import tensorflow as tf
import glob

for dirname in final_dir_list:
    for i, image_name in enumerate(glob.glob(os.path.join(dirname, '*.jpeg'))):
        print(i, image_name)
        
        with tf.Graph().as_default():
            image_contents = tf.read_file(image_name)
            image = tf.image.decode_jpeg(image_contents, channels = 3)
            
            with tf.Session() as sess:
                try:
                    sess.run(tf.global_variables_initializer())
                    tmp = sess.run(image)
                    
                except Exception as err:
                    print("[illegal file: ]", image_name, " will be removed.\n")
                    os.remove(image_name)
```
