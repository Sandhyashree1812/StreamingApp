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
. The Service remains: streaming:3002 
. and Kubernetes distributes traffic to the available Streaming Pods.

============================ 

Phase 3 — Step 8: Scale the Streaming Service 

=========================== 


Run: kubectl get pods 

<img width="392" height="126" alt="image" src="https://github.com/user-attachments/assets/a3dd9649-52e1-4444-b6a0-a6b69396e716" />


Run: kubectl scale deployment streaming --replicas=3 

<img width="635" height="40" alt="image" src="https://github.com/user-attachments/assets/8b11a87a-c184-43c7-9211-5044f2a60a74" />

Run: kubectl get pods

<img width="413" height="149" alt="image" src="https://github.com/user-attachments/assets/0b7baa6f-f2e4-47e1-b62e-39329f4114a6" /> 

--------------------- 

Phase 3 — Step 8, Scaling Complete 

Before
Streaming → 1 Pod
--------------------
After
Streaming → 3 Pods
-------------------------- 

============================= 

Phase 3 — Step 9: Verify the Deployment Replica Count 

Run: kubectl get deployment 

<img width="380" height="129" alt="image" src="https://github.com/user-attachments/assets/cefc13c7-2e15-4223-b81e-b9d6dd61f71b" />


--------------------- 

Notes:
That means Kubernetes currently has:
. 3 desired replicas
. 3 updated replicas
. 3 available replicas

Now we have:

Pod-level scaling
streaming-...-kzs5h   1/1 Running
streaming-...-sdk2c   1/1 Running
streaming-...-t229g   1/1 Running 

Deployment-level scaling
streaming   3/3   3   3 

--------------- 
================= 

Phase 3 — Step 10: Rolling Update 

===================== 

Notes:

What is a rolling update?
Suppose your Streaming Service is currently running:
Version A
   ├── Pod 1
   ├── Pod 2
   └── Pod 3 

When we deploy a new image/version, Kubernetes can gradually replace the old Pods rather than stopping all three at once:

Old Pod → New Pod
Old Pod → New Pod
Old Pod → New Pod
This helps maintain application availability during an update. 

--------------- 

We already have: streamingapp-streaming:latest
Rather than rebuilding the application—which could take a long time on your machine—we can demonstrate the rolling-update mechanism using the same application image with a new Kubernetes image tag.

-------------------
Step 10A — Create the v2 image tag 

First, create a second tag locally:

Run: docker tag streamingapp-streaming:latest streamingapp-streaming:v2 

<img width="607" height="41" alt="image" src="https://github.com/user-attachments/assets/51b2cd3c-86a4-4219-ad20-f5774e7912b4" />


This does not rebuild the image. It simply gives the existing image another tag. 

Then, we'll update the Kubernetes Deployment from: streamingapp-streaming:latest
to: streamingapp-streaming:v2 

So, Kubernetes will recognize this as a Deployment update and perform a rollout. 

---------------- 

Phase 3 — Step 10B: Update the Kubernetes Deployment 

========================= 


Run: kubectl set image deployment/streaming streaming=streamingapp-streaming:v2 

<img width="632" height="46" alt="image" src="https://github.com/user-attachments/assets/b40caa56-e042-4e82-bd68-9723741c6b95" />

Run: kubectl rollout status deployment/streaming 

<img width="473" height="44" alt="image" src="https://github.com/user-attachments/assets/d4810e74-798a-4e49-b044-1d57c23cda8b" />

Run:  kubectl get pods


<img width="428" height="151" alt="image" src="https://github.com/user-attachments/assets/cdf88b46-979e-40be-8f88-0d6a00818d25" />


Notice the Pod names changed from the previous ReplicaSet (7f97ccff69) to the new one (6858cc95f8). That is evidence that Kubernetes created new Pods as part of the Deployment update. 

----------------- 

Rolling update
Old Deployment version
        ↓
Kubernetes rollout
        ↓
New Pods
        ↓
3/3 Running 

<img width="608" height="215" alt="image" src="https://github.com/user-attachments/assets/3baa22a1-7c35-4206-a1e4-78d1c1dfbbfd" /> 

====================== 

Phase 3 — Step 10 ✅ COMPLETE
===============================


<img width="608" height="215" alt="image" src="https://github.com/user-attachments/assets/cf96c1de-b09e-4b07-b368-e0fe87e6b30c" />


============================== 

Phase 3 — Step 11: Open the Kubernetes Frontend 

=============================== 

frontend Service earlier showed: frontend   NodePort   80:31142/TCP
So Kubernetes exposed the frontend through NodePort 31142.

Open this in your browser: http://localhost:31142 

<img width="1920" height="1080" alt="image" src="https://github.com/user-attachments/assets/0e151994-875c-4ff6-a8d3-7c8fe972e4e0" />

============== 
Phase 3 — Step 11 ✅ Complete 
============================= 

We have now proven:

Browser
   ↓
localhost:31142
   ↓
Kubernetes NodePort
   ↓
Frontend Service
   ↓
Frontend Pod
   ↓
Nginx + React
   ↓
StreamFlix page ✅

=========================== 

Phase 3 — Step 12: ConfigMap and Secret 

=============================== 

Notes:

Our current deployments contain configuration such as:
PORT
MONGO_URI
CLIENT_URLS
---------------------------------- 

and sensitive configuration: JWT_SECRET

-------------------------------------- 

Kubernetes provides:

ConfigMap → non-sensitive configuration
Secret → sensitive configuration

We'll create both. 

============================== 

Step 12A — Create config.yaml

============================== 

Run: notepad .\k8s\config.yaml 

Paste the below code:

apiVersion: v1
kind: ConfigMap
metadata:
  name: streamingapp-config
data:
  MONGO_URI: "mongodb://mongo:27017/streamingapp"
  CLIENT_URLS: "http://localhost:31142"
  AUTH_PORT: "3001"
  STREAMING_PORT: "3002"
  ADMIN_PORT: "3003"
  CHAT_PORT: "3004"

---
apiVersion: v1
kind: Secret
metadata:
  name: streamingapp-secret
type: Opaque
stringData:
  JWT_SECRET: "changeme" 


  ---------------------------------------------------------- 

  Notes:

. Instead of putting configuration directly into every Deployment:

Deployment
 ├── PORT
 ├── MONGO_URI
 ├── CLIENT_URLS
 └── JWT_SECRET

--------------------------- 

we can centrally manage it:

             ┌── ConfigMap
Deployments ─┤
             └── Secret

This demonstrates Kubernetes configuration and secret management.
For this assignment, changeme is only a demonstration value. We do not use a real production secret in GitHub.

==================================== 

Step 12B — Apply it 

====================== 

Run: kubectl apply -f .\k8s\config.yaml 

<img width="553" height="47" alt="image" src="https://github.com/user-attachments/assets/6a7dd8f8-dcac-4c68-9e47-db71de974d86" />


Run: kubectl get configmap,secret 


<img width="375" height="110" alt="image" src="https://github.com/user-attachments/assets/284c1110-f45d-433e-ad6c-d88ef6194bc2" />

------------------------------------------------- 

Notes:

So we now have:

Kubernetes
│
├── ConfigMap
│   └── streamingapp-config
│
└── Secret
    └── streamingapp-secret

------------------------------- 
================================= 

Phase 3 — Step 13: Check Ingress support 

-===================================== 

Notes:

our assignment requires exposing the application through Ingress.

