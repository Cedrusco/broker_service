This is a sample application which to illustrate on how to create a service in PCF

Download cf cli using homebrew
```console
$ brew install cloudfoundry/tap/cf-cli@7
```
Login to cloud foundry, here we will be using the public hosting cloud, in enterprise we would have different url
```console
$ cf login -a api.run.pivotal.io
```
find the list of available spaces and a space from list or create a new one
```console
$ cf spaces
$ cf target -s development
```
if you would like to create a new space
```console
$ cf create-space demo-service
$ cf target -s demo-service
```
# Pushing and testing the service on PCF
## Now lets deploy the service to PCF
lets clone the application and build it
```console
$ git clone https://github.com/Cedrusco/broker_service.git
$ cd broker_service
$ ./gradlew assemble
```
## Pushing the application to PCF
Once the gradle assemble is complete, we are ready to push the application to pcf

```console
$ cf push --random-route
```
The cf push would push the app from current directory to PCF and it would use manifest.yml
The above command will assign a random route to the deployed application.

## Listing deployed application
```console
$ cf apps
```
once you run this command, you should see all the available apps deployed to the specific org and spaces
## Invoking the deployed service
```console
$ broker_url=$(cf app broker-service | grep routes: | awk '{print $2}')
$ curl warreng:natedogg@${broker_url}/v2/catalog
```
The above curl would return some data
```console
{"services":[{"id":"30456","name":"broker-service-9434","description":"HaaSh - HashMap as a Service","bindable":true,"plans":[{"id":"21099","name":"basic","description":"Basic Plan"}],"metadata":{"displayName":"HaashBroker","imageUrl":"http://i.ytimg.com/vi/1plPyJdXKIY/mqdefault.jpg","longDescription":"Hash map as a service","documentationUrl":"https://www.youtube.com/watch?v=1plPyJdXKIY","supportUrl":"https://www.youtube.com/watch?v=1plPyJdXKIY"}}]}
```
# Creating and Registering the Service
now lets target to different space
```console
$ cf target -s demo-apps
```
To register the private broker into this space we use the new --space-scoped flag:
```console
$ cf create-service-broker broker-service  \
  warreng natedogg https://${broker_url} \
  --space-scoped
```
List the services that are available for binding
```console
$ cf marketplace
```
you should see your service available now

You can now create and bind service instances, and service keys:
```console
$ cf create-service broker-service basic my-haash-db
cf create-service-key my-haash-db my-keys 
cf service-key my-haash-db my-keys
```