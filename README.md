# StreamingApp

Stream premium video content, host live watch parties, and manage your catalogue with a modern microservice architecture. The platform now ships with a production-ready admin portal, real-time chat, S3-backed adaptive streaming, and a redesigned cinematic frontend experience.

==================================================================== 

## Architecture 


StreamingApp — Overall Project Architecture:


<img width="352" height="332" alt="Architecture-1" src="https://github.com/user-attachments/assets/327ce3ad-3c2b-41f0-bad7-1b9f82cb099b" /> 

<img width="374" height="340" alt="Architecture-2" src="https://github.com/user-attachments/assets/cefeda49-0618-47b3-819a-c9ae18c50f4c" />


The StreamingApp is a MERN-based application consisting of five application services:



| Service | Port | Description |
| --- | --- | --- |
| `authService` | 3001 | User authentication, registration, JWT issuance |
| `streamingService` | 3002 | Video catalogue, S3 playback endpoints, public APIs |
| `adminService` | 3003 | Dedicated admin microservice for asset management and uploads |
| `chatService` | 3004 | Websocket + REST chat for live watch parties |
| `frontend` | 3000 | React SPA with revamped UI and integrated chat |
| `mongo` | 27017 | Shared MongoDB instance |

All backend services share common database models and utilities through `backend/common`. 

Project Summary: 
We take a multi-service StreamingApp, package each service into Docker containers, automatically build and store those images through Jenkins and ECR, deploy and manage them on AWS EKS using Kubernetes and Helm, expose them through Ingress, scale and update them safely, and monitor the running system with CloudWatch.

====================================  

Phase 1
====================================

Step 1.1 — Fork the repository


Step 1.2 — Clone your fork : 

Run: git clone <your-repository-url> 

<img width="582" height="63" alt="image" src="https://github.com/user-attachments/assets/0b6e4aec-6cb7-49c6-952c-e183f3f7ac35" />


GitHub stores the project remotely. The folders would be created as shown below:

<img width="209" height="380" alt="image" src="https://github.com/user-attachments/assets/f58d9bda-5002-4a42-bfc8-79157eba8cdc" /> 


================================= 

Phase 2 — Docker Containerization
================================ 

Convert each of the 5 application components into a Docker image that can run independently.

Step 1 — Repository structure
Run: dir

<img width="545" height="208" alt="image" src="https://github.com/user-attachments/assets/f467fae7-3581-4927-abd1-a002e835209e" />

Step 2 - Check Backend Services
Run: dir .\backend

<img width="428" height="182" alt="image" src="https://github.com/user-attachments/assets/291bee0a-f58c-405a-9d78-75f25427f065" /> 
<img width="234" height="104" alt="image" src="https://github.com/user-attachments/assets/e79ac7a8-205c-4e54-8bae-204a45472c9c" />

Inside backend:

<img width="184" height="58" alt="image" src="https://github.com/user-attachments/assets/c7bb6844-2fba-4df4-adfe-71da00a4f99d" />

So we have: 1 frontend + 4 backend microservices + MongoDB

============================ 

Step 3 — Check Dockerfiles and package.json

Checking for frontend:
Run: 
Get-ChildItem .\backend -Recurse -File |
Where-Object { $_.Name -eq "Dockerfile" -or $_.Name -eq "package.json" } |
Select-Object FullName


<img width="635" height="235" alt="image" src="https://github.com/user-attachments/assets/d3febb50-bb14-4231-8f75-c1109fa20b35" />

Step 4 - Checking for backend:

Run:
Get-ChildItem .\frontend -Recurse -File |
Where-Object { $_.Name -eq "Dockerfile" -or $_.Name -eq "package.json" } |
Select-Object FullName

<img width="482" height="167" alt="image" src="https://github.com/user-attachments/assets/736c98a2-3ab0-4f04-a871-f4adef6aa831" />

 Step 5 - Read the Frontend Dockerfile
displays the contents of the existing Dockerfile.
Run:
Get-Content .\frontend\Dockerfile

<img width="461" height="426" alt="image" src="https://github.com/user-attachments/assets/137f4f82-7519-4582-8b68-1b61a59fcb3e" /> 

================ 

Step 6 - Check frontend package.json

Run:  Get-Content .\frontend\package.json

<img width="614" height="395" alt="image" src="https://github.com/user-attachments/assets/1d2ba4be-803a-4381-af64-7c21d9bbf6fd" /> 

<img width="398" height="245" alt="image" src="https://github.com/user-attachments/assets/39d4c604-e616-4225-8ac9-280e958b08d7" />



Now we have both the frontend Dockerfile and the frontend package.json  

===================  


