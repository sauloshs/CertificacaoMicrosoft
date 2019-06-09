# ***Resumo Exame 70-740*** 

------


## *Indrodução ao Windows Server* 

### *Requisitos minimos*

- Processor: 1,4-GHz 64 btts 
- RAM: 512 MB ECC para Server Core, 2 GB ECC para Servidor com Desktop Experience 
- Espaço em disco: Minimo de 32 GB en uma unidade SATA ou copmpativel
- Adaptador de rede: Ethernet, co Throughout em gigabits
- Monitor: Super VGA (1024 x 768) ou resolução superior 
- Teclado e mouse (ou outro dispositivo apontador compativel)
- Acesso à internet

### *Versões*  

- Windows Server 2016 Datacenter 
  - Número ilimitado de ambientes de sistema operacional ou conteiner do Hyper-V. Recursos não disponiveis nas demais ediçoes: Storage Spaces Direct, Storage Replica, máquina virtual blindada e uma nova pilha de rede com mais opçoes para virtualização.
- Windows Server 2016 Standart 

  - Permite dois OSEs (Ambiente de sistema operacional) e os mesmos recursos basicos da versão Datacenter, ela só não tem os novos recursos de armazenamento e rede listados na descrição anterior.
- Windows Server 2016 Essentials 
  - Inclui quase todos os recursos das ediçoes Standart e Datacenter, mas não inclui  a opção de instalação server core. Restrita a um unico OSE (fisico ou virtual) e um maximo de 25 usuarios e 50 dispositivos.


- Windows Server 2016 Multipoint Premium Server

  - Disponivel apenas por meio de licenciamento academico, permite que vários usuários acessem a mesma instalação de servifor.


- Windows Server Storage Server 2016 Server

  - Disponivel apenas por canais OEM (original equipment manufacture, fabricante original de equipamento), a edição Storage Server é incluida como parte de uma solução de hardware de  armazenamento dedicada.


- Windows Hyper-V Server 2016

  - Disponivel gratuitamente, Não possui interface grafica e server para hospedar máquinas virtuais.


## *Ativação do Windows Server 2016* 

- Chave de ativação multipla (MAKs)
  - Basicamente você tem uma unica chave que serve para todos os computadores, desqde que nao atinja o limite para o uso de KMS. 
  - Caso seja necessario adicionar mais licençar posteriomente as novas licenças são incluidas na novas MAK.
  - A MAK é ideial para implantações atraves de imagens já ativadas ou pode ser unsada em um script que ativa todas as novas máquina da rede.

- Serviço de gerenciamento de chaves (KMS)

  - É um aplicativo cliente servidor que permite que computadores clientes ativem seues sistemas operacional, os clientes não precisa ter acesso a internet mais o servidor que execulta o KMS sim.
  - a limitação minima para uso do KMS é de 25 estações de trabalho ou 5 servidores. 
  - Máquina clientes execultando sistema operacional Enterprise por padrão usa o KMS como ativação, porem máquinas clientes implantadas com outro tipo de licenciamento deve ser fornecidas as GVLKs (Chaves impogenéricas de licenciamento por volume) dessa forma ele se torna cliente KMS, pois não é a chave do host cliente que sera validade e sim a do Servidor KMS.
  - Deve ser liberado no firewall do servidor a porta 1688.
  - A ativação KMS pode ser realizada atraves do modo convencional adcionando a função server KMS ou atraves do Active Directory. Através do Active Directory você precisa de pelo menos um controlador de dominio execultando windows server 2016, 2016 ou 2012 r2 e nivel funcional do AD DS no minimo de windows server 2012.

