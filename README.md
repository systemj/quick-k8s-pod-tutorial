# Kubernetes Pod Tutorial

## Part 0 - kubectl
```
kubectl config current-context    # what cluster
kubectl version                   # client/server versions
kubectl version --short           # readable client server version
kubectl get nodes                 # list cluster nodes
kubectl describe nodes            # lots of node info
```

## Part 1 - As basic as possible

### pod.yaml
```
apiVersion: v1
kind: Pod
metadata:
  name: mypod
spec:
  containers:
    - image: busybox
      name: hello
```

### Create Pod
```
kubectl create -f pod.yaml  # create from declarative configuration
```

### More Info
```
kubectl get pods               # list running pods
kubectl get pods -w            # watch list change
kubectl get pods -o wide       # more information
kubectl get pod mypod          # single pod
kubectl describe pods          # more detail pod info/events
kubectl get pod mypod -o yaml  # single pod full yaml
kubectl get pod mypod -o json  # single pod full json
```

## Part 2 - Container Arguments
### pod.yaml
```
apiVersion: v1
kind: Pod
metadata:
  name: mypod
spec:
  containers:
    - image: busybox
      name: hello
      args:
        - /bin/echo
        - "Hello World!"
```
### Create Pod
```
kubectl create -f pod.yaml  # create from declarative configuration (or apply...)
```

### More Info
```
kubectl get pods            # list running pods
kubectl describe pods       # more detail pod info/events (notice args shown)
kubectl logs mypod          # show pod stdout/err
```

## Part 3 - Command
Sadly the naming is inconsistent/confusing:
```
Description:        | Docker:       | Kubernetes:
--------------------|---------------|-------------
command to run      | entrypoint    | command
command arguments   | cmd           | args
```
  
### pod.yaml
```
apiVersion: v1
kind: Pod
metadata:
  name: mypod
spec:
  containers:
    - image: busybox
      name: hello
      command:
        - '/bin/sh'
        - '-c'
        - '/bin/echo "Hello World!"'
     
```
### Create Pod
```
kubectl create -f pod.yaml  # create from declarative configuration
```

### More Info
```
kubectl get pods            # list running pods
kubectl describe pods       # more detail pod info/events (shows command)
kubectl logs mypod          # show pod stdout/err
```

## Part 4 - Command + Args
### pod.yaml
```
apiVersion: v1
kind: Pod
metadata:
  name: mypod
spec:
  containers:
    - image: busybox
      name: hello
      command:
        - '/bin/sh'
        - '-c'
      args:
        - '/bin/echo "Hello World!"'
     
```
### Create Pod
```
kubectl create -f pod.yaml  # create from declarative configuration
```

### More Info
```
kubectl get pods            # list running pods
kubectl describe pods       # more detail pod info/events (command + args now)
kubectl logs mypod          # show pod stdout/err
```

## Part 5 - Environment
### pod.yaml
```
apiVersion: v1
kind: Pod
metadata:
  name: mypod
spec:
  containers:
    - image: busybox
      name: hello
      command:
        - '/bin/sh'
        - '-c'
      args:
        - '/bin/echo "${MESSAGE}"'
      env:
        - name: MESSAGE
          value: "Hello DFWUUG!"
     
```
### Create Pod
```
kubectl create -f pod.yaml  # create from declarative configuration
```

### More Info
```
kubectl get pods            # list running pods
kubectl describe pods       # more detail pod info/events (command + args now)
kubectl logs mypod          # show pod stdout/err
```

## Part 6 - kubectl exec
Look around a running pod.

### pod.yaml
```
apiVersion: v1
kind: Pod
metadata:
  name: mypod
spec:
  containers:
    - image: busybox
      name: hello
      command:
        - '/bin/sh'
        - '-c'
      args:
        - '/bin/echo "${MESSAGE}" && sleep 1000'
      env:
        - name: MESSAGE
          value: "Hello DFWUUG!"
     
```
### Create Pod
```
kubectl create -f pod.yaml  # create from declarative configuration
```

### More Info
```
kubectl get pods            # list running pods
kubectl describe pods       # more detail pod info/events (updated command + args now)
kubectl logs mypod          # show pod stdout/err
```