Before we create an Ingress manifest, we need to know whether our Docker Desktop Kubernetes cluster has an Ingress controller available. 

Why?

An Ingress resource by itself doesn't route traffic. We need an Ingress Controller to actually process it.

Conceptually:

Browser
   ↓
Ingress Controller
   ↓
Ingress
   ├── frontend
   ├── auth
   ├── streaming
   ├── admin
   └── chat

-------------------------- 

Run: kubectl get ingressclass 


<img width="348" height="50" alt="image" src="https://github.com/user-attachments/assets/c49ef915-59ca-4219-9e58-116749c5b959" />

our Docker Desktop Kubernetes cluster currently has no IngressClass, so there is no Ingress controller installed. 

=================== 

Phase 3 — Step 13A: Install NGINX Ingress Controller 

=================================== 

Run: kubectl apply -f https://raw.githubusercontent.com/kubernetes/ingress-nginx/controller-v1.15.1/deploy/static/provider/cloud/deploy.yaml

<img width="581" height="295" alt="image" src="https://github.com/user-attachments/assets/15e1eb0d-efee-4e5a-8537-6260ea2b3be3" />


-------------------------- 

Notes:

This is the current installation manifest shown in the official ingress-nginx installation guide.

What this creates
It will create the ingress-nginx namespace and the controller resources:

Kubernetes
   │
   └── ingress-nginx
       └── NGINX Ingress Controller
                │
                ↓
             Ingress
                │
                ↓
            frontend


--------------------------------- 

Step 1 — Check the Ingress Controller 

Run: kubectl get pods -n ingress-nginx 

<img width="516" height="69" alt="image" src="https://github.com/user-attachments/assets/6bf1f35c-9ad2-4028-a8d1-ee8305d5d39c" />

Run: kubectl get ingressclass

<img width="413" height="57" alt="image" src="https://github.com/user-attachments/assets/2aec3768-241d-4cbe-8e3f-46fc384e3ad1" />

--------------------------------- 

Step 2 — Create the Ingress 

============================== 

Notes:

. The purpose of this Ingress is to provide a single entry point to your application and route traffic to the existing frontend Service.

. Our current frontend service is: frontend → port 80 

. Create the file:
C:\Users\sandy\StreamingApp\k8s\ingress.yaml

. What this does
Browser
   │
   ▼
NGINX Ingress
   │
   │  /
   ▼
frontend Service :80
   │
   ▼
Frontend Pod

This is useful for our assignment because instead of exposing every application service externally, Kubernetes can use the Ingress as the application's entry point. 

-----------------------------------

Step 2A — Create ingress.yaml

============================== 

Run: notepad k8s\ingress.yaml 

Paste the below code in it and save it:

apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: streamingapp-ingress
spec:
  ingressClassName: nginx
  rules:
    - http:
        paths:
          - path: /
            pathType: Prefix
            backend:
              service:
                name: frontend
                port:
                  number: 80 

--------------------------------------------- 

Step 2B — Verify the file exists

Run: dir

<img width="395" height="200" alt="image" src="https://github.com/user-attachments/assets/85a8852b-d4d9-4833-8295-2a276a8ab8b8" />

---------------------- 

Step 2C — Apply the Ingress


Run: kubectl apply -f k8s\ingress.yaml


<img width="416" height="51" alt="image" src="https://github.com/user-attachments/assets/168e23f0-e5e3-4918-b948-4f357bae6849" />

Run:  kubectl get ingress



<img width="393" height="74" alt="image" src="https://github.com/user-attachments/assets/b68f347d-571e-448b-b28a-922ce4a54b87" />

--------------------------- 

Notes:

This means:

Browser
   │
   ▼
localhost:80
   │
   ▼
NGINX Ingress
   │
   ▼
frontend Service :80
   │
   ▼
StreamFlix Frontend Pod

================================== 

Step 3 — Test the application through Ingress

Open a new browser tab and go to: http://localhost

You should see your StreamFlix / StreamingApp frontend.


<img width="899" height="479" alt="image" src="https://github.com/user-attachments/assets/802cfb9a-ff1c-42e3-888c-cff4fb398217" />

--------------------------- 

our screenshot clearly shows:

. Browser URL: http://localhost
. StreamFlix frontend is loading correctly.
. Traffic is going through the Kubernetes NGINX Ingress.
. This proves the chain:

Browser
   ↓
NGINX Ingress
   ↓
frontend Service
   ↓
Frontend Pod
   ↓
StreamFlix application

-------------------------------- 

Current Kubernetes work completed

=================================== 

Final Kubernetes evidence: 

=========================== 


Run: kubectl get pods 

<img width="365" height="151" alt="image" src="https://github.com/user-attachments/assets/a5b720b7-5260-4d9d-91a0-e38174711c2e" />

Run: kubectl get deployments

<img width="353" height="122" alt="image" src="https://github.com/user-attachments/assets/71f3c346-88dc-4dd3-abf7-2230416ff1bb" />

Run: kubectl get services


<img width="496" height="146" alt="image" src="https://github.com/user-attachments/assets/1211c67d-ffc5-4790-aceb-a7d818717563" />

------------------------- 

our outputs prove the below:

Kubernetes status
*****************

All application pods are healthy:

admin       1/1 Running
auth        1/1 Running
chat        1/1 Running
frontend    1/1 Running
mongo       1/1 Running
streaming   3/3 Running 
------------------------
. We have three Streaming service replicas running.
. the deployment has 3 desired, 3 updated, and 3 available replicas.
. Services

our architecture is as required:

frontend   NodePort   80:31142
auth       ClusterIP  3001
streaming  ClusterIP  3002
admin      ClusterIP  3003
chat       ClusterIP  3004
mongo      ClusterIP  27017


. our NGINX Ingress is serving the frontend through: http://localhost

====================================== 

PHASE 4 — Kubernetes Container Orchestration

Step 4.1 — Prepare Kubernetes cluster 



<img width="659" height="265" alt="image" src="https://github.com/user-attachments/assets/642ec8d3-cc81-42ca-baff-7eaef50b7415" />

shows that MongoDB itself is running and the PVC is bound. 

Next: Phase 4 — Step 4.2 status


Run: get deployment mongo


<img width="404" height="50" alt="image" src="https://github.com/user-attachments/assets/5597269c-eb62-4d63-bb9c-510fc0f0c521" />

This confirms MongoDB is healthy and running inside Kubernetes.

---------------------------- 

Run:  kubectl describe deployment mongo

<img width="593" height="394" alt="image" src="https://github.com/user-attachments/assets/ec3b6933-5abb-4573-b81b-d3aaad525286" />

<img width="497" height="140" alt="image" src="https://github.com/user-attachments/assets/35ffc4f6-96f3-4163-ba95-e3beab7a0145" />

================================ 

Phase 5 — Configuration & Security 

Run the below commands:
kubectl get configmap streamingapp-config
kubectl get secret streamingapp-secret
kubectl get ingress
kubectl get svc 

<img width="471" height="263" alt="image" src="https://github.com/user-attachments/assets/38dd0874-ed59-42e3-b769-23678ecff288" />

================================= 

Step 5.1 Update k8s/config.yml

=================================== 

apiVersion: v1
kind: ConfigMap
metadata:
  name: streamingapp-config
data:
  MONGO_URI: "mongodb://mongo:27017/streamingapp"
  CLIENT_URLS: "http://localhost:31142"
  AUTH_PORT: "3001"
  STREAMING_PORT: "3002"
  ADMIN_PORT: "3003"
  CHAT_PORT: "3004"

