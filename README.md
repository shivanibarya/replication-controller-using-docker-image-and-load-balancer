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
 
sudo systemctl enable docker
 sudo usermod -aG docker ec2-user 
exit 
ssh -i key.pem ec2-user@<EC2-PUBLIC-IP> docker ps
1.	Install kubectl and kOps
kubectl
curl -LO "https://dl.k8s.io/release/$(curl -L -s https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl" chmod +x kubectl sudo mv kubectl /usr/local/bin/ kubectl version --client

kOps
curl -LO https://github.com/kubernetes/kops/releases/download/v1.27.0/kops-linux-amd64 chmod +x kops- linux-amd64 sudo mv kops-linux-amd64 /usr/local/bin/kops kops version

1.	Create S3 Bucket for kOps State Store

aws configure # region=us-east-1 aws s3 mb s3://kops-shivani-state-store --region us-east-1 export KOPS_STATE_STORE=s3://kops-shivani-state-store
1.	Create Kubernetes Cluster with kOps kops create cluster
--name=k8s-demo.k8s.local
--zones=us-east-1a
--node-count=2
--node-size=t3.small
--control-plane-size=t3.small
--dns private
kops update cluster k8s-demo.k8s.local --yes kops validate cluster --wait 10m kubectl get nodes

1.Build Docker
Image mkdir ~/rc-demo
  
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

docker build -t shivanibarya/rc-demo:v1 . docker run -d -p 8080:80 shivanibarya/rc-demo:v1
 

Open in browser: http://<EC2-PUBLIC-IP>:8080

1.	Push Docker Image to Docker
 

docker login docker push shivanibarya/rc-demo:v1


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

 
kubectl apply -f service.yaml kubectl get svc
 

Example EXTERNAL-IP: a7236d1948f70406caa6862704d4c4ca-1911316458.us-
east-1.elb.amazonaws.com

1.	Access Your App

ELB DNS: http://a7236d1948f70406caa6862704d4c4ca-1911316458.us-east-1.elb.amazonaws.com ELB IPs: nslookup a7236d1948f70406caa6862704d4c4ca-1911316458.us-east-1.elb.amazonaws.com
EXTERNAL-IP output-
 
  
Example: 44.222.49.240, 54.146.213.171
Browser: http://44.222.49.240 http://54.146.213.171 You will see: Replication Controller Working 

