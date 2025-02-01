# Warnet: La Batalla de Galen Erso

![vulnerabilidad](./images/vulnerability.gif)

Tu misión es atacar nodos de Bitcoin Core en una red privada que se ejecuta en un clúster de Kubernetes. La red privada consiste en nodos de Bitcoin Core que son vulnerables a ataques históricos completamente divulgados o fallos intencionales novedosos. Un sitio web **FALSO** con publicaciones de blog sobre todo tipo de vulnerabilidades disponibles para explotar en Warnet se puede ver aquí:

https://bitcorncore.org/en/blog/

⚠️ Este sitio web es solo para fines de entretenimiento ⚠️

## Terminología

- Tanques (`Tanks`): Nodos de Bitcoin Core ejecutándose en una red de Warnet
- Campo de batalla (`Battlefield`): Un clúster remoto con 100 tanques vulnerables
  - cadena de signet personalizada con un minero administrado por el administrador
- Escaramuza (`Scrimmage`): Un clúster local con solo unos pocos tanques vulnerables
  - cadena de signet personalizada con el desafío OP_TRUE para que cualquiera pueda generar bloques (como regtest)
- Armada: Un pequeño conjunto de tanques que ejecutan la última versión de Bitcoin Core bajo el control del atacante
- Escenario: Un programa que se despliega en el campo de batalla para atacar los tanques

## Objetivos

1. ¡Clona este repositorio!
2. Instala y configura Warnet
3. Crea ataques
4. Prueba ataques localmente en `Scrimmage`
5. Ataca nodos de Bitcoin Core en el campo de batalla

## Informe de Inteligencia -- ¿Qué es Warnet?

