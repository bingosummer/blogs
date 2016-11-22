# A Demo to Deploy a Microservice Application in Pivotal Cloud Foundry on Azure

The demo was created by [Yoshio Terada](https://github.com/yoshioterada). Here are the [presentation](https://docs.com/terada-yoshio/4173/pivotal-clound-foundry-on-azure) and [demo video](https://www.youtube.com/watch?v=BiY3amrDIo0).

I follow the steps to create the demo. Here are the details.

## 1. Install Pivotal Ops Manager and Spring Cloud Service

1. Install [Pivotal Ops Manager on Azure](https://docs.pivotal.io/pivotalcf/1-8/customizing/azure.html).

1. Install [MySQL](http://docs.pivotal.io/p-mysql/1-8/index.html), [RabbitMQ](http://docs.pivotal.io/rabbitmq-cf/1-7/index.html) and [Spring Cloud Service](https://docs.pivotal.io/spring-cloud-services/1-3/installation.html) on Ops Manager.

## 2. Create the services which the demo APP will consume

1. [Service Registry](https://docs.pivotal.io/spring-cloud-services/1-3/service-registry/creating-an-instance.html)

1. [Circuit Breaker](https://docs.pivotal.io/spring-cloud-services/1-3/circuit-breaker/creating-an-instance.html)

1. storage-service

  1. Create the storage account on Azure manually.

  1. Get `ACCOUNT_NAME` and `ACCOUNT_KEY` from Azure Portal or CLI.
  
  1. Create the user-provided service (cf cups) with the storage account names and keys.

  ```
  cf cups storage-service -p '{"accountName":"AccountName=ACCOUNT_NAME;", "accountKey": "AccountKey=ACCOUNT_KEY"}'
  ```
  
1. facedetect-service

  1. Register facedetect APIs from https://www.microsoft.com/cognitive-services/en-us/sign-up.

  1. Get the `API_KEY` from your account.

  1. Create the user-provided service (cf cups) with the facedetect api keys.

  ```
  cf cups facedetect-service -p '{"subscriptionId": "API_KEY"}'
  ```

1. emotional-service

  1. Register emotional APIs from https://www.microsoft.com/cognitive-services/en-us/sign-up.

  1. Get the `API_KEY` from your account.

  1. Create the user-provided service (cf cups) with the emotional api keys.

  ```
  cf cups emotional-service -p '{"subscriptionId": "API_KEY"}'
  ```

## 3. Deploy the microservice app to the Pivotal Cloud Foundry

1. Migrate one monolithic Java EE application to three Microservice.

  * Monolith: https://github.com/yoshioterada/Face-Detect-Cognitive-Service-with-Java-EE

  * Microservice: https://github.com/FaceDetectServices

1. Install Maven.

  ```
  sudo apt-get install maven default-jdk
  ```

1. Deploy three microservice apps.

  * `EmotionalService`

    ```
    git clone https://github.com/FaceDetectServices/EmotionalService
    cd EmotionalService
    mvn package
    ```

    Change the [manifest](https://github.com/FaceDetectServices/EmotionalService/blob/master/manifest.yml#L10) to your own domain name, and `cf push`.


  * `FaceDetectService`

    ```
    git clone https://github.com/FaceDetectServices/FaceDetectService
    cd FaceDetectService
    mvn package
    ```

    Change the [manifest](https://github.com/FaceDetectServices/FaceDetectService/blob/master/manifest.yml#L10) to your own domain name, and `cf push`.

  * `FaceDetectUI`

    ```
    git clone https://github.com/FaceDetectServices/FaceDetectUI
    cd FaceDetectUI
    mvn package -Dmaven.test.skip=true
    ```

    1. Change the [storage account name](https://github.com/FaceDetectServices/FaceDetectUI/blob/master/src/main/java/com/yoshio3/backingbean/PhotoUploader.java#L45-L46) to your own storage account which is created in the previous steps.
    
    1. Change the [manifest](https://github.com/FaceDetectServices/FaceDetectUI/blob/master/manifest.yml#L11) to your own domain name, and `cf push`.
    

## 4. Confirmation

1. Confirm the behavior of the application (Cognitive Services).

1. Confirm the Service Registry and Circuit Breaker dash board.

1. Execute the Scale out/in.

1. Confirm the availability (after dead the existing service, auto restart the instance).

1. Execute the Circuit Breaker (Confirm on dash board and execute the app and get the alternative value when breaker is owned).
