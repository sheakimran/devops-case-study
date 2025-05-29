
<a id="readme-top"></a>



<!-- PROJECT LOGO -->
<br />
<div align="center">
  <a href="https://github.com/othneildrew/Best-README-Template">
    <img src="images/logo.png" alt="Logo" width="80" height="80">
  </a>

  <h3 align="center">DevOps Case Study</h3>

  <p align="center">
    Here i showcase my full assignment depoly task with code and screenshot from my terminal where needed.
    
  </p>
</div>




<!-- ABOUT THE PROJECT -->
## Section 1: Configuration Management (Ansible)

### Question 1: Display Ansible Configuration for a Host
Answer: The command to display all ansible_ configuration for a host is:

```SH
bash
ansible-config dump --only-changed
```
# or for a specific host:
bash
```sh
 ansible <hostname> -m setup
 ```

 here all the screenshoot for ansible task:

[![Product Name ansible-sc][ansible-sc]](https://example.com)

[![Product Name ansible-sc-2][ansible-sc-2]](https://example.com)

[![Product Name ansible-sc-3][ansible-sc-3]](https://example.com)

[![Product Name ansible-sc-4][ansible-sc-4]](https://example.com)

## Question 2: Configure Cron Job for Logrotate

Here is the yaml file for logrotate

```yaml
 ---
- name: Configure logrotate cron job
  hosts: all
  become: yes
  tasks:
    - name: Install logrotate if not present
      apt:
        name: logrotate
        state: present
        update_cache: yes

    - name: Create cron job for logrotate (every 10 minutes between 2h-4h)
      cron:
        name: "Logrotate job"
        minute: "*/10"
        hour: "2-4"
        job: "/usr/sbin/logrotate /etc/logrotate.conf"
        user: root
 ```

[![Product Name logrotate][logrotate]](https://example.com)

## Question 3: Deploy ntpd Package with Custom Config

### Create ntpd configuration template

```yaml
# NTP configuration file
driftfile /var/lib/ntp/ntp.drift

# Pool servers
pool 0.ubuntu.pool.ntp.org iburst
pool 1.ubuntu.pool.ntp.org iburst
pool 2.ubuntu.pool.ntp.org iburst
pool 3.ubuntu.pool.ntp.org iburst

# Use Ubuntu's ntp server as a fallback
server ntp.ubuntu.com

# Restrict access
restrict -4 default kod notrap nomodify nopeer noquery limited
restrict -6 default kod notrap nomodify nopeer noquery limited
restrict 127.0.0.1
restrict ::1

# Enable this if you want statistics to be logged
statsdir /var/log/ntpstats/
statistics loopstats peerstats clockstats
filegen loopstats file loopstats type day enable
filegen peerstats file peerstats type day enable
filegen clockstats file clockstats type day enable
```

### Create ntpd role tasks
```yaml
---
- name: Install ntp package
  apt:
    name: ntp
    state: present
    update_cache: yes

- name: Configure ntp
  template:
    src: ntp.conf.j2
    dest: /etc/ntp.conf
    backup: yes
  notify: restart ntp

- name: Start and enable ntp service
  systemd:
    name: ntp
    state: started
    enabled: yes
```

### Create handlers
```yaml
---
- name: restart ntp
  systemd:
    name: ntp
    state: restarted

```

### Create playbook for ntpd deployment

```yaml
---
- name: Deploy ntpd to specified servers
  hosts: app_servers,db_servers,web_servers
  become: yes
  roles:
    - ntpd


```

[![Product Name ntpd][ntpd]](https://example.com)

## Nagios Monitoring Setup

### Create Nagios monitoring playbook

```yaml

for yaml file check repo please

```
[![Product Name nagios][nagios]](https://example.com)


## Section 2: Docker/Kubernetes
### Question 1: Docker Compose for Nginx with Persistent Logs

```yaml
version: '3.8'

services:
  nginx:
    image: nginx:latest
    container_name: nginx-server
    ports:
      - "80:80"
      - "443:443"
    volumes:
      - nginx_logs:/var/log/nginx
      - ./nginx.conf:/etc/nginx/nginx.conf:ro
    networks:
      - custom_bridge
    restart: unless-stopped

volumes:
  nginx_logs:
    driver: local

networks:
  custom_bridge:
    driver: bridge
    ipam:
      config:
        - subnet: 172.20.8.0/24


```
### Create basic nginx config

```yaml
user nginx;
worker_processes auto;

error_log /var/log/nginx/error.log notice;
pid /var/run/nginx.pid;

events {
    worker_connections 1024;
}

http {
    include /etc/nginx/mime.types;
    default_type application/octet-stream;

    log_format main '\$remote_addr - \$remote_user [\$time_local] "\$request" '
                    '\$status \$body_bytes_sent "\$http_referer" '
                    '"\$http_user_agent" "\$http_x_forwarded_for"';

    access_log /var/log/nginx/access.log main;

    sendfile on;
    keepalive_timeout 65;

    server {
        listen 80;
        server_name localhost;

        location / {
            root /usr/share/nginx/html;
            index index.html index.htm;
        }
    }
}


```

[![Product Name nginx][nginx]](https://example.com)



## Question 2: Kubernetes Pod Restart Investigation

### Answer: To identify the reason for a pod restart in the project "internal" under namespace "production":

```sh
kubectl describe pod <pod-name> -n production
kubectl logs <pod-name> -n production --previous
kubectl get events -n production --field-selector involvedObject.name=<pod-name>

```

## Question 3: Java App Pod Restart Analysis
### Answer: Based on the resource usage data provided:
Possible reasons for pod restarts:

Memory Limit Exceeded: Total memory usage (951Mi + 45Mi + 84Mi + 62Mi = 1142Mi) exceeds the memory limit of 1000Mi
JVM Heap Issues: Xmx is set to 1000M but memory limit is also 1000Mi, leaving no room for non-heap memory
CPU Throttling: Total CPU usage (3m + 1m + 1m + 4m = 9m) is within limits, so not the primary issue
OOMKilled: The pod is likely being killed due to memory pressure

Solutions:

Increase memory limit to at least 1500Mi
Reduce Xmx to 700-800M to allow for non-heap memory
Add proper resource requests and limits for all containers


## Section 3: Helm
### Elasticsearch Deployment with Helm

I have successfully used the accompanied Elasticsearch Helm template to create a Kubernetes deployment of Elasticsearch in my local environment. The deployment process and results are detailed below:

### Deployment Implementation
The Helm template was used to deploy Elasticsearch with the following components:

1) StatefulSet: customer-abc-elasticsearch configured with 3 replicas

2) Pods: Three Elasticsearch instances (customer-abc-elasticsearch-0, -1, -2)

3) Services: ClusterIP services for internal cluster communication

4) Persistent Volume Claims: Proper storage provisioning for data persistence


### Deployment Process and Observations

The initial deployment was successful, with all pods starting correctly and pulling the required Elasticsearch image (elasticsearch:7.17.3). The Kubernetes resources were properly created and configured according to the Helm template specifications.

### Runtime Issues and Hardware Limitations

After running for a considerable amount of time, the deployment began experiencing failures. The pods entered various problematic states including:

1) CrashLoopBackoff conditions
2) Pending states due to resource constraints
3) Frequent restarts indicating instability

### Root Cause Analysis

The failures appear to be primarily due to hardware limitations in my local development environment. Elasticsearch is a resource-intensive application that requires:

1) Significant memory allocation (minimum 2GB per node)
2) Adequate CPU resources for optimal performance
3) Sufficient storage I/O capabilities
4) Proper JVM heap configuration


