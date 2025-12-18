🚀 ELK Stack (Elasticsearch + Filebeat + Kibana) Setup on EKS
এই README ফাইলটি AWS EKS cluster-এ ELK Stack setup করার জন্য লেখা। এটি Helm-based, production-style setup এবং Elastic Stack 8.5.1 ব্যবহার করে।
________________________________________
🧱 Architecture Overview
Application Pods
     ↓
Kubernetes Logs (/var/log/containers/*.log)
     ↓
Filebeat (DaemonSet)
     ↓
Elasticsearch (StatefulSet + EBS)
     ↓
Kibana (LoadBalancer UI)
________________________________________
✅ Prerequisites
•	AWS EKS Cluster (Running)
•	kubectl configured
•	Helm v3 installed
•	At least 2 worker nodes (t3.large recommended)
Check:
kubectl get nodes
helm version
________________________________________
📁 Folder Structure
ELK/
├── elasticsearch.yaml
├── filebeat.yaml
├── kibana-values-8.5.1.yaml
├── storageclass.yaml
└── README.md
________________________________________
🔧 Step 1: Install Helm
sudo apt update && sudo apt upgrade -y
curl https://raw.githubusercontent.com/helm/helm/main/scripts/get-helm-3 | bash
Verify:
helm version
________________________________________
📦 Step 2: Add Elastic Helm Repository
helm repo add elastic https://helm.elastic.co
helm repo update
________________________________________
🏷️ Step 3: Create Namespace
kubectl create namespace logging
________________________________________
💾 Step 4: Create StorageClass (AWS EBS)
kubectl apply -f storageclass.yaml
Verify:
kubectl get storageclass
________________________________________
🟢 Step 5: Install Elasticsearch
helm upgrade --install elasticsearch elastic/elasticsearch \
  -n logging \
  -f elasticsearch.yaml
Check:
kubectl get pods -n logging
kubectl get pvc -n logging
Service:
elasticsearch-master:9200
________________________________________
🟡 Step 6: Install Filebeat (Log Collector)
helm upgrade --install filebeat elastic/filebeat \
  --version 8.5.1 \
  -n logging \
  -f filebeat.yaml
Check:
kubectl get pods -n logging | grep filebeat
👉 Filebeat runs as DaemonSet and collects logs from all nodes.
________________________________________
🔵 Step 7: Install Kibana (UI)
helm upgrade --install kibana elastic/kibana \
  --version 8.5.1 \
  -n logging \
  -f kibana-values-8.5.1.yaml
Get External URL:
kubectl get svc -n logging | grep kibana
Access:
http://<EXTERNAL-IP>:5601
________________________________________
🔐 Step 8: Get Elasticsearch Username & Password
Elastic 8.x auto-generates credentials.
kubectl get secret elasticsearch-es-elastic-user -n logging \
  -o go-template='{{.data.elastic | base64decode}}'
Login:
username: elastic
password: <generated-password>
________________________________________
📊 Step 9: Configure Kibana Index Pattern
1.	Open Kibana UI
2.	Go to Stack Management → Index Patterns
3.	Create index pattern:
 	filebeat-*
4.	Select time field: @timestamp
🎉 Logs are now visible!
________________________________________
🧪 Step 10: Test with Sample App
kubectl run test-log --image=busybox -n logging \
  --restart=Never -- sh -c 'while true; do echo hello-elk; sleep 5; done'
Check logs in Kibana.
________________________________________
🔁 Useful Commands
kubectl get all -n logging
kubectl logs -n logging <filebeat-pod>
helm list -n logging
________________________________________
🧠 Notes (Important)
•	Elasticsearch uses EBS-backed PVC
•	Filebeat uses TLS + credentials
•	Kibana is exposed via AWS LoadBalancer
•	Suitable for learning + small production
________________________________________
🚀 Next Improvements (Optional)
•	Index Lifecycle Management (ILM)
•	Log retention (7 / 14 / 30 days)
•	JSON structured logs (Node.js / Java)
•	Replace with AWS OpenSearch (cheaper)
•	Alerting with ElastAlert
________________________________________
✅ Conclusion
This setup provides: - Secure logging pipeline - Kubernetes-native log collection - Persistent Elasticsearch storage - Real-time log visualization
Happy Logging 🚀
