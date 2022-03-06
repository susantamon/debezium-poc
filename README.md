# Debezium POC with Oracle 19 & Monitoring with Grafana

## Detailed Documentation
https://medium.com/@susantamon/data-replication-cdc-from-oracle-to-postgresql-using-debezium-run-in-docker-exposed-in-grafana-fb5f8eeaf2c9

## Steps to perform

### Get Oracle Image
    Visit oracle container repo - https://container-registry.oracle.com/ords/f?p=113:1:117569610822412:::1:P1_BUSINESS_AREA:3
    Sign in -> Accept T&C for enterprise db
    docker login container-registry.oracle.com
    docker pull container-registry.oracle.com/database/enterprise:19.3.0.0

### Oracle Client
    Download - https://download.oracle.com/otn_software/linux/instantclient/1914000/instantclient-basic-linux.x64-19.14.0.0.0dbru.zip
    unzip to debezium-with-oracle-jdbc/oracle_instantclient

### Docker Desktop Settings
    Increase docker desktop runtime memory to 6g

### Run the Docker Images
    DEBEZIUM_VERSION=1.8 docker-compose -f docker-compose-oracle.yaml up --build

### One Time Setup (if Oracle Container Mounts on Host)
    docker exec -it debezium_oracledb19_1 /bin/bash
    mkdir /opt/oracle/oradata/recovery_area
    curl https://raw.githubusercontent.com/debezium/oracle-vagrant-box/main/setup-logminer.sh | sh
    cat inventory.sql | sqlplus debezium/dbz@//localhost:1521/ORCLPDB1

### Register the Connectors
    curl -i -X POST -H "Accept:application/json" -H  "Content-Type:application/json" http://localhost:8083/connectors/ -d @register-oracle-logminer.json
    curl -i -X POST -H "Accept:application/json" -H  "Content-Type:application/json" http://localhost:8083/connectors/ -d @jdbc-sink.json

### Data Manipulation & Check
    echo "INSERT INTO customers VALUES (NULL, 'John', 'Doe', 'john.doe@example.com');" | docker exec -i debezium_oracledb19_1 sqlplus debezium/dbz@//localhost:1521/ORCLPDB1

    DEBEZIUM_VERSION=1.8 docker-compose -f docker-compose-oracle.yaml exec kafka /kafka/bin/kafka-console-consumer.sh \
        --bootstrap-server kafka:9092 \
        --from-beginning \
        --property print.key=false \
        --topic CUSTOMERS

    DEBEZIUM_VERSION=1.8 docker-compose -f docker-compose-oracle.yaml exec kafka /kafka/bin/kafka-topics.sh --list --bootstrap-server kafka:9092


### Shutdown Docker Images
    DEBEZIUM_VERSION=1.8 docker-compose -f docker-compose-oracle.yaml down
