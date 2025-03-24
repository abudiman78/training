## Replicaset

1. Create a new Replicaset based on the nginx image with 3 replica

   $ vim rs.yaml

https://kubernetes.io/docs/concepts/workloads/controllers/replicaset/

    apiVersion: apps/v1
    kind: ReplicaSet
    metadata: 
      name: nginx-rs
      labels: 
        env: demo
    spec: 
      replicas: 3
      selector:
        matchLabels:
          env: demo
      template:
        metadata:
          name: nginx
          labels:
            env: demo
        spec:
          containers:
            - name: nginx
              image: nginx

    Run the  ReplicaSet:

    $ kubectl apply -f rs.yaml


2. Update the replicas to 4 from the YAML 

 
     $ vim rs.yaml 

update the replicas to 4

      $ kubectl apply -f rs.yaml


3. Update the replicas to 6 from the command line

   
     $ kubectl scale --replicas=6 rs/nginx-rs


## Deployment

1. Create a Deployment named `nginx` with 3 replicas. The Pods should use the `nginx:1.23.0` image and the name `nginx`. The Deployment uses the label `tier=backend`. The 


   https://kubernetes.io/docs/concepts/workloads/controllers/deployment/



    apiVersion: apps/v1
    kind: Deployment
    metadata:
      name: nginx
      labels:
        tier: backend
    spec:
      replicas: 3
      selector:
        matchLabels:
          tier: backend
          app: v1
      template:
        metadata:
          labels:
            tier: backend
            app: v1 
        spec:
          containers:
          - name: nginx
            image: nginx:1.23.0

2. List the Deployment and ensure the correct number of replicas is running.
    
    
    $ kubectl get  deployments
    $ kubectl get deployment nginx
    $ kubectl get pods -l tier=backend

3. Update the image to nginx:1.23.4


    $ vim deploy.yaml

        spec:
          containers:
          - name: nginx
            image: nginx:1.23.4


4. Verify that the change has been rolled out to all replicas.


      $ kubectl rollout status deployment/nginx


5. Assign the change cause “Pick up patch version” to the revision.


      $ kubectl annotate deployment/nginx kubernetes.io/change-cause="Pick up patch version"
      $ kubectl describe deployments/nginx

6. Scale the Deployment to 5 replicas


      $ kubectl scale deployment/nginx --replicas=5

7. Have a look at the Deployment rollout history.


        $ kubectl rollout history deployment/nginx


8. Revert the Deployment to revision 1


    $ kubectl rollout undo deployment/nginx --to-revision=1


9. Ensure that the Pods use the image nginx:1.23.0


    $ kubectl describe pod (nama-pod)


## Troubleshooting 1 

1. We get the below error upon running the this sample yaml.


    $ vim t1-deploy.yaml

    apiVersion: v1
    kind:  Deployment
    metadata:
      name: nginx-deploy
      labels:
        env: demo
    spec:
      template:
        metadata:
          labels:
            env: demo
          name: nginx
        spec:
          containers:
          - image: nginx
            name: nginx
            ports:
            - containerPort: 80
      replicas: 3
      selector:
        matchLabels:
          env: demo

      
    $ kubectl apply -f t1-deploy.yml

    * To fix this —check the deployment API Version 
    * The apiVersion should have apps/v1

2. We get the below error upon running the this sample yaml.


    $ vim t2-deploy.yaml

      apiVersion: v1
      kind:  Deployment
      metadata:
        name: nginx-deploy
        labels:
          env: demo
      spec:
        template:
          metadata:
            labels:
              env: demo
            name: nginx
          spec:
            containers:
            - image: nginx
              name: nginx
              ports:
              - containerPort: 80
        replicas: 3
        selector:
          matchLabels:
            env: dev

    * To fix this —check the deployment API Version 
    * The apiVersion should have apps/v1
    * Match Label must have the same name

Final corrected version:

        apiVersion: apps/v1
        kind:  Deployment
        metadata:
          name: nginx-deploy
          labels:
            env: demo
        spec:
          template:
            metadata:
              labels:
                env: demo
              name: nginx
            spec:
              containers:
              - image: nginx
                name: nginx
                ports:
                - containerPort: 80
          replicas: 3
          selector:
            matchLabels:
              env: demo



    $ kubectl apply -f t2-deploy.yaml


    arief@k8s-master:~/lab/day2$ k apply -f t2-deploy.yaml
    The Deployment "nginx-deploy" is invalid:
    * spec.template.metadata.labels: Invalid value: map[string]string{"env":"demo"}: `selector` does not match template `labels`
     * spec.selector: Invalid value: v1.LabelSelector{MatchLabels:map[string]string{"env":"dev"}, MatchExpressions:[]v1.LabelSelectorRequirement(nil)}: field is immutable

    Here we have an error because because you cannot change the selector field of a Deployment after it has been created. This field is immutable. For this, we have delete and redeploy the yaml again
   
 
    $ kubectl delete deployment nginx-deploy
    $ kubectl apply -f t2-deploy.yaml