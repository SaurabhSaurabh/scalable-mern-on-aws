# scalable-mern-on-aws
Deployment of the TravelMemory MERN application on AWS EC2 with Nginx reverse proxy, load balancing, and Cloudflare domain integration for scalability and resilience.

# TravelMemory MERN deployment on AWS EC2 with ALB and Cloudflare

## Problem statement
**Goal:** Deploy the MERN-based TravelMemory app to AWS EC2, ensure frontend–backend communication, scale via an AWS Load Balancer, and expose a custom domain through Cloudflare with secure and resilient architecture.

**Introduction:**

The Travel Memory application has been developed using the MERN stack. Your challenge is to deploy this application on an Amazon EC2 instance. This will provide you with hands-on experience in deploying full-stack applications, working with cloud platforms, and ensuring scalable architecture.

**Project Repository:**

Access the complete codebase of the TravelMemory application from the provided GitHub link: https://github.com/UnpredictablePrashant/TravelMemory

**Objective:**

- Set up the backend running on Node.js.

- Configure the front end designed with React.

- Ensure efficient communication between the front end and back end.

- Deploy the full application on an EC2 instance.

- Facilitate load balancing by creating multiple instances of the application.

- Connect a custom domain through Cloudflare.

 **Tasks:**

1. Backend Configuration:

- Clone the repository and navigate to the backend directory.

- The backend runs on port 3000. Set up a reverse proxy using nginx to ensure smooth deployment on EC2.

- Update the .env file to incorporate database connection details and port information.

2. Frontend and Backend Connection:

- Navigate to the `urls.js` in the frontend directory.

 - Update the file to ensure the front end communicates effectively with the backend.

3. Scaling the Application:

- Create multiple instances of both the frontend and backend servers.

- Add these instances to a load balancer to ensure efficient distribution of incoming traffic.

4. Domain Setup with Cloudflare:

- Connect your custom domain to the application using Cloudflare.

- Create a CNAME record pointing to the load balancer endpoint.

- Set up an A record with the IP address of the EC2 instance hosting the front end.

5. Documentation:

- Prepare comprehensive documentation detailing each step of the deployment process. Include relevant screenshots to make the process clear and reproducible.

