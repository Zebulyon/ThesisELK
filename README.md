# ThesisELK


# Start all the containers


Use this commmand in the project directory, which in this case is the ELK directory, it is where the docker-compse.yml file is located

docker-compose up -d


To turn of the containers properly use 

docker-compose down


To pause specfic containers, here persistant data for Filebeat and Logstash will be kept between runs

docker-compose stop CONTAINER-NAME


To disable all paused containers, this will delete the perssistent data for Logstash and Filebeat

docker-compose rm


# Elasticsearch 

# Converting Pcap to Json

tshark -r PATH-TO-PCAP-FILE -T ek > FILE-TO-WRITE-TO.json
 
	When converting the files using this method keep in mind of the problems associated with the conversion process
	which leads to the creation of invalid formated files even if it is the offical method of performing this action.
	The problems comes in the form of multiple fields with the same name within the same event. 


Adding files
		The -H "Content-Type: application/x-ndjson" part is a safty precatuttion added in laterversions of Elasticsearch
		it instructs of what type of content is being added

curl -s -H "Content-Type: application/x-ndjson" -XPOST "localhost:9200/INDEX-NAME-MUST-BE-LOWERCASE/_bulk" --data-binary "@FILE-TO-UPPLOAD.json"


# Retriving Files or Data 

Here is an example of rettriving data from the index logstash_ss7trace

curl -s -H "Content-Type: application/json" -XGET "localhost:9200/logstash_ss7trace/_search?size=0&pretty" -d @curlGet.txt > exportedTerms.json
	
	Where curlGet.txt is a file containing the query specifing which data fields to fetch from the specified index, this file is included in the project
	exportedTerms.json is the file to which teh output is written to. 

For more indepths explenations, tutorials and guides I refere you to these pages

https://logz.io/blog/elasticsearch-queries/

logz has a multitude of easy to read step by step guides for the entire Elastic-stack

Then there is of course the offical documentation on elstics websites

https://www.elastic.co/guide/en/elasticsearch/reference/current/rest-apis.html
	
	
My specific search query example look like this :  

	{
		"query": {
			"match": {
				"log.sequence.keyword": "7663885"
				}
			}, 
			"aggs": {
				"Exported_terms": {
					"terms": {
						"field": "message.keyword",
						"size": 1
					}
				}
			}
	}
	
	This is a very basic query. In "match" field one specifies the index field of interest and the value of interest.
	While in the "aggs" field one specifies which fields from the event, found through the match field, we want to extract in the "field" header. The "size" header dictates how
	many responses that fullfills the "match" field that we want to retrive. 
	The index in question that we want to look at is specified in the curl command. 
	
# Logstash

All patterns use some level of REGEX patterns. 

For GROK filter documentation : https://www.elastic.co/guide/en/logstash/current/plugins-filters-grok.html
	For premade patterns regarding GROK : https://github.com/logstash-plugins/logstash-patterns-core/blob/master/patterns/grok-patterns
	
For DATE filter documentation : https://www.elastic.co/guide/en/logstash/current/plugins-filters-date.html
For MUTATE filter documentation : https://www.elastic.co/guide/en/logstash/current/plugins-filters-mutate.html

For complete list of all official Logstash filters : https://www.elastic.co/guide/en/logstash/current/filter-plugins.html

To create a custom pattern : 
	(?<FIELD-NAME>COMBINATION OF PATTERNS)
Example from the logstash_ss7trace.conf : 
	
	(?<timestamp_string>%{YEAR} %{MONTH} %{MONTHDAY}  %{TIME}:%{INT})
	
	Here timestamp_string is the field name 
	and everything after the '>' symbol is the custom pattern used to handle the unique timestamp format
	
	
	(?<process.sender.module>[^:]*)
	
	Here process.sender.module is the field name
	This field is used as the field can contain any symbol except SPACE which makes the GREEDYDATA pattern that can handle any
	symbol to strong because Logstash do not know when to stop. 
	In this pattern after the instructions after the '>' tells Logstash to include all data until it discovers a ':'
	

In Logstash one can have multiple inputs and outputs active at the same time, but it can not run multiple conf files at the same time
On top of that is the fact that Logstash is very memory consuming as it is based on a JVM, making it so that it could be more benefical 
to run more configuration in different iterations than doing everything at the same time. 

But it is possible to combine all of the Logstash .conf files into a single one because Logstash allows the usage of if-statements, as can be seen in logstash_sss7trace.conf
where I check if a specific field exsist before removing this. A possible way of doing it would be to check the file type being processed and using specific fields depending on that.

In the .conf files I have two GROK filter fields, in which one of them just takes the entire message and stores it in the field log.original field. This is because probelms
could appear when break_on_match is set to true, as certain messages are very similar to one another which could lead to multiple events from the input stream being combined 
into a single event for the output stream. 


