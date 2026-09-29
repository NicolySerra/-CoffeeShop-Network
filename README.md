
#  CoffeeShop-Network
Projeto de implementação e segmentação de uma rede corporativa para cafeteria  Projeto acadêmico/laboratorial desenvolvido no Cisco Packet Tracer, simulando a infraestrutura de rede de uma pequena cafeteria, desde a configuração inicial dos equipamentos até a implementação de segmentação, endereçamento, serviços de rede e controles de acesso.

# 🎯 Objetivo do projeto

O objetivo foi projetar e configurar uma pequena infraestrutura de rede para uma cafeteria, buscando separar os diferentes tipos de usuários e dispositivos da organização.

A rede foi estruturada para atender principalmente três necessidades:

🧑‍💼 Área administrativa — computadores e dispositivos utilizados pela gestão;
💳 Ponto de venda (POS) — equipamentos responsáveis pelas operações de atendimento;
📶 Rede Wi-Fi para visitantes — acesso à internet para clientes, sem acesso aos recursos internos da cafeteria.

Além disso, foi criada uma rede específica para gerenciamento da infraestrutura.

A proposta foi aplicar conceitos de redes utilizados em ambientes reais, priorizando organização, segmentação, controle de acesso e facilidade de gerenciamento.

#   🗺️ Topologia da rede
                    ☁️ ISP
                     │
                 Cisco 2911
                     │
                   TRUNK
                     │
             Cisco 3560 Switch
             ┌───────┼────────┐
             │       │        │
          VLAN 10  VLAN 20  VLAN 30
          Gestão     POS    Guest Wi-Fi
             │       │        │
          PCs/      POS     Laptops
        Impressoras

A infraestrutura foi montada utilizando:

1 × Router Cisco 2911
1 × Switch Cisco 3560
1 × Access Point
Computadores
Impressoras
Laptops para usuários da rede Guest
Conexão simulada com ISP

A comunicação entre o roteador e o switch utiliza trunk, permitindo o transporte das diferentes VLANs através de uma única conexão física.

# 🧩 Segmentação da rede

Um dos principais pontos do projeto foi evitar que todos os dispositivos permanecessem dentro da mesma rede.

Foram criadas quatro VLANs:

| VLAN | Nome               | Rede              | Função                          |
| ---: | ------------------ | ----------------- | ------------------------------- |
|   10 | MANAGEMENT_OFFICE  | `192.168.10.0/24` | Administração                   |
|   20 | POS                | `192.168.20.0/24` | Ponto de venda                  |
|   30 | GUEST_WIFI         | `192.168.30.0/24` | Visitantes                      |
|   99 | NETWORK_MANAGEMENT | `192.168.99.0/24` | Gerenciamento da infraestrutura |


Essa separação permite aplicar diferentes regras de acesso de acordo com o tipo de dispositivo ou usuário.

Por que isso importa?

Imagine que um cliente entre na cafeteria, conecte o notebook ao Wi-Fi e tente acessar uma impressora ou computador utilizado pelo caixa.

Sem segmentação, dependendo da arquitetura da rede, esse tráfego poderia alcançar outros dispositivos internos.

Com VLANs, cada grupo passa a funcionar como uma rede lógica separada.

VLAN = separar logicamente uma rede física em diferentes redes.

# 🌐 Endereçamento IP

Foi utilizado endereçamento IPv4 privado com máscara /24.

|    VLAN | Gateway        |
| ------: | -------------- |
| VLAN 10 | `192.168.10.1` |
| VLAN 20 | `192.168.20.1` |
| VLAN 30 | `192.168.30.1` |
| VLAN 99 | `192.168.99.1` |


O roteador foi configurado utilizando subinterfaces, permitindo que diferentes VLANs utilizassem o mesmo enlace físico entre o switch e o roteador.

Exemplo:

GigabitEthernet0/1.10
        ↓
VLAN 10
192.168.10.1/24

GigabitEthernet0/1.20
        ↓
VLAN 20
192.168.20.1/24

GigabitEthernet0/1.30
        ↓
VLAN 30
192.168.30.1/24

GigabitEthernet0/1.99
        ↓
VLAN 99
192.168.99.1/24

Esse modelo é conhecido como Router-on-a-Stick.

# 🔀 Configuração das VLANs

No switch foram criadas as VLANs:

