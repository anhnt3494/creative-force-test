# creative-force-test
The infrastructure has been presented in the acme.drawio(draw.io) and acme.pdf. This architecture was built based on aws cloud environment.
Here is the resources that i used:
  - EKS cluster: for running image and control pod
      - eks cluster create using cluster.yaml file
      - 3 namespace: ui, api and kube-system
          - ui and api namespace running ui and api pod
          - kube-system namespace running k8s core pod, metric pod and contaner insight pod
          - the k8s-config for the ui and api deployment has been presented in the ui-k8s-config.yaml and api-k8s-config.yaml
          - because the architecture is simple and not need complicate requirement then i use fargate to running those pod
          - we will have another namespace aws-observability for ship fargate log to opensearch. the config map for shipping log create using aws-logging-opensearch-configmap.yaml
      - 2 ALB ingress: ui ingress and api ingress
          - 2 ingress create using ui-ingress.yaml and api-ingress.yaml
  - route53 for create and control dns record for domain acme.com and ACM to create and store SSL certificate.
      - route53 have 2 record to point domain www.acme.com to ui ingress endpoint and api.acme.com to api ingress endpoint
      - ACM should have 1 wildcard SSL certificate *.acme.com or 2 SSL certificate www.acme.com and api.acme.com
  - I am using aurora postgres serverless for minimize effort to maintain the DB. if we need a better price then using RDS.
  - opensearch for store and search systemlog. base on my config each namespace will have diffrent index log in the opensearch so it very easy to find log. for the cheaper price we can using cloudwatch to store these logs.
  - For the pod metric i using container insights to collect and push metric to cloudwatch. from cloudwatch i can visualize those metric and create alarm to trigger SNS to send email to me.
             ![image](https://github.com/anhnt3494/creative-force-test/assets/145828302/d3badf8b-645d-4072-88c0-03d8ad6a1eb8)

  - ECR for store image. we can use scan method of ecr to check the vulnerability of the image.
  - S3 bucket for store file, document,... and if we need to public any file or image to the user or client we can easy make those file public or using cloudfront.

# A way to update all components with zero-downtime
- eks cluster: just create a new cluster with the new version then run the all the deployment it very simple because every config stay the same just the cluster change. when the new cluter running without any error then we change the dns record to new ingre.
-  k8s deployment using rolling update with maxUnavailable = 0 so eachtime we change or update the image it will create new pod and route the traffic to new pod after pass health check. The old pod only delete after the traffic has been sent to new pod so no downtime.
-  ALB ingress: aws managed service no need to update.
-  aurora postgres serverless DB : aws managed service support Blue/Green Deployment easy to update without downtime
-  all other components no need to update or do not affect user or client.

# A way to easily scale the api and ui instances horizontally
- i am using hpa to auto scale the api and ui pod based on CPU and memory usage. if we need to manual increase pod just change the replicas to the number we want and apply the k8s-config.

# Isolating internal from external components
- The ALB ingress create in the public subnet all other components stay in the private subnet. the connect from ALB ingress to the internal pod control by the security group.

# Handling HTTPS termination
- the certificate create and store on the ACM. ingress ALB can using those certificate to handed HTTPS request (simple config on the ingress-config.yaml).

# A schedule for backups and a disaster recovery plan
- in this architect we only have 2 things to backup: image and DB.
    - image store in the ECR we can setup a life cycle and using tag to control the image backup
    - aurora postgres serverless DB: aws managed service can easy setup backup daily and quick restore using daily snapshot.
# An overview of performance metrics that you'd collect from the system
![image](https://github.com/anhnt3494/creative-force-test/assets/145828302/d3badf8b-645d-4072-88c0-03d8ad6a1eb8)

# Indicate potential shortcomings and how you would approach them
- This is a simple setup for small project with hundred or few thousands users and clients. when the project grow bigger we may need more thing to make the system run smoothly like: redis cache for caching session or metadata, read DB to reduce load on primary DB,... but in any case we need to work in a team: cooperating with the DEV team, taking with CS team, asking BE and sale,.. always have solution for any situation with a good teamwork.  
