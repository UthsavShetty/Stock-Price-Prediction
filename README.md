# Predicting stock price on real-time data

This project focuses on predicting Google stock price on real time data. I used past 10 years worth of historical Google (GOOGL) stock data for training and built an effective model for predicting stock prices and displayed the predictions on webpage using Flask, Kafka and Highcharts.


## Prerequisites:

- Python3
```
$ sudo apt-get install python3
$ sudo apt-get install python3-pip
```
- [JDK 8 ](https://jdk.java.net/) - [Read this to install Java](https://github.com/Wolvarun9295/InstallationDocuments/blob/master/JAVA.txt)
- [Apache Zookeeper](https://zookeeper.apache.org/releases.html) - [Read this to install Zookeeper](https://github.com/Wolvarun9295/InstallationDocuments/blob/master/ZOOKEEPER.txt)
- [Apache Kafka](https://kafka.apache.org/downloads) – [Read this to run Kafka](https://kafka.apache.org/quickstart)
- [Amazon AWS Account](https://aws.amazon.com/)
- [Google Historical Data](https://in.finance.yahoo.com/quote/GOOGL?p=GOOGL&.tsrc=fin-srch), download CSV file selecting the range
- [Requirements.txt](Requirements.txt)

Use the following command to install the .txt file:

```
$ pip3 install -r Requirements.txt
```

## Instructions
- This project has been built using Python3 to help predict the future stock close prices of Google stock with the help of Machine Learning and Data Visualization in real time.
- To start, I created an AWS account and created a user with all access.
- Downloaded the Amazon CLI on my system and then added the AWS access keys to be accessed globally.
- Next I started creating python script to create a bucket and upload the downloaded CSV file onto the AWS bucket. To do this, I needed to install the boto3

- After creating and uploading my CSV file, I fetched the file from my S3 bucket with the help of Pandas.
- Since no data is clean and has missing values, it needs to be cleaned.
- Now after the data has been cleaned, we can now built a model using Machine Learning. Keep in mind, the less data we use the higher chances of underfitting occur and the more data we use, the higher chances of overfitting occur. So we need to choose the data not more, not less.
- The model building process has been done using PySpark’s mlib Library.
- I used Linear Regression to train the model and used the Regression Evaluator to give the accuracy of my model.
- After the successfull buliding of my model, I needed to check if it works on real data. For that I registered on a website called AlphaVantage and generated the key to access the live data from their site.

#### ***What is AlphaVantage?***
***Alpha Vantage Inc. is a company that provides realtime and historical stock APIs as well as forex (FX) and digital/crypto currency data feeds.***

- For this, firstly I had to install Apache Zookeeper and Apache Kafka.

- To display the prediction in real time, we first need to start the Zookeeper server and then start the Kafka server.
- I created the Producer and Consumer scripts in Python3 and ran them through Flask app.
- Finally, to display the graph I used Highcharts JS in my HTML file and styled it through CSS.


## Setup to run the project

### Step 1:
- Create an AWS account
- Now go to **IAM** in **Identity and Access Management** services and setup a user with programming access and give full access to the user.
- After the setup, note down the **Public Access Key** and **Secret Access Key**. **(Highly Important)**

### Step 2:
- Install the Amazon CLI (Command Line Interface) on your local machine. (requires curl)
- On your local machine, make a folder **.aws** in root folder and touch two files in the folder: **config** and **credentials**. (to make the aws keys globally accessible)
- Add the following lines in **config**:

### Step 3:
In this step we will run all the python scripts in the following order. Note that you can make changes wherever necessary according to your settings.
- Run the **createBucket.py** file. This will create the bucket in the AWS S3.
- Run the **uploadFile.py** file. This will upload the CSV file in your bucket.
- Run the **fetchFile.py** file. This will fetch the CSV file from your bucket.
- Run the **dataCleaner.py** file. This is clean the data in CSV file.
- Run the **modelBuilder.py** file. This will build the model based on the data and save the model. It will be saved in the project folder named **GoogleStockModel**.


### Step 4:
- After building the model, on terminal start the Zookeeper and Kafka servers.
```
$ cd zookeeper
$ C:\Kafka\bin\windows\zookeeper-server-start.bat C:\Kafka\config\zookeeper.properties


$ cd kafka
$ C:\Kafka\bin\windows\kafka-server-start.bat C:\Kafka\config\server.properties
```
- Create a **key.txt** file in the project and add your AlphaVantage key in it so that the **Producer.py** file can access it.
- Now run the **Producer.py** file. This will start the Producer that will serve the messages by creating the topic called **GoogleStock**.
- Now run **app.py** file. This will run the **Consumer.py** file and start receiving the messages published by **Producer.py**.
- The **app.py** is the flask application which calls the HTML template in the templates folder which uses the CSS and JS files in the static folder.
- Open the browser to see the graph displayed on **127.0.0.1:5000** shown in the below screeenshot.
<img src=Screenshots/graph.gif height=”100”>


#

## Deployment Process
- First create an EC2 instance, download the **filename.pem** file which is available at the time of instance creation.
- Now before logging in to the instance, make sure you assign an elastic IP to your instance. 
- Install Java. Refer **[this](#Prerequisites)**
- Next clone your project using git on the instance.
- For running the flask app on AWS, we need two additional packages: **nginx** and **gunicorn3** (since project is running on Python3)


#### ***What is NGINX?***
***NGINX is open source software for web serving, reverse proxying, caching, load balancing, media streaming, and more. It started out as a web server designed for maximum performance and stability. In addition to its HTTP server capabilities, NGINX can also function as a proxy server for email (IMAP, POP3, and SMTP) and a reverse proxy and load balancer for HTTP, TCP, and UDP servers.***

#### ***What is Gunicorn?***
***Gunicorn ‘Green Unicorn’ is a Python WSGI HTTP Server for UNIX. It’s a pre-fork worker model ported from Ruby’s Unicorn project. The Gunicorn server is broadly compatible with various web frameworks, simply implemented, light on server resource usage, and fairly speedy.***

- Now go in the **sites-enabled** folder inside nginx and do the following:

- Inside the flaskapp file add the following:
```
server{
	listen : 80;
	server_name : your.elastic.IP;
	location / {
		proxy_pass http://127.0.0.1:8000;
	}
}
```
- Save the above file and restart the nginx service.

- Follow the videos from **[1–6](https://www.youtube.com/playlist?list=PL5KTLzN85O4KTCYzsWZPTP0BfRj6I_yUP)** for reference.
- Finally start your Zookeeper and Kafka servers on the current terminal as usual.
- Now open two additional terminals and login to the same instance from them and run the **Producer.py** and **app.py** files on each terminals as follows:
```
$ python3 Producer.py
$ gunicorn3 app:app
```
- Now after successfull execution of the flask app, enter **your.elastic.IP** (No port number necessary) in your browser and voila, the flask app is up and running.


## Job Scheduling on AWS
- To perform job scheduling on AWS, we need to make four bash executable files: **zookeeper.sh, kafka.sh, ProducerJob.sh and AppJob.sh** and store them in the project directory.

- Save the above four files and make them executable using the following commands:
```
$ chmod u+x zookeeper.sh
$ chmod u+x kafka.sh
$ chmod u+x ProducerJob.sh
$ chmod u+x AppJob.sh
```
- Now check if there are any cron jobs running:
```
$ crontab -l
```
#### ***What is Cron and Crontab?***
***Cron is a scheduling daemon that executes tasks at specified intervals. These tasks are called cron jobs and are mostly used to automate system maintenance or administration.
For example, you could set a cron job to automate repetitive tasks such as backing up databases or data, updating the system with the latest security patches, checking the disk space usage, sending emails, and so on.
The cron jobs can be scheduled to run by a minute, hour, day of the month, month, day of the week, or any combination of these.***
```
* * * * * command(s)
- - - - -
| | | | |
| | | | ----- Day of week (0 - 7) (Sunday=0 or 7)
| | | ------- Month (1 - 12)
| | --------- Day of month (1 - 31)
| ----------- Hour (0 - 23)
------------- Minute (0 - 59)
```
- Now edit the crontab file and add the following:
```
$ crontab -e

# Select nano option

# Add the following in the file (Ex: I want to start my project everyday at 10:30 AM):
30 10 * * * /bin/sh /home/ubuntu/StockMarketPredictionProject/zookeeper.sh
35 10 * * * /bin/sh /home/ubuntu/StockMarketPredictionProject/kafka.sh
40 10 * * * /bin/sh /home/ubuntu/StockMarketPredictionProject/ProducerJob.sh
45 10 * * * /bin/sh /home/ubuntu/StockMarketPredictionProject/AppJob.sh
```
- Save and exit.
- Since we have given 5 minutes interval between all the four files, we need to wait 20 minutes.
- To check if all cron jobs are running as intended, do the following:
```
$ ps -ef | grep filename.sh

# or

$ ps -ef | grep taskname
```
- Now check the output on the browser using **your.elastic.IP**.




