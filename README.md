Deploy Docker App on Kubernetes with LoadBalancer

1.	EC2 Instance Setup
2.	Launch an EC2 instance in us-east-1 (t3.small)
3.	Use Amazon Linux 2023 or Ubuntu 22.04
4.	Open ports in security group: 22 (SSH), 80 (HTTP)
5.	Connect via SSH: ssh -i key.pem ec2-user@<EC2-PUBLIC-IP>
6.	Install Docker

sudo yum update -y 

sudo amazon-linux-extras install docker -y 

sudo systemctl start docker 
 <img width="1013" height="570" alt="image" src="https://github.com/user-attachments/assets/bcbeb56a-cb1e-4d83-aeae-24686fa66a2b" />

sudo systemctl enable docker

sudo usermod -aG docker ec2-user 
exit 

ssh -i key.pem ec2-user@<EC2-PUBLIC-IP> 

docker ps

1.	Install kubectl and kOps
kubectl
curl -LO "https://dl.k8s.io/release/$(curl -L -s https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl" chmod +x kubectl sudo mv kubectl /usr/local/bin/ kubectl version --client

kOps
curl -LO https://github.com/kubernetes/kops/releases/download/v1.27.0/kops-linux-amd64 chmod +x kops- linux-amd64 sudo mv kops-linux-amd64 /usr/local/bin/kops kops version

1.	Create S3 Bucket for kOps State Store

aws configure # region=us-east-1 aws s3 mb s3://kops-shivani-state-store --region us-east-1 export KOPS_STATE_STORE=s3://kops-shivani-state-store

1.	Create Kubernetes Cluster with kOps kops create cluster --name=k8s-demo.k8s.local --zones=us-east-1a --node-count=2 --node-size=t3.small --control-plane-size=t3.small --dns private

kops update cluster k8s-demo.k8s.local --yes

kops validate cluster --wait 10m

kubectl get nodes

1.Build Docker

Image mkdir ~/rc-demo
<img width="1013" height="496" alt="image" src="https://github.com/user-attachments/assets/afd6999c-9505-4ec0-ab91-3e300923e927" />
  
 cd ~/rc-demo

<!DOCTYPE html>

<html>

 <head>
 
  <title>RC Demo</title>

</head>

<body>
  
 <h1>Replication Controller Working 🚀</h1>

</body>

</html>

Dockerfile:

 FROM nginx:latest
 
 COPY index.html /usr/share/nginx/html/index.html
 <img width="1013" height="570" alt="image" src="https://github.com/user-attachments/assets/0f80e0b1-f153-4421-a612-6e756a027896" />

docker build -t shivanibarya/rc-demo:v1 . 

docker run -d -p 8080:80 shivanibarya/rc-demo:v1

Open in browser: http://<EC2-PUBLIC-IP>:8080

1.	Push Docker Image to Docker
 
docker login docker push shivanibarya/rc-demo:v1
<img width="943" height="570" alt="image" src="https://github.com/user-attachments/assets/91f1261d-8c3b-452a-8b13-addf5a3e3709" />

1.	Deploy ReplicationController
rc.yaml:

apiVersion: v1 

kind: ReplicationController 

metadata: name: rc-demo 

spec: replicas: 3 

   selector: app: rc-demo 
   
   template: 
   
   metadata: 
   
   labels: 
   
   app: rc-demo 
   
spec: 

   containers: 
   
    - name: rc-demo-container 
    
       image: shivanibarya/rc-demo:v1 
       
       ports: - containerPort: 80

kubectl apply -f rc.yaml kubectl get rc kubectl get pods

1.	Expose RC via LoadBalancer

service.yaml: 

apiVersion: v1 

kind: Service 

metadata: 

  name: rc-demo-service 
  
spec: 

  type: LoadBalancer 
  
selector: 

   app: rc-demo
   
    ports: - port: 80 
    
     targetPort: 80
<img width="1013" height="570" alt="image" src="https://github.com/user-attachments/assets/03b51812-7831-46d9-b542-963afb0b6b67" />

 
kubectl apply -f service.yaml kubectl get svc
<img width="1013" height="570" alt="image" src="https://github.com/user-attachments/assets/6a33c627-4ba9-49b4-9d2e-6b4869f52c4f" />


Example EXTERNAL-IP: a7236d1948f70406caa6862704d4c4ca-1911316458.us-m east-1.elb.amazonaws.com

<img width="1013" height="570" alt="image" src="https://github.com/user-attachments/assets/feeb786f-e199-44c3-9efd-a169bb0f30d7" />

1.	Access Your App

ELB DNS: http://a7236d1948f70406caa6862704d4c4ca-1911316458.us-east-1.elb.amazonaws.com ELB IPs: nslookup a7236d1948f70406caa6862704d4c4ca-1911316458.us-east-1.elb.amazonaws.com
EXTERNAL-IP output-
 
  
Example: 44.222.49.240, 54.146.213.171
Browser: http://44.222.49.240 http://54.146.213.171 You will see: Replication Controller Working 
<img width="1013" height="518" alt="image" src="https://github.com/user-attachments/assets/6bc30644-5134-4024-95ec-ecbe52ae9177" />



