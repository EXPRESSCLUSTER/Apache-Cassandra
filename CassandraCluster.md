# Cassandra DB Cluster Quick Start Guide
## Overview
This guide provieds how to integrate Cassandra DB with EXPRESSCLUSTER X (ECX) and create Cassandra DB cluster.

## Preparation
- Cassandra Download  
	http://cassandra.apache.org/download/
- Cassandra Installation and Configuration  
	http://docs.datastax.com/en/archived/cassandra/3.x/cassandra/configuration/configTOC.html
- JDK Download  
	http://www.oracle.com/technetwork/java/javase/downloads/index.html
- commons-daemon Download  
	http://archive.apache.org/dist/commons/daemon/binaries/windows/
- Python Download  
	https://www.python.org/downloads/

## System Configuration
```bat
[LAN]
 |     Primary Server
 |   +----------------------+
 |   |  Windows Server      |
 +---+   - ECX              |  <--+
 |   |   - Apache Cassandra |     |
 |   |   - JDK              |     |
 |   |   - commons-daemon   |     |
 |   +----------------------+     |
 |                            Clustered
 |    Secondary Server            |
 |   +----------------------+     |
 |   |  Windows Server      |     |
 +---+   - ECX              |     |
 |   |   - Apache Cassandra |  <--+
 |   |   - JDK              |
 |   |   - commons-daemon   |
 |   +----------------------+
 |
 |
 |    Client Machine
 |   +----------------------+
 |   |  Windows Server      |
 +---+   - Python           |
 |   |   - Apache Cassandra |
 |   +----------------------+
 |
 :
```
- Primary/Secondary Servers
	- OS: Windows Server 2016 (Standard)
	- Disk
		- C: 21GB for system drive
		- E: 2GB for Mirror Disk Data Partition
		- F: 1GB for Mirror Disk Cluster Partition
	- Memory: 4GB
	- SW
		- EXPRESSCLUSTER X: 3.3 or 4
		- Apache Cassandra : apache-cassandra-3.11
		- JDK: jdk-8u131-windows-x64.exe
		- commons-daemon: commons-daemon-1.0.14-bin-windows.zip

- Client Machine
	- OS: Windows Server 2016 (Standard)
	- Disk
		- C: 21GB for system drive
	- Memory: 4GB
	- SW
		- Python: python-2.7.13
		- Apache Cassandra : apache-cassandra-3.11

## Setup

1. Install EXPRESSCLUSTER
	- On both Primary and Secondary Servers
 		1. Install EXPRESSCLUSTER.
		1. Start License Manager and register EXPRESSCLUSTER licenses.
			- Core license
			- Rep license
1. Create Cluster
	- On Primary Server
		1. Start WebManager and create Cluster.
		1. Add Servers.
			- Server1: Primary
			- Server2: Secondary
		1. Create Failover group.
			- fip resource
			- md resource
				- E: Data Partition
				- F: Cluster Partition
		1. Apply the configuration.
		1. Start the group on Server1.
1. Install JDK
	- On both Primary and Secondary Servers
		1. Install "jdk-8u131-windows-x64.exe".
		1. Set Environment Variable on both Servers.
			- Variable: JAVA_HOME
			- Value: %JDK Installation Path% (*)  
				\* e.g.) C:\Program Files\Java\jdk1.8.0_131
		1. Start Command Prompt and execute the following command to confirm that java is enabled on both Servers.  
			```bat
			C:> java -version
			java version "1.8.0_131"
			```
1. Install Cassandra
	- On Primary Server
		1. Decompress and store apache-cassandra-3.11.
		1. Set Environment Variable on both Servers.
			- Variable: CASSANDRA_HOME
			- Value: %Cassandra Installation Path% (*)  
				\* e.g.) C:\Program Files\apache-cassandra-3.11
			- Variable: CASSANDRA_CONF
			- Value: %Cassandra Installation Path%\conf (*)  
				\* e.g.) C:\Program Files\apache-cassandra-3.11\conf
		1. Create folders.
			- E:\cassandra\commit_log
			- E:\cassandra\data
			- E:\cassandra\saved_cache
		1. Edit the follwing parametares. on Primary Server.  
			Note to remove "#" and " " before the parameters in each lines. (If " " remains in the head of lines, Cassandra DB will fail to start.)
			- data_file_directories: E:\cassandra\data
			- commitlog_directory: E:/cassandra/commit_log
			- saved_caches_directory: E:\cassandra\saved_cache
			- listen_address: fip
			- rpc_address: fip
			- start_rpc: true
			- seeds: fip
		1. Start Command Prompt and execute the following command to confirm that Cassandra DB can start on Server1.  
			```bat
			C:> cd %CASSANDRA_HOME%
			C:\%CASSANDRA_HOME%> cassandra
			 :
			INFO  [main] 2017-06-25 20:29:02,619 Server.java:156 - Starting listening
			for CQL clients on /10.4.3.147:9042 (unencrypted)...
			 :
			C:\%CASSANDRA_HOME%> Ctrl+C
			```
		1. Move the failover group to Secondary Server
	- On Secondary Server
		1. Do the same as step 4-i - 4-v on Primary Server.
		1. Move the failover group to Primary Server.

