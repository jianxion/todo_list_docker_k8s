# TODO List Application

A Flask-based TODO list application with MongoDB backend, fully containerized using Docker.

## Features

- Create, read, update, and delete TODO items
- Mark tasks as complete/incomplete
- Search functionality
- Priority levels for tasks
- Persistent MongoDB storage

## Architecture

The application uses a microservices architecture with two containers:
- **Flask App Container**: Runs the Python Flask web application
- **MongoDB Container**: Provides data persistence

## Prerequisites

- Docker installed on your machine
- Docker Compose installed

## Quick Start

### 1. Clone the repository

```bash
git clone <repository-url>
cd todo_list_app
```

### 2. Start the application

```bash
docker-compose up -d
```

This command will:
- Build the Flask application Docker image
- Pull the MongoDB 7.0 image
- Start both containers
- Create a persistent volume for MongoDB data

### 3. Access the application

Open your browser and navigate to:
```
http://localhost:5001
```

## Docker Commands

### Start the application
```bash
docker-compose up -d
```

### Stop the application
```bash
docker-compose down
```

### View logs
```bash
# Flask app logs
docker logs todo-flask-app

# MongoDB logs
docker logs todo-mongodb

# Follow logs in real-time
docker logs -f todo-flask-app
```

### Rebuild after code changes
```bash
docker-compose up --build -d
```

## to follow logs when running the app
docker logs -f todo-flask-app

## Project Structure

```
todo_list_app/
├── app.py                 # Main Flask application
├── requirements.txt       # Python dependencies
├── dockerfile            # Docker image definition
├── docker-compose.yml    # Multi-container orchestration
├── .dockerignore        # Files to exclude from Docker image
├── templates/           # HTML templates
│   ├── index.html
│   ├── update.html
│   ├── searchlist.html
│   └── credits.html
└── static/             # Static assets (CSS, JS, images)
    ├── assets/
    └── images/
```

## Configuration

### Environment Variables

The application uses the following environment variables (configured in docker-compose.yml):

- `MONGO_HOST`: MongoDB hostname (default: mongodb)
- `MONGO_PORT`: MongoDB port (default: 27017)
- `FLASK_ENV`: Flask environment (default: production)

### Port Configuration

- **Flask App**: Port 5001 (host) → Port 5000 (container)
- **MongoDB**: Port 27017 (host) → Port 27017 (container)

*Note: Port 5001 is used instead of 5000 to avoid conflicts with macOS AirPlay Receiver*

## Data Persistence

MongoDB data is persisted using Docker volumes:
- Volume name: `mongodb_data`
- Mount point: `/data/db`

Your data will persist even after stopping or removing containers.

## Deployment to Kubernetes

### 1. Tag and Push to Docker Hub

```bash
# Tag the image
docker tag todo_list_app-flask-app <your-dockerhub-username>/todo-flask-app:latest

# Login to Docker Hub
docker login

# Push the image
docker push <your-dockerhub-username>/todo-flask-app:latest
```

### 2. Create Kubernetes Deployment

Create deployment manifests for:
- MongoDB StatefulSet with persistent volume
- Flask Deployment
- Services for both components
- ConfigMaps for configuration

## Development

### Running Locally Without Docker

1. Install Python dependencies:
```bash
pip install -r requirements.txt
```

2. Start MongoDB locally:
```bash
mongod --dbpath /path/to/data
```

3. Run the Flask application:
```bash
python app.py
```

### Making Changes

1. Modify the code
2. Rebuild the Docker image:
```bash
docker-compose up --build -d
```

## Troubleshooting

### Port Already in Use

If port 5001 is already in use, edit `docker-compose.yml` and change the port mapping:
```yaml
ports:
  - "5002:5000"  # Change 5001 to 5002 or another available port
```

### Container Not Starting

Check the logs:
```bash
docker logs todo-flask-app
docker logs todo-mongodb
```

### MongoDB Connection Issues

Ensure MongoDB container is running:
```bash
docker ps | grep mongodb
```

### Reset Everything

To completely reset the application and data:
```bash
docker-compose down -v  # Remove containers and volumes
docker-compose up -d    # Start fresh
```

## License

[Your License Here]

## Contributors

[Your Name/Team]
