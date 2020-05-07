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
Install-ADDSDomainController InstallDNS -DomainName shs.com
```

Rebaixando um controlador de domínio através do PowerShell:

```powershell
# Rebaixando
Uninstall-addsdomaincontroller
# Removendo a Função AD DS
Uninstall-WindowsFeature AD-Domain-Services
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
- O controlador de domínio clonavel deve pertencer ao grupo controladores de domínios clonáveis.

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

## *Funções de Mestre de Operações (FSMO)*

​	As funções FSMO são:

- A nível de Floresta:
  - **Mestre de esquema:** O mestre de esquema mantém o esquema e é responsável por propagar quaisquer alterações feitas nele para outras cópias dessa partição do AD DS em todos os outros controladores de domínio da floresta. Como o esquema raramente muda, a ausência temporária desse mestre de operações pode passar despercebida facilmente. Contudo, ele deve estar online quando você fizer mudanças no esquema, por exemplo, quando instalar um aplicativo, como o Exchange Server, que exige tipos de objetos e atributos adicionais em relação aos tipos de objetos existentes.
  - **Mestre de nomeação de domínio:** O mestre de nomeação de domínio manipula a adição ou a renomeação de domínios na floresta do AD DS. Como esses tipos de alterações não são frequentes, se o mestre de nomeação de domínio ficar indisponível temporariamente, você pode não perceber isso imediatamente.

- A nível de Domínio:

  - **Emulador PDC:** Executa várias operações importantes em nível de domínio.
    - Atua como fonte de sincronização de hora para o domínio.
    - Propaga alterações de senha.
    - Fornece uma fonte primária para GPOs para propósitos de edição.
  - **Mestre de infraestrutura:** Mantém referência entre domínios e, consequentemente, essa função só é relevante em florestas com vários domínios. Por exemplo, o mestre de infraestrutura mantém a integridade da lista de controle de acesso de segurança de um objeto, quando essa lista contém entidades de segurança (Security principals) de outro domínio.
    - Você não deve atribuir a função de mestre de infraestrutura a um servidor de catalogo global, a não ser que sua floresta consista de apenas um domínio. A única exceção a isso é se todos os controladores de domínio também são servidores de catálogo global, nesse caso a função mestre de infraestrutura é redundante.
  - **Mestre RID:** Fornece blocos de identificação para cada um dos controladores de domínio em seu domínio. cada objeto em um domínio exige um identificador único.

  Para ter acesso ao Mestre de Schema, é necessário registrar uma dll com o seguinte comando no prompt de comando.

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
Get-ADForest | fl DomainNamingMaster,SchemaMaster
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

## *Criação de contas de usuários*

- Contas de usuários 

  - Permitir ou negar acesso para efetuar logon em computadores.
  - Conceder acesso a processos e serviços.
  - Gerenciar acesso a recursos da rede.

- As contas de usuários podem ser criadas usando.

  - Usuários e computadores do Active Directory.
  - Centro Administrativo do Active Directory.
  - Windows Powershell.
  - Ferramenta de linha de comando do diretório dsadd.

  Obs. sobre perfil móvel de usuário.
  No perfil móvel os dados do usuário somente serão levados para o servidor após a realização do logoff.

## *Grupos de Usuários*

- Grupos de distribuição 
  - Usados somente com aplicativos de email.
  - Não são habilitados para segurança (sem SID).
  - Não é possível conceder permissões.
- Grupos de segurança
  - Entidade de segurança com SID.
  - É possível conceder permissões.
  - Também pode ser habilitado para email

Obs. Você pode converter grupos de segurança para grupos de distribuição e vice-versa.

​	**Escopo de Grupo.**

