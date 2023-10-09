# GCC-Lowcoder-Setup

## Creating a VPC Network
1. Create a VPC with the settings that you fulfill your requirements 
2. In the subnet setting turn on Private Google Access (it is necessary to enable this setting otherwise the cloud run services will not be able to communicate with one another)
    - This will be in the Edit subnet section is using a custom subnet creation mode
    - If subnet creation mode is automatic then you will have to go to the region of your VPCnetwork and enable the Private Google Access setting from there once the VPCis created.
3. After creating a VPC Network create a Serverless VPC access


## Creating a Redis Instance
1. Within Google Memorystore, in the redis instance, create a redis instance
2. Name Redis instance in the Instance ID
3. Select the tier, capacity and region for your redis instance 
4. Then connect your redis instance to the VPCnetwork created previously

## Setting up the Node Service
- Tag and Push the docker container to the Google Container Registry
   - https://hub.docker.com/r/lowcoderorg/lowcoder-ce-node-service
### Container Settings
1. Container Image URL: Use the node container image url that was pushed into the container registry
2. Container port: 6060
3. Capacity (allocate as necessary) 
    - Memory: 1 GiB 
    - CPU: 1
4. Execution environment: Default 
5. Environment Variable:
    - Name 1: LOWCODER_API_SERVICE_URL
    - Value 1: Paste the Api Service URL
### Networking Settings
- Connect to a VPC for outbound traffic
    - Use Serverless VPC Access Connectors
       - Network: Select the VPC network that was created 
    - Traffic routing:
      - Route only requests to private IPs to the VPC (this is necessary to not expose the Node Service to the rest of the internet)
### Post Deployment Settings 
- After deploying the revision, in the networking tab, set the ingress control to internal (this is necessary to not expose the Node Service to the rest of the internet)

## Setting up the API Service
- Tag and Push the docker container to the Google Container Registry
  - https://hub.docker.com/r/lowcoderorg/lowcoder-ce-api-service
### Container Settings
1. Container Image URL: Use the api container image url that was pushed into the container registry
2. Container port: 8080
3. Capacity (allocate as necessary) 
    - Memory: 1 GiB 
    - CPU: 1
4. Execution environment: Second generation
5. Environment Variable: add any other environment variable as per your requirement (list of environment variables https://raw.githubusercontent.com/lowcoder-org/lowcoder/main/deploy/docker/docker-compose-multi.yaml)
    - Variable 1: 
      - Name 1: REDIS_URL
      - Value 1: use: redis://10.0.0.0:6379?db=databasename and replace 10.0.0.0 with your redis instance ip address and database name with the name of your database
    - Variable 2: 
      - Name 2: MONGODB_URL
      - Value 2: Paste the MongoDB URL
    - Variable 3: 
      - Name 3: LOWCODER_NODE_SERVICE_URL
      - Value 3: Paste the Node Service URL
    - Variable 4: 
      - Name 4: ENABLE_USER_SIGN_UP
      - Value 4: TRUE
    - Variable 5: 
      - Name 5: ENCRYPTION_PASSWORD
      - Value 5: lowcoder.org
    - Variable 6: 
      - Name 6: ENCRYPTION_SALT
      - Value 6: lowcoder.org
    - Variable 7: 
      - Name 7: CORS_ALLOWED_DOMAINS
      - Value 7: *
### Networking Settings
- Connect to a VPC for outbound traffic
    - Use Serverless VPC Access Connectors
       - Network: Select the VPC network that was created 
    - Traffic routing:
      - Route all traffic to the VPC
### Post Deployment Settings
- After deploying the revision, in the networking tab, set the ingress control to internal (this is necessary to not expose the API Service to the rest of the internet)

## Setting up the Front-End Service 
- Tag and Push the docker container to the Google Container Registry
  - https://hub.docker.com/r/lowcoderorg/lowcoder-ce-frontend
 
### Container Settings 
1. Container Image URL: Use the api container image url that was pushed into the container registry
2. Container port: 3000
3. Capacity (allocate as necessary) 
    - Memory: 512 MiB 
    - CPU: 1
4. Execution environment: Default
5. Environment Variable: add any other environment variable as per your requirement (list of environment variables https://raw.githubusercontent.com/lowcoder-org/lowcoder/main/deploy/docker/docker-compose-multi.yaml)
    - Variable 1: 
      - Name 1: LOWCODER_API_SERVICE_URL
      - Value 1: Paste the Api Service URL
    - Variable 2: 
      - Name 2: LOWCODER_NODE_SERVICE_URL
      - Value 2: Paste the Node Service URL
### Networking Settings
- Connect to a VPC for outbound traffic
    - Use Serverless VPC Access Connectors
       - Network: Select the VPC network that was created 
    - Traffic routing:
      - Route all traffic to the VPC
### Post Deployment Settings
- After deploying the revision, in the networking tab, set the ingress control to all (the front-end should be exposed to the internet)