1. Register cassandra service
	- On Primary Server
		1. Decompress and store commons-daemon (commons-daemon-1.0.14-bin-windows.zip).  
			e.g.) C:\temp\commons-daemon-1.0.14-bin-windows
		1. Create "daemon" folder under "%CASSANDRA_HOME%\bin".
		1. Copy and store "commons-daemon-1.0.14-bin-windows\amd64\prunsrv.exe" file under "%CASSANDRA_HOME%\bin\daemon".
		1. Start Command Prompt and execute the following command to register Cassandra DB as a service.
			```bat
			C:> cd %CASSANDRA_HOME%
			C:\%CASSANDRA_HOME%> cassandra -install
			```
		1. Start Services and confirm that "cassandra" service exists.
		1. Move the failover group to Secondary Server.
	- On Secondary Server
		1. Do the same as step 5-i - 5-v on Primary Server.
		1. Move the failover group to Primary Server.

1. Create cassandra cluster
	- On Primary Server
		1. Start WebManager and add service resource.
			- Target service: cassandra
 			- Wait time after service strated: 10sec
		1. Apply the configuration.

1. Install Python
	- On Client Machine
		1. Install python-2.7.13.amd64.msi.
		1. Set Environment Variable on both Servers.
			- Variable: Path
			- Value: ;C:\Python27;C:\Python27\Scripts;
			- Variable: PYTHONPATH
			- Value: C:\Python27\Lib\site-packages
		1. Reboot OS.
		1. Start Command Prompt and execute the following command to confirm that Python is enabled.
			```bat
			C:> python
			Python 2.7.13 (v2.7.13:a06454b1afa1, Dec 17 2016, 20:53:40) [MSC v.1500 64 bit (
			AMD64)] on win32
			Type "help", "copyright", "credits" or "license" for more information.
			>>>Ctrl+Z
			```

1. Install Cassandra for cql
	- On Client Machine
		1. Decompress and store apache-cassandra-3.11.
		1. Set Environment Variable on both Servers.
			- Variable: CASSANDRA_HOME
			- Value: %Cassandra Installation Path% (*)  
				\* e.g.) C:\Program Files\apache-cassandra-3.11
			- Variable: CASSANDRA_CONF
			- Value: %Cassandra Installation Path%\conf (*)  
				\* e.g.) C:\Program Files\apache-cassandra-3.11\conf

1. Connect to Cassandra from Client machine
	- On Client Machine
		1. Start Command Prompt and execute the following command to confirm to connect Cassandra DB server.
			```bat
			C:> cd %CASSANDRA_HOME%\bin
			%CASSANDRA_HOME%\bin> cqlsh <fip address>
			 :
			Connected to Test Cluster at <fip address>:9042.
			```
		1. Execute the following command to create keyspace and table.
			```bat
			cqlsh> create keyspace <keyspace name> with replication = {'class': 'SimpleStrategy', 'replication_factor': 1};
			cqlsh> describe <keyspace name>
			cqlsh> use <keyspace name>;
			cqlsh> create table <table name>(
			         ... id int PRIMARY KEY,
			         ... name text
			         ... );
			cqlsh> describe tables
			cqlsh> insert into <table name> (id, name) values (1, 'Server1');
			cqlsh> select * from <table name>;
			```
	- On Primary or Secondary Server
		1. Start WebManager and move the failover group to Secondary Server.  
	- On Cassandra Client Machine
		1. Execute the following command to confirm the table is mirrored, then update table.  
			```bat
			cqlsh> select * from <table name>;
			cqlsh> update <table name> set name = 'Server2' where id = 1;
			cqlsh> select * from <table name>;
			```
	- On Primary or Secondary Server
		1. Start WebManager and move the failover group to Primary Server.
	- On Cassandra Client Machine
		1. Execute the following command to confirm updated table is mirrored.
	- On Client Machine
		1. Execute the following command to create keyspace and table.  
			```bat
			cqlsh> select * from <table name>;
			```
