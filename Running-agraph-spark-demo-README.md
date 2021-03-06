Running the Allegrograph Spark Demo via AMI
===================================

The Amazon AMI preloaded with all services is:
* Name: Allegrograph Spark Demo
* AMI Name: agraph-spark-demo-ami
* AMI ID: ami-00b2e068

The original instance was type m3.xlarge with 200G EBS Volume.

Configure the security group with the following ports open:
* 8080, 7077, 10035, 8090

For the web views, use the public IP address, substituting for 127.0.0.1 below.

Some views may redirect to 127.0.0.1 as per the configuration, especially Spark.

Please see the Spark Documentation for further customizing its configuration.

<http://spark.apache.org/>
 

Once an instance is running from this AMI, log in with user ec2-user via ssh.

* Start Allegrograph
* Start Spark Master
* Start Spark Worker
* Start Spark Job Server
* Run ML Training Script
* Run ML Classification Script


All components are installed in ec2-user home directory:
```
/home/ec2-user
```
1. Spark Master and Worker

Spark installation directory
```
/home/ec2-user/spark-1.2.0-bin-hadoop2.4
```
Running master:
```
cd /home/ec2-user/spark-1.2.0-bin-hadoop2.4
./run-master.sh
```
Stopping master:
```
cd /home/ec2-user/spark-1.2.0-bin-hadoop2.4
./stop-master.sh
```

Spark master URL: spark://127.0.0.1:7077

Spark master webview: http://127.0.0.1:8080


Running slave:
```
cd /home/ec2-user/spark-1.2.0-bin-hadoop2.4
./run-slave.sh
```
Stopping slave:
```
cd /home/ec2-user/spark-1.2.0-bin-hadoop2.4
./stop-slave.sh
```
Spark worker webview: http://127.0.0.1:8081


2. Allegrograph 4.14.1

AGraph installation directory
```
/home/ec2-user/ag4.14.1
```

Running AG server:
```
cd /home/ec2-user/ag4.14.1
./bin/agraph-control --config lib/agraph.cfg start
```
Stopping AG server:
```
cd /home/ec2-user/ag4.14.1
bin/agraph-control --config lib/agraph.cfg stop
```
AGraph server endpoint URL + webview: http://127.0.0.1:10035


The AGraph is already configured and 20news quads:
```
https://github.com/vital-ai/agraph-spark-demo/blob/master/data/20news-all.nq.gz
```
are loaded into:
```
        username: super
        password: super
        Catalog: / (ROOT)
        Repository: mldemo
        Subjects count: 18,846
        Statements count: 75,384 ( 4 per subject )
```

3. Spark-JobServer

Spark-JobServer installation directory:
```
/home/ec2-user/job-server
```
Running job server:
```
cd /home/ec2-user/job-server
nohup ./server_start.sh &
```
Stopping job server:
```
cd /home/ec2-user/job-server
./server_stop.sh
```

Job Server REST endpoint + webview: http://127.0.0.1:8090


4. ML training and classification scripts

agraph-spark-demo project directory:
```
/home/ec2-user/agraph-spark-demo-master
```

The project is built with maven and scripts already configured with agraph and jobserver details
Scripts directory:
```
/home/ec2-user/agraph-spark-demo-master/scripts
```

The jobs jar is already uploaded into job server (command) - mldemo app:
```
cd /home/ec2-user/agraph-spark-demo-master/
curl --data-binary @target/agraph-spark-demo-0.0.1-SNAPSHOT.jar  127.0.0.1:8090/jars/mldemo
```

Creating job context (ctx1):
```
curl -X POST 127.0.0.1:8090/contexts/ctx1
```

Running training script

Outputs the trained naive-bayes model into /home/ec2-user/models directory (as per script inline config)
```
cd /home/ec2-user/agraph-spark-demo-master/scripts
./MLDemoTraining mldemo ctx1
```

Running classification script

Inserts categorization + inference output quads into agraph repository
```
cd /home/ec2-user/agraph-spark-demo-master/scripts
./MLDemoClassification mldemo ctx1
```

After successful run new quads will appear in agraph

They may be deleted via either 
* webview "Delete Statements" option, all quads with predicate <http://vital.ai/ontology/twentynews#hasCategory>
* webview "View Statements", executing the sparql update query:
```
DELETE where { ?s <http://vital.ai/ontology/twentynews#hasCategory> ?o }
```




