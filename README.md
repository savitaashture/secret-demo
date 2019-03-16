## secret-demo

## To view system created secrets
```
kubectl create ns demo
kubectl get secrets -n demo
```
## Create secrets manually
```
echo -n 'demo' | base64
echo -n 'secret' | base64

cat <<EOF | kubectl create -f -
apiVersion: v1
kind: Secret
metadata:
  name: demosecret
  namespace: demo
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

kubectl create secret generic db-user-pass --from-file=./username.txt --from-file=./password.txt -n demo
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
  namespace: demo
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
            secretName: demosecret
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
  namespace: demo
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
                  name: demosecret
            - name: SECRET_PASSWORD
              valueFrom: 
                secretKeyRef: 
                  key: password
                  name: demosecret
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
  namespace: demo
spec:
  containers:
    - name: demopull
      image: nginx
  imagePullSecrets:
    - name: myregistrykey
```
### MultiContainer pod
```
cat <<EOF | kubectl create -f -
apiVersion: extensions/v1beta1
kind: Deployment
metadata:
  labels:
    app: demosecretmulticontainer
  name: demosecretmulticontainer
  namespace: demo
spec:
  replicas: 1
  selector:
    matchLabels:
      app: demosecretmulticontainer
  template:
    metadata:
      labels:
        app: demosecretmulticontainer
    spec:
      containers:
      - image: nginx
        env:
          - name: SECRET_USERNAME
            valueFrom:
              secretKeyRef:
                name: demosecret
                key: username
          - name: SECRET_PASSWORD
            valueFrom:
              secretKeyRef:
                name: demosecret
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
      restartPolicy: Always
EOF
```
