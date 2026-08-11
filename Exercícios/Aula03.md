# NAT, PAT, STUN/TURN/ICE e CGNAT

## 1. Por que um endereço `192.168.0.5` nunca aparece diretamente na internet?

Porque `192.168.0.5` é um endereço **IP privado**, reservado para uso dentro de redes locais.

Ele **não é roteável na internet pública**. Normalmente, um roteador utiliza **NAT** para que esse dispositivo consiga acessar a internet utilizando um **IP público**.

---

## 2. Diferencie Static NAT, Dynamic NAT e PAT (NAT Overload)

- **Static NAT:** associa permanentemente um IP privado a um IP público específico, geralmente na proporção de **1 para 1**.

- **Dynamic NAT:** associa temporariamente IPs privados a IPs públicos disponíveis em um conjunto chamado **pool**.

- **PAT (NAT Overload):** permite que vários dispositivos privados compartilhem um único IP público, diferenciando as conexões através dos **números de portas**.

---

## 3. Um roteador tem apenas um IP público. Como ele consegue atender 30 dispositivos ao mesmo tempo?

Utilizando **PAT (NAT Overload)**.

O roteador traduz os IPs privados dos dispositivos para o mesmo **IP público** e utiliza **números de portas diferentes** para identificar e controlar cada conexão.

---

## 4. Por que aplicativos de videochamada costumam usar STUN/TURN/ICE?

Porque esses protocolos ajudam os dispositivos a descobrir seus endereços públicos e atravessar **NATs** e **firewalls**.

- **STUN:** ajuda o dispositivo a descobrir seu endereço IP público e informações sobre o NAT.
- **ICE:** procura a melhor forma de estabelecer uma conexão entre os dispositivos.
- **TURN:** utiliza um servidor intermediário quando não é possível estabelecer uma conexão direta.

Dessa forma, aplicativos de videochamada conseguem estabelecer comunicação mesmo quando os dispositivos estão atrás de NATs ou firewalls.

---

## 5. O que é CGNAT e por que ele dificulta expor um serviço próprio à internet?

**CGNAT (Carrier-Grade NAT)** ocorre quando a própria operadora de internet coloca vários clientes atrás de um mesmo **IP público**.

Isso dificulta receber conexões externas porque o usuário não possui controle sobre o NAT realizado pela operadora.

Assim, normalmente não é possível criar um **redirecionamento de portas (Port Forwarding)** diretamente da internet até um dispositivo da rede local.

# Prática: Montando Tabelas de Tradução NAT

## Cenário A — Tradução simples

Um host `192.168.1.5:52000` acessa um site na porta `443` utilizando o IP público `201.10.5.90`.

O roteador utiliza **PAT (NAT Overload)** para substituir o endereço IP e a porta de origem por um endereço e uma porta públicos.

Podemos escolher, por exemplo, a porta pública `62000`.

### Tabela de tradução

| IP privado | Porta privada | IP público | Porta pública | Destino |
|---|---:|---|---:|---|
| `192.168.1.5` | `52000` | `201.10.5.90` | `62000` | `Servidor externo:443` |

### Tradução realizada

```text
192.168.1.5:52000
        ↓ PAT
201.10.5.90:62000
        ↓
Servidor externo:443
```

O roteador mantém essa associação em sua **tabela NAT** para conseguir encaminhar as respostas de volta para o dispositivo correto.

---

## Cenário B — Conflito de portas

Dois hosts internos diferentes tentam acessar a internet utilizando a mesma porta de origem `50000`.

Por exemplo:

```text
192.168.1.5:50000
192.168.1.6:50000
```

Como ambos utilizam o mesmo IP público, o roteador utiliza **PAT** para atribuir portas públicas diferentes para cada conexão.

### Tabela de tradução

| IP privado | Porta privada | IP público | Porta pública |
|---|---:|---|---:|
| `192.168.1.5` | `50000` | `201.10.5.90` | `50001` |
| `192.168.1.6` | `50000` | `201.10.5.90` | `50002` |

### Funcionamento

```text
192.168.1.5:50000 → 201.10.5.90:50001
192.168.1.6:50000 → 201.10.5.90:50002
```

Mesmo que os dois computadores utilizem a mesma porta de origem, o roteador evita o conflito atribuindo **portas públicas diferentes**.

Assim, quando uma resposta chega, ele consulta a tabela NAT e identifica para qual dispositivo interno deve encaminhá-la.

---

## Cenário C — Expondo um serviço

A empresa deseja disponibilizar um servidor web interno:

```text
192.168.1.20:8080
```

para que ele possa ser acessado externamente através da porta `80` do IP público:

```text
201.10.5.90:80
```

Para isso, é necessário configurar um **redirecionamento de porta (Port Forwarding / Static PAT)**.

### Regra de tradução

| IP público | Porta pública | IP privado | Porta interna |
|---|---:|---|---:|
| `201.10.5.90` | `80` | `192.168.1.20` | `8080` |

### Funcionamento

```text
Internet
   ↓
201.10.5.90:80
   ↓
Roteador / NAT
   ↓
192.168.1.20:8080
```

Quando alguém acessar:

```text
http://201.10.5.90
```

a conexão chegará ao roteador pela porta `80`.

O roteador realizará a tradução:

```text
201.10.5.90:80 → 192.168.1.20:8080
```

e encaminhará a solicitação para o servidor web interno.

Além da configuração de NAT, o **firewall deve permitir conexões TCP na porta 80**.
