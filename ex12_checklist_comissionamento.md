# Exercício 12 — Checklist de comissionamento de conectividade

## Checklist

| # | Objetivo | Comando(s) para validar | Evidência esperada | Passou/Falhou |
|---:|---|---|---|---|
| 1 | Validar endereços IPv4 | `ip -4 addr` | `eth0` com endereço IPv4 e prefixo válido | Passou |
| 2 | Validar endereços IPv6 | `ip -6 addr` | Endereço IPv6 e escopo conhecidos | Falhou — somente loopback |
| 3 | Validar rotas IPv4/IPv6 | `ip -4 route; ip -6 route` | Rotas coerentes para os protocolos em uso | Falhou — não há rotas IPv6 |
| 4 | Validar rota default | `ip route` | Rota `default` com gateway e interface | Passou |
| 5 | Validar DNS | `cat /etc/resolv.conf; getent hosts example.com` | Nameserver configurado e resolução funcionando | Passou |
| 6 | Validar `/etc/hosts` | `cat /etc/hosts` | Entradas locais esperadas, sem override indevido | Passou |
| 7 | Validar hostname | `hostname; cat /etc/hostname` | Mesmo hostname atual e persistido | Passou |
| 8 | Validar portas TCP em escuta | `ss -ltnp` | Apenas portas autorizadas ou nenhuma porta inesperada | Passou |
| 9 | Validar evidências de DHCP | `command -v dhclient; find /var/lib/dhcp -type f` | Cliente e/ou lease identificável | Falhou — gerenciado pelo Docker |
| 10 | Validar forwarding IPv4 | `cat /proc/sys/net/ipv4/ip_forward` | Estado conforme a política operacional | Passou — valor 1 |

## Evidências reais dos itens executados

### Item 1 — Endereços IPv4

```bash
ip -4 addr
```

**Evidência:**

```text
2: eth0@if12: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 state UP
    inet 172.17.0.2/16 brd 172.17.255.255 scope global eth0
```

**Resultado:** Passou. A interface `eth0` está ativa e possui IPv4 `172.17.0.2/16`.

### Item 2 — Endereços IPv6

```bash
ip -6 addr
```

**Evidência:**

```text
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 state UNKNOWN qlen 1000
    inet6 ::1/128 scope host
```

**Resultado:** Falhou para conectividade operacional: somente o loopback IPv6 está configurado.

### Item 3 — Rotas IPv4 e IPv6

```bash
ip -4 route
ip -6 route
```

**Evidência:**

```text
default via 172.17.0.1 dev eth0
172.17.0.0/16 dev eth0 proto kernel scope link src 172.17.0.2
[nenhuma rota IPv6 exibida]
```

**Resultado:** Falhou para IPv6; as rotas IPv4 estão coerentes.

### Item 4 — Rota default

```bash
ip route
```

**Evidência:**

```text
default via 172.17.0.1 dev eth0
```

**Resultado:** Passou. Existe gateway default pela `eth0`.

### Item 5 — DNS

```bash
cat /etc/resolv.conf
getent hosts example.com
```

**Evidência:**

```text
nameserver 192.168.65.7
172.66.147.243  example.com
104.20.23.154   example.com
```

**Resultado:** Passou. Há nameserver configurado e a resolução funcionou.

### Item 6 — Portas TCP em escuta

```bash
ss -ltnp
```

**Evidência:**

```text
State Recv-Q Send-Q Local Address:Port Peer Address:Port Process
```

**Resultado:** Passou. Nenhuma porta TCP está em escuta no momento da verificação.

## Decisões técnicas

### Rotas

Em caso de falha, primeiro compararia `ip route` com o estado conhecido e confirmaria gateway, interface e prefixo. Uma rota manual deveria ser específica, preferencialmente `/32` para um único destino, e testada com `ip route get` antes de ser mantida. Toda mudança deve ter comando de rollback documentado e ser removida se desviar tráfego ou causar assimetria.

### Resolução

Usaria `/etc/hosts` apenas para testes controlados, aliases locais ou uma contingência temporária e documentada. Para nomes compartilhados por vários hosts, o DNS deve ser a fonte oficial, pois permite administração centralizada, propagação e controle de TTL. Overrides locais permanentes criam divergências e dificultam a investigação de falhas.
