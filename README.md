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
        - image: demosecretvolume
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

### MultiContainer pod
```
cat <<EOF | kubectl create -f -
apiVersion: extensions/v1beta1
kind: Deployment
metadata:
  labels:
    app: demosecretmulticontainer
  name: demosecretmulticontainer
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
      - image: demosecretmulticontainer
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
