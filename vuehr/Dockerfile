# write a  Dockerfile for current  vue npm  project 
# Stage 1: Build the Vue project
FROM node:14 as build-stage
WORKDIR /app
# replace the repo to aliyun

RUN sed -i 's/deb.debian.org/mirrors.aliyun.com/g' /etc/apt/sources.list \
    && sed -i 's/security.debian.org/mirrors.aliyun.com/g' /etc/apt/sources.list \
    && apt-get update \
    && apt-get install -y python3 build-essential
# Copy package.json and package-lock.json
COPY package*.json ./
# Install dependencies
RUN npm install
# Copy the rest of the project files
COPY . .
# Build the project
Run npm config set registry https://registry.npmmirror.com/ && \
    npm run build

# Stage 2: Serve the built project with Nginx
FROM nginx:1.21-alpine as production-stage
COPY --from=build-stage /app/dist /usr/share/nginx/html
# Copy the custom Nginx configuration (optional)
COPY nginx.conf /etc/nginx/conf.d/default.conf
EXPOSE 80
CMD ["nginx", "-g", "daemon off;"]
