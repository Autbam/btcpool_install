# btcpool矿池-测试环境搭建及使用cgminer测试
<br>

不管你做哪个币的池，都需要比特币源码，因为我们编译矿池的时候需要比特币源码里面的库

# 安装Bitcoind+ZMQ

#### Dependencies
    apt-get -y install build-essential libtool autotools-dev automake autoconf pkg-config bsdmainutils python3
    apt-get -y install libssl-dev libboost-all-dev libevent-dev
    apt-get -y install libdb-dev libdb++-dev
    apt-get -y install libminiupnpc-dev libzmq3-dev
    apt-get -y install libqt5gui5 libqt5core5a libqt5dbus5 qttools5-dev qttools5-dev-tools libprotobuf-dev protobuf-compiler libqrencode-dev



    #To Build
    wget https://github.com/bitcoin/bitcoin/archive/v0.16.3.tar.gz
    tar -zxvf v0.16.3.tar.gz
若我们想要创建非比特币矿池，我们无需对比特币源码进行编译，因此以下命令可以不执行
    cd bitcoin-0.16.3/
    ./autogen.sh
    ./configure --with-incompatible-bdb --prefix=/work/bitcoin
    make
    make install


    #start/stop service
    cd /work/bitcoin/bin/
    ./bitcoind --daemon -testnet -zmqpubhashtx=tcp://0.0.0.0:18331 -zmqpubhashblock=tcp://0.0.0.0:18331
    #./bitcoin-cli -testnet stop

安装ZooKeeper
    #Install ZooKeeper
    apt-get install -y zookeeper zookeeper-bin zookeeperd

    #mkdir for data
    mkdir -p /work/zookeeper
    mkdir /work/zookeeper/version-2
    touch /work/zookeeper/myid
    chown -R zookeeper:zookeeper /work/zookeeper

    #set machine id
    echo 1 > /work/zookeeper/myid

    #edit config file
    vim /etc/zookeeper/conf/zoo.cfg

    initLimit=5
    syncLimit=2
    clientPort=2181
    clientPortAddress=127.0.0.1
    dataDir=/work/zookeeper
    #伪分布式
    server.1=127.0.0.1:2888:3888

    #start/stop service
    service zookeeper restart
    #service zookeeper start/stop/restart/status
    安装Kafka
    #install depends
    apt-get install -y default-jre

    #install Kafka
    mkdir /root/source
    cd /root/source
    wget https://archive.apache.org/dist/kafka/2.2.1/kafka_2.11-2.2.1.tgz
    mkdir -p /work/kafka
    cd /work/kafka
    tar -zxf /root/source/kafka_2.11-2.2.1.tgz --strip 1

    #edit conf
    vim /work/kafka/config/server.properties

    broker.id=1
    offsets.topic.replication.factor=1
    message.max.bytes=20000000
    replica.fetch.max.bytes=30000000
    log.dirs=/work/kafka-logs
    listeners=PLAINTEXT://127.0.0.1:9092
    #伪分布式
    zookeeper.connect=127.0.0.1:2181

    #start server
    cd /work/kafka
    nohup /work/kafka/bin/kafka-server-start.sh /work/kafka/config/server.properties > /dev/null 2>&1 &
安装BTCPool
    #Build
    cd /work

安装需要的包

    apt-get update
    apt-get install -y build-essential autotools-dev libtool autoconf automake pkg-config cmake \
                   openssl libssl-dev libcurl4-openssl-dev libconfig++-dev \
                   libboost-all-dev libgmp-dev libmysqlclient-dev libzookeeper-mt-dev \
                   libzmq3-dev libgoogle-glog-dev libhiredis-dev zlib1g zlib1g-dev \
                   libsodium-dev libprotobuf-dev protobuf-compiler

    # zmq-v4.1.5
    mkdir -p /root/source && cd /root/source
    wget https://github.com/zeromq/zeromq4-1/releases/download/v4.1.5/zeromq-4.1.5.tar.gz
    tar zxvf zeromq-4.1.5.tar.gz
    cd zeromq-4.1.5
    ./autogen.sh && ./configure && make -j $CPUS
    make check && make install && ldconfig

    # glog-v0.3.4
    mkdir -p /root/source && cd /root/source
    wget https://github.com/google/glog/archive/v0.3.4.tar.gz
    tar zxvf v0.3.4.tar.gz
    cd glog-0.3.4
    ./configure && make -j $CPUS && make install

    # librdkafka-v0.9.1
    apt-get install -y zlib1g zlib1g-dev
    mkdir -p /root/source && cd /root/source
    wget https://github.com/edenhill/librdkafka/archive/0.9.1.tar.gz
    tar zxvf 0.9.1.tar.gz
    cd librdkafka-0.9.1
    ./configure && make -j $CPUS && make install

    # libevent-2.0.22-stable
    mkdir -p /root/source && cd /root/source
    wget https://github.com/libevent/libevent/releases/download/release-2.0.22-stable/libevent-2.0.22-stable.tar.gz
    tar zxvf libevent-2.0.22-stable.tar.gz
    cd libevent-2.0.22-stable
    ./configure
    make -j $CPUS
    make install

    # btcpool
    mkdir -p /work && cd /work
    git clone https://github.com/btccom/btcpool.git
    cd /work/btcpool
    mkdir build && cd build

