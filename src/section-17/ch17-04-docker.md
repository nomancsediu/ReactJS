# Docker for React

Docker lets you package your app with everything it needs to run. No more "it works on my machine" problems. I was intimidated by Docker at first, but the basics are simpler than they look.

## A Basic Dockerfile for React

This builds a static React app and serves it with nginx:

```dockerfile
# Stage 1: Build the app
FROM node:20-alpine AS build
WORKDIR /app
COPY package.json package-lock.json ./
RUN npm ci
COPY . .
RUN npm run build

# Stage 2: Serve with nginx
FROM nginx:alpine
COPY --from=build /app/dist /usr/share/nginx/html
EXPOSE 80
CMD ["nginx", "-g", "daemon off;"]
```

The build stage creates the static files. The serve stage uses nginx to serve them. The final image contains only nginx and your built files, not Node.js or your source code.

## Nginx Configuration

React apps need a custom nginx config to handle client-side routing:

```nginx
# nginx.conf
server {
    listen 80;
    root /usr/share/nginx/html;
    index index.html;

    location / {
        try_files $uri $uri/ /index.html;
    }

    location /assets/ {
        expires 1y;
        add_header Cache-Control "public, immutable";
    }
}
```

Copy it into the Docker image:

```dockerfile
FROM nginx:alpine
COPY --from=build /app/dist /usr/share/nginx/html
COPY nginx.conf /etc/nginx/conf.d/default.conf
EXPOSE 80
CMD ["nginx", "-g", "daemon off;"]
```

The `try_files` line ensures that refreshing on a route like `/about` still serves `index.html` instead of a 404.

## Multi-Stage Builds

Multi-stage builds keep your final image small. The first stage has all the build tools. The second stage has only what is needed to run:

```dockerfile
# Build stage: ~1GB
FROM node:20-alpine AS build
# ... build steps ...

# Final stage: ~25MB
FROM nginx:alpine
COPY --from=build /app/dist /usr/share/nginx/html
```

The final image does not include Node.js, npm, or your source code. Just nginx and the static files.

## Docker Compose

Use Docker Compose to run your app alongside a database or API:

```yaml
version: "3.8"
services:
  web:
    build: .
    ports:
      - "3000:80"
    environment:
      - VITE_API_URL=http://api:8000
    depends_on:
      - api

  api:
    build: ./api
    ports:
      - "8000:8000"
    environment:
      - DATABASE_URL=postgresql://user:pass@db:5432/mydb
    depends_on:
      - db

  db:
    image: postgres:16
    environment:
      POSTGRES_USER: user
      POSTGRES_PASSWORD: pass
      POSTGRES_DB: mydb
    volumes:
      - pgdata:/var/lib/postgresql/data

volumes:
  pgdata:
```

Run everything with one command:

```bash
docker compose up
```

## Useful Commands

```bash
# Build the image
docker build -t my-react-app .

# Run the container
docker run -p 3000:80 my-react-app

# Run in background
docker run -d -p 3000:80 my-react-app

# View running containers
docker ps

# Stop a container
docker stop <container-id>
```

Docker is overkill for simple apps, but once your project grows or you need consistent environments across a team, it becomes essential.
