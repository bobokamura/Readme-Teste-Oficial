# Readme-Teste-Oficial

# README - Deploy OFICIAL(Produção)

Este guia fornece instruções detalhadas sobre como realizar um deploy da aplicação em produção. Certifique-se de seguir cada etapa com atenção para garantir um processo de deploy seguro e bem sucedido.

## Passos para o Deploy

1. **Merge na Branch Develop:**
    - No `Github` realize o Merge pull request da `branch nova` para a `develop`.

2. **Atualização da Versão no `application.properties`:**
    - Na `IDE`, em `application.properties`, altere a versão `info.app.version` conforme as seguintes diretrizes:
        - Em caso de FEATURE: Atual 2.20.5 --> Nova 2.21.0
        - Em caso de BUG: Atual 2.20.5 --> Nova 2.20.6   

4. **Commit e Push:**
    - Execute o comando `git log` para identificar o último commit relacionado ao versionamento, que no caso era o `Bumped to version 2.20.5`.
    - Faça um novo commit utilizando o padrão com a nova versão: `Bumped to version 2.21.0`.
    - Dê um `git push origin develop` para atualizar a branch develop.

5. **Pull Request para a Master:**
    - Crie um novo pull request da develop para a master com o nome da versão, ex: `v2.21.0`.
    - Realize o Merge pull request.

6. **Alteração da TAG da Master:**
    - Altere a TAG da master para refletir a nova versão.

7. **Atualização da Master e Geração do JAR:**
    - Faça o checkout para a branch master e dê um pull para atualizar conforme a develop.
    - Utilize o Maven para gerar o JAR: `mvn clean compile install`.

8. **FileZilla - Atualização dos JARs:**
    - `Endereço local`: apontar para diretório local onde está alocada a API e selecionar pasta `target`";
    - `Endereço remoto`: apontar para API que está sendo feito o deploy;
    - Renomear JAR atual(ex: ctfl-respostaapi-1.0.jar para ctfl-respostaapi-1.0.jar-10-07) com a data de modificação e excluir JAR mais antigo;
    - ***EM CASO DE CARD QUE VOLTOU NO TESTE, APAGAR A PRÓPRIA VERSÃO DE TESTE;***
    - Ao lado do `Endereço local`, no diretório da pasta `target`, mover o JAR gerado anteriormente para onde estão os JAR's atuais(verifique se está movendo o tipo correto: `jar-arquivo`);
    - Utilize o FileZilla para transferir o JAR gerado para o servidor.
    - Remova o JAR antigo e renomeie o novo conforme a data de modificação.

9. **AWS - Atualização na Instância:**
    - Acesse a AWS e copie o Public IPv4 address da instância EC2.
      - `EC2` -> `Instances` -> procure a API desejada;

10. **Abra o terminal:**
    - No terminal, acesse o diretório da Amazon e execute o comando:
      - `ssh -i ~/Amazon/api_auth.pem ubuntu@<COLE_O_PUBLIC_IPV4_ADDRESS>`;
    - Digite YES se for a primeira vez;
    - Digite `ls -la` para verificar o JAR que está rodando;
    - Execute o script de deploy: `./deploy.sh`.
    - Abra o navegador, cole o `<Public_IPv4_address>` + `:<PORTA>` + `/actuator/info`.
      - Por exemplo: [http://54.242.230.161:8888/actuator/info](http://54.242.230.161:8888/actuator/info).

10. **AWS - Criação de Imagem e Atualização do Launch Template:**
    - Crie uma imagem da instância atualizada na AWS.
    - Atualize o Launch Template com a nova imagem, definindo-a como a versão padrão.

11. **Forçar Atualização na Auto Scaling:**
    - No Auto Scaling Groups, ajuste a Desired capacity para forçar a subida da nova versão.
    - Verifique as instâncias em "Instance management" para confirmar as versões atualizadas.

**Observação:** Certifique-se de seguir os passos com cuidado e sempre faça testes antes de aplicar em ambientes de produção. Este guia assume familiaridade com as ferramentas e conceitos mencionados.
