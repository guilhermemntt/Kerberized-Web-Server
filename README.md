# Kerberized Web Server

Projeto final da disciplina de ECOE10 - Tópicos Especiais em Segurança da Informação ofertada pela Universidade Federal de Itajubá (UNIFEI).

O objetivo é garantir segurança e autenticação a um Web Server, de maneira que somente os usuários previamente cadastrados possam acessar o contúdo oferecido, sem que suas informações de acesso estejam disponíveis no tráfego de rede.

Foram instanciados dois servidores: um Kerberos Key Distribution Center (KDC) e um Web Server Apache. Mais especificamente, trata-se de duas máquinas virtuais Linux Lubuntu 18.04.1 LTS. Além disso, uma máquina virtual foi usada como Web Client, requisitando conteúdo do Web Server. Todas as máquinas foram virtualizadas com Oracle VirtualBox 5.2.8.

Segue a estrutura de rede implementada para o projeto:

| Endereço IP  | Máscara | Gateway Padrão | Hostname | Função |
| :-----------: | :--: | :-----------: | -------- | :--: |
| 10.0.0.1 | 255.255.255.0 | 10.0.0.1  | kerberos.unifei.edu | Key Distribution Center (KDC) |
| 10.0.0.2 | 255.255.255.0 | 10.0.0.1  | webserver.unifei.edu | Web Server (Apache) |
| 10.0.0.3 | 255.255.255.0 | 10.0.0.1  | client.unifei.edu | Web Client (Browser) |

A seguir é descrito como deve ser feito o setup de cada máquina, observe que o projeto no git é estruturado com as pastas **kerberos**, **client** e **webserver**, que serão citadas posteriormente. Essas pastas contêm arquivos de modelo, cópias de respectivos arquivos funcionais de cada máquina. Recomenda-se não copiá-los, mas seguir as instruções seguintes e compará-los com os resultantes das instruções, a fim de evitar discrepâncias.

# Configurações gerais

1. Criar as máquinas virtuais no VirtualBox. Recomenda-se utilizar nomes sugestivos de acordo com suas respectivas funções.
2. Adicionar um adaptador de rede configurado com uma rede interna comum a todas as máquinas. É importante que o endereço MAC seja diferente para todas as máquinas envolvidas.
3. Definir um fuso-horário comum o qual será utilizado durante a instalação do SO de cada máquina.

### Instalação do SO nas máquinas

1. Durante a instalação, definir o nome da máquina de acordo com o hostname e definir o fuso-horário de acordo com o pré-estabelecido.
2. Após a instalação, checar o hostname através do comando `$hostname`. Caso o hostname difira do desejado, alterá-lo através do comando `$ hostnamectl set-hostname hostname_da_máquina`.
3. Configurar o endereço IP manualmente como estático, além da máscara e gateway padrão.
4. Copiar o arquivo `hosts` fornecido da pasta para /etc/, sobrescrevendo o atual. 
  
Até esse ponto, as máquinas devem ser acessíveis entre si pelo endereço IP e pelo hostname. Continuar a configuração das mesmas apenas se for possível "alcançar" cada máquina pelo comando ping, por exemplo.  
  
Segue, portanto, a instalação de cada máquina virtual, iniciando pela referente ao KDC.

# Key Distribution Center (KDC)

1. Instalar o servidor do Kerberos com `$ sudo apt-get install krb5-kdc krb5-admin-server`.
2. Durante a instalação, o nome do realm requisitado deve ser preeenchido com **WEBSERVER.UNIFEI.EDU**.
3. A seguir são requisitados os hostnames do servidor do Kerberos e do servidor administrativo. Preencher ambos inputs com **kerberos.unifei.edu**.
4. Criar um realm com `$ sudo krb5_newrealm`. Forneça uma senha quando requisitado.
5. Criar os usuários, incluindo o administrativo.`$ sudo kadmin.local` iniciará um terminal de administração do Kerberos. Dentro do mesmo, inserir os usuários (principals) com os comandos abaixo. A cada usuário, é necessário definir uma senha.  
  
  `addprinc webserver`  
  `addprinc webserver/admin`  
  `addprinc teste`  
  `quit`  
  