---
apiVersion: v1
kind: Secret
metadata:
  name: streamingapp-secret
type: Opaque
stringData:
  JWT_SECRET: "changeme" 
  
---------------- 
======================================

2. k8s/auth.yaml

==================================

Replace the entire file with below code:

apiVersion: apps/v1
kind: Deployment
metadata:
  name: auth
spec:
  replicas: 1

  selector:
    matchLabels:
      app: auth

  strategy:
    type: RollingUpdate
    rollingUpdate:
      maxUnavailable: 0
      maxSurge: 1

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

          envFrom:
            - configMapRef:
                name: streamingapp-config
            - secretRef:
                name: streamingapp-secret

          readinessProbe:
            tcpSocket:
              port: 3001
            initialDelaySeconds: 30
            periodSeconds: 10
            timeoutSeconds: 2
            failureThreshold: 6

          livenessProbe:
            tcpSocket:
              port: 3001
            initialDelaySeconds: 60
            periodSeconds: 20
            timeoutSeconds: 2
            failureThreshold: 3

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
  
-----------------------------  

Notes:
Why we used tcpSocket instead of /health
This is intentional.
We haven't confirmed that the Node.js services expose a /health endpoint. A TCP probe checks whether the application is actually listening on its port without assuming a particular HTTP endpoint.
So we can satisfy the readiness/liveness probe requirement without risking the application because of a nonexistent /health route.


------------------ 

3. k8s/app-services.yaml
Replace the entire file with this:


# ============================================================
# STREAMING SERVICE
# ============================================================

apiVersion: apps/v1
kind: Deployment
metadata:
  name: streaming
spec:
  replicas: 3

  selector:
    matchLabels:
      app: streaming

  strategy:
    type: RollingUpdate
    rollingUpdate:
      maxUnavailable: 0
      maxSurge: 1

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

          envFrom:
            - configMapRef:
                name: streamingapp-config
            - secretRef:
                name: streamingapp-secret

          readinessProbe:
            tcpSocket:
              port: 3002
            initialDelaySeconds: 30
            periodSeconds: 10
            timeoutSeconds: 2
            failureThreshold: 6

          livenessProbe:
            tcpSocket:
              port: 3002
            initialDelaySeconds: 60
            periodSeconds: 20
            timeoutSeconds: 2
            failureThreshold: 3

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


# ============================================================
# ADMIN SERVICE
# ============================================================

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

  strategy:
    type: RollingUpdate
    rollingUpdate:
      maxUnavailable: 0
      maxSurge: 1

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

   envFrom:
            - configMapRef:
                name: streamingapp-config
            - secretRef:
                name: streamingapp-secret

   readinessProbe:
            tcpSocket:
              port: 3003
            initialDelaySeconds: 30
            periodSeconds: 10
            timeoutSeconds: 2
            failureThreshold: 6

   livenessProbe:
            tcpSocket:
              port: 3003
            initialDelaySeconds: 60
            periodSeconds: 20
            timeoutSeconds: 2
            failureThreshold: 3

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


# ============================================================
# CHAT SERVICE
# ============================================================

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

  strategy:
    type: RollingUpdate
    rollingUpdate:
      maxUnavailable: 0
      maxSurge: 1

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

          envFrom:
            - configMapRef:
                name: streamingapp-config
            - secretRef:
                name: streamingapp-secret

          readinessProbe:
            tcpSocket:
              port: 3004
            initialDelaySeconds: 30
            periodSeconds: 10
            timeoutSeconds: 2
            failureThreshold: 6

          livenessProbe:
            tcpSocket:
              port: 3004
            initialDelaySeconds: 60
            periodSeconds: 20
            timeoutSeconds: 2
            failureThreshold: 3

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


# ============================================================
# FRONTEND SERVICE
# ============================================================

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

  strategy:
    type: RollingUpdate
    rollingUpdate:
      maxUnavailable: 0
      maxSurge: 1

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

          readinessProbe:
            tcpSocket:
              port: 80
            initialDelaySeconds: 15
            periodSeconds: 10
            timeoutSeconds: 2
            failureThreshold: 6

          livenessProbe:
            tcpSocket:
              port: 80
            initialDelaySeconds: 30
            periodSeconds: 20
            timeoutSeconds: 2
            failureThreshold: 3

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
------------------------ 
================================= 

4. k8s/ingress.yaml 

========================
Now replace the entire file with:

apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: streamingapp-ingress

  annotations:
    nginx.ingress.kubernetes.io/proxy-http-version: "1.1"
    nginx.ingress.kubernetes.io/proxy-read-timeout: "3600"
    nginx.ingress.kubernetes.io/proxy-send-timeout: "3600"
    nginx.ingress.kubernetes.io/proxy-buffering: "off"

spec:
  ingressClassName: nginx

  rules:
    - http:

   paths:

   # Frontend
   - path: /
            pathType: Prefix
            backend:
              service:
                name: frontend
                port:
                  number: 80

       # Authentication
        - path: /api/auth
            pathType: Prefix
            backend:
              service:
                name: auth
                port:
                  number: 3001

          # Streaming
          - path: /api/streaming
            pathType: Prefix
            backend:
              service:
                name: streaming
                port:
                  number: 3002

          # Admin
          - path: /api/admin
            pathType: Prefix
            backend:
              service:
                name: admin
                port:
                  number: 3003

          # Chat / WebSocket
          - path: /api/chat
            pathType: Prefix
            backend:
              service:
                name: chat
                port:
                  number: 3004 

                  

    ------------------------------------------------- 

This gives you the required single entry point: http://localhost

This gives us the required single entry point: http://localhost

with:
/                  → frontend:80
/api/auth          → auth:3001
/api/streaming     → streaming:3002
/api/admin         → admin:3003
/api/chat          → chat:3004

And the NGINX timeout/HTTP 1.1 settings support the chat WebSocket connection. 

--------------- 


5. Validate YAML BEFORE applying

 Run: 
 kubectl apply --dry-run=client -f k8s/config.yaml
 kubectl apply --dry-run=client -f k8s/auth.yaml
 kubectl apply --dry-run=client -f k8s/app-services.yaml
 kubectl apply --dry-run=client -f k8s/ingress.yaml 

 <img width="505" height="221" alt="image" src="https://github.com/user-attachments/assets/9f126ae9-efca-40ce-9892-46c9d854918f" />

Run: 

kubectl apply -f k8s/config.yaml
kubectl apply -f k8s/auth.yaml
kubectl apply -f k8s/app-services.yaml
kubectl apply -f k8s/ingress.yaml 

<img width="501" height="226" alt="image" src="https://github.com/user-attachments/assets/7f8e377e-d00c-4639-835f-95978c6f3794" />

   
Run:
kubectl get pods  

<img width="425" height="134" alt="image" src="https://github.com/user-attachments/assets/79487652-f41d-43a0-895e-a0b8fe61dd60" />


Run:
kubectl get pods,svc,ingress

<img width="507" height="290" alt="image" src="https://github.com/user-attachments/assets/bc72bbbc-8bdf-4064-8e26-9aa4c0ffca72" />

Run: 
kubectl get deployments
kubectl rollout status deployment/streaming
kubectl get ingress streamingapp-ingress -o wide

<img width="598" height="189" alt="image" src="https://github.com/user-attachments/assets/640591e2-7e55-48f3-b1e2-529ec74ca844" />

