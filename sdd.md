# App ABC

## Stakeholders, Roles and Assets

| Role                         | Name            |
|------------------------------|:---------------:|
| Author(s)                    | Tagger Foda |
| Architect                    | Fulano    |
| Product Owner                | Fulaninho   |

Project location:

[Github Project](https://)

## Business purpose
This app is responsible for ...


## Implementation summary / Design Overview
The microservice was written in Python, having modules with specific functions for communication, logging, and persistence.

### Package organization
[img]

### Main Routine
Once started, the main routine is described by the following diagram:

[activity diagram]

And below is the subroutine described on the main one:

[activity diagram]

## ![TCO](./assets/tco4.png) Total Costs of Ownership (TCO)

The costs relies on the following paid services:

- Object Store (S3);
- Rabbit MQ;
- Application Logs;

### Object Store

The object store costs were estimated based on 1,3 & 5 average products per quote, assuming the 
[App KPI](https://) 
of 5000 requests/minute.

The prices were extracted from the [aws costs](https://) and are assuming that the storage will take place at the Object Store from us-east:

- EUR 2.53 per month if block size < 50GB
- EUR 2.46 per month if 50GB <= block size < 100GB

#### Scenarios

| Scenario | Avg. Products per Quote | Avg. Throughput (logs/min) | Logs p/ Put |
|:--------:|:-----------------------:|:--------------------------:|:-----------:|
| 1        | 1                       | 5000                       | 1           |
| 2        | 3                       | 5000                       | 1           |
| 3        | 5                       | 5000                       | 1           |


#### Costs for Storage

| Scenario | Avg. Log Size (kB) | Growth p/ Month (GB) | **Total Price (Euro)**         |
|:--------:|:------------------:|:--------------------:|:------------------------------:|
| 1        | 2.5                | 540                  | (2.46 * 5) + 2.53 = **14,84**  |
| 2        | 3.9                | 842.4                | (2.46 * 8) + 2.53 = **22,21**  |
| 3        | 5.3                | 1144.8               | (2.46 * 11) + 2.53 = **29,59** |   

This cost increases in a monthly basis.


## Main libraries used
Currently, the app depends on 11 public python libraries, among the most important ones are:

- _aio-pika_ is used for the communication (AMQP);
- _aioboto3_ is used for persistence (Amazon S3);


## ![Storage](./assets/storage4.png) Storage strategy
The storage of the app is done by Amazon S3. It is triggered when app receives a message over the AMQP communication - see [Communication Methods section](#communication-methods). After that...


### Data Protection and Privacy

App ought to handle any type of message that respects its contract. However, there are specific paths that require additional `Data Protection and Privacy` checks.

The scenarios where...

### Storage operations

The diagram below illustrates the operations possible to be performed with the storage on the app. 
Since the document in S3 is just a regular file, it is not possible to read them beforehand and then decide which 
ones to maintain. This is why it's important to add some relevant information on the namespace 
(that may also be understood as the directory) of the file for query filtering. 

[img]


## Communication methods
As the app has no HTTP routes, messaging over AMQP is the exclusive communication method 
and no other communication channel with the app is available. RabbitMQ was chosen 
to be responsible for the messaging system. It works as a common platform among microservices to send and 
receive messages.

The message flow uses a fanout exchange, **app.exchange**, that routes the messages to the queues. Currently,
there is only one queue bound to this exchange. The following diagram demonstrates the messaging flow with 
the direction represented by the arrows:

[diagram]

## Scalability & Performance
The overall performance KPI for the app is 5000 requests per minute, which represents a throughput of 83 requests per second. 
However, the _app was configured to process a throughput of 400 requests per second. To support this 
throughput the app is being deployed with 5 instances, each one with 1024MB.


## Test strategy
The testing strategy was defined by having `unit`, `feature` and `performance` tests being executed at every build.

### Unit tests
Unit test are made with _unittest_ library. Coverage and quality metrics can be found at:

[Sonar Project](https://)

### Performance tests
Performance test executes massive batches of requests against the app
from a mock application that simulates the contract from the original application. The test starts every time the 
mock application receives an HTTP request. Once received the HTTP request, the mock application fires
two batches of messages, 15000 & 30000, respectively. Then, it compares the calculated throughput with the
KPI thoughput. The mock application and collection that triggers the tests can be found at:

[App Perfomance](https://)

### Feature tests
Feature tests executes some business scenarios to ensure the application behavior is correct. The tests are executed against a deployed application. The idea is to test only the app independently of its external communications. The tests are triggered by a Postman collection, which also verify if the content saved on the app is right. The mock application and collection that triggers the tests can be found at:

[App Feature](https://)

Test results for builds can be found at:

[Jenkins](https://)


## Security aspects
Currently there is no authorization concept for messaging operations, authenticated users can perform all operations with default scope.
Also, the application has no HTTP routes available reducing attack vectors, all the communication 
is done via RabbitMQ through internal AWS network.


## Operations
The main source of runtime information will be the application logs available on Application Log backing service, 
based on the incoming messages, this backing service provides the event history and real-time analysis. Furthermore, 
it is possible to check general status logs occurring during runtime. The logs might be of 3 severity levels: info, 
warning and error, as described below:

- `Info`: logs with this level are only for information purposes, not representing any unexpected action.

- `Warning`: logs with this level represent unexpected actions which can be resumed.

- `Error`: logs with this level represent runtime errors, actions that could not be performed.

All application logs related to the incoming messages shall contain the **correlation ID** to ensure the request trail.