- Os grupos locais podem conter usuários, computadores, grupos globais, grupos do domínio local e grupos universais do mesmo domínio, de domínio da mesma floresta e de outros domínios confiáveis. Além disso, eles podem receber permissões para recursos apenas do computador local.
- Os grupos de domínio local tem as mesmas possibilidades de associação, mas pode receber permissão para recursos em qualquer lugar do domínio.
- Os grupos globais podem conter apenas usuários, computadores, e outros grupos globais do mesmo domínio. Além disso, eles podem receber permissões para recursos no domínio ou qualquer outro domínio confiavel. 
- Os grupos universais podem conter usuários, computadores, grupos globais e outros grupos universais de qualquer domínio da floresta. Além disso, eles podem receber permissões para qualquer recurso na Floresta.

​	Gerencie cuidadosamente os grupos padrão que fornecem privilégios administrativos, pois eles: 

- Geralmente, têm privilégios mais amplos do que necessário para a maioria dos ambientes delegados.
- Aplicam proteção com frequência aos seus membros.	

| Grupo                      | Local                                             |
| -------------------------- | ------------------------------------------------- |
| Administradores de Empresa | Contêiner de usuários do domínio raiz da floresta |
| Administradores de Esquema | Contêiner de usuários do domínio raiz da floresta |
| Administradores            | Contêiner interno de cada domínio                 |
| Admins. do Domínio         | Contêiner de usuários de cada domínio             |
| Operadores de Servidor     | Contêiner interno de cada domínio                 |
| Opers. de Contas           | Contêiner interno de cada domínio                 |
| Operadores de Backup       | Contêiner interno de cada domínio                 |
| Operadores de Impressão    | Contêiner interno de cada domínio                 |
| Editores de Certificados   | Contêiner de usuários de cada domínio             |

Obs.: O contêiner computadores criado por padrão que é usado para armazenar computadores recém adicionados no domínio não podem ser vinculados a GPOs.

​	Contas de computadores e canais de segurança.

- OS computadores tem contas:
  - SAMAccountName e Senha.
  - Usado para criar um canal de segurança entre  o computador e um controlador de domínio. (As senhas de computadores são negociadas a cada 30 dias  e de forma automática)
- Cenários em que um canal de segurança pode ser interrompido:
  - A reinstalação de um computador, inclusive com o mesmo nome, gera um novo SID e uma nova senha.
  - Restauração de um computador de um backup antigo ou reversão de um computador para um instantâneo antigo.
  - O computador e domínio descordam sobre a senha.

​	Redefinição do canal de segurança.

- Não exclua um computador do domínio e o adicione novamente. Isso cria uma nova conta, resultando em um novo SID e perda das associações de grupo.
- Opções para redefinir o canal de segurança:

```powershell
# Usuários e computadores do active directory => Redefinir computador.

# Centro administrativo do active directory => Redefinir computador.

# dsmod computer "ComputerDN" -reset
# nltest /server:<servername> /sc_reset:<Domain\domaincontroller>
#Test-ComputerSecureChannel -Repair
```

### *Ingressando máquinas no domino off-line.*

```powershell
# Primeiro devemos realizar o provisionamento da conta de computador no AD DS
djoin /provison /domain shs.local /machine srv02 /savefile c:\djoin\srv02.txt 

# No computador ao qual desejamos incluir no domnio executamos o seguinte comando:
djoin /requestODJ /loadfile c:\djoin\srv02.txt /windowspath %systemroot% /localos
```

​	Alterando local padrão dos computadores ingressados no domínio.

```powershell
# No exemplo abaixo as contad de computadores foram redirecionadas para a OU matrizcmp
redircmp ou=matrizcmp,dc=shs,dc=local
```

### *Indenidades Especiais*

​	Elas são tratas como  grupos dentro do sistema operacional, visto que é possível conceder permissões e direitos a elas, como qualquer outro grupo. Contudo, a lista de membros não pode ser editada, ou seja, não é possível associar uma identidade especial a um usuário (ou outro grupo). Em vez disso, os membros de grupo são associados de forma implícita, com base nas características de um usuário em determinada situação. As identidades especiais são:

- **Everyone** Esta identidade representa todo mundo e inclui tanto usuários com conta como convidados sem conta - supondo que contas de convidados estejam habilitadas.
- **Authenticated Users** esta identidade é mais especifica e inclui todos, exceto os convidados.
- **Anonymous Logon** Esta identidade é usada pelos recursos que não exigem nome de usuários nem senha para permitir o acesso. Ela não incluiu convidados.
- **Interactive** Um usuário que está tentando acessar um recurso no computador local possui a identidade interactive.
- **Network** Um usuário que está tentando acessar um recurso em um computador remoto possuiu a identidade Network.
- **Creator Owener** Qualquer individuo que cria um objeto, como um arquivo possui a identidade Creator Owner desse objeto. Um usuário associado a essa identidade tem total controle sobre o objeto. 

## *Ambiente distribuído do AD DS*

Reverter elevação de nível funcional do domínio 

Set-ADDomainMode.

### *Replicação do AD DS* 

​	**Replicação Intrassites** 

​	A replicação Intrassites usa:

- **Objeto de conexão** para replicação de entrada em um controlador de domínio.

- **Knowledge Consistency Checker** para automaticamente criar a topologia que é eficiente (máximo três saltos) e robusta (bidirecional).

- **Notificação** nas quais o controlador de domínio informa aos seus parceiros downstream que uma alteração esta disponível.

- **Sondagem**, na qual o controlador de domínio verifica com seus parceiros upstream se há alterações:

  - O agente de replicação de diretório do controlador de domínio downstream replica as alterações.
  
- As alterações em todas as partições mantidas por ambos os controladores de domínio são replicadas.

**Como Funciona a replicação Sysvol.**

- O Sysvol contém scripts de logon, modelos de políticas de grupo e GPOs com seu conteúdo.

- A replicação sysvol pode ocorrer usando:

  - O FRS, que é usado principalmente no Windows Server 2003 e em estruturas de domínio mais antigas.
  - A replicação do DFS, que é usada no Windows Server 2008 e em domínios mais recentes.

- Para migrar a replicação SYSVOL do FRS para a replicação do DFS:

  - O nível funcional do domínio deve ser pelo menos Windows Server 2008.
  
- Use a ferramente **Dfsrmig.exe** para executar a migração.

**O que são sites do AD DS?**

- Os sites identificam locais de rede com conexão de rede rápidas e confiáveis.
- Os sites são associados a objetos de sub-rede.
- Os sites são usados para gerenciar:
  - Replica quando controladores de domínio são separados por links lentos e caros.
  - Localização do serviço.
    - Autenticação do controlador de domínio.
    - Serviços ou aplicativos que reconhecem o AD DS (reconhecimento de site).

**Como funciona a replicação entre sites.**

- A replicação em sites:
  - Presume links de rede rápidos, baratos e altamente confíaveis.
  - Não compacta o trafego.
  - Usa um mecanismo de notificação de alteração.
- A replicação entre sites:
  - Presume links de rede de custo mais alto, largura de banda limitada e não confíavel.
  - Tem a capacidade de compactar a replicação.
  - Ocorre em uma agenda configurada.
  - Pode ser configurada para replicações imediatas e urgentes. (Uma excessao a essa regra é a alteração de senha do usuário que é iniciada imediatamente no mestre de emução de PDC imediatamente).

**Links de sites**

- Dentro de um link de site, um objeto de conexão pode ser criado entre quaisquer dos controladores de domínio.
- O link de site padrão, **DEFAULTIPSITELINK**, nem sempre é apropriado à sua topologia de rede.
  - O DEFAULTIPSITELINK tem o custo padrão igual a 100 e ao criar um novo link de site podemos mudar o intervalo de replicação.
- Custos de links de sites:
  - A replicação usa conexões com custos mais baixos.
- Replicação:
  - Durante a sondagem, o servidor **Bridgehead** (Servidor responsável pela replicação entre sites) downstream sonda seus parceiros upstream.
    - O padrão é de 3 horas.
    - O mínimo é de 15 minutos.
    - O recomendável é 15 minutos.
  - Agendamentos de replicação:
    - 24 horas por dia.
    - É possível agendar 

