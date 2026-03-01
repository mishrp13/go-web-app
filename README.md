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
5. Kubernetes -> EKS
6. Helm -> dev, Qa, Prod
7. Ingess Controller -> LB(DNS) -> Exposed