### kubectl exec
Like docker exec - enter a running container.

```
kubectl exec -i -t mypod /bin/sh
```

### Look around in the running pod
```
ls        # works as expected
df -h     # mix of container file systems and external  
ps -ef    # only pod processes are visible
free -m   # shows node memory info
uptime    # for node...
```

## Part 7 - Labels and Annotations
Labels are selectors that can be used by Kubernetes (more later).
Annotations are like labels, but for other tools and libraries.

### pod.yaml
```
apiVersion: v1
kind: Pod
metadata:
  name: mypod
  labels:
    app: hello
  annotations:
    appversion: "0.1"
spec:
  containers:
    - image: busybox
      name: hello
      command:
        - '/bin/sh'
        - '-c'
      args:
        - '/bin/echo "${MESSAGE}" && sleep 1000'
      env:
        - name: MESSAGE
          value: "Hello DFWUUG!"
     
```
### Create Pod
```
kubectl create -f pod.yaml  # create from declarative configuration
```

### More Info
```
kubectl get pods            # list running pods
kubectl describe pods       # more detail pod info/events (see labels and annotations...)
```

### Get Pods By Label
Convenient, but labels are really more for use by Kubernetes to do useful things.
```
kubectl get pods -l app=hello  # returns mypod
kubectl get pods -l app=lol    # no resources found
```


## Part 8 - Resource Requests and Limits
Resource requests specifiy a minimum (used by Kubernetes for bin-packing).
Resource limits specifiy a maximum allowed.

### pod.yaml
```
apiVersion: v1
kind: Pod
metadata:
  name: mypod
  labels:
    app: hello
  annotations:
    appversion: "0.1"
spec:
  containers:
    - image: busybox
      name: hello
      command:
        - '/bin/sh'
        - '-c'
      args:
        - '/bin/echo "${MESSAGE}" && sleep 1000'
      env:
        - name: MESSAGE
          value: "Hello DFWUUG!"
      resources:
        requests:
          cpu: "256m"
          memory: "64Mi"
        limits:
          cpu: "1024m"
          memory: "256Mi"
```
### Create Pod
```
kubectl create -f pod.yaml  # create from declarative configuration
```

### More Info
```
kubectl get pods            # list running pods
kubectl describe pods       # more detail pod info/events (notice cpu/memory resource spec)
kubectl describe nodes      # show node details (notice resource utilization)
```

## Part 9 - Ports And Kubectl Port-Forward
Expose container TCP/UDP ports

### pod.yaml
```
apiVersion: v1
kind: Pod
metadata:
  name: mypod
  labels:
    app: hello
  annotations:
    appversion: "0.1"
spec:
  containers:
    - image: nginx
      name: hello
      ports:
        - name: http
          containerPort: 80
```
### Create Pod
```
kubectl create -f pod.yaml  # create from declarative configuration
```

### More Info
```
kubectl get pods            # list running pods
kubectl describe pods       # more detail pod info/events (notice port details)
```

### Port-Forward via kubectl
Connect to pod ports over a secure tunnel with kubectl port-forward
```
kubectl port-forward mypod 8080:80    # now browsable on http://localhost:8080
```

## Part 10 - kubectl cp
Kubectl cp copies files and directories to a running container.
Replace the default Nginx index.html with something else.

### index.html
```
<html>
  <head>
    <title>Hello DFWUUG!</title>
  </head>
  <body>
    <h1>Hello DFWUUG!</h1>
  </body>
</html>
```

### Copy the file into the container
```
kubectl cp index.html mypod:/usr/share/nginx/html/index.html
```

View the updated content in a browser:
```
kubectl port-forward mypod 8080:80    # now browsable on http://localhost:8080
```

### Non-persistent
The content is not persistent and disappears when the pod restarts.

#### Force a pod restart:
```
kubectl exec -i -t mypod /bin/bash
kill 1
```

Check the content in a browser:
```
kubectl port-forward mypod 8080:80    # now browsable on http://localhost:8080 - back to default nginx page
```

## Part 11 - Volumes: emptydir
Persists as long as a pod is on the same node.

