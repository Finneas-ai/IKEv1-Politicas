# VPN Site-to-Site IKEv1 basada en Políticas (Crypto Map)

Video de referencia: https://youtu.be/OSc3m_waZo8

## Descripción

VPN Site-to-Site clásica **basada en políticas** (policy-based) entre dos routers Cisco IOS (Peer A y Peer B), usando **IKEv1** con `crypto isakmp`, un `crypto ipsec transform-set` y una `crypto map` aplicada a la interfaz WAN. El tráfico interesante se define con una ACL extendida.

## Topología

```
LAN A (10.6.82.0/25) -- Peer A -- ISP -- Peer B -- LAN B (10.6.82.128/25)
```

## Direccionamiento IP

| Dispositivo | Interfaz | IP / Máscara            |
|-------------|----------|--------------------------|
| ISP         | e0/0     | 20.6.82.1 /30            |
| ISP         | e0/1     | 20.6.82.5 /30            |
| Peer A      | e0/0     | 20.6.82.2 /30            |
| Peer A      | e0/1     | 10.6.82.1 /25 (LAN A)    |
| Peer B      | e0/0     | 20.6.82.6 /30            |
| Peer B      | e0/1     | 10.6.82.129 /25 (LAN B)  |
| PC1 (VPCS)  | -        | 10.6.82.10/25, gw 10.6.82.1   |
| PC2 (VPCS)  | -        | 10.6.82.138/25, gw 10.6.82.129 |

## Parámetros IKEv1 / IPsec

| Parámetro    | Valor                     |
|--------------|---------------------------|
| encryption   | aes 256                   |
| hash         | sha256                    |
| authentication | pre-share               |
| group (DH)   | 14                        |
| lifetime     | 86400 s                   |
| transform-set| esp-aes 256 esp-sha256-hmac, modo tunnel |
| PSK          | CLAVE0682                 |

## Configuración

### ISP

```
interface e0/0
 ip address 20.6.82.1 255.255.255.252
 no shut
interface e0/1
 ip address 20.6.82.5 255.255.255.252
 no shut
```

### Peer A

```
interface e0/0
 ip address 20.6.82.2 255.255.255.252
 no shut
interface e0/1
 ip address 10.6.82.1 255.255.255.128
 no shut
ip route 0.0.0.0 0.0.0.0 20.6.82.1

crypto isakmp policy 10
 encryption aes 256
 hash sha256
 authentication pre-share
 group 14
 lifetime 86400
crypto isakmp key CLAVE0682 address 20.6.82.6

crypto ipsec transform-set TS0682 esp-aes 256 esp-sha256-hmac
 mode tunnel

ip access-list extended VPN-ACL
 permit ip 10.6.82.0 0.0.0.127 10.6.82.128 0.0.0.127

crypto map MAP0682 10 ipsec-isakmp
 set peer 20.6.82.6
 set transform-set TS0682
 match address VPN-ACL

interface e0/0
 crypto map MAP0682
```

### Peer B

```
interface e0/0
 ip address 20.6.82.6 255.255.255.252
 no shut
interface e0/1
 ip address 10.6.82.129 255.255.255.128
 no shut
ip route 0.0.0.0 0.0.0.0 20.6.82.5

crypto isakmp policy 10
 encryption aes 256
 hash sha256
 authentication pre-share
 group 14
crypto isakmp key CLAVE0682 address 20.6.82.2

crypto ipsec transform-set TS0682 esp-aes 256 esp-sha256-hmac
 mode tunnel

ip access-list extended VPN-ACL
 permit ip 10.6.82.128 0.0.0.127 10.6.82.0 0.0.0.127

crypto map MAP0682 10 ipsec-isakmp
 set peer 20.6.82.2
 set transform-set TS0682
 match address VPN-ACL

interface e0/0
 crypto map MAP0682
```

### Hosts finales (VPCS)

```
PC1: ip 10.6.82.10/25 10.6.82.1
PC2: ip 10.6.82.138/25 10.6.82.129
```

## Verificación

```
show crypto isakmp sa
show crypto ipsec sa
show crypto map
ping 10.6.82.138 source 10.6.82.10
```
