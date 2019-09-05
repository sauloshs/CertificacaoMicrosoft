# *Resumo do Exame 70-742*

-------

## *Active Directory*

​	O que é o Active Directory: É um serviço de diretório que pode ser explicado com varias ilustrações que, em minha opinião, mais demostra o verdadeiro sentido de um serviço de diretório é a figura de uma lista telefônica ou uma agenda pessoal.
​	Em nossa agenda podemos organizar, dias, semanas, meses e até anos, passando por pessoas, nomes, sobrenomes, datas de aniversario, dados importantes dentre outros.
​	O Serviço de diretório tem exatamente o mesmo sentido, o sentido de organizar e principalmente de ter um local centralizado de armazenamento para busca das informações necessárias para o dia a dia, para nossos trabalhos.

​	O AD DS é composto por componentes físicos e lógicos:

| Componentes lógicos | Componentes físicos           |
| ------------------- | ----------------------------- |
| Partições           | Controladores de domínio      |
| Esquema             | Repositório de dados          |
| Domínios            | Servidores de catálogo global |
| Árvores de domínio  | RODCs                         |
| Florestas           |                               |
| Sites               |                               |
| UOs                 |                               |

O que é uma Floresta?

O que são Unidades Organizacionais (OUs)?
	São itens onde podemos agrupar objetos afim de organizar por localidades, departamento, região, computadores, grupos e usuários. Além disso podemos vincular GPOs as OUs que vão determinar configurações a grupos de comutadores ou usuários específicos de uma determinada OU.
	Outra aplicação da OUs é a possibilidade de determinar delegação a um determinado usuário ou grupo de usuários para administrar objetos contidos nela.

- Use contêiner para agrupar objetos dentro de um domínio:
  - Não é possível aplicar GPOs a contêineres.
  - Contêineres são usados para objetos de sistemas e como a localização padrão para novos objetos.
- Cries OUs para:
  - Configurar objetos atribuindo GPOs a eles.
  - Delegar permissões administrativas.