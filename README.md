# Kerberized Web Server

Projeto final da disciplina de ECOE10 - Tópicos Especiais em Segurança da Informação ofertada pela Universidade Federal de Itajubá (UNIFEI).

O objetivo é garantir segurança e autenticação a um Web Server, de maneira que somente os usuários previamente cadastrados possam acessar o contúdo oferecido, sem que suas informações de acesso estejam disponíveis.

Foram instanciados dois servidores: um Kerberos Key Distribution Center (KDC) e um Web Server Apache. Mais especificamente, trata-se de duas máquinas virtuais Linux Lubuntu 18.04.1 LTS. Além disso, uma máquina virtual foi usada como Web Client, requisitando conteúdo do Web Server. Todas as máquinas foram virtualizadas com Oracle VirtualBox 5.2.8.

Segue a estrutura de rede implementada para o projeto:

| Endereço IP  | Máscara | Gateway Padrão | Hostname | Função |
| :-----------: | :--: | :-----------: | -------- | :--: |
| 10.0.0.1 | 255.255.255.0 | 10.0.0.1  | kerberos.unifei.edu | Key Distribution Center (KDC) |
| 10.0.0.2 | 255.255.255.0 | 10.0.0.1  | webserver.unifei.edu | Web Server (Apache) |
| 10.0.0.3 | 255.255.255.0 | 10.0.0.1  | client.unifei.edu | Web Client (Browser) |

A seguir é descrito como deve ser feito o setup de cada máquinas, observe que o projeto no git é estruturado com as pastas **kerberos**, **client** e **webserver**, que serão citadas posteriormente.

# Configurações gerais

1. Criar as máquinas virtuais no VirtualBox.
2. Adicionar um adaptador de rede configurado com uma rede interna comum a todas as máquinas. É importante que o endereço MAC seja diferente para todas as máquinas envolvidas.
3. Definir um fuso-horário comum o qual será utilizado durante a instalação do SO de cada máquina.
4. Segue, portanto, a instalação de cada máquina virtual, iniciando pela referente ao KDC.

### Instalação das máquinas

1. Durante a instalação, definir o nome da máquina de acordo com o hostname e definir o fuso-horário de acordo com o pré-estabelecido.
2. Após a instalação, checar o hostname através do comando `$hostname`. Caso o hostname difira do desejado, alterá-lo através do comando `$ hostnamectl set-hostname hostname_da_máquina`.
3. Configurar o endereço IP manualmente como estático, além da máscara e gateway padrão.
4. Copiar o arquivo `hosts` fornecido da pasta para /etc/, sobrescrevendo o atual. 

# Key Distribution Center (KDC)

1. Instalar o servidor do Kerberos com `$ sudo apt-get install krb5-kdc krb5-admin-server`.
2. Durante a instalação, o nome do realm requisitado deve ser preeenchido com **WEBSERVER.UNIFEI.EDU**.
3. A seguir são requisitados os hostnames do servidor do Kerberos e do servidor administrativo. Preencher ambos inputs com **kerberos.unifei.edu**.
4. Criar um realm com `$ krb5_newrealm`.
5. Criar os usuários, incluindo o administrativo.`$ kadmin.local` iniciará um terminal de administração do Kerberos. Dentro do mesmo, inserir os usuários (principals) com os comandos abaixo. A cada usuário, é necessário definir uma senha.  
  
  `addprinc webserver`  
  `addprinc webserver/admin`  
  `quit`  
  
6. Descomentar a última linha do arquivo /etc/krb5kdc/kadm5.acl, para definir os usuários com permissão de administrador.

# Web Server (Apache)

1.
