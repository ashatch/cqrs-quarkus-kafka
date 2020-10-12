## About CQRS - Command Query Responsibility Segregation

According with [Martin Folwer](https://martinfowler.com/bliki/CQRS.html)
> At its heart is the notion that you can use a different model to update information than the model you use to read information.
> For some situations, this separation can be valuable, but beware that for most systems CQRS adds risky complexity.

## The application

Simulates a bank account scenario where an end user adds a income or expense transaction, and it is processed in a ascyncronous event sourcing and CQRS architecture to recalculate the user's bank account balance. The user can also request the balance of it's account. Down here you can see the design:

![Design](/images/design.png)

## Deploying the external services

```
docker-compose up -d --build
```
It will deploy four docker containers on your environment with MongoDB, PostgreSQL, Kafka and Zookepper (required by Kafka)

After deploying Kafka, you'll need to [create the topic on the Kafka cluster](https://kafka.apache.org/quickstart). For example:

```
docker exec -it bankaccount-kafka \
  ./bin/kafka-topics.sh --create \
  --topic transactions \
  --zookeeper bankaccount-zookeeper:2181 \
  --replication-factor 1 \
  --partitions 1
```

## Testing the application

#### Running a CURL request to create a income transaction
```
curl -X POST -H "Content-Type: application/json" -d @income-transaction.json http://localhost:8080/transactions
```
#### Running a CURL request to create a expense transaction
```
curl -X POST -H "Content-Type: application/json" -d @expense-transaction.json http://localhost:8080/transactions
```
#### Running CURL request to fetch the balance
```
curl http://localhost:8081/balance\?accountId\=wesley | json_pp
```
#### Running [K6's](https://k6.io) simple performance test
````
k6 run --vus 10 --duration 60s performance-tests/income.js
k6 run --vus 10 --duration 60s performance-tests/expense.js
````

### Inspecting the bankaccount database

```
docker exec -it bankaccount-postgres psql -U postgres -d bankaccount

bankaccount=# \dt
                  List of relations
 Schema |         Name          | Type  |    Owner
--------+-----------------------+-------+-------------
 public | flyway_schema_history | table | bankaccount
 public | transactions          | table | bankaccount
(2 rows)

bankaccount=# SELECT * from transactions;
                  id                  | account_id | description |  type   | value
--------------------------------------+------------+-------------+---------+-------
 c02ac0e7-d14c-4b91-825c-27512ee6ce7a | wesley     | Salary      | INCOME  |  1000
 b420d944-eeaf-4403-9fc5-189eb9b244b3 | wesley     | New Bike    | EXPENSE |   250
(2 rows)
```

### Contributing
I'd love to have a frontend for it! [Please reach me out if you got interested](MailTo:wesley.fuchter@gmail.com)
