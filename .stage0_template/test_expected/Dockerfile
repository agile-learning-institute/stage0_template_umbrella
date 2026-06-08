FROM nginx:stable-alpine

LABEL org.opencontainers.image.source="https://github.com/agile-learning-institute/mentorhub"

# Copy the welcome catalog and developer sign-in page
COPY index.html /usr/share/nginx/html/index.html
COPY login.html /usr/share/nginx/html/login.html
COPY welcome-auth.js /usr/share/nginx/html/welcome-auth.js

# Expose port 80
EXPOSE 80

# Start nginx
CMD ["nginx", "-g", "daemon off;"]