# Google kubernetes engine templates


## Started and create cluster 

### First configuration
```sh
# create project
gcloud projects create [project_id] --name="[Project name]"


# Set default project
gcloud config set project [project_id]

# Enable containers
gcloud services enable container.googleapis.com
```

### Create / delete cluster

```sh
# Create cluster  
gcloud container clusters create-auto my-cluster \
  --region us-west1 \
  --project safari-gke-462517

# Check the clusters
gcloud container clusters list --region us-central1

# Delete cluster
gcloud container clusters delete my-free-cluster \
    --region us-central1 \
    --project=safari-gke-462517

```

### Check other resources

```sh
# Check the Master Node and other created resources
gcloud compute disks list --filter="zone:us-central1"
gcloud container images list --repository=gcr.io/safari-gke-462517

gcloud compute disks delete NOMBRE_DISCO --zone=us-central1-a
gcloud container images delete gcr.io/TU_PROYECTO_ID/IMAGEN --force-delete-tags
```

## Pods

```sh
# Create a pod that it keeps up
kubectl run my-pod --image=alpine -l "env=dev,author=labs" sleep infinity

# Create a nginx web services
kubectl run my-nginx --image=nginx

# Mode interative command
kubectl exec my-pod -it -- sh

# Execute this command and show the result in screen
kubectl exec my-pod -- date

# Check the pods
watch -n 1 kubectl get pod
kubectl get pod -o wide
kubectl get event --watch

# Check logs
kubectl logs -f [pod_name]

kubectl port-forward  my-nginx 8080:80

# delete pod 
kubectl delete pod/[pod_name]

# Get pod with specified labels
kubectl get pods -L env,author
kubectl get pods -l author!=labs
```

## Using manifest to create pod

```sh
kubectl create namespace [namespace_name]
kubectl apply -f file.yaml -n [namespace_name]
kubectl get pod -n [namespace_name]


```

### Pod configuration
- volumen
- Life cycle (post start, pre stop)
- Liveness probe: Propósito: Indica si el contenedor está listo para aceptar solicitudes.
  - Una vez que el contenedor está corriendo.
  - Se ejecuta periódicamente.
  - Si la readinessProbe falla, el pod se saca del Service (no recibe tráfico).
  - Si pasa, el pod entra al balanceador de carga y empieza a recibir tráfico.
- Readiness probe: Verifica si el contenedor está en un estado saludable o necesita ser reiniciado.
  - Después de un delay (initialDelaySeconds).
  - Se ejecuta regularmente mientras el contenedor está en ejecución.
  - Si la livenessProbe falla repetidamente, Kubernetes reinicia el contenedor.

```yaml
# nginx-self-healing.yaml
apiVersion: v1
kind: Pod
metadata:
  name: nginx-self-healing
spec:
  containers:
    - image: nginx
      name: nginx
      readinessProbe:
        exec:
          command:
            - /bin/cat
            - /ready.txt
        initialDelaySeconds: 15   # Wait before starting the probe
        timeoutSeconds: 1         # Tolerate latency (e.g., slow web server)
        periodSeconds: 15         # How long to wait before polling again
        failureThreshold: 3       # Number of failures before flagging as not live
        successThreshold: 1       # Number of successes before clearing failure count
      livenessProbe:
        httpGet:
          path: /index.html
          port: 80
        initialDelaySeconds: 5    # Wait before starting the probe
        timeoutSeconds: 1         # Tolerate latency (e.g., slow web server)
        periodSeconds: 10         # How long to wait before polling again
        failureThreshold: 3       # Number of failures before flagging as not live
        successThreshold: 1       # Number of successes before clearing failure count
  restartPolicy: Always
```

### Labels

```sh
kubectl get pod -l author!=labs
kubectl get pod -l author in (labs,user2)
```