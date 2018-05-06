If u want to know the prerequesites of hadoop click on the below link:
<a href="https://github.com/vshivasai/Hadoop-installation.wiki.git">
hadoop prerequesites<br>

### STEP 2: Install Hadoop
***

### 1. Extract Hadoop and modify the permissions
Extract the contents of the Hadoop package to a location of your choice. Say ‘/usr/local/hadoop’. Make sure to change the owner of all the files to the hduser user and hadoop group. But first move the downloaded hadoop-1.2.1.tar.gz to ‘/user/local/’ (image 6). 
1. ` 	sudo mv /home/user_name/Downloads/hadoop-1.2.1.tar.gz /usr/local`
2. ` 	cd /usr/local`
3. ` 	sudo tar xzf hadoop-1.2.1.tar.gz`
4. ` 	sudo mv hadoop-1.2.1 hadoop`
5. ` 	sudo chown –R hduser:hadoop hadoop`<br>

 ![](https://3.bp.blogspot.com/-dxIkVcQ1vJU/Wkm6K11iR_I/AAAAAAAAFpQ/sviw_foxPKMTPVd87beQQyq9Angm_lingCLcBGAs/s1600/Capture%2B6.PNG)

 Image 6. Extracting and Modifying the Permissions.<br>
### 2. Update '$HOME/.bashrc' of hduser

Open the $HOME/.bashrc file of user hduser
1. ` 	sudo gedit /home/hduser/.bashrc`

and add the following lines at the end:<br>
`# Set Hadoop-related environment variables`<br>
`export HADOOP_PREFIX=/usr/local/hadoop`<br>
`# Set JAVA_HOME (we will also configure JAVA_HOME directly for Hadoop later on)`<br>
`export JAVA_HOME=/usr/lib/jvm/java-8-oracle`<br>
`# Some convenient aliases and functions for running Hadoop-related commands`<br>
`unalias fs &> /dev/null`<br>
`alias fs="hadoop fs"`<br>
`unalias hls &> /dev/null`<br>
`alias hls="fs -ls"`<br>
`# If you have LZO compression enabled in your Hadoop cluster and`<br>
`# compress job outputs with LZOP (not covered in this tutorial):`<br>
`# Conveniently inspect an LZOP compressed file from the command`<br>
`# line; run via:`<br>
`#`<br>
`# $ lzohead /hdfs/path/to/lzop/compressed/file.lzo`<br>
`#`<br>
`# Requires installed 'lzop' command.`<br>
`#`<br>
`lzohead () {`<br>
`hadoop fs -cat  | lzop -dc | head -1000 | less`<br>
`}`<br>
`# Add Hadoop bin/ directory to PATH`<br>
`export PATH=$PATH:$HADOOP_PREFIX/bin`<br>


### Step 3: Configuring Hadoop
***


Now we have to configure the directory where Hadoop will store its data files, the network ports it listens to, etc. Our setup will use Hadoop’s Distributed File System, HDFS, even though our little cluster only contains our single local machine.

### 1. Setting up the working directory
We will use the directory ‘/app/hadoop/tmp’ in this tutorial. Hadoop’s default configurations use hadoop.tmp.dir as the base temporary directory both for the local file system and HDFS, so don’t be surprised if you see Hadoop creating the specified directory automatically on HDFS at some later point. Now we create the directory and set the required ownerships and permissions:
1. ` 	sudo mkdir -p /app/hadoop/tmp`
2. `    sudo chown hduser:hadoop /app/hadoop/tmp`
3. `    sudo chmod 750 /app/hadoop/tmp`

If you forget to set the required ownerships and permissions, you will see a java.io.IOException when you try to format the name node in the next section.

### 2. Configuring Hadoop setup files

### I. hadoop-env.sh
The only required environment variable we have to configure for Hadoop is JAVA_HOME. Open conf/hadoop-env.sh (if you used the installation path in this tutorial, the full path is /usr/local/hadoop/conf/hadoop-env.sh) and set the JAVA_HOME environment variable to the java8 directory.
1. ` 	sudo gedit /usr/local/hadoop/conf/hadoop-env.sh`

Replace<br>
 ![](https://3.bp.blogspot.com/-uQ_WyrT3lG8/Wkm6kXKoZNI/AAAAAAAAFpU/BRXRyLvCg20P2fto-N-kTqH5wbwudnSSgCLcBGAs/s1600/copy.png)<br>
 Image 7. Replace this .

With<br>

 `# The java implementation to use. Required.`<br>
 `export JAVA_HOME=/usr/lib/jvm/java-8-oracle`<br>

### II. core-site.xml
Open up the file /usr/local/hadoop/conf/core-site.xml
1. ` 	sudo gedit /usr/local/hadoop/conf/core-site.xml`

and add the following snippet between <configuration>...</configuration> tags (see image 8). You can leave the settings below “as is” with the exception of the hadoop.tmp.dir parameter – this parameter you must change to a directory of your choice.<br>
 `<property>`<br>
 `<name>hadoop.tmp.dir</name>`<br>
 `<value>/app/hadoop/tmp</value>`<br>
 `<description>A base for other temporary directories.</description>`<br>
 `</property>`<br>

 `<property>`<br>
 `<name>fs.default.name</name>`<br>
 `<value>hdfs://localhost:54310</value>`<br>
 `<description>The name of the default file system. A URI whose`<br>
 `scheme and authority determine the FileSystem implementation. The`<br>
 `uri's scheme determines the config property (fs.SCHEME.impl) naming`<br>
 `the FileSystem implementation class. The uri's authority is used to`<br>
 `determine the host, port, etc. for a filesystem.</description>`<br>
 `</property>`<br>

 ![](https://3.bp.blogspot.com/-c195sD2Yr7o/Wkm69FEqiGI/AAAAAAAAFpY/6X-1PDdyBOcviGzEa_zsrS1BkplSVRUzgCLcBGAs/s1600/copy1.png)<br>
Image 8. Add as Shown.

### III. mapred-site.xml
Open up the file /usr/local/hadoop/conf/mapred-site.xml
1. ` 	sudo gedit /usr/local/hadoop/conf/mapred-site.xml`

add the following snippet between <configuration>...</configuration> tags.<br>
`<property>`<br>
`<name>mapred.job.tracker</name>`<br>
`<value>localhost:54311</value>`<br>
`<description>The host and port that the MapReduce job tracker runs`<br>
`at. If "local", then jobs are run in-process as a single map and reduce task.</description>`<br>
`</property>`<br>

### IV. hdfs-site.xml
Open up the file /usr/local/hadoop/conf/hdfs-site.xml
1. ` 	sudo gedit /usr/local/hadoop/conf/hdfs-site.xml`

add the following snippet between <configuration>...</configuration> tags.<br>
`<property>`<br>
`<name>dfs.replication</name>`<br>
`<value>1</value>`<br>
`<description>Default block replication. The actual number of replications can be specified when the file is created. The default is used if replication is not specified in create time.</description>`<br>
`</property>`<br>


### STEP 4: Formatting the HDFS Filesystem via the Namenode
***


The first step to starting up your Hadoop installation is formatting the Hadoop filesystem which is implemented on top of the local filesystem of your “cluster” (which includes only your local machine if you followed this tutorial). You need to do this the first time you set up a Hadoop cluster. 

### NOTE: Do not format a running Hadoop filesystem as you will lose all the data currently in the cluster (in HDFS)!

To format the filesystem (which simply initializes the directory specified by the dfs.name.dir variable), run the commands in a new terminal
1. ` 	su - hduser`
1. ` 	/usr/local/hadoop/bin/hadoop namenode -format`

The output will look like this:<br>
 ![](https://1.bp.blogspot.com/-ZPxMjxmu82Q/Wkm7eKQZfMI/AAAAAAAAFpg/8UurI8TVumsxySEeA91ymIJNwQnIG1o9ACLcBGAs/s1600/copy%2B3.png)<br>
Image 9. Formatting Namenode.


### Step 5: Starting your Single-Node Cluster
***


### 1. Run the command:
1. ` /usr/local/hadoop/bin/start-all.sh`

This will start a Namenode, a Datanode, a Jobtracker and a Tasktracker on your machine. The output will look like this:<br>
 ![](https://4.bp.blogspot.com/-PuhGsWeiTJ4/Wkm73JpjLnI/AAAAAAAAFps/mDbp3TwPaMQ652NytrrJ22FrnVeXre8VQCLcBGAs/s1600/Capture%2B9.PNG)<br>
Image 10. Starting the Single-Node Cluster.

### 2. Check whether the expected Hadoop processes are running:
1. ` 	cd /usr/local/hadoop`
1. ` 	Jps`

The output will look like this (Process ids and ordering of processes may differ):<br>
 ![](https://4.bp.blogspot.com/-XTCkqapizSw/Wkm8Ms6oHpI/AAAAAAAAFp0/jVc9WMFvKd0FSgZC5d_bSZq_EyCxcjLrACLcBGAs/s1600/Capture%2B10.PNG)<br>
Image 11. Running Hadoop Processes.

If all the six processes are running then your Hadoop is working fine.

### 3. You can also check if Hadoop is listening on the configured ports. Open a new terminal and run
1. ` 	sudo netstat -plten | grep java`

Output will look like this:<br>
 ![](https://4.bp.blogspot.com/-hviGTXWl6hg/Wkm8Vh1oVFI/AAAAAAAAFp4/3JTQbvS6mrMLNI40nu8IJSN6gwdwp0QGQCLcBGAs/s1600/Capture%2B11.PNG)<br>
Image 12. Hadoop is Listening.


### Step 6: Stopping your Single-Node Cluster
***


To stop all the daemons running on your machine, run the command:
1 	/usr/local/hadoop/bin/stop-all.sh

The output will look like this:<br>
 ![](https://1.bp.blogspot.com/-MqgK-_a2Ey8/Wkm8d4YNXJI/AAAAAAAAFp8/7IsG3HQi2osyd_0LjDAmJMe-6t-xq2vngCLcBGAs/s1600/Capture%2B12.PNG)<br>
Image 13. Stopping the Single-Node Cluster.

Congratulations! You have successfully installed your Hadoop. You can start working with Hadoop now; just remember to start your cluster first. Happy Hadooping !

