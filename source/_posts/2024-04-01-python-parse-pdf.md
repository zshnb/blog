---
title: Python识别PDF内容
date: 2024-04-01 16:22:23
tags:
  - Python
---
最近在开发的产品[CreateDeep](https://createdeep.com/)需要支持用户上传pdf文件，并读取pdf文件中的文本部分和GPT沟通。实现过程中也是踩了不少坑，正好和大家分享一下。
<!--more-->
首先pdf有两种类别，一种是通过文本直接生成的，其特点是打开pdf可以直接复制里面的文字，像这样

![文字类pdf](img1.webp)
对于这种类型的pdf，识别非常简单，用python的`pymupdf` 库即可，实现代码也是简洁利落

```
import fitz
def parse_text_from_text_pdf():
    with fitz.open(pdf_file) as doc:  # open document
        text = chr(12).join([page.get_text() for page in doc])

    print(text)

```
还有一种是扫描的pdf，其特点是文本无法直接复制，每一页都是一张图片，像这样

![扫描类pdf](img2.webp)
对于这种类型的pdf，识别会复杂一些，但我们可以一步步拆解。

### 逻辑分析
1. 首先我们把pdf当作图片进行处理，对于图片的文字识别是非常成熟的，python有很多OCR的库可以使用，开源最好用的当然是Google的`tesseract` 。

1. 在前面的基础上，只要想办法把pdf文件转换成一堆的图片，就能完成识别。python有另一个库`pdf2image` ，可以完成这个工作。

每个步骤都清楚之后，剩下的就需要把所有串在一起。这里有个要注意的是，`tesseract` 默认智能识别英文，中文识别的结果是乱码，需要单独安装一个中文的训练集后才可以正常识别中文。

```
import sys
from tempfile import TemporaryDirectory
import pytesseract
from pdf2image import convert_from_path
from PIL import Image
import os

def parse_text_from_scan_pdf():
    with TemporaryDirectory() as tempdir:
        output_text = ''
        pdf_pages = convert_from_path(pdf_file, 500)
        os.mkdir(f"{tempdir}/{os.path.basename(pdf_file)}")
        tessdata_dir_config = f'--tessdata-dir "{os.getcwd()}/tools/pdfParser"'
        for page_enumeration, page in enumerate(pdf_pages, start=1):
            filename = f"{tempdir}/{os.path.basename(pdf_file)}/{page_enumeration}.jpg"
            page.save(filename, "jpeg")
            text = str((pytesseract.image_to_string(Image.open(filename), lang="chi_sim", config=tessdata_dir_config)))
            text = text.replace("-\n", "")
            output_text += text
            os.remove(filename)

    print(output_text)

```
代码中的`tessdata_dir_config` 是tesseract的指定训练模型，除了自带的默认模型外，tesseract还提供了识别速度型和识别准确型，有需要可以去[官网](https://tesseract-ocr.github.io/tessdoc/Data-Files.html)下载，然后在代码里指定使用哪个训练模型。个人的使用体验是，中文建议还是用准确型，默认的识别准确度太差，一句话中有1/3都是错误的。

### 依赖安装
代码写完，接下来就需要部署上线了，我用的是Ubuntu22.04 LTS，首先需要安装如下python的包

```
pymupdf==1.23.26
pytesseract==0.3.10
pdf2image==1.17.0

```
接着是系统依赖库`sudo apt install -y poppler-utils tesseract-ocr tesseract-ocr-chi-sim libtesseract-dev` ，安装完就可以正常识别了。

![识别结果](img3.webp)