以下内容根据需要2选1

    # Release build:
    cmake -DJOBS=4 -DCHAIN_TYPE=BTC -DCHAIN_SRC_ROOT=/work/bitcoin-0.16.3 ..
    make -j$(nproc)

    # Debug build:
    cmake -DCMAKE_BUILD_TYPE=Debug -DCHAIN_TYPE=BTC -DCHAIN_SRC_ROOT=/work/bitcoin-0.16.3 ..
    make -j$(nproc)

注意：
-DCHAIN_TYPE=coin   这里的coin根据你想要编译的币种自行修改(仅BTC、BCH等sha256算法的币需要修改，其他币种ETH，Grin等默认使用BTC)；
-DCHAIN_SRC_ROOT=bitcoindir 这里的bitcoindir是比特币源码路径，根据需要自行修改


启动BTCPool及cgminer测试btcpool

将install目录下的init_folders.sh文件复制到 build目录
然后执行
    ./init_folders.sh
以上代码作用为：对编译成功的二进制文件生成相应目录并创建快捷方式

# 启动gbtmaker
## 配置gbtmaker

    cd /work/btcpool/build/
    mkdir run_gbtmaker
    cd run_gbtmaker/
    vim gbtmaker.cfg

    gbtmaker = {
      rpcinterval = 5;
      is_check_zmq = true;
    };
    bitcoind = {
      zmq_addr = "tcp://127.0.0.1:18331";
      rpc_addr    = "http://127.0.0.1:18332";
      rpc_userpwd = "bitcoinrpc:xxxx";
    };
    kafka = {
      brokers = "127.0.0.1:9092";
    };

## 启动gbtmaker
    cd /work/btcpool/build/run_gbtmaker/
    mkdir log_gbtmaker
    ./gbtmaker -c ./gbtmaker.cfg -l ./log_gbtmaker &
    tail -f log_gbtmaker/gbtmaker.INFO
# 启动jobmaker
## 配置jobmaker
    cd /work/btcpool/build/
    cd run_jobmaker/
    vim jobmaker.cfg

    testnet = true;
    jobmaker = {
      stratum_job_interval = 20;
      gbt_life_time = 90;
      empty_gbt_life_time = 15;
      file_last_job_time = "/work/btcpool/build/run_jobmaker/jobmaker_lastjobtime.txt";
      block_version = 0;
    };
    kafka = {
      brokers = "127.0.0.1:9092";
    };
    zookeeper = {
      brokers = "127.0.0.1:2181";
    };
    pool = {
      payout_address = "mi9vpXBWJ31WGpRU7n7VJQG4PvTndHBoCN";
      coinbase_info = "region1/Project BTCPool/";
    };

## 启动jobmaker
    cd /work/btcpool/build/run_jobmaker/
    mkdir log_jobmaker
    ./jobmaker -c ./jobmaker.cfg -l ./log_jobmaker &
    tail -f log_jobmaker/jobmaker.INFO
# 启动sserver
## 配置sserver
    cd /work/btcpool/build/
    cd run_sserver/
    vim ./sserver.cfg

    testnet = true;
    kafka = {
      brokers = "127.0.0.1:9092";
    };
    sserver = {
      ip = "0.0.0.0";
      port = 3333;
      id = 1;
      file_last_notify_time = "/work/btcpool/build/run_sserver/sserver_lastnotifytime.txt";
      enable_simulator = false;
      enable_submit_invalid_block = false;
      share_avg_seconds = 10;
    };
    users = {
      list_id_api_url = "http://index.qubtc.com/apidemo.php";
    };

## 启动sserver
    cd /work/btcpool/build/run_sserver/
    mkdir log_sserver
    ./sserver -c ./sserver.cfg -l ./log_sserver &
    tail -f log_sserver/sserver.INFO
# 测试矿池是否已经正常运作
## cgminer测试BTC币种
### 安装cgminer
    cd /work/
    apt-get -y install build-essential autoconf automake libtool pkg-config libcurl3-dev libudev-dev
    apt-get -y install libusb-1.0-0-dev
    git clone https://github.com/ckolivas/cgminer.git
    cd cgminer
    sh autogen.sh
    ./configure --enable-cpumining --disable-opencl
    make

### cgminer测试
    ./cgminer --cpu-threads 3 -o stratum+tcp://127.0.0.1:3333 -u jack -p x

    ./cgminer --cpu-threads 3 --url 127.0.0.1:3333 --userpass jack:x
    #./cgminer -o stratum+tcp://127.0.0.1:3333 -u jack -p x --debug --protocol-dump
    #--debug，调试模式
    #--protocol-dump，协议输出
## claymore测试ETH币种
    ./EthDcrMiner64 -epool 127.0.0.1:3333 -ewal jack -epsw x -allpools 1
# 启动blkmaker
## 安装MySQL
请自行百度

## 配置blkmaker
    cd /work/btcpool/build/
    cd run_blkmaker/
    vim blkmaker.cfg

    bitcoinds = (
    {
      rpc_addr    = "http://127.0.0.1:18332";
      rpc_userpwd = "bitcoinrpc:xxxx";
    }
    );
    kafka = {
      brokers = "127.0.0.1:9092";
    };
    pooldb = {
      host = "localhost";
      port = 3306;
      username = "develop";
      password = "iZ2ze3r0li2kgfvjkvs2xeZ";
      dbname = "bpool_local_db";
    };

## 启动blkmaker
    cd /work/btcpool/build/run_blkmaker/
    mkdir log_blkmaker
    ./blkmaker -c ./blkmaker.cfg -l ./log_blkmaker &
    tail -f log_blkmaker/blkmaker.INFO