- AVMA

  - Máquinas virtuais podem ser ativas automaticamente a partida das versões Windows Server 2012 r2 Datacenter e Windows Server 2016 Datacenter.

  - Procedimento execultado na máquina virtual no prompt de comando com privilegio de administrador.

    ```
    slmgr /ipk AVMAkey
    ```

    O valor da variavel AVMAkey vai depender do sistema operacional que estiver sendo execultado na máquina virtual.

    | Edição     | Windows Server 2016           | Windows Server 2012 r2        |
    | ---------- | ----------------------------- | ----------------------------- |
    | Datacenter | TMJ3Y-NTRTM-FJYXT-T22BY-CWG3J | Y4TGP-NPTV9-HTC2H-7MGQ3-BD4TW |
    | Standart   | C3RCX-M6NRP-6CXC9-TW2F2-4RHYD | DBGBW-NPF86-BJVTX-K3WKJ-MTB6V |
    | Essentials | B4YNW-62DX9-W8V6M-82649-MHBKQ | K2XGM-NMBT3-2R6Q8-WF2FK-P36R2 |


## Implanatando o Nano Server

Importando modulos da imagem do sistema operacional.

```powershell
Import-Module D:\NanoServer\NanoServerImageGenerator\NanoServerImageGenerator.psm1
```

Criando uma imagem basica do Nano Server com as funções de file server e ISS instaldada.

```powershell
New-NanoServerImage -Edition <Datacenter / Standart> -MediaPath d:\ -BasePath c:\nano -targetPath c:\nano\nano-srv.vhdx  -DeploymenType <Guest / host> -ComputerName nano-srv01 -Storage -package microsoft-nanoserver-iis-package -DomainName shs.local (Caso queira colocar no Dominio)
```

Depois que uma imagem for criada podemos incluir pacotes com o comando Edit-NanoServerImage usando como base o diretorio especificado no atributo -BasePath quando realizamos a criação da imagem.

```powershell
Edit-NanoServeImage -BasePath C:\nano -TargetPath C:\nano\nano-srv.vhdx -packages microsoft-nanoserver-DNS-Package
```

Implantando uma imagem do Nano Server com as configurações de rede.

```powershell
New-NanoServerImage -Edition Standard -DeploymentType Guest -MediaPath E:\ -BasePath d:\nano -TargetPath d:\nano\nano-srv03.vhdx -ComputerName nano-srv3 -Storage -InterfaceNameOrI
ndex Ethernet -Ipv4Address 192.168.17.19 -Ipv4SubnetMask 255.255.255.0 -Ipv4Gateway 192.168.17.1 -Ipv4Dns 192.168.17.11
```

### *Pacotes de Instalação*

Para instalar os pacotes adicionais fornecidos com o Nano Server, como os que contêm funções e recursos, você pode incluir parâmetros opcionais à linha de comando de New-NanoserverImage. Os parâmetros opcionais co cmdlet New-NanoServerImage são os seguintes:

- -Compute: instala a função do Hyper-V.
- -Clustering: Instala a função Failover Cluster.
- -OEMDrivers: Adiciona os drives básicos incluídos na opção Server Core.
- -Storage: Instala a função File Servere outros componentes de armazenamento.
- -Defender: Instala o Windows Defender.
- -Containers: Instala o suporte do host a contêineres.
- -Packages: Instala um ou mais dos seguintes pacotes.
  - Microsoft-NanoServer-DSC-Package: Instala o pacote DSC.
  - Microsoft-NanoServer-DNS-Package: Instala a função DNS Server.
  - Microsoft-NanoServer-IIS-Package: Instala a funçã IIS.
  - Microsoft-NanoServer-SCVMM-Package: Instala o agente do System Center Virtual Machine Manager.
  - Microsoft-NanoServer-SCVMM-Compute-Package: Instala a função Hyper-V na imagem especificada pela variavel -TargetPach, para que ela possa ser gerenciada com  o System Center Virtual Machine Manager.
  - Microsoft-NanoServer-NPDS-Package; Inatala o Network Perfomance Diagnostics Service.
  - Microsoft-NanoServer-DCB-Package: Inatala o Data Center Bridging.
  - Microsoft-NanoServer-SecureStartup-Package: Instala a inicialização segura na imagem especificada.
  - Microsoft-NanoServer-ShieldedVM-Package: Instala o pacote Shielded Virtual Machine.

