# Exercício 9 — Portas TCP

## Portas TCP em escuta

```bash
ss -ltnp
```

**Evidência:**

```text
State Recv-Q Send-Q Local Address:Port Peer Address:Port Process
```

## Porta comum

- **Porta escolhida:** `22/TCP`
- **Está em uso?** Não.
- **Processo:** Nenhum processo.
- **Evidência:**

```text
State Recv-Q Send-Q Local Address:Port Peer Address:Port Process
```

## Porta alta para serviço novo

- **Porta escolhida:** `55000/TCP`
- **Justificativa:** É uma porta alta, fora da faixa privilegiada, portanto não exige a porta padrão de serviços conhecidos e reduz a chance de colisão com serviços convencionais. A disponibilidade foi verificada antes do uso.

## Demonstração de conflito

### Processo utilizado

```bash
nc -l 127.0.0.1 55000
```

### Verificação da ocupação

```bash
ss -ltnp '( sport = :55000 )'
```

**Evidência:**

```text
State  Recv-Q Send-Q Local Address:Port  Peer Address:Port Process
LISTEN 0      1          127.0.0.1:55000      0.0.0.0:*    users:(("nc",pid=213,fd=3))
```

### Encerramento e liberação

```bash
kill 213
```

**Evidência:**

```text
State Recv-Q Send-Q Local Address:Port Peer Address:Port Process
```

A porta `55000` não aparece mais em estado `LISTEN`, confirmando a liberação.

## `0.0.0.0` versus `127.0.0.1`

Escutar em `127.0.0.1` limita o serviço ao próprio host ou container, reduzindo a exposição para outros hosts da rede. Isso é adequado para componentes internos que não precisam aceitar conexões externas. Escutar em `0.0.0.0` associa o serviço a todas as interfaces IPv4 disponíveis. Assim, outros hosts podem tentar acessá-lo por qualquer interface que tenha conectividade, desde que firewall e políticas permitam. O bind em `0.0.0.0` aumenta a superfície de exposição e exige autenticação, firewall e revisão das interfaces publicadas. Em um container, a exposição externa também depende do mapeamento de portas feito pelo Docker.
