# Development

## Running Integration Tests

### Prerequisites

- Docker
- Go 1.23+

### Start HBase

Create the HBase configuration file:

```bash
cat > /tmp/hbase-site.xml << 'EOF'
<?xml version="1.0"?>
<?xml-stylesheet type="text/xsl" href="configuration.xsl"?>
<configuration>
  <property>
    <name>hbase.cluster.distributed</name>
    <value>false</value>
  </property>
  <property>
    <name>hbase.tmp.dir</name>
    <value>./tmp</value>
  </property>
  <property>
    <name>hbase.unsafe.stream.capability.enforce</name>
    <value>false</value>
  </property>
  <property>
    <name>hbase.master.ipc.address</name>
    <value>0.0.0.0</value>
  </property>
  <property>
    <name>hbase.regionserver.ipc.address</name>
    <value>0.0.0.0</value>
  </property>
  <property>
    <name>hbase.master.hostname</name>
    <value>localhost</value>
  </property>
  <property>
    <name>hbase.regionserver.hostname</name>
    <value>localhost</value>
  </property>
  <property>
    <name>hbase.zookeeper.property.clientPortAddress</name>
    <value>0.0.0.0</value>
  </property>
</configuration>
EOF
```

Start the HBase container:

```bash
docker run -d --name hbase \
  -p 2181:2181 \
  -p 16000:16000 \
  -p 16010:16010 \
  -p 16020:16020 \
  -p 16030:16030 \
  -v /tmp/hbase-site.xml:/hbase/conf/hbase-site.xml \
  aristanetworks/hbase:2.5.8 /hbase/bin/hbase master start
```

Wait for HBase to initialize (~45 seconds):

```bash
sleep 45
docker logs hbase 2>&1 | grep "Master has completed initialization"
```

### Run Integration Tests

```bash
go test -v -race -timeout=120s -tags=integration ./...
```

### Stop HBase

```bash
docker rm -f hbase
```

### Troubleshooting

Verify ports are listening:

```bash
docker exec hbase netstat -tlnp | grep -E "2181|16000|16020"
```

All three ports should show `:::` (listening on all interfaces).