VLAN 10 → MANAGEMENT_OFFICE
VLAN 20 → POS
VLAN 30 → GUEST_WIFI
VLAN 99 → NETWORK_MANAGEMENT

As portas destinadas aos dispositivos finais foram configuradas como Access Ports, enquanto o enlace entre switch e roteador foi configurado como Trunk.

Exemplo conceitual
PC Administrativo
       │
   Access Port
       │
    VLAN 10

Enquanto:

Switch
   │
   │ TRUNK
   │ VLAN 10,20,30,99
   │
Router

A configuração foi validada utilizando comandos como:

show vlan brief

e

show interfaces trunk

Os prints do projeto mostram a criação das VLANs e a verificação de sua associação às portas.

# 📡 Wi-Fi para visitantes

Foi configurado um Access Point dedicado à rede de visitantes.

O SSID utilizado foi:

GUEST WIFI

Essa rede foi associada à:

VLAN 30
GUEST_WIFI
192.168.30.0/24

A ideia é simples:

Cliente conectado ao Wi-Fi ≠ funcionário conectado à rede interna.

Os dispositivos Guest recebem endereços da rede 192.168.30.0/24, enquanto os recursos administrativos permanecem em redes separadas.

# 📦 DHCP

Para evitar a necessidade de configurar manualmente o endereço IP de cada dispositivo, o roteador foi configurado como servidor DHCP.

Foram criados pools para:

MANAGEMENT_OFFICE
POS
GUEST_WIFI

Também foram configurados endereços excluídos do DHCP, preservando determinados endereços para gateways e equipamentos que poderiam utilizar IPs reservados.

Exemplo:

192.168.10.1 → Gateway
192.168.10.2 – 192.168.10.20 → Reservados

O mesmo princípio foi aplicado às redes:

192.168.20.0/24
192.168.30.0/24
 
O projeto também configurou o DNS:

8.8.8.8

# 🔐 Controle de acesso — ACL

Aqui está uma das partes mais interessantes do projeto para colocar no portfólio.

Foi criada uma Extended ACL chamada:

GUEST_RESTRICTIONS

A finalidade foi impedir que a rede Guest acessasse determinados segmentos internos.

Regra conceitual
GUEST_WIFI
192.168.30.0/24
        │
        ├── ❌ VLAN 10 – Administração
        │
        ├── ❌ VLAN 20 – POS
        │
        ├── ❌ VLAN 99 – Gerenciamento
        │
        └── ✅ Outros destinos permitidos

A ACL foi aplicada à interface/subinterface correspondente à rede Guest.

Isso transforma a segmentação em algo mais do que apenas uma organização lógica: existem regras explícitas controlando o que os visitantes podem alcançar.

# 🛡️ Hardening básico dos equipamentos

Também foram aplicadas algumas configurações básicas de segurança e administração nos equipamentos Cisco.

Entre elas:

no ip domain-lookup

Evita que comandos digitados incorretamente sejam interpretados como nomes DNS.

Também foram utilizados:

enable secret
service password-encryption

Além de:

banner motd

para apresentar um aviso de acesso autorizado.

Nas linhas de console também foram configuradas autenticação e sincronização das mensagens:

line console 0
password ...
login
logging synchronous

# 🧪 Testes e validação

Depois da implementação, foram realizados testes para verificar se a rede estava funcionando conforme planejado.

Teste de conectividade

Foram utilizados comandos ping, por exemplo:

ping 192.168.10.1

e:

ping 192.168.10.10

Os testes apresentaram respostas, demonstrando conectividade entre o dispositivo de teste e os respectivos destinos.

Também foi utilizado:

show ip dhcp pool

para verificar os pools DHCP.

E:

show ip interface brief

para verificar o estado das interfaces e subinterfaces do roteador.



# 💡 O que este projeto demonstra

Mais do que simplesmente configurar equipamentos, o projeto permitiu aplicar conceitos importantes de infraestrutura de redes:

# Networking

IPv4
Subnetting
VLAN
Access Port
Trunk
Inter-VLAN Routing
Router-on-a-Stick
DHCP
DNS
Wi-Fi

# Segurança

Segmentação de rede
Controle de acesso com ACL
Isolamento da rede Guest
Hardening básico
Controle de acesso ao equipamento

# Troubleshooting

Testes com ping
Verificação de interfaces
Validação de VLANs
Validação de DHCP
Verificação de ACL
Análise da configuração com show running-config
