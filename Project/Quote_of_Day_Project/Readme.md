## How to run it inside a Docker container.
<details>
  <summary>Click me</summary>
  Step1: Create Dockerfile to build image
    
  Step2: Build and Run the Docker image.
    
  Create Dockerfile:
  
    ```yaml
    # Use a small static web server image, e.g. nginx
    FROM nginx:alpine
    
    # Remove default nginx static content
    RUN rm -rf /usr/share/nginx/html/*
    
    # Copy your app’s static files into nginx’s html directory
    COPY . /usr/share/nginx/html
    
    # Expose port 80 to the outside
    EXPOSE 80
    
    # Start nginx (this is default command in nginx image, so you could skip)
    CMD ["nginx", "-g", "daemon off;"]
    ```
  Build and Run image
    
    ```bash
    # Navigate into the project folder
    cd path_to_project
    
    # Build the Docker image
    docker build -t quote-of-day:latest .
    
    # Run a container from it
    docker run -d -p 8080:80 quote-of-day:latest
    ```
  This maps container’s port 80 to your host’s port 8080.
  Then in your browser, go to http://localhost:8080 → you should see the “Quote of the Day” app running.
  <img width="1268" height="713" alt="image" src="https://github.com/user-attachments/assets/b0cfabfb-6674-4462-9163-e0e3da654a60" />
  <img width="1120" height="507" alt="image" src="https://github.com/user-attachments/assets/245884d1-6848-477f-95e6-c199f3c26762" />
  <img width="874" height="190" alt="image" src="https://github.com/user-attachments/assets/d17965f8-8d0e-42e2-89b8-f6e31dec0aaf" /><img width="1822" height="883" alt="image" src="https://github.com/user-attachments/assets/afd5a6db-fac2-4c31-a1ad-122218db183a" />
</details>

## How to run it on Kubernetes (k8s).
<details>
  <summary>Click me</summary>
  
  ```bash
  # Log in to Docker Hub
  docker login
  
  # Tag your image (replace <your-username>)
  docker tag quote-of-day:latest <your-username>/quote-of-day:latest
  
  # Push it
  docker push <your-username>/quote-of-day:latest
  ```
  Deplyment.yaml
  ```yaml
  apiVersion: apps/v1
  kind: Deployment
  metadata:
    name: quote-of-day-deployment
  spec:
    replicas: 1
    selector:
      matchLabels:
        app: quote-of-day
    template:
      metadata:
        labels:
          app: quote-of-day
      spec:
        containers:
        - name: quote-of-day
          image: <your-username>/quote-of-day:latest
          ports:
          - containerPort: 80
  ```
  Service.yaml
  ```yaml
  apiVersion: v1
  kind: Service
  metadata:
    name: quote-of-day-service
  spec:
    type: NodePort
    selector:
      app: quote-of-day
    ports:
    - port: 80
      targetPort: 80
      nodePort: 30080
  ```
  
  Deploy in K8
  ```bash
  kubectl apply -f deployment.yaml
  kubectl apply -f service.yaml
  
  kubectl get pods
  kubectl get svc
  ```
  <img width="540" height="64" alt="image" src="https://github.com/user-attachments/assets/d604dc5c-763c-4cef-83c5-24a07c8b56cf" />
  <img width="1016" height="284" alt="image" src="https://github.com/user-attachments/assets/2aa13c7b-8291-46f4-b4d9-0d04e571d762" />
  <img width="874" height="576" alt="image" src="https://github.com/user-attachments/assets/19fb48cc-c3fb-46e2-a3d4-3a97e3a2f2c2" />
</details>


