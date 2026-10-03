# Direccionamiento IP

## Redes utilizadas

| Segmento | Red | Máscara decimal | Uso |
|---|---|---|---|
| VLAN 10 - Usuarios | `10.21.40.0/25` | `255.255.255.128` | Red de clientes Windows |
| R2 ↔ ISP | `198.51.100.0/30` | `255.255.255.252` | Enlace WAN de R2 |
| ISP ↔ FortiGate | `192.0.2.0/30` | `255.255.255.252` | Enlace WAN del FortiGate |
| Servidor | `10.21.40.128/28` | `255.255.255.240` | Red protegida del servidor Ubuntu |
| Red física | `192.168.18.0/24` | `255.255.255.0` | Acceso físico e Internet |

## Direccionamiento por dispositivo

| Dispositivo | Interfaz | Dirección IP | Gateway / Siguiente salto | Descripción |
|---|---|---|---|---|
| Windows 10 | Ethernet | `10.21.40.10/25` | `10.21.40.1` | Cliente de VLAN 10, asignado por DHCP |
| R2 | FastEthernet1/0.10 | `10.21.40.1/25` | — | Gateway de VLAN 10 |
| R2 | FastEthernet0/0 | `198.51.100.2/30` | `198.51.100.1` | WAN y peer IPsec |
| ISP | FastEthernet0/0 | `198.51.100.1/30` | — | Enlace hacia R2 |
| ISP | FastEthernet0/1 | `192.0.2.1/30` | — | Enlace hacia FortiGate |
| ISP | FastEthernet1/0 | `192.168.18.164/24` | `192.168.18.1` | Enlace hacia red física |
| FortiGate | port1 | `192.0.2.2/30` | `192.0.2.1` | WAN y peer IPsec |
| FortiGate | port2 | `10.21.40.129/28` | — | Gateway de la red del servidor |
| Ubuntu Server | ens3 | `10.21.40.130/28` | `10.21.40.129` | Servidor HTTPS |
| PC física | Ethernet | `192.168.18.40/24` | `192.168.18.1` | Administración del laboratorio |

## Redes protegidas por la VPN

Desde el punto de vista de R2:

- Red local: `10.21.40.0/25`
- Red remota: `10.21.40.128/28`

Desde el punto de vista del FortiGate:

- Red local: `10.21.40.128/28`
- Red remota: `10.21.40.0/25`

## Peers IPsec

| Equipo | IP del peer |
|---|---|
| Cisco R2 | `198.51.100.2` |
| FortiGate | `192.0.2.2` |

## Rutas principales

### R2

```text
0.0.0.0/0 → 198.51.100.1
```

### ISP

```text
0.0.0.0/0 → 192.168.18.1
```

### FortiGate

```text
0.0.0.0/0       → 192.0.2.1 por port1
192.168.18.0/24  → 192.0.2.1 por port1
10.21.40.0/25    → VPN-R2
```

## DNS

El pool DHCP de R2 entrega a Windows:

```text
DNS: 8.8.8.8
```

## Servicios

| Equipo | Servicio | Puerto |
|---|---|---|
| Ubuntu Server | HTTPS / Apache | TCP 443 |
| FortiGate | Administración HTTPS | TCP 443 |
| FortiGate | PING | ICMP |
