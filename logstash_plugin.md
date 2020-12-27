# Using logstash to transfer data from mongo to es
using logstash, there are two ways to insert mongodb data to elasticsearch.
`logstash-input-mongodb plugin` and `jdbc plugin`.
both plugins are useful while they do have different purpose.
hope this example helps people who needs to transfer mongo's _id to es's _id

# Instruction

1. ## **run mongodb/elasticsearch/kibana**

   ```
   docker-compose -f 1.docker-compose-esmongo.yml up -d
   ```


2. ## **run logstash to test examples**
   
    ``` 
    docker container prune
    docker-compose -f 2.docker-compose-xxxxx.yml up
    ```
    
    Divided docker-compose files . I know String `_id` doesn't make sense since it should be hash. But there are occasions things are already set like it. 

| docker-compose | explanation | etc |
| ------------------------------------- | ---- | ---- |
| 2-1.dc-input-stringId.yml | `mongo` _id needs to be ObjectId | - error due to `_id`'s type |
| 2-2.dc-input-hashId.yml | `input` generateId => true<br />`filter` remove _id <br />`output` document_id => ${mongoId} | - only catches insert action |
| 3-1.dc-jdbc-stringd.yml<br />3-2.dc-jdbc-hashId.yml | `input` generateId => true<br />`filter` extract result from `value` <br />`output` document_id => ${mongoId} | - db.test.find({})<br />- jdbc.jar is tricky |
| 3-3.dc-jdbc-withOutId.yml |`input` generateId => true<br />`filter` remove _id <br />|- db.test.find({},{'_id': false})<br />- jdbc.jar is tricky|

3. ## insert data into mongodb

   There are lots of ways to import data. the easiest way is to use GUI tool like `mongo compass`.If not follow the above script inside the container. When testing `logstash_input_plugin` insert the data after logstash is running

   ```
    docker exec -it c-mongodb bash
    mongoimport -d mydb -c test_stringId --type csv --file /csv/test_stringId.csv --headerline
    mongoimport -d mydb -c test_hashId --type csv --file /csv/test_hashId.csv --headerline
   ```

4. ## check in Elasticsearch

   ```
   localhost:9200/test/_search
   # GET test/_search
   ```

# Examples

## Logstash-input-plugin

### 2-1.dc-input-stringId.yml  : <ERROR>

```
docker-compose -f .\2-1.docker-compose-input-stringId.yml up -d 
docker exec -it c-mongodb bash
mongoimport -d mydb -c test_stringId --type csv --file /csv/test_stringId.csv --headerline
```

error comes out because I inserted _id which is a string type
- ` "_id"=>"A001" `
- :exception=>#<BSON::ObjectId::Invalid: 'A001' is an invalid ObjectId.>

```
[logstash.inputs.mongodb  ][main][88302e7ea6ef3ac784d5537611eb0478d8803f1f4089f4269a9a58ffb444163c] init placeholder for logstash_since_test_stringId: {"_id"=>"A001", "Name"=>"Jane Doe", "Address"=>"123 Main St", "City"=>"Whereverville", "State"=>"CA", "ZIP"=>90210}

[WARN ][logstash.inputs.mongodb  ][main][88302e7ea6ef3ac784d5537611eb0478d8803f1f4089f4269a9a58ffb444163c] MongoDB Input threw an exception, restarting {:exception=>#<BSON::ObjectId::Invalid: 'A001' is an invalid ObjectId.>}
```

![1609078467777](C:\Users\deet1\AppData\Roaming\Typora\typora-user-images\1609078467777.png)

### 2-2.dc-input-hashId.yml

```
docker-compose -f .\2-2.docker-compose-input-hashId.yml up -d 
docker exec -it c-mongodb bash
mongoimport -d mydb -c test_stringId --type csv --file /csv/test_hashId.csv --headerline
```

![1609078455968](C:\Users\deet1\AppData\Roaming\Typora\typora-user-images\1609078455968.png)

![1609078445921](C:\Users\deet1\AppData\Roaming\Typora\typora-user-images\1609078445921.png)

---

## Jdbc plugin 

### 3-1.dc-jdbc-stringId.ym

### 3-2.dc-jdbc-hashId.yml

### 3-3.dc-jdbc-withOutId.yml

# etc
if your korean try reading my blog below