=================================================================== 

Phase 8 — Helm Packaging

==========================



Step 8.1 — Create the Helm folders 

=========================== 

run:
mkdir helm
mkdir helm\streamingapp
mkdir helm\streamingapp\templates 

<img width="422" height="272" alt="image" src="https://github.com/user-attachments/assets/3fb34397-303c-4724-8828-736248e15ee5" />


<img width="418" height="315" alt="image" src="https://github.com/user-attachments/assets/205fad08-1af3-4eba-9972-aeeb1030cc52" />

-------------------------------------- 

Step 8.2 — Create Chart.yaml 

Run: 

@'
apiVersion: v2
name: streamingapp
description: Helm chart for the StreamingApp microservices platform
type: application
version: 1.0.0
appVersion: "1.0.0"

keywords:
  - streaming
  - microservices
  - kubernetes
  - mongodb
  - nginx
'@ | Set-Content helm\streamingapp\Chart.yaml

------------------------- 

3. Create values.yaml
Run:

@'
replicaCount:
  auth: 1
  streaming: 3
  admin: 1
  chat: 1
  frontend: 1
  mongo: 1

images:
  auth:
    repository: streamingapp-auth
    tag: latest
    pullPolicy: IfNotPresent

  streaming:
    repository: streamingapp-streaming
    tag: latest
    pullPolicy: IfNotPresent

  admin:
    repository: streamingapp-admin
    tag: latest
    pullPolicy: IfNotPresent

  chat:
    repository: streamingapp-chat
    tag: latest
    pullPolicy: IfNotPresent

  frontend:
    repository: streamingapp-frontend
    tag: latest
    pullPolicy: IfNotPresent

  mongo:
    repository: mongo
    tag: "6"
    pullPolicy: IfNotPresent

service:
  auth:
    port: 3001

  streaming:
    port: 3002

  admin:
    port: 3003

  chat:
    port: 3004

  frontend:
    port: 80

  mongo:
    port: 27017

config:
  mongoUri: mongodb://mongo:27017/streamingapp
  clientUrls: http://localhost:31142

secret:
  jwtSecret: changeme

ingress:
  enabled: true
  className: nginx
  host: ""
'@ | Set-Content helm\streamingapp\values.yaml 

-------------------- 

4. Create templates/configmap.yaml
Run:

@'
apiVersion: v1
kind: ConfigMap
metadata:
  name: streamingapp-config
data:
  MONGO_URI: {{ .Values.config.mongoUri | quote }}
  CLIENT_URLS: {{ .Values.config.clientUrls | quote }}
  AUTH_PORT: {{ .Values.service.auth.port | quote }}
  STREAMING_PORT: {{ .Values.service.streaming.port | quote }}
  ADMIN_PORT: {{ .Values.service.admin.port | quote }}
  CHAT_PORT: {{ .Values.service.chat.port | quote }}
'@ | Set-Content helm\streamingapp\templates\configmap.yaml 

------------------- 

5. Create templates/secret.yaml
Run:

@'
apiVersion: v1
kind: Secret
metadata:
  name: streamingapp-secret
type: Opaque
stringData:
  JWT_SECRET: {{ .Values.secret.jwtSecret | quote }}
'@ | Set-Content helm\streamingapp\templates\secret.yaml 

--------------- 

6. Create templates/auth.yaml
Run:

@'
apiVersion: apps/v1
kind: Deployment
metadata:
  name: auth
spec:
  replicas: {{ .Values.replicaCount.auth }}

  selector:
    matchLabels:
      app: auth

  strategy:
    type: RollingUpdate
    rollingUpdate:
      maxUnavailable: 0
      maxSurge: 1

  template:
    metadata:
      labels:
        app: auth

    spec:
      containers:
        - name: auth
          image: "{{ .Values.images.auth.repository }}:{{ .Values.images.auth.tag }}"
          imagePullPolicy: {{ .Values.images.auth.pullPolicy }}

          ports:
            - containerPort: 3001

          envFrom:
            - configMapRef:
                name: streamingapp-config
            - secretRef:
                name: streamingapp-secret

          readinessProbe:
            tcpSocket:
              port: 3001
            initialDelaySeconds: 30
            periodSeconds: 10
            timeoutSeconds: 2
            failureThreshold: 6

          livenessProbe:
            tcpSocket:
              port: 3001
            initialDelaySeconds: 60
            periodSeconds: 20
            timeoutSeconds: 2
            failureThreshold: 3

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
'@ | Set-Content helm\streamingapp\templates\auth.yaml 

------------- 

7. Create templates/app-services.yaml
This one contains streaming + admin + chat + frontend.
Run:

@'
# ============================================================
# STREAMING SERVICE
# ============================================================

apiVersion: apps/v1
kind: Deployment
metadata:
  name: streaming
spec:
  replicas: {{ .Values.replicaCount.streaming }}

  selector:
    matchLabels:
      app: streaming

  strategy:
    type: RollingUpdate
    rollingUpdate:
      maxUnavailable: 0
      maxSurge: 1

  template:
    metadata:
      labels:
        app: streaming

    spec:
      containers:
        - name: streaming
          image: "{{ .Values.images.streaming.repository }}:{{ .Values.images.streaming.tag }}"
          imagePullPolicy: {{ .Values.images.streaming.pullPolicy }}

          ports:
            - containerPort: 3002

          envFrom:
            - configMapRef:
                name: streamingapp-config
            - secretRef:
                name: streamingapp-secret

          readinessProbe:
            tcpSocket:
              port: 3002
            initialDelaySeconds: 30
            periodSeconds: 10
            timeoutSeconds: 2
            failureThreshold: 6

          livenessProbe:
            tcpSocket:
              port: 3002
            initialDelaySeconds: 60
            periodSeconds: 20
            timeoutSeconds: 2
            failureThreshold: 3

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


# ============================================================
# ADMIN SERVICE
# ============================================================

---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: admin
spec:
  replicas: {{ .Values.replicaCount.admin }}

  selector:
    matchLabels:
      app: admin

  strategy:
    type: RollingUpdate
    rollingUpdate:
      maxUnavailable: 0
      maxSurge: 1

  template:
    metadata:
      labels:
        app: admin

    spec:
      containers:
        - name: admin
          image: "{{ .Values.images.admin.repository }}:{{ .Values.images.admin.tag }}"
          imagePullPolicy: {{ .Values.images.admin.pullPolicy }}

          ports:
            - containerPort: 3003

          envFrom:
            - configMapRef:
                name: streamingapp-config
            - secretRef:
                name: streamingapp-secret

          readinessProbe:
            tcpSocket:
              port: 3003
            initialDelaySeconds: 30
            periodSeconds: 10
            timeoutSeconds: 2
            failureThreshold: 6

          livenessProbe:
            tcpSocket:
              port: 3003
            initialDelaySeconds: 60
            periodSeconds: 20
            timeoutSeconds: 2
            failureThreshold: 3

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


# ============================================================
# CHAT SERVICE
# ============================================================

