# COVID19 NYC - ETL Pipeline - Approach 1

## Problem Statement
Design a workflow that would run daily at 9:00 AM that performs ETL on COVID-19 tests occuring in New York state. 

## Approach:
I have acheived this by leveraging multithreading and scheduling concepts in Python. I have written two python scripts in which one of them performs the ETL operations such as extracting the county wise data from the API and loading it into database where as another script that triggers this process automatically to run daily at 9:00 AM. 

I have also used Airflow on docker to automate the scheduling of this ETLPipeline (Almost completed but couldn't get the job to run multiple time, the job runs for once sucessfully. So, this process is a little bit incomplete as I am new to Airflow, so this process took a little longer time in trobleshooting and solving the errors)

###### Environment Used:
Python (Pandas, Multithreading, requests, sqlalchemy), SQLlite database, Airflow, Docker

## Steps to run this application:
###### Assuming that your system have python 3.7 and git configured and installed. 
Required packages to be installed with pip before running this code. If it gives you any command not found error then try replacing pip with pip3 and python with python3.

           pip install sqlalchemy
           pip install pandas
           pip install schedule
           pip install requests
           pip install datetime
           
### Step 1: 
Open CLI depending on your OS and Clone the repository "DataEngineeringAssignment". Or copy paste the below link in the command line prompt.

           https://github.com/AshikaDevaguptapu/DataEngineeringAssignment.git

### Step 2: 
First you should make sure you have the below libraries in your system installed or clone this git respository and run requirements.txt file to install all the dependencies. Then run ""CovidETLPipeline.py" file using the command below
           
            python CovidETLPipeline.py
                                 
Wait for the success message and then run Schedulefile.py using python or follow the below command:
                      
             python schedulefile.py
                                                 
If nothing appears on the screen while running this file, please hold on for atleast 1 minute and then check the prompt on command prompt displayed. This file schedules your ETLPipeline to run for daily 9:00 AM

Then, to check the testcases run Testcases.py file using the same command 
                      
             python Testcases.py         
      
### Step 3:
If the messages "Success, Your data was loaded or testcases ok", are seen on your command line then the results are as expected. Otherwise you should trace back and follow the process again from step1.

### Urls Used:
As the given data is small, we can get the county names from this url "https://health.data.ny.gov/resource/xdss-u53e.csv". But when there is tera bytes of data then it becomes difficult to open the url and read countynames. So, from this url "https://ballotpedia.org/Counties_in_New_York" I have extracted all the county names into a list and formed a url with each of the county name required for our application. Example url of county: "https://health.data.ny.gov/resource/xdss-u53e.csv?county=Orleans"

## Detailed explaination of the python scripts:
### CovidETLPipeline.py:
This file contains all the ETL Process using multithreading, concurrent reads and concurrent writes to the database. This file gives you an output sucess if all the tables data were inserted into the SQLlite database and fetch was also performed sucessfully from the database.

### Custom functions used for ETL Process in this file: 
#### currentJobScheduling: 
This fucntion runs concurrent jobs for each county and runs the ETL Pipeline parallely. Here, I have initiated thread pool executor from concurrent.features and a map which takes for every countyname in list of counties, executes a pipeline. Thus, on a high level it seems like the function runs once but in depth it executes a number of threads concurrently in a thread pool. 
#### Pipeline:
This function creates a pipeline with the three steps Extract,Transform,Load. It takes a countyname as an argument for which it should download the data. In this function, if sucessfull ETL is performed, then a success message is returned from ETL functions. Else, an execption string is thrown. 
#### Extract:
This function takes countyname as a parameter and hits the respective csv file of the county with the formed county url and gets the data into a pandas dataframe. 
#### Transform:
I have analyzed the data and checked for null values, categorical variables, and irrevelant information by leveraging python pandas. I have also checked whether I can scale or normalize any of the attributes or not. But, since the data is in count of people, it doesnt make much sense in scaling or normalizing the attributes. Remaining this seemed perfect for me in the data and all the results of nan,null values etc are clean. But, I have seen that we need to add load date as a final column into the sql table. So, this function takes an input parameter rawdata i.e., the dataframe which is the output of etract step and transforms this dataframe to have load date column. 
#### Load:
This function takes the input parameter from transform step which is the database and converts it into a SQLLite database table.It loads the countywise data into a SQLlite table concurrently. Table name is same as the countyname but without spaces. 

After loading the data as a check, I have performed sql queries on the data I have inserted and the results were as expected.
#### getDataByCounty:
This is a function to check whether the data was sucessfully ingested and the results were as exepected. So, I have performed select * from countyname to fetch the data with the connection object. It returns success if the fetch from respective county is successfull otherwise it returns an execption.

### SchedulerFile.py
This script will schedule your CovidETLPipeline.py to run automatically at 9:00 AM every day. 

##### Note: If you wanted to check for every one minute instead of every day 9:00 AM, please uncomment and comment some lines of the code as instructed inside this code. 

### TestCases.py:

### Modules and packages that were used in the code:
#### pandas: 
It is a library in python mostly used for data analysis and manipulation. I have used pandas package to read the csv files.
#### requests: 
It is a library used for HTTP requests in python. I have used requests to hit the given data url. 
#### sqlalchemy: 
It is a toolkit for leveraging efficient sql with ORM (Object Relational Mapping), can handle conections and pooling with a user understandle python code. I have imported create engine from this library to create a connection to sqlite database and keep the connection globally through out the application.
#### concurrent.features:
It is a module in python which provides an interface for our asychronously callable threads. This asynchronous execution can be done by importing ProcessPoolExcetor, Threads and Process Pool Executor. From current.features I have imported ProcessPoolExecutor which executes each of our workers in seperate threads in the main process.
#### schedule: 
It is a library in python which is used to schedule a particular task. I have used schedule library to schedule my code to run daily at 9:00 AM. I have also used run_pending method to run all the scheduled jobs that are pending for run. 
#### time: 
I have imported sleep from time module of python to stop my execution for 1 second. 

Apart from these, I have followed OOPS design in python in my code. 
                                  
## Summary:
In this application, I have performed ETL to concurrently extract and load county wise data of COVID-19 tests in New York into SqlLite database. I have scheduled job to perform this task daily at 9:00 AM. 

# Building COVID-19 ETLPipeline with AirFlow on Docker - Approach 2
Airflow is a scheduling tool to ochestrate the ETL process.  
I have tried to use Airflow to schedule my python code to run daily at 9:00 AM. I am halfway through, I was able to trigger my python code once but not for every day. 
I wasn't able to figure out why my scheduler is not able to run daily, where as it run perfectly if i schedule it for once. 
I am new to airflow so, It took some time for me to do this process.

Here are the steps I have followed to make automte the ETL of given problem :

### Step1: Install docker and docker compose
First we should install docker on our system. I am running Mac Os, so I have downloaded docker desktop which comes along with docker compose and docker swarm. 
You can download docker desktop from this url: 
           
                     https://www.docker.com/products/docker-desktop

### Step2: Pull the puckel image for airflow 
We have the puckel docker image which can be run airflow on port 8080 in our local system. You can find the image in docker hub or run this command in CLI
                    
                    docker pull puckel/docker-airflow
                    
You can specify the tag as latest here or make sure that you have downloaded the latest image from docker hub. 
We also have a yml file which can initiate postgres sql, redis in memory DB, and workers, webservers needed
for our application. The networking of these docker containers will be taken care by docker compose. 
If you want to execute python files or use operators in your dags then you should take celeryexecutordockercompose.yaml file. If you opt for sequential executor, you cannot run the tasks in your dag file. The yaml file can be downloaded from the bwlow repository. 
                      
                      https://github.com/puckel/docker-airflow

### Step3: Run the Puckel image with celery executor
After downloading the image and yaml file, you should make a container out of the image downloaded. Run this command on CLI 

                      docker-compose -f docker-compose-CeleryExecutor.yml up -d
                      
Make sure that you have the celeryexecutor.yml file in the same forlder you run this command. Otherwise, specify the full path to celeryexecutor file. 
-d specifies that this container will run in the background as a deamon. 

### Step4: Check airflow on "localhost:8080"
After running the image, you can check below command whether airflow was initiated or not. 
                      
                      docker container logs <containername/id>

If you see airflow image in the container logs, it specifies that airflow is successful on your machine. Now, hit the url localhost:8080. 
If airflow UI pops up, then you have sucessfully installed airflow on your system. 

### Step5: Check dags folder inside the container
Now go inside the container with the command the below command.
                      
                      docker exec -it <containername/id> /bin/bash
Cd into the directory dags in /usr/local/airflow/dags
The detault $AIRFLOW_HOME is /usr/local/airflow. Here we need to write our dag py file to schedule our python code to run. 

### Step6: Write a dag file inside dags folder
We need to write a dag file here. I have written the dag file to call my python script and I have scheduled it to run for 1 minute as sample. You can find my dag file here "" and my python file here "" in this repository.

### Step7: Run the tasks inside the dag in Airflow UI
Now go the url localhost:8080 and check whether you dag was reflected in the airflow UI. If reflected, open the dag and go to Tree view and click on the individual 
task to run your individual tasks. You can also triger the total dag at once. 

Now you can go to details to check whether your dag is successfully executed or not. 
If it gives Success then you can go the logs to check your output. If failed, then you can also check the logs and see where you have gone wrong. 

I was struck after this step, because my dag doesn't run my task after 1 iteration. So, I couldn't move further. I have changed the schedule_interval parameter 
to run for 1 minute as a check, but no luck. It didn't work where as it works for schedule_interval="@once". Thus, changing the schedule_interval to "0 9 0 0 0" would satisfy the requirement. 

#### Some errors and Fixes which I have come across:
1) I have faced an error while executing my py file form dag. It said file not found every time I run my dag even though the file is found in the same path as
mentioned in the dag. So, it took me a lot time to debug this and finally found that, airflow is checking for files present in the docker container of airflow not on our system. So, we should make sure that we login in to our webserver container and go inside /usr/local/airflow/dags and create our dag file and python file there and specify this path in our dag file. Otherwise, our system path is unknown to the container. So, our dag will not pick python file path. You can also mount your path to the container path or write a custom docker file to cpy your files on to container. But, you may face errors and missing of files when the containers are in running state. So, better option is to write these dag and python file by using vi inside the container in dags folder.

2) You need to install all the dependencies inside your container like pip install pandas etc. You can also write a requirements.txt but, as my imports are less, 
I have gone inside the container and installed all using pip. 

You can find the results of my ETL python code in Airflow in the images attached. 