### Provisionado uma máquina no Dominio Offline.

Essa função se aplica a qualquer outra instalação do WIndows Server 2016

Operação realizada no AD 

```shell
djoin.exe /provision /domain  shs.local /machine nano-srv01 /savefile c:\nanosrv01
```

Operação realizada na estação 

```shell
djoin.exe /requestodj /loadfile c:\nanosrv01 /windowspath c:\windows /localos
```

também podemos usar o atributo -DomainBlobPath no momento da criação do nano server e apontar para o local do arquivo blob criado no AD.

```powershell
New-NanoServerImage -Edition <Datacenter / Standart> -MediaPath d:\ -BasePath c:\nano -targetPath c:\nano\nano-srv.vhdx  -DeploymenType <Guest / host> -ComputerName nano-srv01 -Storage -package microsoft-nanoserver-iis-package -DomainBlobPath C:\temp\nanoblob.txt
```

Dica para adicionar uma máquina que esteja fora do dominio para ser gerenciada pelo AD.

```powershell
set-item wsman:\localhost\client\trustedhosts "192.168.17.15"
ou
winrm set winrm/config/client @{Trustedhost="192.168.17.15"}
```

Adicionando um endereço DNS por linda de comando.(Prompt de comando)

```shell
netsh interface ipv4 set dnsservers ethernet static 192.168.10.1 primary
```

Adicionando um endereço DNS por linha de comando (Powershell).

```powershell
set-dnsclientserveraddress -interfaceindex 6 serveraddress ("192.168.17.11","8.8.8.8")
New-netipaddress -interfaceindex 3 -ipaddress 192.168.17.17 -prefixlength 24 -defaultgateway 192.168.17.1
```

## *Trabalhando com repositorio oline* 

## <!--Pendente--> 

## *Implantação Windows Server Core*

### *Adcionando Windows Server Core ao Dominio*

Powershell

```powershell
Add-Computer -domainname shs.local -newname srv-servercore -credencial shs\saulo
```

Prompt de Comando 

```shell
netdow renamecomputer %computername% /newname: srv-srvcore
netdow join %computername% /domain: shs.local /userd: saulo /password:*
```

## *Trabalhando com  discos no Windows Server 2016*

### *Powershell*

#### *Básico para criação de uma unidade*

```powershell
#Listar dicos.
Get-Disk
#Limpar discos.
Clear-Disk -Number 1 -RemoverData
#Listando partições do disco.
Get-Partion -DiskNumber 1
#Listando todos os discos offline.
Get-Disk | Where-Object IsOffline -eq $true
#Alterando o Status dos discos de Offline para Online.
Get-Disk | Where-Object IsOffline -eq $true | Set-Disk -IsOffline $false
#Inicializando o disco.
Initialize-Disk -Number 1 -PartitionStyle MBR
#Criando uma nova partiçao.
New-Partition -DiskNumber 1 -Size 100mb -DriveLetter t
#Formantando.
Format-Volume -DiskLetter t -FileSystem NTFS
```

#### *Redimecionando tamanha da unidade*

```powershell
resize-Partition -DiskNumber 1 -PartitionNumber 1 -size 800mb
# Para saber o tamanho máximo e tamanho minimo da partição 
Get-PartitionSuppotedSize -DiskNumber 1 -PertitionNumber 1
```

#### *Trabalhando com VHD Powershell*

