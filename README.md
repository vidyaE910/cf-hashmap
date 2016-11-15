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
   ----
   $ ./gradlew assemble 
   ----
   
* Target required CF Organization and Space to deploy the Service

$ cf push -m 512m -p build/libs/haash-broker-0.0.1-SNAPSHOT.jar haash-broker
----

. Once the app is running, register the broker with the Cloud Controller (substitute the route for your broker app):
+
[source,bash]
----
$ cf create-service-broker haash-broker warreng natedogg http://haash-broker.54.235.203.95.xip.io
----

. Next, we need to make the service plan public, as all plans start private by default. This step is a little more manual than the others. First, you need to gain access to the Service Plan GUID:
+
[source, bash]
----
$ cf curl /v2/service_plans -X 'GET'
{
  "total_results": 1,
  "total_pages": 1,
  "prev_url": null,
  "next_url": null,
  "resources": [
    {
      "metadata": {
        "guid": "ec1da994-eba0-4f50-9453-e00b155a938b",
        "url": "/v2/service_plans/ec1da994-eba0-4f50-9453-e00b155a938b",
        "created_at": "2014-06-26T19:59:02+00:00",
        "updated_at": "2014-06-26T20:00:50+00:00"
      },
      "entity": {
        "name": "basic",
        "free": true,
        "description": "Basic Plan",
        "service_guid": "d458ac57-993a-40cf-925c-d56934a0e494",
        "extra": null,
        "unique_id": "1",
        "public": false,
        "service_url": "/v2/services/d458ac57-993a-40cf-925c-d56934a0e494",
        "service_instances_url": "/v2/service_plans/ec1da994-eba0-4f50-9453-e00b155a938b/service_instances"      }    }
  ]
}
----
+
You find the Plan GUID at +resources.metadata.guid+. Next you'll send a request to update it:
+
[source, bash]
----
$ $ cf curl /v2/service_plans/ec1da994-eba0-4f50-9453-e00b155a938b -X 'PUT' -d '{"public":true}'
{
  "metadata": {
    "guid": "ec1da994-eba0-4f50-9453-e00b155a938b",
    "url": "/v2/service_plans/ec1da994-eba0-4f50-9453-e00b155a938b",
    "created_at": "2014-06-26T19:59:02+00:00",
    "updated_at": "2014-06-27T13:21:50+00:00"
  },
  "entity": {
    "name": "basic",
    "free": true,
    "description": "Basic Plan",
    "service_guid": "d458ac57-993a-40cf-925c-d56934a0e494",
    "extra": null,
    "unique_id": "1",
    "public": true,
    "service_url": "/v2/services/d458ac57-993a-40cf-925c-d56934a0e494",
    "service_instances_url": "/v2/service_plans/ec1da994-eba0-4f50-9453-e00b155a938b/service_instances"
  }
}
----

. You should now be able to see your service in the marketplace:
+
[source, bash]
----
$ cf marketplace
Getting services from marketplace in org default / space development as admin...
OK

service   plans   description
HaaSh     basic   HaaSh - HashMap as a Service
----

. Next, create an instance of your service:
+
[source, bash]
----
$ cf create-service HaaSh basic my-hash
----

. Now it's time to push and bind to the client app. First clone the haash-client repo:
+
[source,bash]
----
$ git clone https://github.com/mstine/haash-client.git
$ cd haash-client
----

. Build the project using Gradle Wrapper:
+
[source,bash]
----
$ ./gradlew assemble
----

. We'll use a CF application manifest to take care of our metadata, including binding to the HaaSh service. It's already in the project, but you can see it reproduced here:
+
[source,yaml]
----
applications:
- name: haash-client
  memory: 512M
  path: build/libs/haash-client-0.0.1-SNAPSHOT.jar
  services:
  - my-hash
----
+
This allows a simpler push experience:
+
[source,bash]
----
$ cf push
----

. Once the application is running, you can test it:
+
[source,bash]
----
$ curl haash-client.54.235.203.95.xip.io/HaaSh/foo -d '{"value":"bar"}' -X PUT
{}

$ curl haash-client.54.235.203.95.xip.io/HaaSh/foo -X GET
{"value":"bar"}
----

. Congratulations! You've completed the lab. Don't forget to spend some time looking at the source code for haash-broker and compare what you see to the [Service Broker API Documentation](http://docs.cloudfoundry.org/services/api.html).
