# WebSocket API Machine Readable News Example with Python
- Last update: May 2021
- Environment: Windows and Linux OS 
- Compiler: Python
- Prerequisite: Refinitiv Real-Time Advanced Data Hub and Refinitiv Real-Time Advanced Distribution servers version 3.2.1 and above, MRN service

## Overview

This example shows how developers may use the [Websocket API for Pricing Streaming and Real-Time Service](https://developers.refinitiv.com/en/api-catalog/refinitiv-real-time-opnsrc/refinitiv-websocket-api) aka Websocket API application to subscribe Machine Readable News (MRN) from Refinitiv Real-Time Distribution System (Refinitiv Real-Time Advanced Data Hub and Refinitiv Real-Time Advanced Distribution Server). The example just connects to Refinitiv Real-Time via a WebSocket connection, then subscribes and displays MRN News data in a console or classic Jupyter Notebook. The project is implemented with Python language for both console and Jupyter Notebook applications, but the main concept for consuming and assembling MRN News messages are the same for all technologies. 

For example of MRN from Refinitiv Real-Time Optimized (formerly known as ERT in Cloud), please switch to [ERT-in-Cloud](https://github.com/Refinitiv-API-Samples/Example.WebSocketAPI.Python.MRN/tree/ERT-in-Cloud) branch.

Please see the full documentation of this example application in [this article](https://developers.refinitiv.com/en/article-catalog/article/introduction-machine-readable-news-elektron-websocket-api-refinitiv).

*Note:* The news message is in UTF-8 JSON string format. Some news messages that contain special Unicode character may not be able to show in Windows OS console (cmd, git bash, powershell, etc) due to the OS limitation. Those messages will be print as ```UnicodeEncodeError exception. Cannot decode Unicode character``` message in a console instead.

## Prerequisite

This example is focusing on the Refinitiv Machine Readable News (MRN) data processing only. I highly recommend you check the  [WebSocket API Tutorials](https://developers.refinitiv.com/en/api-catalog/refinitiv-real-time-opnsrc/refinitiv-websocket-api/tutorials) page if you are not familiar with WebSocket API. 

The Tutorials page provides a step-by-step guide (connect, log in, request data, parse data, etc) for developers who are interested in developing a WebSocket application to consume real-time data from Refinitiv Real-Time. 

## Machine Readable News Overview

Refinitiv Machine Readable News (MRN) is an advanced service for automating the consumption and systematic analysis of news. It delivers deep historical news archives, ultra-low latency structured news and news analytics directly to your applications. This enables algorithms to exploit the power of news to seize opportunities, capitalize on market inefficiencies, and manage event risk.

### MRN Data model

MRN is published over Refinitiv Real-Time using an Open Message Model (OMM) envelope in News Text Analytics domain messages. The Real-time News content set is made available over MRN_STORY RIC. The content data is contained in a FRAGMENT field that has been compressed and potentially fragmented across multiple messages, to reduce bandwidth and message size.

A FRAGMENT field has a different data type based on a connection type:
* RSSL connection (RTSDK [C++](https://developers.refinitiv.com/en/api-catalog/refinitiv-real-time-opnsrc/rt-sdk-cc)/[Java](https://developers.refinitiv.com/en/api-catalog/refinitiv-real-time-opnsrc/rt-sdk-java)): BUFFER type
* WebSocket connection: Base64 ASCII string

The data goes through the following series of transformations:

1. The core content data is a UTF-8 JSON string
2. This JSON string is compressed using gzip
3. The compressed JSON is split into several fragments (BUFFER or Base64 ASCII string) which each fit into a single update message
4. The data fragments are added to an update message as the FRAGMENT field value in a FieldList envelope

![Figure-1](diagram/mrn_process.png "MRN data compression process") 

Therefore, to parse the core content data, the application will need to reverse this process. The WebSocket application also needs to convert a received Base64 string in a FRAGMENT field to bytes data before further process this field. This application uses Python [base64](https://docs.python.org/3/library/base64.html) and [zlib](https://docs.python.org/3/library/zlib.html) modules to decode Base64 string and decompress JSON string. 

If you are not familiar with MRN concept, please visit the following resources which will give you a full explanation of the MRN data model and implementation logic:
* [Webinar Recording: Introduction to Machine Readable News](https://developers.refinitiv.com/news#news-accordion-nid-12045)
* [Introduction to Machine Readable News (MRN) with Enterprise Message API (EMA)](https://developers.refinitiv.com/en/article-catalog/article/introduction-machine-readable-news-mrn-elektron-message-api-ema).
* [MRN Data Models and Refinitiv Real-Time SDK Implementation Guide](https://developers.refinitiv.com/en/api-catalog/elektron/elektron-sdk-java/documentation#mrn-data-models-implementation-guide).
* [Introduction to Machine Readable News with WebSocket API](https://developers.refinitiv.com/en/article-catalog/article/introduction-machine-readable-news-elektron-websocket-api-refinitiv).

## Example Prerequisite
This example requires the following dependencies software and libraries.
1. Refinitiv Real-Time Advanced Data Hub and Refinitiv Real-Time Advanced Distribution Server version 3.2.x with WebSocket connection and MRN Service.
2. [Python](https://www.python.org/) compiler and runtime
3. Python's [requests 2.x](https://pypi.org/project/requests/) library.
4. Python's [websocket-client](https://pypi.org/project/websocket-client/) library (*version 0.49 or greater*).
5. [Classic Jupyter Notebook](https://jupyter.org/) runtime (for the Notebook example only)
6. [Docker Engine - Community Edition](https://docs.docker.com/install/) (for running a console example in Docker only)

*Note:* 
- The Python example has been qualified with Python versions 3.6.5 and Python 3.7.4 (Docker 19.03.1 - CentOS 7)
- Please refer to the [pip installation guide page](https://pip.pypa.io/en/stable/installing/) if your environment does not have the [pip tool](https://pypi.org/project/pip/) installed. 
- If your environment already have a websocket-client library installed, you can use ```pip list``` command to verify a library version, then use ```pip install --upgrade websocket-client``` command to upgrade websocket-client library. 
- It is not advisable to change the Refinitiv Real-Time Distribution System configuration if you are not familiar with the configuration procedures. Please consult your Market Data administrator for any questions regarding Refinitiv Real-Time Distribution System-MRN service configuration.


## Application Files
This example project contains the following files and folders
1. *mrn_console_app.py*: The example application file
2. *notebook_python/mrn_notebook_app.ipynb*: The example Jupyter Notebook application file
3. *Dockerfile*: The example application Dockerfile
3. *requirements.txt*: The application dependencies configuration file
4. LICENSE.md: Project's license file
5. README.md: Project's README file

## How to run this example

Please be informed that your Refinitiv Real-Time Advanced Data Hub and Refinitiv Real-Time Advanced Distribution Server should have a Service that contains MRN data. The first step is to unzip or download the example project folder into a directory of your choice, then choose how to run the application based on your environment below.

### A console example
1. Go to the project folder in the console
2. Run ```$> pip install -r requirements.txt``` command in a console to install all the dependencies libraries.
3. Then you can run mrn_console_app.py application with the following command
    ```
    $> python mrn_console_app.py --hostname <Real-Time Advanced Distribution Server IP Address/Hostname> --port <WebSocket Port> --ric <MRN RIC name>
    ```
Optionally, the application subscribes ```MRN_STORY``` RIC code from Real-Time Advanced Distribution Server by default. You can pass your interested MRN RIC code to ```--ric``` parameter on the application command line. The supported MRN RIC codes are ```MRN_STORY```, ```MRN_TRNA```, ```MRN_TRNA_DOC``` and ```MRN_TRSI``` only. the application 

### Docker example
1. Go to the project folder in the console 
2. Run ```$> docker build -t <project tag name> .``` command in a console to build an image from a Dockerfile.
    ```
    $> docker build -t rtsdk_ws_mrn_python .
    ```
3. Once the build is a success, you can create and run the container with the following command
    ```
    $> docker run rtsdk_ws_mrn_python --hostname Real-Time Advanced Distribution Server IP Address/Hostname> --port <WebSocket Port> --ric <MRN RIC name>
    ```
4. If you want to connect the Docker container to a localhost, please use ```host.docker.internal``` as the Host name. 

### Classic Jupyter Notebook example

Please be informed that  Python [Anaconda](https://www.anaconda.com/distribution/) or [MiniConda](https://docs.conda.io/en/latest/miniconda.html) distribution/package manager is highly recommended for running the Jupyter Notebook example.

1. Open Anaconda Prompt and go to the project's folder
2. Run the following command in an Anaconda Prompt to create a Conda environment named *mrn_python_notebook* for the project.
  ```
  (base) $>conda create --name mrn_python_notebook python=3.7
  ```
3. Once the environment is created, activate Conda environment named ```mrn_python_notebook``` with this command in Anaconda Prompt
  ```
  (base) $>conda activate mrn_python_notebook
  ```
4. In mrn_python_notebook environment, install the following prerequisite libraries
  ```
  (mrn_python_notebook) $>conda install -c conda-forge notebook

  (mrn_python_notebook) $>conda install -c conda-forge pandas

  (mrn_python_notebook) $>pip install -r requirements.txt
  ```
5. Go to the project's notebook folder in the console 
6. Run the following command in a console to start the classic Jupyter Notebook in the notebook folder.
  ```
  (mrn_python_notebook) $>jupyter notebook
  ```
7. Open *mrn_notebook_app.ipynb* Notebook document, then follow through each notebook cell.

*Note:* 
- You can install a classic Jupyter Notebook on your local machine and then test the example on the machine. The alternate choice is a free Jupyter Notebook on cloud environments such as [Azure Notebook](https://notebooks.azure.com/) provided by Microsoft. You can find more details from [this tutorial](https://docs.microsoft.com/en-us/azure/notebooks/tutorial-create-run-jupyter-notebook). If you are not familiar with Jupyter Notebook, the following [tutorial](https://www.datacamp.com/community/tutorials/tutorial-jupyter-notebook) created by DataCamp may help.

## Example Results
### Send MRN_STORY request to Real-Time Advanced Distribution Server
```
SENT:
{
  "Domain":"NewsTextAnalytics",
  "ID":2,
  "Key":{
    "Name":"MRN_STORY"
  }
}
RECEIVED: 
[
  {
    "Domain":"NewsTextAnalytics",
    "Fields":{
      "ACTIV_DATE":"2019-07-20",
      "CONTEXT_ID":3752,
      "DDS_DSO_ID":4232,
      "FRAGMENT":null,
      "FRAG_NUM":1,
      "GUID":null,
      "MRN_SRC":"HK1_PRD_A",
      "MRN_TYPE":"STORY",
      "MRN_V_MAJ":"2",
      "MRN_V_MIN":"10",
      "PROD_PERM":10001,
      "RDN_EXCHD2":"MRN",
      "RECORDTYPE":30,
      "SPS_SP_RIC":".[SPSML2L1",
      "TIMACT_MS":37708132,
      "TOT_SIZE":0
    },
    "ID":2,
    "Key":{
      "Name":"MRN_STORY",
      "Service":"ELEKTRON_DD"
    },
    "PermData":"AwhCEAAc",
    "Qos":{
      "Rate":"TickByTick",
      "Timeliness":"Realtime"
    },
    "SeqNumber":32240,
    "State":{
      "Data":"Ok",
      "Stream":"Open",
      "Text":"All is well"
    },
    "Type":"Refresh"
  }
]
```

### Receive Update message and assemble News 
```
RECEIVED: 
[
  {
    "DoNotCache":true,
    "DoNotConflate":true,
    "DoNotRipple":true,
    "Domain":"NewsTextAnalytics",
    "Fields":{
      "ACTIV_DATE":"2019-07-25",
      "FRAGMENT":"H4sIAAAAAAAC/71UW2/bNhR+36840MNe5sq2HCcx0W5QbTfW4miO5bRp2mCgRcpmLZEaSVnJhv33HUpyUKAvfSgGCfwOb+c7PLd/PJrbiHnEkxGrx+xyUTOv59GKCS5TbjzyyYtXZBqvQ1xGKZrdx95jz9sq9oy3lrRUmkpYUC0QqKUGYpFTAeHBiiOFt1wbsQeGm9eqKJURRsBKaasylQsF8Fl+liFjwgolaY4aLE33BZfWQIqXthwyVUmGQq5q4k7vrS1Jv1+yzNe8skjgp6pwc8lrc0KfmvI38eZsNBhP0tE2G/DRxfhs+HP1ptKSuBNFTr66T4LBcDK4CMbEeeLc3mbLD47t/2KcTB4Wye7E6L6ZqrY5hzQX6QGUBLvncLdeAt2qIwer4Ch43axSbUWac3+Vc2o4SGVxf08tCIlkklugKQbTgDCg+V+V0Jz5EGXwrCrgTyXXTbSh1AoZi+60kLtG+0lJD8pWf6qkqXLrbmvAjVrpA1BWCCmM1dQqDfhbnu6lSDGmpioxS6x705JabiwcXVbgk1QGIVMY4zBFajRYc8q4bu1EF2EeMM5e3rqavYNM5Nz4ABu0LP9Gm7O309Flj9paKiQqybQqoItkXdc+dcRtHLViVWpNn7ZW9J0GY3kZ+Htb5M7ud/igNXd8DGKMZA+66ExP0VGSdwY0HmLckE+uWB7BAY5hEoXNBAdXTwjJxm3MbzcfEZbNzv0qufJ/v36Er1j5kUoLKy0wRJHMlC6oq5bvsIG87vT9Cli+mdDGTvFx+AqsXZd+rwYXr4LxZjggowkZXfrn5+cPeHKPDsjRaz+kwlGfcHwvDebPNu2DKLhYTy7vlju1+uW5vzCjjTpmwTWbpA+35vj28JT9fThzt6Wx1HWjPzLsR9h8cip3Fd0584RrVwVmK043zyUuBTgXRTfxLH+y/RLNlXgOA30UmBu4Hieulbm1aptYaitsdR7SWFIZik7FHVNtv3BMC9cDQzLElatuDMYtzK8bnEYNtJMExxsyvJ01OApaWDQQvm8gvmvAOXpNugC57hqQMJmH8UmMwpN0vwqnrTy9WcUfW3F+s77qJEygVopmLS5Pws0Ga6YV4/mHdSut591FZHthwVx0fd3SA0+wRbh+4JFhz6v0DmXs9aOe1xXad6TQvz/9B7krulxaBgAA",
      "FRAG_NUM":1,
      "GUID":"Idw5d8Hwd_1907252I27R98ULgoP+y/Hs3Tovf2Kd9cZQsvBkxfzk4",
      "MRN_SRC":"HK1_PRD_A",
      "MRN_TYPE":"STORY",
      "MRN_V_MAJ":"2",
      "MRN_V_MIN":"10",
      "TIMACT_MS":38378746,
      "TOT_SIZE":882
    },
    "ID":2,
    "Key":{
      "Name":"MRN_STORY",
      "Service":"ELEKTRON_DD"
    },
    "PermData":"AwhCEBkrEiFLEkMM",
    "SeqNumber":32350,
    "Type":"Update",
    "UpdateType":"Unspecified"
  }
]
FRAGMENT length = 882
decompress News FRAGMENT(s) for GUID  Idw5d8Hwd_1907252I27R98ULgoP+y/Hs3Tovf2Kd9cZQsvBkxfzk4
News = {'altId': 'nIdw5d8Hwd', 'audiences': ['NP:CNRA', 'NP:IDXN'], 'body': 'Laporan Harian atas Nilai Aktiva Bersih dan Komposisi Portofolio  \n\nAdditional attachments can be found below:\n\nhttp://pdf.reuters.com/pdfnews/pdfnews.asp?i=43059c3bf0e37541&u=urn:newsml:reuters.com:20190725:nIdw6tQfLW\n\n\n\nhttp://pdf.reuters.com/pdfnews/pdfnews.asp?i=43059c3bf0e37541&u=urn:newsml:reuters.com:20190725:nIdw99ZHSg\n\n\n\n\n\nDouble click on the URL above to view the article.Please note that internet access is required. If you experience problem accessing the internet, please consult your network administrator or technical support\n\nLatest version of Adobe Acrobat reader is recommended to view PDF files.  The latest version of the reader can be obtained from http://www.adobe.com/products/acrobat/readstep2.html\n\nFor Related News, Double Click on one of these codes:[IDXN] [IDX] [ASIA] [ID] [CNRA] [STX] [EQTY] [LID] [XPSG.JK] \n\nFor Relevant Price Information, Double Click on one of these code:<XPSG.JK> ', 'firstCreated': '2019-07-25T10:39:38.666Z', 'headline': 'Laporan Harian atas Nilai Aktiva Bersih dan Komposisi Portofolio  ', 'id': 'Idw5d8Hwd_1907252I27R98ULgoP+y/Hs3Tovf2Kd9cZQsvBkxfzk4', 'instancesOf': [], 'language': 'id', 'messageType': 2, 'mimeType': 'text/plain', 'provider': 'NS:IDX', 'pubStatus': 'stat:usable', 'subjects': ['A:1', 'G:1', 'G:25', 'G:2EK', 'G:CI', 'G:K', 'G:S', 'M:1QD', 'M:32', 'M:3H', 'M:AV', 'M:NU', 'M:Z', 'R:XPSG.JK', 'N2:ASEAN', 'N2:ASIA', 'N2:ASXPAC', 'N2:CMPNY', 'N2:EMRG', 'N2:EQTY', 'N2:ID', 'N2:LID', 'N2:MTPDF', 'N2:NEWR', 'N2:REG', 'N2:SEASIA', 'N2:STX'], 'takeSequence': 1, 'urgency': 3, 'versionCreated': '2019-07-25T10:39:38.666Z'}
```

## References

For further details, please check out the following resources:
* [Refinitiv Real-Time & Distribution Family page](https://developers.refinitiv.com/en/use-cases-catalog/refinitiv-real-time) on the [Refinitiv Developer Community](https://developers.refinitiv.com/) web site.
* [WebSocket API page](https://developers.refinitiv.com/en/api-catalog/refinitiv-real-time-opnsrc/refinitiv-websocket-api).
* [Developer Webinar Recording: Introduction to WebSocket API](https://www.youtube.com/watch?v=CDKWMsIQfaw).
* [Introduction to Machine Readable News with WebSocket API](https://developers.refinitiv.com/en/article-catalog/article/introduction-machine-readable-news-elektron-websocket-api-refinitiv).
* [Machine Readable News (MRN) & N2_UBMS Comparison and Migration Guide](https://developers.refinitiv.com/en/article-catalog/article/machine-readable-news-mrn-n2_ubms-comparison-and-migration-guide).
* [Introduction to Machine Readable News (MRN) with Enterprise Message API (EMA)](https://developers.refinitiv.com/en/article-catalog/article/introduction-machine-readable-news-mrn-elektron-message-api-ema).
* [MRN Data Models and Real-Time SDK Implementation Guide](https://developers.refinitiv.com/en/api-catalog/elektron/elektron-sdk-java/documentation#mrn-data-models-implementation-guide).
* [MRN WebSocket JavaScript example on GitHub](https://github.com/Refinitiv-API-Samples/Example.WebSocketAPI.Javascript.NewsMonitor).
* [MRN WebSocket C# NewsViewer example on GitHub](https://github.com/Refinitiv-API-Samples/Example.WebSocketAPI.CSharp.MRNWebSocketViewer).

For any question related to this example or WebSocket API, please use the Developer Community [Q&A Forum](https://community.developers.refinitiv.com/spaces/152/websocket-api.html).