***Comandos para administração de sites no prompt de comando***

```powershell
# Recalcular a topologia de replicação do servidor 
repadmin /kcc
# Exibir informações de replicação com o seu DC
repadmin /showrepl
# Verifica quais são os servidor Bridgeheads
repadmin /bridgeheads
# Exibe um resumo das tarefas de replicação
repadmin /replsummary 
# Testa a replicação
dcdiag /test:replicarions
```

### *Relação de Confiança*

Sempre que for criar uma relação de confiança entre duas empresas que pertençam a florestas distintas devemos criar uma **zona stub** no DNS para que ele replique somente os registros de nomes da outra empresa.
Também podemos criar encaminhadores condicionais mas, é recomentado a zona stub.

- **Transitividade da relação de confiança** (Entre florestas) - A transitividade determina se uma relação de confiança pode ser estendida para fora dos dois domínios entre os quais ela foi formada. É possível usar uma relação de confiança transitiva para estender relações de confiança com outros domínios. Você pode usar uma relação de confiança intransitiva para negar relações de confiança com outros domínios.
- **Relação de confiança intransitiva** (Externa) - Uma relação de confiança intransitiva é restrita a dois domínios da relação de confiança. Ela não segue na direção de nenhum outro domínio da floresta. Além disso, ela pode ser unidirecional ou bidirecional. As relações de confiança intransitivas são unidirecionais por padrão, embora também seja possível criar uma relação bidirecional durante a criação de relações de confiança unidirecionais.

## *RODC*

Considere as seguintes limitações ao implantar o RODC:

- Os RODCs não podem ser proprietários da função de mestre de operações.
- Os RODCs não podem ser servidores bridgehead.
- Você deve ter apenas um RODC por site, por domínio.
- OS RODCs não podem autenticar através de relação de confiança quando a conexão de WAN não disponível.
- Nenhuma alteração de replicação se origina em RODC.
- Os RODCs não da suporte apropriado a aplicação que precisam atualizar o AD DS interativamente.

No momento da instalação de um servidor RODC devemos criar uma conta de administrador delegado, pois as contas de administrador do domínio não tem suas senhas replicadas para o mesmo.
Nas propriedades do servidor RODC (Usuário e computadores do Active Directory) temos a opção "Diretiva de replicação de Senha" onde veremos as contas que são permitidas o armazenamento em cache no servidor RODC.

## *GPO*

​	A política de grupo é simplesmente a maneira mais fácil de abranger e configurar o computador e as configurações do usuário em redes baseadas no AD DS.
​	Os requisitos para utilizar as políticas de grupos são:

- A rede deve ser baseada em AD DS.
- Os computadores que você deseja gerenciar devem estar ingressados no domínio, e os usuários que você deseja gerenciar dever usar credencias de domínio para fazer logon nos respectivos computadores.
- É necessário ter permissão para editar a política de grupo no domínio.

​	Na GPO default a política de senha do domínio por padrão tem o seu nível de complexibilidade habilitado por padrão, e a senha do usuário deve satisfazer 3 dos 4 solicitados, que são letras maiúsculos, minúsculas, números e caracteres especiais. 

**Armazenar senha utilizando criptografia reversível** permite que você utilize a senha de login do usuário sem criptografia. Essa alternativa não é uma opção segura para o ambiente, mas pode ser necessário ativar para uso com aplicações de terceiros que não permite trabalhar com criptografia. 

**Atribuição de direitos de usuários** você pode dar algumas permissões que nativamente vem desabilitadas para usuários padrão. Com isso não precisamos dar permissões de administrador para executar determinada tarefa. Ex: alterar a hora do sistema.

**Nível de precedência das GPOs**

Os GPOs são processados nos computadores clientes nas seguintes ordens:

- GPOs locais.
- GPOs no nível de site.
- GPOs no nível de domínio.
- GPOs OU, inclusive todas as OU anilhadas.

