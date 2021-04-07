# Bank in a Box Workshop for Capital Markets : Speaker Notes 

## Installing and Configuring a local Kubernetes cluster

```sh
brew install docker kubectl helm 
```

> You can do this with docker for mac by enabling kubernetes  
> Make sure to set docker for mac's resources for this demo. Ideally 8GB CPU, and 12GB of Memory. 

add kubernetes dashboard to helm
```sh
helm repo add kubernetes-dashboard https://kubernetes.github.io/dashboard/
# deploy kubernetes dashboard 
helm install dashboard kubernetes-dashboard/kubernetes-dashboard -n kubernetes-dashboard --create-namespace


# to proxy traffic to kubernetes cluster and the local browser, run this in a separate terminal: 
kubectl proxy 

# visit the kubernetes dashboard 
http://localhost:8001/api/v1/namespaces/kubernetes-dashboard/services/https:dashboard-kubernetes-dashboard:https/proxy/#/login

# download a kubernetes admin user config file 
# I've written an example one here 
wget https://gist.githubusercontent.com/davidawad/48f7873d843e966cb81f7f6cb4df5466/raw/d7094df6d7c7c65bcb50534025482121bf619a0d/service-account.yaml

# apply the config file to create an admin-user account in our kubernetes cluster 
kubectl apply -f service-account.yaml

# get the token for our dashboard 
kubectl describe serviceaccount admin-user -n kubernetes-dashboard
```

You'll see output like this: 

```
Name:                admin-user
Namespace:           kubernetes-dashboard
Labels:              <none>
Annotations:         <none>
Image pull secrets:  <none>
Mountable secrets:   admin-user-token-bvthf
Tokens:              admin-user-token-bvthf # <-- this one 
Events:              <none>


# Get the string for the token and inject it into this command like so: 
kubectl describe secret admin-user-token-bvthf -n kubernetes-dashboard
```


You'll get output like this: 

```
Data
====
ca.crt:     1066 bytes
namespace:  20 bytes
token:      eyJhbGciOiJSUzI1NiIsImtpZCI6IkJ3X2lwc25SV21rREh2NnFiS1NPZDhUd2ZsZldiR0RrRFZjcEZydXlZVlEifQ.eyJpc3MiOiJrdWJlcm5ldGVzL3NlcnZpY2VhY2NvdW50Iiwia3ViZXJuZXRlcy5pby9zZXJ2aWNlYWNjb3VudC9uYW1lc3BhY2UiOiJrdWJlcm5ldGVzLWRhc2hib2FyZCIsImt1YmVybmV0ZXMuaW8vc2VydmljZWFjY291bnQvc2VjcmV0Lm5hbWUiOiJhZG1pbi11c2VyLXRva2VuLWJ2dGhmIiwia3ViZXJuZXRlcy5pby9zZXJ2aWNlYWNjb3VudC9zZXJ2aWNlLWFjY291bnQubmFtZSI6ImFkbWluLXVzZXIiLCJrdWJlcm5ldGVzLmlvL3NlcnZpY2VhY2NvdW50L3NlcnZpY2UtYWNjb3VudC51aWQiOiI0M2RhZWNhNC05YTBiLTQ5NjYtODhiMy04NDgyM2JmYzA3ZmEiLCJzdWIiOiJzeXN0ZW06c2VydmljZWFjY291bnQ6a3ViZXJuZXRlcy1kYXNoYm9hcmQ6YWRtaW4tdXNlciJ9.LWB1POuxpIRR-6CkJJAgiZsQ-dceHbgCBlzt120StcVdTANnEYoPjjyjRx-y_HGz4FGq2tZisp7NPSlQr6ie2eVAqljZVUztiOC_ISnVwKL6sAS12-6FRdvTMmI8EJyPgF4LTnH9EZtA98inYPA7Cdm4_rFuBzPFdzeuAvKfUAsxKWKWmdGJEIEaYQUcY31_WybGPBT-HObDnro9mQ2c0tKXO-QYl_X32zTz1ke5WO5cWYX4Vt7BbuKcYYRV3ymLfbJUQSoLZmqlX-MmmvxmbOGQvnpYehJ-fQMxrtWMITw59O2z461oOmZipg9sv8cs59ZsCWQOwlO8pBK2lHKA3w

# That Massive token is your token to login to the dashboard! 
```

###### Setting up the cluster for the app to run 

Once we've built the jars and the cordapps, installed and run them we can view the logs. 

