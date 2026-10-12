# Exercício 5 — Configuração IPv6

## Configuração atual

### Endereços IPv6

```bash
ip -6 addr show
```

**Evidência:**

```text
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 state UNKNOWN qlen 1000
    inet6 ::1/128 scope host
       valid_lft forever preferred_lft forever
```

### Rotas IPv6

```bash
ip -6 route show
```

**Evidência:**

```text
[nenhuma rota IPv6 exibida]
```

## Classificação por interface

| Interface | Endereços link-local | Endereços global/ULA | Observações |
|---|---|---|---|
| `lo` | Nenhum; `::1/128` é loopback e tem escopo `host` | Nenhum | Possui somente o endereço local do próprio host |
| `eth0` | Nenhum | Nenhum | A interface está ativa, mas não recebeu configuração IPv6 |

## Rota default IPv6

- **Existe?** Não.
- **Evidência:** `ip -6 route show` não retornou nenhuma entrada.
- **Impacto:** O host não possui caminho IPv6 para redes locais ou destinos externos; portanto, não consegue operar com conectividade IPv6 além do loopback.

## Comparação entre interfaces

### Interface principal

- **Interface:** `eth0`
- **Endereços:** Nenhum endereço IPv6 configurado.

### Interface virtual

- **Interface:** `lo`
- **Endereços:** `::1/128` com escopo `host`.

### Diferenças

As interfaces podem diferir porque têm funções distintas: `lo` é uma interface local criada para comunicação do próprio host, enquanto `eth0` é a interface de rede usada para alcançar outros hosts. Neste container, o Docker configurou IPv4 na `eth0`, mas não habilitou ou delegou IPv6 para ela. Por isso, `lo` mantém `::1/128`, enquanto `eth0` não possui nem endereço link-local nem rota IPv6.

## Causas prováveis de ausência de conectividade global

1. O Docker ou a rede do host não habilitou IPv6 para a rede do container, impedindo a atribuição de endereços e rotas IPv6.
2. Não existe Router Advertisement (RA) ou DHCPv6 fornecendo prefixo, endereço e gateway IPv6 para a interface `eth0`.
3. A configuração de roteamento IPv6 está ausente ou bloqueada por política de rede/firewall, mesmo que um endereço IPv6 venha a ser atribuído.