**GPO de inicio**.

- Armazena configurações de modelo administrativas nas quais novos GPOs se basearão.
- Pode ser exportado para arquivos .cab.
- Pode ser importado para outras áreas de uma organização.

**Configurações para configuração de redirecionamento de pasta.**

- Opções de configuração de redirecionamento de pasta:
  - Usar o redirecionamento de pasta básico quando todos os usuários salvarem os arquivos no mesmo local.
  - Usar o redirecionamento de pasta avançado quando o servidor que hospeda o local da pasta estiver baseado em associação de grupo.
  - usar seguir pasta documentos para forçar determinadas pastas a se tornarem subpastas de documentos. 
- Opções de local da pasta de destino.
  - Criar uma pasta para cada usuário no caminho raiz.
  - Redirecionar para localização de perfil de usuário local.
  - Redirecionar ao diretório base do usuário (Somente pasta Documentos).  

### Backup de GPO

```powershell
# Backup de GPO
backup-gpo -all -Path c:\backup
# Restaurar GPO compras
restore-gpo -path c:\backup -Name compras
```

### Relatórios RSOP

​	Pode usar ter assistentes:

- Assistente de resultados de politicas de grupo
- GPResults
- Get-GPResultantSetOfPolicy

### **O que são modelos administrativos**

Os modelos administrativos fornecem a capacidade para controlar o ambiente do sistema operacional e a experiência do usuário:

- Seção do modelo administrativo para computadores:
  - Painel de controle.
  - Rede.
  - Impressoras.
  - Sistema.
  - Componentes baseados no Windows.
- Seção do modelo administrativo dos usuários:
  - Painel de controle.
  - Área de trabalho.
  - Rede.
  - Menu iniciar e barra de tarefas.
  - Sistema.
  - Componentes baseados no Windows.
- Cada uma dessas seções principais contém muitas subpastas para mais configurações de organização.

### **O que são os arquivos .adm e admx?**

- Arquivos .adm.
  - São copiados em todos os GPOs no SYSVOL.
  - São difíceis de serem personalizados.
  - Não apresentam neutralidade de idioma.
  - Pode causar SYSVOL sovrecarga se houver vários GPOs.
- Arquivos .admx.
  - Apresentam neutralidade de idioma.
  - Os arquivos .adml fornecem o idioma localizado.
  - Não são armazenados no GPO.
  - São extensíveis através de XML.

### **Visão geral do repositório central**

O repositório central.

- É um repositório central para arquivos .admx e adml.
- é armazenado no SYSVOL.
- Deve ser criado manualmente.
- É detectado automaticamente pelos sistemas operacionais Windows Vista, Windows Server 2008 e pelos sistemas operacionais mais recentes. 

**Instalação de aplicativos através de GPO.**

- Métodos:
  - Publicado - O usuário vai ate o painel de controle para instalar a aplicação disponibilizada pela rede (mesmo que não seja administrador).
  - Atribuído - O usuário recebe o software automaticamente através da politica de grupo.
  - Avançado - É o método onde você atribui configurações especificas.

### **O que são preferencias de politicas de grupo**

As extensões de preferência de politica de grupo expandem o intervalo de definições configuráveis dentro de um GPO.

- permite que você gerencie configurações que estavam anteriormente não gerenciável usando Política de Grupo.
- São suportadas nativamente no Windows Server 2008 e mais recente e no Vista SP2 e mais recente.
- Podem ser criadas, excluídas, substituídas ou atualizadas.
- As categorias incluem unidades mapeadas, atalhos, alterações de registro, opções de energia, agendas, tarefas e definições do Internet Explorer.

**Comparação de preferência de Política de Grupo e configurações da Política de Grupo.**

