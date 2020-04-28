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
El script no solo crea la red sino que ademas habilita un canal de comunicacion (mychannel), vincula el peer0 al canal y hace explicito la utilizacion del couchDB.

```
./network.sh up createChannel -s couchdb
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

Adicionalmente a continuacion vamos a llevar acabo una descripcion detallada de las etapas del proceso de ciclo de vida del contrato. El ciclo de vida del contrato hace referencia a varios procesos de administracion de los contratos inteligentes en la red: instalacion y habilitacion de del contrato a lo largo de la red, actualizacion (update) u otros escenario de despliegue. Este tutorial se enfoca en el primer tipo de procesos de instalacion y habilitacion del contrato a lo largo de la red.

La instalacion y habilitacion del contrato a lo largo de la red tambien contiene una serie de etapas: 

1. Creacion de un paquete (*.tar.gz) que contiene el programa que soporta el contrato, un archivo con la metadata (Chaincode-Package-Metadata.json) que describe entre otras cosas las dependencias y puede contener la informacion sobre la indexacion particular que se va a utilizar para la base de datos que va a ser creada mediante la utilizacion del contrato inteligente (ver [Add the index to your chaincode folder](https://hyperledger-fabric.readthedocs.io/en/release-2.0/couchdb_tutorial.html)). La descripcion detallada de estos dos archivos se encuantra en el repositorio [Contratos-js](https://github.com/FabricColombia/Contratos-js). La creacion de este archivo se puede hacer de diferentes formas, en este tutorial se utilizara las imagenes del peer que var a realizar la operacion 'peer0'. En el caso del ejemplo de marbles el chaincode y archivos auxiliares debe contener una estrucura de archivos similar a la que se encuantra en la [documentacion](https://github.com/hyperledger/fabric-samples/tree/master/chaincode/marbles02/javascript) y que esta incluida en el folder clonado. 

Despues de tener identificados los contenidos que del paquete e iniciar la red, se deben fijar unas variables de configuracion para interactuar sobre la red como peer0 de la Org1. Las variables de configuracion definen el tipo de lenguaje a utilizar, en este caso javascript. Adicionalmente,las variables de configuracion permiten identificar los artefactos que permiten establecer la identificacion segura de
nodo que ejecuta las transacciones y por lo tanto deben ajustarce de acuerdo a las interacciones a nivel de nodo. 

```
starttime=$(date +%s)
CC_SRC_LANGUAGE=javascript
CC_RUNTIME_LANGUAGE=node
CC_SRC_PATH=../chaincode/marbles/javascript
export PATH=${PWD}/../../bin:${PWD}:$PATH
export FABRIC_CFG_PATH=${PWD}/../../config
# Activar conjunto de funciones para indicar el nodo que ejecuta el chaincode a traves de CLI
. ./envVar.sh
#cuando se indica 1 hace referencia a Org1
setGlobals 1
```

Ahora vamos a crear el paquete que contiene la infromacion necesaria para la instalacion del contrato inteligente,

```
peer lifecycle chaincode package marbles.tar.gz  \
  --path $CC_SRC_PATH \
  --lang $CC_RUNTIME_LANGUAGE \
  --label marblesv1
```

Se debio crear un archivo 'marbles.tar.gz' en el directorio actual.

2. Instalacion del chaincode en el peer0.
```
peer lifecycle chaincode install marbles.tar.gz
```
Despues de instalado deben observar un mensaje que indique que la instalacion ha sido exitosa y tambien reportar el identificador del paquete (nombre del paquete mas el hash del paquete). Como medida de seguridad ahora se debe guardar este informacion como una variable de configuracion. Tambien se puede solicitar el indicador del paquete mediante la siguiente instruccion, 

```
peer lifecycle chaincode queryinstalled
```
La informacion obtenida debe guardarse en una variable de configuracion, por ejemplo,
```
export PACKAGE_ID=marblesv1:8b210cb330fb0e1fd376d9727ed29b59fb569a4e6e7cde71bbbaef31895489e7
```

3. Aprobar el chaincode por parte de la organizacion
Como parte de un sistema de administracion descentralizado de los contratos inteligentes la organizacion (Org1) a la que hace parte el peer0 debe dar su aprobacion (mensaje) antes de que el chaincode se pueda incorporar dentro del canal. Esta aprobacion junto con las demas que se requieran de acuerdo a la politica del canal de comunicacion permitira la utilizacion de esta version del contrato inteligente a los largo de la red.
Para abrobar el contrato actual se debe utilizar la siguiente instruccion,

```
peer lifecycle chaincode approveformyorg \
    -o localhost:7050 \
    --ordererTLSHostnameOverride orderer.example.com \
    --channelID mychannel \
    --name marbles \
    --version 1.0 \
    --init-required \
    --signature-policy OR"('Org1MSP.member')" \
    --sequence 1 \
    --package-id $PACKAGE_ID \
    --tls \
    --cafile ${ORDERER_CA}
