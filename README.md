# **TP Kubernetes Ingress**
# 

1. Installer Kind et créer votre premier cluster

```
curl -Lo ./kind https://kind.sigs.k8s.io/dl/v0.17.0/kind-linux-amd64
chmod +x ./kind
sudo mv ./kind /usr/local/bin/kind
```
- Pour créer le cluster, voici la commande :

```
kind create cluster --name cluster-kind1
```

2. Installer le Nginx ingress Controller

**Partie 1 :**

![](https://i.imgur.com/Qw5HBRh.png)

```
Creating cluster "kind" ...
 ✓ Ensuring node image (kindest/node:v1.25.3) 🖼
 ✓ Preparing nodes 📦  
 ✓ Writing configuration 📜 
 ✓ Starting control-plane 🕹️ 
 ✓ Installing CNI 🔌 
 ✓ Installing StorageClass 💾 
Set kubectl context to "kind-kind"
You can now use your cluster with:

kubectl cluster-info --context kind-kind
```

**Partie 2 :**

```
Root@Nathan:~/tp2$ kubectl apply -f https://raw.githubusercontent.com/kubernetes/ingress-nginx/main/deploy/static/provider/kind/deploy.yaml
namespace/ingress-nginx created
serviceaccount/ingress-nginx created
serviceaccount/ingress-nginx-admission created
role.rbac.authorization.k8s.io/ingress-nginx created
role.rbac.authorization.k8s.io/ingress-nginx-admission created
clusterrole.rbac.authorization.k8s.io/ingress-nginx created
clusterrole.rbac.authorization.k8s.io/ingress-nginx-admission created
rolebinding.rbac.authorization.k8s.io/ingress-nginx created
rolebinding.rbac.authorization.k8s.io/ingress-nginx-admission created
clusterrolebinding.rbac.authorization.k8s.io/ingress-nginx created
clusterrolebinding.rbac.authorization.k8s.io/ingress-nginx-admission created
configmap/ingress-nginx-controller created
service/ingress-nginx-controller created
service/ingress-nginx-controller-admission created
deployment.apps/ingress-nginx-controller created
job.batch/ingress-nginx-admission-create created
job.batch/ingress-nginx-admission-patch created
ingressclass.networking.k8s.io/nginx created
validatingwebhookconfiguration.admissionregistration.k8s.io/ingress-nginx-admission created
```


3. Compléter le schéma suivant avec des objets Kubernetes

![](https://i.imgur.com/kILNAXL.png)


4. Builder et publier (à partir de l’image nginx) sur le DockerHub, une image docker pour chacun des sites web présent sur le schéma précédent. Vous devez avoir 3 images (une par magasin tacos, pizzas et burgers)

- Build des images

```
docker build -t pizza -f Dockerfile .
docker build -t burger -f Dockerfile .
docker build -t tacos -f Dockerfile .
```
- Tag des images

```
Root@Nathan:~/tp2/config-dockerfile$ docker tag pizza nathanynov/restaurants:pizza
Root@Nathan:~/tp2/config-dockerfile$ docker tag burger nathanynov/restaurants:burger
Root@Nathan:~/tp2/config-dockerfile$ docker tag tacos nathanynov/restaurants:tacos
```

- Push des images

```
docker push nathanynov/restaurants:pizza 
docker push nathanynov/restaurants:burger
docker push nathanynov/restaurants:tacos
```

5. Ecrire les fichiers yaml vous permettant de déployer sur votre cluster kind installé en local les composants décrits sur le schéma de la question 3 et les images crées à la question 4

- Arborescence update avec le fichiers ingress.yml et conf.yml pour chaque restaurants : 

```
└── tp2
    ├── pizza
    │   ├── ConfPizza.yml
    │   ├── Dockerfile
    │   └── index.html
    ├── ingress.yml
    ├── burger
    │   ├── ConfBurger.yml
    │   ├── Dockerfile
    │   └── index.html
    └── tacos
        ├── ConfTacos.yml
        ├── Dockerfile
        └── index.html
```

Fichier ingress.yml

```
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: ingress-resource-backend-restaurants
  annotations: 
    nginx.ingress.kubernetes.io/rewrite-target: /
spec:
  rules:
  - host: mypizza.eatsout.com
    http:
      paths:
        - pathType: Prefix
          path: "/"
          backend:
            service:
              name: nginx-pizza-web-service
              port:
                number: 80
  - host: burgerandtacos.eatsout.com
    http:
      paths:
        - pathType: Prefix
          path: "/burgers"
          backend:
            service:
              name: nginx-burger-web-service
              port:
                number: 80
        - pathType: Prefix
          path: "/tacos"
          backend:
            service:
              name: nginx-tacos-web-service
              port:
                number: 80
```

Fichier ConfPizza.yml

```
apiVersion: v1
kind: Service
metadata:
  name: nginx-pizza-web-service
spec:
  selector:
    app: nginx-web-pizza
  ports:
  - name: http
    port: 80
    targetPort: 80
  type: ClusterIP
  
---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-pizza-web-deployment
spec:
  replicas: 1
  selector:
    matchLabels:
      app: nginx-web-pizza
  template:
    metadata:
      labels:
        app: nginx-web-pizza
    spec:
      containers:
      - name: cont-pizza
        image: nathanynov/restaurants:pizza
        ports:
        - containerPort: 80
```

Fichier ConfBurger.yml

```
apiVersion: v1
kind: Service
metadata:
  name: nginx-burger-web-service
spec:
  selector:
    app: nginx-web-burger
  ports:
  - name: http
    port: 80
    targetPort: 80
  type: ClusterIP

---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-burger-web-deployment
spec:
  replicas: 1
  selector:
    matchLabels:
      app: nginx-web-burger
  template:
    metadata:
      labels:
        app: nginx-web-burger
    spec:
      containers:
      - name: cont-burger
        image: nathanynov/restaurants:burger
        ports:
        - containerPort: 80
```

Fichier ConfTacos.yml

```
apiVersion: v1
kind: Service
metadata:
  name: nginx-tacos-web-service
spec:
  selector:
    app: nginx-web-tacos
  ports:
  - name: http
    port: 80
    targetPort: 80
  type: ClusterIP

---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-tacos-web-deployment
spec:
  replicas: 1
  selector:
    matchLabels:
      app: nginx-web-tacos
  template:
    metadata:
      labels:
        app: nginx-web-tacos
    spec:
      containers:
      - name: cont-tacos
        image: nathanynov/restaurants:tacos
        ports:
        - containerPort: 80
```


6. Votre magasin de tacos devient très populaire (il va avoir 3 fois plus de commandes). Il va vous falloir gérer une charge importante sur le Service de commande des tacos. Comment gérez-vous cela ? Comment vérifier que la charge est bien répartie (avec quelle commande kubectl ?) ?


Pour vérifier que la charge soit bien répartie, on peut utiliser la commande “kubectl top pod”.
Elle permet de donner des informations sur l’utilisation des ressources sur les pods.