---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: chat
spec:
  replicas: {{ .Values.replicaCount.chat }}

  selector:
    matchLabels:
      app: chat

  strategy:
    type: RollingUpdate
    rollingUpdate:
      maxUnavailable: 0
      maxSurge: 1

  template:
    metadata:
      labels:
        app: chat

    spec:
      containers:
        - name: chat
          image: "{{ .Values.images.chat.repository }}:{{ .Values.images.chat.tag }}"
          imagePullPolicy: {{ .Values.images.chat.pullPolicy }}

          ports:
            - containerPort: 3004

          envFrom:
            - configMapRef:
                name: streamingapp-config
            - secretRef:
                name: streamingapp-secret

          readinessProbe:
            tcpSocket:
              port: 3004
            initialDelaySeconds: 30
            periodSeconds: 10
            timeoutSeconds: 2
            failureThreshold: 6

          livenessProbe:
            tcpSocket:
              port: 3004
            initialDelaySeconds: 60
            periodSeconds: 20
            timeoutSeconds: 2
            failureThreshold: 3

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


# ============================================================
# FRONTEND SERVICE
# ============================================================

---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: frontend
spec:
  replicas: {{ .Values.replicaCount.frontend }}

  selector:
    matchLabels:
      app: frontend

  strategy:
    type: RollingUpdate
    rollingUpdate:
      maxUnavailable: 0
      maxSurge: 1

  template:
    metadata:
      labels:
        app: frontend

    spec:
      containers:
        - name: frontend
          image: "{{ .Values.images.frontend.repository }}:{{ .Values.images.frontend.tag }}"
          imagePullPolicy: {{ .Values.images.frontend.pullPolicy }}

          ports:
            - containerPort: 80

          readinessProbe:
            tcpSocket:
              port: 80
            initialDelaySeconds: 15
            periodSeconds: 10
            timeoutSeconds: 2
            failureThreshold: 6

          livenessProbe:
            tcpSocket:
              port: 80
            initialDelaySeconds: 30
            periodSeconds: 20
            timeoutSeconds: 2
            failureThreshold: 3

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
'@ | Set-Content helm\streamingapp\templates\app-services.yaml 

------------ 

8. Create templates/mongo-statefulset.yaml
Run:

@'
apiVersion: v1
kind: Service
metadata:
  name: mongo
spec:
  clusterIP: None
  selector:
    app: mongo
  ports:
    - port: 27017
      targetPort: 27017

---
apiVersion: apps/v1
kind: StatefulSet
metadata:
  name: mongo
spec:
  serviceName: mongo
  replicas: {{ .Values.replicaCount.mongo }}

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
          image: "{{ .Values.images.mongo.repository }}:{{ .Values.images.mongo.tag }}"
          imagePullPolicy: {{ .Values.images.mongo.pullPolicy }}

          ports:
            - containerPort: 27017

          volumeMounts:
            - name: mongo-storage
              mountPath: /data/db

  volumeClaimTemplates:
    - metadata:
        name: mongo-storage
      spec:
        accessModes:
          - ReadWriteOnce
        resources:
          requests:
            storage: 1Gi
'@ | Set-Content helm\streamingapp\templates\mongo-statefulset.yaml 

---------------- 

9. Create templates/ingress.yaml
Run:

@'
{{- if .Values.ingress.enabled }}
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: streamingapp-ingress
  annotations:
    nginx.ingress.kubernetes.io/proxy-http-version: "1.1"
    nginx.ingress.kubernetes.io/proxy-read-timeout: "3600"
    nginx.ingress.kubernetes.io/proxy-send-timeout: "3600"
    nginx.ingress.kubernetes.io/proxy-buffering: "off"

spec:
  ingressClassName: {{ .Values.ingress.className }}

  rules:
    - {{- if .Values.ingress.host }}
      host: {{ .Values.ingress.host }}
      {{- end }}
      http:
        paths:

          - path: /
            pathType: Prefix
            backend:
              service:
                name: frontend
                port:
                  number: 80

          - path: /api/auth
            pathType: Prefix
            backend:
              service:
                name: auth
                port:
                  number: 3001

          - path: /api/streaming
            pathType: Prefix
            backend:
              service:
                name: streaming
                port:
                  number: 3002

          - path: /api/admin
            pathType: Prefix
            backend:
              service:
                name: admin
                port:
                  number: 3003

          - path: /api/chat
            pathType: Prefix
            backend:
              service:
                name: chat
                port:
                  number: 3004
{{- end }}
'@ | Set-Content helm\streamingapp\templates\ingress.yaml 

-------------------- 

10. Verify that all files were created

Run: Get-ChildItem helm\streamingapp -Recurse 

<img width="503" height="361" alt="image" src="https://github.com/user-attachments/assets/5558b71a-fb40-42c0-b401-f432ed071a92" />



------------------------
=============== 

11. Run Helm validation

 Run: helm lint helm\streamingapp 

 <img width="433" height="74" alt="image" src="https://github.com/user-attachments/assets/f204582e-b963-49ba-adad-497ec1a8adb7" />

Run: helm template streamingapp helm\streamingapp 

<img width="450" height="251" alt="image" src="https://github.com/user-attachments/assets/efd6fc17-f350-4101-8348-5f2f0ba02704" />


Run: helm install streamingapp helm\streamingapp 


<img width="659" height="84" alt="image" src="https://github.com/user-attachments/assets/975f6ade-d330-46b2-9a79-b680273db3f8" />

Notes: 
The helm install failure is not a problem with the chart. It happened because streamingapp-secret already exists from our kubectl deployment and Helm refuses to take ownership of it without Helm ownership metadata.

================= 

Step 1 — Capture final Kubernetes evidence

Run: kubectl get pods,svc,ingress

<img width="610" height="309" alt="image" src="https://github.com/user-attachments/assets/00e67894-2bfb-4e00-820b-80331994ab72" />


Run: kubectl get deployments

<img width="391" height="133" alt="image" src="https://github.com/user-attachments/assets/57a55114-4df7-428b-8bf5-c39169490f62" />


Run: kubectl rollout status deployment/streaming


<img width="449" height="46" alt="image" src="https://github.com/user-attachments/assets/739d5dad-dcd3-4362-8461-f636d263277e" />

Run: helm lint helm\streamingapp



<img width="464" height="224" alt="image" src="https://github.com/user-attachments/assets/425a5559-de7c-4e6b-8eb9-b73040d3bef1" /> 

========================= 

Step 2 — Verify the application

Open: http://localhost


<img width="1920" height="1080" alt="image" src="https://github.com/user-attachments/assets/2664935e-a794-4ce9-9091-fda57e26fca8" /> 

=========================== 

Step 3 — Test self-healing 


---------------------------- 

First get the streaming Pods:


Run: kubectl get pods -l app=streaming


<img width="374" height="77" alt="image" src="https://github.com/user-attachments/assets/9b8e1989-e5f9-4dd0-9f9f-ff48e8fffb39" />

------------------------- 

Choose one of the three Pod names and delete it: 

Run: kubectl delete pod streaming-5b44ffd75-nlkqx 


<img width="484" height="39" alt="image" src="https://github.com/user-attachments/assets/d50cccf4-912f-4218-8119-ae1b1c673ddd" />

--------------------------- 

Then immediately:

Run: kubectl get pods -l app=streaming



<img width="456" height="87" alt="image" src="https://github.com/user-attachments/assets/79f4c95e-14ba-494b-be33-20436e47ee6b" />

----------------------------- 

Kubernetes creates a replacement.

Run: kubectl get pods -l app=streaming


<img width="401" height="94" alt="image" src="https://github.com/user-attachments/assets/ac46a14c-f3b7-47e4-9566-3c26061740a3" />

---------------------- 

This gives us evidence for self-healing + replica management. 

So self-healing is successfully demonstrated: the Deployment maintained the desired 3 replicas after a pod was manually deleted. 