step 7 - Check .env.example, as We want to see the project's existing environment-variable definitions.

 Run: Get-Content .\.env.example

 <img width="438" height="284" alt="image" src="https://github.com/user-attachments/assets/590199b3-0a56-4616-aabf-be199f90b0d4" /> 

 ======================= 

 Step 8 - Understand the Environment Configuration:

   . file contains four groups: 
   Shared
   AWS
   Services
   Frontend build-time values 

 1. Shared configuration has the below client URLs:
     CLIENT_URLS=http://localhost:3000
     JWT_SECRET=changeme
     MONGO_DB=streamingapp 

   2. CLIENT_URLS=http://localhost:3000 :
      tells the backend that the frontend is currently available at: http://localhost:3000

   3. So the relationship is:

         Browser
            ↓
      localhost:3000
            ↓
         Frontend

   4. JWT_SECRET: JWT_SECRET= changeme :
       JWT is commonly used for authentication.
       The backend uses the secret to sign/verify authentication tokens.

  5. MONGO_DB: MONGO_DB=streamingapp, the MongoDB database name.
  
  6. it can be actually seen as below:

     Backend services
             ↓
         MongoDB
             ↓
      streamingapp database
     -----------------------------
  ----------------------------

   2. AWS configuration
         AWS S3: is likely involved in storing streaming-related objects/content.

--------------------------------- 
---------------------------------
   3. Backend service ports:
     AUTH_PORT=3001
     STREAMING_PORT=3002

------------------------------------
------------------------------------ 

4. Frontend build-time values:

   So currently the frontend is configured to communicate like as below:

Frontend
   │
   ├── Auth API
   │      http://localhost:3001/api
   │
   ├── Streaming API
   │      http://localhost:3002/api
   │
   └── Streaming public URL
          http://localhost:3002 

========================================= 
========================================= 

Step 9: Backend Dockerfiles: 

   Run: Get-Content .\backend\adminService\Dockerfile

   <img width="461" height="187" alt="image" src="https://github.com/user-attachments/assets/bf8e748c-3a30-439e-b8cf-8307d21c3973" /> 

   this has:
   package.json
     ↓
   npm install --production
     ↓
   node_modules

----------------------- 

   Overall flow is:
   Docker container starts
       ↓
   npm run start
       ↓
   Admin Service
       ↓
   Port 3003 

   ======================= 

   Inspecting the Second Backend Service:

   Run: Get-Content .\backend\authService\Dockerfile 

   
   <img width="673" height="206" alt="image" src="https://github.com/user-attachments/assets/98339174-f5e8-49da-8306-cc8da4a906e1" />



   The Auth service is a Node.js application, so we start with Node.js 18. :

   Node.js 18
     +
 Alpine Linux
     ↓
 Base container 
 ---------------- 
 
 the Auth service uses: 3001

 ------------------------------- 


Admin vs Auth
Now we can compare the two:

 <img width="382" height="179" alt="image" src="https://github.com/user-attachments/assets/8a962135-1da9-41cc-8eee-0e6db5850f99" />


===================== 
================== 

chatService:

Run: Get-Content .\backend\chatService\Dockerfile 

<img width="468" height="205" alt="image" src="https://github.com/user-attachments/assets/c8490ad9-d261-4fca-a4d8-7fd1d068710c" />

. The Chat service uses Node.js. 


=====================================

. the container starts and the flow would be as below:

Container
   ↓
 npm run start
   ↓
 Chat Service
   ↓
 Port 3004

========================= 

What we know about the backend now:

<img width="392" height="196" alt="image" src="https://github.com/user-attachments/assets/9e64d37c-0fa7-40c3-a649-2eb6dba38f3f" />


================= 
Step 10 -  Last backend Dockerfile

Run: Get-Content .\backend\streamingService\Dockerfile 

<img width="569" height="193" alt="image" src="https://github.com/user-attachments/assets/833f3b63-b5d2-4f51-97c3-a56c1dc628e6" /> 

. The Streaming service is a Node.js application. 
. Streaming port: 3002 

. When the container starts:

 Container
    ↓
 npm run start
    ↓
 Streaming Service
    ↓
 Port 3002

 
======================================== 


. All five Dockerfiles are now understood

We have:

<img width="296" height="133" alt="image" src="https://github.com/user-attachments/assets/feafd99e-6f71-45d2-b286-1b433f8be94d" />

--------------------------------- 

<img width="414" height="167" alt="image" src="https://github.com/user-attachments/assets/ac5f8c6e-8210-4a49-ab89-3543475d1985" /> 

. MangoDB is provided separately through mongo:6 

<img width="257" height="194" alt="image" src="https://github.com/user-attachments/assets/5f1e3061-2747-4879-88de-ebf32a0f264f" /> 

=============================== 

Step 10: Inspect docker-compose.yml

Run: Get-Content .\docker-compose.yml 

The architecture is: 


<img width="351" height="188" alt="image" src="https://github.com/user-attachments/assets/de9bd0ba-a1d9-45b2-b82a-2c3728d6c3fb" /> 

MongoDB works: Docker doesn't build MongoDB from your source code instead it uses mongo:6 from the MongoDB image repository.

<img width="133" height="84" alt="image" src="https://github.com/user-attachments/assets/80e373d3-5510-484b-9ac6-01179f3cdc90" /> 

. So MongoDB is accessible on port 27017. 
. MongoDB stores its database files inside: /data/db 
. The Compose volume: mongo-data, stores that data outside the temporary container filesystem.
. the MongoDB container is recreated, the database data can persist in the volume. 
. Conceptually:

MongoDB container
       │
       ▼
 /data/db
       │
       ▼
 mongo-data volume 

 =============================== 

 . Auth depends on MongoDB 
 . Inside Docker Compose, mongo is the service name.
 . So Auth communicates with MongoDB using: mongodb://mongo:27017/streamingapp
 . http://localhost:3001: can reach the Auth container.
 . backend
    └── streamingService
          ├── package.json
          └── Dockerfile

 . Streaming port:
 Host → Container
 3002 → 3002

 . Admin
    ↓
  localhost:3003

============================================= 

Step 11 - Validate the Compose configuration:

Run: docker compose config

<img width="653" height="370" alt="image" src="https://github.com/user-attachments/assets/41dad46a-709e-4ac5-982d-75f24690ba21" />

Check Exiting Docker images:

<img width="656" height="175" alt="image" src="https://github.com/user-attachments/assets/9e53ddc7-831d-4b76-a43b-b71bc5168a61" />


Step 11: Test the Frontend

Run: docker compose ps

<img width="640" height="255" alt="image" src="https://github.com/user-attachments/assets/2b4f81a8-6956-4b18-beb8-67f683a23f0c" />




This takes the docker-compose.yml, resolves the variables/defaults, and shows Docker the configuration it will actually use.

Your output confirms that Docker understands all 6 services


===================== 

Step 12 - Start the application:

Run: docker compose up -d

<img width="652" height="151" alt="image" src="https://github.com/user-attachments/assets/e13c1931-d32a-4541-b2d3-269b644953b0" />

. Verify the containers 

Run: docker compose ps


<img width="640" height="255" alt="image" src="https://github.com/user-attachments/assets/43f5e147-4a32-4186-bdf3-603c3029b94f" />


 ============================= 

 Open your browser and enter:

http://localhost:3000 , You should get the StreamingApp frontend.

<img width="1920" height="1080" alt="image" src="https://github.com/user-attachments/assets/cdcab8fa-1387-4589-8614-02711877ed9d" />


Note: Your Compose file contains:
 
ports: 
    - "3000:80"

. So, 3000 is the port you use in your browser, while Nginx listens on port 80 inside the container. 

. It works as below:

Your Windows browser
        ↓
localhost:3000
        ↓
Frontend Docker container
        ↓
Nginx :80
        ↓
React application 

================================== 
================================ 

Phase 2 – Frontend Running Successfully
**************************************** 


Step 13 - Verify the Backend:

Step 13A: Find the Auth Service files 

Run: Get-ChildItem .\backend\authService -File | Select-Object Name 



<img width="557" height="196" alt="image" src="https://github.com/user-attachments/assets/95139763-bd5d-4987-85bf-e02c683e6d25" />

Step 13B: Inspect the Auth routes 

Run: Get-Content .\backend\authService\index.js, to see which routes the Auth Service actually provides


<img width="611" height="410" alt="image" src="https://github.com/user-attachments/assets/1bcf7da1-5fbb-481f-83e1-314966b490e2" /> 

Step 13C: Test Auth Service health

Open this in your browser: http://localhost:3001/health

<img width="441" height="228" alt="image" src="https://github.com/user-attachments/assets/b4fd90b1-b1df-4810-bb20-d0cbcda094bb" />  

. What this tests is:
Browser
   ↓
localhost:3001
   ↓
Auth Docker container
   ↓
Express
   ↓
/health route

============ 
============ 

. This confirms:

Browser
   ↓
localhost:3001
   ↓
Auth Docker container
   ↓
Express application
   ↓
/health
   ↓
{"status":"OK"} 

======================================== 
================================== 

Phase 2 — Step 13: Test Streaming Service

Now we'll check the second backend service.

Open: http://localhost:3002 

<img width="221" height="107" alt="image" src="https://github.com/user-attachments/assets/e9ed35bb-8e02-4ff5-931a-bd0414c82efe" />

That means the Streaming Service is reachable, but it doesn't define a route for /.

Therefore, Streaming Service is reachable. 

. Checking Next: Admin Service 

Open: http://localhost:3003

<img width="208" height="107" alt="image" src="https://github.com/user-attachments/assets/0e3d9346-bed8-4ab0-a20c-94dc6ad17922" />

Again, this means the server responded but does not define a root / route.  

. Next — Chat Service

Open: http://localhost:3004 

<img width="276" height="124" alt="image" src="https://github.com/user-attachments/assets/40af8c81-934d-479a-9536-a6153e1bc6ec" /> 

Chat Service is also reachable. 

==================================== 

Backend connectivity complete
****************************** 

=================================== 

The containerized application is successfully running as a multi-service application:

<img width="297" height="98" alt="image" src="https://github.com/user-attachments/assets/90c181e1-a3a6-4588-bbf9-50a9f59d23de" />


 =================================== 

 Step 14 - Verify the service logs

 Run: docker compose logs --tail=30 auth 
 

 <img width="651" height="398" alt="image" src="https://github.com/user-attachments/assets/852a2766-b83d-4118-8cfe-9ea0dd67209e" />

