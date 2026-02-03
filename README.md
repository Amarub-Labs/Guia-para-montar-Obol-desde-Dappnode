# obol-dvt-dappnode-guide

> Guía completa para configurar un cluster de **Obol Distributed Validator (DVT)** con **4 operadores usando DappNode**, incluyendo la ceremonia DKG paso a paso y configuración con **Lido Community Staking Module (CSM)**.

[![License](https://img.shields.io/badge/license-MIT-blue.svg)](LICENSE)
[![Red](https://img.shields.io/badge/Ethereum-Mainnet%20%7C%20Hoodi-gray.svg)](https://ethereum.org)
[![Obol](https://img.shields.io/badge/Obol-Charon-green.svg)](https://obol.org)
[![DappNode](https://img.shields.io/badge/DappNode-obol%20generic-purple.svg)](https://dappnode.com)
[![Lido](https://img.shields.io/badge/Lido-CSM-blue.svg)](https://lido.fi)

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
