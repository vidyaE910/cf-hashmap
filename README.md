## cf-hashmap-service

#### Contents
```
  * Make Service Broker
  * making the single plan in the catalog ``public''
  * creating an instance of a service
  * pushing and binding a test app to that service
  * testing the app's interaction with the service
```

The below are the steps to follow with hashmap-service.

#####Steps

* Download cf-hash-service.zip
* unzip the package
* Build the app using Gradle:
```
   $ ./gradlew assemble 
```   
* Target required CF Organization and Space to deploy the Service
```
cf target [-o ORG] [-s SPACE]
```
* Push the app ```cf-hash-service``` to the targeted space
```
$ cf push -m 512m -p build/libs/cf-hash-service-1.0.jar cf-hash-service
``` 

* Make sure the app is running, then register service with the Cloud Controller
```
$ cf create-service-broker cf-hash-service dbsadmin dbsadmin <<cf-hash-service URL>>
```

* Make sure the service plan public
```
$ cf curl /v2/service_plans -X 'GET'
``` 
Find the GID from above command, Make as Public
```
$ $ cf curl /v2/service_plans/<<GID>> -X 'PUT' -d '{"public":true}'
```

* verify Marketplace nowe:
```
$ cf marketplace
```

* Now Create Service Instance 
```
$ cf create-service HaaSh basic my-hash
```

* the Service is ready to consume. get the app and bind the above service
* Download cf-hash-service.zip
* unzip the package
* Build the app using Gradle:
```
$ ./gradlew assemble
```

* Push the client app to CF by targeting the cf ORG and SPACE
```
$ cf push
```

* Make sure the app is running
* Now ready to test
```
$ curl <<Client URL>>/HaaSh/foo -d '{"1":"15264"}' -X PUT

$ curl <<Client URL>>/HaaSh/foo -d '{"2":"52412"}' -X PUT

$ curl <<Client URL>>/HaaSh/foo -X GET
{"1":"15264"}
{"2":"52412"}
```