```powershell
# Deve instalar o modulo do powershell para hyper-v
# Buscando função
get-WindowsFeature *hyper-v
# Instalando função
install-windowsfeature -name <nome da feature>
# Criando um novo VHD
new-vhd -Path c:\testePs.vhdx -Dynamic -Sizebytes 1gb
# Montando um VHD
mount-vhd -Path c:\testePs.vhdx
# Desmontando um VHD
Dismount-VHD -path c:\testePs.vhdx
inittialize-disk -Number 2
get-vhd -DiskNumber 2
new-vhd -Path c:\testefull.vhdx -Dynamic -sizebytes 2GB | Mount-Vhd -Passthru | Inittialize-disk -PassThru | New-Partition -AssignDriveLetter -userMaximumSize | Format-Volume -FileSystem ntfs -Confirm $false -force
# Convertendo um VHDX para VHD
Convert-vhd path c:\testePs.vhdx -DestinationPath c:\novoVHD.vhd
```

Há dois comandos para montar um VHD no powershell. O cmdlet Mount-VHD e o Mount-DiskImage, são semelhantes mas não identicos. O primeiro faz parte do modulo do Hyper-V e o segundo faz parte do modulo Storage e pode ser envontrado em computadores que estejam execultando o Windows Server 2016.

### *Diskpart*

#### *Básico para criação de uma unidade* 

```powershell
#Listando os discos.
List Disk	
#Selecionando um disco
Select disk
#Limpando um disco.
clean
#Criando uma nova partição no disco.
Create Partition Primary size=100 
#Associando uma letra a unidade.
assign letter k
#Formatando uma unidade.
format fs=NTFS Label="Nova Partição" Quick
#Selecionar uma partição.
Select partition
#Inicializado uma unidade.
Convert MBR
```

####   *Redimencionando tamanho da unidade*

```powershell
#Diminuir tamanho da partição.
shrink desired=200
#Comando para saber o quanto podemos diminuir uma partição.
Shrink querymax
#Aumentando o tamanho de uma partição.
Extend size 800
```

#### *Criando volumes*

```powershell
#Converter unidade para dinamico.
Convert dynamic
#Volume simples.
Create volume simple size=400 disk=1 
#Volume distribuido (RAID 0)
Create volume stripe size=200 disk=1,2 
#Volume espelhado (RAID 1)
Create volume mirror size=300 disk=1,2 
#Volume Raid Faill (RAID 3)
Create volume raid size=400 disk=1,2,3 
#listar volumes 
List volume
#Selecionar Volumes
select volume 3
```

#### *Trabalhando com VHD Disk Part*

```powershell
#Cria um novo VHD
Create vdisk file="C:\teste.vhdx" maximum=500
# Anexa um VHO
attach vdisk 
```

### *Determinar quando usar os sistemas NTFS e ReFS*

O Tamanho máximo do de um volume NTFS, usando o tamanho de unidade de alocação padrão de 4 KB, é 16TB. Com unidade de alocação máxima de 64 KB, o tamanho máximo do volume é de 256TB.
O ReFS foi introduzido no Windows Server 2012 r2 , com recursos de armazenamento praticamente ilimitado e maior resiliencia que elimina a necessidde de utilizar ferramentar como Chkdsk.exe.
O ReFS não da suporte a recursos do NTFS como a compactação de arquivos, Encrypted File System e as cotas de disco. O ReFS não pode ser lido por qualquer sistema operacional mais antigo que o Windows Server 2012 r2 e Windows 8.

## *Trabando SMB via Powershell*

### *Console de gerenciamento de compartilhamento*

fsmgmt.msc

### *Comandos Powershell para manipular compartilhamentos*

