# Mongo2Es examples

There are lots of ways to transfer data from Mongodb to Es. 

People would have different purpose

- Some would seek for log data purpose, so `_id` won't matter. 

- Some would want Mongodb's `_id` to remain in Es

- Some would even have Mongodb with String _id instead of hash (This was me)

- Some would want to sync ES and Mongodb

Will cover all 4 of these circumstances.

# 1. Logstash-input-plugin

using logstash, there are two ways to insert mongodb data to elasticsearch.
`logstash-input-mongodb plugin`
both plugins are useful while they do have different purpose.
hope this example helps people who needs to transfer mongo's _id to es's _id

## Instruction

1. ### **run mongodb/elasticsearch/kibana**

   ```
   docker-compose -f 1.docker-compose-esmongo.yml up -d
   ```


2. ### **run logstash to test examples**
   
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

3. ### insert data into mongodb

   There are lots of ways to import data. the easiest way is to use GUI tool like `mongo compass`.If not follow the above script inside the container. When testing `logstash_input_plugin` insert the data after logstash is running

   ```
    docker exec -it c-mongodb bash
    mongoimport -d mydb -c test_stringId --type csv --file /csv/test_stringId.csv --headerline
    mongoimport -d mydb -c test_hashId --type csv --file /csv/test_hashId.csv --headerline
   ```

4. ### check in Elasticsearch

   ```
   localhost:9200/test/_search
   # GET test/_search
   ```

## Examples

### Logstash-input-plugin

#### 2-1.dc-input-stringId.yml  : <ERROR> comes out

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

| sample csv                                                   | Fail log                                                     |
| ------------------------------------------------------------ | ------------------------------------------------------------ |
| ![image](https://user-images.githubusercontent.com/38391144/103174318-703fd500-48a4-11eb-822d-7a3cb9d3df6f.png) | ![image](https://user-images.githubusercontent.com/38391144/103173877-1558ae80-48a1-11eb-88c2-6681ffac78cf.png) |


#### 2-2.dc-input-hashId.yml

```
docker-compose -f .\2-2.docker-compose-input-hashId.yml up -d 
docker exec -it c-mongodb bash
mongoimport -d mydb -c test_hashId --type csv --file /csv/test_hashId.csv --headerline
```

_id comes out to be safe

| sample csv                                                   | Sucess Log                                                   | Es Example                                                   |
| ------------------------------------------------------------ | ------------------------------------------------------------ | ------------------------------------------------------------ |
| ![image](https://user-images.githubusercontent.com/38391144/103175085-cadc2f80-48aa-11eb-8627-63e635a230c7.png) | ![image](https://user-images.githubusercontent.com/38391144/103175092-d7608800-48aa-11eb-9462-93bf6d1d8283.png) | ![image](https://user-images.githubusercontent.com/38391144/103175101-e6dfd100-48aa-11eb-9d70-4ccb9ab63ee0.png) |

  

# 2. Logstash-jdbc-plugin

This part was tricky. versions are different, most of the info on stackoverflow are `mongojdbc1.3.jar/mongojdbc1.2.jar/mongojdbc1.6.jar` 

Doesn't really matter, but the whole folder needs to be in it like below.

https://dbschema.com/jdbc-driver/MongoDb.html-> click `Download JDBC driver

or

https://dbschema.com/jdbc-drivers/MongoDbJdbcDriver.zip

![image](https://user-images.githubusercontent.com/38391144/104128660-48746500-53ac-11eb-99ad-808db7246181.png)

#### 3-1.dc-jdbc-stringId.ym

working on it :(

#### 3-2.dc-jdbc-hashId.yml

working on it :(

#### 3-3.dc-jdbc-withOutId.yml

working on it :(

# 3. Monstache

working on it :(

# Etc

if your korean try reading my blog below

1. logstash-input-plugin : https://blog.naver.com/deet1107/222187443947
2. jdbc plugin : https://blog.naver.com/deet1107/222187516046
3. monstache : https://blog.naver.com/deet1107/222201939292


