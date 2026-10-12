# Exercício 3 — IPv4 vs IPv6: inventário e prontidão operacional

## Endereços IPv4

### Comando

```bash
ip -4 addr
```

**Evidência:**

```text
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default qlen 1000
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
2: eth0@if8: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc noqueue state UP group default link-netnsid 0
    inet 172.17.0.2/16 brd 172.17.255.255 scope global eth0
       valid_lft forever preferred_lft forever
```

## Endereços IPv6

### Comando

```bash
ip -6 addr
```

**Evidência:**

```text
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 state UNKNOWN qlen 1000
    inet6 ::1/128 scope host
       valid_lft forever preferred_lft forever
```

## Rotas IPv4

### Comando

```bash
ip -4 route
```

**Evidência:**

```text
default via 172.17.0.1 dev eth0
172.17.0.0/16 dev eth0 proto kernel scope link src 172.17.0.2
```

## Rotas IPv6

### Comando

```bash
ip -6 route
```

**Evidência:**

```text
[nenhuma rota IPv6 exibida]
```

## Resolução de domínio

**Domínio escolhido:** `example.com`

```bash
getent ahosts example.com
```

**Evidência:**

```text
104.20.23.154   STREAM example.com
104.20.23.154   DGRAM
104.20.23.154   RAW
172.66.147.243  STREAM
172.66.147.243  DGRAM
172.66.147.243  RAW
```

## Respostas do inventário

- **IPv4 principal e prefixo:** `172.17.0.2/16` na interface `eth0`.
- **IPv6 `fe80::/10` e interface:** Não existe endereço `fe80::/10` em nenhuma interface.
- **IPv6 global/ULA e prefixo:** Não existe endereço IPv6 global ou ULA; há somente `::1/128` na interface `lo`.
- **Rota default IPv4:** Existe: `default via 172.17.0.1 dev eth0`.
- **Rota default IPv6:** Não existe; `ip -6 route` não retornou entradas.

## Prontidão operacional para IPv6

**Conclusão:** Meu host não está operacionalmente pronto para IPv6 além do loopback.

1. `eth0` não possui endereço IPv6 link-local, global ou ULA; existe somente `::1/128` na interface `lo`.
2. `ip -6 route` não apresenta nenhuma rota IPv6, incluindo rota default.
3. `getent ahosts example.com` retornou somente endereços IPv4, sem endereço IPv6 para o domínio escolhido.
