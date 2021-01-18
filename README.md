# TML: Prediction and Optimization
Pre-requisites:
1) MAADS-VIPER
2) MAADS-HPDE
3) Python
4) Python libraries
5) Kafka cloud account (use Confluent cloud)
6) Beginner knowledge of Python, VIPER, HPDEm Kafka

```python
# Developed by: OTICS Advanced Analytics Inc.
# Date: 2021-01-18 
# Toronto, Ontario Canada
# For help email: support@otics.ca 

# Import the core libraries
import maads
import nest_asyncio
import json

nest_asyncio.apply()

```


```python
# Set Global variables for VIPER and HPDE - You can change IP and Port for your setup of 
# VIPER and HPDE
VIPERHOST="http://192.168.0.13"
VIPERPORT=8000
hpdehost="http://192.168.0.13"
hpdeport=8001

# Set Global variable for Viper confifuration file - change the folder path for your computer
viperconfigfile="C:/MAADS/Golang/go/bin/viper.env"

#############################################################################################################
#                                      STORE VIPER TOKEN
# Get the VIPERTOKEN from the file admin.tok - change folder location to admin.tok
# to your location of admin.tok
def getparams():
        
     with open("c:/maads/golang/go/bin/admin.tok", "r") as f:
        VIPERTOKEN=f.read()
  
     return VIPERTOKEN

VIPERTOKEN=getparams()

#############################################################################################################
#                                     JOIN DATA STREAMS 

# Set personal data
companyname="OTICS Advanced Analytics"
myname="Sebastian"
myemail="Sebastian.Maurice"
mylocation="Toronto"

# Joined topic name
joinedtopic="joined-viper-test20"
# Replication factor for Kafka redundancy
replication=3
# Number of partitions for joined topic
numpartitions=3
# Enable SSL/TLS communication with Kafka
enabletls=1
# If brokerhost is empty then this function will use the brokerhost address in your
# VIPER.ENV in the field 'KAFKA_CONNECT_BOOTSTRAP_SERVERS'
brokerhost=''
# If this is -999 then this function uses the port address for Kafka in VIPER.ENV in the
# field 'KAFKA_CONNECT_BOOTSTRAP_SERVERS'
brokerport=-999
# If you are using a reverse proxy to reach VIPER then you can put it here - otherwise if
# empty then no reverse proxy is being used
microserviceid=''

description="Topic containing joined streams for Machine Learning training dataset"

streamstojoin="viperdependentvariable,viperindependentvariable1,viperindependentvariable2"
# Call MAADS python function to create joined stream topic
result=maads.vipercreatejointopicstreams(VIPERTOKEN,VIPERHOST,VIPERPORT,joinedtopic,
                    streamstojoin,companyname,myname,myemail,description,mylocation,
                    enabletls,brokerhost,brokerport,replication,numpartitions,microserviceid)

print(result)

#Load the JSON object and get producerid
y = json.loads(result)
producerid=y['ProducerId']

# Subscribe consumer to the topic just created with some information about yourself
# If subscribing to a group and add group id here
groupid=''
description="Topic contains joined data streams for transactional machine learning"
result=maads.vipersubscribeconsumer(VIPERTOKEN,VIPERHOST,VIPERPORT,joinedtopic,companyname,
                                    myname,myemail,mylocation,description,
                                    brokerhost,brokerport,groupid,microserviceid)
print(result)
# Load the JSON object and extract the consumer id
y = json.loads(result)
consumerid=y['Consumerid']
print(consumerid)


```

    {"WARN":"Topic already exists","ProducerId":"ProducerId-w12uV0QiPx8nAegnvIW9A6UYBxBxJP","Topic":"joined-viper-test20"}
    {"Consumerid":"ConsumerId-dcmn1YGcfexRBJyIFoi9sceThEqtO0","Topic":"joined-viper-test20"}
    ConsumerId-dcmn1YGcfexRBJyIFoi9sceThEqtO0
    


```python
#############################################################################################################
#                                    PRODUCE TO TOPIC STREAM

# Roll back each data stream by 50 offsets - change this to a larger number if you want more data
# For supervised machine learning you need a minimum of 30 data points in each stream
rollbackoffsets=50
# Go to the last offset of each stream: If lastoffset=500, then this function will rollback the 
# streams to offset=500-50=450
startingoffset=-1
# Max wait time for Kafka to response on milliseconds - you can increase this number if
# Kafka takes longer to response.  Here we tell the functiont o wait 10 seconds
delay=10000

# Call the Python function to produce data from all the streams
result2=maads.viperproducetotopicstream(VIPERTOKEN,VIPERHOST,VIPERPORT,joinedtopic,producerid,
                                        startingoffset,rollbackoffsets,enabletls,
                                        delay,brokerhost,brokerport,microserviceid)
#print(result2)
y = json.loads(result2)
#get the partition
for elements in y:
  try:
    if 'Partition' in elements:
       partition=elements['Partition'] 
  except Exception as e:
    continue
    
```


```python
#############################################################################################################
#                           CREATE TOPIC TO SAVE TRAINING DATA SET FROM STREAM

# Name the topic
producetotopic="trainingdata2"
description="Topic containing the training dataset for TML"

result3=maads.vipercreatetopic(VIPERTOKEN,VIPERHOST,VIPERPORT,producetotopic,companyname,
                               myname,myemail,mylocation,description,enabletls,
                               brokerhost,brokerport,numpartitions,replication,microserviceid)
# Load the JSON and get the producer id 
y = json.loads(result3)
producetotopic=y['Topic']
producerid=y['ProducerId']

#############################################################################################################
#                           CREATE TRAINING DATA SET FROM JOINED STREAM TOPIC

consumefrom=joinedtopic
description="Subscribing to training dataset"
result3=maads.vipersubscribeconsumer(VIPERTOKEN,VIPERHOST,VIPERPORT,consumefrom,companyname,
                                    myname,myemail,mylocation,description,
                                    brokerhost,brokerport,groupid,microserviceid)
#print(result3)
# Load the JSON and extract the consumerid
y = json.loads(result3)
consumerid=y['Consumerid']
# Assign the dependent variable stream
dependentvariable="viperdependentvariable"
# Assign the independentvariable streams
independentvariables="viperindependentvariable1,viperindependentvariable2"
#set the delay in milliseconds - or 60 seconds to wait for Kafka to respond 
# before backing out - for large datasets or slow internet connection you may
# need to adjust this variable
delay=60000
result4=maads.vipercreatetrainingdata(VIPERTOKEN,VIPERHOST,VIPERPORT,consumefrom,producetotopic,
                             dependentvariable,independentvariables, 
                             consumerid,producerid,companyname,partition,
                             enabletls,delay,brokerhost,brokerport,microserviceid)

print(result4)
# Load the JSON object and extract the Kafka partition for the training dataset 
y = json.loads(result4)
partition_training=y['Partition']
print(partition_training)


```