### pod.yaml
```
apiVersion: v1
kind: Pod
metadata:
  name: mypod
  labels:
    app: hello
  annotations:
    appversion: "0.1"
spec:
  containers:
    - image: nginx
      name: hello
      ports:
        - name: http
          containerPort: 80
      volumeMounts:
        - name: myhtml
          mountPath: /usr/share/nginx/html
  volumes:
    - name: myhtml
      emptyDir: {}
```
### Create Pod
```
kubectl create -f pod.yaml  # create from declarative configuration
```

### Check content in a browser:
```
kubectl port-forward mypod 8080:80    # browsable on http://localhost:8080 ... no content.. emptyDir
```

### Add Content and check
```
kubectl cp index.html mypod:/usr/share/nginx/html/index.html
kubectl port-forward mypod 8080:80    # browsable on http://localhost:8080 - custom content as expected
```

### Kill pod and check
```
kubectl exec -i -t mypod /bin/bash
kill 1
kubectl port-forward mypod 8080:80    # browsable on http://localhost:8080 - custom content still exists
```

Deleting the pod destroys the emptyDir volume.


## Part 12 - Multiple Containers
Volumes can be shared between containers in a pod.

### pod.yaml
```
apiVersion: v1
kind: Pod
metadata:
  name: mypod
  labels:
    app: hello
  annotations:
    appversion: "0.1"
spec:
  containers:
    - image: nginx
      name: hello
      ports:
        - name: http
          containerPort: 80
      volumeMounts:
        - name: myhtml
          mountPath: /usr/share/nginx/html
    - image: busybox
      name: datewriter
      command:
        - '/bin/sh'
        - '-c'
      args:
        - 'while true ; do date > /content/index.html ; sleep 1 ; done'
      volumeMounts:
        - name: myhtml
          mountPath: /content
  volumes:
    - name: myhtml
      emptyDir: {}
```
### Create Pod
```
kubectl create -f pod.yaml  # create from declarative configuration
```

### More Info
```
kubectl get pods            # list running pods (notice 2/2)
kubectl describe pods       # more detail pod info/events (notice details for each container)
```

### Check content in a browser:
```
kubectl port-forward mypod 8080:80    # browsable on http://localhost:8080 - updating content
```

## Part 13 - Even More Containers
Containers in the same pod share localhost.

### pod.yaml
```
apiVersion: v1
kind: Pod
metadata:
  name: mypod
  labels:
    app: hello
  annotations:
    appversion: "0.1"
spec:
  containers:
    - image: nginx
      name: hello
      ports:
        - name: http
          containerPort: 80
      volumeMounts:
        - name: myhtml
          mountPath: /usr/share/nginx/html
    - image: busybox
      name: datewriter
      command:
        - '/bin/sh'
        - '-c'
      args:
        - 'while true ; do date > /content/index.html ; sleep 1 ; done'
      volumeMounts:
        - name: myhtml
          mountPath: /content
    - image: jcdemo/flaskapp
      name: flaskapp
      ports:
        - name: flask
          containerPort: 5000
  volumes:
    - name: myhtml
      emptyDir: {}
```
### Create Pod
```
kubectl create -f pod.yaml  # create from declarative configuration
```

### Check content in a browser:
```
kubectl port-forward mypod 8080:80    # browsable on http://localhost:8080 - updating content
kubectl port-forward mypod 5000       # browsable on http://localhost:5000 - flask app content
```

### Update Nginx configuration

#### default.conf
```
server {
    listen       80;
    server_name  localhost;
    location / {
        root   /usr/share/nginx/html;
        index  index.html index.htm;
    }
    location /api/ {
      proxy_pass http://localhost:5000/;
    }
}   
```

#### Copy to container
```
kubectl cp default.conf mypod:/etc/nginx/conf.d/default.conf -c hello
```

#### Reload Nginx
```
kubectl exec -i -t mypod -c hello /bin/bash
kill -HUP 1
```

#### Check content in a browser:
```
kubectl port-forward mypod 8080:80    # browsable on http://localhost:8080/     - same date content
                                      # browsable on http://localhost:8080/api/ - flask content
```

