# NETClientServerQuickStart

- [01. Explica métodos y variables de los siguientes scripts](#01-explica-metodos-y-variables-de-los-siguientes-scripts)
- [02. Explica los siguientes componentes del GameObject `NetworkManager`](#02-explica-los-siguientes-componentes-del-gameobject-networkmanager)
- [03. Explica los componentes del Prefab `Player`](#03-explica-los-componentes-del-prefab-player)

> En el prefab **Player** he desactivado el script **NetworkTransformTest.cs** ya que al ejecutar el programa, según instanciaba un jugador, aparecía la cápusula en rotando sin responder a ningún botón. Daba igual que lo pusiera modo **Server** o **Host**.
> 
> Si instanciaba otro jugador, las cápsulas tenían la misma posición y movimiento, no pudiendo diferenciarlas.

## 01. Explica métodos y variables de los siguientes scripts

### Script `_HelloWorldManager.cs_`

Clase que hereda de _MonoBehaviour_. Su ojetivo es:

- Añadir un menú UI para arrancar el **NetworkManager** como:
  - **Host:** Inicia el servidor y se une como cliente instanciado un jugador
  - **Client:** Se une al servidor como jugador cliente
  - **Server:** El servidor inicia el juego sin instanciar un jugador
- Muestra las etiquetas de estado
- Lanza una acción que llama al método _Move()_ de los jugadores

#### Propiedad

- **m_NetworkManager:** Almacena la referencia del componente **NetworkManager**.

#### Método _Awake()_

Obtiene el componente **NetworkManager** del GameObject cuando el objeto se inicializa.

#### Método _OnGUI()_

Método para mostrar los botones y etiquetas la en pantalla

- Si el componente **NetworkManager** no es cliente o servidor (no se ha inicializado), llama al método _StartButtons()_ para mostrar los botones Host/Client/Server.
- En caso contrario, ejecuta los métodos _StatusLabels()_ (mostrar el estado) y después _SubmitNewPosition()_ (mostrar el botón de movimiento).

#### Método _StartButtons()_

Dibuja tres botones y si se pulsan, arranca el **NetworkManager** en el modo correspondiente:

- **Host:** Inicia el servidor y se conecta como cliente local
- **Client:** Se conecta al servidor como cliente
- **Server:** Inicia el servidor sin jugador local

#### Método _StatusLabels()_

Muestra información en la pantalla para distinguir instancias:

- Si el componente **NetworkManager** está como Host: “Host”
- Si el componente **NetworkManager** está como Client: “Client”
- Si el componente **NetworkManager** está como Server: “Server”

También añade el texto con el tipo de capa de transporte.

#### Método _SubmitNewPosition()_

Muestra un botón que cambia según el tipo de **NetworkManager** y mueve al jugador:

- **Si es servidor:** El botón pone “Move”
- **Si no es servidor:** Pone “Request Position Change”

Al pulsarlo, ejecuta la acción en dos maneras:

- **Si es un servidor y no es un cliente:**
  - Recorre todos los clientes conectados
  - Obtiene el **PlayerNetworkObject** por su UID
  - Busca el componente
  - Llama al método _Move()_ de cada jugador

- En caso contrario, obtiene el Player local
  - Obtiene el componente **HelloWorldPlayer**
  - Llama al método _Move()_ del jugador local

### Script `_HelloWorldPlayer.cs_`

Clase que hereda de _NetworkBehaviour_. Representa al jugador y sincroniza su posición entre el servidor y el cliente.

Gestiona los movimientos del jugador.

#### Propiedad _NetworkVariable_

Variable sincronizada por red que almacena la posición del jugador.

#### Método _OnNetworkSpawn()_

Se ejecuta cuando el objeto de red aparece en una instancia cliente/servidor. Si el cliente es el dueño del objeto (_IsOwner_), llama al método _Move()_.

#### Método _Move()_

Método público que llama al método _SubmitPositionRequestRpc()_ a través del RPC.

#### Método _SubmitPositionRequestRpc()_

Envía una petición la servidor mediante un RPC (_SendTo.Server_). El servidor genera una posición aleatoria y actualiza el **transform.position** en el servidor y después actualiza la variable de red **Position**.

#### Método _GetRandomPositionOnPlane()_

Devuelve una posición en un Vector3 aleatoria.

#### Método _Update()_

Actualiza la posición con el valor de la propiedad **Position**.

### Script `NetworkTransformTest.cs`

Clase que hereda de _NetworkBehaviour_. Se utiliza como movimiento controlado únicamente por el servidor. Mueve el objeto en forma circular.

#### Método _Update()_

Si la propiedad es el servidor (_IsServer_), mueve el objeto en forma circular.

### Script `_RPCTest.cs_`

Clase que hereda de _NetworkBehaviour_ que se utiliza de RPC.


#### Método _OnNetworkSpawn()_

Se ejecuta cuando el **NetworkObject** aparece (spawn) en una instancia. Asegura que se ejecute solo si el cliente es el dueño del objeto e inicia la cadena enviando al servidor.

#### Método _ClientAndHostRpc()_

Se ejecuta en todos los clientes y también en el host. Escribe mensajes en la consola en todos y solo el propietario (owner) responde de vuelta al servidor incrementando el _value_.

### Método _ServerOnlyRpc()_

Se invoca desde un cliente y se ejecuta en el servidor. El servidor escribe un mensaje en la consola y envía a los clientes el _value_ incrementado en 1, y el UID del NetworkObject.

## 02. Explica los siguientes componentes del GameObject `NetworkManager`

### NetworkManager

Componente principal de **Netcode for GameObjects** que gestiona el ciclo de vida del juego en red:

- Centraliza el control del estado de red (cliente/servidor/host)
- Gestiona la conexión y desconexión de clientes
- Gestiona la creación y destrucción de jugadores
- Coordina la sincronización de escenas entre servidor y clientes
- Mantiene la configuración  red

### Unity Transport

Componente de transporte de bajo nivel:

- Basada en UDP
- Se encarga del envío y recepción de paquetes entre clientes y servidor
- Independiente de la lógica del juego (solo transporta datos).

### Hellow World Manager

Script personalizado que proporciona una interfaz gráfica para controlar el **NetworkManager**.

- Permite iniciar la red como Host, Client o Server
- Muestra información del estado de red y del transporte utilizado
- Facilita pruebas rápidas del flujo cliente-servidor

## 03. Explica los componentes del Prefab `Player`

### Network Object

Componente que identifica un GameObject como parte de una sesión multijugador:

- Asigna un ID único de red al objeto.
- Permite que el servidor gestione su creación (spawn) y destrucción (despawn).
- Controla la propiedad (ownership) del objeto.
- Es imprescindible para que el objeto pueda sincronizarse entre servidor y clientes.

### Hello World Player

Script que hereda de **NetworkBehaviour** y gestiona el comportamiento del jugador en red.

- Controla la posición del jugador mediante una **NetworkVariable**.
- Solicita cambios de posición al servidor usando RPCs.
- Garantiza que el servidor tenga la autoridad sobre el movimiento.
- Actualiza la posición local del GameObject a partir del estado sincronizado.

### Rpc Test

Script de prueba que demuestra el uso de **RPCs** en **Netcode for GameObjects**.

- Permite la comunicación directa cliente → servidor y servidor → clientes.
- Solo el cliente propietario (owner) del objeto puede iniciar el envío de RPCs.
- El servidor recibe las peticiones y las redistribuye a todos los clientes.
- Se utiliza para comprender el flujo de mensajes y la autoridad en red, no como lógica de juego final.

### Network Transform

Componente encargado de sincronizar automáticamente las transformaciones del GameObject:

- Posición
- Rotación
- Escala
- Replica los cambios realizados por el servidor al resto de clientes.
- Evita la necesidad de usar **NetworkVariable** para transformaciones simples.
- Habitual en objetos con movimiento continuo controlado por el servidor.