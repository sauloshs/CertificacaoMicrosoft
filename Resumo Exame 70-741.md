# *Resumo do Exame 70-741*

---------

## *Comandos básicos para editar o IP através do prompt de comando*

```powershell
# Configurndo IP
Netsh interface ipv4 set adress name= "Nome da Conexão" source= Static address= 192.168.17.10 mask= 255.255.255.0 gateway= 192.168.17.1
# Configurando DNS
Netsh interface ipv4 set dns "nome da conexão" static 8.8.8.8
```

Testando conexão com outros equipamentos através do powershell:

```powershell
Test-Connection -ComputerName srv01, srv02	
```

## *Comando básico para editar o IP através do  Powershell*

```powershell
# Atibuindo IP a uma interface de rede 
New-NetIPAddress -InterfaceAlias "Local Area Conection" -IPAddress 192.168.17.10 -PrefixLength 24 -DefaultGateway 192.168.17.1
# Definindo configurações de DNS
Set-DNSClientServerAddresses -InterfaceAlias "Local Area Conection" -ServerAddresses 8.8.8.8, 8.8.4.4
```

## *Calculo de Sub-rede*

As dicas a seguir serve para ajudar a calcular mascaras e sub-rede.
Tabela para orientação de calculo de rede:

| 128  | 64   | 32   | 16   | 8    | 4    | 2    | 1    |
| ---- | ---- | ---- | ---- | ---- | ---- | ---- | ---- |
| 1    | 1    | 1    | 1    | 1    | 1    | 1    | 1    |

| 32768 | 16384 | 8192 | 4096 | 2048 | 1024 | 512  | 256  |
| ----- | ----- | ---- | ---- | ---- | ---- | ---- | ---- |
| 1     | 1     | 1    | 1    | 1    | 1    | 1    | 1    |

Identificando número de subredes
$$
2^N
$$
Onde N é o total de números positivos no octeto 
Exemplo:
$$
2^3=8
$$
Identificando o total de hosts por sub-rede
$$
2^N-2
$$
Onde N é o total de zeros no octeto e o -2 representa o identificador de rede e o broadcast .
CIDR é a representação do número da rede de forma decimal 

Exemplo: 255.255.255.0 = /24

## *DHCP*

Comandos para autorizar, desautorizar e consultar servidores DHCP no domínio

```powershell
# Autorizando um servidor 
Add-DhcpServerInDC srv01.saulo.local
# Desautorizando um servidor 
Remove-DhcpServerInDC srv01.saulo.local
# Consultando servidores
Get-DhcpServerInDC
```

### *DHCP Powershell*

```powershell
# Adcionando um novo Scopo DHCP
Add-DhcpServerv4Scope -Name "Powershell" -StartRange 192.168.16.10 -EndRange 192.168.16.254 -SubnetMask 255.255.255.0
# Consultando Scopos
Get-DhcpServerv4Scope
# Configurando um range de IP no scopo DHCP
Set-DhcpServerv4Scope -ScopeId 192.168.16.0 -StartRange 192.168.16.20 -EndRange 192.168.16.254
# Alterando o tempo de atribuição de um clinte DHCP
Set-DhcpServerv4Scope -ScopeId 192.168.16.0 -LeaseDuration 9.00:00:00
# Consultando opções do DHCP (Roteado, DNS)
Get-DhcpServerv4OptionValue -ScopeId 192.168.17.0
# Alterando ou Adcionando informações de DNS nas opções de DHCP
Set-DhcpServerv4OptionValue -ScopeId 192.168.16.0 -DnsServer 192.168.17.11 -DnsDomain saulo.local
# Adicionando uma faixa de exclusão no Scopo DHCP
Add-DhcpServerv4ExclusionRange -ScopeId 192.168.16.0 -StartRange 192.168.16.30 -EndRange 192.168.16.40
# Consultando range de Exclusão
Get-DhcpServerv4ExclusionRange
# Adcionando uma reserva ao Scopo DHCP
Add-DhcpServerv4Reservation -ScopeId 192.168.16.0 -IPAddress 192.168.16.50 -ClientId 02-00-02-02-02-56 -Description "Srvteste01"
# Consultando reservas de um Scopo especifico 
Get-DhcpServerv4Reservation -ScopeId 192.168.16.0
# Importando lista de reservas de um arquivo CSV 
import-csv -Path c:\reservas.csv | Add-DhcpServerv4Reservation
# Removendo um Scopo DHCP 
Remove-DhcpServerv4Scope -ScopeId 192.168.16.0 -Force
```

### *Níveis hierárquicos do DHCP*

O DHCP possui níveis hierárquicos que nas suas configurações que podem sobrescrever outras configurações, por exemplo: As configurações de opções definidas a níveis de servidor valem para todos os Scopos de DHCP configurado em um servidor, porem se for realizada alguma alteração das opções a níveis de Scopo elas sobrescrevem as já realizadas anteriormente a nível de servidor.
Configurações de Opções a níveis de reserva sobrescrevem configurações realizadas a níveis de Scopo de DHCP e a níveis de servidor.

### *Classe DHCP*

