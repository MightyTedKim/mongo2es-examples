# working on monstache demo.
monstache is a great third party tool for synchronizing data
it supports all insert/update/delete. also has a great example on github to follow
- https://github.com/rwynn/monstache
- https://github.com/rwynn/monstache-showcase

## 3. monstache

# instruction

1. run mongodb/elasticsearch/kibana
```docker-compose -f 1.docker-compose-esmongo.yml up -d ```

2. run logstash
``` docker-compose -f 2.docker-compose-monstache.yml up ```
running without -d is recommended to see the logs

3. insert data into mongodb
```
 docker exec -it c-mongodb bash
 mongoimport -d mydb -c test --type csv --file /test.csv --headerline
```
There are lots of ways to import data. the easiest way is to use GUI tool(mongo compass).
If not follow the above script inside the container

4. check in Es

``` localhost:9200/test/_search ```

# etc
if your korean try reading my blog below
