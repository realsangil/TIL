# 도커를 이용한 elasticsearch 클러스터 구성 예제

> 이 설정 파일은 엘라스틱서치의 버전이 7이상인 이미지에서만 사용 가능

cloud 환경에서 하나의 인스턴스 내에서 클러스터 구성을 할 땐 상관 없지만, 복수의 인스턴스와 클러스터를 구성할 때는 `host` 네트워크를 사용해야 한다.
`host` 네트워크를 사용하지 않고 `bridge` 모드나 다른 네트워크 모드를 사용할 때는 내부 아이피로 연결하기 때문에 아래와 같은 에러가 발생한다.

```bash
{
  "type": "server",
  "timestamp": "2020-07-16T04:51:29,960Z",
  "level": "WARN",
  "component": "o.e.d.HandshakingTransportAddressConnector",
  "cluster.name": "es-docker-cluster",
  "node.name": "es03",
  "message": "[connectToRemoteMasterNode[10.0.10.141:9300]] completed handshake with [{es01}{J_T5Zc6VRGOEr0XvuruOaA}{9-4VRtxFRWCurKpQbIpvbA}{172.21.0.3}{172.21.0.3:9300}{dilmrt}{ml.machine_memory=4134494208, ml.max_open_jobs=20, xpack.installed=true, transform.node=true}] but followup connection failed",
  "stacktrace": [
    "org.elasticsearch.transport.ConnectTransportException: [es01][172.21.0.3:9300] connect_exception",
    "at org.elasticsearch.transport.TcpTransport$ChannelsConnectedListener.onFailure(TcpTransport.java:966) ~[elasticsearch-7.8.0.jar:7.8.0]",
    "at org.elasticsearch.action.ActionListener.lambda$toBiConsumer$2(ActionListener.java:198) ~[elasticsearch-7.8.0.jar:7.8.0]",
    "at org.elasticsearch.common.concurrent.CompletableContext.lambda$addListener$0(CompletableContext.java:42) ~[elasticsearch-core-7.8.0.jar:7.8.0]",
    "at java.util.concurrent.CompletableFuture.uniWhenComplete(CompletableFuture.java:859) ~[?:?]",
    "at java.util.concurrent.CompletableFuture$UniWhenComplete.tryFire(CompletableFuture.java:837) ~[?:?]",
    "at java.util.concurrent.CompletableFuture.postComplete(CompletableFuture.java:506) ~[?:?]",
    "at java.util.concurrent.CompletableFuture.completeExceptionally(CompletableFuture.java:2152) ~[?:?]",
    "at org.elasticsearch.common.concurrent.CompletableContext.completeExceptionally(CompletableContext.java:57) ~[elasticsearch-core-7.8.0.jar:7.8.0]",
    "at org.elasticsearch.transport.netty4.Netty4TcpChannel.lambda$addListener$0(Netty4TcpChannel.java:68) ~[?:?]",
    "at io.netty.util.concurrent.DefaultPromise.notifyListener0(DefaultPromise.java:577) ~[?:?]",
    "at io.netty.util.concurrent.DefaultPromise.notifyListeners0(DefaultPromise.java:570) ~[?:?]",
    "at io.netty.util.concurrent.DefaultPromise.notifyListenersNow(DefaultPromise.java:549) ~[?:?]",
    "at io.netty.util.concurrent.DefaultPromise.notifyListeners(DefaultPromise.java:490) ~[?:?]",
    "at io.netty.util.concurrent.DefaultPromise.setValue0(DefaultPromise.java:615) ~[?:?]",
    "at io.netty.util.concurrent.DefaultPromise.setFailure0(DefaultPromise.java:608) ~[?:?]",
    "at io.netty.util.concurrent.DefaultPromise.tryFailure(DefaultPromise.java:117) ~[?:?]",
    "at io.netty.channel.nio.AbstractNioChannel$AbstractNioUnsafe$1.run(AbstractNioChannel.java:263) ~[?:?]",
    "at io.netty.util.concurrent.PromiseTask.runTask(PromiseTask.java:98) ~[?:?]",
    "at io.netty.util.concurrent.ScheduledFutureTask.run(ScheduledFutureTask.java:170) ~[?:?]",
    "at io.netty.util.concurrent.AbstractEventExecutor.safeExecute(AbstractEventExecutor.java:164) ~[?:?]",
    "at io.netty.util.concurrent.SingleThreadEventExecutor.runAllTasks(SingleThreadEventExecutor.java:472) ~[?:?]",
    "at io.netty.channel.nio.NioEventLoop.run(NioEventLoop.java:500) ~[?:?]",
    "at io.netty.util.concurrent.SingleThreadEventExecutor$4.run(SingleThreadEventExecutor.java:989) ~[?:?]",
    "at io.netty.util.internal.ThreadExecutorMap$2.run(ThreadExecutorMap.java:74) ~[?:?]",
    "at java.lang.Thread.run(Thread.java:832) [?:?]",
    "Caused by: io.netty.channel.ConnectTimeoutException: connection timed out: 172.21.0.3/172.21.0.3:9300",
    "at io.netty.channel.nio.AbstractNioChannel$AbstractNioUnsafe$1.run(AbstractNioChannel.java:261) ~[?:?]",
    "... 8 more"
  ]
}
```

### docker-compose.yaml

```yaml
version: '2.2'
services:
  es01:
    image: docker.elastic.co/elasticsearch/elasticsearch:7.8.0
    container_name: es01
    network_mode: host
    environment:
      - node.name=es01
      - cluster.name=es-docker-cluster
      - discovery.seed_hosts=127.0.0.1:9301,{other_instances_ip_and_port}
      - cluster.initial_master_nodes=es03
      - bootstrap.memory_lock=true
      - "ES_JAVA_OPTS=-Xms512m -Xmx512m"
    ulimits:
      nofile:
        soft: 65536
        hard: 65536
      memlock:
        soft: -1
        hard: -1
    volumes:
      - data01:/usr/share/elasticsearch/data
    ports:
      - 9200:9200
      - 9300:9300
        # networks:
        # - elastic
  es02:
    image: docker.elastic.co/elasticsearch/elasticsearch:7.8.0
    container_name: es02
    network_mode: host
    environment:
      - node.name=es02
      - cluster.name=es-docker-cluster
      - discovery.seed_hosts=127.0.0.1,{other_instances_ip_and_port}
      - cluster.initial_master_nodes=es03
      - bootstrap.memory_lock=true
      - "ES_JAVA_OPTS=-Xms512m -Xmx512m"
    ulimits:
      nofile:
        soft: 65536
        hard: 65536
      memlock:
        soft: -1
        hard: -1
    ports:
      - 9201:9200
      - 9301:9300
    volumes:
      - data02:/usr/share/elasticsearch/data
        #    networks:
        #      - elastic

volumes:
  data01:
    driver: local
  data02:
    driver: local

networks:
  elastic:
```

### References
- https://www.elastic.co/guide/en/elasticsearch/reference/7.8/docker.html
