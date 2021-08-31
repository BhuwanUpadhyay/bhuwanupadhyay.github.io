---
title: Convert text to speech using Google Text-to-Speech (gTTS) library
cover: /gallery/media/fsqs-tips-tricks-notes-.png
date: 2021-08-30T10:08:22.921Z
categories:
  - gTTS
tags:
  - text-to-speech
  - gTTS
  - python
---

How to Convert text to speech using Google Text-to-Speech (gTTS) library?

<!-- more -->

# gTTS

> gTTS (Google Text-to-Speech), a Python library and CLI tool to interface with Google Translate’s > text-to-speech API. Writes spoken mp3 data to a file, a file-like object (bytestring) for further > audio manipulation, or stdout. It features flexible pre-processing and tokenizing. [Read More](https://gtts.readthedocs.io/en/latest/)

## Installation

```bash
pip install gTTS
```

Verify command

```bash
gtts-cli --all
```

## Python: `requirements.txt`

```
gTTS==2.2.3
playsound==1.3.0
PyPDF2==1.26.0
```

## Convert Text to Speech: `helloTxt.py`

```python
# import dependencies
import os

from gtts import gTTS
from playsound import playsound

# convert text to speech
helloTxt="Hello Everyone! This is Bhuwan Prasad Upadhyay! Welcome to Convert Text To Speech using GTTS Library"
language='en'
helloGTTS=gTTS(text=helloTxt,lang=language,slow=False)
if os.path.exists("helloGTTS.mp3"):
  os.remove("helloGTTS.mp3")
else:
  print("The file does not exist")
helloGTTS.save("helloGTTS.mp3")

# play mp3
playsound("helloGTTS.mp3")
```

## Convert PDF Text to Speech: `pdfTxt.py`

```python
# import dependencies
import io
import PyPDF2
import os
 
from gtts import gTTS
from playsound import playsound

# read pdf as string
pdf = PyPDF2.PdfFileReader(str('paper.pdf'))
buf = io.StringIO()
for page in pdf.pages:
    buf.write(page.extractText())

# convert text to speech
pdfTxt=buf.getvalue()
language='en'
pdfGTTS=gTTS(text=pdfTxt,lang=language,slow=False)

# delete if file exits
if os.path.exists("pdfGTTS.mp3"):
  os.remove("pdfGTTS.mp3")
else:
  print("The file does not exist")

# write mp3 file  
pdfGTTS.save("pdfGTTS.mp3")

# play mp3
playsound("pdfGTTS.mp3")
```

# Github Source Code 

[Github Link](https://github.com/bhuwanupadhyay/codes/tree/main/convert-text-to-speech-using-gTTS-library)