## Part 14 - Config Maps
Store configuration as a Kubernetes object.

### Create a configmap from a file:
```
kubectl create configmap --from-file=default.conf my-nginx-config
```

### Show configmap
```
kubectl get configmaps                         # list configmaps
kubectl get configmap my-nginx-config -o yaml  # display configmap content
```

### pod.yaml
```
apiVersion: v1
kind: Pod
metadata:
  name: mypod
  labels:
    app: hello
  annotations:
    appversion: "0.1"
spec:
  containers:
    - image: nginx
      name: hello
      ports:
        - name: http
          containerPort: 80
      volumeMounts:
        - name: myhtml
          mountPath: /usr/share/nginx/html
        - name: nginx-config
          mountPath: /etc/nginx/conf.d
    - image: busybox
      name: datewriter
      command:
        - '/bin/sh'
        - '-c'
      args:
        - 'while true ; do date > /content/index.html ; sleep 1 ; done'
      volumeMounts:
        - name: myhtml
          mountPath: /content
    - image: jcdemo/flaskapp
      name: flaskapp
      ports:
        - name: flask
          containerPort: 5000
  volumes:
    - name: myhtml
      emptyDir: {}
    - name: nginx-config
      configMap:
        name: my-nginx-config
```

### Create Pod
```
kubectl create -f pod.yaml  # create from declarative configuration
```

#### Verify content in a browser:
Make sure the configMap is being used
```
kubectl port-forward mypod 8080:80    # browsable on http://localhost:8080/     - same date content
                                      # browsable on http://localhost:8080/api/ - flask content
```

## Part 15 - Services
Services have their own IPs and use selector (labels) to forward traffic to pods.

### service.yaml
```
apiVersion: v1
kind: Service
metadata:
  name: myservice
spec:
  selector:
    app: hello
  ports:
    - port: 80
      targetPort: 80
  type: LoadBalancer
```

### Create Service
```
kubectl create -f service.yaml
```

### List services
```
kubectl get services    # shows services names, ip(s), ports
```

### Utilize External IPs
Browse to app on service external IP from service.


## Part 16 - Deployments
Deployments allow for replica sets of pods, autoscaling, and rolling updates.

### deployment.yaml
```
apiVersion: apps/v1
kind: Deployment
metadata:
  name: mydeployment
spec:
  replicas: 1
  selector:
    matchLabels:
      app: nginx
  template:

    metadata:
      name: mypod
      labels:
        app: hello
      annotations:
        appversion: "0.1"
    spec:
      containers:
        - image: nginx
          name: hello
          ports:
            - name: http
              containerPort: 80
          volumeMounts:
            - name: myhtml
              mountPath: /usr/share/nginx/html
            - name: nginx-config
              mountPath: /etc/nginx/conf.d
        - image: busybox
          name: datewriter
          command:
            - '/bin/sh'
            - '-c'
          args:
            - 'while true ; do date > /content/index.html ; sleep 1 ; done'
          volumeMounts:
            - name: myhtml
              mountPath: /content
        - image: jcdemo/flaskapp
          name: flaskapp
          ports:
            - name: flask
              containerPort: 5000
      volumes:
        - name: myhtml
          emptyDir: {}
        - name: nginx-config
          configMap:
            name: my-nginx-config
```

### Create the Deployment
```
kubectl create -f deployment.yaml
```

### Show deployment
```
kubectl get deployments                   # list deployments
kubectl describe deployment mydeployment  # additional deployment details
```

### Confirm Content
Browse To Public IP http://service-public-ip/

### Trivial change deployment
#### FROM:
```
args:
  - 'while true ; do date > /content/index.html ; sleep 1 ; done'
```
#### TO:
```
args:
  - 'while true ; do date +"%s" > /content/index.html ; sleep 1 ; done'
```

### Watch Deployment Status

```
kubectl get deployments -w   # show events as the deployment is updated
```

### Update Deployment
```
kubectl apply -f deployment.yaml  # update the existing deployment 
```

### Confirm Updated Content
Browse To Public IP http://service-public-ip/ - now showing date as epoch.




