# Go Web Application

This is a simple website written in Golang. It uses the `net/http` package to serve HTTP requests.

## Running the server

To run the server, execute the following command:


```bash
go run main.go
```


The server will start on port 8080. You can access it by navigating to `http://localhost:8080/courses` in your web browser.

## Looks like this

![Website](static/images/golang-website.png)



--> Flow:

1. Containerization-> Docker File
2. Kubernetes manifests -> deployment file,services file, ingress
3. CI -> Github Actions
4. CD -> Continous Delivery (Argo CD)
5. Kubernetes -> EKS( will set up k8s cluster because cicd has to deploy the application in target 
   platform and target platform would be k8s )
6. Helm -> dev, Qa, Prod (we will also setup helm charts for the application because in future if the deve
    -lopment team want to deploy the application on to dev,qa and prod , instead of writing the manifest for each environment they can use the helm chart and pass the values.yaml)
7. Ingess Controller -> LB(DNS) -> Exposed (Finally we will also setup ingresscontroller configuration so that Ingress controller can create a load balancer depending upon the ingress configuration so that application is exposed to outside world)
8. we can also map the ip address of Load balancer to the local dns so that we can test our application is accessed to outside world


For runnung local:
-> go build -o main .
-> ./main -< Now you will see application will be running locally> 


Everything we will be doing in ec2 instance

1. will start writing dockerfile
   multistage docker file: 
   stage1: base image -> build 
   stage 2: Distroless image -> base(that adds security and reduced image size)
   In stage 2 will copy the binary that is build on stage1 and will expose the port and run the application
   Now refer Dockerfile


Contianerization:
--> docker build -t mishrp/go-web-app:v1 .
--> docker run -p 8081:8080 -it mishrp/go-web-app:v1
--> docker push mishrp/go-web-app:v1 -> image pushed to dockerhub registory


2. so Kuberentes deployment will pull the image from dockerhub registory or any other registory.
  Now create : k8s\manifests
  deployment.yaml,service.yaml,ingress.yaml
  Ingress class name is basically for the ingress resource to be identified by ingress controller. In some organization there might be multiple ingress controller . so some companies might use nginx + Aws application load-balancer + traffic. Because multiple development team might ask for multiple ingress controller.Now ingress resources within the k8s cluster need to be identified by those ingress controller. For that what we usually do we provide ingress class name and this ingress class name tells the ingress nginx controller that this is the ingress resource that you have to watch for. If we are not providing ingress class name then nginx ingress controller will not watch for resource. we are going to implement nginx ingress controller that's why have put ingressclassName: Nginx! 
  ** we can also use AWS application loadbalancer controller, Azure ingress controller as well.

  Now we need to create Kubernetes Cluster cluster such as AKS,EKS!
  kubectl – A command line tool for working with Kubernetes clusters. For more information, see Installing or updating kubectl.

    eksctl – A command line tool for working with EKS clusters that automates many individual tasks. For more information, see Installing or updating.

    AWS CLI – A command line tool for working with AWS services, including Amazon EKS. For more information, see Installing, updating, and uninstalling the AWS CLI in the AWS Command Line Interface User Guide. After installing the AWS CLI, we recommend that you also configure it. For more information, see Quick configuration with aws configure in the AWS Command Line Interface User Guide.

    Now command to install the EKS cluster in AWS:
    Install a EKS cluster with EKSCTL
    eksctl create cluster --name demo-cluster --region us-east-1 
    Delete the cluster
    eksctl delete cluster --name demo-cluster --region us-east-1

    **we can create EKS cluster using Terraform as well****

    As we have already configured the authentication from our local machine to AWS  with aws configure
    Now we can use below commands to create the deployments an all in k8s cluster:
    kubectl apply -f deployment.yaml
    kubectl apply -f service.yaml
    kubectl apply -f ingress.yaml
    
    once we do kubectl get ing: will see address is not assigned
    so what we need here is we need ingress controller so that ingress controller assigns the address for the ingress resource. once the address is assigned we will take the ip address and then will map the the ip address to the domain name that we have created to the etc host.

    But before that we can expose the service in nodeport mode. To check whether service is working fine

    kubectl edit svc go-weba-app
    (change clusterIP to NodePort)

    Now once do kubectl get svc then we will see our service is running on nodeport and running on some port.

    kubectl get nodes -o wide
    Here take the IP address of any node and copy the nodePort and search we will see application will be running on (nodeIP/nodePort)


 3.  Now we will create the Ingress controller configuration
  
   In Aws nginx ingress controller creates Network Load balancer

   kubectl apply -f https://raw.githubusercontent.com/kubernetes/ingress-nginx/controller-v1.11.1/deploy/static/provider/aws/deploy.yaml

   Ingress resource (watches)<- Ingress controller ->(creates) LB
   So Ingress controller watches Ingress resource and creates the loadbalancer

   Kubectl get ing -n ingress-nginx
   kubectl get ing (it will wacth our ingress resource and do the things)
   we will get fully qualified domain name 
   so what happens here ingress controller watches ingress resource and creates the Network load balancer in AWS and same details we will be getting in when we do kubectl get ing (in the addrss column)

   Now still we will not be able to access the website with the address beacuse we have explictly mentioned that in ingress.yaml file to access the website in  (- host: go-web-app.local)..so in corporate or in orgainzation we do such as amazon.com where DNS is already mapped but in our case DNS is not mapped.

   So first thing we will take the load balancer that is created in address column and do nslook up(loadbalancer name)  there we will get the ip address and then we will go to the sudo vim /etc/hosts
   ** So what we are doing here is DNS Mapping : IP Address(that we got) with go-web-app.local
   
   Now we will be able to see our application  running on go-web-app.local


4. Now Helm chart creation:
    
   helm create go-web-app chart
   we can rm -rf charts
   we need to understand chart.yaml ,templates and values.yaml
   Now go the templates folder and remove everything and just copy K8s manifest such deployment.yaml,service.yaml, ingress.yaml
   ** The advantage of helm is that we can variablise our yaml files so we can go to the deployment.yaml
   and in image section we can use: mishrp/go-web-app:{{ .values.image.tag }}
   That means whenever helm is executed it will look for the tags from the values.yaml file where from values.yaml file we will be reading  images.tag
   ** so the deployment.yaml file from templates will go to the values file which is values.yaml go to the image.tag as of now tag is v1 later once we will do cicd then we can update the latest tag per deployment

   Now:

   helm install go-web-app ./go-web-app-chart
   Now everything is installed via helm sucha s deployment,svc, ing.
   Now we can do helm uninstall go-web-app to uninstall everything
   

5. Github Actions
   CI: Build & Test
       static code analysis
       Docker image and push
       update helm

    CD: ArgoCD will pull latest helm chart and deploy it into k8s cluster

    





























CI->
Build and Test,Static code analysis, Docker image and push, Update Helm using Github Actions

CD->
Deploy it kuberentes cluster via argocd







