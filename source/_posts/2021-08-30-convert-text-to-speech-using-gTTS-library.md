---
title: Convert text to speech using Google Text-to-Speech (gTTS) library
cover: /images/2021-08-convert-text-to-speech-using-gTTS-library.png
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

## Python: `requirements.txt`

```
gTTS==2.2.3
playsound==1.3.0
PyPDF2==1.26.0
```

## Installation

```bash
pip install -r requirements.txt
```

## Convert Text to Speech: `helloTxt.py`

```python
# import dependencies
import os

from gtts import gTTS
from playsound import playsound

outputFile='helloGTTS.mp3'

# convert text to speech
helloTxt="Hello Everyone! This is Bhuwan Prasad Upadhyay! Welcome to Convert Text To Speech using GTTS Library"
language='en'
helloGTTS=gTTS(text=helloTxt,lang=language,slow=False)
if os.path.exists(outputFile):
  os.remove(outputFile)
else:
  print("The file does not exist")

# write mp3 file  
helloGTTS.save(outputFile)

# play mp3
playsound(outputFile)
```

To run:

```bash
python3 helloTxt.py
```

## Convert PDF Text to Speech: `pdfTxt.py`

```python
# import dependencies
import io
import PyPDF2
import os
 
from gtts import gTTS
from playsound import playsound

inputFile='paper.pdf'
outputFile='pdfGTTS.mp3'

# read pdf as string
pdf = PyPDF2.PdfFileReader(str(inputFile))
buf = io.StringIO()
for page in pdf.pages:
    buf.write(page.extractText())

# convert text to speech
pdfTxt=buf.getvalue()
language='en'
pdfGTTS=gTTS(text=pdfTxt,lang=language,slow=False)

# delete if file exits
if os.path.exists(outputFile):
  os.remove(outputFile)
else:
  print("The file does not exist")

# write mp3 file  
pdfGTTS.save(outputFile)

# play mp3
playsound(outputFile)
```

To run:

```bash
python3 pdfTxt.py
```

---

### [Github Source](https://github.com/bhuwanupadhyay/codes/tree/main/convert-text-to-speech-using-gTTS-library)