```powershell
# Criando uma pasta para ser compartihada 
New-Item -Path C:\saulo -ItemType Directory 
# Criando um novo compartilhamento e dando permissão total a o grupo toos (Sistema operacional PT)
New-SmbShare -Name saulo -Path C:\saulo -FullAccess Todos
# Criando um novo compatilhamento e dando permiçao somente leitura e escrita a todos
New-SmbShare -Name saulo -Path C:\saulo -ChangeAccess todos
# Listando compartilhaentos 
Get-SmbShare
# Listando compartilhamento especifico e com detalhes
Get-SmbShare saulo | fl *
# Listando todas as sessoes 
Get-SmbSession
# Listando as sessão de um usuario especifico 
Get-Session -ClienteUserName shs\saulo | fl *
# Finalizando a Sessão de um usuário em um compartilhamento
Close-SmbSession -SessionId 9338498493
# Finalizando todas as sessões de um usuário
Close-SmbSession -ClientUserName shs\saulo
# Consedeno permissões 
Grant-SmbShareAccess -name saulo -AccountName shs\saulo -AccessRight Change / Full / Read 
# Bloqueando acesso de um usuário a um compartilhamento
Block-SmbShareAccess -Name saulo -AccountName shs\saulo
# Desbloquear acesso de um usuário
Unblock-SmbShareAccess -saulo -AccountName shs\saulo
# Revogar permssões de um usuário a um compartilhamento
Revoke-SmbShareAccess -Name saulo -AccountName shs\saulo
```

## *Desduplicação de Dados*

### *Comandos usado no Powershell*

```powershell
# Iniciando a desduplicação de imediato 
start-dedupjob e: -type optimization
#  
Get-Dedupjob -Volume e: 
# Verificando se a função esta ativa 
Get-DebupStatus  -Volume e:
# Verificando o status da deduplicação de uma unidade, visualizando em lista
get-DebupStatus -Volume e: | fl 

Get-DedupVolume -volume e:
# Desabilitando a desduplicação em uma unidade
Disable-DedupVolume -Volume e:
# Abilitando a desduplicação em uma unidade
Enable-DeduVolume -Volume e: -usagetype Default 
```

#### *Ferramenta de linha de comando*

Essa ferramenta serva para avaliar a situação da economia gerada pela desduplicação de dados.

Ddpeval.exe

## *Hyper-V*

### *Comandos usado no Powershell*

```powershell
# Criar máquina virtual
 New-VM -Name "teste" -Generation 2 -MemoryStartupBytes 1gb -NewVHDSizeBytes 40gb -NewVHDPath c:\hyper-v\teste.vhdx
# Listando VMs em Execulção
Get-VM
# Setando memoria na VM  
set-VMmemory -vmname serve01 -startupbytes 1024mb
# Adicionando um disco a uma VM existente 
Add-VMHarDrive -VMName srv01 -Path C:\Exemplo.vhdx
```

### *Editando discos no Hyper-V*

Comandos basicos para edição de discos com powershell

```powershell
# Criando um VHD
new-vhd -Path c:\testePs.vhdx -Dynamic -Sizebytes 1gb
# Otimizando um VHD (Compactando)
Optimize-VHD -Path C:\testePs.vhdx -Mode Full
# Convertend um VHD para VHDX / VHDX para VHD
 Convert-VHD -Path C:\testePs1.vhdx -DestinationPath c:\testePSconvertido.vhd -VHDType Dynamic / Fixed / Differencing
 # Redimecionando tamanho do disco 
 Resize-VHD -Path C:\testePs2.vhdx -SizeBytes 15mb
 # Mesclando dois discos (Disco pai com o disco filho para a criação de uma VM confiavel)
 Merge-VHD -Path C:\testePs.vhdx -DestinationPath C:\testePs1.vhdx
 # Habilitando o resourse midia  (Metricas)
 Enable-VMResourceMetering -VMName teste
 # Exibindo metricas da VM
 Measure-VM -VMName teste  | fl
 # Habilitando o QoS do Dicos
 Set-VMHardDiskDrive -VMName teste -ControllerType SCSI -ControllerNumber 0 -MinimumIOPS 100 -MaximumIOPS 200
```

### *Checkpoint*

Trabalhando com checkpoint no hyper-v

```powershell
# Criando um novo ponto de verificação
Checkpoint-VM -VMName srv01 -SnapshotName "Teste"
# Restore de ponto de verificação 
Restore-VMCheckpoint -name "teste" -VMName srv01
# Listando checkpoints
Get-VMSnapshot -VMName srv01
# Aplicando o tipo de Checkpoint
set-vm -Name srv01 -CheckpointType Production / standard
# Exportando uma VM a partir de um Snapshot 
Export-VMSnapshot -VMName srv01 -Name "teste" -Path "C:\checkpoint"
```

