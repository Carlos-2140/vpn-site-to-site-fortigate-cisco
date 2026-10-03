# VPN Site-to-Site FortiGate ↔ Cisco en GNS3

> 🎥 **Video demostrativo:** pendiente de agregar.

## Descripción

Este repositorio documenta la implementación y validación de una VPN IPsec Site-to-Site entre un router Cisco R2 y un FortiGate en GNS3.

El laboratorio permite que un equipo Windows de la red de usuarios `10.21.40.0/25` se comunique de forma segura con un servidor Ubuntu ubicado en la red `10.21.40.128/28`. También se documentan DHCP, VLAN 10, NAT/PAT, acceso HTTPS, rutas estáticas y pruebas de funcionamiento con la VPN activa e inactiva.

## Objetivos

- Configurar la red de usuarios en VLAN 10.
- Asignar direccionamiento IPv4 mediante DHCP desde R2.
- Establecer una VPN IPsec Site-to-Site entre R2 y FortiGate.
- Permitir acceso desde Windows al servidor Ubuntu mediante HTTPS.
- Implementar NAT/PAT para salida a Internet.
- Verificar que la comunicación entre las redes protegidas dependa del túnel VPN.
- Documentar configuraciones, scripts y evidencias del laboratorio.

## Topología lógica

```text
Windows 10
10.21.40.10/25
GW 10.21.40.1
      |
      | VLAN 10
      |
     R2
Fa1/0.10: 10.21.40.1/25
Fa0/0:    198.51.100.2/30
      |
      |
     ISP
Fa0/0: 198.51.100.1/30
Fa0/1: 192.0.2.1/30
Fa1/0: 192.168.18.164/24
      |
      |
 FortiGate
port1: 192.0.2.2/30
port2: 10.21.40.129/28
      |
      |
Ubuntu Server
10.21.40.130/28
GW 10.21.40.129
HTTPS TCP/443
```

### Topología real en GNS3

![Topología GNS3](Topologia/Captura%20de%20pantalla%202026-10-02%20214049.png)

## Direccionamiento IP

La tabla completa de direccionamiento se encuentra en:

[Direccionamiento IP](Direccionamiento%20IP/direccionamiento.md)

Resumen:

| Dispositivo | Interfaz | Dirección IP | Función |
|---|---|---|---|
| Windows 10 | Ethernet | `10.21.40.10/25` | Cliente VLAN 10 |
| R2 | FastEthernet1/0.10 | `10.21.40.1/25` | Gateway VLAN 10 |
| R2 | FastEthernet0/0 | `198.51.100.2/30` | WAN / Peer IPsec |
| ISP | FastEthernet0/0 | `198.51.100.1/30` | Hacia R2 |
| ISP | FastEthernet0/1 | `192.0.2.1/30` | Hacia FortiGate |
| ISP | FastEthernet1/0 | `192.168.18.164/24` | Red física / Internet |
| FortiGate | port1 | `192.0.2.2/30` | WAN / Peer IPsec |
| FortiGate | port2 | `10.21.40.129/28` | Gateway del servidor |
| Ubuntu Server | ens3 | `10.21.40.130/28` | Servidor HTTPS |

## VPN IPsec

La VPN protege las siguientes redes:

- Red de usuarios: `10.21.40.0/25`
- Red del servidor: `10.21.40.128/28`

Parámetros principales utilizados en el laboratorio:

- IKEv1
- Main Mode
- Autenticación mediante Pre-shared Key
- Cifrado DES
- Integridad SHA256
- Diffie-Hellman Group 14
- PFS Group 14 en Phase 2

> La clave precompartida real no debe publicarse en el repositorio.

## Servicios implementados

### Windows 10

- Cliente de la VLAN 10.
- Dirección IPv4 recibida mediante DHCP.
- Acceso al servidor remoto por ICMP y HTTPS.
- Resolución DNS con `8.8.8.8`.

### Cisco R2

- Gateway de la VLAN 10.
- DHCP.
- NAT/PAT.
- Peer de la VPN IPsec.
- Exclusión de NAT para el tráfico protegido por la VPN.

### ISP

- Interconexión entre R2 y FortiGate.
- Salida hacia la red física.
- NAT/PAT para acceso a Internet.

### FortiGate

- Peer remoto de la VPN.
- Políticas de firewall.
- Rutas estáticas.
- Objetos de red.
- NAT para la salida del servidor.

### Ubuntu Server

- Dirección: `10.21.40.130/28`
- Gateway: `10.21.40.129`
- Apache HTTPS.
- Puerto TCP/443.

## Evidencias de funcionamiento

Las pruebas documentadas incluyen:

- Asignación DHCP correcta a Windows.
- Ping al gateway `10.21.40.1`.
- IKE en estado `QM_IDLE / ACTIVE`.
- Contadores IPsec con tráfico cifrado y descifrado.
- Ping desde Windows a `10.21.40.130`.
- `tracert` hacia el servidor.
- Acceso exitoso a `https://10.21.40.130`.
- Acceso a Internet mediante NAT/PAT.
- Resolución DNS.
- Estado de Apache y puerto 443 en Ubuntu.
- Túnel FortiGate en estado **Up**.
- Prueba con VPN **Inactive**.
- Ping y HTTPS fallando cuando el túnel está deshabilitado.

Los documentos se encuentran en:

[Evidencias](Evidencias/)

## Estructura del repositorio

```text
vpn-site-to-site-fortigate-cisco/
├── Configuracion Fortigate/
├── Configuracion switch/
├── Direccionamiento IP/
├── Evidencias/
├── Scripts/
├── Show running-config/
├── Topologia/
└── README.md
```

## Documentación

- [Configuración FortiGate](Configuracion%20Fortigate/)
- [Configuración del switch](Configuracion%20switch/)
- [Direccionamiento IP](Direccionamiento%20IP/)
- [Evidencias](Evidencias/)
- [Scripts](Scripts/)
- [Running-configs](Show%20running-config/)
- [Topología](Topologia/)

## Scripts incluidos

En la carpeta `Scripts/` se incluyen:

- `FortiGate-config-referencia.txt`
- `Script-Ubuntu-HTTPS-configuracion.txt`

## Running-configs

En `Show running-config/` se incluyen las configuraciones finales de:

- Cisco R2
- Router ISP

Antes de publicar configuraciones se deben ocultar claves, contraseñas y cualquier otro dato sensible.

## Resultado final

Con el túnel VPN activo, Windows puede comunicarse con el servidor Ubuntu mediante ICMP y HTTPS.

Al deshabilitar `VPN-R2` en el FortiGate:

- el ping a `10.21.40.130` presenta 100% de pérdida;
- el acceso HTTPS termina en timeout.

Esto demuestra que la comunicación entre la red de usuarios y la red del servidor depende del túnel VPN Site-to-Site.

## Conclusión

El laboratorio implementa correctamente una VPN IPsec Site-to-Site entre Cisco R2 y FortiGate. Además, integra VLAN, DHCP, rutas estáticas, NAT/PAT y un servidor HTTPS, y documenta tanto el funcionamiento normal como la pérdida de conectividad cuando el túnel VPN se deshabilita.
