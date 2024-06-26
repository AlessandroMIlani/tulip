# Services



### General idea
We create pcap of N minutes on the virtual machine. We somehow download them, and use the `importer.py` script to analyze and import them into mongodb. The webapp does rest request to the webservices, that does query to mongodb.


### MongoDB structure
We use a single collection for all the pcaps
Each document will have:
```{
        "inx": //progressive flow index inside pcap
        "time": //start timestamp
        "duration": //end_time-start_time
        "src_ip": "127.0.0.1",
        "src_port": 1234 ,
        "dst_ip": "127.0.0.1",
        "dst_port": 1234,
        "contains_flag": //true if the importer have found that the flow contains a flag based on the env var regex
        "flow": [
            {
                "data": "...", // session data (capped at 15 MB)
                "from": "c" // "c" for client, "s" for server
                "time": //timestamp
            }, 
            ...
        ],

    }

```

# Services description
All the end-points return an object or an array of objects.

##### POST /query
Accept the following payload
```
    {
       flow.data: "regex on data field of flow",
       dst_ip: "1.2.3.4"
       dst_port: "1.2.3.4"
       time : {"$gte": from_millis,
               "$lt": to_millis}
    }

```
It returns an array of documents, WITHOUT the "flow" field

##### GET /services
Returns informations about all services. It is configurable on `configurations.py`

##### GET /flow/(flow_id)
Returns the all document with `flow_id` id, including the field `flow`

##### GET /star/(flow_id)/(0,1)
Set the flow favourite (1) or not (0)

##### POST /to_python_request/(tokenize)
convert the request to python syntax. Tokenize is used to toggle the auto-parsing of args.

##### GET /to_pwn/(id)
Convert the flow with the specified id in pwntools syntax

##### POST /update
Accepts a file via a post call, designed to work with the following curl command:
```bash
curl -F "file=/path/to/file" http://$ip:5000/upload -u "tulip:$password"
```
