#### Initialize gcloud config 
````
gcloud init
````

#### Create K8s cluster
````
gcloud container clusters create poc-keycloak-cluster --num-nodes=1 --machine-type=e2-small
````

#### Connect to cluster
````
gcloud container clusters get-credentials poc-keycloak-cluster --zone us-central1-a
````

#### Deploy Keycloak
````
kubectl create -f ./keycloak.yaml
````

#### Deploy Hello App
````
kubectl create -f ./hello-app.yaml
````

#### Access Keycloak admin console: http://<IP>:8080/auth and create justice-league realm with justice-league.json file (users must be created manually)

````
Justice League realm structure:

- realm: justice-league
  client: service-gatekeeper
  roles:
  - edit (will be able to access isprime function)
  - view (won't be authorized to access isprime function)
  groups:
    - privileged (is associated to service-gatekeeper's edit role)
    - unprivileged (is associated to service-gatekeeper's view role)
  users:
    - superman (belongs to privileged group)
    - clarkkent (belongs to unprivileged group)
````

#### Edit gatekeeper-config.yaml:
````
- update discovery-url IP with keycloak cluster external IP
- update upstream-url IP with hello-app cluster internal IP
- update client-secret with service-gatekeeper client secret
````

#### Deploy Gatekeeper config map
````
kubectl create -f ./gatekeeper-config.yaml
````

#### Deploy Gatekeeper
````
kubectl create -f ./gatekeeper-deployment.yaml
````

#### Generate token 
````
POST x-www-form-urlencoded
client_id = service-gatekeeper
username = <username>
password = <password>
grant_type = password
client_secret = <client_secret>

http://<keycloak_IP>:8080/auth/realms/justice-league/protocol/openid-connect/token
````

#### Test API with
````
GET or POST
Authorization: Bearer <token>
http://<gatekeeper_IP>:3000