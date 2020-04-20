# Tutorial para desplegar chaincode Marble Network sobre Red Basica de la version 2.0.

Este tutorial esta basado en el despliegue de test-network [documentacion de Fabric 2.0](https://hyperledger-fabric.readthedocs.io/en/release-2.0/test_network.html) y una guia desarrollada para desplegar el
ejemplo marble-network desarrollado por OrderStack y descrito en el siguiente blog [Deploy a Single Organization Blockchain Network Using Hyperledger Fabric 2.0 and NodeJs](https://medium.com/@social_52848/deploy-a-single-organization-blockchain-network-using-hyperledger-fabric-2-0-and-nodejs-acdadb2b4654)


Antes de empezar es importante:
1. [Instalar la version de desarrollo de Fabric 2.0. junto con los prerequisitos](https://hyperledger-fabric.readthedocs.io/en/release-2.0/getting_started.html)
2. Se recomienda probar la instalacion siguiendo el tutorial de [test-network](https://hyperledger-fabric.readthedocs.io/en/release-2.0/test_network.html)


Marbles-network es una version simplificada del test-network que contiene una organizacion con un solo nodo peer y un ordenador (el certification authority es opcional). El objetivo del tutorial es iniciar la red, crear un canal de comunicacion, instalar el chaincode 'marbles_chaincode.js' e interactural con el chaincode a traves de la linea de comando, CLI, del peer.

## 1. Crear la carpeta marbles-network

En la instalacion se debio clonar una carpeta fabric-samples que contiene los ejemplo correpondientes a la distribucion 2.0 de Hyperledger Fabric.

```
cd fabric-samples
```
Dentro de esta carpeta se debe clonar una carpeta del repositorios de OrderStack
```
git clone https://github.com/orderstack/hyperledger-fabric-v2-starter/tree/master/network
```
Se debe haber creado una carpeta network dentro de fabric-samples (es recomendable cambiarle el nombre a marbles-network)

## 2. Iniciar las red y el canal

Para iniciar la red se navega a la carpeta scripts y se utiliza el siguiente script. 

```
cd marbles-network/scripts/
```
El script no solo crea la red sino que ademas habilita un canal de comunicacion (mychannel) y vincula el peer0 al canal.

```
./network.sh up createChannel
```
inspeccionar la red
```
docker ps
```

## 3. Instalar y habilitar chaincoide en la red

Las labores de instalacion y habilitacion del contrato inteligente (chaincoide) en Fabric 2.0. debe tener en cuenta el [ciclo de vida del contrato](https://hyperledger-fabric.readthedocs.io/en/release-2.0/chaincode_lifecycle.html), uno de los elementos novedosos de la distribucion 2.0.
En particular este tutorial se concentra en la etapa de intalacion y definicion del chaincode a lo largo de la red.

Para llevar acabo todo el proceso se puede utilizar el sigueinte script desarrollado por OrderStack

```
./deployCCSingleOrg.sh
```

Adicionalmente a continuacion vamos a llevar acabo una descripcion detallada de las etapas del proceso (pendiente). 


## 4. Metodos de Escritura y lectura del contrato inteligente

Para utilizar el chaincode debemos utilizar una serie de comandos que nos permiten escribir y leer sobre el registro compartido. 

Para llevar acabo un test que permita visualizar la interaccion con el contrato se puede utilizar el sigueinte script de ejemplo desarrollado por OrderStack

```
./testCCSingleOrg.sh
```

Adicionalmente a continuacion vamos a llevar acabo una descripcion detallada de las diferentes funcionalidades del contrato inteligente. 

Debemos fijar unas variables de configuracion para utilizar como peer0 de la Org1 (se podria utilizar una configuracion mas ligera sin TLS?).
Estas variables de configuracion permiten identificar los artefactos que permiten establecer la identificacion segura de
nodo que ejecuta las transacciones y por lo tanto deben ajustarce de acuerdo a las interacciones a nivel de nodo. 

```
export PATH=${PWD}/../../bin:${PWD}:$PATH
export FABRIC_CFG_PATH=${PWD}/../../config
# Activar conjunto de funciones para indicar el nodo que ejecuta el chaincode a traves de CLI
. ./envVar.sh
#cuando se indica 1 hace referencia a Org1
setGlobals 1
parsePeerConnectionParameters 1
```

Ahora vamos a Escribir el primer objeto en el registro

```
peer chaincode invoke \
        -o localhost:7050 \
        --ordererTLSHostnameOverride orderer.example.com \
        -C mychannel \
        -n marbles \
        -c '{"Args":["initMarble","marble1","blue","35","tom"]}' \
        --tls \
        --cafile ${ORDERER_CA} \
		$PEER_CONN_PARMS

```

Ahora vamos a Leer el primer objeto en el registro

```
peer chaincode invoke \
        -o localhost:7050 \
        --ordererTLSHostnameOverride orderer.example.com \
        -C mychannel \
        -n marbles \
        -c '{"Args":["readMarble","marble1"]}' \
        --tls \
        --cafile ${ORDERER_CA} \
		$PEER_CONN_PARMS

```

Ahora vamos a Escribir en el registro mediante la ejecucion de una transaccion donde se transfiere la propiedad de un marble

```
peer chaincode invoke \
        -o localhost:7050 \
        --ordererTLSHostnameOverride orderer.example.com \
        -C mychannel \
        -n marbles \
        -c '{"Args":["transferMarble","marble1","jerry"]}' \
        --tls \
        --cafile ${ORDERER_CA} \
		$PEER_CONN_PARMS

```

Ahora vamos a Leer el objeto en el registro para ver el cambio

```
peer chaincode invoke \
        -o localhost:7050 \
        --ordererTLSHostnameOverride orderer.example.com \
        -C mychannel \
        -n marbles \
        -c '{"Args":["readMarble","marble1"]}' \
        --tls \
        --cafile ${ORDERER_CA} \
		$PEER_CONN_PARMS

```


### Mirar el ultimo bloque

```
docker exec peer0.org1.example.com peer channel getinfo -c mychannel
```

### Revisar el log file de un contenedor

```
docker logs peer0.org1.example.com
```


## Limpieza de los contenedores

Parar detener y destruir los Contenedores de Fabric y por lo tanto la red,

```
./network.sh down
```

Parar los que no se eleminan automaticamente, debe utilizar nombre o numero
```
docker stop [number]
```