As Classes DHCP funcionam como politicas que são aplicas a determinas grupos assim como é o conceito de GPO.
Quando criamos uma nova Classe de usuário (Clicando com o botão direito em IPv4, Definição de Classes de usuários) como foi o exemplo apresentado na Aula devemos executar os seguintes procedimentos na máquina cliente.

```powershell
# Consultando todas as Classes definidas no servidor DHCP
ipconfig /showclassid ethernet (Nesse Exemplo ethernet é o nome do adaptador de rede)
# Atribuindo a máquina clinte a class que foi criada no servidor e atribuida a policita 
ipconfig /setclassid ethernet ClientClass (ClinetClass foi o nome da classe criada no servidor)
```

 Após realizar essa atribuição a máquina cliente recebera as definições estabelecidas na politica e que posteriormente foi vinculada a classe

### *Agente de retransmissão DHCP*

Um agente de retransmissão DHCP realiza escuta de difusões DHCP de clientes DHCP e as retransmite para servidores DHCP em sub-redes diferentes. 
Como a difusão DHCP não passa pelo roteador temos que configurar o agente de retransmissão no roteador para encaminhar os pacotes de difusão para os servidores de DHCP.

### *Super Scopo*

É um agrupamento de Scopo do meu servidor DHCP e ele deve ser usado quando o meu servidor DHCP já esgotou todos os endereços disponíveis em meu Scopo. Dessa forma agrupamos mais de um Scopo para atender a nova requisições.
Devemos lembrar de criar rotas em nosso roteador para que seja possível a comunicação entre todos os scopos mesmo estando em sub-redes diferentes.
Segue comandos para criação e exclusão de um Super scopo:

```powershell
# Adicionando um super scopo
Add-DhcpServerv4Superscope -SuperscopeName "Lab Super Scope" -ScopeId 192.168.17.0, 192.168.16.0
# Consutando um Super Scopo
Get-DhcpServerv4Superscope
# Removendo um Super Scopo
Remove-DhcpServerv4Superscope -SuperscopeName "Lab Super Scope"
```

### *Failover DHCP*

Anteriormente para garantirmos a alta disponibilidade de um servidor DHCP era necessário configurar um Cluster de DHCP ou realizar a divisão do scopo de DHCP.
Mas a partir do Windows Server 2012 surgiu o recurso Failover de DHCP que pode trabalhar das seguintes formas:

- Espera Ativa
- Balanceamento de carga

Características do Failover de DHCP

- Permitir que dois servidores DHCP forneçam endereços IP e configurações opcionais aos mesmos escopos ou sub-redes.
- Exige que as relações de failover tenha nomes exclusivos.
- Oferece suporte ao modo de compartilhamento de carga e de espera ativa.
- Quando você usa o failover de DHCP:
  - O MCLT (Prazo de entrega máximo a um cliente) determina quando um parceiro de failover assume o controle da sub-rede ou de escopo.
  - A autenticação de mensagem pode validar as mensagens de failover.
- As regras de firewall são configuradas automaticamente durante a instalação do DHCP. Caso esteja utilizando um firewall de terceiro deve liberar a porta TCP 647.

Segue comandos para gerenciar um Failover de DHCP por Powershell.

```powershell
# Adicionando um Failover de DHCP
Add-DhcpServerv4Failover -ComputerName srv01.saulo.local -Name "Failover Powershell"  -PartnerServer srv02.saulo.local -ScopeId 192.168.17.0, 192.168.16.0 -SharedSecret "P@ssw0rd"
# Consutando um Failover de DHCP
Get-DhcpServerv4Failover
# Alterando configurações de Failover de DHCP (No Exemplo a seguir faço uma alterçao do modo de balanceamento de carga para espera ativa)
Set-DhcpServerv4Failover -Name "Failolover Poowershell" -Mode HotStandby
```

## *IPv6*

### *Conceito de utilização básica*

Vantagens do uso do IPv6.

- Amplo espaço de endereços.
- Infraestrutura hiérasquica de endereçamento e roteamento.
- Configuração de endereços com e sem monitoração de estado.
- Suporte exigido para IpSec.

Comparativo IPv4 com IPv6:

|                             IPv4                             |                             IPv6                             |
| :----------------------------------------------------------: | :----------------------------------------------------------: |
|              Endereço de 32 bits de compirmento              |             Endereço de 128 bits de comprimento              |
|                  Suporte a IPSec é opcional                  |                 Suporte IPSec é obrigatório                  |
| Descoberta de roteadores ICMP, para determinar o endereço do melhor Gateway | Descoberta de roteadores ICMP, que é obrigatória é substituída pelas mensagens de solicitação e anúncios de roteadores ICMPv6 |
|     Deve ser configurado manualmente ou através de DHCP      |            Não requer configuração manual ou DHCP            |
| usa registro de recursos (A) de endereços de host no sistema DNS para mapear host em IPv4 | Usa registro de recurso (AAAA) de endereço de host no DNS para mapear nomes de host para endereços IPv6 |
| Difunde endereços que são usados para o envio de trafego para todos os nós em uma sub-rede | Não há endereços de difusão do IPv6, há um endereço multicast de todos os nós do escopo de link local |

