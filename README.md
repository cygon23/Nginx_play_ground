# Load Balancer with NGINX, Docker, and Node.js (Express)

This project demonstrates the use of NGINX as a reverse proxy and load balancer to distribute traffic across multiple Node.js (Express) containers running in Docker. It also demonstrates setting up self-signed SSL certificates for secure HTTPS communication on port 443.

## Features

- Load balancing across multiple replicas of a static webpage served by Node.js.
- NGINX configured as a reverse proxy and load balancer to distribute incoming traffic evenly.
- Self-signed SSL certificates to run the application over HTTPS (port 443).
- Docker Compose used to manage the multi-container setup.

## Project Structure

The project uses the following components:

1. **NGINX**: Acts as a reverse proxy and load balancer.
2. **Node.js (Express)**: Serves a static webpage with a simple backend.
3. **Docker**: Used to create and manage containers for NGINX and the Node.js application.
4. **Docker Compose**: Orchestrates the multi-container Docker application.

## Prerequisites

Before setting up the application, ensure you have the following installed:

- Node,Ngix, and docker
- [Docker](https://www.docker.com/get-started) (including Docker Compose)
- Basic understanding of NGINX, Docker, and Node.js (Express)

## Installation and Setup

### Step 1: Clone the Repository

Clone the project repository to your local machine:

```bash
git https://github.com/cygon23/Nginx_play_ground
cd  Nginx_play_ground
```

### Step 2: Build and Start the Containers

1. **Self-Signed Certificates**: The project uses self-signed SSL certificates for HTTPS. If you don’t have them, you can create them using the following command:

```bash
openssl req -newkey rsa:2048 -nodes -keyout ssl/server.key -x509 -out ssl/server.crt -days 365 OR
openssl req -x509 -nodes -days 356 -newkey rsa:2048 -keyout name.key -out name.crt
```

2. **Build and Start the Docker Containers**:

Run Docker Compose to build and start the services:

```bash
docker-compose up --build -d
```

This will:

- Build the Docker images for the Node.js application and NGINX reverse proxy.
- Create and run 3 replicas of the Node.js application.
- Set up NGINX as the reverse proxy and load balancer.

### Step 3: Access the Application

1. Open your browser and navigate to `https://localhost`. You should see the static webpage served by the Node.js application, and the traffic will be load-balanced across the three replicas.
2. Since the certificates are self-signed, you may need to accept the security warning in your browser.

### Step 4: View the Logs

You can check the logs of the containers to ensure everything is running properly:

```bash
docker-compose logs -f
```

This will show the logs of all the services (NGINX and Node.js).

### Step 5: Scaling the Application (Optional)

You can easily scale the Node.js replicas by updating the `docker-compose.yml` file and changing the number of replicas for the `node-app` service. For example, change this section:

```yaml
services:
  node-app:
    build: ./node-app
    deploy:
      replicas: 3 # You can change this value to scale the app
```

After making the change, restart the services:

```bash
docker-compose up -d
```

### Step 6: Stopping the Application

To stop the running containers, use the following command:

```bash
docker-compose down
```

This will stop and remove all containers, networks, and volumes created by `docker-compose down`.

## Configuration Details

### NGINX Configuration

The NGINX configuration (`nginx.conf`) is set up to:

- Proxy HTTPS traffic to the Node.js application.
- Load balance requests across the 3 Node.js replicas.
- Use SSL certificates for secure communication.

```nginx
server {
    listen 443 ssl;
    server_name localhost;

    ssl_certificate /etc/nginx/ssl/server.crt;
    ssl_certificate_key /etc/nginx/ssl/server.key;

    location / {
        proxy_pass http://node-app;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
    }
}

upstream node-app {
    server node-app:3000;
    server node-app:3001;
    server node-app:3002;
}
```

### Docker Compose Configuration

The `docker-compose.yml` file defines the services for the application:

- **nginx**: The NGINX container acting as a reverse proxy and load balancer.
- **node-app**: The Node.js application container, which is replicated 3 times for load balancing.

```yaml
version: "3.8"

services:
  nginx:
    image: nginx:latest
    container_name: nginx
    volumes:
      - ./nginx/nginx.conf:/etc/nginx/nginx.conf
      - ./ssl:/etc/nginx/ssl
    ports:
      - "443:443"

  node-app:
    build: ./node-app
    container_name: node-app
    ports:
      - "3000:3000"
    networks:
      - app-network
    deploy:
      replicas: 3

networks:
  app-network:
    driver: bridge
```

## Conclusion

This project provides a simple example of using NGINX as a reverse proxy and load balancer in a Dockerized environment with Node.js (Express). The setup allows you to scale the application, use SSL for secure communication, and distribute the traffic across multiple replicas of the Node.js app.

Feel free to modify the configuration to suit your needs!

## License

This project is licensed under the MIT License - see the [LICENSE](LICENSE) file for details.

---

Feel free to modify this `README.md` as per your exact setup and requirements. This guide will help users set up the project easily and understand the core functionalities of load balancing, reverse proxying, and using self-signed SSL certificates.
