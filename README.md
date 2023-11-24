# README - Deploy OFICIAL(Produção)

Este guia fornece instruções detalhadas sobre como realizar um deploy da aplicação em produção. Certifique-se de seguir cada etapa com atenção para garantir um processo de deploy seguro e bem sucedido.

## Passos para o Deploy

1. **Merge na Branch Develop:**
    - No *Github* realize o Merge pull request da *branch nova* para a *develop*.

2. **Atualização da Versão no `application.properties`:**
    - Na *IDE*, em `application.properties`, altere a versão `info.app.version` conforme as seguintes diretrizes:
        - Em caso de *FEATURE*:
            > *Atual: 2.20.5* --> *Nova: 2.21.0*
        - Em caso de *BUG*:
            > *Atual: 2.20.5* --> *Nova: 2.20.6*   

3. **Commit e Push:**
    - Execute o comando ```git log``` para identificar o último commit relacionado ao versionamento, que no caso era o
        > `Bumped to version 2.20.5`.
    - Faça um novo commit utilizando o padrão com a nova versão:
        > `Bumped to version 2.21.0`.
    - Dê um `git push origin develop` para atualizar a branch develop.

4. **Pull Request para a Master:**
    - Crie um novo pull request da develop para a master com o nome da versão, ex: `v2.21.0`.
    - Realize o Merge pull request.

5. **Alteração da TAG da Master:**
    - Altere a TAG da master para refletir a nova versão.

6. **Atualização da Master e Geração do JAR:**
    - Faça o checkout para a branch master e dê um pull para atualizar conforme a develop.
    - Utilize o Maven para gerar o JAR:
        ```
        mvn clean compile install
        ```

7. **FileZilla - Atualização dos JARs:**
    - `Endereço local`: apontar para diretório local onde está alocada a API e selecionar pasta *"target"*;
    - `Endereço remoto`: apontar para API que está sendo feito o deploy;
    - Renomear JAR atual:
      > - ctfl-respostaapi-1.0.jar
    - Para:
      > - ctfl-respostaapi-1.0.jar-10-07
    - Altere a data de modificação e excluia o JAR mais antigo;
    - ***EM CASO DE CARD QUE VOLTOU NO TESTE, APAGAR A PRÓPRIA VERSÃO DE TESTE;***
    - Em `Endereço local`, no diretório da pasta `target`, mover o JAR gerado anteriormente para onde estão os JAR's atuais(verifique se está movendo o tipo correto: `jar-arquivo`);
    - Utilize o FileZilla para transferir o JAR gerado para o servidor.
    - Remova o JAR antigo e renomeie o novo conforme a data de modificação.

8. **AWS - Atualização na Instância:**
    - Acesse a AWS e copie o Public IPv4 address da instância EC2.
      - `EC2` -> `Instances` -> procure a API desejada;

9. **Abra o terminal:**
    - No terminal, acesse o diretório da Amazon e execute o comando:
      - `ssh -i ~/Amazon/api_auth.pem ubuntu@<COLE_O_PUBLIC_IPV4_ADDRESS>`;
    - Digite YES se for a primeira vez;
    - Comando `ls -la` para verificar o JAR que está rodando;
    - Execute o script de deploy: `./deploy.sh`.
    - Abra o navegador, cole o `<Public_IPv4_address>` + `:<PORTA>` + `/actuator/info`.
      - Por exemplo: [http://54.242.230.161:8888/actuator/info](http://54.242.230.161:8888/actuator/info).

10. **AWS - Criação de Imagem e Atualização do Launch Template:**
    - Crie uma imagem da instância atualizada na AWS:
        -  `Images` -> `AMIs`;
        -  Digite a API na busca e filtre a data de criação mais recente e selecione;
        -  Na aba `Detais`, copie o `AMI name` e altere para a data e versão atual;
        -  Ex:
            - AMI ANTERIOR:
                > *ami-api-pedido-27-10-2023-v2.20.5*
            - AMI NOVA:
                > *ami-api-pedido-01-11-2023-v2.20.6*
    - Em EC2:
        - `Instances` -> procure a API desejada;
        - Clique na instância e em `Actions`, selecione `Image and templates` -> `Create image`.
    - Criando a imagem:
        - `Image name` -> cole o nome da AMI NOVA criada.
            > Ex: *ami-api-pedido-01-11-2023-v2.20.6*.
		- `Image description - optional` -> cole novamente a *ami-api-pedido-01-11-2023-v2.20.6*;
		    > [!!!] No `reboot` -> *MARCAR O CHECK "ENABLE" E VERIFICAR SE O "Delete on termination" TAMBÉM ESTÁ MARCADO*.
		- `Add new tag` -> Em `Key` insira `"Name"` e cole a AMI NOVA, ex: *ami-api-pedido-01-11-2023-v2.20.6*;
		- Verifique os campos novamente e clique em `Create image`, vai demorar um pouco até que a imagem seja criada e o status fique `"Available"` na aba AMIs.   
    - Com a imagem criada:
        - `Instances` -> `Lauch Templates`, procure a API desejada e selecione;
		- Em `Launch template version details` -> `Actions` -> `Modify template (Create new version)`;
		- No campo `Template version description` -> Cole a AMI NOVA, porém sem o "ami-".
            > Ex: *api-pedido-01-11-2023-v2.20.6*.
		- Em `Application and OS Images (Amazon Machine Image)` -> cole a AMI NOVA e *ENTER* para buscar;
            > Ex: *ami-api-pedido-01-11-2023-v2.20.6*        
		- Agora em `My AMIs`, clique em `Select` na imagem buscada e em seguida `Create template version`;
		- Novamente em `Launch template version details` -> `Actions` -> `Set default version`, selecione a versão criada e clique em `Set as default version`. 
    
11. **Forçar Atualização na Auto Scaling se necessário:**
    - No Auto Scaling Groups, ajuste a Desired capacity para forçar a subida da nova versão.
        - `Auto Scaling` -> `Auto Scaling Groups`;
        - Busque a API que deseja e clique nela;
        - Em `Instance management` -> `Lauch template/configuration` -> verifique a versão que está rodando;
    - Para forçar e subir a nova versão:
        - `Detais` -> `Edit` -> Em base na `Maximum capacity`, altere a `Desired capacity`.
        - Ex: Se a Maximum capacity esteja setada para até 10 máquinas e a Desired capacity estiver rodando 5, suba "até" o limite e clique em Update;
		- Volte em `Instance management` -> `Lauch template/configuration` -> verifique as versões novas que deverão subir.

12. **Observação:**
    - Certifique-se de seguir os passos com cuidado e sempre faça testes antes de aplicar em ambientes de produção. Este guia assume familiaridade com as ferramentas e conceitos mencionados.
