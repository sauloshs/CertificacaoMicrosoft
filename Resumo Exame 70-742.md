# *Resumo do Exame 70-742*

-------

## *Active Directory*

​	O que é o Active Directory: É um serviço de diretório que pode ser explicado com varias ilustrações que, em minha opinião, mais demostra o verdadeiro sentido de um serviço de diretório é a figura de uma lista telefônica ou uma agenda pessoal.
​	Em nossa agenda podemos organizar, dias, semanas, meses e até anos, passando por pessoas, nomes, sobrenomes, datas de aniversario, dados importantes dentre outros.
​	O Serviço de diretório tem exatamente o mesmo sentido, o sentido de organizar e principalmente de ter um local centralizado de armazenamento para busca das informações necessárias para o dia a dia, para nossos trabalhos.

​	O AD DS é composto por componentes lógicos e físicos. Um componente físico é algo tangível, com um controlador de domínio, enquanto uma floresta de AD DS é um componente lógico, intangível. O AD DS consiste nos seguintes componentes lógicos e físicos:

| Componentes lógicos | Componentes físicos           |
| ------------------- | ----------------------------- |
| Partições           | Controladores de domínio      |
| Esquema             | Repositório de dados          |
| Domínios            | Servidores de catálogo global |
| Árvores de domínio  | RODCs                         |
| Florestas           |                               |
| Sites               |                               |
| UOs                 |                               |

- **Floresta** é um conjunto de domínios do AD DS que compartilham um mesmo esquema e são ligados por relações de confiança recíprocas criadas automaticamente. A maioria das organizações implementam AD DS com apenas uma floresta.
- **Domínio** é uma unidade administrativa lógica que contém usuários, grupos, computadores e outros objetos. Vários domínios podem fazer parta de uma ou de várias florestas, dependendo das necessidades organizacionais. Relações pai-filho e de confiança definem a estrutura do domínio.
- **Árvores** é um conjunto de domínios do AD DS que compartilham um domínio raiz comum e tem o namespace contíguo. Por exemplo, sales.adatum.com e marketing.adatum.com compartilham uma raiz comum adatum.com, e também um namespace contíguo, adatum.com. É possível construir sua floresta do AD DS usando uma ou várias árvores.
- **Esquema** o esquema do AD DS é o conjunto de tipos de objetos e sua propriedades, também conhecidas como atributos, que definem os tipos de objetos que podem ser criados, armazenados e gerenciados dentro da floresta do AD DS. Por exemplo um usuário é um tipo de objeto lógico, tendo várias propriedades, incluindo um nome completo, departamento e uma senha. A relação entre o objeto e seus atributos é mantida no esquema, sendo que todos os controladores de domínio de uma floresta contêm uma cópia do esquema.
- **OU** é um contêiner dentro de um domínio que contém usuários, grupos, computadores e outras OUs. Elas são usadas para oferecer simplificação administrativa. Com OUs, você pode facilmente delegar direitos administrativo para uma coleção de objetos, agrupado-os em uma OU e atribuindo o direito nessa OU. Também é possível usar Group Policy Objects (GPOs, Objetos de politicas de grupos) para definir configurações de usuários e computadores e vincular essas configurações da GPO a uma OU, otimizando o processo de configuração. Uma OU é criada por padrão quando você instala o AD DS e cria o domínio: Domain Controllers.
- **Contêiner** Além das OUs, também é possível usar contêineres para agrupar coleções de objetos. Não é possível vincular GPOs aos contêineres.
- **Site** é uma representação lógica de uma localização física dentro de sua organização. Ele pode representar uma área física grande, como uma cidade, ou uma área física menor, como um conjunto de sub-redes definido pelos limites de seu datacenter. Os sites também permitem controlar a replicação do AD DS, através da configuração de uma agenda e um intervalo para a replicação entre sites.
  - Um site é criado por padrão quando você instala o AD DS e cria sua floresta: Default-First-Site-name e todos os controladores de domínio pertencem a esse site.
- **Sub-rede** É uma representação lógica de uma sub-rede física em sua rede. Com a definição de sub-redes, um computador em sua floresta do AD DS pode determinar sua localização física em relação aos serviços oferecidos na floresta. Não existem sub-redes por padrão. depois de criar sub-redes, você as associa a sites. Um site pode conter mais de uma sub-rede.

Instalando a função de AD DS através do PowerShell:

```powershell
# Instalando função AD DS
Install-WindowsFeature AD-Domain-Services
# Promovendo a um controlador de domínio e adicionar a função de DNS
InstallADDSDomainController InstallDNS -DomainName shs.com
```

Rebaixando um controlador de domínio através do PowerShell:

```powershell
# Rebaixando
Uninstall -addsdomaincontroller
# Removendo a Função AD DS
Uninstall -WindowsFeature AD-Domain-Services
```

## *Clonagem do Active Directory*

Você pode clonar AD para:

- Implantação mais rápida.
- Nuvens privadas.
- Estratégia de recuperação.

Requisitos:

- O Hypervisor precisa oferecer suporte a identificação de geração em VMs.
- Controlador de domínio baseados em Windows Server 2012 ou posterior 
- Emulador PDC online.
- O controlador de domínio colunável deve pertencer ao grupo controladores de domínios clonáveis.

Comandos que devem ser executados no PowerShell para clonar o controlador de domínio:

```powershell
# Para listar aplicações imcompativeis com a clonagem 	
Get-addcCloningExcludeApplicationList
# Utilize o parametro para importar a lista para um arquivo XML
Get-addcCloningExcludeApplicationList -GenerateXml
# Configurações iniciais do clone que serão armazenadas na pasta c:\windows\NTDS
New-addcCloneConfigFile -static ipv4address "192.168.0.5" -Ipv4DnsResolver "192.168.0.1" -Ipv4SubnetMask "255.255.255.0" -Ipv4DefaultGateway "192.168.0.254" -ClonecomputerName "srvdc05"
```

## *IFM*

​	O metodo IFM é utilizado para implemtanção de AD em ambientes que duas filias estão conectadas atraves de link de internet lento. Com isso utilizamos a a midia de intação a fim de minimizar o impacto da implantação do AD, pois sabemos que a replicação inicial pode ser muito grande.

Comandos utilizados para criação da mídia:

```powershell
ntdsutil # Após ececutar esse comando entramos no prompt 
active instace ntds # Para acessar o prompt NTDS
ifm # Com esse comando entramos no modulo NTDS
create sysvol full c:\ifm # Com esse comando é criado os arquivos necessários para a criação da midia IFM
```

## *FSMO*

​	As funções FSMO são:

- Mestre RID responsável por gerar os SIDs dos objetos, porem para evitar problemas na criação de objetos na rede por indisponibilidade do mestre de operação ele distribui um bloco de SID para cada controlador de domínio.

- Mestre de Infraestrutura 

- Mestre de nomeação de domínio

- Emulador PDC ou Controlador de domínio primário, responsável por atualizar a hora de todos os computadores do domínio e necessário na clonagem de AD DS.

  - Mestre de Schema, Para ter acesso ao console do mestre de schema é necessário registrar uma dll com o seguinte comando no prom	pt de comando.

    ```powershell
    regsvr32 schmgmt.dll
    ```

    Em seguida através do mmc adcionar o console do schema e realizar a alteração.
  
  Alteração das funções FSMO através da linha de comando:

```powershell
# Consultar o mestre de esquema do domínio
netdom query fsmo 
# Sequencia de comandos necessarios para realizar a alterção das funções.
ntdsutil
roles
connections
connect to server srv01.shs.local # após realizar a conexão devemos sair com o comando abaixo.
quit
# Dando inicio a transferencia de funções.
transfer pdc
trabsfer rid master
transfer naming master
transfer schema master
transfer infrastructure master
```

​	Transferência dos mestres de operação através do Powershell 

```powershell
# Consulta ao Domínio onde mostra os mestres de operações do domínio e outras opções.
Get-ADDomain
# Para refinar a consulta utilizamos o comando da seguinte maneira.
Get-ADDOmain | fl Infrastructuremaster,PDCEmulation,RDIMaster
# Consultar os mestres de operações a nivel de floresta.
Get-AFForest | fl DomainNamingMaster,SchemaMaster
# Transferindo funções FSMO 
Move-ADDirectoryServerOperationMasterRole -identity srv02 -OperationMasterRole DomainNamingMaster,SchemaMaster,Infrastructuremaster,PDCEmulation,RDIMaster
```

​	Captura de funções FSMO
​	Esse procedimento é utilizado em ocasiões onde perdemos o mestre de operações da rede seja por motivos de falha no sistema operacional ou administrativa. Esse procedimento só é possível através de linha de comando.

​	Executando captura através do prompt de comando.

```powershell
ntdsutil
roles
connections
connection to server srv01.shs.local
quit
seize PDC # Note que esse comando é a diferença que permite a captura do mestre de operação.
seize rid master
seize schema master
seize naming master
seize infrastructure master
```

  	Captura através do powershell.

```powershell
# Note que a unica diferença é o comando -force ao final do comando.
Move-ADDirectoryServerOperationMasterRole -identity srv02 -OperationMasterRole DomainNamingMaster,SchemaMaster,Infrastructuremaster,PDCEmulation,RDIMaster -force
```