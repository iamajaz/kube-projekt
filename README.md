Stage 1:  
    Provision a ec2 machine of the required instance type  

Stage 2:  
    Install Minikube Cluster & start cluster  

    ```console
    curl -LO https://storage.googleapis.com/minikube/releases/latest/minikube-linux-amd64
    sudo install minikube-linux-amd64 /usr/local/bin/minikube
    ```  

    Install kubectl  

    ```console
    curl -LO https://storage.googleapis.com/kubernetes-release/release/$(curl -s https://storage.googleapis.com/kubernetes-release/release/stable.txt)/bin/linux/amd64/kubectl
    chmod +x ./kubectl
    sudo mv ./kubectl /usr/local/bin/kubectl
    ```  

    Start Minikube Cluster  

    ```console
    minikube start
    ```  

Stage 3:  
    Create “hello world” node.js service / container.  
    So I do not have much idea on node.js so i have tried to use a publicly available node js "hello world" app   

Stage 4:  
    Convert the node.js to dockerfile and push the same to docker registry, i've written this dockerfile though    

    Dockerfile: Location: kube-projekt/node-hello-master  
    ```console
    FROM node:14
    WORKDIR /usr/src/app
    COPY package*.json index.js ./
    RUN npm install
    EXPOSE 3000
    CMD ["node", "index.js"]
    ```  

Stage 5:  
    Build, check & push docker image without cicd from the node-hello-master  

    ```console
    docker build -t first-try-node .
    ```  
    
    Test if the node js container is in running state or not
    ```console
    docker run -d -p 3000:3000 --name node-app first-try-node
    ```  

    Do a docker login and push the image created
    ```console
    docker tage first-try-node iamajaz/first-try-node
    docker image push iamajaz/first-try-node
    ```  

Stage 6:
    Leverage the image to run in the kubernetes cluster, convert the resource structure 
    to a more appropriate kubernetes resource definition.

    kube-projekt/kube-resources
    ```console
    apiVersion: apps/v1
    kind: Deployment
    metadata:
      name: nodejs-dep
    spec:
      replicas: 1
      template:
        metadata:
          labels:
            app: nodejs
        spec:
          containers: 
          - name: nodejs
            image: iamajaz/first-try-node
            ports:
            - containerPort: 3000
    ```

Stage 7:
    Deploy the node.js

    ```console
    kubectl create -f node-js.yaml
    ```

Stage 8:
    Expose the deployment

    ```console
    kubectl expose deployment nodejs-dep --type="LoadBalancer"
    ```

Stage 9:
    Scale the node js manually.  

    ```console
    kubectl scale --replicas=2 deployment/nodejs-dep
    ```  

Stage 10:
    Scaling using the HPA Kubernetes  

    Create and deploy HPA resource kube-project/kube-resources/auto-scale-nodejs.yaml
    ```console
    apiVersion: autoscaling/v1
    kind: HorizontalPodAutoscaler
    metadata:
    name: nodejs-hpa
    spec:
    scaleTargetRef:
        apiVersion: apps/v1
        kind: Deployment
        name: test-nodejs
    minReplicas: 1
    maxReplicas: 10
    targetCPUUtilizationPercentage: 85
    ```  

    ```console
    kubectl create -f kube-project/kube-resources/auto-scale-nodejs.yaml
    ```  

    Now with this whenever there is a CPU utilization of greater than 85%, the service will be horizontally scaled

Stage 11:  

    For the CI/CD Pipeline
    
    Have a jenkins machine setup up and running 
    
    Connect Jenkins to the kubernetes cluster  
    
    ```console
    $ sudo cp ~/.kube/config ~jenkins/.kube/$ sudo chown -R jenkins: ~jenkins/.kube/
    ```



    We now create a jenkinsfile pipeline, which does the following
    The code can be read at kube-project/kube-resources/jenkins/Jenkinsfile
    ```console
    Node js build
    Docker Build
    Docker Push
    Apply Kubernetes Files
    ```

    So idea of the whenever we trigger the jenkins build, it is create a new app and the same would be then pushed to the docker registry and would be applied on the kubernetes cluster  

Stage 12:
    To Use secrets to a pod, we can do the following  

    For API keys, encode the API key using base_64  
    ```
    echo -n <DD_API_KEY> | base64
    ```  

    ```console
    apiVersion: v1
    kind: Secret
    metadata:
    name: mysecret
    labels:
        app: "mysecret"
    type: Opaque
    data:
    api-key: "<YOUR_BASE64_ENCODED_DATADOG_API_KEY>"
    ```  

    In the pod defintion we can append the follwing under the spec
    ```console
    valueFrom:
    secretKeyRef:
        name: mysecret
        key: api-key
    ```  