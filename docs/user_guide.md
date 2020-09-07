# User's Guide

## How to install

The full installation process relies on docker-compose.
The process of the installation is split in three different steps:

* Configuration : You must adapt the docker-compose-*.yaml file and feed the different secrets value
* Build : it is the docker-compose build process, a helper script ./build.sh is provided
* Installation : it is the docker-compose up process, a helper script is provided

Installation of Security Layer	

```
cd DataAcquisitionLayer
./build.sh 
./install.sh
```

## Configuration
Platform specifics configurations are done using the docker-compose-*.yaml file and with the configuration of the secret files for each component:

### Environment variables

* Wilma

| Variable | Description |
| --- | --- |
| PEP_PROXY_AZF_PORT|	The port use to contact AuthZForce|
|PEP_PROXY_AZF_HOST|	The host that exposed AuthZForce|
|PEP_PROXY_APP_PORT | The port use to contact the backend protected by Wilma (DAL-Proxy for example)|
| PEP_PROXY_APP_HOST | The host that exposed the backend protected by Wilma (DAL-Proxy for example)|
|PEP_PROXY_APP_ID	|The APP ID created in Keyrock for this PEP Proxy|
|PEP_PROXY_IDM_PORT	|The port use to contact KeyRock|
|PEP_PROXY_IDM_HOST	|The host that exposed KeyRock|
|PEP_PROXY_PORT	|The port Wilma listen to|

* KeyRock

| Variable | Description |
| --- | --- |
| IDM_DB_HOST	|The address of the MySQL Database|
|IDM_DB_USER|	The MySQL user to connect the database|
|IDM_HOST|	The URL to contact KeyRock|
|IDM_PORT	|The port KeyRock listen to|
|IDM_AUTHZFORCE_HOST|	The host to contact AuthZForce|



### Secrets

* Wilma

| Secret | Description |
| --- | --- |
| sec_wilma_pub.token.secret|	A token use for encryption (random)|
|sec_wilma_pub.password|The password for the PEP Proxy created with the application in KeyRock
|sec_wilma_pub.proxy.username|The id of the PEP Proxy created with the application in KeyRock|

* MySQL

| Secret | Description |
| --- | --- |
| idm.db.pass	|The MySQL root password (random)|

* KeyRock

| Secret | Description |
| --- | --- |
| idm.db.pass	|The MySQL root password (random)|
|idm.admin.pass|The password  for the admin user of KeyRock (random)|
|idm.session.secret|A token use for encryption (random)|


## Component status

How do you verify the service has been correctly deployed?	

* MySQL
Executing ```docker-compose ps``` to check that the service is in "Up" state , and with the command ```telnet IP 3306``` that the TCP port is listening

* KeyRock
With the command ```docker-compose ps``` you check that the service is in "Up" status, and check you can connect to KeyRock with the admin/password

* Wilma
With the command ```docker-compose ps``` you check that the service is in "Up" status.

* AuthZForce
With the command ```docker-compose ps``` you check that the service is in "Up" status.

## PIXEL specifics deployment

When you have deployed Keyrock and Authzforce, you can deploy as many Wilma that you need. Wilma is just a simple HTTP Proxy that you install in the middle of the API flow to check the requester provide a valid token and is allow to access the resources.

By default we use Wilma to protect the NGSI Agents that is configured as daemon, and we will also use it to protect the access to private API from the outside of the platform.

<p align="center">
<img src="https://github.com/pixel-ports/docs-hub-security/raw/master/docs/img/security_wilma.png" alt="PIXEL Security Layer architecture" align="center" />
</p>

To install other WILMA, you have to create the corresponding Application in KeyRock using the UI or the API

Then retrieve the application id, pep_proxy id and password and then set those parameters in the docker compose file.

## Issues & Solution

Most of the problems with FIWARE Security modules are the provisioning of the application parameters on Wilma. A simple test could allow to control everything works as expected.

### Authorization Basic	

We authenticate against an application, we need 2 information for that application
* Client id
* Client secret

Then we can combine them to create the Authorization Basic token:
```base64(client_id:client_secret)```

### Access Token Request
```
POST /oauth2/token HTTP/1.1
Host: id.<pilot>.port-pixel.eu
Authorization: Basic <authorization basic token>
Content-Type: application/x-www-form-urlencoded

grant_type=password&username=<user email>&password=<user password>
```

user email and password have to be URL Encoded, the token is valid 1 hour. To refresh it you can authenticate again, or use the refresh token.

### Access Token Response
```
HTTP/1.1 200 OK
Content-Type: application/json;charset=UTF-8
Cache-Control: no-store
Pragma: no-cache

{
    "access_token":"2YotnFZFEjr1zCsicMWpAA",
    "token_type":"bearer",
    "expires_in":3600,
    "refresh_token":"tGzv3JOkF0XG5Qx2TlKWIA",
}
```

### Token Verification
```
curl https://id.<pilot>.port-pixel.eu/user?access_token=<access_token>
    {
      "organizations": [
        {
          "id": "13e88767-7473-472d-9c33-110c5bed2a57",
          "name": "test_org",
          "description": "my org",
          "website": null,
          "roles": [
            {
              "id": "9c4e8db4-a56b-4731-bfc6-7dd8fb2fbea3",
              "name": "test"
            }
          ]
        }
      ],
      "displayName": "My User",
      "roles": [
        {
          "id": "9c4e8db4-a56b-4731-bfc6-7dd8fb2fbea3",
          "name": "test"
        }
      ],
      "app_id": "ff03921a-a772-4220-9854-e2d499ae474a",
      "isGravatarEnabled": false,
      "email": "myuser@test.com",
      "id": "myuser",
      "authorization_decision": "",
      "app_azf_domain": "",
      "username": "myuser"
    }
```

### Request A Resource Access
For example to access ```https://dal.<pilot>.pixel-ports.eu/orion/version```
```
curl –H “X-Auth-Token: <access_token>” https://dal.<pilot>.pixel-ports.eu/orion/version
```

An HTTP 200 OK should be received, otherwise check the components logs to investigate


Refer to the official documentation for [KeyRock](https://fiware-idm.readthedocs.io/en/latest/oauth/oauth_documentation/index.html), [Wilma](https://fiware-pep-proxy.readthedocs.io/en/latest/) and [AuthZForce](https://authzforce-ce-fiware.readthedocs.io/en/latest/).


