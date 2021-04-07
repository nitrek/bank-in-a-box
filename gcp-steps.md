https://github.com/corda/bank-in-a-box


# jar files
./gradlew workflows:jar
./gradlew contracts:jar
./gradlew credit-rating-oracle:jar
./gradlew clients:bootJar
./gradlew webservices:bootJar

# updload jar to gsutil
gsutil cp workflows/build/libs/bank-in-a-box-workflows-0.1.jar gs://ownera-tokenization/
gsutil cp contracts/build/libs/bank-in-a-box-contracts-0.1.jar gs://ownera-tokenization/
gsutil cp credit-rating-oracle/build/libs/credit-rating-oracle-0.1.jar gs://ownera-tokenization/
gsutil cp webservices/build/libs/webservices-0.1.jar gs://ownera-tokenization/
gsutil cp clients/build/libs/clients-0.1.jar gs://ownera-tokenization/
# docker images

gcloud builds submit --tag gcr.io/digitalvault-innovation/bank-in-a-box:0.0.1 docker/corda-node/
gcloud builds submit --tag gcr.io/digitalvault-innovation/credit-rating-server:0.0.1  docker/credit-rating-server/
gcloud builds submit --tag gcr.io/digitalvault-innovation/web-api-server:0.0.1  docker/web-api-server/
gcloud builds submit --tag gcr.io/digitalvault-innovation/front-end:0.0.1 clients/FrontEnd/bank-in-a-box/