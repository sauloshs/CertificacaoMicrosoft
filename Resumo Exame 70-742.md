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
