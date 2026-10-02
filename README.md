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



 
========================================
****************************************
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
