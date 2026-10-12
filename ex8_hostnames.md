# Exercício 8 — Hostnames

## Estado atual

### Hostname

```bash
hostname
```

**Evidência:**

```text
deaf506ee7a6
```

### `hostnamectl` ou equivalente

```bash
hostnamectl
```

**Evidência:**

```text
bash: hostnamectl: command not found
```

Como alternativa equivalente no container:

```bash
cat /etc/hostname
```

```text
deaf506ee7a6
```

## Convenção proposta

- **Padrão:** `dr4-<usuario>-<papel>`
- **Hostname escolhido:** `dr4-lab-rede`
- **Justificativa:** O padrão identifica o bloco (`dr4`), o contexto do laboratório (`lab`) e a função principal (`rede`). O nome é curto, legível e facilita a correlação com logs e inventário. A convenção também pode ser estendida para outros papéis, como `dr4-lab-dns` ou `dr4-lab-router`.

## Aplicação do novo hostname

```bash
hostname dr4-lab-rede
```

**Evidência:**

```text
hostname: you must be root to change the host name
```

O comando falhou porque o container não possui a capacidade necessária para alterar o hostname em execução. Em Docker, a forma persistente é definir o hostname na criação do container:

```powershell
docker run -dit --name tp1-redes-hostname-lab --hostname dr4-lab-rede ubuntu:24.04 bash
```

**Evidência no container de laboratório:**

```text
dr4-lab-rede
dr4-lab-rede
```

Os valores correspondem às saídas de `hostname` e `cat /etc/hostname`.

## Verificação de `/etc/hosts`

- **Necessidade de ajuste:** Não no container principal; o Docker gerencia a entrada automaticamente.
- **Ação realizada:** Foi consultada a entrada do hostname original.
- **Evidência:**

```text
172.17.0.2	deaf506ee7a6
```

## Reversão

- **Hostname original:** `deaf506ee7a6`
- **Comando de reversão:**

```powershell
docker rm -f tp1-redes-hostname-lab
```

**Evidência da reversão:**

```text
deaf506ee7a6
deaf506ee7a6
```

Após remover o container de laboratório, o container principal voltou a apresentar `deaf506ee7a6` tanto em `hostname` quanto em `/etc/hostname`.
