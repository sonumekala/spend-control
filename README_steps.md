Create EKS Cluster
Push Images to Amazon ECR
   -- Tag Image and push
     docker tag management-service:latest <aws_account_id>.dkr.ecr.us-east-1.amazonaws.com/management-service:latest
     docker push <aws_account_id>.dkr.ecr.us-east-1.amazonaws.com/management-service:latest

Create namespace
  apiVersion: v1
  kind: Namespace
  metadata:
    name: vendor-app

Create Secrets
  apiVersion: v1
  kind: Secret
  metadata:
    name: db-credentials
    namespace: vendor-app
  type: Opaque
  stringData:
    jdbc_url: jdbc:postgresql://yourdbhost:5432/dbname?sslmode=require
    username: your-db-user
    password: your-db-password
    ssl_key: |-
      -----BEGIN PRIVATE KEY-----
      ...your_key...
      -----END PRIVATE KEY-----
    ssl_cert: |-
      -----BEGIN CERTIFICATE-----
      ...your_cert...
      -----END CERTIFICATE-----
    ssl_root_cert: |-
      -----BEGIN CERTIFICATE-----
      ...your_root_cert...
      -----END CERTIFICATE-----

ConfigMap for shared config
    apiVersion: v1
    kind: ConfigMap
    metadata:
      name: vendor-config
      namespace: vendor-app
    data:
      PGHOST: your-pg-instance.rds.amazonaws.com
      PGPORT: "5432"
      PGDATABASE: vendor_db


Create management-service.yaml

Create definition-service.yaml

Create business-event-processor.yaml

Crearte ingress.yaml

Deploy All YAMLs

  kubectl apply -f 01-namespace.yaml
  kubectl apply -f 02-secrets.yaml
  kubectl apply -f 03-management-service.yaml
  kubectl apply -f 04-definition-service.yaml
  kubectl apply -f 05-business-event-processor.yaml
  kubectl apply -f 06-ingress.yaml

Test and Verify
  # Check pods are running
  kubectl get pods -n vendor-app
  
  # Check services
  kubectl get svc -n vendor-app
  
  # Describe Ingress and note ALB hostname
  kubectl get ingress -n vendor-app


---------------------------------------

Once the db is setup:

Annotate Kubernetes ServiceAccount
Create a Kubernetes ServiceAccount:
    apiVersion: v1
    kind: ServiceAccount
    metadata:
      name: app-sa
      namespace: vendor-app
      annotations:
        eks.amazonaws.com/role-arn: arn:aws:iam::<account_id>:role/eks-postgres-access-role

Update your Deployment YAMLs to include:
  spec:
    serviceAccountName: app-sa

  Amazon RDS IAM authentication, we should use `aws rds generate-db-auth-token` or `Java’s IAM JDBC URL`

  Example Use Case with IRSA + RDS IAM Auth --If using RDS IAM auth from Java app in EKS:

    String url = "jdbc:postgresql://your-db-instance.region.rds.amazonaws.com:5432/dbname?ssl=true&sslmode=verify-full";
    String token = RdsIamTokenGenerator.generate(region, dbUser, dbEndpoint); // SDK-based

    conn = DriverManager.getConnection(url, dbUser, token);





