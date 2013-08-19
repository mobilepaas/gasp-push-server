Gasp! Amazon SNS Mobile Push Server
-----------------------------------

Push data synchronization server for Gasp! demo: uses CloudBees PaaS and Foxweave to provide automatic data sync between the Gasp! server database and native iOS/Android data stores. This demo uses [Amazon SNS Mobile Push](http://docs.aws.amazon.com/sns/latest/dg/SNSMobilePush.html) services to send push notifications: it supports both Google Cloud Messaging (for Android) and Apple Push Notification (for iOS).


Setup
-----

1. Set up the Gasp! server and database: see [gasp-server](https://github.com/cloudbees/gasp-server) on GitHub

2. Configure a FoxWeave Integration (Sync) App with a pipeline as follows:
   - Source: MySQL 5 (pointing at your CloudBees MySQL Gasp database)
   - SQL Statement: select #id, #comment, #star, #restaurant_id, #user_id from review where id > ##id
   - Target: WebHook
   - Target URL: http://gasp-snsmobile-server.partnerdemo.cloudbees.net/reviews
   - JSON Message Structure:
`
{
    "id":1, 
    "comment":"blank", 
    "star":"three", 
    "restaurant_id":1, 
    "user_id":1
}
`
   - Data Mapping: `id->${id}, comment->${comment}` etc

3. Configure Provisioning Profiles and Certificates for iOS Apple Push Notification Services
   - This [tutorial](http://www.raywenderlich.com/32960/apple-push-notification-services-in-ios-6-tutorial-part-1) explains the steps
   - Create the provisioning profile and certificate using the [iOS Developer Portal](https://developer.apple.com/devcenter/ios/index.action)

4. Configure Google APIs for Google Cloud Messaging
   - Logon to [Google APIs Console](https://code.google.com/apis/console)
   - Services -> Google Cloud Messaging for Android = ON
   - API Access -> Simple API Access -> Key for server apps (note API Key)
   - Overview (note 12-digit Project Number for Android client)

5. Deploy your FoxWeave Integration App on CloudBees and start it

6. Build the app
   - Copy your iOS Push Services certificate (in PEM format) to src/main/webapp/WEB-INF/classes/apnsappcert.pem
   - Copy your iOS Push Services private key (in PEM format) to src/main/webapp/WEB-INF/classes/apnsappkey.pem
   - Copy your AWS credentials properties file to src/main/webapp/WEB-INF/classes/AwsCredentials.properties
   - `mvn build install`
   - (to test locally) `mvn bees:run -DGCM_APIKEY=your_gcm_apikey` and use localhost:8080 for all curl commands

6. Deploy to CloudBees:
   - `bees app:deploy -a gasp-snsmobile-server target/gasp-snsmobile-server.war`
   - `bees config:set -a gasp-snsmobile-server -P GCM_APIKEY=your_gcm_apikey`

7. To test the service:
   - `curl -X POST http://gasp-snsmobile-server.partnerdemo.cloudbees.net/gcm/register -d 'regId=test_gcm-regid'`
   - `curl -X POST http://gasp-snsmobile-server.partnerdemo.cloudbees.net/apn/register -d 'token=test_apn_token'`
   - `curl -H "Content-Type:application/json" -X POST http://gasp-snsmobile-server.partnerdemo.cloudbees.net/reviews -d '{ "id":1, "comment":"blank", "star":"three", "restaurant_id":1, "user_id":1 }'`


Viewing the Server Log
----------------------

You can view the server log using `bees app:tail -a gasp-snsmobile-server` 
