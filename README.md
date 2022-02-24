# Hello from Opta

This is an example repo which used across our examples.

The directory contains the following:

    .
    ├── app.py            # HTTP server implementation. It responds to all HTTP requests with a  "Hello, World!" response.
    ├── Dockerfile        # build the Docker image for the application
    └── README.md         # Instructions on how to use Opta to successfully Deploy docker image to Local as well as Provider Cluster

## Build and run Locally:

1. Build the image
    ```shell
    docker build . -t hello-opta:v1
    ```
2. Run the Hello opta app
    ```shell
    docker run -p 80:80 hello-app:v1
    # or use a different port
    docker run -p 8080:8080 -e PORT=8080 hello-app:v1
    ```
3. Test
    ```shell
    curl http://localhost:80/hello
    ```

## Deploy to Local Kubernetes using opta

1. Create the service configuration.
    ```yaml
    # hello.yaml
    name: hello
    org_name: my-org
    modules:
      - type: k8s-service
        name: hello
        port:
          http: 80
        image: AUTO
        healthcheck_path: "/"
        public_uri: "/hello"
    ```

2. Create a Local Cluster and deploy the Docker image
   ```shell
   opta deploy --local --auto-approve -c hello.yaml --image hello-opta:main
   ```

3. Test
   ```shell
   curl http://localhost:8080/hello
   ```

4. Destroy the Local Cluster
   ```shell
   opta destroy --local --auto-approve -c hello.yaml
   ```

## Deploy to a Cloud Provider

1. Create the configuration based on the Provider.

   a. AWS
      ```yaml
      # infra.yaml
      name: aws # name of the environment
      org_name: my-org # A unique identifier for your organization
      providers:
        aws:
          region: us-east-1
          account_id: 000000000000 # Your 12 digit AWS account id
      modules:
        - type: base
        - type: k8s-cluster
        - type: k8s-base
      ```

   b. GCP
      ```yaml
      # infra.yaml
      name: gcp # name of the environment
      org_name: my-org # A unique identifier for your organization
      providers:
        google:
          region: us-central1
          project: XXXXX # the name of your GCP project
      modules:
        - type: base
        - type: k8s-cluster
        - type: k8s-base
      ```

   c. Azure
      ```yaml
      # infra.yaml
      name: azure # name of the environment
      org_name: my-org # A unique identifier for your organization
      providers:
        azurerm:
          location: centralus
          tenant_id: XXX  # your Azure tenant id
          subscription_id: YYY # your Azure subscription id
      modules:
        - type: base
        - type: k8s-cluster
          admin_group_object_ids: [""]
        - type: k8s-base
      ```

2. Update the service configuration
   ```yaml
    # hello.yaml
    environments:
      - name: env-tg
        path: "infra.yaml"
    name: hello
    modules:
      - type: k8s-service
        name: hello
        port:
          http: 80
        image: AUTO
        healthcheck_path: "/"
        public_uri: "/hello"
    ```

3. Use Opta to Create the Infrastructure (VPC, Kubernetes, etc.)
   ```shell
   opta apply --auto-approve -c infra.yaml
   # when done, find load_balancer_raw_dns or load_balancer_raw_ip in the output and save it
   export load_balancer=[Value from output]
   ```

4. Deploy the Service: Push the image and Deploy it to Kubernetes
   ```shell
   opta deploy --auto-approve -c hello.yaml --image hello-opta:main
   ```

5. Test
   ```shell
   curl http://${load_balancer}/hello
   # you can run any kubectl command at this point
   kubectl -n hello get all
   ```

6. Cleanup
   ```shell
   opta destroy --auto-approve -c hello.yaml
   opta destroy --auto-approve -c infra.yaml
   ```