# Run K8s locally with KIND



## Create cluster
`kind create cluster --name demo-cluster`

To check the newly created cluster  
`kind get clusters` 

To get the nodes of a cluster  
`kubectl get nodes`

> This will return the master node of the connected cluster *demo-cluster* 
> `demo-cluster-control-plane`.

Tp get a list of images present on a cluster node use *docker exec*  
`docker exec -it demo-cluster-control-plane crictl images`

## Loading an Image Into Your Cluster
`kind load docker-image docker-image-name --name demo-cluster`

## Creating namespace *[ optional ]*
create a json file named `create-namespace.json` with the following code  
`{  
  "apiVersion": "v1",  
  "kind": "Namespace",  
  "metadata": {  
  "name": "demo-namespace",  
  "labels": {  
  "name": "demo-namespace"  
  }  
 }}`

Finally, create the namespace running   
`kubectl create -f create-namespace.json`  

To check the newly created namespace  
`kubectl get namespaces --show-labels`  



## Writing configuration/manifest file
create a *yaml* file named `demo-config.yaml` with the following code  

    apiVersion: apps/v1
    kind: Deployment
    metadata:
	    name: demo-app
	    namespace: demo-namespace
	spec:
		replicas: 2
		selector:
			matchLabels:
				app: demo-app
		template:
			metadata:
				labels:
					app: demo-app
			spec:
				containers:
					- name: demo-app-container
					  image: docker.io/library/docker-image-name
					  imagePullPolicy: IfNotPresent
					  # env:
					  #  - name: "ENV_VARIABLE_KEY"
					  #    value: "ENV_VARIABLE_VALUE"
					  ports:
						  - containerPort: 15073

now create *deployment* object with `kubectl apply -f demo-config.yaml`

This will create two pods from the docker image that we loaded into the cluster. To check created pods 
`kubectl get pods -n demo-namespace`

## Port forwarding - expose port to local 

- ***port forwarding for a single pod***  
For a single replica deployment object, only 1 pod will be created. To, forward the port of that specific pod in detached mode  
`kubectl port-forward demo-app-77f85d7fd5-9sbbf 15073:15073 -n demo-namespace &`  
*replace* "demo-app-77f85d7fd5-9sbbf " *with your pod name*  

- ***port forwarding for multiple pods (replicas)***  
Create a *service* object named `demo-port-forwarding.yaml` with the following code  

.

    apiVersion: v1
    kind: Service
    metadata:
	    name: demo-app
	    namespace: demo-namespace
	    labels:
		    app: demo-app
	spec:
		type: ClusterIP
		ports:
			- port: 15073
			  targetPort: 15073
		selector:
			  app: demo-app

To apply the configuration in detached mode  
`kubectl port-forward service/demo-app -n demo-namespace 15073:15073 &`  


To check logs of a pod  
`kubectl logs -f demo-app-67b696c99-klthg -n demo-namespace`  
*replace* "demo-app-77f85d7fd5-9sbbf " *with your pod name*