Note: The mongo hostname is important. Inside Docker Compose, the Auth container can reach the MongoDB container using the Compose service name mongo. 

========================== 

Step 15B — Check Streaming Service

Now we'll verify MongoDB connectivity for the Streaming Service as well.

Run: docker compose logs --tail=30 streaming

<img width="651" height="375" alt="image" src="https://github.com/user-attachments/assets/d0f322f5-1461-462c-9737-802a451f333c" /> 

Streaming Service is also successfully connected to MongoDB. 

. Streaming Service :3002
        │
        │ mongodb://mongo:27017/streamingapp
        ↓
    MongoDB :27017
        ↓
     Connected 

================================== 


  Step 14C: Admin logs 

  Run: docker compose logs --tail=30 admin 


  <img width="647" height="413" alt="image" src="https://github.com/user-attachments/assets/bba828f4-ecfc-4899-8122-ca8e5e54ac3d" /> 

  <img width="645" height="319" alt="image" src="https://github.com/user-attachments/assets/61c621fb-081f-4e3f-ba6b-ec610b110471" />


Admin Service is also working correctly and connected to MongoDB. 

. Admin Service :3003
       │
       │ mongodb://mongo:27017/streamingapp
       ↓
   MongoDB :27017
       ↓
   Connected 

================================= 

Step 15D: Check Chat Service

Run: docker compose logs --tail=30 chat

<img width="654" height="379" alt="image" src="https://github.com/user-attachments/assets/b6b477c9-5f41-4f94-8fae-604de953f611" /> 

Chat Service is also successfully connected to MongoDB.

. Completed: 

<img width="425" height="175" alt="image" src="https://github.com/user-attachments/assets/fe4613e2-b7f9-4f53-8e89-4f96348eab59" />


Your complete Docker architecture is working

<img width="374" height="215" alt="image" src="https://github.com/user-attachments/assets/48c39cc7-f623-472d-bda2-9a659c6808fa" />

================================================================================================ 
============================================================================================== 

Phase 3 — Kubernetes / Orchestration 


====================================================
Notes:
Now we will take these containers and run/manage them with Kubernetes. 

Why Kubernetes?
Docker Compose is excellent for running our application locally.
However, Kubernetes adds orchestration capabilities such as:
    . managing multiple containers as Pods
    . restarting failed containers
    . maintaining the desired number of replicas
    . scaling services
    . providing stable networking
    . performing rolling updates
    . managing configuration and secrets 

 =========================================================== 

 Phase 3 — Step 1: Check Kubernetes

 Step 1A — Check kubectl
 Run: kubectl version --client

  
  <img width="328" height="107" alt="image" src="https://github.com/user-attachments/assets/e3acb3a5-ccd0-4038-a6a1-55c87bc8f4ca" /> 

  kubectl is the command-line tool we will use to communicate with Kubernetes:

Step 1B: Check the Kubernetes Cluster

Run: kubectl cluster-info 

<img width="647" height="104" alt="image" src="https://github.com/user-attachments/assets/a4f5cc2e-2d67-4fc6-81c5-0e55d8e0fe75" /> 

The below is all set:

Windows
   │
   ├── Docker Desktop
   │       │
   │       ├── Docker Engine
   │       │      └── StreamingApp containers
   │       │
   │       └── Kubernetes
   │              ├── Control Plane
   │              └── CoreDNS
   │
   └── kubectl 

   ====================================== 

   Step 2: Verify the Kubernetes Nodes 
   
============================================= 

   Notes:
What is a node?
A Node is the machine where Kubernetes runs your application Pods.
In our Docker Desktop Kubernetes environment, Docker Desktop provides the Kubernetes node for us.

Conceptually:

Kubernetes Cluster
        │
        └── Node
             │
             ├── Pod → Auth
             ├── Pod → Streaming
             ├── Pod → Admin
             ├── Pod → Chat
             ├── Pod → Frontend
             └── Pod → MongoDB 

Later, Kubernetes will manage these Pods for us. 

======================================================== 

Run: kubectl get nodes

<img width="375" height="61" alt="image" src="https://github.com/user-attachments/assets/d5c1eb2f-a338-447d-a390-bb2213252b08" /> 

====================================
Notes:
Explanation of output:

. docker-desktop → your Kubernetes node
. Ready → Kubernetes can schedule and run workloads
. control-plane → this node is currently providing Kubernetes control-plane functions
. v1.36.1 → Kubernetes version

==============================================

Step 3: Create a Kubernetes folder structure 

=========================================== 

Now we need to prepare the project files for Kubernetes.

Notes:

Why do we need separate Kubernetes files?
. Our current application is controlled by: docker-compose.yml
. Docker Compose describes: Services → Containers → Ports → Environment
. Kubernetes will describe the same application using Kubernetes resources:

 . Deployments → Pods
 . Services → Networking
 . ConfigMaps → Configuration
 . Secrets → Sensitive configuration
 . Ingress → External access

 We do not want to replace your Docker Compose file.

. Instead, we'll add a Kubernetes directory:

StreamingApp
│
├── backend
├── frontend
├── docker-compose.yml
│
└── k8s
    ├── ...
    └── ... 