```
El resultado debe indicar que el mensaje ha sido valido. Se puede verificar si el chaincode esta listo para ser enviado (commited) al canal (es decir si ha recibido los suficientes mensajes de aprobacion, en este caso como solo hay una organizacion, que en el paso anterior dio la aprobacion entonces es de esperar que este listo) mediante la siguiente instruccion,

```
peer lifecycle chaincode checkcommitreadiness \
    --channelID mychannel \
    --name marbles \
    --version 1.0 \
    --sequence 1 \
    --output json \
    --init-required \
    --signature-policy AND"('Org1MSP.member')"
```

Deben observar un mensaje de aprobacion afirmativo de parte de la Org1.

4. Enviar el chaincode al canal utilizando el peer0 de la organizacion 1.
Actuando como el peer0 de la organizacion 1 se va a enviar el contrato inteligente para ser utilizado en el canal. Para esto hay que indicar que se va actuar con los artefactos del peer0 de la Org1, lo que requiere definir unas variables de configuracion (utilizando la fundion definida en envVar.sh),
```
parsePeerConnectionParameters 1
```
Cuyo resultado son los parametros PEER_CONN_PARMS, que se pueden consultar mediante, echo $PEER_CONN_PARMS

```
peer lifecycle chaincode commit \
    -o localhost:7050 \
    --ordererTLSHostnameOverride orderer.example.com \
    --channelID mychannel \
    --name marbles \
    --version 1.0 \
    --init-required \
    --signature-policy AND"('Org1MSP.member')" \
    --sequence 1 \
    --tls \
    --cafile ${ORDERER_CA} \
    $PEER_CONN_PARMS
```
Debe observar un mensaje indicando que el chaincode ha sido enviado (commiteed). Se puede verificar el envio mediante la instruccion,
```
peer lifecycle chaincode querycommitted \
  --channelID mychannel \
  --name marbles
```
Por ultimo, se puede invocar el chaincode utilizando la funcion 'Init' que hace un llamado y prueba la ejecucion del contrato inteligente sin generar instancias en el registro.
```
peer chaincode invoke \
        -o localhost:7050 \
        --ordererTLSHostnameOverride orderer.example.com \
        -C mychannel \
        -n marbles \
        --isInit \
        -c '{"Args":["Init"]}' \
        --tls \
        --cafile ${ORDERER_CA} \
        $PEER_CONN_PARMS
```

## 4. Metodos de Escritura y lectura del contrato inteligente

Para utilizar el chaincode debemos utilizar una serie de comandos que nos permiten escribir y leer sobre el registro compartido. 

Para llevar acabo un test que permita visualizar la interaccion con el contrato se puede utilizar el sigueinte script de ejemplo desarrollado por OrderStack

```
./testCCSingleOrg.sh
```

Adicionalmente a continuacion vamos a llevar acabo una descripcion detallada de las diferentes funcionalidades del contrato inteligente. 

Debemos fijar unas variables de configuracion para utilizar como peer0 de la Org1.
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

## 5. Utilizacion del CouchDB para consultas

Verificar que el indice definido en el momento de crear el paquete ha sido creado

```
docker logs peer0.org1.example.com  2>&1 | grep "CouchDB index"
```

Para consultar el couchDB en el navegador utilizar la sigiente instruccion (en el navegador)
```
http://localhost:5984/_utils/
```
Pueden entrar y consultar las caracteristicas del primer objeto generado en el punto 4. navegando a,
```
http://localhost:5984/_utils/#database/mychannel_marbles/marble1
```
La informacion se recupera en formato JSON.

Contar con el couchDB habilita la utilizacion de queries predefinidos en el chaincode, por ejemplo vamos a invocar uno de los queries desde la perspectiva del peer0 de la Org1,

```
export PATH=${PWD}/../../bin:${PWD}:$PATH
export FABRIC_CFG_PATH=${PWD}/../../config
# Activar conjunto de funciones para indicar el nodo que ejecuta el chaincode a traves de CLI
. ./envVar.sh
#cuando se indica 1 hace referencia a Org1
setGlobals 1
parsePeerConnectionParameters 1
```
Ahora vamos a consultar los objetos a nombre de 'jerry'

```
peer chaincode query \
        -o localhost:7050 \
        --ordererTLSHostnameOverride orderer.example.com \
        -C mychannel \
        -n marbles \
        -c '{"Args":["queryMarbles", "{\"selector\":{\"docType\":\"marble\",\"owner\":\"jerry\"}, \"use_index\":[\"_design/indexOwnerDoc\", \"indexOwner\"]}"]}' \
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
