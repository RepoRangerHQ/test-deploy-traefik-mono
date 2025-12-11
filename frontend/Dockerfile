# Stage 1: Build Vue App
FROM node:20-alpine AS build
WORKDIR /app

# รับ build argument
ARG VITE_API_URL
ENV VITE_API_URL=$VITE_API_URL

COPY package*.json ./
RUN npm install
COPY . .
RUN npm run build

# Stage 2: Serve with Nginx
FROM nginx:alpine
# Copy config ของเราไปทับ default
COPY nginx.conf /etc/nginx/conf.d/default.conf
# Copy ไฟล์ที่ build แล้วจาก Stage 1 (Vite build to /dist)
COPY --from=build /app/dist /usr/share/nginx/html

EXPOSE 3000
CMD ["nginx", "-g", "daemon off;"]