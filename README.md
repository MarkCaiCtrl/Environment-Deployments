# 1. Create namespace for production environment
```
kubectl create namespace production
```
# 2. Configure Environment-Specific ConfigMaps.
```
kubectl create configmap app-config \
  --from-literal=APP_ENV=production \
  --namespace=production
```
# 3. Production SECRETS
```
kubectl create secret generic app-secret \
  --from-literal=DB_PASSWORD=prodpassword123 \
  --namespace=production
```
# 4. Create Production YAML file
```
sed -e 's/PLACEHOLDER_NAMESPACE/production/‘ \
    -e 's/replicas: .*$/replicas: 3/‘ app-deployment-template.yaml > production-deployment.yaml
```
```
**if previous app deployment template is present

**if not present:

apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-app
  namespace: production
spec:
  replicas: 3
  selector:
    matchLabels:
      app: nginx-app
  template:
    metadata:
      labels:
        app: nginx-app
    spec:
      containers:
      - name: nginx
        image: nginx
        env:
        - name: APP_ENV
          valueFrom:
            configMapKeyRef:
              name: app-config
              key: APP_ENV
        - name: DB_PASSWORD
          valueFrom:
            secretKeyRef:
              name: app-secret
              key: DB_PASSWORD
```
# 4. Expose the production environment
```
kubectl expose deployment nginx-app \
  --type=NodePort \
  --name=nginx-service \
  --port=80 \
  --target-port=80 \
  --namespace=production
```
# 5. Check if pods and service were created
```
kubectl get pods -n production
NAME                         READY   STATUS    RESTARTS   AGE
nginx-app-65968468b6-d8j9x   1/1     Running   0          40s
nginx-app-65968468b6-nzkxr   1/1     Running   0          40s
nginx-app-65968468b6-qr2wv   1/1     Running   0          40s
```
```
kubectl get service -n production
NAME            TYPE       CLUSTER-IP      EXTERNAL-IP   PORT(S)        AGE
nginx-service   NodePort   10.100.67.147   <none>        80:30096/TCP   30s
```
# 6. Access the production app:
```
kubectl exec -it <pod-name> -n production -- /bin/bash
```
# 7. Echo env variables to check
```
echo $APP_ENV
production
```
```
echo $DB_PASSWORD
prodpassword123
```
``