### *Limitações de VMs de segunda geração*

Não pode executar alguns sistemas convidados, como os seguintes:

- Windows Server 2008 R2
- Windows Server 2008
- Windows 7
- Algumas distribuições Linux mais antigas
- Todas as distribuições FreeeBSD
- Todos os sistemas operacionais de 32 bits

###  *Trabalhando com adaptadores de rede virtuais*

Abaixo temos algumas funções que podemos habilitar nas configurações do adaptador de rede do Hyper-V

```powershell
# Habilitar falsificação de MAC
set-vmnaetworkadapter -vmname srv01 -MacAddressSpoofing ON
# Proteção de DHCP
set-vmnetworkadapter -VMName srv01 -DhcpGuard ON
# Proteção de roteador
set-VmnetworkAdapter -VMName srv01 -RouterGuard ON
# Criando um adaptador de Rede no Hyper-V para uma VM
Add-vmnetworkadapter -vmname srv01 -switchname "Rede Interna"
# Removendo um adaptador de Rede no Hyper-V 
Remove-vmnetworkadapter -vmname srv01 -vmnetworkadapter "Rede Interna"
# Criando um novo switch virtual 
New-VMSwitch -name lan1 -netadaptername "Eternet 2"
# Ou
New-VMSwitch -name private1 -switchtype private
```

### *vEthernet NatSW*

Comandos basicos para criação do Nat-Switch

```powershell
# Criando um Comutador virtual 
 New-VMSwitch -Name "Nat" -SwitchType Internal / External / Private 
# Criando o Nat-Swich 
New-NetNat -Name "NATteste" -InternalIPInterfaceAddressPrefix 10.0.0.0/24
# lisntando os NAT-Switch 
get-netnat
# Removendo um NAT
Get-NetNat | Remove-NetNat	
```

Apos realizarmos a criação do comutador virtua que sera usado como Nat switch devemos associar um Ip a esse adaptador na máquina host.

### *Hyper-V Nested Vistualization*

Comandos utilizados no Powershell

```powershell
# Na máquina host execultar o seguinte comando para a ativar a virtualização anilhada 
Set-VMProcessor -VMName teste  -ExposeVirtualizationExtensions $true
# Na máquina host execultar o seguinte comando para desativar a virtualização anilhada 
Set-VMProcessor -VMName teste  -ExposeVirtualizationExtensions $false
# Verificando o status da função 
Get-VMProcessor -VMName teste  | fl
```

Para que a máquina virtual que esta rodando dentro de outra máquina virtual devemos ativar o recurso de falsificação de MAC nas configurações do adaptador de rede do host virtual que esta rodando a VM.

## *Docker*

Comandos basicos para ser usado no Docker e Conteiner do Windows