```
# run the setup for jars, docker images, and helm chart installs 

# jar files
./gradlew workflows:jar
./gradlew contracts:jar
./gradlew credit-rating-oracle:jar
./gradlew clients:bootJar
./gradlew webservices:bootJar


# docker images
docker build -t bank-in-a-box:0.0.1 -f docker/corda-node/Dockerfile .
docker build -t credit-rating-server:0.0.1 -f docker/credit-rating-server/Dockerfile .
docker build -t web-api-server:0.0.1 -f docker/web-api-server/Dockerfile .
docker build -t front-end:0.0.1 clients/FrontEnd/bank-in-a-box/.


# install helm charts and deploy in kubernetes
cd helm

helm install bank bank-in-a-box -f bank-in-a-box/values-bank.yaml
helm install notary bank-in-a-box -f bank-in-a-box/values-notary.yaml
helm install oracle bank-in-a-box -f bank-in-a-box/values-oracle.yaml
helm install credit-rating-server web-server -f web-server/values-credit-rating.yaml
helm install web-api-server web-server -f web-server/values-web-api.yaml
helm install front-end frontend -f frontend/values.yaml

# Now we can open the bank in a box app in the browser
# in order to get the ip of the browser app just get the service information from kubernetes 
davidawad@boron ~/D/bank-in-a-box>release/bankinabox/1.0>kubectl get svc
NAME                           TYPE           CLUSTER-IP       EXTERNAL-IP   PORT(S)                                          AGE
corda-bank-service             LoadBalancer   10.98.4.97       localhost     40000:31814/TCP,30000:30111/TCP,2223:32637/TCP   6m24s
corda-notary-service           LoadBalancer   10.98.106.245    localhost     40004:31414/TCP,30003:30482/TCP,2224:30169/TCP   6m22s
corda-oracle-service           LoadBalancer   10.107.105.106   localhost     40007:32372/TCP,30005:30275/TCP,2224:32221/TCP   6m19s
credit-rating-server-service   LoadBalancer   10.97.221.68     localhost     8090:30092/TCP                                   6m12s
front-end-service              NodePort       10.106.55.196    <none>        3003:30646/TCP. <-- This one, use port 30646
kubernetes                     ClusterIP      10.96.0.1        <none>        443/TCP                                          5h48m
web-api-server-service         LoadBalancer   10.106.236.38    localhost     7777:32748/TCP


# to view logs 
kubectl logs -f <POD ID>

# to get a shell on the machine 
kubectl exec -it corda-bank-deployment-6cd6bbffd6-tlg4q -- /bin/bash
```

If everything worked you should be able to view the cordapp on http://localhost:30646

## A Demo of the bank app itself! 

Here is our high level workflow of how a bank account is created. 
  - a customer visits the bank
  - the admin creates an account for them with their "passport" (a zip file under 1mb in size that must be unique per customer)
  - the customer then goes home and registers their new account with a login (register with the same customer id and zip attachment)
  - Now that the customer is saved as a guest in the system, the admin can then change their user account from a guest to a customer
  - The bank can then create an account for them and set the account as active
  - That customer can then have the admin deposit money to a their bank account (to reflect a customer deposit in person) 

###### login as the bank admin

> login for admin account: 
```
u: admin
p: password1! 
``` 
  
###### So again your process is this:  

  - login as admin
  - create accounts for two new customers (alice and bob)
  - Save their customer ids in a seperate document 
  - logout of admin 
  - register account as alice with customer id and document
  - register account as bob with customer id and document 
  - (if you want you can login as alice or bob to demo that they can't do anything yet)
  - login to admin and go to the user management screen.
  - Move both alice and bob to customers and not guests
  - Go to their accounts and Mark their account states as active 
  - Deposit some money (meant to mimic them depositing at the bank)
  - Now login as bob or alice and go to the intrabank page to perform a transaction (from one to the other)
  - Start a transaction from one person's account to the other (make sure to use the account key, if you see a vault query error just double check the account key is correct) 
  - Specify for audience that admins can observe these transactions (if you want to login as admin to do so feel free)
  



## for code demo inside intellij

Inside intellij

Open the code, make sure to specify that you have to click 'import gradle project' in the bottom right. 

There is SO MUCH CODE to look at here so I'll specify a couple that you should look at: 

- AccountData (in states)
- CurrentAccountState (in states)
- SavingsAccountState (in states)
- FinancialAccountContract (remember to note these are the things you can do with an account object)
- CreateCustomerFlow (simple flow here)
- CreateCurrentAccountFlow (note that a current account is the uk equivalent of a us checking account)
- OracleSignCreditRatingRequestFlowResponder (it's inside the credit rating oracle: note responder flow signs only part of a transaction which could be an interesting detail to share as it's rare for cordapps to do that) 

### Additional Resources: 
- Refer Users to How to guide on this cordapp in the documentation: https://docs.corda.net/docs/apps/bankinabox/user-interface/how-to.html 
- Link to the cordapp on GitHub https://github.com/corda/bank-in-a-box




#### If you somehow still have time take questions. 



### Issues 

- Make sure docker has 8GB of CPU, and 12GB of memory  
- be careful that the customer attachments are UNIQUE and are UNDER 1MB. 
- Make sure that you create acounts in EURO. OTHER CURRENCIES ARE NOT SUPPORTED
- make sure to write down the correct customer id's when logging in after activating an account












