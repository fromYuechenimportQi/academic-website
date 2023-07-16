---
title: 生信算法1-滑动窗口
subtitle: 生信算法学习笔记
summary: 生信算法学习笔记
authors:
- admin
tags: [生信算法]
categories: [生信算法]
date: "2023-07-16T00:00:00Z"
lastMod: "2023-07-16T00:00:00Z"
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
**注：本系列文章是<em>[Bioinformatics Algorithms](https://www.bioinformaticsalgorithms.org/)</em>的学习笔记，包含大量的个人理解**

## 一、背景
  在物种的某个基因中，如果某个motif的重复次数很多，那么该motif对于该基因的调控可能起到非常关键的作用
  
  打个比方，我们知道，DNA聚合酶可以结合到基因组的复制起始位点(_ori_)上的DnaA box，从而复制某个基因。
  
  下面给定的序列是霍乱致病菌(_Vibrio cholerae_)的*ori*


```python
ori = '''atcaatgatcaacgtaagcttctaagcatgatcaaggtgctcacacagtttatccacaac
ctgagtggatgacatcaagataggtcgttgtatctccttcctctcgtactctcatgacca
cggaaagatgatcaagagaggatgatttcttggccatatcgcaatgaatacttgtgactt
gtgcttccaattgacatcttcagcgccatattgcgctggccaaggtgacggagcgggatt
acgaaagcatgatcatggctgttgttctgtttatcttgttttgactgagacttgttagga
tagacggtttttcatcactgactagccaaagccttactctgcctgacatcgaccgtaaat
tgataatgaatttacatgcttccgcgacgatttacctcttgatcatcgatccgattgaag
atcttcaattgttaattctcttgcctcgactcatagccatgatgagctcttgatcatgtt
tccttaaccctctattttttacggaagaatgatcaagctgctgctcttgatcatcgtttc'''.replace('\n','')
```

我们如何找到它的DnaA box呢？

**研究者们已通过实验证明，DnaA box通常是一段9-mer(长度为9bp的核苷酸序列)**

## 二、思路

我们可以将这个问题看作是：在一个给定的字符串中，找到**出现频率最高的**子字符串，而我们知道DnaA box通常是9-mer，因此，我们需要找到**出现频率最高、且长度为9的**子字符串



## 三、解题

1. 要找到子字符串，我们需要使用双指针法

2. 我们定义两个指针，start_pointer和end_pointer，并使**index(end_pointer)-index(start_pointer)=9**

3. 两个指针同时往前走，遍历字符串

由于两个指针的行动方式酷似一个在滑动的窗子，因此该方法又叫做滑动窗口(slide a window)

要找到出现频率最高的子字符串，我们首先需要计算子字符串在字符串中出现的次数


```python
def count(text, pattern):
    #初始化计数器
    count = 0
    for i in range(len(text) - len(pattern) + 1):
        #滑动窗口start_pointer和end_pointer
        start_pointer = i
        end_pointer = i + len(pattern)
        #字符串切片得到子字符串
        if text[start_pointer:end_pointer] == pattern:
            count += 1
    return count

text = "ACGTTTCACGTTTTACGG"
pattern = "ACG"
print(count(text, pattern))
```

    3
    

接下来，我们就可以统计出现频率最高、且长度为9的子字符串了


```python
def frequent_words(text,k):
    frequent_patterns = []
    count_list = []
    #计算每个kmer的频率
    for i in range(len(text) - k + 1):
        pattern = text[i:i+k]
        count_list.append(count(text,pattern))
    #找到最大频率
    max_count = max(count_list)
    #找到最大频率的kmer
    for i in range(len(text) - k + 1):
        if count_list[i] == max_count:
            frequent_patterns.append(text[i:i+k])
    return list(set(frequent_patterns))

frequent_words(ori,9)
```




    ['cttgatcat', 'ctcttgatc', 'atgatcaag', 'tcttgatca']



我们通过上述方法，找到了出现次数最多的9-mer，但是该算法真的好吗？

我们在寻找9-mer的过程中，遍历了字符串，但是在计算9-mer出现频率时，又从头到尾遍历了字符串，那么该算法的时间复杂度为O(n^2)

这个问题有更好的算法吗？

我们不妨这么想，假设手头只有纸笔，我们会边找k-mer，边把该k-mer及对应的出现次数记录在纸上，再次遇到该k-mer时，次数+1，那么这样就只需要遍历一次就好了！

在计算机中，该方法成为哈希法，是一种经典的用空间换取时间的方法


```python
def frequent_words_hash(text,k):
    #初始化哈希表
    frequent_dict = {}
    #计算每个kmer的频率
    for i in range(len(text) - k + 1):
        pattern = text[i:i+k]
        if pattern in frequent_dict.keys():
            frequent_dict[pattern] += 1
        else:
            frequent_dict[pattern] = 1
    #找到最大频率
    max_count = max(frequent_dict.values())
    #找到最大频率的kmer
    frequent_patterns = []
    for key in frequent_dict.keys():
        if frequent_dict[key] == max_count:
            frequent_patterns.append(key)
    return frequent_patterns

print(frequent_words_hash(ori,9))
```

    ['atgatcaag', 'ctcttgatc', 'tcttgatca', 'cttgatcat']
    
