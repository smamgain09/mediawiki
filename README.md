**Automated Deployment of Mediawiki using Helm and CloudBuild on GKE Cluster **

1. Connect to GKE created cluster
   > gcloud container clusters get-credentials saur-kube --zone us-central1-c --project winter-accord-276603
 
2. Update project config at root after doing sudo login
 
3. Check the status of helm and kubectl 
   > kubectl cluster-info  
   > kubectl get nodes && helm ls
 
4. Add the required basic helm repos 
 
 5. Get the required helm repo from official bitnami chart
   > helm pull bitnami/mediawiki –untar

6. Change the values of Mediawiki user, email and Mediwiki name, DB name, DB user, email and    
   password (if any) in values.yaml file

7. Deploy the helm release using below command :
   > helm install mediawiks ./mediawiki --debug > ./mediawiki/helmout.log
      #helmout.log consists of debug logs of first deployment.
    It creates 8 different Kubernetes resources.
 
8. Check the helm release
   > helm ls     

9. Perform a helm upgrade to upgrade the release and deploy the remaining mediawiki pod and service.
   > helm status mediawiks
  
  Perform below steps :
  
  Get the Mediawiki URL by running:
  Watch the status with: 'kubectl get svc --namespace default -w mediawiks-mediawiki'
  export APP_HOST=$(kubectl get svc --namespace default mediawiks-mediawiki --template "{{ range  (index .status.loadBalancer.ingress 0) }}{{ . }}{{ end }}")
  export APP_PASSWORD=$(kubectl get secret --namespace default mediawiks-mediawiki -o jsonpath="{.data.mediawiki-password}" | base64 --decode)
  export MARIADB_ROOT_PASSWORD=$(kubectl get secret --namespace default mediawiki-mariadb -o jsonpath="{.data.mariadb-root-password}" | base64 --decode)
  export MARIADB_PASSWORD=$(kubectl get secret --namespace default mediawiki-mariadb -o jsonpath="{.data.mariadb-password}" | base64 --decode)

  Complete your Mediawiki deployment by running:
  helm upgrade mediawiks bitnami/mediawiki \
  --set mediawikiHost=$APP_HOST,mediawikiPassword=$APP_PASSWORD,mariadb.auth.rootPassword=$MARIADB_ROOT_PASSWORD,mariadb.auth.password=$MARIADB_PASSWORD
  export SERVICE_IP=$(kubectl get svc --namespace default mediawiks-mediawiki --template "{{ range (index .status.loadBalancer.ingress 0) }}{{.}}{{ end }}")
  echo "Mediawiki URL: http://$SERVICE_IP/"

10. After the upgrade check helm release and pod and service status:
        
11. The deployment type is RollingUpdate as mentioned in the Helm Chart.
 
12. Scale the deployment up or down
   > kubectl get all
 
   Scale up the pods to 3 using kubectl scale command : 
   > kubectl scale –replicas 3 pod/mediawiks-mediawiki-cf5d54b9-6vq9p 
   > kubectl scale --replicas 2 deployment/my-todo-app-mean

13. Performing rolling updates or rollout
   > helm status mediawiks
 
   To upgrade the release with rolling update strategy already defined within helm chart
   > helm upgrade mediawiks .
   To rollback to previous revision (revision number 1 here)
   > helm rollback mediawiks 1
   > helm history mediawiks
 
There's also a setup with cloudbuild.yaml script in the project which facilitates the updation of image for mediawiki or if anybody wants to use a customised image for it. 
The image name can be changed from the GCR in the build script. 

The pre-configured SCM trigger will execute the build and will upgrade the current helm release.
