# Exercício 4 — Routing table com `ip route`

## Estado da tabela de rotas

```bash
ip route
```

**Evidência:**

```text
default via 172.17.0.1 dev eth0
172.17.0.0/16 dev eth0 proto kernel scope link src 172.17.0.2
```

### Tabela local consultada

```bash
ip route show table local
```

**Evidência:**

```text
local 127.0.0.0/8 dev lo proto kernel scope host src 127.0.0.1
local 127.0.0.1 dev lo proto kernel scope host src 127.0.0.1
broadcast 127.255.255.255 dev lo proto kernel scope link src 127.0.0.1
local 172.17.0.2 dev eth0 proto kernel scope host src 172.17.0.2
broadcast 172.17.255.255 dev eth0 proto kernel scope link src 172.17.0.2
```

## Interpretação

### Rota default

- **Gateway:** `172.17.0.1`
- **Interface:** `eth0`
- **Métrica:** Não exibida; a decisão usa a rota default quando não existe uma rota mais específica.
- **Interpretação:** Todo destino sem uma rota mais específica é encaminhado ao gateway Docker pela `eth0`.

### Rotas conectadas

#### Rota conectada 1

- **Prefixo:** `172.17.0.0/16`
- **Interface:** `eth0`
- **Interpretação:** Rota diretamente conectada à rede do container; usa `172.17.0.2` como endereço de origem.

#### Rota conectada 2

- **Prefixo:** `127.0.0.0/8`
- **Interface:** `lo`
- **Interpretação:** Rota local para o loopback. Ela pertence à tabela `local` e atende somente ao próprio host, não sendo usada para encaminhamento externo.

## Simulação de decisões

| Destino | Entrada vencedora | Justificativa |
|---|---|---|
| `172.17.0.10` — destino local | `172.17.0.0/16 dev eth0` | A rota `/16` é mais específica que a default; o tráfego permanece na rede Docker pela `eth0`. |
| `8.8.8.8` — destino público | `default via 172.17.0.1 dev eth0` | Não há rota específica para esse IP; a rota default vence e encaminha o tráfego ao gateway. |
| `192.168.1.10` — fora da LAN do container | `default via 172.17.0.1 dev eth0` | O destino não pertence a `172.17.0.0/16`; sem outra rota, aplica-se a rota default. |

## Riscos

1. O tráfego pode ser encaminhado ao gateway errado, causando perda de conectividade com redes externas.
2. Uma rota mais específica incorreta pode desviar o tráfego para uma interface ou caminho que não alcança o destino.
3. Uma métrica inadequada pode priorizar um caminho instável ou menos confiável quando existem múltiplas rotas disponíveis.
