---
title: Selenium学习笔记1
subtitle: Selenium最佳实践
summary: 爬虫学习
authors:
- admin
tags: [爬虫]
categories: [爬虫]
date: "2023-07-18T00:00:00Z"
lastMod: "2023-07-18T00:00:00Z"
featured: false
draft: false

# Featured image
# To use, add an image named `featured.jpg/png` to your page's folder. 
image:
  caption: ""
  focal_point: ""

# Projects (optional).
#   Associate this post with one or more of your projects.
#   Simply enter your project's folder or file name without extension.
#   E.g. `projects = ["internal-project"]` references 
#   `content/project/deep-learning/index.md`.
#   Otherwise, set `projects = []`.
projects: []
---

## 一、Selenium简介

[Selenium](https://www.selenium.dev/zh-cn/documentation/webdriver/getting_started/)是一个网页自动化工具，提供了Java、Python和JS等语言的接口

在爬虫项目中，我们常常会遇到一个页面是通过JS渲染的，这就造成爬取html的不便

而Selenium可以模拟鼠标的操作，达到爬取的目的

## 二、Selenium实践

以网站[Published Plant Genomes](https://www.plabipd.de/plant_genomes_pa.ep)为例，该网站前端页面非常漂亮，但是Published信息需要将鼠标移动到叶子(leaf)上才能显示，而该效果是通过JS渲染得到的，传统的beautifulSoup的HTML解析对此只能表示无能为力


```python
url = 'https://www.plabipd.de/plant_genomes_pa.ep'
```

测试Chrome浏览器能否打开


```python
from selenium import webdriver
#初始化driver
diver = webdriver.Chrome()
#访问域名
diver.get(url)
#关闭浏览器
diver.close()
```

找到所有的leaf


```python

from selenium.webdriver.common.by import By
import time
driver = webdriver.Chrome()
driver.get(url)
##由于渲染是耗时操作，所以需要阻塞线程，等待渲染完成，这里等待5秒
time.sleep(5)
leaves = driver.find_elements(By.CLASS_NAME,'leaf')
for i in leaves:
    print(i.text)
driver.close()

```

    Amborella trichopoda
    Euryale ferox
    Nymphaea colorata
    Nymphaea thermarum
    ...
    ...
    Bursera cuneata
    Bursera palmeri
    Commiphora wightii
    

模拟鼠标操作


```python
from selenium.webdriver.common.action_chains import ActionChains
from selenium.webdriver.support.wait import WebDriverWait
from selenium.webdriver.support import expected_conditions as EC
from selenium.common.exceptions import TimeoutException
import time
driver = webdriver.Chrome()
driver.get(url)
#渲染耗时，因此阻塞，等待渲染完成，这里等待5秒
time.sleep(5)
#找到所有leaf元素
leaves = driver.find_elements(By.CLASS_NAME,'leaf')
for leaf in leaves[:5]:
    ActionChains(driver).move_to_element(leaf).perform()
    wait = WebDriverWait(driver, timeout=10,ignored_exceptions=TimeoutException)
    element = wait.until(EC.presence_of_element_located((By.CLASS_NAME, 'd3-tip')))
    print(element.get_attribute('innerHTML'))
    print('------------------')
driver.close()

```

    <span> Amborella trichopoda</span><br>genome size: 748 Mbp<p class="refset"><span> published: 2013-12-20</span>Amborella Genome Project<br>The Amborella genome and the evolution of flowering plants.<br><a href="http://dx.doi.org/10.1126/science.1241089">Science. 2013 Dec 20 342(6165):1241089.</a></p>
    ------------------
    <span> Euryale ferox</span> <br>(Prickly waterlily)<br>genome size: 768 Mbp<p class="refset"><span> published: 2020-02-24</span>Yang Y et al.<br>Prickly waterlily and rigid hornwort genomes shed light on early angiosperm evolution.<br><a href="http://dx.doi.org/10.1038/s41477-020-0594-6">Nat Plants. 2020 Feb 24, Epub ahead of print</a></p>
    ------------------
    <span> Nymphaea colorata</span> <br>(Blue pigmy water lily)<br>genome size: 409 Mbp<p class="refset"><span> published: 2019-12-18</span>Zhang L et al.<br>The water lily genome and the early evolution of flowering plants.<br><a href="http://dx.doi.org/10.1038/s41586-019-1852-5">Nature. 2019 Dec 18, Epub ahead of print</a></p>
    ------------------
    <span> Nymphaea thermarum</span> <br>(Rwandan pigmy water lily)<br>genome size: 500 Mbp<p class="refset"><span> published: 2020-03-31</span>Povilus RA et al.<br>Water lily (Nymphaea thermarum) genome reveals variable genomic signatures of ancient vascular cambium losses.<br><a href="http://dx.doi.org/10.1073/pnas.1922873117">Proc Natl Acad Sci U S A. 2020 Mar 31, Epub ahead of print</a></p>
    ------------------
    <span> Aristolochia contorta</span> <br>(Northern pipevine)<br>genome size: 290 Mbp<p class="refset"><span> published: 2022-02-11</span>Cui X et al.<br>Chromosome-level genome assembly of Aristolochia contorta provides insights into the biosynthesis of benzylisoquinoline alkaloids and aristolochic acids.<br><a href="http://dx.doi.org/10.1093/hr/uhac005">Hortic Res. 2022 Feb 11: uhac005. Online ahead of print.</a></p>
    ------------------
    

找到div.d3-tip后，其内部的html可以通过beautifulSoup4来解析，进而找到我们需要的所有信息！

使用beautifulSoup4进行HTML的解析


```python
from bs4 import BeautifulSoup
class Publication:
    def __init__(self,published_date=None,author=None,title=None,journal=None,doi=None):
        self.published_date = published_date
        self.author = author
        self.title = title
        self.journal = journal
        self.doi = doi
    def __str__(self):
        return f'发表日期:{self.published_date}\n作者:{self.author}\n标题:{self.title}\n期刊:{self.journal}\ndoi:{self.doi}'
class Paper:
        def __init__(self,species:str,genome_size:str,publication):
            self.species = species
            self.genome_size = genome_size
            self.publication = publication
        def __str__(self):
            return f'物种:{self.species}\n基因组大小:{self.genome_size}'
class PPGParser():
    def __init__(self,html):
        self.html = html
        self.soup = BeautifulSoup(html,'html.parser')
    def get_species(self):
        species = self.soup.find('span')
        return species.text.strip()
    def get_genome_size(self):
        genome_size = self.soup.find('p').previous_sibling.strip().split(':')[-1].strip()
        return genome_size
    def get_publish_information(self):
        result = []
        papers = self.soup.find_all('p',attrs={'class':'refset'})
        for paper in papers:
            publication = Publication()
            publication.published_date = paper.find('span').text.strip().split(':')[-1].strip()
            publication.author = paper.find('span').next_sibling.strip()
            publication.title = paper.find_all('br')[-1].previous_sibling.strip()
            publication.doi = paper.find('a').get('href')
            publication.journal = paper.find('a').text
            ##由于后续要输出一个excel，因此这里将publication对象转换为字典，之后json.loads方便转换为json
            result.append(publication.__dict__)
        return result
    def parse(self):
        species = self.get_species()
        genome_size = self.get_genome_size()
        publication = self.get_publish_information()
        paper = Paper(species,genome_size,publication)
        return paper
test = """
<span> Amborella trichopoda</span><br>genome size: 748 Mbp<p class="refset"><span> published: 2013-12-20</span>Amborella Genome Project<br>The Amborella genome and the evolution of flowering plants.<br><a href="http://dx.doi.org/10.1126/science.1241089">Science. 2013 Dec 20 342(6165):1241089.</a></p>
"""
ppg = PPGParser(test)
result = ppg.parse()
print(result)
for i in result.publication:
    print(i)

```

    物种: Amborella trichopoda
    基因组大小:748 Mbp
    {'published_date': '2013-12-20', 'author': 'Amborella Genome Project', 'title': 'The Amborella genome and the evolution of flowering plants.', 'journal': 'Science. 2013 Dec 20 342(6165):1241089.', 'doi': 'http://dx.doi.org/10.1126/science.1241089'}
    

OK，经测试，爬取任务完成！

接下来便是保存数据啦，我们可以用pandas包，将结果写成一个excel~


```python
import pandas as pd
import json
def save_PPG_as_excel(papers:list):
    df = pd.DataFrame(columns=['物种名','基因组大小','发表情况'])
    for paper in papers:
        #print(paper)
        df = df.append({'物种名':paper.species,'基因组大小':paper.genome_size,'发表情况':json.dumps(paper.publication,ensure_ascii=False)},ignore_index=True)
    df.to_excel('PPG.xlsx',index=False)    
    
```

那么综合所有代码块，达成最终目的！


```python
from selenium.webdriver.common.action_chains import ActionChains
from selenium.webdriver.support.wait import WebDriverWait
from selenium.webdriver.support import expected_conditions as EC
from selenium.common.exceptions import TimeoutException
import time
papers = []
driver = webdriver.Chrome()
driver.get(url)
#渲染耗时，因此阻塞，等待渲染完成，这里等待5秒
time.sleep(5)
#找到所有leaf元素
leaves = driver.find_elements(By.CLASS_NAME,'leaf')
#取前五个做测试~
for leaf in leaves[:5]:
    ActionChains(driver).move_to_element(leaf).perform()
    wait = WebDriverWait(driver, timeout=10,ignored_exceptions=TimeoutException)
    element = wait.until(EC.presence_of_element_located((By.CLASS_NAME, 'd3-tip')))
    html = element.get_attribute('innerHTML')
    ppg = PPGParser(html)
    paper = ppg.parse()
    papers.append(paper)
driver.close()
save_PPG_as_excel(papers)
```

    