Later, this k8s directory will contain the Kubernetes manifests required by the assignment.

============================================ 

We're only creating the k8s folder right now:

Run: New-Item -ItemType Directory -Name k8s

<img width="433" height="129" alt="image" src="https://github.com/user-attachments/assets/69d59b9c-ab80-42d2-a4f3-6fb039404b65" /> 


Run: Get-ChildItem 

<img width="550" height="285" alt="image" src="https://github.com/user-attachments/assets/731047a6-24a8-4d65-88ec-7774b172e5a3" /> 


======================== 
. Current structure:

C:\Users\sandy\StreamingApp
│
├── backend
├── frontend
├── docker-compose.yml
├── README.md
└── k8s
    └── (empty) 

============================= 

Notes:

. Kubernetes uses different resource types for different purposes:

Kubernetes resource	Purpose
Deployment -->	      Runs and manages application Pods
Service	   -->       Gives Pods stable networking
ConfigMap	 -->       Stores non-sensitive configuration
Secret	    -->          Stores sensitive values
PersistentVolume / PVC -->	Keeps MongoDB data persistent
Ingress	   -->      Provides external HTTP access  



<img width="428" height="190" alt="image" src="https://github.com/user-attachments/assets/10385276-f520-49d8-9500-06e4e37df5d5" />  


========================= 

Step 4: Decide the Kubernetes architecture

=============================== 

Notes:

Current Docker Compose
docker-compose.yml
       │
       ├── mongo
       ├── auth
       ├── streaming
       ├── admin
       ├── chat
       └── frontend 


------------------------------ 

Kubernetes equivalent:

Kubernetes Cluster
│
├── MongoDB
│   ├── Deployment
│   ├── Service
│   └── PersistentVolumeClaim
│
├── Auth
│   ├── Deployment
│   └── Service
│
├── Streaming
│   ├── Deployment
│   └── Service
│
├── Admin
│   ├── Deployment
│   └── Service
│
├── Chat
│   ├── Deployment
│   └── Service
│
└── Frontend
    ├── Deployment
    └── Service       

-------------------------------- 

Important points:
. For MongoDB, we shouldn't treat its container exactly like a stateless backend service. Its data needs persistence. That's why we'll use a PersistentVolumeClaim. 
. For the Node.js services and frontend, Kubernetes can manage them as stateless workloads and scale them with replicas.

======================================== 

 . First Kubernetes manifest: MongoDB 

. We'll start with MongoDB, because the backend services depend on it.
. Next we'll create k8s/mongo.yaml containing:
  MongoDB Deployment
  MongoDB Service
  PersistentVolumeClaim

. We'll then apply it and verify MongoDB before moving to Auth.

.Why start with MongoDB?
 Because the dependency chain is:

 MongoDB
   ↓
 Auth / Streaming / Admin / Chat
   ↓
 Frontend

========================== 

Step 4A: Create MongoDB Kubernetes resources 

====================== 

Notes:
We will create: k8s/mongo.yaml

It will contain three Kubernetes resources:
  1. PersistentVolumeClaim (PVC) — keeps MongoDB data
  2. Deployment — runs the MongoDB container
  3. Service — gives MongoDB a stable internal DNS name

     
------------------------------- 

     
. Why all three?

PersistentVolumeClaim
        ↓
   MongoDB Pod
        ↓
MongoDB Service
        ↓
mongodb:27017
        ↓
Auth / Streaming / Admin / Chat


. The Service is especially important because our current Docker Compose configuration uses:
  mongodb://mongo:27017/streamingapp 
. In Kubernetes, we'll give MongoDB the Service name mongo, so the backend services can continue using:
  mongo:27017

================================= 

Create mongo.yaml:

Run: notepad .\k8s\mongo.yaml 

Add the below code:

apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: mongo-pvc
spec:
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 1Gi

---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: mongo
spec:
  replicas: 1
  selector:
    matchLabels:
      app: mongo
  template:
    metadata:
      labels:
        app: mongo
    spec:
      containers:
        - name: mongo
          image: mongo:6
          ports:
            - containerPort: 27017
          volumeMounts:
            - name: mongo-storage
              mountPath: /data/db
      volumes:
        - name: mongo-storage
          persistentVolumeClaim:
            claimName: mongo-pvc

---
apiVersion: v1
kind: Service
metadata:
  name: mongo
spec:
  selector:
    app: mongo
  ports:
    - port: 27017
      targetPort: 27017
  type: ClusterIP 

  --------------------------  
  
  If Notepad asks whether you want to create the file, choose Yes. Save the file and close Notepad.

What this YAML means:
. asks Kubernetes for 1 GB of persistent storage: 
   PVC: storage: 1Gi
. we initially run one MongoDB Pod: 
  Deployment: replicas: 1
, other Kubernetes Pods can reach MongoDB using:
  Service: name: mongo
. MongoDB is internal to the Kubernetes cluster and . We don't need to expose the database to your Windows host:
  mongo:27017 and type: ClusterIP  

 Now We have:

k8s/
└── mongo.yaml

------------------- 

And the file contains:

