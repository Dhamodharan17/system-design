### Deploying in K8

1. Create a spring boot project

```
@SpringBootApplication
@RestController
public class ArcSystemsApplication {

	@GetMapping("/welcome")
	public String welcome(){
		return "Spring Boot Docker Demo";
	}

	public static void main(String[] args) {
		SpringApplication.run(ArcSystemsApplication.class, args);
	}

}
```
2. Build using Maven (Intellij) -> check jar has been generated in target folder
3. Create a Dockerfile
```
FROM openjdk:17
ADD target/arc-systems-0.0.1-SNAPSHOT.jar app1.jar
ENTRYPOINT [ "java", "-jar","app1.jar" ]
```
4.  execute 'docker build -t demo:latest'  -> will generate docker image
5.  check using 'docker images'
6. to run 'docker run -p 8081:8080 demo'
7.  Create a app-deployment.yaml file
```
# app-deployment.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: spring-app
spec:
  replicas: 1
  selector:
    matchLabels:
      app: spring-app
  template:
    metadata:
      labels:
        app: spring-app
    spec:
      containers:
        - name: spring-app
          image: demo:latest  # Replace with your Docker image name
          imagePullPolicy: Never  # Prevents Kubernetes from pulling the image from a registry
          ports:
            - containerPort: 8080  # Replace with your app’s port if different

---
apiVersion: v1
kind: Service
metadata:
  name: spring-app-service
spec:
  selector:
    app: spring-app
  ports:
    - protocol: TCP
      port: 80  # Port exposed by the service
      targetPort: 8080  # Port your app is listening on
      nodePort: 30000  # Specify a port number (30000-32767) for NodePort
  type: NodePort

```
7. Execute ' kubectl config use-context docker-desktop ' to set context
8. run 'kubectl apply -f app-deployment.yaml'

### Commands
* kubectl get services
* kubectl get pods
* kubectl delete service  spring-app-service
* kubectl delete deployment spring-app
* kubectl get services spring-app-service
* kubectl describe pod spring-app -n default
* docker build -t demo:latest .
* kubectl config current-context

### Trouble Shooting

1. Run in context of docker desktop and disable pull of image from docker hub, to fetch from local
2. since we are running from local, use nodeport instead of load balancer