# Filebeat

The big thing here is that the .yml files must be made by a superuser if created outside of the container and transfered into it.

This is because in filebeat only a superuser can use the files even when they are only used for starting the service, and even if set it so in the docker-compose file that it is a 
superusers that is logged into the container the problem persist and nothing i tried in regards to chaing the permission in the filebeat container solved it. 
I do not know if this bug is associated with the container versions I'm using, if the problem is with my own computer, if it is with docker or if it is a general bug in filebeat. 

I will also mention that the filebeat .yml files is in a way similar to Python very strict when it comes to spacing and could easily cause probelms if there every is to many or to few. 

Filebeat is able to read and handle multiple different types of files in a single file, an example would be the filebeat_LOG_ss7trace.yml file, in there I have commented out a section in the filebeat.input
section which would allow this .yml file to also read all of the systemlog files that would be copied to filebeat. 


# Kibana 

To access Kibana go to localhost:5601, this port number is the standard one for Kibana


# JOINING INDEXES

	 Is impossible in Elasticsearch it is one of its drawback. The only work around is to format the data in the logs such that they could all be in the same index. 
	 This means that fields such that timestamp must be named the same. 



# Field Mapping
	
	For a complete of all the offical field naming as per the Elastic Common Schema (ECS): https://www.elastic.co/guide/en/ecs/current/ecs-field-reference.html

# Custom Field Mapping description
	As there were certain field in the log files that I could not find a good representaion for in ECS I had to create some custom fields
	
	# ss7trace
	
	event.type 				: Containes the information regarding if the Event in question is a SENT, RECEIVED, LINKED.
	process.type 				: Is the prcess number
	log.sequence 				: Is the sequence number for the logged event
	process.sender.module			: The name of the sending process module
	process.sender.module_instance		: The PID of the sending process module
	process.sender.module			: The name of the receiving process module
	process.receiver.module_instance	: The PID of the receiving process module
	event.message.type			: The Primitive value
	message_size				: The size value
	message_data.size			: The MD_size value
	
	log.internal.level			: The L: value
	
	log.module.name				: Name of the module that gave the Syslog message
	log.module.instance			: The instance of the module that gave the Syslog message
	log.module.pid				: The PID of the module that gave the Syslog message
	log.module.thread.id			: The TID of the module that gave the Syslog message
	log.module.level.internal		: The L: of the module that gave the Syslog message
	
	n1.zero					: These values I was unable to fund a good naming scheme for but they are the values 	
	error.arg1				  associated in this documentation
	error.arg2
	error.arg3
	
YYYY MON DAY hh:mm:ss:msec <process type>:<sequence number><\n>
**** <MODULE>:<instance> PID:<pid> TID:<TID>
L:<ll> <filename><ln><nl><n2><p1><p2><p3>
<Error_Code><\n>
Where:
YYYY MON DAY
hh:mm:ss:msec
Date and time the error was reported by the module.
<process type> Message port owner of the process reported the error.
<sequence number> Sequence number of this log message. Each process counts its messages separately starting with 1.
**** Indicates the line in the ss7trace.log file, contains error code, see the Note.
<MODULE> Is the name of the module which reported the error.
<instance> Instance of the logging module.
<pid> PID of the logging module.
<TID> Thread ID of the logging module.
<ll> Log level of the logging module.
<filename> Is the source code file name (for internal use).
<ln> Line number of the source code (for internal use).
<n1> 0 (zero).
<n2> The process ID (Identification).
<p1> Meaning of this parameter depends on the error code.
<p2> Meaning of this parameter depends on the error code.
<p3> Meaning of this parameter depends on the error code.
<Error_Code> The error code which holds the actual error value.
<\n> Line break.


	# Syslog
	
	process.user 				: Contains the information about which user created the log, example root user


	# Functions of Interest
	
	If one goes to the Dev Tool tab, then you get access to a console window where one can write and test out search queries and filters with the aid of autocomplete
		There is a also a subwindow for Debuggin GROK Patterns


# To create an Index Pattern

This is needed if one would want to access the index in Kibana

	Go to:
	management -> Index Patterns -> Create Index Pattern
	
# To solve a minor Timebug that you might encounter

	The bug is that while no changes are done to the data, because a incorrect Timezone is set Kibana will change will treat all of the timestamps as something else
	which could lead to confusion during error searching
	
	Go to:
	management -> Advanced Settings -> Timezone for date formatting 
		Set the value to UCT 
		
		
		
# Docker
	
	# Store data between sessions
	
	This is no problem as their persistant data is stored in a docker voluem specified in the docker-compose.yml file 
	
	# Move the persistant data between machines
	
	I admit to not having been able to figure this one out because of a lack of time, but it is possible to do. 
	With persistant data I refere to all of the already stored logs in Elasticsearcha and the created Visualization in Kibana
	
	