# Web3SDK

[Web3SDK](https://github.com/FISCO-BCOS/web3sdk) supports accessing nodes to check node status, modify the settings and send transactions. FISCO BCOS 2.0 documentation only adapts to Web3SDK 2.0 or above version (which only adapt to FISCO BCOS 2.0 or above version in turn). For Web3SDK 1.2.x please check [Web3SDK 1.2.x Documentation](https://fisco-bcos-documentation.readthedocs.io/zh_CN/release-1.3/docs/web3sdk/config_web3sdk.html).

Main features of version 2.0 include:
- provide Java API to call FISCO BCOS JSON-RPC
- support managing blockchain by precompiled contract
- provide secure and efficient message channel with [Amop](../manual/amop_protocol.md)
- support transaction in OSCCA standard

## Environment requirements

```eval_rst
.. important::

    - java version
     `JDK8 or above version <https://openjdk.java.net/>`_ is required. Because the lack of JCE(Java Cryptography Extension) of OpenJDK in YUM repository of CentOS will fail the connection between Web3SDK and nodes, we recommend users to download it from OpenJDK website when it is CentOS system. `Download here <https://jdk.java.net/java-se-ri/8>`_  `Installation Guide <https://openjdk.java.net/install/index.html>`_
    - FISCO BCOS environment building
     please check `FISCO BCOS installation guide <../installation.html>`_
    - network connectivity
     using telnet command to test if channel_listen_port of nodes connected with Web3SDK is open. If not, please check the network connectivity and security strategy.

```

## Import SDK to java application

   Import SDK to java application through gradle or maven

   gradle:
```bash
compile ('org.fisco-bcos:web3sdk:2.0.0-rc1')
```
   maven:
```bash
<dependency>
    <groupId>org.fisco-bcos</groupId>
    <artifactId>web3sdk</artifactId>
    <version>2.0.0-rc1</version>
</dependency>
```
Because the relative jar archive of the solidity compiler of Ethereum is imported, we need to add a remote repository of Ethereum in the gradle configuration file build.gradle of the java application.

```bash
repositories {
        mavenCentral()
        maven { url "https://dl.bintray.com/ethereum/maven/" }
    }
```
**Note:** if the downloading of the dependent `solcJ-all-0.4.25.jar` is too slow, you can check [here](../manual/console.html#jar) for help.

## Configuration of SDK

### FISCO BCOS node certificate configuration
FISCO BCOS requires SDK to pass two-way authentication on certificate(ca.crt、node.crt) and private key(node.key) when connecting with nodes. Therefore, we need to copy `ca.crt`, `node.crt` and `node.key` under `nodes/${ip}/sdk` folder of node to the resource folder of the project for SDK to connect with nodes.

### Configuration of config file
The config file of java application should be configured. It is noteworthy that FISCO BCOS 2.0 supports [Multi-group function](../design/architecture/group.md), and SDK needs to configure the nodes of the group. The configuration process will be exemplified in this chapter by Spring and Spring Boot project.

### Configuration of Spring project
The following picture shows how `applicationContext.xml` is configured in Spring project.
```xml
<?xml version="1.0" encoding="UTF-8" ?>

<beans xmlns="http://www.springframework.org/schema/beans"
           xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xmlns:p="http://www.springframework.org/schema/p"
           xmlns:tx="http://www.springframework.org/schema/tx" xmlns:aop="http://www.springframework.org/schema/aop"
           xmlns:context="http://www.springframework.org/schema/context"
           xsi:schemaLocation="http://www.springframework.org/schema/beans
    http://www.springframework.org/schema/beans/spring-beans-2.5.xsd
         http://www.springframework.org/schema/tx
    http://www.springframework.org/schema/tx/spring-tx-2.5.xsd
         http://www.springframework.org/schema/aop
    http://www.springframework.org/schema/aop/spring-aop-2.5.xsd">


        <bean id="encryptType" class="org.fisco.bcos.web3j.crypto.EncryptType">
                <constructor-arg value="0"/> <!-- 0:standard 1:guomi -->
        </bean>

        <bean id="groupChannelConnectionsConfig" class="org.fisco.bcos.channel.handler.GroupChannelConnectionsConfig">
                <property name="allChannelConnections">
                        <list>  <!-- each group needs to configure a beam, each group can configure multiple nodes-->
                                <bean id="group1"  class="org.fisco.bcos.channel.handler.ChannelConnections">
                                        <property name="groupId" value="1" /> <!-- groupID -->
                                        <property name="connectionsStr">
                                                <list>
                                                        <value>127.0.0.1:20200</value>  <!-- IP:channel_port -->
                                                        <value>127.0.0.1:20201</value>
                                                </list>
                                        </property>
                                </bean>
                                <bean id="group2"  class="org.fisco.bcos.channel.handler.ChannelConnections">
                                        <property name="groupId" value="2" /> <!-- groupID -->
                                        <property name="connectionsStr">
                                                <list>
                                                        <value>127.0.0.1:20202</value>
                                                        <value>127.0.0.1:20203</value>
                                                </list>
                                        </property>
                                </bean>
                        </list>
                </property>
        </bean>

        <bean id="channelService" class="org.fisco.bcos.channel.client.Service" depends-on="groupChannelConnectionsConfig">
                <property name="groupId" value="1" /> <!-- configure and connect group 1 -->
                <property name="agencyName" value="fisco" /> <!-- configure agency name -->
                <property name="allChannelConnections" ref="groupChannelConnectionsConfig"></property>
        </bean>

</beans>
```
Configuration items of `applicationContext.xml`:
- encryptType: the switch of oscca algorithm (default as 0)                              
  - 0: not use oscca algorithm for transactions                              
  - 1: use oscca algorithm for transactions (switch on oscca algorithm, nodes will be in oscca standard, for reference of building oscca-standard FISCO BCOS blockchain please check [here](../manual/guomi_crypto.md))
- groupChannelConnectionsConfig:
  - configure the group to be connected, which can be one or more, each should be given a group ID
  - each group configures one or more nodes, set `listen_ip` and `channel_listen_port` of `[rpc]` in node config file **config.ini**.
- channelService: configure the the group connected with SDK through the given group ID, which is in the configuration of groupChannelConnectionsConfig. SDK will be connected with each node of the group and randomly choose one node to send request.

### Configuration of Spring Boot project
The configuration of `application.yml` in Spring Boot is exemplified below.
```yml
encrypt-type: 0  # 0:standard, 1:guomi
group-channel-connections-config:
  all-channel-connections:
  - group-id: 1  #group ID
    connections-str:
                    - 127.0.0.1:20200  # node listen_ip:channel_listen_port
                    - 127.0.0.1:20201
  - group-id: 2  
    connections-str:
                    - 127.0.0.1:20202  # node listen_ip:channel_listen_port
                    - 127.0.0.1:20203

channel-service:
  group-id: 1 # The specified group to which the SDK connects
  agency-name: fisco # agency name
```
The configuration items of `application.yml` corresponds with `applicationContext.xml`. For detailed introduction please check the configuration description of `applicationContext.xml`.

## Operation of SDK

### Guide for development of Spring
#### to call API of SDK (check the settings of [Web3SDK API list](#web3sdk-api) or related blockchain data).
##### call API of SDK Web3j
load the config file, connect SDK with nodes, acquire web3j object and call API. The codes are exemplified here:
```java
    //read config file and connect SDK with nodes
    ApplicationContext context = new ClassPathXmlApplicationContext("classpath:applicationContext.xml");
    Service service = context.getBean(Service.class);
    service.run();
    ChannelEthereumService channelEthereumService = new ChannelEthereumService();
    channelEthereumService.setChannelService(service);

    //acquire Web3j object
    Web3j web3j = Web3j.build(channelEthereumService, service.getGroupId());
    //call API getBlockNumber through Web3j object
    BigInteger blockNumber = web3j.getBlockNumber().send().getBlockNumber();
    System.out.println(blockNumber);
```
**Note:** The transaction process of SDK is default limited to 60 seconds. Transactions with no response in 60 seconds will be considered time out. The timeline can be modified through `ChannelEthereumService`. Here is the example:
```java
// set the transaction timeline to 100000 milliseconds， namely 100 second
channelEthereumService.setTimeout(100000);
```
##### call API of SDK Precompiled
load config file, connect SDK with nodes, acquire Precompiled Service object of SDK and call API. The codes are as below:
```java
    //read config file, connect SDK with nodes and acquire Web3j object
    ApplicationContext context = new ClassPathXmlApplicationContext("classpath:applicationContext.xml");
    Service service = context.getBean(Service.class);
    service.run();
    ChannelEthereumService channelEthereumService = new ChannelEthereumService();
    channelEthereumService.setChannelService(service);
    Web3j web3j = Web3j.build(channelEthereumService, service.getGroupId());
    //fill in user private key for signature in transactions
    Credentials credentials = Credentials.create("b83261efa42895c38c6c2364ca878f43e77f3cddbc922bf57d0d48070f79feb6");
    //acquire SystemConfigService object
    SystemConfigSerivce systemConfigSerivce = new SystemConfigSerivce(web3j, credentials);
    //call API setValueByKey through SystemConfigService object
    String result = systemConfigSerivce.setValueByKey("tx_count_limit", "2000");
    //call API getSystemConfigByKey through Web3j object
    String value = web3j.getSystemConfigByKey("tx_count_limit").send().getSystemConfigByKey();
    System.out.println(value);
```
##### Create and use specific external account
sdk needs an external account to send transactions. Here is a way to create a external account.
```java
//create regular external account
EncryptType.encryptType = 0;
//create OSCCA external account, which is needed when sending transaction to OSCCA-standard blockchain nodes
// EncryptType.encryptType = 1;
Credentials credentials = GenCredential.create();
//account address
String address = credentials.getAddress();
//account private key
String privateKey = credentials.getEcKeyPair().getPrivateKey().toString(16);
//account public key
String publicKey = credentials.getEcKeyPair().getPublicKey().toString(16);
```
Use specific external account
```java
//Set specific external account by specifying the private key
Credentials credentials = GenCredential.create(privateKey);
```

##### Load private key file
After the script `get_accounts.sh` generates PEM or PKCS12 private key file (the use of get-account script is introduced in [Account management document](../tutorial/account.md)), the account can be operated through loading PEM or PKCS12 private key file. There are 2 ways to load private key: P12Manager and PEMManager. P12Manager is to load PKCS12 private key file; PEMManager is to load PEM private key file.

* P12Manager example:
Configure private key file route and password of PKCS12 account in applicationContext.xml
```xml
<bean id="p12" class="org.fisco.bcos.channel.client.P12Manager" init-method="load" >
	<property name="password" value="123456" />
	<property name="p12File" value="classpath:0x0fc3c4bb89bd90299db4c62be0174c4966286c00.p12" />
</bean>
```
development code
```java
//load Bean
ApplicationContext context = new ClassPathXmlApplicationContext("classpath:applicationContext.xml");
P12Manager p12 = context.getBean(P12Manager.class);
//offer password to get ECKeyPair, password is set when creating p12 account file
ECKeyPair p12KeyPair = p12.getECKeyPair(p12.getPassword());

//output private key and public key in hexadecimal string
System.out.println("p12 privateKey: " + p12KeyPair.getPrivateKey().toString(16));
System.out.println("p12 publicKey: " + p12KeyPair.getPublicKey().toString(16));

//generate Credentials for web3sdk
Credentials credentials = Credentials.create(p12KeyPair);
System.out.println("p12 Address: " + credentials.getAddress());
```

* PEMManager example

Configure private key route of PEM account in applicationContext.xml
```xml
<bean id="pem" class="org.fisco.bcos.channel.client.PEMManager" init-method="load" >
	<property name="pemFile" value="classpath:0x0fc3c4bb89bd90299db4c62be0174c4966286c00.pem" />
</bean>
```
load code
```java
//load Bean
ApplicationContext context = new ClassPathXmlApplicationContext("classpath:applicationContext-keystore-sample.xml");
PEMManager pem = context.getBean(PEMManager.class);
ECKeyPair pemKeyPair = pem.getECKeyPair();

//output private key and public key in hexadecimal string
System.out.println("PEM privateKey: " + pemKeyPair.getPrivateKey().toString(16));
System.out.println("PEM publicKey: " + pemKeyPair.getPublicKey().toString(16));

//generate Credentials for web3sdk
Credentials credentialsPEM = Credentials.create(pemKeyPair);
System.out.println("PEM Address: " + credentialsPEM.getAddress());
```

#### To deploy and call contract through SDK
##### Prepare java contract file
Console offers a contract compiler for developers to compile the solidity contract to java contract. For operation steps please check  [here](../manual/console.html#id6).

##### Deploy and call contract
The core function of SDK is to deploy/load contract, and to call API of contract to realize transactions. The deployment of contract calls the deploy method of java class and obtains contract object, which can call getContractAddress method to get the address for contract or other methods for other functions. If the contract has been deployment already, contract object can be loaded by calling load method according to the address of the deployed contract, other methods of which can also be called.
```java
    //read config file, connect SDK with nodes, get web3j object
    ApplicationContext context = new ClassPathXmlApplicationContext("classpath:applicationContext.xml");
    Service service = context.getBean(Service.class);
    service.run();
    ChannelEthereumService channelEthereumService = new ChannelEthereumService();
    channelEthereumService.setChannelService(service);
    channelEthereumService.setTimeout(10000);
    Web3j web3j = Web3j.build(channelEthereumService, service.getGroupId());
    //prepare parameters for deploying and calling contract
    BigInteger gasPrice = new BigInteger("300000000");
    BigInteger gasLimit = new BigInteger("300000000");
    String privateKey = "b83261efa42895c38c6c2364ca878f43e77f3cddbc922bf57d0d48070f79feb6";
    //specify external account private key for transaction signature
    Credentials credentials = GenCredential.create(privateKey);
    //deploy contract
    YourSmartContract contract = YourSmartContract.deploy(web3j, credentials, new StaticGasProvider(gasPrice, gasLimit)).send();
    //load contract according to the contract address
    //YourSmartContract contract = YourSmartContract.load(address, web3j, credentials, new StaticGasProvider(gasPrice, gasLimit));
    //send transaction by calling contract method
    TransactionReceipt transactionReceipt = contract.someMethod(<param1>, ...).send();
    //check data status of the contract by inquiry contract method
    Type result = contract.someMethod(<param1>, ...).send();
```
### Guide for Spring Boot development
We take [spring-boot-starter](https://github.com/FISCO-BCOS/spring-boot-starter) as an example. Spring Boot and Spring are similar in development process, except the distinction in config file. Here we will provide some test examples. For detail description on the projects please check the README documents.

### Operations on OSCCA function of SDK
- Preconditions: FISCO BCOS blockchain in OSCCA standard, to build it using OSCCA algorithm, please check [the operation tutorial](../manual/guomi_crypto.md).
- switch on OSCCA function:  set encryptType as 1 in application.xml/application.yml configuration.

The OSCCA version of SDK shares the same method on calling API with the regular version. The difference is that OSCCA SDK requires a OSCCA version of java contract. To download the jar archive of OSCCA compiler, which is needed for transforming solidity contract into OSCCA version of java contract, please check [here](../manual/console.html#jar). You can also create a lib folder in src folder to place the jar archive when finishing downloading, and modify build.gradle, remove the regular jar archive to replace it with the OSCCA version.
  ```
    compile ("org.fisco-bcos:web3sdk:x.x.x"){ //For example: web3sdk:2.0.0
         exclude module: 'solcJ-all'
    }
    // jar archive of OSCCA contract compiler 0.4
    compile files('lib/solcJ-all-0.4.25-gm.jar')
    // jar archive of OSCCA contract compiler 0.5
    // compile files('lib/solcJ-all-0.5.2-gm.jar')
  ```
It shares the same steps with regular SDK in transformation of solidity contract to OSCCA java contract and deploy/call methods.

## Web3SDK API

Web3SDK API is separated into Web3j API and Precompiled Service API. Through Web3j API we can check the blockchain status, send/inquiry transactions; Precompiled Service API is to manage configurations and to realize some functions.

### Web3j API
Web3j API is the RPC API of FISCO BCOS called by web3j object, and it has the same API name with RPC API. Please check [RPC API Documentation](../api.md).

### Precompiled Service API
### Precompiled Service API
Precomplied contract is an efficient smart contract realized in C++ on FISCO BCOS platform. SDK provides java interface of the precompiled contract, through calling which console can execute commands. For operational reference you can check [the guide for console operations](../manual/console.md). SDK also provides Precompiled Servide class, including PermissionService for distributed permission control, CnsService for [CNS](../design/features/cns_contract_name_service.md), SystemConfigService for system property configuration and ConsensusService for node type configuration. The related error codes are collected here: [Precompiled Service API Error Codes](../api.html#precompiled-service-api)

#### PermissionService
SDK supports [distributed permission control](../manual/permission_control.md). PermissionService can configure permission information. The APIs are here:
- **public String grantUserTableManager(String tableName, String address):** set permissions according to user list name and exterior account address.
- **public String revokeUserTableManager(String tableName, String address):** revoke permissions according to user list name and exterior account address.
- **public List\<PermissionInfo\> listUserTableManager(String tableName):** inquire permission record list according to the user list name (each record contains exterior account address and effective block number).
- **public String grantDeployAndCreateManager(String address):** grant permission for exterior account to deploy contract and create user list.
- **public String revokeDeployAndCreateManager(String address):** revoke permission for exterior account to deploy contract and create user list.
- **public List\<PermissionInfo\> listDeployAndCreateManager():** inquire the permission record list for exterior account to deploy contract and create user list.
- **public String grantPermissionManager(String address):** grant permission to exterior accounts for permission control.
- **public String revokePermissionManager(String address):** revoke permission to exterior accounts for permission control
- **public List\<PermissionInfo\> listPermissionManager():** inquire permission record list of exterior accounts on permission control.
- **public String grantNodeManager(String address):** grant permission to exterior accounts for node management.
- **public String revokeNodeManager(String address):** revoke permission to exterior accounts for node management.
- **public List\<PermissionInfo\> listNodeManager():** inquire permission records on node management.
- **public String grantCNSManager(String address):** grant permission to exterior account for CNS.
- **public String revokeCNSManager(String address):** revoke permission to exterior account for CNS.
- **public List\<PermissionInfo\> listCNSManager():** inquire permission records of CNS
- **public String grantSysConfigManager(String address):** grant permission to exterior account for parameters management.
- **public String revokeSysConfigManager(String address):** revoke permission to exterior account for parameters management.
- **public List\<PermissionInfo\> listSysConfigManager():** inquire permission records of parameters management.

#### CnsService
SDK supports [CNS](../design/features/cns_contract_name_service.md). CnsService can configure CNS. The APIs are here:
- **String registerCns(String name, String version, String address, String abi):** register CNS according to contract name, version, address and contract abi.
- **String getAddressByContractNameAndVersion(String contractNameAndVersion):** inquire contract address according to contract name and version (connected with colon). If lack of contract version, it is defaulted to be the latest version.
- **List\<CnsInfo\> queryCnsByName(String name):** inquire CNS information according to contract name.
- **List\<CnsInfo\> queryCnsByNameAndVersion(String name, String version):** inquire CNS information according to contract name and version.

#### SystemConfigSerivce
SDK offers services for system configuration. SystemConfigSerivce can configure system property value (currently support tx_count_limit and tx_gas_limit)。 The API is here:
- **String setValueByKey(String key, String value):** set value according to the key（to check the value, please refer to getSystemConfigByKey in Web3j API).

#### ConsensusService
SDK supports configuration of [node type](../design/security_control/node_management.html#id6). ConsensusService is used to set node type. The APIs are here:
- **String addSealer(String nodeId):** set the node as consensus node according to node ID.
- **String addObserver(String nodeId):** set the node as observer node according to node ID.
- **String removeNode(String nodeId):** set the node as free node according to node ID.

#### CRUDService
SDK supports CRUD (Create/Retrieve/Updata/Delete) operations. CRUDService of table include create, insert, retrieve, update and delete. Here are its APIs:
- **int createTable(Table table)：** create table and table object, set the name of table, main key field and other fields; names of other fields are character strings separated by comma; return table status value, return 0 when it is created successfully.
- **int insert(Table table, Entry entry)：** insert records, offer table object and Entry object, set table name and main key name; Entry is map object, offer inserted field name and its value, main key field is necessary; return the number of inserted records.
- **int update(Table table, Entry entry, Condition condition)：** update records, offer table object, Entry object, Condition object. Table object needs to be set with table name and main key field name; Entry is map object, offer new field name and value; Condition object can set new conditions； return the number of new records.
- **List\<Map\<String, String\>\> select(Table table, Condition condition)：** retrieve records, offer table object and Condition object. Table object needs to be set with table name and main key field name; Condition object can set condition for retrieving; return the retrieved record.
- **int remove(Table table, Condition condition)：** remove records, offer table object and Condition object. Table object needs to be set with table name and main key field name; Condition object can set conditions for removing; remove the number of removed records.
- **Table desc(String tableName)：** inquire table information according to table name, mainly contain main key and other property fields; return table type, mainly containing field name of main key and other property.
