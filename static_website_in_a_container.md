# Running my website in a container

Octopress generates my site to a directory `public`
```
[igou]$ ls public/
archives  atom.xml  blog  contact  dale  david  favicon.png  font  fonts  images  index.html  javascripts  robots.txt  sitemap.xml  stylesheets
```

Building this into a container is pretty simple:

```
[igou]$ cat Dockerfile
FROM nginx:alpine
COPY public /usr/share/nginx/html
```

To build it

```
[igou]$ docker build -t igou.io-nginx .
[igou]$ docker run --name igou.io-nginx -d -p 8080:80 igou.io-nginx
[igou]$ docker ps
[igou]$ docker commit [id] quay.io/igou/igou.io-nginx
[igou]$ docker push quay.io/igou/igou.io-nginx
```

To deploy it on Kubernetes:

Deployment:
```
apiVersion: apps/v1
kind: Deployment
metadata:
  namespace: igou
  name: igou-website
  labels:
    igou-app: website
spec:
  replicas: 1
  selector:
    matchLabels:
      igou-app: website
  template:
    metadata:
      labels:
        igou-app: website
    spec:
      containers:
      - env:
        image: quay.io/igou/igou.io-nginx:latest
        imagePullPolicy: Always
        name: igou-website
        ports:
        - containerPort: 80
        livenessProbe:
          httpGet:
            scheme: HTTP
            path: /
            port: 80
          initialDelaySeconds: 30
          timeoutSeconds: 30

```

Service:
```
apiVersion: v1
kind: Service
metadata:
  labels:
    igou-app: website
  name: igou-website
  namespace: igou
spec:
  clusterIP: 10.43.227.80
  ports:
  - port: 80
    protocol: TCP
    targetPort: 80
  selector:
    igou-app: website
  sessionAffinity: None
  type: ClusterIP

```

Ingress:
```
apiVersion: v1
items:
- apiVersion: extensions/v1beta1
  kind: Ingress
  metadata:
    labels:
      igou-app: website
    name: igou-website
    namespace: igou
  spec:
    rules:
    - host: websitetest.igou.io
      http:
        paths:
        - backend:
            serviceName: igou-website
            servicePort: 80
```

So let's create them

```
[igou]$ kubectl create igou-blog-deployment.yml
[igou]$ kubectl create igou-blog-service.yml
[igou]$ kubectl create igou-blog-ingress.yml
```


:wala:
