
# MERN Stack Application Deployment Guide with NGINX and SSL (Let's Encrypt)

This guide provides the steps to set up and deploy a **MERN** (MongoDB, Express, React, Node) stack application on a server with **NGINX** as a reverse proxy and **SSL** (using **Let's Encrypt**) for secure HTTPS.

## Prerequisites

- A **VPS** (e.g., AWS, DigitalOcean, Linode) running **Ubuntu 20.04** or later.
- A **domain name** (e.g., `brainbloom.sbs`) pointing to your server's IP address.
- **Node.js** and **npm** installed on the server.

## 1. Set Up the Server

1. **Provision Your Server (VPS)** using your preferred cloud provider (AWS, DigitalOcean, etc.).
2. **Update the system packages**:
   ```bash
   sudo apt update
   sudo apt upgrade
   ```

## 2. Install NGINX

1. **Install NGINX**:
   ```bash
   sudo apt install nginx
   ```

2. **Start and Enable NGINX**:
   ```bash
   sudo systemctl start nginx
   sudo systemctl enable nginx
   ```

## 3. Set Up the Backend (Node.js + Express)

1. **Install Node.js and npm**:
   ```bash
   sudo apt install nodejs npm
   ```

2. **Create Your Express App**:
   - Navigate to your project folder and initialize a new Node.js project:
   ```bash
   mkdir myapp
   cd myapp
   npm init -y
   ```

3. **Install Express and other dependencies**:
   ```bash
   npm install express mongoose cors
   ```

4. **Create `index.js`** for your Express server:
   ```javascript
   const express = require('express');
   const cors = require('cors');

   const app = express();

   app.use(cors());
   app.use(express.json());

   app.get('/api/hello', (req, res) => {
       res.send({ message: 'Hello World' });
   });

   app.listen(5000, () => {
       console.log('Backend is running on port 5000');
   });
   ```

5. **Run the Backend**:
   ```bash
   node index.js
   ```

Now, your API should be available at `http://localhost:5000`.

## 4. Set Up the Frontend (React)

1. **Create a React App**:
   - In a new directory, create a React application:
   ```bash
   npx create-react-app frontend
   cd frontend
   ```

2. **Build the React App for Production**:
   ```bash
   npm run build
   ```

3. **Move the React Build to Your Server**:
   - Copy the build folder (which contains your static files) to your server:
   ```bash
   scp -r build/* username@server:/var/www/brain-bloom/frontend/dist
   ```

## 5. Configure NGINX as a Reverse Proxy

1. **Edit NGINX Configuration**:
   ```bash
   sudo nano /etc/nginx/sites-available/default
   ```

2. **Update NGINX Configuration**:
   Add the following to the `server` block in the NGINX config:
   ```nginx
   server {
       listen 80;
       server_name brainbloom.sbs www.brainbloom.sbs;

       # Serve the frontend (React app)
       root /var/www/brain-bloom/frontend/dist;
       index index.html;

       location / {
           try_files $uri /index.html;
       }

       # Proxy API requests to the backend
       location /api/ {
           proxy_pass http://localhost:5000/;  # Backend API on port 5000
           proxy_http_version 1.1;
           proxy_set_header Upgrade $http_upgrade;
           proxy_set_header Connection 'upgrade';
           proxy_set_header Host $host;
           proxy_cache_bypass $http_upgrade;
       }
   }
   ```

3. **Test NGINX Configuration**:
   ```bash
   sudo nginx -t
   ```

4. **Restart NGINX**:
   ```bash
   sudo systemctl restart nginx
   ```

## 6. Set Up SSL with Let's Encrypt

1. **Install Certbot and the NGINX Plugin**:
   ```bash
   sudo apt install certbot python3-certbot-nginx
   ```

2. **Obtain SSL Certificates**:
   ```bash
   sudo certbot --nginx -d brainbloom.sbs -d www.brainbloom.sbs
   ```

3. **Auto-Renewal**:
   Certbot automatically configures a cron job for auto-renewal. You can check its status:
   ```bash
   sudo systemctl status certbot.timer
   ```

   To manually test the renewal:
   ```bash
   sudo certbot renew --dry-run
   ```

## 7. Final Steps

1. **Make Sure the Frontend is Accessible**:
   Visit `https://brainbloom.sbs` to check if your React app is being served correctly.

2. **Test Your API Endpoints**:
   Visit `https://brainbloom.sbs/api/hello` to verify your API is working.

3. **Update Axios in React**:
   Ensure your React app points to the correct backend API URL (using HTTPS):
   ```javascript
   const axiosInstance = axios.create({
       baseURL: "https://brainbloom.sbs/api", // Use HTTPS
       withCredentials: true,
       headers: {
           "Content-Type": "application/json"
       }
   });

   export default axiosInstance;
   ```

## 8. Troubleshooting

- **Check NGINX Logs**:
   If things aren't working, check the NGINX error log for details:
   ```bash
   sudo tail -f /var/log/nginx/error.log
   ```

- **Backend Logs**:
   If your backend isn't working, check the backend logs or run the backend with `nodemon` for easier debugging.

- **SSL Issues**:
   Ensure your SSL certificates are valid. Use online tools like [SSL Labs](https://www.ssllabs.com/ssltest/) to check your domain's SSL status.

---

## Recap of Steps:

1. Set up and configure NGINX as a reverse proxy for the frontend and backend.
2. Build and deploy your React app’s production build to the server.
3. Configure NGINX to serve the frontend and proxy backend API requests.
4. Set up SSL using Certbot to secure your site with HTTPS.
5. Test everything thoroughly (frontend, backend, API).

---

Following these steps should give you a fully functional, secure MERN stack application running with HTTPS.
