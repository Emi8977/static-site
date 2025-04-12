# Sitio Estático en Kubernetes con Minikube


########## IMPORTANTE ############

## Estructura del proyecto ##
devops-k8s-sitio-estatico/
│
├── README.md
├── sitio-estatico/       # Archivos HTML/CSS y otros del sitio
└── k8s-manifiestos/
    ├── deployments/
    ├── services/
    └── volumes/

##################################


Este archivo contiene la configuración necesaria 
para desplegar un sitio web estático utilizando NGINX 
sobre Kubernetes (Minikube).

## Requisitos

- Minikube
- kubectl
- Git
- Navegador con acceso local



##### Pasos para ejecutar el proyecto ####

#1.Crear 1 directorio general donde trabajar ej: "devops-k8s-sitio-estatico"

#Crear dos carpetas dentro de ese directorio
#Primera subcarpeta llamada "k8s-manifiestos"
#Segunda subcarpeta llamada "sitio-estatico"

#2.Abrimos la consola e iniciamos Minikube

minikube start
minikube addons enable ingress

#2.1.Ingresar a la carpeta "sitio-estatico"
#Luego desde la consola (en este caso consola git-bash de GIT en Window)
### a.Clonar el repositorio

gh repo fork ewojjowe/static-website --clone

Esto genera:

    Hacer un fork a tu cuenta de GitHub

    Clonar el fork a tu máquina local (ademas también inicia un repositorio)

Verificamos:
git remote -v

origin  https://github.com/usuarioXX/static-site.git (fetch)
origin  https://github.com/usuarioXX/static-site.git (push)


#Moverse a la carpeta "k8s-manifiestos"
### b.Creamos un repositorio GIT en esta carpeta
 
git init
gh repo create usuarioXX/k8s-manifiesto --public --source=. --remote=origin

Verificamos:
git remote -v

origin  https://github.com/usuarioXX/k8s-manifiesto.git (fetch)
origin  https://github.com/usuarioXX/k8s-manifiesto.git (push)


#Desde la consola: crear varios archivos .yaml vacios

touch persistent-volumen.yaml
touch persistent-volumen-claim.yaml
touch deployment.yaml
touch service.yaml
touch ingress.yaml

#Luego los completo:

persistent-volumen.yaml
____________________________
apiVersion: v1
kind: PersistentVolume
metadata:
  name: website-pv
spec:
  capacity:
    storage: 1Gi
  volumeMode: Filesystem
  accessModes:
    - ReadWriteOnce  # Cambié ReadOnlyMany a ReadWriteOnce
  storageClassName: manual   # Sin clase de almacenamiento, puedes agregarla si es necesario
  hostPath:
    path: /mnt/sitio-estatico  # <== punto de montaje en Minikube  # Verifica que este directorio exista

____________________________
____________________________

persistent-volumen-claim.yaml
____________________________
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: website-pvc
spec:
  accessModes:
    - ReadWriteOnce  # Asegúrate de que coincida con el acceso configurado en el PersistentVolume
  resources:
    requests:
      storage: 1Gi
  storageClassName: manual   # Debe coincidir con el PersistentVolume si es necesario

____________________________
____________________________

deployment.yaml
____________________________
apiVersion: apps/v1
kind: Deployment
metadata:
  name: static-website
spec:
  replicas: 1
  selector:
    matchLabels:
      app: static-website
  template:
    metadata:
      labels:
        app: static-website
    spec:
      containers:
      - name: nginx
        image: nginx:latest
        ports:
          - containerPort: 80
        volumeMounts:
          - name: website-storage
            mountPath: /usr/share/nginx/html
      volumes:
        - name: website-storage
          persistentVolumeClaim:
            claimName: website-pvc

____________________________
____________________________

service.yaml
____________________________
apiVersion: v1
kind: Service
metadata:
  name: static-website
spec:
  selector:
    app: static-website
  ports:
    - protocol: TCP
      port: 80
      targetPort: 80
      nodePort: 30000  # Esto abre el puerto 30000 en tu máquina
  type: NodePort  # Esto cambia el tipo de servicio


____________________________
____________________________

ingress.yaml
____________________________
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: static-website-ingress
  annotations:
    nginx.ingress.kubernetes.io/rewrite-target: /
spec:
  ingressClassName: nginx
  rules:
    - host: static-website.local
      http:
        paths:
          - path: /
            pathType: Prefix
            backend:
              service:
                name: static-website
                port:
                  number: 80
____________________________
____________________________

#3. Nos movemos al directorio principal de trabajo
#Realizamos un montaje

minikube mount "C:\Usuario\Pepe\Desktop\devops-k8s-sitio-estatico\sitio-estatico:/mnt/sitio-estatico"

### - NO CERRAR ESTA CONSOLA

#4. Abrimos otra consola
#Nos movemos a la carpeta de los manifiestos
#Empezamos a aplicar los .yaml

kubectl apply -f .

#3.2. Revisamos lo hemos hecho

kubectl get pods
kubectl describe svc static-website

#4. Verificamos si la pagina web estatica esta funcionando:

minikube service static-website --url

#Nos da una url con puertos efimeros
ej:
http://127.0.0.1:55949
http://127.0.0.1:56104

#4. Agregamos el archivo README.md al repositorio GIT local y en GITHUB

cd ~/desktop/devops-k8s-sitio-estatico
git add README.md

git commit -m "Agregando archivo README.md con instrucciones del proyecto"
git push

#PD: desde la carpeta "k8s-manifiesto"
agregamos los archivos .yaml a git

cd ~/devops-k8s-sitio-estatico/k8s-manifiesto
git add deployment.yaml
git commit -m "Agregando archivo deployment.yaml "
git push -u origin master

git add service.yaml
git commit -m "Agregando archivo service"
git push

git add ingress.yaml
git commit -m "Agregando archivo ingress.yaml "
git push

git add persistent-volumen.yaml
git commit -m "Agregando archivo persistent-volumen"
git push

git add persistent-volumen-claim.yaml
git commit -m "Agregando archivo persistent-volumen-claim"
git push