======================== 

Phase 9 — Docker Hub 

Step 9.1 — Confirm the images 


Run: docker images | findstr streamingapp


<img width="521" height="174" alt="image" src="https://github.com/user-attachments/assets/9536c66b-168d-432e-8d6a-0991b32bd027" />

=============== 

Step 9.2 — Login to Docker Hub
Run: docker login


<img width="635" height="111" alt="image" src="https://github.com/user-attachments/assets/a26e1925-fc12-46c5-9a29-ddeb2a2f1410" />

----------------------------- 

Step 9.4 — Tag the images

---------------- 

Run this exact block:
docker tag streamingapp-auth:latest sandhya1812/streamingapp-auth:1.0.0
docker tag streamingapp-streaming:latest sandhya1812/streamingapp-streaming:1.0.0
docker tag streamingapp-admin:latest sandhya1812/streamingapp-admin:1.0.0
docker tag streamingapp-chat:latest sandhya1812/streamingapp-chat:1.0.0
docker tag streamingapp-frontend:latest sandhya1812/streamingapp-frontend:1.0.0 

Run:
docker images | findstr sandhya1812


<img width="664" height="272" alt="image" src="https://github.com/user-attachments/assets/41605840-99a0-404a-9e7f-1265517c37db" />

Run: 
docker tag streamingapp-auth:latest sandhya1812/streamingapp-auth:1.0.0
docker images | findstr sandhya1812  


<img width="645" height="170" alt="image" src="https://github.com/user-attachments/assets/3a2d5ffc-be04-4491-8dbf-8e7b9d88d455" />

===================== 

Phase 9 — Step 9.3: Push to Docker Hub 



Run:
docker push sandhya1812/streamingapp-auth:1.0.0
docker push sandhya1812/streamingapp-streaming:1.0.0
docker push sandhya1812/streamingapp-admin:1.0.0
docker push sandhya1812/streamingapp-chat:1.0.0
docker push sandhya1812/streamingapp-frontend:1.0.0


<img width="584" height="319" alt="image" src="https://github.com/user-attachments/assets/1b1e4e00-4cbc-4269-8a73-90d45e6d555e" />



<img width="649" height="317" alt="image" src="https://github.com/user-attachments/assets/fe192fce-b13b-4cf1-9655-03eede03810c" />


<img width="677" height="186" alt="image" src="https://github.com/user-attachments/assets/824ab59b-3fa7-420e-a219-55fbf79cac6f" />


================================= 

Phase 10 — AWS ECR 

=========================== 

Step 10.1 — Confirm AWS account and region

Run: aws sts get-caller-identity 


<img width="424" height="78" alt="image" src="https://github.com/user-attachments/assets/fdbc1574-00d7-4b77-a3d0-74a1ccfe72f6" />



Run: aws configure get region


<img width="374" height="45" alt="image" src="https://github.com/user-attachments/assets/4656a09a-b22c-440e-b2f6-07b8929fb986" />


=============================== 

Step 10.2 — Create the five ECR repositories

Run:

aws ecr create-repository --repository-name streamingapp-auth --region us-east-1
aws ecr create-repository --repository-name streamingapp-streaming --region us-east-1
aws ecr create-repository --repository-name streamingapp-admin --region us-east-1
aws ecr create-repository --repository-name streamingapp-chat --region us-east-1
aws ecr create-repository --repository-name streamingapp-frontend --region us-east-1

----------------- 

Then verify all five

Run:

aws ecr describe-repositories --region us-east-1 --query "repositories[].repositoryUri" --output table


<img width="653" height="193" alt="image" src="https://github.com/user-attachments/assets/8bf1b589-be5a-4a77-baf7-90e68e478366" />

========================= 

Step 10.3 — Authenticate Docker with Amazon ECR
Run:

aws ecr get-login-password --region us-east-1 | docker login --username AWS --password-stdin 759910672539.dkr.ecr.us-east-1.amazonaws.com


======================= 

Phase 10 — Step 10.4: Tag images for ECR

=============================== 

Run: 
docker tag sandhya1812/streamingapp-auth:1.0.0 759910672539.dkr.ecr.us-east-1.amazonaws.com/streamingapp-auth:1.0.0
docker tag sandhya1812/streamingapp-streaming:1.0.0 759910672539.dkr.ecr.us-east-1.amazonaws.com/streamingapp-streaming:1.0.0
docker tag sandhya1812/streamingapp-admin:1.0.0 759910672539.dkr.ecr.us-east-1.amazonaws.com/streamingapp-admin:1.0.0
docker tag sandhya1812/streamingapp-chat:1.0.0 759910672539.dkr.ecr.us-east-1.amazonaws.com/streamingapp-chat:1.0.0
docker tag sandhya1812/streamingapp-frontend:1.0.0 759910672539.dkr.ecr.us-east-1.amazonaws.com/streamingapp-frontend:1.0.0 

<img width="672" height="131" alt="image" src="https://github.com/user-attachments/assets/22fd4b7a-d60d-4224-8eea-0de5658b57ba" />


Verify the tags
Run: docker images | findstr 759910672539

<img width="523" height="170" alt="image" src="https://github.com/user-attachments/assets/6c32eacb-05e4-475e-84f7-c356223816a7" />

=============================== 

Step 10.5 — Push the five images 

================================ 

Run:
docker push 759910672539.dkr.ecr.us-east-1.amazonaws.com/streamingapp-auth:1.0.0
docker push 759910672539.dkr.ecr.us-east-1.amazonaws.com/streamingapp-streaming:1.0.0
docker push 759910672539.dkr.ecr.us-east-1.amazonaws.com/streamingapp-admin:1.0.0
docker push 759910672539.dkr.ecr.us-east-1.amazonaws.com/streamingapp-chat:1.0.0
docker push 759910672539.dkr.ecr.us-east-1.amazonaws.com/streamingapp-frontend:1.0.0 

<img width="671" height="314" alt="image" src="https://github.com/user-attachments/assets/86e925a1-cd1f-4f79-adb9-1ef9bcbf496e" />



<img width="663" height="312" alt="image" src="https://github.com/user-attachments/assets/32a65e68-9d96-4dcf-b394-5fcd27faaf05" />



<img width="642" height="185" alt="image" src="https://github.com/user-attachments/assets/8ee8e5a9-1182-4f7a-8381-e7f75aa06a55" />


Phase 10 — AWS ECR is now COMPLETE:
All five application images were successfully pushed to Amazon ECR with version 1.0.0:

====================================================== 

Phase 10A — MongoDB Atlas

Step 1 — Open MongoDB Atlas
Open:
MongoDB Atlas
Sign in with your MongoDB account.

==================================== 

Step 10A.2 — Create the Atlas project and cluster

Create a project:
On the Atlas dashboard:
1. Click Projects.
2. Click New Project.
3. Enter: StreamingApp
4. Click Next / Create Project.

<img width="1920" height="1080" alt="image" src="https://github.com/user-attachments/assets/48dae0fd-9920-4e37-b647-e190d0c60875" />

<img width="1920" height="1080" alt="image" src="https://github.com/user-attachments/assets/f88d6469-0a95-4086-bf84-708130f6ccfd" />

=========================== 

Step 3 — Create the MongoDB cluster
Inside the StreamingApp project: 


1. Click Create a Cluster.

   
<img width="1920" height="1080" alt="image" src="https://github.com/user-attachments/assets/3209e72d-3b7d-42fb-841b-54dd0ff121ea" />



