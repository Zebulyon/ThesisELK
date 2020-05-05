# ThesisELK


# Start all the containers
# Use this commmand in the project directory, which in this case is the ELK directory, it is where the docker-compse.yml file is located
docker-compose up -d

# Elasticsearch 

# Converting Pcap to Json
tshark -r C:\Users\Zebulyon\Documents\logs\logFiles\20191202_2\pll -T ek > tieto2.json
# To convert the files use this command, though takes note of the problems associated with the conversion process
# which leads to the creation of invalid formated files even if it is the offical method of performing this action

#Adding files
To add files use this curl command
curl -s -H "Content-Type: application/x-ndjson" -XPOST "localhost:9200/test_one/_bulk" --data-binary "@ModifiedJson.json"
# Where test_one is the name of the index while ModifiedJson.json is the file you want to add

# Retriving Files or Data 

curl -s -H "Content-Type: application/json" -XGET "localhost:9200/logstash_ss7trace/_search?size=0&pretty" -d @curlGet.txt > exportedTerms.json
# Where curlGet.txt is a file containing the query specifing which data fields to fetch from the specified index
# exportedTerms.json is the file to which teh output is written to

