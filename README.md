# quiz-web

## Deployment - Virtual hosting

1. Clone the repo to /var/www by `git clone`.
2. Configure the service like the following examples.
3. To open the site use http://quiz-master.mydomain.hu/
4. To open the editor use http://quiz-master.mydomain.hu/quiz-editor.html

> You can define another virtual hosting for the editor page.

### Apache2

```xml
<VirtualHost *:80>
    DocumentRoot /var/www/quiz-web
    DirectoryIndex quiz-master.html
    ServerName quiz-master.mydomain.hu
    ServerAdmin webmaster@mydomain.hu
    ErrorLog /var/log/apache2/quiz-master.error.log
    CustomLog /var/log/apache2/quiz-master.access.log combined
</VirtualHost>
```

### Nginx

```json
server {
    listen       80;
    server_name  quiz-master.mydomain.hu;

    location / {
        root   /var/www/quiz-web;
        index  quiz-master.html;
    }
}
```

## Deployment - Docker

### Docker

```bash
git clone https://github.com/viktorgirhiny/quiz-web.git
ln -s quiz-master.html quiz-web/index.html
docker run --name quiz-master -v ./quiz-web:/usr/share/nginx/html:ro -p 80:80 -d nginx
```

### Docker Compose

```yaml
services:
  quiz-master:
    image: nginx:latest
    container_name: quiz-master
    ports:
      - "80:80"
    volumes:
      - ./quiz-web:/usr/share/nginx/html:ro
    restart: always
```

## Deployment - Kubernetes

```
apiVersion: apps/v1
kind: Deployment
metadata:
  name: quiz-master
  labels:
    app: quiz
spec:
  replicas: 1
  selector:
    matchLabels:
      app: quiz
  template:
    metadata:
      labels:
        app: quiz
    spec:
      containers:
      - name: nginx
        image: nginx:latest
        ports:
        - containerPort: 80
        volumeMounts:
        - name: html-volume
          mountPath: /usr/share/nginx/html
          readOnly: true
      volumes:
      - name: html-volume
        hostPath:
          path: /abszolút/útvonal/a/quiz-web-hez # <-- CHANGE THIS OR USE PV/PVC
          type: Directory
---
apiVersion: v1
kind: Service
metadata:
  name: quiz-service
spec:
  selector:
    app: quiz
  ports:
    - protocol: TCP
      port: 80
      targetPort: 80
  type: LoadBalancer
---
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: quiz-ingress
  annotations:
    kubernetes.io/ingress.class: nginx
spec:
  rules:
  - host: quiz-master.mydomain.hu
    http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: quiz-service
            port:
              number: 80
```
