
# docker compose 配置

docker-compose.opensearch.yml

```yaml

version: '3.8'  
name: open-search  
  
services:  
  opensearch:  
    image: opensearchproject/opensearch:2.12.0  
    container_name: opensearch  
    environment:  
      - discovery.type=single-node  
      - OPENSEARCH_JAVA_OPTS=-Xms512m -Xmx512m  
      - DISABLE_SECURITY_PLUGIN=true  
  
    ports:  
      - "9200:9200"  
      - "9600:9600"  
    volumes:  
      - opensearch-data:/usr/share/opensearch/data  
    networks:  
      - opensearch-network  
  
  opensearch-dashboards:  
    image: opensearchproject/opensearch-dashboards:2.12.0  
    container_name: opensearch-dashboards  
    ports:  
      - "5601:5601"  
    environment:  
      - OPENSEARCH_HOSTS=http://opensearch:9200  
      - DISABLE_SECURITY_DASHBOARDS_PLUGIN=true  
      - OPENSEARCH_SECURITY_ENABLED=false  
    depends_on:  
      - opensearch  
    networks:  
      - opensearch-network  
  
volumes:  
  opensearch-data:  
    driver: local  
  
networks:  
  opensearch-network:  
    driver: bridge

```

# 启动

```bash
docker compose docker-compose.opensearch.yml up -d 
```

# 验证

访问 OpenSearch http://localhost:9200/

```json
{
  "name": "db823aba8cc7",
  "cluster_name": "docker-cluster",
  "cluster_uuid": "MkKLDugOQpSxdX153oE9kA",
  "version": {
    "distribution": "opensearch",
    "number": "2.12.0",
    "build_type": "tar",
    "build_hash": "2c355ce1a427e4a528778d4054436b5c4b756221",
    "build_date": "2024-02-20T02:18:49.874618333Z",
    "build_snapshot": false,
    "lucene_version": "9.9.2",
    "minimum_wire_compatibility_version": "7.10.0",
    "minimum_index_compatibility_version": "7.0.0"
  },
  "tagline": "The OpenSearch Project: https://opensearch.org/"
}
```

访问 OpenSearch Dashboard http://localhost:5601/

