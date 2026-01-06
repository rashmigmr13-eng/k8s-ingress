Install microk8s:
sudo apt update
sudo snap install microk8s --classic

check the status:
microk8s status --wait-ready

Enable core-add-ons:
microk8s kubectl run curlpod --image=radial/busyboxplus:curl -it --rm --restart=Never -- curl http://flask-service

microk8s enable dns dashboard storage ingress

microk8s kubectl get nodes
microk8s kubectl get pods -A

microk8s kubectl exec -it deploy/flask-app -- curl postgres:5432


curl http://flask.local
<EC2_PUBLIC_IP> flask.local






