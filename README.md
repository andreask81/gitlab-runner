# Gitlab runner on Kubernetes
* Create a kubernetes namespace for gitlab runners.   

```
kubectl create namespace gitlab
```

* Go to the folder that you have cloned the source.
* Create a ConfigMap in kubernetes, as follows:

```
kubectl apply -n gitlab -f configmap.yml
```

* Create a PV claim for maven cache, as follows:

```
kubectl apply -n gitlab -f pv-claim.yml
```

* Create a Deployment which uses the above ConfigMap, as follows:

```
kubectl apply -n gitlab -f deploy.yml
```

* Attach to the created gitlab-runner pod:

```
kubectl -n gitlab exec -it gitlab-runner-#########-##### bash
```

and run the following command to register the runner to gitlab:
```
gitlab-runner register
```
```
Running in system-mode.                            
                                                   
Please enter the gitlab-ci coordinator URL (e.g. https://gitlab.com/):
https://my-gitlab.com/
Please enter the gitlab-ci token for this runner:
xxxxxxxxxxxxxxxxxxxx
Please enter the gitlab-ci description for this runner:
[gitlab-runner-64f57bb5dc-c6qgf]: gitlab-runner-my-gitlab-group
Please enter the gitlab-ci tags for this runner (comma separated):

Registering runner... succeeded                     runner=xxxxxxxx
Please enter the executor: docker-ssh+machine, kubernetes, docker, parallels, shell, virtualbox, docker+machine, docker-ssh, ssh:
kubernetes 
Runner registered successfully. Feel free to start it, but if it's running already the config should be automatically reloaded! 
```
Note: when asked for gitlab-ci token provide the one displayed under Group → Settings → CI/CD → Runner Settings → Group Runners → Setup a group Runner manually

* After the runner registration move to Gitlab’s Admin Area→ Runners →  Click on the created runner → get Token → and add it to the “Token” attibute in the configmap.yml file

Run again:

```
kubectl apply -n gitlab -f configmap.yml
```

* Run the following yaml file in kubernetes so that the default account of the created namespace can access all the resources:

```
kubectl apply -n gitlab -f role.yml
```

* Delete the gitlab-runner pod so that it gets restarted carrying the latest configuration changes.

```
kubectl delete pod -n gitlab gitlab-runner-#########-#####
```

* You should be now ready to run you builds on the Group’s projects using the kubernetes cluster   
  (Go to Project Settings → CI/CD → Check that only the group runner is enabled (green icon)) 


# gitlab CI yaml
* In your .gitlab-ci.yml include the following:

```
  cache:
    paths:
    - .m2/repository/
    - target/  
```
