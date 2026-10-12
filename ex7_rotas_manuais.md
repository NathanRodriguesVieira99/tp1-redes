# Exercício 7 — Rotas manuais

## Estado inicial

```bash
ip route
```

**Evidência:**

```text
default via 172.17.0.1 dev eth0
172.17.0.0/16 dev eth0 proto kernel scope link src 172.17.0.2
```

## Rota manual

- **Destino escolhido:** `203.0.113.10`
- **Prefixo:** `/32`
- **Gateway/interface:** `172.17.0.1` pela `eth0`
- **Ambiente:** A tentativa no container original falhou com `Operation not permitted` porque ele não tinha a capacidade `NET_ADMIN`. Para realizar a alteração, foi usado o container `tp1-redes-ubuntu-netadmin`, criado com `--cap-add NET_ADMIN`.
- **Comando de adição:**

```bash
ip route add 203.0.113.10/32 via 172.17.0.1 dev eth0
```

## Verificação da alteração

```bash
ip route
```

**Evidência:**

```text
default via 172.17.0.1 dev eth0
172.17.0.0/16 dev eth0 proto kernel scope link src 172.17.0.3
203.0.113.10 via 172.17.0.1 dev eth0
```

**Descrição do que mudou:** Foi adicionada uma rota específica para `203.0.113.10`, encaminhando esse destino pelo gateway `172.17.0.1` e pela interface `eth0`. As rotas default e conectada permaneceram inalteradas.

## Justificativa técnica

O prefixo `/32` representa um único endereço IPv4 e limita a mudança exatamente ao destino escolhido. Isso reduz o risco de desviar tráfego de outros hosts. Um prefixo maior, como `/24` ou `/16`, poderia capturar muitos destinos e encaminhá-los para um gateway incorreto, causando perda de conectividade ou caminhos assimétricos.

## Remoção e verificação

```bash
ip route del 203.0.113.10/32 via 172.17.0.1 dev eth0
```

```bash
ip route
```

**Evidência:**

```text
default via 172.17.0.1 dev eth0
172.17.0.0/16 dev eth0 proto kernel scope link src 172.17.0.3
```

## Plano de rollback

1. Remover a rota adicionada usando o mesmo destino, gateway e interface: `ip route del 203.0.113.10/32 via 172.17.0.1 dev eth0`.
2. Conferir `ip route` e comparar com o estado inicial, confirmando que a rota específica não existe mais.
3. Se a sessão ficar inacessível, acessar o host por um console alternativo e remover a rota antes de restaurar o serviço de rede.
