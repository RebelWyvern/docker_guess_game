# Step 1: Build the React app using Node.js
FROM node:18.17.0 AS build

# Set the working directory inside the container
WORKDIR /app

# Copy package.json and yarn.lock to install dependencies
COPY frontend/package.json frontend/yarn.lock ./

# Install the dependencies
RUN yarn install

# Set environment variable
ARG REACT_APP_BACKEND_URL
ENV REACT_APP_BACKEND_URL=${REACT_APP_BACKEND_URL}

# Copy the rest of the application code to the container
COPY frontend/ ./

# Build the React app for production
RUN yarn build

# Step 2: Serve the built files with Nginx
FROM nginx:latest

# Copy the built files from the build stage to Nginx's default HTML directory
COPY --from=build /app/build /usr/share/nginx/html

# Optional: Add custom Nginx configuration
COPY ./nginx/default.conf /etc/nginx/conf.d/default.conf


# Expose port 80 for Nginx
EXPOSE 80

# Start Nginx in the foreground
CMD ["nginx", "-g", "daemon off;"]

