# Exercício 2 — OSI aplicado

## Tabela operacional OSI → Linux

| Camada OSI | Conceitos-chave | Comandos Linux | Evidência esperada |
|---|---|---|---|
| 2 — Enlace | Interfaces, MAC, estado do enlace e estatísticas de quadros | `ip link`, `ip -s link` | Interface ativa, endereço MAC e contadores RX/TX |
| 3 — Rede | Endereçamento IP, sub-redes e roteamento | `ip addr`, `ip route` | Endereço/prefixo configurado e rotas do kernel |
| 4 — Transporte | TCP/UDP, portas, conexões e estados | `ss -tulpen`, `ss -tan` | Portas em escuta, conexões e estados TCP |
| 5 — Sessão | Controle e manutenção de sessões entre aplicações | `ss -tan`, `curl -v URL` | Estado da conexão e ciclo de estabelecimento/manutenção |
| 6 — Apresentação | TLS, certificados, criptografia e formato dos dados | `curl -v https://example.com`, `openssl s_client` | Negociação TLS, certificado e protocolo/cifra |
| 7 — Aplicação | Protocolos e respostas das aplicações | `curl -I https://example.com`, `getent hosts example.com` | Resposta HTTP, cabeçalhos e resolução usada pela aplicação |

## Validação das linhas selecionadas

### Linha 1 — Camada 2 — Enlace

**Comando(s):**

```bash
ip link
```

**Evidência:**

```text
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN mode DEFAULT group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
2: eth0@if8: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc noqueue state UP mode DEFAULT group default
    link/ether 96:48:20:09:94:83 brd ff:ff:ff:ff:ff:ff link-netnsid 0
```

**Justificativa técnica:** O comando `ip link` consulta as interfaces de enlace disponíveis no namespace de rede. A saída mostra o endereço MAC, o tipo da interface e os estados administrativo e físico (`UP`/`LOWER_UP`). Isso representa a camada 2 porque observa a conectividade local e a identificação da interface antes do endereçamento IP.

### Linha 2 — Camada 3 — Rede

**Comando(s):**

```bash
ip route
```

**Evidência:**

```text
default via 172.17.0.1 dev eth0
172.17.0.0/16 dev eth0 proto kernel scope link src 172.17.0.2
```

**Justificativa técnica:** O comando `ip route` exibe as decisões de encaminhamento do kernel. A tabela permite identificar redes conectadas, gateway default, interface de saída e endereço de origem. Esses elementos são próprios da camada 3, que entrega pacotes entre redes usando endereços IP.

### Linha 3 — Camada 7 — Aplicação

**Comando(s):**

```bash
curl -I https://example.com
```

**Evidência:**

```text
HTTP/2 200
date: Mon, 12 Oct 2026 13:35:03 GMT
content-type: text/html; charset=utf-8
server: cloudflare
last-modified: Fri, 09 Oct 2026 20:19:47 GMT
allow: GET, HEAD
accept-ranges: bytes
age: 4964
cf-cache-status: HIT
cf-ray: a4967c4ebcb5645b-GIG
alt-svc: h3=":443"; ma=86400
```

**Justificativa técnica:** O comando `curl -I` realiza uma requisição HTTP HEAD para um serviço de aplicação. A resposta contém o status HTTP e cabeçalhos definidos pelo servidor, como tipo de conteúdo e servidor responsável. Portanto, observa diretamente o comportamento do protocolo de aplicação, e não apenas a existência de conectividade IP.

## Ambiguidade e limites

1. **Falha causada por política de firewall:** o modelo OSI não informa sozinho se um pacote foi bloqueado por `nftables`, pelo firewall do host ou por um equipamento intermediário; é necessário consultar as políticas e os logs.
2. **Problema de configuração ou credencial da aplicação:** uma porta pode estar aberta e o TCP funcionar, mas autenticação, autorização ou parâmetros da aplicação ainda podem falhar. Isso exige análise de logs e do protocolo específico.
3. **Componentes que atravessam várias camadas:** TLS, proxies, VPNs, containers e NAT combinam funções de diferentes camadas. A classificação OSI ajuda a organizar a investigação, mas não identifica sozinha qual componente introduziu o problema.