Warnet es un sistema escrito en Python para desplegar, administrar e interactuar con redes p2p de Bitcoin dentro de un clúster de Kubernetes. El campo de batalla oficial será un clúster remoto con más de 100 nodos de Bitcoin (referidos como "Tanques") ejecutándose en una cadena de signet personalizada (donde solo el administrador de la red puede generar bloques). Muchos de estos nodos serán versiones antiguas de Bitcoin Core con [vulnerabilidades públicamente divulgadas](https://bitcoincore.org/en/blog/). También habrá nodos adicionales que se han compilado con fallos intencionales y [divulgaciones FALSAS](https://bitcorncore.org/en/blog/)

Para ayudar a facilitar las estrategias de ataque de tanques en el campo de batalla, un red más pequeña de 12 nodos llamada `Scrimmage` se puede ejecutar localmente por los atacantes mientras desarrollan escenarios. `Scrimmage` requiere ejecutar kubernetes localmente (ya sea Docker Desktop o minikube), lo cual no es necesario para ejecutar ataques en el campo de batalla remoto. `Scrimmage` también se ejecuta en una cadena de signet con un desafío de `OP_TRUE` para que cualquier nodo pueda generar bloques.

### Instalar Warnet

La documentación para Warnet está disponible en el repositorio.

Instala en un entorno virtual de Python:

https://github.com/bitcoin-dev-project/warnet/blob/main/docs/install.md#install-warnet

### Configurar Warnet

Warnet te guiará a través del proceso de configuración.

> [!TIP]
> **Hay varias opciones para elegir cuidadosamente al configurar Warnet!**
> - Solo necesitas instalar minikube o kubernetes de Docker Desktop si planeas ejecutar la red de `Scrimmage` localmente para experimentación y desarrollo.
> - Acceder al campo de batalla (`Battlefield`) de 100 nodos signet remoto no requiere una distribución local de kubernetes, pero aún requerirá la instalación de `kubectl` y `helm`.
> - El asistente `warnet setup` instalará estas dependencias por ti.
> - Si ejecutas Docker Desktop, sé generoso con la cantidad de recursos que le asignas.

#### Una vez instalado Warnet, ejecuta `warnet setup`

Ejemplo:

```
(.venv) $ warnet setup

                 ╭───────────────────────────────────╮
                 │  Welcome to Warnet setup          │
                 ╰───────────────────────────────────╯

    Let's find out if your system has what it takes to run Warnet...

[?] Which platform would you like to use?:
 > Minikube
   Docker Desktop
   No Backend (Interacting with remote cluster, see `warnet auth --help`)
```

### Una vez configurado Warnet en un entorno virtual, ¡clona este repo!

```
(.venv) $ git clone https://github.com/bitcoin-dev-project/battle-of-galen-erso
(.venv) $ cd battle-of-galen-erso/
```

### Herramientas adicionales

#### K9s

Ya sea que ejecutes Kubernetes localmente o uses el clúster remoto, recomendamos la interfaz de usuario de terminal [k9s](https://github.com/derailed/k9s) para monitorear el estado del clúster.

![captura de pantalla de k9s](./images/k9s-screenshot.png)

#### ktop

Si deseas observar el uso de recursos en un clúster con métricas habilitadas, puedes considerar usar [ktop](https://github.com/vladimirvivien/ktop)

![captura de pantalla de ktop](./images/ktop-screenshot.png)

## Operaciones de Red

### Iniciar y Detener la Red

> [!TIP]
> **Esta sección solo es relevante en `Scrimmage` (estás ejecutando kubernetes localmente)**

Puedes ver la topología de la red que se desplegará y hacer modificaciones viéndolo en: `networks/scrimmage/network.yaml`
Esto también te permitirá ver qué tanques están ejecutando qué versión de Bitcoin Core.

Despliega la red de `Scrimmage` de 12 nodos incluida en este repositorio en un clúster local de Kubernetes con el comando:

```
warnet deploy ./networks/scrimmage
```

La red local se puede apagar con el comando:

```
warnet down
```

### Reconocimiento de Red

> [!TIP]
> **Esta sección solo es relevante en `Scrimmage` (estás ejecutando kubernetes localmente)**

Puedes abrir el visualizador basado en web con paneles de Grafana y Fork Observer ejecutando el comando:

```
warnet dashboard
```

Warnet obtendrá el puerto `localhost` del servidor web del panel y
lo abrirá en el navegador predeterminado de tu sistema.

![captura de pantalla de fork observer](./images/fo.png)

### Comunicaciones de Red

> [!TIP]
> **En `Battlefield`, estos comandos solo podrán recuperar datos de tanques en tu armada. En el modo de `Scrimmage` local, tendrás acceso a todos los tanques.**

Consulta la [documentación de Warnet](https://github.com/bitcoin-dev-project/warnet/blob/main/docs/warnet.md) para todos los comandos CLI disponibles para recuperar registros, mensajes p2p y otra información de estado.

#### Ejemplos:
#### Estado

```
(.venv) --> warnet status
╭────────────────────── Warnet Overview ───────────────────────╮
│                                                              │
│                        Warnet Status                         │
│ ┏━━━━━━━━━━━┳━━━━━━━━━━━━━━━━━━━━━┳━━━━━━━━━┳━━━━━━━━━━━━━━┓ │
│ ┃ Component ┃ Name                ┃ Status  ┃ Namespace    ┃ │
│ ┡━━━━━━━━━━━╇━━━━━━━━━━━━━━━━━━━━━╇━━━━━━━━━╇━━━━━━━━━━━━━━┩ │
│ │ Tank      │ armada-0            │ running │ wargames-red │ │
│ │ Tank      │ armada-1            │ running │ wargames-red │ │
│ │ Tank      │ armada-2            │ running │ wargames-red │ │
│ │ Scenario  │ No active scenarios │         │              │ │
│ └───────────┴─────────────────────┴─────────┴──────────────┘ │
│                                                              │
╰──────────────────────────────────────────────────────────────╯

Total Tanks: 3 | Active Scenarios: 0
Network connected
```

#### bitcoin-cli
```
(.venv) --> warnet bitcoin rpc armada-0 -getinfo
Chain: signet
Blocks: 0
Headers: 0
Verification progress: 100.0000%
Difficulty: 4.656542373906925e-10

Network: in 0, out 1, total 1
Version: 270000
Time offset (s): 0
Proxies: n/a
Min tx relay fee rate (BTC/kvB): 0.00001000

Warnings: (none)
```

## Armamento

### Desarrollo de Ataques

El método principal de interactuar con la red y montar un ataque es desplegando 
un escenario.

Un escenario es un script de Python escrito con la misma estructura y biblioteca que
una prueba funcional de Bitcoin Core, utilizando una copia del `test_framework`, 
que está [incluida](/scenarios/test_framework) en este repositorio y puede ser 
modificada si es necesario.
La principal diferencia es que la lista familiar `self.nodes[]` contiene referencias 
a nodos de Bitcoin Core en contenedores ejecutándose dentro del clúster en lugar de 
procesos bitcoind localmente accesibles.

También está disponible una lista adicional `self.tanks[str]` para abordar nodos de 
Bitcoin por su nombre de pod de Kubernetes (en lugar de su índice numérico).

Los objetos en `self.nodes[]` y `self.tanks[]` son objetos proxy de RPC que interpretan 
todas las llamadas como comandos RPC que se reenvían al nodo de bitcoin core.


Ejemplo:
```python
self.tanks["armada-0"].getpeerinfo()
```

Si no estás familiarizado con la interfaz RPC de Bitcoin Core, puedes obtener ayuda 
directamente de un nodo de Bitcoin Core:

```
(.venv) $ warnet bitcoin rpc armada-0 help
```

También hay varios recursos en línea para aprender sobre los comandos RPC disponibles:

https://chainquery.com/bitcoin-cli

https://developer.bitcoin.org/reference/rpc/


**Los únicos tanques a los que como atacante tienes acceso RPC están en tu propia armada**

Se incluyen varios escenarios de ejemplo en el directorio [`scenarios/`](/scenarios/).
En particular, [`scenarios/reconnaissance.py`](/scenarios/reconnaissance.py) está escrito 
con comentarios detallados para demostrar cómo ejecutar comandos RPC en nodos disponibles, 
así como cómo utilizar la clase `P2PInterface` del framework para enviar mensajes arbitrarios a nodos objetivo.

Los tanques en la red de kubernetes tienen URI que incluyen un espacio de nombres. Los URI de todos los tanques se pueden ver en el campo "descripción" en la interfaz web de fork-observer.

Ejemplo:
```python
attacker = P2PInterface()
attacker.peer_connect(
    dstaddr=socket.gethostbyname("tank-0000-red.default.svc"),
    dstport=38333,
    net="signet",
    timeout_factor=1
)()
attacker.wait_until(lambda: attacker.is_connected, check_connected=False)

# Create a malicious p2p packet and send...
```

### Despliegue de Ataques

Para crear un ataque, modifica los archivos existentes en `scenarios/` o crea nuevos y 
despliégalos en la red. La bandera `--debug` imprimirá la salida del registro 
del escenario en la terminal y eliminará el contenedor cuando termine de ejecutarse, 
ya sea por éxito, fallo o interrupción por `ctrl-C`

Ejemplo:

```
(.venv) --> warnet run scenarios/reconnaissance.py --debug
...
Successfully deployed scenario commander: reconnaissance
Commander pod name: commander-reconnaissance-1727792531
initContainer in pod commander-reconnaissance-1727792531 is ready
Successfully copied data to commander-reconnaissance-1727792531(init):/shared/warnet.json
Successfully copied data to commander-reconnaissance-1727792531(init):/shared/archive.pyz
Successfully uploaded scenario data to commander: reconnaissance
Waiting for commander pod to start...
Reconnaissance Adding TestNode #0 from pod tank-0000 with IP 10.1.38.68
Reconnaissance Adding TestNode #1 from pod tank-0001 with IP 10.1.38.69
Reconnaissance Adding TestNode #2 from pod tank-0002 with IP 10.1.38.70
Reconnaissance Adding TestNode #3 from pod tank-0003 with IP 10.1.38.71
Reconnaissance Adding TestNode #4 from pod tank-0004 with IP 10.1.38.72
Reconnaissance Adding TestNode #5 from pod tank-0005 with IP 10.1.38.73
Reconnaissance Adding TestNode #6 from pod tank-0006 with IP 10.1.38.74
Reconnaissance Adding TestNode #7 from pod tank-0007 with IP 10.1.38.75
Reconnaissance Adding TestNode #8 from pod tank-0008 with IP 10.1.38.76
Reconnaissance Adding TestNode #9 from pod tank-0009 with IP 10.1.38.77
Reconnaissance Adding TestNode #10 from pod tank-0010 with IP 10.1.38.78
Reconnaissance Adding TestNode #11 from pod tank-0011 with IP 10.1.38.79
Reconnaissance User supplied random seed 131260415370612
Reconnaissance PRNG seed is: 131260415370612
Reconnaissance Getting peer info
Reconnaissance tank-0001 /Satoshi:27.0.0/
Reconnaissance tank-0002 /Satoshi:27.0.0/
Reconnaissance tank-0003 /Satoshi:27.0.0/
Reconnaissance tank-0005 /Satoshi:25.1.0/
Reconnaissance 10.1.38.75:59622 /Satoshi:0.21.1/
Reconnaissance tank-0006 /Satoshi:24.2.0/
Reconnaissance tank-0004 /Satoshi:26.0.0/
Reconnaissance 10.1.38.79:36980 /Satoshi:0.16.1/
Reconnaissance 10.1.38.76:40020 /Satoshi:0.20.0/
Reconnaissance tank-0010 /Satoshi:0.17.0/
Reconnaissance tank-0011 /Satoshi:0.16.1/
Reconnaissance Attacking 10.111.179.151:18444
Reconnaissance Got notfound message from 10.111.179.151:18444
Reconnaissance Stopping nodes
Reconnaissance Cleaning up /tmp/bitcoin_func_test_4zh_53n0 on exit
Reconnaissance Tests successful
Deleting pod...
pod "commander-reconnaissance-1727792531" deleted

```

También puedes consultar el escenario con `-- --help` para conocer sus argumentos.

Ejemplo:

```
(.venv) --> warnet run scenarios/miner_std.py -- --help
usage: warnet run /path/to/miner_std.py [options]

Generate blocks over time

options:
  -h, --help           show this help message and exit
  --allnodes           When true, generate blocks from all nodes instead of just nodes[0]
  --interval INTERVAL  Number of seconds between block generation (default 60 seconds)
  --mature             When true, generate 101 blocks ONCE per miner
  --tank TANK          Select one tank by name as the only miner
```

## Reglas de Compromiso

Un ataque se considera exitoso cuando un nodo objetivo ya no está sincronizado con la cadena 
de mayor trabajo. Esto podría ser el resultado de:
- Un ataque de eclipse
- Un error de memoria que mata el proceso del nodo (los nodos están programados para no reiniciar)
- Una denegación de servicio de CPU que impide que el nodo verifique nuevos bloques
Los atacantes no podrán generar sus propios bloques en la cadena de signet del campo de
batalla, pero tienen permiso ilimitado en su red de signet de `Scrimmage` local.

## Fondos

En el campo de batalla, solo el administrador puede generar bloques. Periódicamente ejecutará 
un script que financia los nodos armada de todos con el saldo gastable que tenga el minero. 
Ese script asegurará que cada nodo armada tenga una billetera llamada "miner".

> [!TIP]
> ¡Tus nodos armada tendrán una billetera llamada "miner" con muchos BTC de prueba!


## PISTAS

💡`Scrimmage` es una cadena de signet, lo que significa que aunque el objetivo de 
dificultad sea el mínimo, el proof of work aún importa. Por esa razón, generar 
bloques puede requerir más de un intento.

Prueba: `warnet run scenarios/miner_std.py --tank=miner --interval=1 --debug`

💡 Recuerda que los subsidios de bloque solo serán gastables después de 100 bloques y 
mina algunos bloques para sacar al nodo de IBD. Los nodos en `Scrimmage` no aceptarán 
transacciones hasta que estén fuera de IBD.

💡 Puede que no estés familiarizado con el marco de pruebas funcionales de Bitcoin Core. 
Para obtener algunas pistas sobre su uso en escenarios p2p, ¡revisa algunas de las pruebas 
existentes!

Ejemplos:
- [envía mensajes `addr`](https://github.com/bitcoin/bitcoin/blob/28.x/test/functional/p2p_addrfetch.py)
- [crea transacciones huérfanas](https://github.com/bitcoin/bitcoin/blob/28.x/test/functional/p2p_orphan_handling.py)
- [envía bloques no válidos](https://github.com/bitcoin/bitcoin/blob/28.x/test/functional/p2p_invalid_block.py)

También mira los escenarios de ejemplo y de ejemplo en el directorio de `scenarios` para obtener inspiración.

## Corre tu primer ataque

Intentaremos derribar el nodo 5k-inv. Para averiguar cuál es ese nodo, podemos usar forkobserver utilizando `warnet dashboard` si estamos ejecutando Scrimmage localmente o usando la URL de forkobserver proporcionada por los administradores para tu Battlefield.

En fork-observer, el campo "description" mostrará qué versión de bitcoin se está ejecutando. Encuentra el nodo 5k-inv y actualiza `scenarios/my_first_attack_5kinv.py` con el nombre de este nodo. Busca una variable llamada `victim`. En Battlefield te habrán asignado un color para atacar y localmente en Scrimmage solo habrá rojo. El enlace que revela este ataque en particular se puede encontrar:
[5K Inv Disclosure](https://bitcorncore.org/en/2024/10/23/fake-disclosure-5kinv/).

Desde la raíz de este repositorio, ejecuta este escenario con:

`warnet run scenarios/my_first_attack_5kinv.py --debug`

Después de enviar los 5000 INVs, deberías observar que este nodo se vuelve no responsivo en fork-observer.

## En El Campo de Batalla

Cuando estés listo para lanzar tu ataque de verdad, comienza por "cambiar el contexto"
de tu clúster local al clúster remoto. Tendrás un archivo de configuración
proporcionado por el administrador:

```
warnet auth /path/to/battlefield-100-large-kubeconfig.yaml
```

### Cambiar el contexto de nuevo a local / de vuelta al campo de batalla

Para ver los contextos disponibles, ejecuta:

```shell
kubectl config get-contexts
```

Para local, el contexto que probablemente estás buscando sería `docker-desktop` o `minikube`. Para el Campo de Batalla, será un nombre más largo.

Cambia a ese contexto usando:

```shell
kubectl config use-context <context name>
```

Los comandos de Warnet se ejecutarán contra el contexto que esté actualmente configurado en tu kubeconfig (como se muestra usando kubectl)

¡Buena suerte!
