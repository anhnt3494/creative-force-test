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