PersistentVolumeClaim
        ↓
   MongoDB Deployment
        ↓
    MongoDB Service

  ----------------------- 
  =========================== 

  Step 4B — Validate mongo.yaml

  =============================== 

  Run: kubectl apply --dry-run=client -f .\k8s\mongo.yaml  

  <img width="513" height="57" alt="image" src="https://github.com/user-attachments/assets/8625548d-a750-4e24-8627-106222273333" />


Notes:
  What --dry-run=client does:
  It checks whether Kubernetes can understand the manifest without actually creating the resources.

==================== 

Step 4C — Deploy MongoDB

======================== 

Run: kubectl apply -f .\k8s\mongo.yaml 

<img width="471" height="77" alt="image" src="https://github.com/user-attachments/assets/c81421b9-e859-4fce-9dcd-6c38d3298d71" />


What this does:
This time Kubernetes will actually create:

. mongo-pvc → persistent storage request
. mongo Deployment → MongoDB Pod
. mongo Service → internal access at mongo:27017 

================== 

Step 4D — Verify MongoDB Pod

==================== 

Run:  kubectl get pods


<img width="370" height="61" alt="image" src="https://github.com/user-attachments/assets/89f51a15-f05e-4aa6-8094-a4050aa060ce" />

MongoDB is running successfully inside Kubernetes.

=========================================

Our Kubernetes environment now looks like:

Kubernetes Cluster
       │
       └── MongoDB
            │
            ├── Deployment 
            ├── Pod      Running
            ├── Service
            └── PVC       

This is important:
. Previously, MongoDB was managed by Docker Compose:

docker-compose
      ↓
mongo container


------------------- 

Now Kubernetes is managing it:

Kubernetes
      ↓
Mongo Deployment
      ↓
Mongo Pod
      ↓
Persistent storage      

--------------------------- 

Step 4E: Verify MongoDB Service

=============================

Run: kubectl get services

<img width="427" height="63" alt="image" src="https://github.com/user-attachments/assets/0ba042c2-454e-404d-9251-98e70424990b" />

--------------------------
Notes:
Outout means:

. Kubernetes has created an internal Service called: mongo
. It listens on: 27017
. and has the internal cluster IP: 10.108.196.246
. So our backend services will be able to use: mongodb://mongo:27017/streamingapp
. The important part is that we do not use 10.108.196.246 directly in our application configuration. Kubernetes DNS lets the services use the stable name: mongo

------------------------------- 

Our MongoDB Kubernetes setup is now fully verified:

<img width="221" height="161" alt="image" src="https://github.com/user-attachments/assets/a985a6d0-f68f-4e64-8eea-2eba11ae76b7" />


================================= 

Phase 3 — Step 5: Auth Service

================================== 

. Now we'll start converting the Auth Service from Docker Compose to Kubernetes.
. In Docker Compose, Auth currently gets: MONGO_URI=mongodb://mongo:27017/streamingapp
. Kubernetes will use the same MongoDB service name: mongo
. So the Auth Pod can connect to: mongodb://mongo:27017/streamingapp
. Rather than putting sensitive configuration directly into a Deployment, we'll eventually separate configuration into ConfigMaps and Secrets. 

============================== 

let's check whether your Kubernetes cluster can pull/use the existing streamingapp-auth image that we already have locally.

This is important because your image currently exists in Docker Desktop, and your Kubernetes cluster is also running through Docker Desktop. We want to verify the exact image/tag before creating the Auth.

Run: docker images streamingapp-auth

<img width="643" height="65" alt="image" src="https://github.com/user-attachments/assets/d8e297af-20f9-4695-af05-fa435b838277" />

======================================= 

Step 5B — Create the Auth Deployment 

===================================== 

Run: notepad .\k8s\auth.yaml

Add the below code and save the file:

apiVersion: apps/v1
kind: Deployment
metadata:
  name: auth
spec:
  replicas: 1
  selector:
    matchLabels:
      app: auth
  template:
    metadata:
      labels:
        app: auth
    spec:
      containers:
        - name: auth
          image: streamingapp-auth:latest
          imagePullPolicy: IfNotPresent
          ports:
            - containerPort: 3001
          env:
            - name: PORT
              value: "3001"
            - name: MONGO_URI
              value: "mongodb://mongo:27017/streamingapp"
            - name: JWT_SECRET
              value: "changeme"
            - name: CLIENT_URLS
              value: "http://localhost:3000"

---
apiVersion: v1
kind: Service
metadata:
  name: auth
spec:
  selector:
    app: auth
  ports:
    - port: 3001
      targetPort: 3001
  type: ClusterIP 

  ------------------------- 

  Notes:

. We're creating Two resources:

1. Deployment

auth Deployment
      ↓
auth Pod
      ↓
streamingapp-auth:latest 

----------------------------- 

2. Service

auth Service
      ↓
auth:3001
      ↓
Auth Pod 

-------------------- 

. The Service gives other Kubernetes Pods a stable DNS name: auth
. Similarly, Auth connects to: mongo:27017
. So Kubernetes service discovery becomes:

Auth Pod
   │
   └── mongo:27017
             ↓
       Mongo Service
             ↓
       MongoDB Pod 