### Conclusion

While the Helm template deployment was executed correctly and initially functional, the local environment's hardware constraints prevented sustained operation. The attached screenshots demonstrate both the successful initial deployment and the subsequent resource-related failures. For production deployment, adequate hardware resources would be essential for stable Elasticsearch operation.

#### Evidence:

Please refer to the attached screenshots showing the kubectl outputs demonstrating the deployment progression from successful initialization to resource-constrained failure states.

and the ```deployment.yaml``` file in github repo

[![Product Name helm-1][helm-1]](https://example.com)

[![Product Name helm-2][helm-2]](https://example.com)

[![Product Name helm-3][helm-3]](https://example.com)

[![Product Name helm-4][helm-4]](https://example.com)

[![Product Name helm-5][helm-5]](https://example.com)

[![Product Name helm-6][helm-5]](https://example.com)





## Section 4: Metrics (Prometheus & Grafana)
### Question 1: How Prometheus Works
Answer:
Prometheus is a monitoring and alerting toolkit that works by:

Pull-based Architecture: Scrapes metrics from configured targets at regular intervals

Time Series Database: Stores all data as time series with metric name and key-value pairs

Service Discovery: Automatically discovers targets through various mechanisms

PromQL: Uses its own query language for data retrieval and analysis

Alerting: Evaluates rules and sends alerts to Alertmanager

### Question 2: Custom Prometheus Alerts

#### Create Prometheus configuration with custom alerts

```yaml
global:
  scrape_interval: 15s
  evaluation_interval: 15s

rule_files:
  - "alert_rules.yml"

alerting:
  alertmanagers:
    - static_configs:
        - targets:
          - alertmanager:9093

scrape_configs:
  - job_name: 'prometheus'
    static_configs:
      - targets: ['localhost:9090']

  - job_name: 'kubernetes-pods'
    kubernetes_sd_configs:
    - role: pod
    relabel_configs:
    - source_labels: [__meta_kubernetes_pod_annotation_prometheus_io_scrape]
      action: keep
      regex: true

````
### Create alert rules

```yaml
groups:
- name: kubernetes_alerts
  rules:
  - alert: PodCrashLooping
    expr: rate(kube_pod_container_status_restarts_total[5m]) > 0
    for: 2m
    labels:
      severity: warning
    annotations:
      summary: "Pod {{ $labels.pod }} is crash looping"
      description: "Pod {{ $labels.pod }} in namespace {{ $labels.namespace }} has restarted {{ $value }} times in the last 5 minutes"

  - alert: HighMemoryUsage
    expr: (container_memory_usage_bytes / container_spec_memory_limit_bytes) > 0.8
    for: 5m
    labels:
      severity: warning
    annotations:
      summary: "High memory usage on {{ $labels.pod }}"
      description: "Memory usage is above 80% for pod {{ $labels.pod }}"

  - alert: NodeNotReady
    expr: kube_node_status_condition{condition="Ready",status="true"} == 0
    for: 1m
    labels:
      severity: critical
    annotations:
      summary: "Node {{ $labels.node }} is not ready"
      description: "Node {{ $labels.node }} has been not ready for more than 1 minute"


```
### Deploy Prometheus with Docker


```yaml
version: '3.8'

services:
  prometheus:
    image: prom/prometheus:latest
    ports:
      - "9090:9090"
    volumes:
      - ./prometheus.yml:/etc/prometheus/prometheus.yml
      - ./alert_rules.yml:/etc/prometheus/alert_rules.yml
    command:
      - '--config.file=/etc/prometheus/prometheus.yml'
      - '--storage.tsdb.path=/prometheus'
      - '--web.console.libraries=/etc/prometheus/console_libraries'
      - '--web.console.templates=/etc/prometheus/consoles'
      - '--web.enable-lifecycle'

  grafana:
    image: grafana/grafana:latest
    ports:
      - "3000:3000"
    environment:
      - GF_SECURITY_ADMIN_PASSWORD=admin123
    volumes:
      - grafana_data:/var/lib/grafana

volumes:
  grafana_data:

```
### Start monitoring stack
```sh
docker-compose -f docker-compose-prometheus.yml up -d
```

## Question 3: Prometheus Query for Counter Metrics in Grafana
### Answer: For showing usage trend of a counter metric in Grafana:
promql# 
```sh
Use rate() function for counter metrics
rate(your_counter_metric_name[5m])

# For better visualization, you might also use:
increase(your_counter_metric_name[1h])

# Example for HTTP requests: 
rate(http_requests_total[5m])
```
[![Product Name prom-1][prom-1]](https://example.com)
[![Product Name prom-2][prom-2]](https://example.com)
[![Product Name prom-3][prom-3]](https://example.com)


## Section 5: Databases
### Cassandra Issue Analysis
Answer:
Problem: Query inconsistency and seeing deleted data
Likely Reason:

Consistency Level: Queries are using eventual consistency (CL=ONE)

Tombstone Issues: Deleted data appears due to uncompacted tombstones

Node Synchronization: Nodes are not properly synchronized




```yaml
# Use stronger consistency levels
# Read with QUORUM consistency
SELECT * FROM table_name USING CONSISTENCY QUORUM;

# Configure proper gc_grace_seconds (default 864000 = 10 days)
ALTER TABLE table_name WITH gc_grace_seconds = 864000;

# Force compaction to remove tombstones
nodetool compact keyspace_name table_name
```

[![Product Name casa-1][casa-1]](https://example.com)

[![Product Name casa-2][casa-2]](https://example.com)

[![Product Name casa-3][casa-3]](https://example.com)





## MongoDB Sharding Setup


Thank you for the opportunity to attempt the MongoDB sharding task. 

I reviewed the requirement to shard the sanfrancisco.company_name collection based on _id. I made a genuine effort to solve it but was unable to complete the full implementation within the given timeframe. I encountered difficulties setting up the proper environment and configuring the sharded cluster, particularly with MongoDB 3.6, which has some limitations compared to newer versions. 

However, I did go through the theoretical steps required and would like to share my understanding of the expected approach: 

1) Add Sharding Support: Enable sharding on the sanfrancisco database. 

2) Enable Sharding for the Collection: Use _id as the shard key for the company_name collection. 

3) Deploy Config Servers: Set up 3 config servers to manage metadata. 

4) Add Shards: Add both replicaset_1 and replicaset_2 as shard members. 

5) Start Mongos Router: Use mongos as the routing service to connect to the sharded cluster. 

6) Move Chunks (Optional): Allow MongoDB to balance the chunks automatically across shards. 

Though I could not implement the full solution, I would be happy to reattempt this with more time or clarify any part of the approach in an interview discussion. 

Thank you again for your consideration, and I appreciate the learning opportunity this exercise provided. 

<p align="right">(<a href="#readme-top">back to top</a>)</p>













<!-- LICENSE -->
## License

Distributed under the Unlicense License. See `LICENSE.txt` for more information.

<p align="right">(<a href="#readme-top">back to top</a>)</p>



<!-- CONTACT -->
## Contact

Md Sheak Imran - (https://twitter.com/your_username) - sheak.imran2010@gmail.com

Protfolio Link : [https://sheakimran.github.io/mine-website/]((|https://github.com/sheakimran/devops-case-study))

Project Link: [https://github.com/your_username/repo_name](|https://github.com/sheakimran/devops-case-study)

<p align="right">(<a href="#readme-top">back to top</a>)</p>









<!-- MARKDOWN LINKS & IMAGES -->
<!-- https://www.markdownguide.org/basic-syntax/#reference-style-links -->
[contributors-shield]: https://img.shields.io/github/contributors/othneildrew/Best-README-Template.svg?style=for-the-badge
[contributors-url]: https://github.com/othneildrew/Best-README-Template/graphs/contributors
[forks-shield]: https://img.shields.io/github/forks/othneildrew/Best-README-Template.svg?style=for-the-badge
[forks-url]: https://github.com/othneildrew/Best-README-Template/network/members
[stars-shield]: https://img.shields.io/github/stars/othneildrew/Best-README-Template.svg?style=for-the-badge
[stars-url]: https://github.com/othneildrew/Best-README-Template/stargazers
[issues-shield]: https://img.shields.io/github/issues/othneildrew/Best-README-Template.svg?style=for-the-badge
[issues-url]: https://github.com/othneildrew/Best-README-Template/issues
[license-shield]: https://img.shields.io/github/license/othneildrew/Best-README-Template.svg?style=for-the-badge
[license-url]: https://github.com/othneildrew/Best-README-Template/blob/master/LICENSE.txt
[linkedin-shield]: https://img.shields.io/badge/-LinkedIn-black.svg?style=for-the-badge&logo=linkedin&colorB=555
[linkedin-url]: https://linkedin.com/in/othneildrew
[product-screenshot]: images/screenshot.png
[ansible-sc]: images/question-1/1.jpg
[ansible-sc-2]: images/question-1/2.jpg
[ansible-sc-3]: images/question-1/3.jpg
[ansible-sc-4]: images/question-1/4.jpg
[logrotate]: images/question-2/1.jpg
[ntpd]: images/question-3/1.jpg
[nagios]: images/question-3/2.jpg
[nginx]: images/question-4/1.jpg
[prom-1]: images/question-5/1.jpg
[prom-2]: images/question-5/2.jpg
[prom-3]: images/question-5/3.jpg
[casa-1]: images/question-6/1.jpg
[casa-2]: images/question-6/2.jpg
[casa-3]: images/question-6/3.jpg

[helm-1]: images/question-7/1.jpg
[helm-2]: images/question-7/2.jpg
[helm-3]: images/question-7/3.jpg
[helm-4]: images/question-7/4.jpg
[helm-5]: images/question-7/5.jpg
[helm-6]: images/question-7/6.jpg

[Next.js]: https://img.shields.io/badge/next.js-000000?style=for-the-badge&logo=nextdotjs&logoColor=white
[Next-url]: https://nextjs.org/
[React.js]: https://img.shields.io/badge/React-20232A?style=for-the-badge&logo=react&logoColor=61DAFB
[React-url]: https://reactjs.org/
[Vue.js]: https://img.shields.io/badge/Vue.js-35495E?style=for-the-badge&logo=vuedotjs&logoColor=4FC08D
[Vue-url]: https://vuejs.org/
[Angular.io]: https://img.shields.io/badge/Angular-DD0031?style=for-the-badge&logo=angular&logoColor=white
[Angular-url]: https://angular.io/
[Svelte.dev]: https://img.shields.io/badge/Svelte-4A4A55?style=for-the-badge&logo=svelte&logoColor=FF3E00
[Svelte-url]: https://svelte.dev/
[Laravel.com]: https://img.shields.io/badge/Laravel-FF2D20?style=for-the-badge&logo=laravel&logoColor=white
[Laravel-url]: https://laravel.com
[Bootstrap.com]: https://img.shields.io/badge/Bootstrap-563D7C?style=for-the-badge&logo=bootstrap&logoColor=white
[Bootstrap-url]: https://getbootstrap.com
[JQuery.com]: https://img.shields.io/badge/jQuery-0769AD?style=for-the-badge&logo=jquery&logoColor=white
[JQuery-url]: https://jquery.com 