O endereço IPv6 é formado por 8 blocos, cada bloco formado por 16 bits  totalizando 128 bits.
Exemplo: 2001:0DB8:0000:2F3B:02AA:00FF:FE28:9C5A
Simplificado: 2001:DB8:0:2F3B:2AAA:FF:FE28:9C5A

Prefixos do IPv6:

| Alocação                   | Prefixo                  |
| -------------------------- | ------------------------ |
| Loopback                   | 0:0:0:0:0:0:0:1 ou ::1   |
| Global Unicast             | Começando com 2 ou 3     |
| Link local Unicast (Apipa) | Fe80::/64                |
| Unicast Local              | FC00::  FD00::  e FE00:: |
| Endereço multicast         | FF                       |

Zone ID: 

- É um identificador único que é adicionado ao endereço link-local
- É baseado no índice de interface 
- Devem ser incluídos quando se comunica com um endereço link-local

Exemplos:
fe80::2B0:d0ff:fee9:4143 %3
fe80::243b:f351:d5b8:1aac %10

Para acessar uma pagina Web através do seu endereço IPv6 se faz necessário colocar o endereço IPv6 dentro de Colchetes http://[2001:DB8:0:2F3B:2AAA:FF:FE28:9C5A].
Para acessar um compartilhamento de rede através de um endereço IPv6 se faz necessário colocar o IPv6 da seguinte forma: \\\fd00-aaaa-bbbb-cccc--1.ipv6-literal.net. (Substituímos os dois pontos pelo sinal -).

### *Tecnologia de Transição*

As tecnologia de transição desenvolvidas são:

- ISATAP
- 6top4
- Teredo

ISATAP é uma tecnica de tunelamento que permite que a rede IPv6 se comunique com a rede IPv4 através de um roteador ISATAP.
Cada cliente ISATAP recebera um IPv4 encapsulado no IPV6.
6to4 é uma tecnica de tunelamento baseada em roteadores 6to4 porem com seu uso aplicado para a internet, diferentemente do ISATAP que tem o seu uso restrito a redes internas.
Teredo é uma tecnologia que é usada quando outras tecnologias como 6to4 não estão disponíveis. Usando NAT

Comandos para configuração de um roteador ISATAP

```powershell
# Criando uma rota no roteador ISATAP
New-NetRoute -InterfaceAlias "Ethernet 2" -DestinationPrefix 2001:db8:0:1::/64 -Publish yes
# Setando configurações no adaptador de rede
Set-NetIPInterface -InterfaceAlias "Ethernet 2" -AddressFamily IPv6 -Advertising Enabled
# Setando ISATAP Configuration
Set-NetIsatapConfiguration -Router 192.168.17.254
# Listando configurações das interfaces
Get-NetIPAddress | Format-Table InterfaceAlias, InterfaceIndex, IPv6Address
# Listando configurações do adaptador 8
Get-NetIPInterface -InterfaceIndex "8" -PolicyStore ActiveStore | Format-List
# Setando configurações no adaptador 
Set-NetIPInterface -InterfaceIndex "8" -Advertising Enabled
# Criando rota para o segundo adaptador 
New-NetRoute -InterfaceIndex "8" -DestinationPrefix 2001:db8:0:2::/64
```

## *DNS*

### *Conceitos de DNS, tipos de zonas e registros*

DNS é um sistema usado em redes TCP/IP para nomear computadores e serviços de redes. A nomeação DNS localiza computadores e serviços por meio de nomes simples. Quando um usuário insere um nome DNS em um aplicativo, os serviços DNS podem resolve-lo para informação associada a ele, como um endereço IP.
Entre os componentes da infraestrutura DNS estão?

- Servidor DNS
- Zona DNS
- Revolvedores DNS (Equipamentos que possuem o cliente DNS)
- Registro de recursos (Todos os tipos de entrada criada em um servidor DNS)

Tipos de Zona em um servidor DNS:

- Zona primária (é uma zona que pode ser atualizada diretamente nesse servidor, criar modificar)
- Zona secundária (recebe uma copia da zona secundaria e é criada apenas para equilibrar o trafego entre os servidores ou como contingencia caso um servidor primário apresente falhas)
- Zona de stub (zona que abriga apenas registros de identificação dos servidores principais, essas zonas são usadas em fusões de empresas)

OBS: Somente as zonas primarias e as zonas de stub podem ser armazenadas no AD.
Ao se criar uma nova zona de DNS são criados dois registros automáticos:

- SOA (Inicio de autoridade)
- NS (Name Server / Servidor de nome)

Como criar uma zona DNS com o Powershell

```powershell
# Criando uma nova zona primaria
Add-DnsServerPrimaryZone -Name "saulohenrique.local" -ZoneFile "saulohenrique.local.dns" -DynamicUpdate None
# Criando um registro tipo no A no DNS
 Add-DnsServerResourceRecord -ZoneName saulohenrique.local -a -Name "Filesever"  -IPv4Address 192.168.17.10
# Consultando todos os meus registro em uma zona especifica 
Get-DnsServerResourceRecord -ZoneName saulohenrique.local
```

