### neo4j-pvc.yaml
```yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: neo4j-data
spec:
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 10Gi
```
---
### neo4j-deployment.yaml
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: neo4j
spec:
  replicas: 1
  selector:
    matchLabels:
      app: neo4j
  template:
    metadata:
      labels:
        app: neo4j
    spec:
      containers:
        - name: neo4j
          image: neo4j:5-enterprise
          ports:
            - containerPort: 7474
            - containerPort: 7687
          env:
            - name: NEO4J_AUTH
              value: "neo4j/password" # укажи свой пароль
            - name: NEO4J_ACCEPT_LICENSE_AGREEMENT
              value: "yes"
            - name: NEO4J_LICENSE
              valueFrom:
                secretKeyRef:
                  name: neo4j-license
                  key: NEO4J_LICENSE
          volumeMounts:
            - name: neo4j-data
              mountPath: /data
      volumes:
        - name: neo4j-data
          persistentVolumeClaim:
            claimName: neo4j-data
```
---
### neo4j-service.yaml
```yaml
apiVersion: v1
kind: Service
metadata:
  name: neo4j
spec:
  type: ClusterIP  # или NodePort, LoadBalancer, если нужен внешний доступ
  selector:
    app: neo4j
  ports:
    - name: http
      port: 7474
      targetPort: 7474
    - name: bolt
      port: 7687
      targetPort: 7687
```
---
### neo4j-ingress.yaml
```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: neo4j-ingress
  annotations:
    nginx.ingress.kubernetes.io/proxy-body-size: 50m
spec:
  rules:
    - host: neo4j.example.com  # поменяй на свой домен
      http:
        paths:
          - backend:
              service:
                name: neo4j
                port:
                  number: 7474
            path: /
            pathType: Prefix
```
---
### neo4j-license-secret.yaml
```yaml
apiVersion: v1
kind: Secret
metadata:
  name: neo4j-license
type: Opaque
stringData:
  NEO4J_LICENSE: |
    твой-лицензионный-ключ-сюда
```
---
