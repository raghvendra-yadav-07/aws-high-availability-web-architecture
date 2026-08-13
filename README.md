# 🚀 High-Availability Web Architecture on AWS

An enterprise-ready, multi-tier web application architecture deployed on AWS. The system features DNS routing via **Route 53**, load balancing across high-availability **EC2 instances**, a **Node.js/Express** backend, and a managed **AWS RDS MySQL** database.

---

## 🏛️ System Architecture

```text
               [ Users / Web Browsers ]
                          │
                          ▼
             [ AWS Route 53 (DNS / Domain) ]
                          │
                          ▼
        [ AWS Application Load Balancer (ALB) ]
                          │
            ┌─────────────┴─────────────┐
            │ (Port 80/443 Routing)     │
            ▼                           ▼
  [ EC2 Instance 1 ]          [ EC2 Instance 2 ]
  (Frontend / Nginx)          (Frontend / Nginx)
            │                           │
            └─────────────┬─────────────┘
                          │ (Internal VPC / Port 3000)
                          ▼
             [ Express API / Backend ]
                          │
                          ▼
            [ AWS RDS MySQL Database ]
```

---

## ✨ Key Architectural Highlights

* **DNS Management & Alias Routing:** Integrated **Route 53** with ALB via A-Alias records for zero-latency IP resolution.
* **Traffic Distribution & Fault Tolerance:** Configured an **Application Load Balancer (ALB)** with health checks to route traffic across redundant **EC2 instances**.
* **Reverse Proxying:** Utilized **Nginx** on frontend nodes to serve static content and proxy `/api/` requests internally across the VPC.
* **Process Management:** Managed backend **Node.js / Express** instances with **PM2** for zero-downtime reloads and automatic crash restarts.
* **Managed Data Tier:** Migrated state to **AWS RDS MySQL** for automated backups, scaling, and data persistence.
* **Least-Privilege Security:** Configured strict **AWS Security Group chains** ensuring RDS is only accessible from the backend application tier.

---

## 🛠️ Tech Stack & Infrastructure

* **Cloud Provider:** AWS (EC2, ALB, Route 53, RDS MySQL, Security Groups, VPC)
* **Web / Proxy Server:** Nginx
* **Backend Runtime:** Node.js, Express.js
* **Process Manager:** PM2
* **Database:** AWS RDS (MySQL 8.0)
* **Operating System:** Ubuntu 24.04 LTS

---

## 🚦 Application Data Flow

1. **DNS Resolution:** User requests `https://yourdomain.com`. Route 53 resolves the domain directly to the ALB's canonical DNS name.
2. **Load Balancing:** The ALB distributes incoming requests across the **EC2 instance fleet** based on target group health.
3. **Frontend / Proxying:** Nginx receives the request on Port 80, serves static assets (`index.html`), and proxies `/api/*` endpoints to Node.js on Port 3000.
4. **Backend Processing:** Express processes the API logic and performs parameterized SQL queries against the **AWS RDS MySQL** instance.
5. **Database Response:** RDS returns query results back through the VPC isolation boundary to the user interface.

---

## 🔧 Environment & Deployment Setup

### 1. Database Setup (AWS RDS)
* Created a MySQL instance in private subnets.
* Security Group Rule: Allow Inbound TCP **Port 3306** solely from EC2 Security Group (`sg-ec2-backend`).

### 2. Backend Instance Setup (`/home/ubuntu`)
```bash
# Move to the application directory containing uploaded files
cd /home/ubuntu
npm install

# Environment variables (.env)
DB_HOST=your-rds-endpoint.amazonaws.com
DB_USER=admin
DB_PASS=yourpassword
DB_NAME=production_db
PORT=3000

# Start background process with PM2
pm2 start server.js --name "express-backend"
pm2 save
```

### 3. Frontend Instance Setup (Nginx)
`/etc/nginx/sites-available/default` configuration:
```nginx
server {
    listen 80;
    server_name _;

    root /var/www/html;
    index index.html;

    location / {
        try_files $uri $uri/ =404;
    }

    location /api/ {
        proxy_pass http://10.0.158.41:3000/api/;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
    }
}
```

---

## 🧪 Verification & Testing

Verify end-to-end connectivity:

```bash
# Test ALB Routing
curl -i http://<ALB-DNS-NAME>/api/entries

# Test Domain Resolution
dig +short yourdomain.com
```