```powershell
# Pesquisado lista de pacotes 
Find-PackageProvider
# Instalando o modulo Docker Provider
Install-Module -name dockermsftprovider -Repository PSgallery -force
# Instalando o pacote Docker Provider
Install-Package -Name docker -ProviderName dockermsftprovider
# Com o Docker já intalado execultamos o seguinte comandos para visualizar todas as imagnes disponives no repositorio do docker com o conteudo da microsoft
docker search microsoft
# Baixando uma imagem do docker 
docker pull microsof
# Criando um Conteiner a partir da imagem baixada
Docker run -d -p 80:80 microsoft/iis:windowsservercore cmd
# Lista todos os conteiner em execulsão 
Docker ps 
# Parando um Conteiner 
Docker stop <ID do conteiner>
# Iniciando um conteiner 
Docker start <ID do conteiner>
# Abrir o prompt de comando do coteiner 
docker exec -it <ID do conteiner>
# Listando todas as imagens baixadas 
Docker images
# Para visualizar informações do conteiner
Docker inspect <ID do conteiner>
# Criando uma imagems a partir de um conteiner
Docker comit -a Autor -m Mensagem <ID do Conteiner> debian/Minha_Imagem :1.0
# Removemndo conteiner 
Docker rm -f <ID do conteiner>
# Removendo uma Imagem 
Docker rmi <ID da Imagem>
# Criando conteiner isolado dentro de uma VM
docker rum -it --isolation=hyperv microsoft/windowsservercore powershell
# Criandouma rede transparente 
docker network create -d transparent trans
# Criando uma rede transparente com IP estatico
docker network create -d transparent --subnet=10.0.0.0/24 --gateway=10.0.0.1 trans
# Listando todas as redes de conteiner 
docker network ls
# Para criar novos conteiner utilizando a rede transparentes devemos usar o seguinte comando
docker run -it --network=trans microsoft/Windowsservercore powershell
# Para criar um conteiner com endereço IP estatico na rede
docker run -it --network --ip=10.0.0.16 --dns=10.0.0.10 microsoft/windowsservercore powershell
# Para criar um volume de dados adicionamos o seguinte comando
docker run -it -v c:\appdate microsoft/windowsservercore powershell
```

## *Cluster*

Comandos para ser usado na criação de um cluster atraves do Powershell.

```powershell
# Esse comnado serve para lista todos os teste que o windows realizara no sistema para validar o ambinete de cluster
Test-Cluster -list
# Teste de Cluster que deve ser execultado antes da crianção de um cluster
Test-Cluster -Node srv06, srv07
# Criando um cluster 
New-Cluster -Name SauloCluster -StaticAddress 192.168.17.24 -Node srv06, srv07 -IgnoreNetwork 172.16.0.0/24, 10.0.0.0/24
# Comando para listar recursos do Cluster, com esse comando é possivel verificar o disco disponivel antes de adcionar uma nova funão ao Cluster
Get-ClusterResource
# Adicionando uma nova função ao cluster
Add-ClusterFileServerRole -Name FileServer -Storage "Disco de cluster 2" -StaticAddress 192.168.17.25 -Wait 0
# Criando uma pasta para compartilhar na minha função de File Server Clusterizada 
New-Item -Path d:\shares\ -name documentos -ItemType Directory
# Compartilhando pasta com acesso para todos os usuários
New-SmbShare -Name documentos -Path d:\shares\documentos\ -ChangeAccess todos
```

### *Quorum*

Alterando algumas propiedades da testemunha do Cluster

```powershell
# Comando para verificar as testemunhas dos nós do clustes 
Get-ClusterNode | select name, NodeWeight
# Listando qual meu Quorum e qual o tipo 
Get-ClusterQuorum | select Cluster, QuorumResource, QuorumType
# Alterando o Quorum para um compartilhamento de arquivo 
Set-ClusterQuorum -NodeAndFileShareMajority \\srv05\testemunha$
```

### *Backup e Restore*

Para realizar o backup de uma função do Cluster podemos utilizar o utilitario de backup do propio Windows, mas para realizar o restore da função deve ser realizado de forma autoritativa e esse procedimento de restore só é possivel de ser realizado atraves do Powershell.

```powershell
# Listando todos os itens de backups 
wbadmin get versions 
# restaurando o backup 
Wbadmin start recovery -version: 21/10/2018 09:44 -Itemtype:app -Items: Cluster  
```

### *Atualização do Cluster CAU*

Esse recurso permite a atualização dos nós do Cluster sem a interrupção da função clusterizada. Pois ele faz a orquestração dos updates dos nós desligando, movendo recursos e tudo que for necessario para o sistema não ser interrompido.
Para execultar a atualização com o CAU antes do periodo determinado no agendamento devemos executar o seguinte comando.

```powershell
Set-CauClusterRole -ClusterName SauloCluster -UpdateNow -force	
```

### *NLB*

Abaixo segue alguns comandos basicos para ser utilizado em um Cluster NLB

