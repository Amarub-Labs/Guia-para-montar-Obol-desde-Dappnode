# obol-dvt-dappnode-guide

> Guía completa para configurar un cluster de **Obol Distributed Validator (DVT)** con **4 operadores usando DappNode**, incluyendo la ceremonia DKG paso a paso y configuración con **Lido Community Staking Module (CSM)**.

[![License](https://img.shields.io/badge/license-MIT-blue.svg)](LICENSE)
[![Red](https://img.shields.io/badge/Ethereum-Mainnet%20%7C%20Hoodi-gray.svg)](https://ethereum.org)
[![Obol](https://img.shields.io/badge/Obol-Charon-green.svg)](https://obol.org)
[![DappNode](https://img.shields.io/badge/DappNode-obol%20generic-purple.svg)](https://dappnode.com)
[![Lido](https://img.shields.io/badge/Lido-CSM-blue.svg)](https://lido.fi)

---

## Tabla de contenido

- [Contexto](#contexto)
- [1. Prerequisitos en DappNode](#1-prerequisitos-en-dappnode)
  - [1.1 Clientes de ejecución y consenso](#11-clientes-de-ejecución-y-consenso-sincronizados)
  - [1.2 Wallet configurada](#12-wallet-configurada)
  - [1.3 Instalación del paquete Obol](#13-instalación-del-paquete-obol-desde-el-dappstore)
- [2. Obtención de los ENR](#2-obtención-de-los-enr-desde-dappnode)
- [3. Creación del cluster en el Launchpad](#3-creación-del-cluster-en-el-obol-dv-launchpad)
- [4. Aceptación del cluster por los operadores](#4-aceptación-del-cluster-por-los-otros-operadores)
- [5. Carga del Definition File en DappNode](#5-carga-del-definition-file-en-dappnode)
- [6. Ejecución del DKG](#6-ejecución-del-dkg-desde-dappnode)
- [7. Depósito y activación de validadores](#7-depósito-y-activación-de-validadores)
- [8. Checks finales y buenas prácticas](#8-checks-finales-y-buenas-prácticas-en-dappnode)
- [Resumen del flujo completo](#resumen-del-flujo-completo)
- [Referencias](#referencias)
- [Lido CSM con Obol DVT](#lido-csm-con-obol-dvt)
  - [¿Qué es Lido CSM?](#qué-es-lido-csm)
  - [Ventajas de CSM + Obol DVT](#ventajas-de-csm--obol-dvt)
  - [Flujo simplificado CSM + Obol en DappNode](#flujo-simplificado-csm--obol-en-dappnode)
  - [Diferencias clave vs setup DVT estándar](#diferencias-clave-vs-setup-dvt-estándar)
  - [Recursos oficiales CSM](#recursos-oficiales-csm)

---

## Contexto

> **Nota sobre testnets:** Hoodi es la testnet activa de Ethereum para staking desde marzo de 2025, reemplazando a Holesky. El paquete Obol en DappNode (`obol-generic`) ya soporta Hoodi. Algunos enlaces en la documentación oficial aún referencian "Holesky" porque no han sido actualizados, pero el flujo técnico es idéntico.

Un Distributed Validator (DV) es un validator lógico de Ethereum cuya clave privada se divide entre varios nodos que operan simultáneamente. Cada nodo controla solo una fracción (key share) de esa clave. Para firmar un mensaje, se necesitan al menos **3 de los 4 nodos** activos (umbral 3-de-4). Esto significa que el cluster puede tolerar la caída de hasta 1 nodo sin perder eficiencia de validación.

En un cluster de 4 operadores, cada nodo ejecuta:

| Componente | Ejemplos |
|---|---|
| Execution Client | Geth, Nethermind, Besu, Erigon |
| Consensus Client | Lodestar, Lighthouse, Prysm, Teku, Nimbus |
| Charon (middleware DV) | Gestionado por el paquete Obol |
| Validator Client | Lodestar (integrado en el paquete Obol de DappNode) |

---

## 1. Prerequisitos en DappNode

Cada uno de los 4 operadores debe cumplir estos requisitos **antes** de proceder con la configuración del cluster.

### 1.1 Clientes de ejecución y consenso sincronizados

Accede a tu DappNode desde `http://my.dappnode` y navega a la pestaña **Stakers** del panel lateral. Selecciona un Execution Client y un Consensus Client, y aplica los cambios. El proceso de sincronización puede tomar varias horas dependiendo de la red y la conexión.

Una vez que la sincronización esté completa, el Dashboard de DappNode mostrará ambos clientes en estado activo y sincronizados. **No instales el paquete Obol hasta que ambos clientes estén completamente sincronizados.**

### 1.2 Wallet configurada

Cada operador necesita una wallet de Ethereum (compatible con MetaMask) que pueda firmar mensajes. Esta dirección se usará en el Obol DV Launchpad para autenticar la participación en el cluster. Asegúrate de tener acceso a los fondos necesarios en la wallet del operador que realizará el depósito final (32 ETH por validator).

### 1.3 Instalación del paquete Obol desde el DappStore

El paquete actual es **obol-generic**, que soporta múltiples redes desde una sola instancia: Mainnet, Hoodi y Gnosis. Es un paquete único; no hay versiones separadas por red como en versiones anteriores.

1. En tu DappNode, ve a **Packages > DappStore** y selecciona la pestaña **Public**.
2. Busca **"Obol"** en el DappStore. El paquete correcto se llama **Obol** (basado en `obol-generic`). Es posible que también veas un paquete antiguo llamado "Holesky Obol" de versiones previas; si lo ves, ignórálo y usa el actual.
3. En la pantalla de instalación, expande **Advanced Options** y activa **"Bypass only signed safe restriction"**.
4. En la pantalla del **Setup Wizard**, selecciona **"New Cluster"** y confirma. Durante esta primera instalación, deja todos los campos de Definition File en blanco; el objetivo es únicamente generar las ENR iniciales.
5. Acepta los términos y condiciones. El paquete se instalará y quedará visible en la pestaña **Packages**.

> **Nota:** El paquete Obol soporta hasta **5 ENR simultáneas**, lo que permite participar en hasta 5 clusters diferentes desde el mismo DappNode.

---

## 2. Obtención de los ENR desde DappNode

El ENR (Ethereum Node Record) es un identificador público que permite a los clientes Charon de otros operadores encontrar y conectarse a tu nodo durante el DKG y en la operación continua del cluster.

### Cómo obtener tu ENR

1. En tu DappNode, ve a **Packages** y selecciona el paquete **Obol**.
2. Navega a la pestaña **Info**.
3. Verás una lista de ENR pre-generadas (hasta 5). Cada una corresponde a un slot de cluster diferente (Cluster-1, Cluster-2, etc.).
4. Selecciona una ENR que **no esté actualmente en uso** por otro cluster. Cópiala íntegramente.

Ejemplo de formato ENR:

```
enr:-JG4QGQpV4qYe32QFUAbY1UyGNtNcrVMip83cvJRhw1brMslPeyELIz3q6dsZ7GblVaCjL_8FKQhF...
```

### Intercambio de ENR entre operadores

Los 4 operadores deben compartir sus ENR entre sí por un medio seguro (grupo privado, videollamada, plataforma cifrada). El operador que cree el cluster (el **Creator**) necesitará las ENR de todos los participantes para configurar el cluster en el Launchpad.

> ⚠️ Una ENR es un identificador público, no una clave secreta. Sin embargo, la **clave privada asociada** (almacenada internamente en el paquete Obol de DappNode) **nunca debe compartirse**. Si necesitas hacer backup, usa la función de backup integrada en el paquete.

---

## 3. Creación del cluster en el Obol DV Launchpad

Una persona del grupo actúa como **Creator** (este rol no tiene privilegios especiales dentro del cluster, solo coordina la configuración inicial). Los otros 3 actúan como **Operators**.

### 3.1 Conectar wallet y seleccionar rol

1. Abre el **Obol DV Launchpad**:
   - Mainnet: `https://launchpad.obol.org/`
   - Hoodi Testnet: `https://hoodi.launchpad.obol.org/`
2. Conecta tu wallet (MetaMask).
3. Selecciona **"Create a Cluster with a group"** → **"Get Started"**.
4. Acepta los avisos y términos del Launchpad.

### 3.2 Configurar el cluster

El Creator introduce la siguiente información:

| Campo | Valor / Descripción |
|---|---|
| Cluster Name | Nombre descriptivo (ej. "Mi Cluster DVT") |
| Cluster Size | **4** operadores. El umbral se calcula automáticamente: **3 de 4** |
| Direcciones operadores | Las 4 direcciones Ethereum. Usar "Use My Address" para la posición 1 si el Creator es operador |
| Número de validadores | Cantidad de validators distribuidos (32 ETH cada uno). 1 es suficiente para validar el proceso |
| ENR del Creator | Su ENR copiada desde DappNode (si es operador) |
| Principal Address | Dirección que recibe los 32 ETH y recompensas de consensus al retirar el validator. Puede ser multisig/splitter |
| Fee Recipient Address | Dirección que recibe recompensas de ejecución (MEV, tips). Puede ser la misma o diferente a la anterior |

### 3.3 Firmar y generar el enlace de invitación

1. Haz clic en **"Create Cluster Configuration"**.
2. Revisa todos los detalles en el resumen.
3. Firma con tu wallet. El Launchpad solicita firmar 2-3 transacciones:
   - **config_hash:** representación hashada de la configuración del cluster, garantizando que todos los operadores acuerdan un setup idéntico.
   - **operator_config_hash:** aceptación de los términos de participación como operador.
   - **ENR** (si eres operador): autoriza a la clave privada correspondiente a actuar en el cluster.
4. Una vez firmado, el Launchpad genera un **enlace de invitación** para el cluster.

### 3.4 Compartir el enlace

El Creator comparte el enlace de invitación con los otros 3 operadores por un medio seguro. Este enlace permite que cada operador revise la configuración, envíe su ENR y firme su participación.

---

## 4. Aceptación del cluster por los otros operadores

Cada uno de los 3 operadores que no es el Creator debe:

1. Abrir el enlace de invitación recibido del Creator. **Verificar que el dominio sea legítimo** (`launchpad.obol.org` o `hoodi.launchpad.obol.org`) para evitar phishing.
2. Conectar su wallet usando la dirección Ethereum que el Creator registró.
3. Revisar las direcciones de los operadores y hacer clic en **"Get Started"**.
4. Aceptar los términos y condiciones del Launchpad.
5. Revisar la configuración del cluster e **introducir su ENR** (copiada desde la pestaña Info del paquete Obol en su DappNode).
6. Firmar las transacciones solicitadas:
   - **config_hash** (confirmación de configuración).
   - **ENR** (autorización de participación).
7. Esperar a que los otros operadores completen el mismo proceso.

Una vez que los 4 operadores han firmado, el Launchpad avanza automáticamente a la siguiente pantalla.

---

## 5. Carga del Definition File en DappNode

Cuando todos los operadores han firmado exitosamente, el Launchpad genera el **Definition File URL**.

### 5.1 Obtener la Definition File URL

1. En el Launchpad, navega a la pestaña **"DappNode"** (o **"Dappnode/Avado"**) en la pantalla de preparación para el DKG.
2. Copia la **URL completa** mostrada en esa pestaña.

### 5.2 Introducir la URL en DappNode

Cada uno de los 4 operadores debe completar estos pasos en su propio DappNode:

1. Navegar al paquete **Obol** → pestaña **Config**.
2. En el menú despregable **"Config Mode"**, seleccionar **"URL"**.
3. Pegar la **Definition File URL** en el campo del cluster que corresponde a la ENR usada en el Launchpad.

   > **Importante:** Si usaste la ENR del slot **ENR1**, pega la URL en **Cluster-1**. Si usaste **ENR2**, en **Cluster-2**, y así sucesivamente. Un desajuste aquí bloquea el DKG.

4. Hacer clic en **"Update"** para guardar la configuración.

### 5.3 Reiniciar el contenedor Charon

1. Volver a la pestaña **Info** del paquete Obol.
2. Reiniciar el contenedor Charon correspondiente al cluster configurado.

> **Verificación previa al DKG:** Los 4 DappNodes deben tener la misma Definition File URL cargada en el slot correcto, y los 4 contenedores Charon deben estar reiniciados. Si algún operador no ha completado este paso, el DKG no podrá iniciarse.

---

## 6. Ejecución del DKG desde DappNode

### 6.1 Inicio automático

Una vez que los 4 nodos Charon están activos y cada uno tiene la Definition File URL cargada, el proceso DKG **se inicia automáticamente**. No es necesario ejecutar comandos manuales ni intervenir desde la línea de comandos.

El flujo interno es: los 4 clientes Charon se conectan entre sí, verifican que todos comparten el mismo **cluster_definition_hash**, y una vez que el handshake es exitoso en los 4 nodos, la ceremonia de generación de claves comienza.

### 6.2 Monitoreo del DKG

Navega a la pestaña **Logs** del paquete Obol. Los mensajes clave a buscar son:

| Mensaje en los logs | Significado |
|---|---|
| Conexión exitosa hacia los 3 peers | La comunicación p2p entre nodos funciona |
| `cluster_definition_hash` coincide en todos | Todos los participantes están de acuerdo sobre la configuración |
| `DKG ceremony in progress` | La generación de claves está activa |
| `DKG ceremony completed successfully` | ✅ El DKG fue exitoso |

Si el DKG no avanza, los problemas más comunes son: un operador no ha cargado la URL correcta, un nodo no está conectado a internet, o hay un problema de conectividad p2p (el puerto **3610** debe ser accesible desde el exterior).

### 6.3 Archivos generados

Tras el DKG exitoso, el paquete Obol almacena internamente:

| Artefacto | Descripción | ¿Único por operador? |
|---|---|---|
| `validator_keys/` | Key shares privadas y contraseñas de los validators | ✅ Sí |
| `cluster-lock.json` | Configuración que Charon necesita para operar el cluster | ❌ Idéntico en todos |
| `deposit-data.json` | Información para activar los validators en la red | ❌ Idéntico en todos |

### 6.4 Backup obligatorio

> ⚠️ **CRÍTICO:** Antes de activar cualquier validator, **todos los operadores deben realizar un backup del paquete Obol.** Si pierdas tus key shares, no podrás operar el cluster ni recuperar el depósito de 32 ETH.

1. Dentro del paquete Obol, navegar a la pestaña **Backup**.
2. Hacer clic en **"Backup now"**.
3. Se descargará un archivo `.tar` comprimido. Guardarlo en un lugar seguro y offline.
4. Para verificar, descomprimir el `.tar`. Encontrarás carpetas para cada nodo Charon (hasta 5). Dentro de la carpeta correspondiente a tu cluster estarán todos los artefactos.

**Cada operador debe completar este backup de forma independiente antes de proceder al depósito.** Las key shares son únicas por operador y no pueden recuperarse si se pierden.

---

## 7. Depósito y activación de validadores

### 7.1 Obtener el deposit-data.json

1. Descomprimir el archivo `.tar` del backup del paquete Obol.
2. Navegar a la carpeta del cluster correspondiente (ej. `charon-1/`).
3. Localizar el archivo `deposit-data.json`.

> Este archivo es idéntico en todos los operadores. Solo necesita hacerse **una sola vez** por cada validator del cluster.

### 7.2 Realizar el depósito

Solo **un operador** necesita realizar el depósito.

**Opción A — Desde el Obol DV Launchpad (recomendada):**

1. En el Launchpad, tras completar el DKG, aparece una pantalla con la opción de subir el `deposit-data.json`.
2. Conectar la wallet que contenga los fondos necesarios (32 ETH por cada validator).
3. Subir el archivo `deposit-data.json`.
4. El Launchpad verifica el saldo y presenta un resumen final.
5. Confirmar la firma de la transacción de depósito.

**Opción B — Desde el Ethereum Staking Launchpad:**

1. Ir a `https://ethereum.org/en/staking/launchpad/` (mainnet) o al launchpad de testnet correspondiente.
2. Subir el archivo `deposit-data.json` y completar el proceso estándar.

### 7.3 Tiempo de activación

La activación mínima tarda aproximadamente **16 horas**, pero el tiempo real depende de la cola de activación en la red Ethereum y puede extenderse hasta varias semanas en periodos de alta demanda.

### 7.4 Confirmación

- El paquete Obol en cada DappNode mostrará los validadores en estado **activo**.
- Es normal ver mensajes de 404 en los logs **antes** de que la activación sea confirmada por la red. Desaparecen una vez que el validator es reconocido por la capa de consenso.
- Verificar el estado en exploradores como [beaconchain.info](https://beaconchain.info/) usando la clave pública del validator (presente en el `cluster-lock.json`).

---

## 8. Checks finales y buenas prácticas en DappNode

### 8.1 Verificar conectividad Charon entre nodos

Desde la pestaña **Logs** del paquete Obol, verificar:

- Que no aparezcan errores de conexión persistentes hacia alguno de los peers.
- Que los logs no muestren timeouts repetidos al comunicarse con otros nodos.
- Si la conectividad p2p no se establece automáticamente mediante NAT, considerar **port forwarding** del puerto **3610** en el router hacia la IP del DappNode.

### 8.2 Backups recomendados

| Cuándo hacer backup | Motivo |
|---|---|
| Inmediatamente tras el DKG | Obligatorio antes de activar validators |
| Semanalmente o mensualmente | Protección rutinaria |
| Antes de actualizar el paquete Obol | Evitar pérdidas durante actualizaciones |

Mantener al menos una copia en un medio offline y seguro (USB cifrada, disco externo desconectado).

### 8.3 Consideraciones de seguridad

- **No compartir key shares** con nadie bajo ninguna circunstancia.
- **Verificar la integridad de los ENR** antes de participar en un cluster.
- **Revisar siempre el dominio** del Launchpad antes de conectar la wallet. Dominios oficiales: `launchpad.obol.org` (mainnet) y `hoodi.launchpad.obol.org` (Hoodi).
- **Mantener los clientes actualizados:** paquete Obol, clientes de ejecución y consenso.

### 8.4 Alta disponibilidad y tolerancia a fallos

El cluster con umbral 3-de-4 puede tolerar la caída de **1 nodo** sin perder capacidad de validación. La caída de 2 o más nodos simultáneamente resultará en missed duties y penalizaciones.

Para maximizar la disponibilidad:

- Asegurar conexión a internet estable y de alta disponibilidad en cada operador.
- Mantener los nodos sincronizados continuamente.
- Mantener comunicación abierta entre operadores para coordinar mantenimientos.
- Usar alertas o monitoreo externo para detectar caídas de nodos.

---

## Resumen del flujo completo

```
[Cada operador] Sincronizar EL + CL en DappNode
        ↓
[Cada operador] Instalar paquete Obol desde DappStore
        ↓
[Cada operador] Copiar ENR desde pestaña Info del paquete Obol
        ↓
[Creator] Crear cluster en Obol DV Launchpad (4 operadores, ENRs, validators)
        ↓
[Creator] Firmar configuración → Generar enlace de invitación
        ↓
[Operators x3] Abrir enlace, conectar wallet, enviar ENR, firmar
        ↓
[Launchpad] Genera Definition File URL (todas las firmas completadas)
        ↓
[Cada operador] Copiar Definition File URL → Pestaña Config → campo Cluster-N → Update
        ↓
[Cada operador] Reiniciar contenedor Charon desde pestaña Info
        ↓
[Automático] DKG se ejecuta entre los 4 nodos
        ↓
[Cada operador] Verificar DKG exitoso en pestaña Logs
        ↓
[Cada operador] ⚠️ Realizar backup OBLIGATORIO desde pestaña Backup
        ↓
[1 operador] Extraer deposit-data.json del backup → Depositar 32 ETH por validator
        ↓
[Todos] Esperar activación (mín. 16h) → Validators activos y funcionando
```

---

## Lido CSM con Obol DVT

### ¿Qué es Lido CSM?

El **Community Staking Module (CSM)** de Lido es el primer módulo de Lido con **entrada permissionless** (sin necesidad de autorización), que permite a cualquier persona convertirse en operador de nodos de Lido proporcionando un **bond (colateral)** en ETH. A diferencia del staking solo tradicional que requiere 32 ETH, el CSM permite participar con un **bond de solo 1.5 ETH** (en Hoodi testnet, 2.4 ETH en mainnet), que además puede dividirse entre los miembros de un cluster DVT.

**Beneficios clave de Lido CSM:**
- **Recompensas hasta 2.3x mayores** que el solo staking tradicional
- **Bond reducido** (1.5 ETH en testnet, 2.4 ETH en mainnet vs 32 ETH)
- **Acceso permissionless**: cualquiera puede registrarse como operador
- **Recompensas suavizadas (smoothing)** con otros módulos de Lido
- **Compatible con DVT**: Obol y SSV Network soportados oficialmente

### Ventajas de CSM + Obol DVT

Combinar Lido CSM con Obol DVT ("Squad Staking") ofrece ventajas adicionales:

| Ventaja | Descripción |
|---|---|
| **Bond compartido** | El colateral de 1.5 ETH se divide entre los operadores del cluster (ej. 0.375 ETH por operador en un cluster de 4) |
| **Resiliencia** | El cluster tolera la caída de 1 operador sin penalizaciones |
| **Menores barreras de entrada** | Accesible para operadores caseros sin necesidad de 32 ETH completos |
| **Credencial Techne** | Obtener credenciales on-chain de Obol demostrando experiencia en DVT mainnet |
| **Recompensas premium** | Beneficios de CSM (2.3x vs solo staking) + incentivos adicionales DVT del 20% |
| **Comunidad y soporte** | Operar en squad reduce riesgos y proporciona un grupo de apoyo |

### Flujo simplificado CSM + Obol en DappNode

El flujo para configurar Lido CSM con Obol DVT en DappNode es **casi idéntico** al flujo estándar DVT documentado en esta guía, con las siguientes diferencias clave:

#### Pasos adicionales específicos de CSM:

**1. Preparación previa al cluster (antes del paso 3 del flujo estándar):**
   - Crear una **Safe multisig** con todos los operadores del cluster (threshold 3/4 o 4/7)
     - Hoodi Testnet: `https://app.safe.protofire.io/welcome`
     - Mainnet: `https://app.safe.global`
   - Crear un **Splitter contract** en `https://splits.org` para distribuir recompensas entre operadores
   - Opcional: incluir `protocoldevelopmentfee.obol.eth` con 0.1% en el splitter para contribuir al desarrollo de Obol y recibir incentivos adicionales del 20%

**2. En el Obol DV Launchpad (paso 3.2):**
   - En el campo **"Withdrawal Configuration"**, seleccionar **"LIDO CSM"** en lugar de ingresar direcciones manualmente
   - Esto configura automáticamente:
     - **Withdrawal Address**: Lido Withdrawal Vault (`0xF0179dEC45a37423EAD4FaD5fCb136197872EAd9` en Hoodi)
     - **Fee Recipient Address**: Lido EL Rewards Vault (`0x9b108015fe433F173696Af3Aa0CF7CDb3E104258` en Hoodi)

**3. Configuración de MEV-Boost (obligatoria para CSM):**
   - En DappNode, configurar MEV-Boost con **solo relays aprobados por Lido**
   - Lista de relays vetados: [Lido CSM Vetted Relays](https://enchanted-direction-844.notion.site/6d369eb33f664487800b0dedfe32171e)
   - Configurar `min-bid` máximo de 0.07 ETH
   - Configurar `builder-boost-factor` al 100%
   - ⚠️ **Crítico**: Todos los operadores del cluster deben usar los **mismos relays** para evitar fallos de consenso

**4. Registro en el Lido CSM Widget (después del DKG exitoso):**
   - Abrir el **Lido CSM Widget**:
     - Mainnet: `https://csm.lido.fi`
     - Hoodi Testnet: `https://csm.testnet.fi`
   - Seleccionar **modo Extended**
   - Configurar:
     - **Manager Address**: dirección de la Safe multisig del cluster
     - **Rewards Address**: dirección del Splitter contract
     - **Permission type**: Extended (otorga control total al Manager Address)
   - Subir el archivo `deposit-data.json` extraído del backup del paquete Obol
   - Proporcionar el **bond requerido** en ETH/stETH/wstETH (1.5 ETH en Hoodi, 2.4 ETH en mainnet)
   - ⚠️ **Importante**: Solo un operador del cluster realiza el registro y el bond. Los 32 ETH por validator los deposita Lido automáticamente desde su pool.

**5. Monitoreo y reclamación de recompensas:**
   - **Bond rewards**: El bond en stETH se revalúa diariamente (rebalancing)
   - **Operator rewards**: Recompensas de EL + CL distribuidas cada 7 días (Hoodi) o 28 días (mainnet)
   - Reclamar recompensas en el Widget CSM → pestaña "Bond & Rewards" → "Claim"
   - ⚠️ **Crítico**: Reclamar siempre en **wstETH** (no en stETH), ya que es el único token compatible con el Splitter contract

### Diferencias clave vs setup DVT estándar

| Aspecto | DVT Estándar (esta guía) | Lido CSM + DVT |
|---|---|---|
| **Depósito inicial** | 32 ETH por validator del cluster | 1.5-2.4 ETH bond por cluster. Lido deposita los 32 ETH |
| **Withdrawal address** | Definida libremente por el cluster | Fija: Lido Withdrawal Vault |
| **Fee recipient** | Definida libremente por el cluster | Fija: Lido EL Rewards Vault |
| **MEV-Boost** | Opcional | **Obligatorio** con relays vetados por Lido |
| **Registro del validator** | Depósito directo vía Launchpad | Registro vía Lido CSM Widget |
| **Recompensas** | Recompensas estándar de Ethereum | Recompensas premium (2.3x) + smoothing + incentivos DVT |
| **Gestión del cluster** | Informal | Requiere Safe multisig + Splitter contract |
| **Salida de validators** | Comandos `charon exit` | Comandos `charon exit` + proceso CSM Widget |

### Recursos oficiales CSM

- [Lido CSM — Documentación oficial](https://docs.lido.fi/run-on-lido/csm/)
- [Lido CSM + Obol — Guía Lido Docs](https://docs.lido.fi/run-on-lido/csm/node-setup/DVT-setup/obol)
- [Obol — Create a Lido CSM DV](https://docs.obol.org/run-a-dv/integrations/lido-csm)
- [Lido CSM Widget Mainnet](https://csm.lido.fi)
- [Lido CSM Widget Hoodi Testnet](https://csm.testnet.fi)
- [Lido CSM Vetted MEV Relays](https://enchanted-direction-844.notion.site/6d369eb33f664487800b0dedfe32171e)
- [Obol Blog — Squad Staking in Lido CSM](https://blog.obol.org/csm/)

---

## referencias

- [Obol — Create a DV With a Group](https://docs.obol.org/next/run-a-dv/start/create-a-dv-with-a-group)
- [DappNode — Obol Network Package](https://docs.dappnode.io/docs/user/staking/ethereum/dvt-technologies/obol-network/)
- [Obol — Activate a DV](https://docs.obol.org/run-a-dv/running/activate-a-dv)
- [Obol — DV Launchpad](https://docs.obol.org/learn/readme/launchpad)
- [Obol — Distributed Key Generation (DKG)](https://docs.obol.org/learn/charon/dkg)
- [DappNode Discord (soporte comunidad)](https://discord.gg/dappnode)

---

## Licencia

MIT
