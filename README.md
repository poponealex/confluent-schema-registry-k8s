# Deploy a Confluent Schema Registry with mTLS authentication to the Kafka cluster

1. Create a `KafkaUser` named `registry-user`
```
kubectl -n kafka create -f registry-user.yml
```
2. Download the Kafka cluster's CA Cert and the `registry-user`'s PKCS12 keys
```
kubectl -n kafka get secret kafka-cluster-ca-cert -o jsonpath="{.data.ca\.crt}" | base64 -d > ca.crt 
 
kubectl -n kafka get secret kafka-cluster-ca-cert -o jsonpath="{.data.ca\.password}" | base64 -d > truststore_password
 
kubectl -n kafka get secret registry-user -o jsonpath="{.data.user\.password}" | base64 -d > keystore_password
 
kubectl -n kafka get secret registry-user -o jsonpath="{.data.user\.p12}" | base64 -d > user.p12
```

3. Convert the `ca.cert` to `truststore.jks` and `user.p12` to `keystore.jks` (use the passwords fetched in the previous step at each password prompt)
```
keytool -keystore truststore.jks -alias CARoot -import -file ca.crt
keytool -importkeystore -srckeystore user.p12 -srcstoretype pkcs12 -destkeystore keystore.jks -deststoretype jks
```
 
4. Create a kafka-registry-creds secret
```
kubectl -n kafka create secret generic kafka-registry-creds --from-file=truststore_password --from-file=keystore_password --from-file=keystore\.jks --from-file=truststore\.jks
```
5. Update the namespace and update kafka ports in the `deploy.yml` file 
7. Deploy the registry
```
kubectl -n kafka create -f deploy.yml
```