### *Zonas inversas, encaminhadores e cache*

Um encaminhador é um servidor de sistema de nome de domínio (DNS)  de uma rede que encaminha consultas DNS sobre nomes DNS externos para servidores DNS que estão fora da rede. Você também pode encaminhar consultas de acordo com nomes de domínio específicos usando encaminhadores condicionais.
Você indica um servidor DNS de uma rede como um encaminhador ao configurar os outros servidores DNS da rede para que eles encaminhem para esse servidor DNS consultas que não consigam resolver localmente. 

Encaminhadores condicionais é uma servidor DNS de uma rede que encaminha consultas DNS de acordo com o nome de domínio DNS na consulta. Por exemplo, você pode configurar um servidor DNS para encaminhar todas as consultas recebidas sobre nomes que terminam com corp.contoso.com para o endereço IP de um servidor DNS específico ou para os endereços IP de vários servidores DNS.Comandos em Powershell para adicionar encaminhadores e encaminhadores condicionais.

```powershell
# Adicionando encaminhadores.(O parametro PassThru é utilizado para que seja exibido na tela a confirmação do comando)
Add-DnsServerForwarder -IPAddress 192.168.17.12 -PassThru 
# Adicionando encaminhadores condicionais
Add-DnsServerConditionalForwarderZone -Name "sith.local" -MasterServers 192.168.17.13 -PassThru 
```

Os computadores cliente DNS podem usar a atualização dinâmica para registrar dinamicamente seus registros de recursos em um servidor DNS sempre que houver alguma alteração. Isso reduz a necessidade da administração manal de registros de zona.
Comando em Powershell e no Prompt de comando para registrar uma máquina de forma automática no DNS.

```powershell
# Atraves do Prompt de Comando
ipconfig /registerdns
# Atraves do Powershell
Register-DnsClient
```

Atualização dinâmica segura só esta disponivel para as zonas que estão integradas ao AD DS. Al integrar uma zona ao diretório, estão disponíveis recursos de edição para a lista de controle de acesso (ACL) no gerenciador DNS para que você possa adicionar ou remover usuários ou grupos da ACL de uma zona ou de um registro de recurso específicos. 
A atualização dinâmica segura só permite o armazenamento do seu arquivo DNS no AD. 

Cache DNS tem a função de aglizar uma pesquisa por nome.
Comandos para limpar e consultar o cache nas máquinas cliente em Powershell e Prompt de Comando.

```powershell
# Listando o cache no prompet de comando
ipconfig /displaydns
# Listando o cache através do Powershell
Get-DnsClientCache
# Limpando o chahe no prompt de comando
ipconfig /flushdns
# Limpando o cache através do Powershell
Clear-DnsClientCache
# Limpando o cache no servidor Prompt de comando 
dnscmd /clearcache
# Limpando o cache no servidor Powershell
Clear-DnsServerCache
```

### *Root Hints, Zonas secundarias, stub e globalnames*

Em configurações do servidor DNS, na guia avançado podemos realizar algumas configurações avançadas no servidor:
Consultando os servidores Root Hints com o Powershell

```powershell
# Consulta
Get-DnsServerRootHint
```

- Desabilitar recursão (Também desabilita encaminhadores)
  - Essa opção desabilita o encaminhamento de solicitações DNS inclusive para os Root Hints
- Habilitar BIND secundário 
  - Essa opção é habilitada quando desejamos integrar no servidor DNS com uma zona secundaria linux. Com esse recurso ativado algumas compactações não serão habilitadas a fim de compatibilidade com o servidor BIND do linux.
- Falha no carregamento se forem dados de zona danificada
  - Ocorrendo erros no arquivo de zona ela não será carregada
- Habilitar round robin
  - Essa opção por padrão vem habilitada e serve para que possamos criar mais de um registro DNS apontando para endereços diferentes a fim de balancear a resposta de requisições. Muito utilizado em servidores 
- Habilitar classificação de máscaras de rede
  - Dara preferencia a retornar registros com a mesma mascara do cliente que fez a requisição 
- Proteger cache contra poluição
  - Impede que registros falsos sejam adicionados ao cache DNS

A eliminação de recursos obsoletos por padrão é de 7 dias para todo o servidor DNS quando é habilitado, mas podemos definir tempos por zonas.
Essas opções podem ser habilitadas através dos seguintes comandos em Powershell

```powershell
# Ativando o recurso de eliminação para uma zona especifica
Set-DnsServerZoneAging saulohenriqaue.local -Aging $true -ScavengeServers 192.168.17.11
# Alterando o tempo de autualização para eliminação em todo o servidor DNS
 Set-DnsServerScavenging -RefreshInterval 0:02:00:00 -ApplyOnAllZones -Confirm
# Consultando o tempo de Eliminação de uma zona
Get-DnsServerZoneAging -Name saulohenrique.local
```

Globalnames ele parece com o NetBins quando habilitado através do powershell podemos criar CNAME apontando para registros em outras zonas e quando clientes DNS realizarem consultas apenas com o nome do host o globalnames se encarrega de atribuir o sufixo do domínio para o registro especificado. 
Comando em Powershell

