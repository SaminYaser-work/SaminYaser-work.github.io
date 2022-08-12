---
title: "Generating Word Cloud from Bengali Text using Python"
date: 2022-08-10T18:49:26+06:00
draft: false
tags: ["python", "wordcloud", "Bengali", "NLP"]
toc: true
---


{{< figureCupper
img="wc1.png" 
caption="Bengali wordcloud"
command="Resize" 
options="700x" >}}

## Introduction
Word cloud is a visual representation of a set of words. It is a useful tool for understanding the frequency of words in a body of text. Word clouds are often used to describe the contents of a dataset in various **NLP** related projects.

In this tutorial, we will generate a word cloud from a Bengali dataset using `wordcloud` library in python. We will be working with a dataset consists of Wikipedia articles, most commonly known as the **Wikipedia Dataset**. It can be downloaded from [here](https://huggingface.co/datasets/wikipedia). You can also get it from [here](https://dumps.wikimedia.org/), but in this case you have to manually **extract and clean** the articles from the XML file. Maybe I'll write a tutorial on how to do that in the future.

Note that it is a big dataset, so the code in this tutorial is optimized for memory.

## Importing Libraries
First, we import the necessary libraries.

```python
import re
import numpy as np
from PIL import Image
from wordcloud import WordCloud, ImageColorGenerator
import matplotlib.pyplot as plt
from collections import Counter
```
`re`, `wordcloud` and `matplotlib` are the libraries we need to generate and display the word cloud. 

`numpy` and `PIL` are needed if you want generate the word cloud in a specific shape, so it can be ommitted if your are not interested in that.

`Counter` is recommended if you have a large dataset and low on RAM.

## Removing Stop Words
Stop words are words which are not important to the meaning of the text. They are usually used most frequently in the text. So, generating a wordcloud with stop words will not give us the most meaningful result. Look at the figure below and see for yourself.

{{< figureCupper
img="wc3.png" 
caption="Bengali Wordcloud with stop words"
command="Resize" 
options="700x" >}}

Easiest way to remove stop words are to use [bnlp-toolkit](https://pypi.org/project/bnlp-toolkit/) library. This is how you do it.

```python
# Install bnlp-toolkit with this command
# pip install bnlp-toolkit

from bnlp.corpus import stopwords
from bnlp.corpus.util import remove_stopwords

raw_text = 'আমি ভাত খাই।' 
result = remove_stopwords(raw_text, stopwords)
print(result)
# ['ভাত', 'খাই', '।']
```

Pretty easy, right? However, if you have your own list of stopwords, you can do this to clean your dataset.

```python
with open('../Datasets/stopwords.txt', 'r', encoding='utf-8') as fin:
  stopwords = fin.read().splitlines()

with open('../Datasets/wiki/all_v2_clean_sentence_no_stopwords.txt', 'w', encoding='utf-8') as fout:
  with open('../Datasets/wiki/all_v2_clean_sentence.txt', 'r', encoding='utf-8') as fin:
    for line in tqdm(fin):
      words = line.split(' ')
      clean_line = [word for word in words if word not in stopwords]
      clean_line = ' '.join(clean_line)
      fout.write(clean_line)
```
Here, we are cleaning the dataset from stop words *line by line*, which is a bit slow, but saves memory if you have a large dataset. Feel free to do it all at once if your dataset is small.

Note that you are also saving the cleaned dataset in a new text file so that we don't have to repeat this time consuming process ever again.

## Getting the most frequent words
This step is optional if you have a small dataset. But if you have a large dataset, `wordcloud` function might eat up all your memory and crash your PC. To stop this, we use `Counter` from `collections` library, which can count the frequency of each word in a dataset in a memory friendly way. We can then use these frequently used words to generate the word cloud. In my experience, this method is also faster than just simply passing the whole dataset to generate the word cloud.

```python
with open('../Datasets/wiki/all_v2_clean_sentence_no_stopwords.txt', 'r', encoding='utf-8') as fin:
  text = fin.read().split(' ')

counter = Counter(text)

most_common_words = counter.most_common(2000) # Use 2000 most common words to generate the word cloud
```

## Generating the Word Cloud ☁️
Finally, we can generate the word cloud. We are also giving it the shape of Bangladeshi flag.

I also suggest getting a Bengali font. I am using [Nirmala UI](https://www.wfonts.com/font/nirmala-ui), which is the default Bengali font in _Windows_.


{{< figureCupper
img="bd.jpg" 
caption="Flag of Bangladesh"
command="Resize" 
options="300x" >}}

```python
regex = r"([\S]+)"
mask = np.array(Image.open("bd.jpg")) # Load your picture here

def plot_world_cloud(text):

    wordcloud = WordCloud(
        # Width and height of the word cloud image
        width = 1000, 
        height = 500, 

        mode='RGBA',
        background_color ='white', 
        font_path='fonts/nirmala.ttf', # Use your own font here
        regexp=regex,
        collocations=True,
        mask=mask,

        # Maximum number of words in the word cloud
        max_words=2000,
    )

    wordcloud.generate(text) 

    # plot the WordCloud image                        
    image_colors = ImageColorGenerator(mask)
    plt.figure(figsize = (15, 15)) 
    plt.imshow(wordcloud.recolor(color_func=image_colors), interpolation="bilinear")
    plt.axis("off") 
    plt.tight_layout(pad = 0) 

    plt.show() 

plot_world_cloud(' '.join([i[0] for i in most_common_words]))
```
{{< warning >}}
Passing `regex = r"([\S]+)"` to the `wordcloud` function is **crucial**. Otherwise, your output image might look something like this where conjunctive letters are broken.
{{< /warning >}}

{{< figureCupper
img="wc2.png" 
caption="Broken Conjunctive Letters"
command="Resize" 
options="700x" >}}

## Alternatives for Word Cloud
You might also want check out [this](https://www.kaggle.com/code/paultimothymooney/most-common-words-on-kaggle-wordcloud-bargraph/notebook) where a guy generated a bar chart out of the most common words in his dataset. This may not be as good looking as a word cloud, but if you want go a more statistical route, this is the way to go.

## Conclusion
This is a good starting point for generating word clouds. Hope it helped you. Heres a Github [gist]() with the code.