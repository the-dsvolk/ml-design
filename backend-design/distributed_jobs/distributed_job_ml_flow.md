# Detailed ML Job Flow in Distributed Job Scheduler

```mermaid
flowchart TD
    %% Job Submission Layer
    U[User/ML Engineer] --> API[API Gateway/Load Balancer]
    API --> Auth[Authentication Service<br/>JWT/OAuth2]
    Auth --> JC[Job Controller<br/>Validates & Enriches Job Spec]
    
    %% Queue System
    JC --> MQ[Message Queue<br/>Redis/RabbitMQ/Kafka]
    MQ --> PQ[Priority Queues<br/>High/Medium/Low Priority]
    PQ --> JDB[(Job Metadata DB<br/>PostgreSQL/etcd)]
    
    %% Kubernetes Orchestration
    JS[Job Scheduler Service] --> K8S_API[Kubernetes API Server]
    PQ --> JS
    K8S_API --> K8S_SCHED[K8s Scheduler]
    K8S_SCHED --> RM[Resource Manager<br/>Node Affinity/Anti-affinity]
    
    %% Resource Selection & Hardware
    RM --> GPU_NODES[GPU Nodes<br/>NVIDIA A100/V100<br/>CUDA Cores]
    RM --> CPU_NODES[CPU Nodes<br/>High Memory<br/>Multi-core]
    RM --> TPU_NODES[TPU Nodes<br/>Google TPU v4/v5<br/>ML Optimized]
    
    %% Storage Systems
    K8S_API --> PV[Persistent Volumes<br/>NFS/Ceph/EBS]
    PV --> DIST_FS[Distributed File System<br/>HDFS/GlusterFS]
    DIST_FS --> OBJ_STORE[Object Storage<br/>S3/GCS/Azure Blob]
    
    %% ML-Specific Infrastructure
    JC --> MR[Model Registry<br/>MLflow/Kubeflow]
    JC --> FS[Feature Store<br/>Feast/Tecton]
    JC --> ET[Experiment Tracking<br/>Weights & Biases/Neptune]
    
    %% Pod Execution
    GPU_NODES --> GPOD[GPU Pods<br/>TensorFlow/PyTorch<br/>Containers]
    CPU_NODES --> CPOD[CPU Pods<br/>Data Processing<br/>Inference]
    TPU_NODES --> TPOD[TPU Pods<br/>JAX/TensorFlow<br/>Large Models]
    
    %% Monitoring & Observability
    GPOD --> PROM[Prometheus<br/>Metrics Collection]
    CPOD --> PROM
    TPOD --> PROM
    PROM --> GRAF[Grafana<br/>Dashboards]
    
    GPOD --> ELK[Elasticsearch<br/>Log Aggregation]
    CPOD --> ELK
    TPOD --> ELK
    ELK --> KIB[Kibana<br/>Log Analysis]
    
    %% Service Mesh & Networking
    GPOD --> SM[Service Mesh<br/>Istio/Linkerd]
    CPOD --> SM
    TPOD --> SM
    SM --> LB[Load Balancer<br/>NGINX/HAProxy]
    
    %% Results & Cleanup
    GPOD --> RS[Result Storage<br/>Model Artifacts<br/>Checkpoints]
    CPOD --> RS
    TPOD --> RS
    RS --> NOTIFY[Notification Service<br/>Slack/Email/Webhook]
    
    %% Auto-scaling
    PROM --> HPA[Horizontal Pod Autoscaler]
    HPA --> K8S_API
    PROM --> CA[Cluster Autoscaler<br/>Node Scaling]
    CA --> CLOUD[Cloud Provider<br/>AWS/GCP/Azure]
    
    %% Styling
    classDef submission fill:#e3f2fd,stroke:#1976d2,stroke-width:2px,color:#000
    classDef queue fill:#f3e5f5,stroke:#7b1fa2,stroke-width:2px,color:#000
    classDef k8s fill:#e8f5e8,stroke:#388e3c,stroke-width:2px,color:#000
    classDef hardware fill:#fff3e0,stroke:#f57c00,stroke-width:2px,color:#000
    classDef storage fill:#fce4ec,stroke:#c2185b,stroke-width:2px,color:#000
    classDef ml fill:#e0f2f1,stroke:#00796b,stroke-width:2px,color:#000
    classDef monitoring fill:#f1f8e9,stroke:#689f38,stroke-width:2px,color:#000
    classDef execution fill:#fff8e1,stroke:#ffa000,stroke-width:2px,color:#000
    
    class U,API,Auth,JC submission
    class MQ,PQ,JDB queue
    class JS,K8S_API,K8S_SCHED,RM,HPA,CA k8s
    class GPU_NODES,CPU_NODES,TPU_NODES,CLOUD hardware
    class PV,DIST_FS,OBJ_STORE,RS storage
    class MR,FS,ET ml
    class PROM,GRAF,ELK,KIB,SM,LB,NOTIFY monitoring
    class GPOD,CPOD,TPOD execution
```

## Detailed Component Architecture

### Job Submission & API Layer
- **API Gateway**: Rate limiting, request routing, SSL termination
- **Authentication**: JWT tokens, OAuth2, RBAC policies
- **Job Controller**: Validates job specs, enriches with defaults, cost estimation

### Queue & Message System
- **Message Queue**: Redis for speed, Kafka for durability, RabbitMQ for complex routing
- **Priority Queues**: Separate queues for different priority levels
- **Metadata DB**: PostgreSQL for complex queries, etcd for configuration

### Kubernetes Orchestration
- **K8s Scheduler**: Custom schedulers for ML workloads
- **Resource Manager**: GPU/CPU/memory allocation, node affinity rules
- **Horizontal Pod Autoscaler**: Scale pods based on CPU/memory/custom metrics
- **Cluster Autoscaler**: Add/remove nodes based on demand

### Hardware Infrastructure
- **GPU Nodes**: NVIDIA A100/V100 for training, T4 for inference
- **CPU Nodes**: High-memory instances for data processing
- **TPU Nodes**: Google TPUs for large model training
- **Network**: High-bandwidth interconnects (InfiniBand, 100GbE)

### Storage Systems
- **Persistent Volumes**: NFS for shared storage, EBS for performance
- **Distributed FS**: HDFS for big data, Ceph for object storage
- **Object Storage**: S3/GCS for model artifacts, datasets

### ML-Specific Components
- **Model Registry**: Version control for models, metadata tracking
- **Feature Store**: Centralized feature management, real-time serving
- **Experiment Tracking**: Hyperparameter logging, metric comparison

### Monitoring & Observability
- **Prometheus**: Time-series metrics, alerting rules
- **Grafana**: Visualization dashboards, SLA monitoring
- **ELK Stack**: Centralized logging, error tracking
- **Service Mesh**: Traffic management, security policies 