# spend-control

CREATE SECRETS & CONFIGMAPS
DEPLOY NAMESPACE + DB MIGRATORS + SERVICES
INGRESS + ALB + DNS
ROUTE 53 DNS RECORDS
IAM TOKEN AUTH FOR POSTGRES (CLIENT SIDE)
  Your application should:
    Fetch IAM token using AWS SDK (e.g., Java)
    Connect to Postgres using token in JDBC URL: jdbc:postgresql://your-pg-endpoint:5432/vendor_db?ssl=true&sslmode=verify-full&sslrootcert=cert.pem
          String token = RdsIamAuthTokenGenerator.builder()
              .credentials(new DefaultCredentialsProvider())
              .region(Region.US_EAST_1)
              .build()
              .getAuthToken(GetIamAuthTokenRequest.builder()
                  .hostname("your-pg-endpoint")
                  .port(5432)
                  .userName("eksuser")
                  .build());
      
          DataSource ds = new PGSimpleDataSource();
          ds.setUrl("jdbc:postgresql://your-pg-endpoint:5432/vendor_db");
          ds.setUser("eksuser");
          ds.setPassword(token);
