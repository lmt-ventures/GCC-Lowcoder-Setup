# Google Cloud Console-Lowcoder-Setup

## Creating a VPC Network
1. In the Google Cloud Console, in VPC Network, click on VPC Networks
    - Enable Compute Engine API
3. Create a VPC Network
    - Name the VPC Network
    - Disable IPv6
    - Create a custom subnet 
        - Name the subnet 
        - Choose the region best for your
        - Choose a IP Range
        - Private Google Access: On
        - Flow Logs: Off
        - In the subnet setting turn on Private Google Access (it is necessary to enable this setting otherwise the cloud run services will not be able to communicate with one another)
            - This will be in the Edit subnet section is using a custom subnet creation mode
            - If subnet creation mode is automatic then you will have to go to the region of your VPCnetwork and enable the Private Google Access setting from there once the VPCis created.
4. In the VPC Network, create a Connector
    - Enable serverless vpc access API
6. Create a Serverless VPC Connector
    - Name the Serverless VPC Connector
    - Choose the same region as the VPC Network
    - Network: Select the VPC Network that was previously created
    - Subnet:
        - Custom IP range
        - IP range (cannot be the same subnet that was created before) 

## Creating a Redis Instance
1. Within Google Cloud Console in Memorystore, in the redis page, create a redis instance
2. Enable Google Cloud Memorystore for Redis API
3. Create a Redis Instance:
    - Name Redis instance in the Instance ID
    - Basic Tier
    - 1 GB of capacity is more than sufficient in our environment however if you face trouble then you can always adjust the capacity as necessary
    - Choose the same region as the VPC Network
    - Then connect your redis instance to the VPC network created previously


## Setting up the Node Service
1. In the Google Console go to Cloud Run
2. Create a Service 
3. In the Container Image URL input: lowcoderorg/lowcoder-ce-node-service
    - This is from the lowcoder docker hub (https://hub.docker.com/r/lowcoderorg/lowcoder-ce-node-service)
4. Select the Region, same as the region for the VPC Network
5. CPU allocation and pricing: CPU is only allocated during request processing
6. Ingress Control is set to Internal (this is necessary to not expose the Node Service to the rest of the internet)
7. Authentication: Allow unauthenticated access invocations

### Container Settings
1. Container port: 6060
2. CPU allocation:
    - Only allocated during the request processing
3. Capacity (allocate as necessary) 
    - Memory: 1 GiB 
    - CPU: 1
4. Execution environment: Default 
5. Environment Variable:
    - Name 1: LOWCODER_API_SERVICE_URL
    - Value 1: Paste the Api Service URL (Once the API Service is created after the next step

### Networking Settings
- Connect to a VPC for outbound traffic
    - Use Serverless VPC Access Connectors
       - Network: Select the VPC network that was created 
    - Traffic routing:
      - Route only requests to private IPs to the VPC (this is necessary to not expose the Node Service to the rest of the internet)


## Setting up the API Service
1. In the Google Console go to Cloud Run
2. Create a Service 
3. In the Container Image URL input: lowcoderorg/lowcoder-ce-api-service
    - This is from the lowcoder docker hub (https://hub.docker.com/r/lowcoderorg/lowcoder-ce-api-service)
4. Select the Region, same as the region for the VPC Network
5. CPU allocation and pricing: CPU is only allocated during request processing
6. Ingress Control is set to Internal (this is necessary to not expose the API Service to the rest of the internet)
7. Authentication: Allow unauthenticated access invocations

### Container Settings
1. Container port: 8080
2. Capacity (allocate as necessary) 
    - Memory: 1 GiB 
    - CPU: 1
3. Execution environment: Second generation
4. Environment Variable: add any other environment variable as per your requirement (list of environment variables https://raw.githubusercontent.com/lowcoder-org/lowcoder/main/deploy/docker/docker-compose-multi.yaml)
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
            - If it is a new set up then set it to true, otherwise if it will be used for a existing set up then set it to FALSE
    - Variable 5: 
      - Name 5: ENCRYPTION_PASSWORD
      - Value 5: lowcoder.org
    - Variable 6: 
      - Name 6: ENCRYPTION_SALT
      - Value 6: lowcoder.org
    - Variable 7: 
      - Name 7: CORS_ALLOWED_DOMAINS
      - Value 7: *
![API Service Container Settings.JPG](https://github.com/lmt-ventures/GCC-Lowcoder-Setup/blob/a0c34bf799035819f04f48ef884f4200c973e3e9/API%20Service%20Container%20Settings.JPG)
### Networking Settings
- Connect to a VPC for outbound traffic
    - Use Serverless VPC Access Connectors
       - Network: Select the VPC network that was created 
    - Traffic routing:
      - Route all traffic to the VPC


## Setting up the Front-End Service 
1. In the Google Console go to Cloud Run
2. Create a Service 
3. In the Container Image URL input: lowcoderorg/lowcoder-ce-frontend
    - This is from the lowcoder docker hub (https://hub.docker.com/r/lowcoderorg/lowcoder-ce-frontend)
4. Select the Region, same as the region for the VPC Network
5. CPU allocation and pricing: CPU is only allocated during request processing
6. Ingress Control is set to all (the front-end should be exposed to the internet)
7. Authentication: Allow unauthenticated access invocations

### Container Settings 
1. Container port: 3000
2. Capacity (allocate as necessary) 
    - Memory: 512 MiB 
    - CPU: 1
3. Execution environment: Default
4. Environment Variable: add any other environment variable as per your requirement (list of environment variables https://raw.githubusercontent.com/lowcoder-org/lowcoder/main/deploy/docker/docker-compose-multi.yaml)
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
- Can set up a DNS through Google Domains 

