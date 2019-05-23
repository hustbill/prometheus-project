# MONITORING DOCKER CONTAINER METRICS USING CADVISOR

## Prometheus configuration
First, you'll need to configure Prometheus to scrape metrics from cAdvisor. Create a prometheus.yml file and populate it with this configuration:
```YAML
scrape_configs:
- job_name: cadvisor
  scrape_interval: 5s
  static_configs:
  - targets:
    - cadvisor:8080
```  

## Docker Compose configuration
Now we'll need to create a Docker Compose configuration that specifies which containers are part of our installation as well as which ports are exposed by each container, which volumes are used, and so on.

In the same folder where you created the prometheus.yml file, create a docker-compose.yml file and populate it with this Docker Compose configuration:
```YAML
version: '3.2'
services:
  prometheus:
    image: prom/prometheus:latest
    container_name: prometheus
    ports:
    - 9090:9090
    command:
    - --config.file=/etc/prometheus/prometheus.yml
    volumes:
    - ./prometheus.yml:/etc/prometheus/prometheus.yml:ro
    depends_on:
    - cadvisor
  cadvisor:
    image: google/cadvisor:latest
    container_name: cadvisor
    ports:
    - 8080:8080
    volumes:
    - /:/rootfs:ro
    - /var/run:/var/run:rw
    - /sys:/sys:ro
    - /var/lib/docker/:/var/lib/docker:ro
    depends_on:
    - redis
  redis:
    image: redis:latest
    container_name: redis
    ports:
    - 6379:6379
```  

```YAML  
This configuration instructs Docker Compose to run three services, each of which corresponds to a Docker container:  

- The prometheus service uses the local prometheus.yml configuration file (imported into the container by the volumes parameter).  
- The cadvisor service exposes port 8080 (the default port for cAdvisor metrics) and relies on a variety of local volumes (/, /var/run, etc.).  
- The redis service is a standard Redis server. cAdvisor will gather container metrics from this container automatically, i.e. without any further   configuration.  

To run the installation:  
```bash
docker-compose up
```

If Docker Compose successfully starts up all three containers, you should see output like this:  
```bash  
prometheus  | level=info ts=2018-07-12T22:02:40.5195272Z caller=main.go:500 msg="Server is ready to receive web requests."
```
You can verify that all three containers are running using the ps command:
```bash
docker-compose ps
```  
## Exploring the cAdvisor web UI  
You can access the cAdvisor web UI at http://localhost:8080. You can explore stats and graphs for specific Docker containers in our installation at http://localhost:8080/docker/<container>. Metrics for the Redis container, for example, can be accessed at http://localhost:8080/docker/redis, Prometheus at http://localhost:8080/docker/prometheus, and so on.

## Exploring metrics in the expression browser  
cAdvisor's web UI is a useful interface for exploring the kinds of things that cAdvisor monitors, but it doesn't provide an interface for exploring container metrics. For that we'll need the Prometheus expression browser, which is available at http://localhost:9090/graph. You can enter Prometheus expressions into the expression bar, which looks like this: [Prometheus expression browser](https://prometheus.io/assets/prometheus-expression-bar.png)  
