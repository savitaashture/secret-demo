## secret-demo

## To view system created secrets
```
kubectl get secrets
```
## Verify how system created secrets will be used by pods
```
cat <<EOF | kubectl create -f -
apiVersion: apps/v1
kind: Deployment
metadata:
  labels:
    app: test
  name: test
spec:
  replicas: 1
  selector:
    matchLabels:
      app: test
  template:
    metadata:
      labels:
        app: test
    spec:
      containers:
      - image: nginx
        name: nginx
EOF
```

## Delete deployment
```
kubectl delete deployment test
```

## Create secrets manually
```
echo -n 'demo' | base64
echo -n 'secret' | base64

cat <<EOF | kubectl create -f -
apiVersion: v1
kind: Secret
metadata:
  name: test-secret
type: Opaque
data:
  username: ZGVtbw==
  password: c2VjcmV0
EOF
```
## Create secrets using kubectl
```
echo -n 'demo' > ./username.txt
echo -n 'secret' > ./password.txt

kubectl create secret generic db-user-pass --from-file=./username.txt --from-file=./password.txt
```
## Secret Mount
### Volume Mount
```
cat <<EOF | kubectl create -f -
apiVersion: extensions/v1beta1
kind: Deployment
metadata: 
  labels: 
    app: demosecretvolume
  name: demosecretvolume
spec: 
  replicas: 1
  selector: 
    matchLabels: 
      app: demosecretvolume
  template: 
    metadata: 
      labels: 
        app: demosecretvolume
    spec: 
      containers: 
        - image: nginx
          imagePullPolicy: Always
          name: demosecretvolume
          ports: 
            - containerPort: 80
              protocol: TCP
          resources: {}
          volumeMounts: 
            - mountPath: /tmp/demo
              name: foo
              readOnly: true
      restartPolicy: Always
      volumes: 
        - name: foo
          secret: 
            secretName: test-secret
EOF		  
```
### Using Environment Variable
```
cat <<EOF | kubectl create -f -
apiVersion: extensions/v1beta1
kind: Deployment
metadata: 
  labels: 
    app: demosecretenv
  name: demosecretenv
spec: 
  replicas: 1
  selector: 
    matchLabels: 
      app: demosecretenv
  template: 
    metadata: 
      labels: 
        app: demosecretenv
    spec: 
      containers: 
        - env: 
            - name: SECRET_USERNAME
              valueFrom: 
                secretKeyRef: 
                  key: username
                  name: test-secret
            - name: SECRET_PASSWORD
              valueFrom: 
                secretKeyRef: 
                  key: password
                  name: test-secret
          image: nginx
          imagePullPolicy: Always
          name: demosecretenv
          ports: 
            - containerPort: 80
              protocol: TCP
          resources: {}
      restartPolicy: Always
EOF
```
### ImagePullSecrets
```
kubectl create secret docker-registry myregistrykey --docker-server=DOCKER_REGISTRY_SERVER --docker-username=DOCKER_USER --docker-password=DOCKER_PASSWORD --docker-email=DOCKER_EMAIL

secret/myregistrykey created.

apiVersion: v1
kind: Pod
metadata:
  name: demopull
spec:
  containers:
    - name: demopull
      image: myimage
  imagePullSecrets:
    - name: myregistrykey
```
### MultiContainer pod
```
cat <<EOF | kubectl create -f -
apiVersion: v1
kind: Pod
metadata:
  name: demosecretmulticontainer
spec:
  containers:
  - image: nginx
    env:
      - name: SECRET_USERNAME
        valueFrom:
          secretKeyRef:
            name: test-secret
            key: username
      - name: SECRET_PASSWORD
        valueFrom:
          secretKeyRef:
            name: test-secret
            key: password
    imagePullPolicy: Always
    name: demosecretmulticontainer
    ports:
    - containerPort: 80
      protocol: TCP
    resources: {}
  - image: redis
    imagePullPolicy: Always
    name: redis
EOF
```
## Verification
```
kubectl exec -it demosecretmulticontainer -c demosecretmulticontainer env | grep SECRET_
SECRET_USERNAME=admin
SECRET_PASSWORD=a62fjbd37942dcs

kubectl exec -it demosecretmulticontainer-687b95d57c-7h6v5 -c redis env | grep SECRET_
```
