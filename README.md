# Image Containers
A seguir está documentado as origens das imagens containers

## LDAP
  - Para o server LDAP, foi usado a imagem ```bitnami/openldap:latest``` encontrada em https://hub.docker.com/r/bitnami/openldap sob a licença https://www.apache.org/licenses/LICENSE-2.0
 - Mais informações de como foi construído o container, bem como, forma de configurá-lo, pode ser encontrado em: 
    - https://github.com/bitnami/containers/tree/main/bitnami/openldap
 - Foi configurado um volume, com pasta local, com objetivo de manter o dados caso o container venha a cair.

## PHP LDAP ADMIN
 - Ferramenta para se conectar visualmente ao server LDAP via http/web com objetivo de listar os objetos, principalmente usuários.
 - Para o server de Administração LDAP, foi usado a imagem ```leenooks/phpldapadmin:2.0.0-dev``` encontrada em https://hub.docker.com/r/leenooks/phpldapadmin sob a licença https://www.gnu.org/licenses/old-licenses/gpl-2.0.pt-br.html
 - Mais informações de como foi construído o container, bem como, forma de configurá-lo, pode ser encontrado em: 
    - https://github.com/leenooks/phpLDAPadmin
 - Este container não é obrigatório, é só um auxiliar para verificar o LDAP visualmente.

# Procedimentos iniciais:
 - Para funcionar o PHP LDAP ADMIN, antes de iniciar o docker-compose, é necessário gerar a chave de acesso, seguido o comando:
    - ```docker run -it --rm leenooks/phpldapadmin:2.0.0-dev ./artisan key:generate --show```
    - Se gerar erros, ignore, o que importa é a chave.
    - Com a chave gerada, semelhante esta: ```base64:oOvBcyc2B936ONUxnz5CUb6brgM59X74U7o8mYNMwjw=``` você terá que colocar o conteúdo dela na variável APP_KEY no environment do container phpldapadmin, dessa forma: ```APP_KEY="base64:oOvBcyc2B936ONUxnz5CUb6brgM59X74U7o8mYNMwjw="```
    - Após isso, poderá seguir os próximos passos e iniciar o docker-compose.

# Iniciando o docker-compose
 - O docker-compose.yaml já está todo configurado de forma básica para funcionar, basta levantá-lo com o comando a seguir:
    - ```docker-compose up``` ou ```docker-compose up -d``` para rodar em background.
 - Para derrubas os containers, basta um CTRL+C se tiver na sessão ou em caso de background ```docker-compose down```

## Testando LDAP pelo PHP LDAP ADMIN
 - Ao utilizar o PHP LDAP ADMIN, você acessará o site: http://localhost:8001 e como já está configurado via variáveis a autenticação de admin, você já terá acesso à árvore do LDAP.

## Testando com ldapsearch local
 - Se você não tiver o ldapsearch, instale localmente ...
 - A seguir está demonstrado alguns exemplos de como fazer filtros no LDAP se autenticando por linha de comando.
  
### Testando login com admin
 - ```ldapsearch -H ldap://localhost:1389 -x -D "cn=admin,dc=example,dc=com" -b "dc=example,dc=com" -w "admin@123" -s sub "(&(objectClass=person)(uid=julianorinaldi))"```

### Testando pesquisa com usuário criado
 - ```ldapsearch -H ldap://localhost:1389 -x -D "cn=julianorinaldi,ou=users,dc=example,dc=com" -b "dc=example,dc=com" -w "julianorinaldi@123" -s sub "(&(objectClass=person)(uid=wmsaude))"```
 - Para funcionar a search acima, você terá que criar o usuário ```julianorinaldi```, isso pode ser realizado com arquivos ```.ldif``` conforme há um exemplo na raiz ```add_user.ldif```.
 - Usando o comando ```ldapadd -x -D "cn=admin,dc=example,dc=com" -w "admin@123" -H ldap://localhost:1389 -f add_user.ldif```, será criado o usuário ```julianorinaldi```, ou você poderá fazer isso via ferramenta PHP LDAP ADMIN importante o arquivo LDIF.

 # Reconfigure as variáveis de ambiente
  - Observe as variáveis abaixo no container LDAP, nelas você configura o LDAP conforme sua necessidade
  ``` 
    environment:
      - LDAP_ROOT=dc=example,dc=com              # base do LDAP - Representa a identificação do seu server LDAP
      - LDAP_ADMIN_DN=cn=admin,dc=example,dc=com # dn do admin - Representa o path completo do user admin
      - LDAP_ADMIN_USERNAME=admin                # username do admin - Representa o login do admin
      - LDAP_ADMIN_PASSWORD=admin@123            # password do admin - Representa a senha do admin
      - LDAP_USERS=euax,wmsaude                  # username dos usuários - Representa os usuários que deseja criar automaticamente, separado por virgula.
      - LDAP_PASSWORDS=euax@123,wmsaude@123      # password dos usuários - Representa as senhas dos usuários criados automaticamente.
      #- LDAP_LOGLEVEL=-1                        # level de log - Representa o level de log, está comentado pois gera muito log, o -1 representa tudo.
  ```
  - Observe as variáveis abaixo no container PHP LDAP ADMIN, você precisa deixar que o PHP LDAP ADMIN conheça o server LDAP
  ``` 
    environment:
      - APP_KEY="base64:oOvBcyc2B936ONUxnz5CUb6brgM59X74U7o8mYNMwjw=" # chave de acesso gerada conforme documentação acima.
      - APP_URL="http://localhost:8001"                               # url do PHP LDAP ADMIN
      - CACHE_DRIVER="memcached"                                      # driver de cache
      - LDAP_CACHE=true                                               # habilita o cache
      - MEMCACHED_START=true                                          # habilita o memcached
      - APP_TIMEZONE=America/Sao_Paulo                                # timezone do PHP LDAP ADMIN
      - LDAP_BASE_DN=dc=example,dc=com                                # base do LDAP    
      - LDAP_HOST=openldap                                            # host do LDAP
      - LDAP_PORT=1389                                                # porta do LDAP
      - LDAP_USERNAME=cn=admin,dc=example,dc=com                      # dn do admin
      - LDAP_PASSWORD=admin@123                                       # password do admin
``` 