| Configurações de Política de Grupo                           | Preferências de Política de Grupo                            |
| ------------------------------------------------------------ | ------------------------------------------------------------ |
| Impõe configurações de política por meio da gravação de configurações em áreas do registro que os usuários padrão não podem modificar. | São gravadas em locais normais no registro que o recurso do aplicativo ou do sistema operacional usa para armazenar a configuração. |
| Geralmente, desabilita a interface de usuário para configurações que a Política de Grupo está gerenciando. | Não faz o recurso do aplicativo ou do sistema operacional desabilitar a interface de usuário para as configurações que eles configuram. |
| Atualiza configurações de política em intervalos regulares.  | Atualiza as preferências usando o mesmo intervalo como configuração da Política do Grupo, mas pode ser configurada para aplicar somente uma vez. |



## *PSO*

​	Objeto de politica de senha serve para criar uma senha mais refinada para um grupo especifico de usuários, e foi introduzida a partir do Windows Server 2008.
​	O caminho para configurar essa opção no Windows Server 2016 é através da Central Administrativa do Active Directory, em seguida clicando no domínio, System,Password Settings Container e em seguida clicar em novo.

## *Criando contas de serviços gerenciadas*

```powershell
# No AD DS atraves do powershell
New-ADServiceAccount -Name SauloService -RestrictToSingleComputer -Enabled $True
# Depois devemos associar essa conta a um computador
Add-ASComputerServiceAccount -Identity srv02 -ServiceAccount SauloService 
# No servidor de destino no nosso caso srv02 devemos instalar a ferramenta de RSAT caso o mesmo não seja um controlador de domínio
Add-WindowsFeature Rsat-AD-Powershell
# Apos a intalação instalar a conta no servidor de destino
Install-ADServiceAccount -identity SauloService
# Apos esse procedimento podemos associar essa conta shs\SauloService a um serviço em srv02
```

 ## *AD CS*

​	Para gerenciar a hierarquia do AD CS 

- Console de gerenciamento da AC
- Windows Powershell
- Ferramenta de linha de comando CertUtil.exe

**Solucionar problemas de ACc**

- Ferramentas para o gerenciamento de ACs:
  - Snap-in de certificados.
  - Console PKIView.msc.
  - Console de autoridade de certificação.
  - Certutil.exe.
  - Sanp-in de modelos de certificados.
- Problemas comuns do AD CS:
  - Problemas de registro automático do cliente.
  - Opção de AC corporativa indisponível.
  - Erro ao acessar as páginas da Web da AC.
  - Registrição de registro de agente.

**Renovação do certificado de uma AC**

- O certificado de AC precisa ser renovado quando seu período de validade estiver próximo da data de expiração.
- A AC nunca emite certificado que tenha período de validade maior que seu próprio certificado.
- Considerações para a renovação de um certificado de AC raiz:
  - Comprimento da chave.
  - período de validade.
- Considerações paa a renovação de um certificado de uma AC emissora:
  - Novo par de chaves.
  - CRLs pequenas.
- Procedimento de renovação de um certificado de AC.

**Migração de uma AC raiz para outro computador**

- Para mover uma AC de um computador para outro, você deve realizar o backup e restauração:
  - Para fazer o backup de um computador, siga este procedimento.
    - Registe os nomes dos modelos de certificados.
    - Faça um backup de uma AC no console de administração de AC.
    - Exporte a subchave de registro.
    - Desinstale a função da AC.
    - Confirme os locais da pasta %SystemRoot%
    - Remova a AC antiga do domínio.
  - Para restaurar, siga este procedimento:
    - Instale o AD CS.
    - Use a chave privada existente.
    - Restaure o arquivo de registro.
    - Restaure o banco de dados e as configurações da AC.
    - Restaure os modelos de certificado.

**ACs autônomas vs ACs corporativas** 

| ACs autônomas                                                | Acs cosporativas                                             |
| :----------------------------------------------------------- | :----------------------------------------------------------- |
| Deve ser usada caso uma CA (raiz/intermediaria/de politica) estiver off-line porque uma AC autônoma não está adicionada a um domínio do AD DS. | Exige o uso de ADDS e armazena informações em AD DS.         |
|                                                              | Pode Usar a política de grupo para propagar certificados para o repositório de certificado AC raiz confiável. |
| Os usuários devem fornecer informações de identificação e especificar o tipo de certificado. | Publica certificados de usuário e CRLs em AD DS.             |
| Não suporta modelos de certificado.                          | Emite certificados com base em modelo de certificado.        |
| Todas as solicitações de certificados são mantidas pendentes até a aprovação do administrador. | Dá suporte ao registro automático para emitir certificados.  |

