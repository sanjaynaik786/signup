# Use the lightweight Nginx Alpine image as the base
FROM nginx:alpine

# Remove default nginx static assets
RUN rm -rf /usr/share/nginx/html/*

# Copy the static HTML file to the Nginx content directory
COPY index.html /usr/share/nginx/html/index.html

# Expose port 80 to the host
EXPOSE 80

# Start Nginx in the foreground
CMD ["nginx", "-g", "daemon off;"]