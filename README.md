# Criar POC de clusterização com Tomcat e load balance

Como criar: (com base [nesse tutorial](https://www.ramkitech.com/2015/10/docker-tomcat-clustering.html))

Tenho as seguintes anotações de como rodar utilizando os  arquivos descritos a seguir em cada nó (pra resolver os problemas que enfrentei quando eu decidi em criar um unico container antes de sair criando 3 com load balance):

1. A pasta `webapps` vem vazia e seu conteúdo está em `webapps.dist` o que gera o erro 404 ao acessar a home do container;
2. Para configurar o container como cluster precisa adicionar a tag `<Cluster/>` no arquivo *conf/web.xml* (usei a config disponível na [doc online do tomcat](https://tomcat.apache.org/tomcat-9.0-doc/cluster-howto.html)). E usei o mesmo arquivo nos outros nós (linkando pelo `volume` do docker);
3. Para ter acesso a pagina tomcat-ip:port/manager (onde faz o deploy de aplicações) precisa configurar um usuário e senha junto com a role de permissões em *conf/tomcat-users.xml* ([link pra doc](https://tomcat.apache.org/tomcat-9.0-doc/manager-howto.html));
4. A tag `<Valve/>`  é responsável por configurar o acesso remoto ao container e a aplicação publicada, então pra acessar fora do container docker, precisa alterar a mesma que fica no arquivo *conf/context.xml* (no meu caso eu só comentei mesmo pra fazer funcionar - [link da doc](https://tomcat.apache.org/tomcat-9.0-doc/config/valve.html#Remote_Address_Valve));
5. Ao criar o container com a imagem do nginx precisa criar links dos nós do tomcat, pra isso é necessário configurar na criação do container e dentro do arquivo *default.conf* do próprio nginx. Assim ele vai conseguir redirecionar as requisições pra cada nó do tomcat;
6. Não Consegui fazer o deploy pelo link do nginx, só pelo link de um dos containers;
7. o arquivo final docker-compose.yml (sim, o conteúdo de server.xml, contex.xml e tomcat-users-xml tem que ter o mesmo em todos os nós, o próprio docker gerencia o IP de cada container):
    
    ```yaml
    portal:
      image: nginx
      ports:
       - "8080:80"
      volumes:
       - /home/adriane/dev/tomcat/default.conf:/etc/nginx/conf.d/default.conf
      links:
       - node01:node01
       - node02:node02
       - node03:node03
    node01:
      image: tomcat
      volumes:
       - <host-path-to-file>/server.xml:/usr/local/tomcat/conf/server.xml
       - <host-path-to-file>/tomcat-users.xml:/usr/local/tomcat/conf/tomcat-users.xml
       - <host-path-to-file>/context.xml:/usr/local/tomcat/webapps/manager/META-INF/context.xml
       - <host-path-to-file>/webapps:/usr/local/tomcat/webapps/
    node02:
      image: tomcat
      volumes:
       - <host-path-to-file>/server.xml:/usr/local/tomcat/conf/server.xml
       - <host-path-to-file>/tomcat-users.xml:/usr/local/tomcat/conf/tomcat-users.xml
       - <host-path-to-file>/context.xml:/usr/local/tomcat/webapps/manager/META-INF/context.xml
       - <host-path-to-file>/webapps:/usr/local/tomcat/webapps/
    node03:
      image: tomcat
      volumes:
       - <host-path-to-file>/server.xml:/usr/local/tomcat/conf/server.xml
       - <host-path-to-file>/tomcat-users.xml:/usr/local/tomcat/conf/tomcat-users.xml
       - <host-path-to-file>/context.xml:/usr/local/tomcat/webapps/manager/META-INF/context.xml
       - <host-path-to-file>/webapps:/usr/local/tomcat/webapps/
    ```
    

Para ver qual o IP de cada nó e do nginx é só usar o comando `inspect` do docker (com `| grep` para exibir só o IP):

```bash
docker inspect tomcat_portal_1 | grep IPAddress
```

Ai é só acessar `<tomcat-ip>:<port>` ou `nginx-ip`  no navegador.

Da pra criar os nós sem o nginx, é só tirar a tad portal do yml e rodar, mas pra acessar cada nós vc precisa usar o IP de cada um.