- Design a deployment architecture diagram using [draw.io](https://www.draw.io/) to visualize the flow and connections.

**Expected Deliverables:**

1. A fully functional Travel Memory application hosted on an EC2 instance, accessible via a custom domain.

2. Detailed documentation of the deployment process with appropriate screenshots.

3. A deployment architecture diagram.

---

## Steps to resolve

### 1️⃣ Provision EC2 and Install Prerequisites

#### 🖥️ Operating System & Packages
- Ubuntu 22.04 LTS
- Nginx
- Node.js (v18 LTS)
- PM2
- Git

#### 🌐 Network Configuration
Security Group must allow:
| Port | Purpose        |
|------|----------------|
| 22   | SSH            |
| 80   | HTTP           |
| 443  | HTTPS          |
| 3000 | Frontend (React) |
| 3001 | Backend (Node.js) |

#### 📋 Steps
1. Log into **AWS Management Console** → EC2 Dashboard → **Launch Instance**.  
2. Select **Ubuntu Server 20.04 LTS** (Free tier eligible).  
3. Choose instance type: `t2.micro`.  
4. Configure default VPC + public subnet.  
5. Allocate storage (default 8 GB).  
6. Configure **Security Group** with ports listed above.  
7. Launch with a key pair for SSH access.  

```
# Example SSH command to connect
ssh -i your-key.pem ubuntu@EC2_PUBLIC_IP
```
> Screenshot: EC2 instance list  
> Screenshot: Security group rules

---

### 2. Update EC2 and Install Node.js 22
SSH into your instance:
```bash
ssh -i your-key.pem ubuntu@EC2_PUBLIC_IP
```

Update packages: ```sudo apt update && sudo apt upgrade -y```

Install Node.js 22 + npm:
```
curl -fsSL https://deb.nodesource.com/setup_22.x | sudo -E bash -
sudo apt install -y nodejs
```
Verify installation:
```
node -v
npm -v
```
Screenshot: Node.js and npm versions

---

### 3. Configure MongoDB Atlas
Sign in to MongoDB Atlas.

Create a free-tier cluster (M0) on AWS in your nearest region.

Add Database User with read/write privileges.

Configure Network Access: Allow from Anywhere (0.0.0.0/0) or restrict to EC2 IP.

Download and install MongoDB Compass.

Connect using URI:
```
mongodb+srv://<username>:<password>@cluster0.mongodb.net/travelmemory?retryWrites=true&w=majority
```

Create database travelmemory and collection trips.

Screenshot: Atlas cluster dashboard Screenshot: Compass showing travelmemory database

---

### 4. Deploy Backend Application

Clone the repository:
```
git clone https://github.com/UnpredictablePrashant/TravelMemory.git
cd TravelMemory/backend
```
Create .env file:
```
MONGO_URI='mongodb+srv://<username>:<password>@cluster0.mongodb.net/travelmemory?retryWrites=true&w=majority'
PORT=3001
JWT_SECRET=<your_secret>
CLIENT_ORIGIN=http://EC2_PUBLIC_IP:3000
```
Install dependencies: ``` npm install ```  
Run backend server: ```node index.js ```   
Test API endpoint: ```http://EC2_PUBLIC_IP:3001/hello```   


** Expected output: ** Hello World

Screenshot: .env file (redacted) Screenshot: Browser showing Hello World

---

### 5. Deploy Frontend Application
Navigate to frontend folder:
```cd TravelMemory/frontend```

Create .env file:
```echo "REACT_APP_BACKEND_URL=http://EC2_PUBLIC_IP:3001" > .env```

Install dependencies:
```npm install```

Run React app:
```npm start```

Access frontend:
``` http://EC2_PUBLIC_IP:3000 ```

Screenshot: Frontend homepage in browser

---

### 6. Next Steps (Production Deployment)

Nginx Reverse Proxy: Serve React build (npm run build) and proxy backend requests.

Load Balancer: Create AWS Application Load Balancer, register multiple EC2 instances, configure health checks (/hello).

Cloudflare DNS:

CNAME api.yourdomain.com → ALB DNS name.

A record yourdomain.com → EC2 public IP (or frontend ALB).

TLS Certificates: Use AWS ACM + Cloudflare Full/Strict mode.

Screenshot: ALB target group health check Screenshot: Cloudflare DNS records

---

Implementations

Backend .env
```
MONGO_URI='mongodb+srv://<username>:<password>@cluster0.mongodb.net/travelmemory?retryWrites=true&w=majority'
PORT=3001
JWT_SECRET=<your_secret>
CLIENT_ORIGIN=http://EC2_PUBLIC_IP:3000
```

Frontend .env
```
REACT_APP_BACKEND_URL=http://EC2_PUBLIC_IP:3001
```

---

## Implementations

### Backend PM2
```
pm2 start server.js --name travelmemory-backend
pm2 save
sudo env PATH=$PATH:/usr/bin pm2 startup systemd -u $USER --hp $HOME
```

### Nginx backend (reverse proxy)
```
server {
  listen 80;
  server_name api.yourdomain.com;
  location / {
    proxy_pass http://127.0.0.1:3000;
    proxy_set_header Host $host;
    proxy_set_header X-Real-IP $remote_addr;
    proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
    proxy_set_header X-Forwarded-Proto $scheme;
  }
}
```

### Nginx frontend (static hosting)
```
server {
  listen 80;
  server_name yourdomain.com www.yourdomain.com;
  root /var/www/travelmemory;
  index index.html;

  location / {
    try_files $uri /index.html;
  }
}
```

### Frontend urls.js
``` -javascript-
const API_BASE = process.env.REACT_APP_API_BASE_URL || 'http://localhost:3000';
export const ENDPOINTS = {
  AUTH_LOGIN: `${API_BASE}/auth/login`,
  AUTH_REGISTER: `${API_BASE}/auth/register`,
  TRIPS: `${API_BASE}/trips`,
};
```
---

## Learnings from the assignment
- Separation of concerns: Nginx handles public traffic and static files; Node handles API logic behind a proxy.

- Environment-driven builds: React reads REACT_APP_* vars at build time—bake correct API URL for production.

- Health checks matter: Simple endpoints like /hello or /health let ALB judge instance health and keep traffic resilient.

- DNS correctness: CNAME for LB endpoints; A records for direct instance IPs (or use CNAME if frontend is also behind ALB).

- Security posture: Terminate TLS at ALB with ACM, never expose Node directly, keep secrets off Git, and restrict SG ingress.

  ---

## Deep-dive links
- AWS ALB basics and target groups: Official AWS docs (search “Application Load Balancer user guide”)

- Nginx reverse proxy best practices: Nginx docs (search “nginx reverse proxy headers”)

- React env vars (REACT_APP_*): CRA documentation

- MongoDB Atlas IP allowlist and connection strings: MongoDB Atlas docs

- Cloudflare DNS and SSL modes: Cloudflare docs (search “Full vs Full (Strict)”)

- Assignment portal and repository references used for task scope and links. Public validation pages referenced for example outputs during deployment54.183.207.178+1.