----------------------------------------- 

===================================== 

Step 5C — Deploy Auth

Run: kubectl apply -f .\k8s\auth.yaml

<img width="452" height="44" alt="image" src="https://github.com/user-attachments/assets/0e51c677-a8ff-46bb-8746-8bfa142b1b26" />

Run: kubectl get pods

<img width="366" height="75" alt="image" src="https://github.com/user-attachments/assets/461f643e-28d0-4f94-b2df-42d73956b3d8" />

===================================== 

Phase 3 — Step 6: Create the remaining application manifests 

================================== 

. Create: k8s/app-services.yaml 

Run: notepad .\k8s\app-services.yaml

Paste the below code in the file:

apiVersion: apps/v1
kind: Deployment
metadata:
  name: streaming
spec:
  replicas: 1
  selector:
    matchLabels:
      app: streaming
  template:
    metadata:
      labels:
        app: streaming
    spec:
      containers:
        - name: streaming
          image: streamingapp-streaming:latest
          imagePullPolicy: IfNotPresent
          ports:
            - containerPort: 3002
          env:
            - name: PORT
              value: "3002"
            - name: MONGO_URI
              value: "mongodb://mongo:27017/streamingapp"
            - name: JWT_SECRET
              value: "changeme"
            - name: CLIENT_URLS
              value: "http://localhost:3000"
            - name: STREAMING_PUBLIC_URL
              value: "http://localhost:3002"

---
apiVersion: v1
kind: Service
metadata:
  name: streaming
spec:
  selector:
    app: streaming
  ports:
    - port: 3002
      targetPort: 3002
  type: ClusterIP

---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: admin
spec:
  replicas: 1
  selector:
    matchLabels:
      app: admin
  template:
    metadata:
      labels:
        app: admin
    spec:
      containers:
        - name: admin
          image: streamingapp-admin:latest
          imagePullPolicy: IfNotPresent
          ports:
            - containerPort: 3003
          env:
            - name: PORT
              value: "3003"
            - name: MONGO_URI
              value: "mongodb://mongo:27017/streamingapp"
            - name: JWT_SECRET
              value: "changeme"
            - name: CLIENT_URLS
              value: "http://localhost:3000"

---
apiVersion: v1
kind: Service
metadata:
  name: admin
spec:
  selector:
    app: admin
  ports:
    - port: 3003
      targetPort: 3003
  type: ClusterIP

---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: chat
spec:
  replicas: 1
  selector:
    matchLabels:
      app: chat
  template:
    metadata:
      labels:
        app: chat
    spec:
      containers:
        - name: chat
          image: streamingapp-chat:latest
          imagePullPolicy: IfNotPresent
          ports:
            - containerPort: 3004
          env:
            - name: PORT
              value: "3004"
            - name: MONGO_URI
              value: "mongodb://mongo:27017/streamingapp"
            - name: JWT_SECRET
              value: "changeme"
            - name: CLIENT_URLS
              value: "http://localhost:3000"

---
apiVersion: v1
kind: Service
metadata:
  name: chat
spec:
  selector:
    app: chat
  ports:
    - port: 3004
      targetPort: 3004
  type: ClusterIP

---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: frontend
spec:
  replicas: 1
  selector:
    matchLabels:
      app: frontend
  template:
    metadata:
      labels:
        app: frontend
    spec:
      containers:
        - name: frontend
          image: streamingapp-frontend:latest
          imagePullPolicy: IfNotPresent
          ports:
            - containerPort: 80

---
apiVersion: v1
kind: Service
metadata:
  name: frontend
spec:
  selector:
    app: frontend
  ports:
    - port: 80
      targetPort: 80
  type: NodePort 

  ---------------------------------------------- 

  Notes:

  Why we're using NodePort for Frontend
The backend services don't need to be exposed directly to your Windows machine. They're internal Kubernetes services.
The frontend, however, needs to be accessible from your browser.


So:
Backend:
ClusterIP → internal
----------------
Frontend:
NodePort → accessible externally
------------------------ 

Step 6A — Apply the remaining services 

Run: kubectl apply -f .\k8s\app-services.yaml

 <img width="512" height="135" alt="image" src="https://github.com/user-attachments/assets/5d2ac965-8b32-4e72-929b-c3cf6e4089f2" />


Run: kubectl get pods 

<img width="492" height="151" alt="image" src="https://github.com/user-attachments/assets/e0bf5e1b-de03-4899-b5b6-47ba35b450fc" />

------------------------ 

All six Kubernetes Pods are now running successfully:

admin      1/1 Running
auth       1/1 Running
chat       1/1 Running
frontend   1/1 Running
mongo      1/1 Running
streaming  1/1 Running 

===================== 

Current Kubernetes architecture
Kubernetes Cluster
│
├── MongoDB
│   ├── Deployment
│   ├── Pod ✅
│   ├── Service
│   └── PVC
│
├── Auth
│   ├── Deployment
│   ├── Pod ✅
│   └── Service
│
├── Streaming
│   ├── Deployment
│   ├── Pod ✅
│   └── Service
│
├── Admin
│   ├── Deployment
│   ├── Pod ✅
│   └── Service
│
├── Chat
│   ├── Deployment
│   ├── Pod ✅
│   └── Service
│
└── Frontend
    ├── Deployment
    ├── Pod ✅
    └── NodePort Service 

