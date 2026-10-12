# Exercício 11 — Linux como roteador

## Estado inicial do forwarding IPv4

### Comando

```bash
sysctl net.ipv4.ip_forward
```

**Evidência:**

```text
net.ipv4.ip_forward = 1
```

O container de laboratório já iniciou com forwarding IPv4 habilitado.

## Habilitação temporária

```bash
sysctl -w net.ipv4.ip_forward=1
```

**Evidência da alteração:**

```text
sysctl: setting key "net.ipv4.ip_forward", ignoring: Read-only file system
net.ipv4.ip_forward = 1
```

O valor permaneceu `1`. A tentativa de escrita foi bloqueada porque o sysctl está montado como somente leitura no ambiente do container; o estado efetivo já era habilitado.

## Implicações técnicas

### O que `ip_forward=1` permite

Com `net.ipv4.ip_forward=1`, o kernel pode encaminhar pacotes IPv4 recebidos por uma interface para outra interface, em vez de tratar o host apenas como destino final. Isso habilita a função básica de encaminhamento, mas não cria por si só rotas, NAT ou políticas de segurança.

### O que ainda é necessário para atuar como roteador

Ainda seriam necessárias pelo menos duas interfaces conectadas a redes distintas, endereçamento válido, rotas de retorno nos demais hosts, regras de firewall/forwarding e, quando aplicável, NAT. Também seria necessário validar que o gateway upstream conhece o caminho de volta para as redes atrás do host.

## Reversão

```bash
sysctl -w net.ipv4.ip_forward=1
```

**Evidência da reversão:**

```text
sysctl: setting key "net.ipv4.ip_forward", ignoring: Read-only file system
net.ipv4.ip_forward = 1
```

O valor final permaneceu `1`, igual ao estado inicial; portanto, não houve alteração persistente no container.

## Riscos de segurança

1. O host pode encaminhar tráfego não autorizado entre redes, criando um caminho de acesso não documentado.
2. Sem firewall restritivo, serviços e redes internas podem ficar expostos a conexões que antes não alcançavam esses segmentos.
3. A combinação com NAT ou rotas incorretas pode causar assimetria, vazamento de tráfego e dificuldade de auditoria ou resposta a incidentes.