```python
#############################################################################################################
#                         SUBSCRIBE TO TRAINING DATA TOPIC  

producetotopic="trainingdata2"
description="Subscribing to training dataset topic"
result=maads.vipersubscribeconsumer(VIPERTOKEN,VIPERHOST,VIPERPORT,producetotopic,companyname,
                                    myname,myemail,mylocation,description,
                                    brokerhost,brokerport,groupid,microserviceid)
y = json.loads(result)
consumeridtrainingdata2=y['Consumerid']

#############################################################################################################
#                         CREATE TOPIC TO STORE TRAINED PARAMS FROM ALGORITHM  

consumefrom=producetotopic
producetotopic="trainined-params"
description="Topic to store the trained machine learning parameters"
result5=maads.vipercreatetopic(VIPERTOKEN,VIPERHOST,VIPERPORT,producetotopic,companyname,
                               myname,myemail,mylocation,description,enabletls,
                               brokerhost,brokerport,numpartitions,replication,
                               microserviceid='')
print(result5)
# Load JSON data and extract the producer id
y = json.loads(result5)
producetotopic=y['Topic']
producerid=y['ProducerId']

#############################################################################################################
#                         VIPER CALLS HPDE TO PERFORM REAL_TIME MACHINE LEARNING ON TRAINING DATA 

consumefrom="trainingdata2"
producetotopic="trainined-params"
# deploy the algorithm to ./deploy folder - otherwise it will be in ./models folder
deploy=1
# number of models runs to find the best algorithm
modelruns=10
# Go to the last offset of the partition in partition_training variable
offset=-1
# If 0, this is not a logistic model where dependent variable is discreet
islogistic=0
# set network timeout for communication between VIPER and HPDE in seconds
networktimeout=200
result6=maads.viperhpdetraining(VIPERTOKEN,VIPERHOST,VIPERPORT,consumefrom,producetotopic,
                                companyname,consumeridtrainingdata2,producerid, hpdehost,
                                viperconfigfile,enabletls,partition_training,
                                deploy,modelruns,hpdeport,offset,islogistic,
                                brokerhost,brokerport,networktimeout,microserviceid)    
print(result6)
# Load the JSON and extract the producer id and algorithm key if needed
y = json.loads(result6)
algokey=y['Algokey']
hpdetraining_partition=y['Partition']

#return

#############################################################################################################
#                                     SUBSCRIBE TO STREAM TOPIC

producetotopic="trainined-params"
description="Subscribing to trained machine learning parameters"
result=maads.vipersubscribeconsumer(VIPERTOKEN,VIPERHOST,VIPERPORT,producetotopic,companyname,
                                    myname,myemail,mylocation,description,
                                    brokerhost,brokerport,groupid,microserviceid)
# Load the JSON and extract the consumer id
y = json.loads(result)
consumeridtraininedparams=y['Consumerid']
print(consumeridtraininedparams)
consumefrom=producetotopic

#############################################################################################################
#                         CREATE TOPIC TO STORE PREDICTIONS FROM ALGORITHM  

producetotopic="hyper-predictions"
description="Topic to store the predictions"
result=maads.vipercreatetopic(VIPERTOKEN,VIPERHOST,VIPERPORT,producetotopic,companyname,
                              myname,myemail,mylocation,description,enabletls,
                              brokerhost,brokerport,numpartitions,replication,
                              microserviceid)
print(result)
# Load the JSON and extract the producer id
y = json.loads(result)
produceridhyperprediction=y['ProducerId']
print(produceridhyperprediction)

#############################################################################################################
#                                     SUBSCRIBE TO STREAM PREDICTION TOPIC

result=maads.vipersubscribeconsumer(VIPERTOKEN,VIPERHOST,VIPERPORT,producetotopic,companyname,
                                    myname,myemail,mylocation,description,
                                    brokerhost,brokerport,groupid,microserviceid)
print(result)
# Load the JSON and extract the consumer id
y = json.loads(result)
streamconsumerid=y['Consumerid']
consumefrom=producetotopic


#############################################################################################################
#                                     START HYPER-PREDICTIONS FROM ESTIMATED PARAMETERS
# name the topic
producetotopic="hyper-predictions"        
# Use the topic created from function viperproducetotopicstream for new data for 
# independent varibles
inputdata=joinedtopic

consumefrom="trainined-params"
# if you know the algorithm key put it here - this will speed up the prediction
mainalgokey=algokey
# Offset=-1 means go to the last offset of hpdetraining_partition
offset=-1
# wait 60 seconds for Kafka - if exceeded then VIPER will backout
delay=60000
# use the deployed algorithm - must exist in ./deploy folder
usedeploy=1
#Start predicting with new data streams
result6=maads.viperhpdepredict(VIPERTOKEN,VIPERHOST,VIPERPORT,consumefrom,producetotopic,
                               companyname,consumeridtraininedparams,
                               produceridhyperprediction, hpdehost,inputdata,mainalgokey,
                               hpdetraining_partition,offset,enabletls,delay,hpdeport,
                               brokerhost,brokerport,networktimeout,usedeploy,microserviceid)
print(result6)

#############################################################################################################
#                         CREATE TOPIC TO STORE OPTIMAL PARAMETERS FROM ALGORITHM  

producetotopic="hpde-optimal-parameters"
description="Topic for optimization results"
result=maads.vipercreatetopic(VIPERTOKEN,VIPERHOST,VIPERPORT,producetotopic,companyname,
                              myname,myemail,mylocation,description,enabletls,
                              brokerhost,brokerport,numpartitions,replication,
                              microserviceid='')
print(result)
# Load the JSON and extract the producer id
y = json.loads(result)
producerid=y['ProducerId']
#############################################################################################################
#                                     START MATHEMATICAL OPTIMIZATION FROM ALGORITHM
consumefrom="trainined-params"
delay=10000
offset=-1
# we are doing minimization if ismin=1, otherwise we are maximizing the objective function
ismin=1
# choosing constraints='best' will force HPDE to choose the constraints for you
constraints='best'
# We are going to expand the lower and upper bounds on the constraints by 20%
stretchbounds=20
# we are going to use MIN and MAX for the lower and upper bounds on the constraints
constrainttype=1
# We are going to see if there are 'better' optimal values around an epsilon distance (10%)
# from the local optimal values found
epsilon=10
# network timeout in seconds between VIPER and HPDE
timeout=120

# Start the optimization
result7=maads.viperhpdeoptimize(VIPERTOKEN,VIPERHOST,VIPERPORT,consumefrom,producetotopic,
                                companyname,consumeridtraininedparams,
                                producerid,hpdehost,hpdetraining_partition,
                                offset,enabletls,delay,hpdeport,usedeploy,ismin,
                                constraints,stretchbounds,constrainttype,epsilon,
                                brokerhost,brokerport,timeout,microserviceid)
print(result7)

```


```python

```