6. Descomentar a última linha do arquivo /etc/krb5kdc/kadm5.acl, para definir os usuários com permissão de administrador.
7. Reiniciar o serviço referente ao servidor administrador do Kerberos com `$ sudo systemctl restart krb5-admin-server.service`

# Web Server (Apache)

1. Instalar o cliente do Kerberos com `$ sudo apt-get install krb5-user`.
2. Durante a instalação, o nome do realm requisitado deve ser preeenchido com **WEBSERVER.UNIFEI.EDU**.
3. A seguir são requisitados os hostnames do servidor do Kerberos e do servidor administrativo. Preencher ambos inputs com **kerberos.unifei.edu**.
4. Instalar o servidor HTTP Apache Server com `$ sudo apt-get install apache2`.
5. Instalar o módulo Apache de autenticação do Kerberos com `$ sudo apt-get install libapache2-mod-auth-kerb`.
6. Adicionar o principal responsável pelo servidor web com `$ sudo kadmin -p webserver/admin -q "addprinc -randkey HTTP/webserver.unifei.edu"`.
7. Adicionar o arquivo *keytab* ao Kerberos com `$ sudo kadmin -p webserver/admin -q "ktadd -k /etc/apache2/http.keytab HTTP/webserver.unifei.edu"`
8. Redefinir a propriedade do arquivo *keytab* gerado com `$ sudo chown www-data /etc/apache2/http.keytab`.
9. Iniciar o principal, requisitando o TGT oo KDC com `$ sudo kinit -k -t /etc/apache2/http.keytab HTTP/webserver.unifei.edu`.
10. Se tudo ocorrer de acordo com o previsto, o comando `$ sudo klist` deve exibir o(s) tickets adquiridos.
11. No arquivo /etc/apache2/apache2.conf, adicionar as seguintes linhas:  
  
`LoadModule auth_kerb_module /usr/lib/apache2/modules/mod_auth_kerb.so`  
`<Location />`  
  `AuthType Kerberos`  
  `AuthName "Apache Web Server"`  
  `KrbMethodNegotiate on`  
  `KrbMethodK5Passwd off`  
  `Krb5Keytab /etc/apache2/http.keytab`  
  `Require user teste@WEBSERVER.UNIFEI.EDU`  
`</Location>`  


12. Reiniciar o serviço do Apache com `$ service apache2 force-reload`.

# Web Client (Browser)

1. Instalar o cliente do Kerberos com `$ sudo apt-get install krb5-user`.
2. Durante a instalação, o nome do realm requisitado deve ser preeenchido com **WEBSERVER.UNIFEI.EDU**.
3. A seguir são requisitados os hostnames do servidor do Kerberos e do servidor administrativo. Preencher ambos inputs com **kerberos.unifei.edu**.
4. Acessar o endereço `about:config` no navegador Firefox.
5. Aceitar os riscos e continuar.
6. Pesquisar na barra de pesquisas por *network.n*.
7. Redefinir os valores das variáveis *network.negotiate-auth.delegation-uris* e *network.negotiate-auth.trusted-uris* para **webserver.unifei.edu**.

# Funcionamento

1. Na máquina Web Client, tentar acessar através do Firefox o endereço **webserver.unifei.edu**. Observar que o usuário não está autorizado e a resposta do servidor web é um código 401 (Unauthorized).
2. Requisitar a autorização ao KDC `$ kinit teste`.
3. Tentar acessar novamente a página. Agora o acesso é permitido.

## Observações

1. Múltiplos usuários podem ter permissão de concessão dos tickets. Basta adicionar na tag `Location` adicionada ao arquivo /etc/apache2/apache2.conf demais usuários, tal como `Require user nome_do_usuario1@WEBSERVER.UNIFEI.EDU nome_do_usuario2@WEBSERVER.UNIFEI.EDU`. Da mesma forma, basta simular outras máquinas como Web Clients, adicionando-as devidamente à rede interna e requisitando os tickets de acesso ao KDC.

# Demonstração

![Alt text](sample.gif?raw=true "Demonstração")

# Informações sobre o projeto

## Grupo

* Guilherme Marques Netto - 33419  
* Ivan Lacerda de Rezende - 30704  
* Rafael Miranda Ferrari Picolo - 33571  