============================= 

Phase 3 — Step 7: Verify Kubernetes Services 

=================================== 

Run: kubectl get services  

<img width="518" height="143" alt="image" src="https://github.com/user-attachments/assets/fd132aeb-e726-42b7-8fc7-a64d4636bb2e" />

from the output:

. So Kubernetes has assigned NodePort 31142 to the frontend. 
. We now have the complete Kubernetes application:

<img width="253" height="191" alt="image" src="https://github.com/user-attachments/assets/456ab396-7bc9-47e1-8f9c-f9bcb6897a6b" />


=========================== 

Notes:
Currently:

auth       → 1 Pod
streaming  → 1 Pod
admin      → 1 Pod
chat       → 1 Pod
frontend   → 1 Pod 

Note: We'll demonstrate scaling by increasing the Streaming Service from: 1 replica → 3 replicas 
The Service remains: streaming:3002
============================ 

Phase 3 — Step 8: Scale the Streaming Service 

=========================== 





===================================

## Environment Configuration

Create an `.env` for each service (or export variables before running). All services accept the standard AWS credentials for S3 access.

### Auth Service (`backend/authService/.env`)
```ini
PORT=3001
MONGO_URI=mongodb://localhost:27017/streamingapp
JWT_SECRET=changeme
CLIENT_URLS=http://localhost:3000
AWS_ACCESS_KEY_ID=
AWS_SECRET_ACCESS_KEY=
AWS_REGION=ap-south-1
AWS_S3_BUCKET=
```

### Streaming Service (`backend/streamingService/.env`)
```ini
PORT=3002
MONGO_URI=mongodb://localhost:27017/streamingapp
JWT_SECRET=changeme
CLIENT_URLS=http://localhost:3000
AWS_ACCESS_KEY_ID=
AWS_SECRET_ACCESS_KEY=
AWS_REGION=ap-south-1
AWS_S3_BUCKET=
AWS_CDN_URL=
STREAMING_PUBLIC_URL=http://localhost:3002
```

### Admin Service (`backend/adminService/.env`)
```ini
PORT=3003
MONGO_URI=mongodb://localhost:27017/streamingapp
JWT_SECRET=changeme
CLIENT_URLS=http://localhost:3000
AWS_ACCESS_KEY_ID=
AWS_SECRET_ACCESS_KEY=
AWS_REGION=ap-south-1
AWS_S3_BUCKET=
```

### Chat Service (`backend/chatService/.env`)
```ini
PORT=3004
MONGO_URI=mongodb://localhost:27017/streamingapp
JWT_SECRET=changeme
CLIENT_URLS=http://localhost:3000
```

### Frontend build variables (`frontend/.env` or Docker build args)
```ini
REACT_APP_AUTH_API_URL=http://localhost:3001/api
REACT_APP_STREAMING_API_URL=http://localhost:3002/api
REACT_APP_STREAMING_PUBLIC_URL=http://localhost:3002
REACT_APP_ADMIN_API_URL=http://localhost:3003/api/admin
REACT_APP_CHAT_API_URL=http://localhost:3004/api/chat
REACT_APP_CHAT_SOCKET_URL=http://localhost:3004
```

## Running with Docker Compose

1. Populate the environment variables above (or rely on the defaults baked into `docker-compose.yml`).
2. Build and start the stack:
   ```bash
   docker-compose up --build
   ```
3. Navigate to `http://localhost:3000` for the web app.

The compose file provisions MongoDB plus all four Node.js microservices. S3 credentials are optional for local testing—you can still browse seeded metadata, but streaming requires valid S3 objects.

## Local Development

Install dependencies for each service:

```bash
# auth service
cd backend/authService && npm install

# streaming service
cd ../streamingService && npm install

# admin service
cd ../adminService && npm install

# chat service
cd ../chatService && npm install

# frontend
cd ../../frontend && npm install
```

Run the services (in separate terminals) after starting MongoDB:

```bash
cd backend/authService && npm run dev
cd backend/streamingService && npm run dev
cd backend/adminService && npm run dev
cd backend/chatService && npm run dev
cd frontend && npm start
```

## Feature Highlights

- **S3-backed adaptive streaming** with secure signed uploads for admins.
- **Dedicated admin microservice** for video ingestion, metadata management, and featured curation.
- **Real-time chat** overlay in the player (Socket.IO + persistent message history).
- **Modern React experience** featuring cinematic hero sections, dynamic carousels, and responsive design.
- **Role-aware access control** across frontend routes and backend microservices.

## Testing

Automated tests are not yet included. Recommended smoke checks:

1. Register and log in through the web UI.
2. Upload a small video + thumbnail via the admin dashboard (requires valid S3 credentials).
3. Confirm playback from the browse page and verify that chat messages broadcast between multiple browser tabs.

## License

MIT © StreamFlix Team
