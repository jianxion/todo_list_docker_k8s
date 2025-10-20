# Docker Containerization Steps

The first step to containerize the app is to create a dockerfile. The Dockerfile established `/app` as the working directory within the container, copy the requirements.txt file first and installing dependencies before copying the application code. All application files were then copied into the container, and the Flask application was configured to expose port 5000 internally. The final CMD instruction specified that the container should execute `python app.py` when started, launching the Flask development server.


For the next step, the docker-compose.yml file was created to orchestrate both the Flask application container and the MongoDB database container. This configuration defined two services: mongodb and flask-app. The MongoDB service used the official mongo:7.0 image and was configured with port 27017 exposed to the host machine. Critically, a named volume called `mongodb_data` was mounted to `/data/db` within the MongoDB container to ensure data persistence across container restarts and removals.

The Flask application service was configured to build from the local Dockerfile I just created. Environment variables were passed to the container to specify that MongoDB could be reached at the hostname `mongodb` on port 27017. This hostname-based connection works because docker-compose creates a custom bridge network where containers can reference each other by their service names. The depends_on directive ensured that the MongoDB container would be started before the Flask container.

Initially, the Flask application was configured to map port 5000 on the host to port 5000 in the container. However, this caused conflicts with macOS's AirPlay Receiver service, which occupies port 5000 by default on newer macOS versions. The port mapping was changed to `5001:5000`, exposing the application on port 5001 on the host machine while maintaining port 5000 internally within the container.


To handle the incompatible version of Werkzeug package and the Flask package, the solution was to explicitly pin Werkzeug to version 2.0.3 in the requirements.txt file. This version was known to be compatible with Flask 2.1.3. 
To make the application accessible from outside the container, the code needed to bind to 0.0.0.0 instead of local host. After making this change, the application listens on all network interfaces and the application became accessible at localhost:5001 on the host machine.

Afterwards, the Flask application image was tagged and pushed to Docker Hub. The image was tagged as `sjxshen/todo-flask-app:latest`, using my Docker Hub username. The image is available at `https://hub.docker.com/r/sjxshen/todo-flask-app` for pulling by any Docker-enabled system.

# Kubernetes Deployment Steps

The next step was to deploy the application on Kubernetes using Minikube. First, Minikube was started which created a single-node Kubernetes cluster on the local machine.

Two Kubernetes deployment files were created in the k8s directory. The mongodb-deployment.yaml file defined three resources: a PersistentVolumeClaim requesting 1Gi of storage for data persistence, a Deployment that creates one MongoDB pod running mongo:7.0, and a ClusterIP Service named `mongodb` that exposes port 27017 for internal cluster communication. The PersistentVolumeClaim ensures that MongoDB data survives pod restarts and deletions.

The flask-deployment.yaml file defined the Flask application deployment with two replicas for load balancing and high availability. The deployment pulls the `sjxshen/todo-flask-app:latest` image from Docker Hub and configures each pod with environment variables. The deployment also created a NodePort Service that exposes the application on port 30000, making it accessible from outside the cluster.

When both deployments were applied using kubectl, Kubernetes created the resources in order. The MongoDB pod started first, followed by two Flask application pods. Kubernetes automatically handles service discovery through its internal DNS system, so when the Flask pods try to connect to `mongodb:27017`, Kubernetes resolves `mongodb` to the MongoDB Service's cluster IP address, enabling seamless communication between the pods.

To access the application, the `minikube service flask-service --url` command was used, which creates a tunnel and provides a URL like `http://127.0.0.1:60423`. This tunnel must remain open for the application to be accessible from the host machine. The NodePort Service automatically load balances incoming requests between the two Flask replicas, providing better performance and fault tolerance.