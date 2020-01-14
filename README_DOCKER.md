### Prerequisite
 - docker
 - docker-compose
 
Install docker
```
	sudo apt install docker.io
```

Install docker-compose
Check the current release and if necessary, update it:
```
	sudo curl -L https://github.com/docker/compose/releases/download/1.21.2/docker-compose-`uname -s`-`uname -m` -o /usr/local/bin/docker-compose
```
Set permission:
```
	sudo chmod +x /usr/local/bin/docker-compose
```
Verify it's working
```
	docker-compose --version
```
### Installation

1. Generate certificates for Elasticsearch by bringing up the create-certs container:
```
	docker-compose -f create-certs.yml run --rm create_certs
```
2. Bring up the three-node Elasticsearch cluster:
```
	docker-compose -f elastic-docker-tls.yml up -d
```
3. Run the elasticsearch-setup-passwords tool to generate passwords for all built-in users, including the kibana user. You can then change your password with `curl` or using `kibana`:
```
	docker exec es01 /bin/bash -c "bin/elasticsearch-setup-passwords \
		auto --batch \
		-Expack.security.http.ssl.certificate=certificates/es01/es01.crt \
		-Expack.security.http.ssl.certificate_authorities=certificates/ca/ca.crt \
		-Expack.security.http.ssl.key=certificates/es01/es01.key \
		--url https://localhost:9200"
```
4. Set back ELASTICSEARCH_PASSWORD in the elastic-docker-tls.yml compose file to the password generated for the `kibana` user:
```
	ELASTICSEARCH_USERNAME: kibana
   	ELASTICSEARCH_PASSWORD: <generated-password>
```
5. Restart the cluster:
```
	docker-compose down
	docker-compose -f elastic-docker-tls.yml up -d
```
6. Open Kibana to load sample data and interact with the cluster: https://localhost:5601

### Extra
Kibana command to change password of user:
```
    POST /_security/user/elastic/_password
    {
      "password" : <your_password>
    }
```

NOTE:
Use https when connect to cluster via kibana. If you use `curl`, add `-k` parameter. For example:
```
	curl -u <user> -XGET -k 'https://localhost:9200/_cluster/health?pretty'
```
 
