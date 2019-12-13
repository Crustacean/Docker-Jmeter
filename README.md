RUNNING A JMETER DOCKER CONTAINER IN A SERVER FOR CI/CD WITH INFLUXDB AND GRAFANA FOR REPORTING

Create a jmx file in jmeter and give it a name e.g. 'jmeter.jmx'

In your server that has docker installed, create a docker network called 'jnet':

#docker network create jnet

To inspect your network that you just created, run:

#docker network inspect jnet

Run influxdb in your server within the 'jnet' network that you just created:

#docker run --network jnet --name influxdb -d -p 8086:8086 -v influxdb:/var/lib/influxdb influxdb

	[ to run influx db with custom config file run the following command to have the config file in your
	[ working directory. Navigate to your working directory and run:
	[
	[ #docker run --rm influxdb influxd config > influxdb.conf
	[ 
	[ Modify the default configuration, which will now be available under $PWD.
	[ Then start the InfluxDB container
	[ #docker run --network jnet --name influxdb -v influxdb:/var/lib/influxdb -d -p 8086:8086 -v $PWD/influxdb.conf:/etc/influxdb/influxdb.conf:ro influxdb -config /etc/influxdb/influxdb.conf influxdb
	[
	[ This will start influx db with the new configs. However, modifying this config file is not neccesary if you are using an influxdb backend listener like we are here.

Run grafana in your server within the 'mynet' network that you just created and link it to influx db using the '--link' command:

#docker run --network jnet --name=grafana --link influxdb -d -p 3000:3000 -v grafana:/var/lib/grafana grafana/grafana

Access your running influxdb container to create a Database for your measurements.
#docker exec -it influxdb influx

When promped by the terminal run the following commands:
#show databases
#create database jmeter
#show databases

Now you have a database for your jmeter results.

	[ You can also create a database from the commandline into the running container without neccesarily accessing the running container
	[
	[ #curl -i -XPOST http://[yourServerIPAddress]:8086/query --data-urlencode "q=CREATE DATABASE jmeter"
	[
	[ To view your databases created rn,
	[ #curl -i -XPOST http://[yourServerIPAddress]:8086/query --data-urlencode "q=show databases"
	[

You now need to get the IP address of your running influxdb container to allow jmeter be able to post data to it.
Run:
#docker inspect influxdb | grep IP
or
#docker ps
#docker inspect <your running influxdb container ID> | grep IP

Search for 'IPAddress'. This is your container IP address in your server.
Place this in you jmeter file like so:
http://[yourContainerIPAddress]:8086/write?db=jmeter

Build the docker container using the docker file

#docker build --no-cache=True -t jmeter/testenv .

Run docker container in your server within the 'jnet' network that you just created. The volume is mapped to the working directory 
specified in the Docker file. If at all the jmeter version in the Docker file is changed, then you should definately change this volume path 
accordingly:

#docker run --network jnet --rm -v jmeterResults:/opt/apache-jmeter-5.2.1 jmeter/testenv influxdb 8086