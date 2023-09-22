# cicd
DevOps CI/CD Pipeline

Title: Mastering DevOps: Building, Containerizing, and Deploying Python Web Server on Kubernetes

Introduction:

In this tutorial, we'll embark on a DevOps journey to create a robust Python-based web server, containerize it with Docker, and deploy it to Kubernetes. Follow along as we break down each step. 🚀

Step 1: Create a Python Web Server

Begin by creating a Python file named server.py. Copy and paste the following code:

python
Copy code
import http.server
import socketserver

class MyHandler(http.server.SimpleHTTPRequestHandler):
    def do_GET(self):
        self.send_response(200)
        self.end_headers()
        self.wfile.write(b"Hello, World!")

if __name__ == "__main__":
    PORT = 8000
    with socketserver.TCPServer(("", PORT), MyHandler) as httpd:
        print(f"Server running on port {PORT}")
        httpd.serve_forever()
Step 2: Run the Server Locally

Open a terminal, navigate to the directory containing server.py, and run:

bash
Copy code
python server.py
Visit http://localhost:8000 to see "Hello, World!" displayed.

Step 3: Create a Dockerfile

Create a file named Dockerfile (no extension) with the following content:

Dockerfile
Copy code
FROM python:3.9
WORKDIR /app
COPY server.py .
EXPOSE 8000
CMD ["python", "server.py"]
Step 4: Build and Run the Docker Image

In the same directory as the Dockerfile, run the following commands:

bash
Copy code
docker build -t my-app .
docker run -d -p 8000:8000 my-app
Step 5: Deploy to Kubernetes

Create a file named deployment.yaml and add the following content:

yaml
Copy code
apiVersion: apps/v1
kind: Deployment
metadata:
  name: my-app-deployment
spec:
  replicas: 1
  selector:
    matchLabels:
      app: my-app
  template:
    metadata:
      labels:
        app: my-app
    spec:
      containers:
      - name: my-app-container
        image: my-app
        ports:
        - containerPort: 8000
Apply the deployment:

bash
Copy code
kubectl apply -f deployment.yaml
Step 6: Expose to the Outside World

Create a file named service.yaml with the following content:

yaml
Copy code
apiVersion: v1
kind: Service
metadata:
  name: my-app-service
spec:
  selector:
    app: my-app
  ports:
  - protocol: TCP
    port: 80
    targetPort: 8000
  type: LoadBalancer
Apply the service:

bash
Copy code
kubectl apply -f service.yaml
Conclusion:

Congratulations! You've successfully built, containerized, and deployed a Python web server on Kubernetes. 🎉 Let's continue exploring the vast world of DevOps and keep building amazing things!

Step 7: Update the code to wait 30 seconds before it starts

Modify the server.py file to include a 30-second delay before starting the server. Add the following code snippet at the beginning of the if __name__ == "__main__": block:

python
Copy code
import time
time.sleep(30)
This code will cause the server to wait for 30 seconds before it starts, simulating a delay that can occur during startup procedures.

Step 8: Add a Kubernetes health check to serve traffic only when the app is ready

Update the deployment.yaml file to include a readiness probe. Modify the containers section as follows:

yaml
Copy code
containers:
- name: my-app-container
  image: my-app
  ports:
  - containerPort: 8000
  readinessProbe:
    httpGet:
      path: /
      port: 8000
    initialDelaySeconds: 5
    periodSeconds: 10
This readiness probe specifies that the app is ready if it responds with a successful HTTP GET request to the root path (/) on port 8000. The probe starts 5 seconds after the container starts and checks every 10 seconds.

Apply the updated configuration by running:

bash
Copy code
kubectl apply -f deployment.yaml
This ensures that traffic is only served to the application when it is in a ready state.

Step 9: Update the app to return a message from ConfigMap

Create a file named configmap.yaml and add the following content:

yaml
Copy code
apiVersion: v1
kind: ConfigMap
metadata:
  name: my-app-config
data:
  message: "Hello from ConfigMap!"
This ConfigMap defines a key-value pair where the key is message and the value is "Hello from ConfigMap!".

Apply the ConfigMap by running:

bash
Copy code
kubectl apply -f configmap.yaml
Now, update the server.py file to retrieve and return the message from the ConfigMap. Modify the do_GET method as follows:

python
Copy code
import os

# ...

def do_GET(self):
    message = os.getenv("MESSAGE", "Hello, World!")
    self.send_response(200)
    self.end_headers()
    self.wfile.write(message.encode())
The updated code retrieves the MESSAGE environment variable, which will be set using the ConfigMap, and uses it as the response message. If the MESSAGE environment variable is not set, it falls back to "Hello, World!".

Rebuild and redeploy the app by running the following commands:

bash
Copy code
docker build -t my-app .
kubectl rollout restart deployment my-app-deployment
