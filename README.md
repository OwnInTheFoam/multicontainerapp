
# Multi Container Application

**THIS REPOSITORY IS FORKED FROM:** [codeching](https://gitlab.com/codeching/docker-multicontainer-application-react-nodejs-postgres-nginx-basic)

It contains React client, Node.js backend, PostgreSQL and Nginx

You can run it in development mode: docker-compose up --build
It contains Dockerfiles for client, server which you should push to your docker hub to be able
to pull them down when in next tutorial we will use them in Kubernetes.

## Resources
- [Forked reposity](https://gitlab.com/codeching/docker-multicontainer-application-react-nodejs-postgres-nginx-basic)
- [Codeching - Dockerizing a React application with Node.js Postgres and NginX - dev and prod - step by step](https://www.youtube.com/watch?v=-pTel5FojAQ)
- [Codeching - Kubernetes Multi Container Deployment | React | Node.js | Postgres | Ingress Nginx | step by step](https://www.youtube.com/watch?v=OVVGwc90guo)

## Requirements
- cluster with kubectl
- docker cli ($ curl -sSL https://get.docker.com | sh -s -- --version <version>)

## Startup

Setup the client and server by installing the dependancies for each.

```bash
npm install
```

## Testing

Test the server in development with
```bash
npm run dev
```
Then open browser and check for HI output
```bash
curl localhost:5000
```

Test the client in development with
```bash
npm run start
```
and open browser to
```bash
curl localhost:3000
```
Viewing the network developer tools tab, send a value and notice a http request is sent.

## Create dockerfile for dev and prod
These (`Dockerfile`, `Dockerfile.dev`) should inherit from a base image like node, nginx, postgres etc. The docker hub for each base image will have documentation for each.
A `docker-compose.yml` file to run the server, client and nginx together in locally environment.

## Testing - dev
_Ensure you set `group`, `project` and `image` variables_
Locally on docker desktop

### Client
Create Dockerfile.dev in client folder.

Test your docker file and provide a image tag
```bash
docker build -f Dockerfile.dev -t registry.gitlab.com/project/group/client-dev:latest .
```

Run your docker image
```bash
docker run -it -p 4002:3000 registry.gitlab.com/project/group/client-dev:latest
```

Open browser
```bash
localhost:4002
```

### Server
Create Dockerfile.dev in server folder

Test your docker file and provide a image tag
```bash
docker build -f Dockerfile.dev -t registry.gitlab.com/project/group/server-dev:latest .
```

Run your docker image
```bash
docker run -it -p 4003:5000 registry.gitlab.com/project/group/server-dev:latest
```

Open browser
```bash
localhost:4003
```
### Docker Compose
Create docker compose to run everything together in local docker desktop
Use nginx has a ingress. Create `default.conf` in nginx folder
Create `Dockerfile.dev` in nginx folder

Test all deployments:
```bash
docker compose up --build
localhost:3050
```

## Testing - prod
_Ensure you set `group` and `project` variables_
Locally on docker desktop

### Client
Create Dockerfile in client folder.

Test your docker file and provide a image tag
```bash
docker build -t registry.gitlab.com/project/group/client:latest .
```

Run your docker image
```bash
docker run -it -p 3000:3000 registry.gitlab.com/project/group/client:latest
```

Open browser
```bash
localhost:3000
```

### Server
Create Dockerfile in server folder

Test your docker file and provide a image tag
```bash
docker build -t registry.gitlab.com/project/group/server:latest .
```

Run your docker image
```bash
docker run -it -p 4002:5000 registry.gitlab.com/project/group/server:latest
```

Open browser
```bash
localhost:4002
```

## Upload to docker registry

Ensure docker desktop is running, then:

Log in to your Docker Hub account using the docker command-line tool:
```bash
docker login registry.gitlab.com
```
### Client
Build docker image and provide a tag
```bash
cd client
docker build -t registry.gitlab.com/project/group/client:latest .
```
Push to your container registry
```bash
docker push registry.gitlab.com/project/group/client:latest
```
### Server
Build docker image and provide a tag
```bash
cd server
docker build -t registry.gitlab.com/project/group/server:latest .
```
Push to your container registry
```bash
docker push registry.gitlab.com/project/group/server:latest
```

## Test deployment to kubernetes
### Create deployment token
_Ensure you update all `project/group/` instances and `secret-registry.yml`!_
- Create [gitlab deploy access token](https://docs.gitlab.com/ee/user/project/deploy_tokens/)
- Create `k8s/secret-registry.yml` OR `kubectl create secret docker-registry regcred --docker-server=registry.gitlab.com --docker-username=<your-name> --docker-password=<token>`
- Apply with `k8s/secret-registry.yml`
- Test with `kubectl get secret regcred -o yaml`
- Create `k8s/client-deployment.yml`
- Apply with `kubectl apply -f k8s/client-deployment.yml`
- Add to deployment `kubectl edit deployment client-deployment` and add:
```yaml
spec:
  template:
    spec:
      imagePullSecrets:
      - name: regcred
```
- Test with `kubectl get pods`
- Create `k8s/client-cluster-ip-service.yml`
- Apply with `kubectl apply -f k8s/client-cluster-ip-service.yml`
- Test with `kubectl get services`
- Create `k8s/database-persistent-volume-claim.yml`
- Apply with `kubectl apply -f k8s/database-persistent-volume-claim.yml`
- Test with `kubectl get pvc,pv`
- Create secret for postgres: `kubectl create secret generic pgpassword --from-literal PGPASSWORD=yourPassword`
- Test with, `kubectl get secrets`
- Create `k8s/postgres-deployment.yml`
- Apply with `kubectl apply -f k8s/postgres-deployment.yml`
- Test with `kubectl get pods`
- Create `k8s/postgres-cluster-ip-service.yml`
- Apply with `kubectl apply -f k8s/postgres-cluster-ip-service.yml`
- Test with `kubectl get services`
- Create `k8s/server-deployment.yml`
- Apply with `kubectl apply -f k8s/server-deployment.yml`
- Add to deployment `kubectl edit deployment server-deployment` and add:
```yaml
spec:
  template:
    spec:
      imagePullSecrets:
      - name: regcred
```
- Test with `kubectl get pods`
- Create `k8s/server-cluster-ip-service.yml`
- Apply with `kubectl apply -f k8s/server-cluster-ip-service.yml`
- Test with `kubectl get services`
- Install [nginx ingress controller](https://github.com/kubernetes/ingress-nginx/blob/main/docs/deploy/index.md#docker-desktop) with `kubectl apply -f https://raw.githubusercontent.com/kubernetes/ingress-nginx/controller-v1.8.1/deploy/static/provider/cloud/deploy.yaml`
- Test with `kubectl get all -n ingress-nginx`
- Create `k8s/ingress-service.yml`
- Apply with `kubectl apply -f k8s/ingress-service.yml`
- Test with `kubectl get Ingress`
- Test application with browser `curl localhost`
- Delete the application with `kubectl delete -f k8s`

## Pull images from container registry

Ensure docker desktop is running with kubernetes enabled in settings.
Log in to your Docker Hub account using the docker command-line tool:
```bash
docker login registry.gitlab.com
```
Ensure you have access to your cluster with `kubectl`
```bash
kubectl version
```

### Client
Pull the image from Docker Hub to your local machine:
```bash
docker pull registry.gitlab.com/project/group/client:latest
```
Alternative use `kubectl` to retrieve the image. A deployment will be created without any pods.
```bash
docker run -it -p 4002:3000 registry.gitlab.com/project/group/client:latest
```
### Server
Pull the image from Docker Hub to your local machine:
```bash
docker pull registry.gitlab.com/project/group/server:latest
```
Alternative use `kubectl` to retrieve the image. A deployment will be created without any pods.
```bash
docker run -it -p 4003:5000 registry.gitlab.com/project/group/server:latest
```