```powershell
# Consultando se o recurso esta habilitado 
Get-DnsServerGlobalNameZone
# HAbilitando recurso
Set-DnsServerGlobalNameZone -Enable $true -PassThru
# habilitando através do prompt de comando
dnscmd /confg/enableglobalnamesupport 1
```

### *Segurança para servidores DNS*

O DNSSEC funciona da seguinte maneira:

- Se uma zona foi assinada digitalmente, uma resposta de consulta conterá assinaturas digitais.
- O DNSSEC uma âncoras de confiança que são zonas especiais que armazenam chave públicas associadas a assinaturas digitais.
- Os revolvedores usam  âncoras de confiança para recuperar chaves públicas e para construir cadeias confiáveis.
- O DNSSEC requer que as âncoras de confiança sejam configuradas em todos os servidores DNS que participam de DNSSEC.
- O DNSSEC usa o NRPT que contém regras que controlam o comportamento do computador cliente solicitante no envio de consultas e no tratamento de respostas.
  - NRPT (tabela de politica de resolução de nomes)

```powershell
# Consultando se o DNSSEC esta habilitado 
Resolve-DnsName srv01.saulohenrique.local -DnssecOK -Server srv01
# verificando se o NRPT foi atribuido a uma zona DNS via GPO
Get-DnsClientNrptPolicy
```

Socket Pool faz com que o servidor DNS use portas aleatórias sempre que realizar uma consulta (A porta padrão é a 53) ele vem habilitado por padrão com 2500 portas diferentes.
podemos alterar esse numero com o seguinte comando:

```powershell
# Alternado tamanho do pool de portas
dnscmd /config /socketpoolsize 4000
# Consultando o número de portas
dnscmd /info /socketpoolsize
```

Bloqueio de Cache é usado para controlar quando as informações do cache podem ser substituídas, por padrão essa configuração é definida em 100% que significa que ela pode ser excluída pelo TTL do registro. 
Comando do Powershell para confirmação da informação:

```powershell
# Confirmando o tempo de cache do servidor
Get-DnsServerCache
# Confirmando através do prompt 
dnscmd /info /cachelockingpercent
```

Através do powershell podemos alterar o tempo de bloqueio de exclusão de um registro para que um usuário mal intencionado não consiga fazer essa alteração até que passe pelo menos o tempo que determinamos. No exemplo a segui vamos configurar o mesmo para 50%

```powershell
# Alterando o tempo de bloqueio de cache
Set-DnsServerCache -LockingPercent 50
# Comando equivalente no prompt de comando
dnscmd /config /cachelockingpercent 50
```

Para visualizar os logs de eventos do DNS devemos acessar o Visualizador de Eventos e ir até a opção aplicativos.
Ou podemos usar o Log de depuração que vem desabilitado por padrão.

Limitar a taxa de resposta é um recurso que por padrão vem desabilitado e serve para limitar o número de requisição a um servidor DNS. Para habilitar esse recurso usamos o seguinte comando em Powershell:

```powershell
# Limitando a taxa de resposta de um servidor DNS
set-dnsServerResponseRateLimiting -mode Enable
# Consultando a taxa de negação de servidor DNS
Get-DnsServerResponeRateLimiting
```

Recursividade Seletiva (autoritária) o servidor DNS poderá realizar consultas recursiva somente para um grupo exclusivo de servidores.
Comandos no powershell

```powershell
# Verificar se a opção de recursividade esta habilidada 
Get-DnsServerRecursionScope
# Deabilitando a opção de recursividade 
Set-DnsServerRecursionScope -Name . -EnableRecursion $False
# Criando um Scope de recursão
Add-DnsServerRecursionScope -Name "ClientesInternos" -EnableRecursion $True 
# Politica de recursão 
Add-DnsServerQueryResolutionPolicy -Name "ControleDeRecursao" -Action ALLOW -ApplyOnRecursion -RecursionScope "ClientesInternos" -ServerInterfaceIP "EQ,192.168.17.11"
```

## *IPAM*

### *Instalação e configuração*

O **IPAM** (Gerenciamento de Endereço IP) no Windows Server é um conjunto integrado de ferramentas que permitem o planejamento e o monitoramento ponta a ponta de sua infraestrutura de endereços IP, com uma rica experiência de usuário. O IPAM descobre automaticamente os servidores de infraestrutura de endereços IP na rede e permite que você os gerencie com base em uma interface central.
O IPAM consiste em quatro módulos que oferecem as seguintes funcionalidades:

- Descoberta de IPAM
- Gerenciamento de espaço de endereço
- Gerenciamento e monitoramento multiservidor 
- Auditoria operacional e acompanhamento de endereço IP

Ao implantar o IPAM, é possível selecionar três topologias:

- Distribuída (Com um servidor IPAM para cada site da floresta)
- Centralizada (Faz a centralização de toda floresta em um único servidor IPAM)
- Híbrida (Combinamos servidores centralizados com servidores distribuídos)

**Requisitos para implantação do IPAM**
Para garantira uma implementação de IPAM com sucesso, uma infraestrutura de rede da organização deve atender aos seguintes pré-requisitos:

- O servidor de IPAM deve ser um membro do domínio 
- O servidor de IPAM de ser um servidor de finalidade única 
- Para gerenciar o espaço do endereço IPV6, habilite IPV6 no servidor de IPAM
- Entre no servidor de IPAM com uma conta de domínio 
- Pertencer ao grupo de segurança local do IPAM correto no servidor de IPAM
- Habilitar o registro em log de eventos de conexão de conta para o recurso de rastreamento e auditoria de endereço IP do IPAM 
- Atender aos requisitos de software e hardware

**Durante a criação de uma implantação do IPAM, considere os seguintes fatores:**

- Você pode gerenciar diversas florestas AD DS se existir a confiança necessária entre elas.
- Os servidores do IPAM não se comunicam entre si
- Você pode definir o escopo da descoberta de um subconjunto de domínio na floresta 
- Um único servidor do IPAM pode comportar diversos servidores DHCP e zonas DNS (150 servidores DHCP, 500 servidores DNS, 6000 escopos DHCP e 150 zonas DNS)
- O IPAM armazena três anos de dados forenses 
- O IPAM comporta os bancos de dados WID ou SQL Server
- As tendências de uso do endereço IP são fornecidas somente para IPV4
- O suporte de reclamação de endereço IP é fornecido somente para IPV4 
- O IPAM não verifica a consistência do endereço IP com roteadores e switches 

O recurso **ASM** (Gerenciamento de espaço de endereço) do IPAM permite que você ganhe visibilidade em todos os aspectos da sua infraestrutura de endereços IP, em um único console. Com IPAM, você pode criar uma hierarquia multinível de endereços na rede usa-lá  para gerenciar endereços IPV6 e IPV4 público ou privado. 
Os principais recursos são:

- Gerenciamento integrado do espaço de endereço IP dinâmico e estático 
- Detecção e gerenciamento de conflitos, sobreposições e duplicatas no espaço de endereço dos sistemas
- Exibição de inventário altamente personalizada do espaço de endereço IP
- Monitoramento centralizado e relatórios de estatísticas e tendências do uso de endereços
- Suporte para IPV4 e monitoramento de uso de endereço IPV6 sem monitoração de estado
- Descoberta automática de intervalos de endereço IP a partir de escopos DHCP
- Exportação e importação de endereços IP e intervalos de endereços IP com suporte ao Windows Powerhell
- Alertas de uso de endereço IP e notificações com limites personalizados
- Detecção e atribuição de endereços IP disponíveis

O IPAM a partir do Windows Server 2012 r2 inclui a capacidade de gerenciar o espaço de endereço IP virtual usando o System Center Virtual Machine Manager (VMM)
O recurso **VASM** (Gerenciamento de espaço de endereço virtual) permite as mesmas funções e capacidade para sua infraestrutura de endereço IP virtual que o recurso ASM.

O recurso **MSM** (Gerenciamento de Multisservidor) do IPAM permite que você descubra automaticamente os servidores DNS e DHCP na rede, monitore a disponibilidade do serviço e gerencie de forma centralizada sua configuração. 
Os principais recursos são:

- Descoberta automática dos servidores DHCP e DNS da Microsoft através de uma floresta do Active Directory
- Adição ou remoção manual de servidores gerenciados 
- Configuração e gerenciamento completos de servidores e escopo DHCP
- Suporte a construção avançadas para permitir adicionar, sobrescrever ou localizar e substituir operações em vários âmbitos e servidores DHCP
- Atualização simultânea de configurações comuns em vários escopos DHCP ou servidores DHCP
- Disponibilidade para monitorar serviços DHCP e DNS e zonas DNS
- Gerenciamento de servidores DHCP e DNS da Microsoft  que executam o Windows 2008 ou sistemas operacionais mais recentes.
- Adição de informações personalizadas ao servidores, permitindo a visualização usando grupos lógicos com base na lógica de negócios 
- Monitoramento do uso do escopo DHCP
- Recuperação automática e sob demanda de dados do servidor com base em servidores DHCO e DNS gerenciados 
- Monitoramento do status de zona DNS com base em eventos da zona DNS
- Classifique servidores e funções descobertos como gerenciados ou não

Durante a configuração do IPAM devemos selecionar o tipo de banco de dados e se escolhermos WID essa configuração só pode ser alterada através do comando abaixo:

```powershell
Move-IpamDatabase e Set-IpamDatabase
```

Após definir o modo de configuração baseado em GPO devemos criar em cada domínio gerenciado do IPAM GPOs de provisionamento usando o seguinte comando:

```powershell
Invoke-IpamGpoProining
# Usando comando em um controlador de domínio gerenciado do IPAM
 Invoke-IpamGpoProvisioning -Domain saulo.local -DomainController srv01.saulo.local -GpoPrefixName IPAM -IpamServerFqdn srv02.saulo.local -DelegatedGpoUser saulo
# EM resumo esse comando pegou a GPO que foi criada no servidor IPAM e adicnioa no controlador de dominio... Apos esse procedimento voltamos ao console do IPAM e atulizamos as configurações do servidor para status gerenciado.  
```