2. Select the Free / M0 option.

   <img width="1920" height="1080" alt="image" src="https://github.com/user-attachments/assets/7106275b-a10d-4892-a464-af9e734507fa" />


3. Choose a cloud provider.
For example: AWS

<img width="1920" height="1080" alt="image" src="https://github.com/user-attachments/assets/c144454f-609f-4de3-b3d7-eb6565946db2" />


5. Choose a region reasonably close to your deployment. Since your AWS work is currently in us-east-1, you can select an AWS region available for the free tier there.

<img width="1920" height="1080" alt="image" src="https://github.com/user-attachments/assets/b1da0020-ee11-4131-a890-30f031e8b14c" />



7. Give the cluster a name, for example: StreamingAppCluster

8. Click Create Cluster
9. A dialogue box opens, for username and password, close it and create a user friendly password.:
   . Go to Database Acces, in the left menu, go to:
     Security → Database Access

<img width="1920" height="1080" alt="image" src="https://github.com/user-attachments/assets/a8cc4590-92bf-410c-8cbe-61e018ea7746" />


   . Create our application user:
    . Click: Add New Database User
    . Create Username: streamingapp-app-user1 
    . For authentication, use a password and create a new strong password.

<img width="1920" height="1080" alt="image" src="https://github.com/user-attachments/assets/ff3dbcc3-3198-4e1b-bc45-73f01c592485" /> 

<img width="1920" height="1080" alt="image" src="https://github.com/user-attachments/assets/6425aff5-a4ef-4d52-ab1d-734e9e1292c3" /> 

<img width="1920" height="1080" alt="image" src="https://github.com/user-attachments/assets/f364d252-eec6-4a69-820b-bbd928ea713d" />


<img width="1920" height="1080" alt="image" src="https://github.com/user-attachments/assets/8f5e034d-629f-46ea-9004-c25a2e8f1fe8" />


. Database User Privileges
  choose the option equivalent to: Read and write to any database

. Rest all keep it as it is.
. Then click: Add User


<img width="1920" height="1080" alt="image" src="https://github.com/user-attachments/assets/21ee2e67-89e7-4958-9eb6-7bde20997546" />

============== 

Step 4 — Configure Network Access

4.1 Open IP Access List
In MongoDB Atlas, on the left side, under:
NETWORK ACCESS, click: IP Access List
You should see the IP address that Atlas automatically added when the cluster was created.

<img width="1920" height="1080" alt="image" src="https://github.com/user-attachments/assets/34284988-a2e5-449c-b1a2-f183cec86f61" />

4.3 Add the IP
On the IP Access List page:
1. Click Add IP Address.
2. Let's add temporary access for testing the AWS/EKS deployment.
3. Access List Entry : 
   Enter: 0.0.0.0/0
4. Comment
   Enter: StreamingApp development and EKS testing

5. Temporary access
   For initial testing, turn the temporary entry switch ON and select a suitable duration, such as 1 week, if available. This will ensure the broad access entry expires automatically.


<img width="1920" height="1080" alt="image" src="https://github.com/user-attachments/assets/be73d420-b381-4700-a16b-0067d4cb981f" />

6. Confirm
   Click the green Confirm button.


   <img width="1920" height="1080" alt="image" src="https://github.com/user-attachments/assets/02dbb9c4-e2ea-42e5-b5fd-9b3bf3bee1ad" />


=============================== 

Phase 10A — Step 5: Get the MongoDB connection string
Now let's get the connection string for our streamingapp-app-user1 user. 

----------------------- 

5.1 Go to Clusters
On the left side, click: Clusters
You should see: StreamingAppCluster

<img width="1920" height="1080" alt="image" src="https://github.com/user-attachments/assets/b91a8471-577b-4152-ba64-db9c852cb361" />



---------------------------- 

5.2 Click Connect
On the StreamingAppCluster row, click: Connect 

A window will appear: Connect to StreamingAppCluster  


<img width="1920" height="1080" alt="image" src="https://github.com/user-attachments/assets/ab7e753d-e2b7-447f-bf6d-5f6671277c14" />

-------------------- 

5.3 Choose Drivers
    Select: Drivers 

<img width="1920" height="1080" alt="image" src="https://github.com/user-attachments/assets/cf44094a-1f73-4408-afa1-1e39cf4f9a81" />


. Keep Language: JavaScript
. Client Library: Node.js Driver

. Copy the Atlas connection string :
mongodb+srv://<db_username>:<db_password>@streamingappcluster.se9p9lu.mongodb.net/?appName=StreamingAppCluster

. Replace: <db_username> --> with: streamingapp-app-user1

. Replace: <db_password> --> with the new password we created for streamingapp-app-user1.

. Then change the part:.mongodb.net/?appName= --> to:.mongodb.net/streamingapp?retryWrites=true&w=majority


Connection string is:

mongodb+srv://streamingapp-app-user1:<db_password>@streamingappcluster.se9p9lu.mongodb.net/streamingapp?retryWrites=true&w=majority 

Note replace the <db_password> with the actual password

-------------------------- 

Notes:

Why streamingapp?
Because all the four backend services should use the same database named streamingapp.    

==============================================

Phase 10A — Step 6: Test the Atlas Connection

==================================================

. we shall test that the streamingapp-app-user1 credentials can actually connect to StreamingAppCluster.

. Since our project is Node.js-based, we'll test it from your Windows/PowerShell environment.

----------------------------------------------- 

6.1 Check Node.js/npm
Because the project uses Node.js:

Run:
npm.cmd --version 

Run:
Get-Content .\backend\authService\package.json | Select-String "mongoose|mongodb"


<img width="635" height="116" alt="image" src="https://github.com/user-attachments/assets/b8e91d56-0050-45e2-9823-2f7baf1f8e82" />

----------------------------- 

Step 6.2 - Install Auth Service dependencies

============================= 

Step 6.2.1 — Create a temporary test file

In PowerShell,
Run: notepad atlas-test.js 

paste the below code:

const mongoose = require("mongoose");

const uri = "PASTE_YOUR_ATLAS_CONNECTION_STRING_HERE";

mongoose.connect(uri)
  .then(() => {
    console.log("MongoDB Atlas connection SUCCESSFUL");
    return mongoose.disconnect();
  })
  .then(() => {
    console.log("MongoDB connection closed");
  })
  .catch((err) => {
    console.error("MongoDB Atlas connection FAILED");
    console.error(err.message);
    process.exit(1);
  }); 

  ======================= 

Step 6.2.2 — Put your connection string in the file
Replace: PASTE_YOUR_ATLAS_CONNECTION_STRING_HERE --> with your actual Atlas connection string. 
and Save.


--------------------------------------- 


Step 6.2.3  — Install the Auth Service dependencies

Run:
cd C:\Users\sandy\StreamingApp\backend\authService 

Run:
npm.cmd install

<img width="670" height="305" alt="image" src="https://github.com/user-attachments/assets/df608672-9916-411d-b505-71dcfba22d6d" />

-------------------------- 

Step 6.3 - Verify Mongoose

Run:
dir node_modules\mongoose 

<img width="539" height="253" alt="image" src="https://github.com/user-attachments/assets/93ac304e-ba24-4959-bd44-c0cc4d950dbf" />

--------- 

Step 6.4 - Create a temporary Atlas connection test

Return to the project root:
cd C:\Users\sandy\StreamingApp 

------------ 

Create the temporary file:

Run: notepad atlas-test.js  

replace with below code:

const mongoose = require("./backend/authService/node_modules/mongoose");

const uri = "YOUR_ATLAS_CONNECTION_STRING";

mongoose.connect(uri)
  .then(() => {
    console.log("MongoDB Atlas connection SUCCESSFUL");
    return mongoose.disconnect();
  })
  .then(() => {
    console.log("MongoDB connection closed");
  })
  .catch((err) => {
    console.error("MongoDB Atlas connection FAILED");
    console.error(err.message);
    process.exit(1);
  }); 

  ====================  

  . Replace YOUR_ATLAS_CONNECTION_STRING with the private Atlas connection string.
  . Save and close Notepad 

------------------------ 

Step 6.5 - Run the Atlas connection test

Run: node atlas-test.js

Error: 

<img width="482" height="53" alt="image" src="https://github.com/user-attachments/assets/14024a6c-b2cf-4ee5-98cc-549ddd929c2f" />

-------------------------- 

Step 6.6 — Test DNS from PowerShell
Run this:
nslookup streamingappcluster.se9p9lu.mongodb.net


<img width="563" height="295" alt="image" src="https://github.com/user-attachments/assets/2a1fcb67-c5e0-4c3f-a8db-f7ce76789c82" />

------------------------- 

<img width="652" height="259" alt="image" src="https://github.com/user-attachments/assets/634b10b8-912b-48f5-8f1f-e0add90c3725" /> 


<img width="494" height="268" alt="image" src="https://github.com/user-attachments/assets/491a6850-96b1-4a4b-ae19-af7c537ec4f2" />



<img width="596" height="147" alt="image" src="https://github.com/user-attachments/assets/60be4639-9914-4986-b405-e82292a7a10c" />  


--------------------------------------- 


Step 6.7 - Test Atlas TXT DNS record 

Run: nslookup -type=TXT streamingappcluster.se9p9lu.mongodb.net 8.8.8.8



<img width="671" height="256" alt="image" src="https://github.com/user-attachments/assets/67ff7222-fb84-4213-b54f-6b475382f906" />


<img width="576" height="125" alt="image" src="https://github.com/user-attachments/assets/38391524-96ee-443d-91de-5206538d0600" />

-------------------------- 

6.8 Test direct connectivity to Atlas
Use one of the Atlas hosts returned by the SRV lookup:

Run: Test-NetConnection ac-wrndmtd-shard-00-01.se9p9lu.mongodb.net -Port 27017


<img width="623" height="113" alt="image" src="https://github.com/user-attachments/assets/dd581e59-c3c7-4100-a80c-4649d2b3954b" />


Note: If it returns False, test the other Atlas nodes:
Run: 
Test-NetConnection ac-wrndmtd-shard-00-00.se9p9lu.mongodb.net -Port 27017 
Test-NetConnection ac-wrndmtd-shard-00-02.se9p9lu.mongodb.net -Port 27017


----------------------- 

6.9 Test general internet connectivity
We also verified that normal HTTPS connectivity works: 

Run: Test-NetConnection google.com -Port 443

Our result was:
TcpTestSucceeded : True 
So general internet connectivity is working.

---------------------- 

Therefore, we temporarily bypassed the SRV lookup for diagnostic purposes.

Phase 10A — Step 6.10: Test Using Direct MongoDB Hosts

Lets Verify that the StreamingApp backend can connect to the MongoDB Atlas cluster using Node.js and Mongoose.

From the successful SRV lookup, we obtained:
ac-wrndmtd-shard-00-00.se9p9lu.mongodb.net
ac-wrndmtd-shard-00-01.se9p9lu.mongodb.net
ac-wrndmtd-shard-00-02.se9p9lu.mongodb.net

The temporary connection string:

mongodb://streamingapp-app-user1:YOUR_PASSWORD@ac-wrndmtd-shard-00-00.se9p9lu.mongodb.net:27017,ac-wrndmtd-shard-00-01.se9p9lu.mongodb.net:27017,ac-wrndmtd-shard-00-02.se9p9lu.mongodb.net:27017/streamingapp?authSource=admin&replicaSet=atlas-12tkog-shard-0&tls=true&retryWrites=true&w=majority
Run: node atlas-test.js

Correct the Temporary Test File
We will use: atlas-test.js

<img width="323" height="26" alt="image" src="https://github.com/user-attachments/assets/16787d50-fa67-4427-ac50-027b99339203" />

Paste the below code in Test file:
const mongoose = require("./backend/authService/node_modules/mongoose");

const uri = "YOUR_ATLAS_CONNECTION_STRING";

mongoose.connect(uri)
  .then(() => {
    console.log("MongoDB Atlas connection SUCCESSFUL");
    return mongoose.disconnect();
  })
  .then(() => {
    console.log("MongoDB connection closed");
  })
  .catch((err) => {
    console.error("MongoDB Atlas connection FAILED");
    console.error(err.message);
    process.exit(1);
  }); 

  ------------- 

Step 6.11 - Correct the Temporary Test File

  Run: node atlas-test.js

------------------------------- 

Step 6.12 — Successful Atlas Connection

---------------------------------- 

Step 6.13 — Remove Temporary Test File

  Remove Temporary Test File:
  Run: Remove-Item .\atlas-test.js

  Verify that it is gone:
  Run: Test-Path .\atlas-test.js


<img width="359" height="87" alt="image" src="https://github.com/user-attachments/assets/9a4420d1-76cb-4505-98b6-2cfc3ef95eb2" />


====================================== 

PHASE 10B — AWS S3 

============================= 

objective: 
Configure Amazon S3 for the StreamingApp's:
- Video uploads
- Thumbnail uploads
- Video playback/storage
The Admin Service and Streaming Service need the AWS S3 configuration for these operations.


--------------------------------- 

Step 10B.1 — Create the S3 bucket
We will create the bucket in us-east-1, because our ECR and EKS environment are in us-east-1.
1. Open AWS Console
Go to:
AWS Console → S3 → Create bucket

<img width="1920" height="1080" alt="image" src="https://github.com/user-attachments/assets/22ab56a0-99f8-40cb-8fb1-da7e447e236d" />



3. Bucket name
Use:

<img width="1920" height="1080" alt="image" src="https://github.com/user-attachments/assets/d5bc888c-d1d9-49c0-904a-45d0ff017fc9" />

<img width="1920" height="1080" alt="image" src="https://github.com/user-attachments/assets/fc3aea29-4f1f-40ac-85ab-0e4014abe016" />

<img width="1920" height="1080" alt="image" src="https://github.com/user-attachments/assets/44c87d7e-86ef-48b3-a366-dc84339e9424" />

<img width="1920" height="1080" alt="image" src="https://github.com/user-attachments/assets/2a42e237-47c5-4960-8449-fa6856b8c490" /> 

<img width="1920" height="1080" alt="image" src="https://github.com/user-attachments/assets/8218e457-5cba-4169-9f38-7b7de3ea9382" />

<img width="1920" height="1080" alt="image" src="https://github.com/user-attachments/assets/c9901d84-c6dc-48b7-82b2-cc46872498d9" />


<img width="1920" height="1080" alt="image" src="https://github.com/user-attachments/assets/ab179753-7a3b-4217-b878-d3678ec292be" />


Click Create Bucket:

<img width="1920" height="1080" alt="image" src="https://github.com/user-attachments/assets/b93216e8-e83f-48d0-89fc-143f3bd9e17b" />



























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