```powershell
# Removendo um Cluster NLB 
Remove-NlbCluster
# Listando Cluster 
Get-NlbCluster
# Criando um novo Cluster NLB (Nesse meu exemplo a minha máquina esta com o adaptador de rede renomeado para Empresarial)
New-NlbCluster -InterfaceName "Empresarial" -OperationMode Multicast -ClusterPrimaryIP 192.168.17.27 -ClusterName nlb.saulo.local
# Listando nós do Cluster 
Get-NlbClusterNode
# Adcionando um novo nó a um cluster existente 
Add-NlbClusterNode -InterfaceName "Empresarial" -NewNodeName "srv06" -NewNodeInterface "Empresarial"
# Listando regras da porta do cluster
Get-NlbClusterPortRule
# Removendo uma regra de porta 
Get-NlbClusterPortRule | Remove-NlbClusterPortRule
# Adcionando uma regra de porta a um cluster 
Get-NlbCluster | Add-NlbClusterPortRule -Protocol Both (ambos) -Mode Multiple -Affinity None -StartPort 80 -EndPort 80
Get-NlbCluster | Add-NlbClusterPortRule -Protocol Both -Mode Single -StartPort 5678 -EndPort 5678
# Listando uma regra de porta especifica a partir da porta
Get-NlbClusterPortRule -Port 5678
# Listando um regra de porta especifica a partir da porta e de um nó especifico 
Get-NlbClusterPortRule -Port 5678 -NodeName srv06
# Editando uma regra de porta de nó (Nessa situação vou adcionar uma prioridade mais alta a esse nó para que as solicitações em destino a essa porta dem prefetencia o outro nó)
Get-NlbClusterPortRule -Port 5678 -NodeName srv06 | Set-NlbClusterPortRuleNodeHandlingPriority -HandlingPriority 10
# Parando um nó do cluster 
Get-NlbClusterNode -NodeName srv06 | Stop-NlbClusterNode
# Iniciando um nó do cluster
Get-NlbClusterNode -NodeName srv06 | Start-NlbClusterNode
# Interrupção de descarga (Não permite novas conexões ao nó do cluster e para o serviço assim que todas que tiverem ativas forem encerradas)
Get-NlbClusterNode | Stop-NlbClusterNode -Drain
# Suspender um nó do cluster  (Apos um nó do cluser ser suspenço ele precisa ser continuado para que assim ele possa ser iniciado)
Get-NlbClusterNode -NodeName srv06 | Suspend-NlbClusterNode
# Continuado um nó do CLuster
Get-NlbClusterNode -NodeName srv06 | Resume-NlbClusterNode
```

## *WSUS*

Portas que devem ser liberadas no Firewall do Windows para uso do Wsus.

- Para conexões HTTP 8530
- Para conexões HTTPS 8531

Comandos básicos para serem utilizados no Wsus

```powershell
# Força que um computador seja registrado no Wsus
Wuauclt.exe /detectnow /reportnow
# Lsiar todos os computadores adcionados ao Wsus
Get-WsusComputer -all
# Movendo computadores para outro grupo de atualização do Wsus 
Get-WsusComputer -NameIncludes srv | add-WsusComputer -TargerGroupName "Ti"
# Listando updates no Wsus 
Get-WsusUpdate -Classification All -Approval Approved
# Listando Update do tipo segurança com status não paorvado e Status Necessaria 
Get-WsusUpdate -Classification Security -Approval Unapproved -Status Needed 
# combinando comnados para em seguida aprovar atualizações para o grupo de update Teste
Get-WsusUpdate -Classification Security -Approval -Status Needed | Approve-WsusUpdate -Action Install -TargetGroupName "Teste"
```

## *Gerenciamento de processos*

Comandos basicos para gerencias processos com o Powershell

```powershell
# listando processos 
Get-Process 
# Listando processos especificos
Get-Process Calculator, cmd
# Finalizando processos 
Stop-Process <ID do processo>
Stop-Process -Name Calculator
```



##  

