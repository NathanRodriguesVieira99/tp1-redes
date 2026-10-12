# Exercício 10 — DHCP

## Interface ativa e DHCP

- **Interface:** `eth0`
- **Comando:**

```bash
ip -4 addr show eth0
ip route
```

**Evidência:**

```text
2: eth0@if12: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc noqueue state UP group default  link-netnsid 0
    inet 172.17.0.2/16 brd 172.17.255.255 scope global eth0
       valid_lft forever preferred_lft forever
default via 172.17.0.1 dev eth0
172.17.0.0/16 dev eth0 proto kernel scope link src 172.17.0.2
```

**Conclusão:** Não há `dhclient` nem NetworkManager no container. A interface é configurada pela rede bridge do Docker; portanto, não é possível afirmar, a partir de dentro do container, que um cliente DHCP tradicional esteja ativo.

## Evidências do lease

- **Arquivos consultados:** `/var/lib/dhcp` e `/run`; nenhum arquivo de lease DHCP foi encontrado.
- **Logs/mecanismo de rede:** processo PID 1 é `bash`; não há NetworkManager ou `dhclient`. O mecanismo efetivo é a atribuição de rede do Docker Engine.
- **Evidência:**

```text
find /var/lib/dhcp /run -maxdepth 3 -type f
[nenhum arquivo de lease DHCP encontrado]
```

Metadados observados no host Docker:

```text
Gateway: 172.17.0.1
IPAddress: 172.17.0.2
IPPrefixLen: 16
IPv6: não configurado
```

## Parâmetros obtidos

- **IP concedido/atribuído:** `172.17.0.2/16`
- **Gateway:** `172.17.0.1`
- **DNS:** `192.168.65.7`, conforme `/etc/resolv.conf`
- **Tempo de lease/renovação:** Não disponível dentro do container; o endereço aparece com `valid_lft forever` e é gerenciado pelo ciclo de vida do container Docker.

## Cenários de falha e sintomas

### DNS vazio ou incorreto

- **Sintomas esperados:** Resolução de nomes falha, embora o host ainda possa alcançar destinos diretamente por IP. Comandos como `getent hosts` e aplicações que usam nomes retornam erro ou timeout.
- **Evidências:** Consultar `/etc/resolv.conf`, executar `getent hosts example.com` e comparar com um teste por endereço IP.

### Gateway ausente

- **Sintomas esperados:** O host mantém a interface e talvez o endereço IP, mas não alcança redes fora da sub-rede local. Conexões externas falham enquanto destinos da LAN podem continuar acessíveis.
- **Evidências:** `ip route` sem `default`, `ip route get 8.8.8.8` sem caminho válido e falha de conexão a um destino externo.

### Lease expirado sem renovação

- **Sintomas esperados:** O endereço pode expirar ou ser removido, causando perda progressiva de conectividade; novos pedidos DHCP podem falhar e o host pode ficar sem IP, gateway ou DNS.
- **Evidências:** Logs do cliente DHCP indicando falha de renovação, ausência de lease válido e mudanças observáveis em `ip addr` e `ip route`.

## Ações de contingência

1. Configurar temporariamente endereço, rota default e DNS estáticos, usando valores aprovados para a rede e registrando a mudança para rollback.
2. Restaurar a conectividade por uma rede alternativa ou por um container/host com configuração estática, enquanto o serviço DHCP é investigado e recuperado.