No processo de restauração do backup do AD CS também devemos realizar o backup do registro no regedit no seguinte caminho: HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Services\CertSvc\Configuration.

**Configuração das permissões do modelo de certificado**

| Permissão           | Descrição                                                    |
| ------------------- | ------------------------------------------------------------ |
| Controle total      | Permite que um usuário, um grupo ou um computador designado modifique todos os atributos (inclusive propriedades e permissões). |
| Leitura             | Permite que um usuário, um grupo ou um computador designado leia o certificado no AD DS ao se registrar. |
| Gravação            | Permite que um usuário, um grupo o um computador designado modifique todos os atributos, excetos permissões. |
| Registro            | Permite que um usuário, um grupo ou um computador designado se registr no modelo de certificado. |
| Registro automático | Permite que um usuário, um grupo ou um computador designado receba um certificado pelo processo de registro automático. |

**Métodos de registro de certificado**

| Método                 | Uso                                                          |
| ---------------------- | ------------------------------------------------------------ |
| Registro automático    | Para automatizar a solicitação, a recuperação e o armazenamento de certificados para computadores baseados em domínio |
| Registro manual        | Para solicitar certificados usando o console Certificados ou o Certq.exe quando o solicitante não pode comunicar diretamente com a AC |
| Registro via Web na AC | Para pedir certificados de um site que está localizado em uma AC.<br /> Para Emitir certificados quando o registro automático não estiver mais disponível. |
| Registrar em nome de   | Para fornecer a equipe de TI os direitos de solicitação de certificados em nome de outro usuário (Agente de Registro). |

**Visão geral do registro automático de certificado**

- Um modelo de certificado é configurado para conceder as permissões (Permissão: Registro e Registro automático) aos usuários que recebem os certificados.
- A AC é configurada para emitir o modelo
- Um objeto de Politica de Grupo do AD DS deve ser criado para habilitar o registro automático.
- O GPO deve esta vinculado ao site, o domínio ou a unidade organizacional apropriada.
- O usuário ou computador recebe os certificados durante o próximo intervalo de atualização da Politica de Grupo.

**O que é um agente de registro?**

- Um agente de registro é uma conta de usuário utilizada para solicitar certificados em nome de outra conta de usuário.
- Um agente de registro deve ter um certificado baseado no modelo do agente agente de registro.
- Os agentes de registro geralmente são membros da empresa ou dos departamentos de segurança de TI.
- Você pode limitar o escopo de um agente de registro:
  - Usuários ou grupos de segurança específicos.
  -  Modelos de certificados específicos.

**Visão geral de arquivamento de chave**

É de fundamental importancia os certificados digitais e suas chaves correspondentes serem mantidas em segurança. Se você perder as chaves que foram usadas para criptografar arquivos de dados, talvez não possa mais acessar esses arquivos.  

- As chaves podem ser comprometidas nas seguintes situações:
  - Você reinstalou o sistema operacional.
  - Um disco rígido foi corrompido.
  - O computador de um usuário foi roubado ou perdido.
  - o perfil de trabalho de um usuário foi corrompido ou excluído.
- Você deve configurar o arquivamento da chave na AC e no modelo de certificado.
- A recuperação da chave é um processo de duas fases :
  - Restauração da chave
  - Recuperação da chave