Caso posteriormente seja necessário alterar do método de manual  para GPO podemos usar o comando em powershell abaixo:

```powershell
Set-IpamConfiguration -Provisioning-Method
```

Comandos básicos em powershell para utilizar no IPAM:

```powershell
# Comando para adicionar um blobo de endereços ips no IPAM
Add-IpamBlock
# Modificar um bloco de endereços IPAM 
Set-IpamBlock
# Criando um intervalo de endereços IPs (Range)
Add-IpamRange
# Editando o range de IPs
Set-IpamRange
```

```powershell
# Comando para importar Blocos de endereços IPs, endereços IPs e intervalo de endereços IPs
Import-IpamRange / Import-IpamAddress / Import-IpamSubnet
```

 

```powershell
# Consultar servidores DHCP gerenciados pelo IPAM
Get-IpamDhcpServer
# Consultar escopos DHCP no IPAM
Get-IpamDhcpScope -AddressFamily IPv4
# Busca Eventos de confuguração
Get-IpamDhcpConfigurationEvent
```

```powershell
# Consultar servidores DNS gerenciados pelo IPAM
Get-IpamDnsServer
# Consultar zonas de pesquisas diretas 
Get-IpamDnsZone -ZonaType Forward
# Consultar registros da zona especifica
Get_IpamDnsResourceRecord -ZoneName Saulohenrique.local 
```

## *NAT*

NAT permite a implementação de um esquema de endereçamento de IPv4 privado em sua organização, e também que usuários, aplicativos e serviços acessem a internet. NAT é um dispositivo ou um serviço de software que permite aos computadores de sua organização acessarem recursos da internet, transformando endereços IPv4 privados de sua intranet em endereços IPv4 públicos na internet.
Todos os dispositivos que conectam a internet exigem um endereço IPv4 público exclusivo. No entanto, no espaço de endereços IPv4 não existem endereços públicos suficientes para todos os dispositivos que exigem esse tipo de conexão. Como resultado as empresas usam endereços IPv4 privados para os dispositivos dentro de suas intranet.

## Direct Acess

É uma função do Windows Server que serve para promover acesso remoto a um cliente sem que seja necessário a configuração de uma VPN.
**Opões de implantação do servidor de DirectAcess:** 

- Implantação simples usando o assistente do guia de introdução.
- Implantação complexa usando as opções avançadas de configuração.

**Opções avançadas de implantação do servidor de DirectAcess:** 

- Implantar vários pontos de extremidade.
- Suporte a vários domínios.
- implantar um servidor atrás de um dispositivo NAT.
- Suporte para OTP e cartões inteligentes virtuais.
- Suporte para agrupamento NIC.
- Provisionamento fora do local.

Comandos básicos para verificação.

```powershell
# Verificação do Diect Acess 
Get-DAClientExperienceConfiguration	
# Comando para verificar se o clinete esta usando o IPHttps
Get_NetIPHttpstate
# Para resolução de conectividade do direct acess 
Get-NetIPhttpsConfiguration
# Comando utilizadao para validar as configurações existentes na NRPT
Get-DnsClientNrptPolicy
```

```powershell
# Comando para verificar os serviços do DirectAcess
Get-RemoteAccessHealth
# Estatisticas de uso por usuários.
Get-RemoteAccessConnectionStatistics
```

## *VPN*

### *Conceito*

É uma conexão entre um computador remoto e um ambiente empresarial. Além de computadores podemos conectar filias através da conexão S2S.

### *VPN site a site* 

- Conecta duas partes de uma rede privada.
- O roteador de chamada (o cliente VPN) se autentica para o roteador de resposta (o servidor VPN)
- Requer que você criar uma interface de discagem por demanda.
- Você pode criar três tipos de VPNs site a site no Windows Server:
  - PPTP
  - L2TP
  - IKEv2
- Você pode controlar o tráfego usando filtros IP de discagem por demanda ou persistente. 
  - O filtro de demanda ele fara com que a conexão VPN seja iniciada quando seja necessário.
  - O filtro de persistência a conexão é continua e caso ocorra uma desconexão a conexão é restabelecida novamente. 

### *Características:*

- **Autenticação** ajuda a garantir que o cliente e o servidor VPN possam se identificar um com o outro. É possível escolher entre vários métodos de autenticação diferentes, dependendo do protocolo VPN selecionado e de outros fatores da infraestrutura.
- **Criptografia** como dados privados são roteados em uma rede pública, é importante garantir a segurança desses dados em trânsito. A criptografia de dados é usada para essa finalidade.
- **Encapsulamento** uma VPN faz o roteamento de dados em meio de uma rede pública, usando protocolos de tunelamento. Os dados privados são encapsulados em uma estrutura, com um cabeçalho público contendo as informações de roteamento apropriadas, as quais podem transitar em uma rede pública e chegar ao destino privado correto.

### *Opções de protocolos de túnel de VPN*

