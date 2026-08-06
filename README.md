# three-tier-frontend

#Create Frontend Dockerfile
===========================
FROM node:20-alpine as builder
WORKDIR /app
COPY ./code/package*.json ./
RUN npm install
COPY ./code .
COPY .env .
RUN npm run build
FROM nginx:alpine
COPY --from=builder /app/dist /usr/share/nginx/html
COPY ./code/nginx/nginx.conf /etc/nginx/nginx.conf
EXPOSE 80
CMD ["nginx", "-g", "daemon off;"]


#Create docker-compose.yml
===========================
services:
 frontend:
  build:
   context: ./frontend
   dockerfile: Dockerfile
  container_name: frontend
  ports:
   - "80:80"
  depends_on:
   - backend
     
 backend:
  build:
   context: ./backend
   dockerfile: Dockerfile
  container_name: backend
  ports:
   - "3000:3000"
  env_file:
   - ./backend/.env
  depends_on:
   - redis
   - mysql
     
 mysql:
  image: mysql:8.0
  container_name: mysql
  environment:
   - MYSQL_ROOT_PASSWORD=admin123
   - MYSQL_DATABASE=schooldb
   - MYSQL_USER=admin
   - MYSQL_PASSWORD=admin123
  ports:
   - "3306:3306"
  volumes:
   - ./mysql_data:/var/lib/mysql
   - ./init.sql:/docker-entrypoint-initdb.d/init.sql
     
 redis:
  image: redis:alpine
  container_name: redis
  ports:
   - "6379:6379"
   volumes:
   - ./redis_data:/data


#Create .env file (/frontend/.env). add your db url or public-IP <server-ip>
=================
VITE_API_URL=http://<server-ip>/api


#Create init.sql file 
=====================
CREATE TABLE IF NOT EXISTS students (
 id INT AUTO_INCREMENT PRIMARY KEY,
 name VARCHAR(255) NOT NULL,
 age INT NOT NULL,
 class VARCHAR(100) NOT NULL,
 created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);


#tree
<img width="598" height="336" alt="image" src="https://github.com/user-attachments/assets/4a1918cc-c51e-44c0-93be-4d0317f7cb22" />