Após emitir o certificado de agente de recuperação e adicionar ao servidor como modelo de certificado de usuário devemos ir na propriedades da CA e habilitar a opção de recuperação de chave em **Agente de Recuperação** e selecionar o certificado vinculado anteriormente.
Nas propriedades de um modelos de certificado vamos em **tratamento de solicitação** para  **Arquivar a chave privada de criptografia de requerente**. Também nessa opção podemos **Permitir que a chave seja exportada**.
**Em Nome do requerente** - Ao emitir um certificado para usuário se não estivermos utilizando o campo e-mail nas propriedades da conta de usuário devemos desmarcar a opção que solicita o e-mail.

Para recuperar o certificado perdido devemos utilizar a ferramenta certutil.exe. 

```powershell
# Comonado para recuperaçao de certificado
certutil -getkey <nummero_de_serie_do_certificado_sem_espaco> ArquivoRecuperao
# Convertendo o arquivo recuperado na extenção valida
Certutil -recoverkey ArquivoRecuperado Certificado.pfx
```



## *AD FS*

​	Comando para criar a chave raiz do KDS do AD FS

```powershell
Add-KdsRootKey -EfefectivTime ((Get-Date).addhours(-10))
```

## *AD Sync*

Comandos do powershell usado com o AD Sync do Microsoft Azure.

```powershell
# Consultando tarefas agendadas de sincronização
Get-ADSyncScheduler
# Alterando o intervalo de sincronização
Set-ADSyncScheduler -CustomizadSymcCycleInterval 01:00:00
# Startando a tarefa de sincronização
Start-ADSyncSyncCycle -Policytype delta
```

## *Desfragmentação do AD DS*

Para a realização dessa operação o serviço de AD DS deve ser parado na console services.msc

```powershell
# com o prompt de comando elevado digita
ntdsutil
active instance ntds
files
compact to c:\
# A desfragmentação se inicia...
# Em seguida é #necessario copiar os seguintes arquivos
copy "C:\ntds.dit" "C:\Windows\NTDS\ntds.dit"
# E deletar o seguinte arquivo
del C:\Windows\NTDS\*.log
```

Após esses procedimentos o serviço do AD DS pode ser iniciado.

## *Lixeira do AD DS*

Requisito mínimo

- Nível funcional da floresta Windows Server 2008 R2

## *Backup do AD DS*

O backup do AD DS pode ser realizado por três ferramentas:

- Backup do Windows Server
- Backup do Microsoft Azure
- Data Protection Manager

Os backups dos AD DS pode ser realizado de duas maneiras?:

- Autoritativa - Quando à necessidade de que a partir desse backup seja restaurado alguma unidade organizacional que foi excluída acidentalmente.
- Não autiritativa - Quando somente é restaurada a função de AD DS de um servidor com problema.

Para iniciar um AD DS no modo DSRM é necessário o seguinte procedimento no prompt de comando:

```powershell
bcdedit /set safeboot dsrepair
```

Após a reinicialização do servidor deve fazer login com a conta de Administrador e com a senha que foi definida para o DSRM no momento da instalação da função do AD DS.
Partindo do pressuposto que foi realizado o backup utilizando a ferramenta Backup do Windows Server devemos executar os seguintes procedimentos no prompt de comando: 

```powershell
# Listando backups em uma Unidade E: no nosso exemplo
wbadmin get versions -backuptarget:e: -machine:srv01
# Restaurando o backup do Systemstate com data 25/04/2020-17:07.
wbadmin start Systemstaterecovery -version:26/04/2020-17:07 -backuptarget:e: -machine:srv01
# Confirme todas as proximas telas...
# Após a reinialização o retore esta concluido.
```

Para realizar a restauração de modo autoritativo após a restauração do Windows, deve entrar no prompt de comando e executar os seguintes procedimentos:

```powershell
# Nesse exemplo vamos restaurar uma unidade organizacional chamada tecnologia de modo autoritativo para que sejá replicada para todos os controladores de doínio
ntdsutil
activate instace ntds 
authoritative restore  
restore subtree "ou=tecnologia,dc=shs,dc=local"
```

Para sair do modo DSRM devemos executar o seguinte procedimento no prompt de comando:

```powershell
bcdedit /deletevalue safeboot
```