| **Protocolo de túnel** | **Acesso firewall**                                   | **Descrição**                                                |
| ---------------------- | ----------------------------------------------------- | ------------------------------------------------------------ |
| PPTP                   | Porta TCP 1723                                        | Fornece confidencialidade de dados, mas não integridade dos dados ou autenticação de dados. |
| L2TP/IPsec             | Porte UDP 500, porta UDP 4500 e ID de protocolo IP 50 | Usa certificado ou chaves pré-compartilhadas para autenticação; é recomendável a autenticação de certificado |
| SSTP                   | Porta TCP 443                                         | Usa SSL para fornecer confidencialidade de dados, mas não integridade dos dados e autenticação de dados |
| IKEv2                  | Porta UDP 500                                         | Dá suporte aos algoritmos de criptografia mais recentes do IPsec para fornecer confidencialidade dos dados e autenticação de dados |

***Opções de autenticação de VPN* **

| **Protocolo** | **Descrição**                                                | **Nível de segurança**                                       |
| ------------- | ------------------------------------------------------------ | ------------------------------------------------------------ |
| PAP           | Usa Senhas de texto não criptografado. Em geral, é utilizado se o cliente de acesso remoto e o servidor de acesso remoto não podem negociar uma forma de validação mais segura. | O protocolo de autenticação menos seguro. Não protege contra os ataques por repetição, representação do cliente remoto ou representação do servidor remoto. |
| CHAP          | Um protocolo de autenticação de desafio/resposta que usa o esquema de hash MD5 padrão do setor. | Um avanço em relação ao PAP, uma vez que a senha não é enviada através do link PPP. |
| MS-CHAPv2     | Uma atualização do MS-CHAP. Fornece uma autenticação bidirecional, também conhecida como autenticação mútua. O cliente de acesso remoto recebe a confirmação de que o servidor de acesso remoto para o qual está discando tem acesso à senha do usuário. | Fornece um nível de segurança mais alto do que o CHAP        |
| EAP           | Permite a autenticação arbitrária de uma conexão de acesso remoto através do uso de esquemas de autenticação, conhecidos como tipos de EAP. | Oferece o nível de segurança mais forte, fornecendo o máximo de flexibilidade na variações da autenticação. |

### *O que é reconexão VPN?* 

- Reconexão VPN mantém a conectividade nas interrupções de rede
- Reconexão VPN:
  - Fornece conectividade VPN direta e consistente.
  - Usa a tecnologia IKEv2.
  - Restabelece automaticamente conexão VPN quando a conectividade está disponível.
  - Mantém a conexão se os usuários se movem entre redes diferentes.
  - Fornece status de conexão transparente aos usuários.

### *VPN disparado pelo aplicativo*

- O VPN disparado pelo aplicativo permite que um aplicativo dispare um perfil VPN automaticamente. 
- Você configura o VPN disparado pelo aplicativo usando o cmdlet ***AddVpnConnectionTriggerAplication*** do powershell.
- Computadores membros do domínio não dão suporte a VPN disparado pelo aplicativo.
- A VPN disparada pelo aplicativo requer que você habilite a divisão de túnel para o perfil VPN.

### *Opções de modificação de configuração de VPN*

Para configurar sua solução VPN, talvez você precise:

- Configurar filtros de pacote estático.
- Configurar serviços e portas.
- Ajustar níveis de log dos protocolos de roteamento.
- Configurar o número de portas VPN disponíveis.
- Criar um perfil do gerenciador de conexão para usuários.
- Adicionar AD CS.
- Aumentar segurança de acesso remoto.
- Aumentar segurança de VPN.
- Implementar reconexão VPN.

***O CMAK***

- Permite personalzar a experiência de conexão remota do usuário, criando conexões predefinidas em servidores remotos e redes.
- Cria um arquivo executável que pode ser executado em um computador cliente para estabelecer uma conexão de rede que você criou.
- Reduz solicitações de suporte técnico relacionadas à configuração de conexões RAS:
  - Auxiliando na solução de problemas porque a configuração é conhecida.
  - Reduzindo a probabilidade de erros dos usuários quando eles configurarem os próprios objetos de conexão.

## *NPS*

Um servidor de políticas de rede do Windows Server 2016 fornece as funções a seguir:

- Servidor RADIUS. O servidor de políticas de rede realiza a autenticação, a autorização e a contabilização de conexões centralizadas, para conexões sem fio, de comutação, de autenticação, bem como dial-up e VPN.
- Proxy RADIUS. Você configura as políticas de solicitação de conexão que indiquem quais solicitações de conexão o servidor de políticas de rede encaminhará para outros servidores RADIUS, e os servidores RADIUS a os quais essas solicitações serão encaminhadas. 
- O servidor de políticas de rede oferece suporte às políticas que gerenciam e controlam conexões de clientes de acesso remoto.
- Existem dois tipos de políticas:
  - Políticas de solicitação de conexão:
    - Utilizado quando o servidor de políticas de rede deve atuar como um servidor ou proxy RADIUS.
  - Políticas de rede:
    - Usado para autenticar e autorizar a tentativa de conexão.
- Você define as condições e as restrições para controlar o acesso.
- Ao implantar o servidor de políticas de rede pela primeira vez, o acesso remoto é negado, sendo necessário configurar pelo menos uma política para permitir o acesso.













