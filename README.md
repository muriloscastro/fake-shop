# Desafio Fake Shop - DevOps - Resumo Pipeline
A criação da pipeline de CI / CD será feita utilizando a ferramenta GitHub Action.

O processo de Pipeline vai serguir os seguintes conceitos:

- Workflow - "Fluxo de trabalho" Vai ser a representação dos passos necessários para concluir tarefas e processos.

- Events - É o evento que vai gerar alguma execução, pull request, commit entre outros.

- Jobs - Representa um conjunto de execuções, ações e tarefas que vão ser executadas em uma determinada ordem.

- Steps - É aonde  é declarado os passos a passos das execuções, fica dentro do Jobs

- Actions - Ficam dentro dos steps e são as acões e tarefas que serão executadas

- Runners - Responsável por pegar a automação em executar em algum ambiete, windows, linuux ou algum lugar.

# Pontos de atenção
Para a execução dos próximos é importante verificar o status de execução do cluster, diretório da aplicação e garantir que as aplicações do ambiente de testes estão funcionando corretamente. corretamente. 

# Testes deployments
Para iniciar o processo de Pipeline, vamos precisar executar os seguintes itens: 
1) Verificar se o ambiente / deployment está funcionando no cluster "local". Se estiver ok, pode deletar o deployment.
    - Comando para iniciar deploy: **sudo kubectl apply -f k8s/deployment.yaml**
    - Comando para veriricar deploy: **sudo kubectl get all**
    - Testar acesso web no browser com o IP **local** do cluster **http://localhost**
    - Comando para  deletar o deployment: **sudo kubectl delete -f k8s/deployment.yaml**

2) Iniciar o cluster kubernets com 3 pools na digiral ocean "Maquina básica".

    2.1) Realizar o download do arquivo "Download config" no link Actions do cluster kubernetes na DigitalOcean

    2.2) Alterar o arquivo "config" de configuração do kubernets, localizado no diretório do usuário ~/.kube/config e adicionar o conteudo do arquivo "Download config" com as configuração de cluster.
    - Comando para editar o arquivo config: **vim ~/.kube/config**
    - O arquivo pode ser editado no visual code, bloco de notas entre outros, o exemplo acima foi editando o arquivo no ubuntu.

    2.3) Verificar se o ambiente / deployment está funcionando no cluster "DigitalOcean". Se estiver ok, pode deletar o deployment.
    - Comando para iniciar deploy: **sudo kubectl apply -f k8s/deployment.yaml**
    - Comando para veriricar deploy: **sudo kubectl get all**
    - Testar acesso web no browser com o IP **Exerno** do cluster **http://ip-exteno-cluster**
    - Comando para  deletar o deployment: **sudo kubectl delete -f k8s/deployment.yaml**

- Após a validação e garantindo que o deployment funcionou corretamnente, podemos iniciar o processo de automação utilizando o GitHub Action.

# Criar pipeline
1) Acessar o seu repositório no GitHub --> USUARIO/fake-shop-desafio 

	1.1) - Selecionar a aba Actions na barra superior do projeto no GitGub, proximo a aba pull requests. Vai abrir a pagina "<u>Get started with GitHub Actions"</u> com alguns templates de exemplo. 

    1.2) Criar o workflow do zero, sem utilizar template. Clicar no link <u>"set up a workflow yourself"</u> proximo ao campo de buscas "Search workflows". 
    
    1.3) Vai abrir um editor de texto, informando o diretório / estrutura do caminho do arquivo main.yaml. Neste arquivo vão ser adicionas as informações da pipeline.

# Arquivo de Pipeline 
O arquivo yaml neste exemplo, vai ser dividido basicamente em 3 blocos e o CI / CD será criado no mesmo workflow.

No primeiro bloco são adicionadas as informações de nome, especificação do evento, especificar branhces, criar tarefas entre outros.
Nos demais blocos vão ser realizadas as configurações dos passos, acções entre outros. 

- Inicio do arquivo<br>
name: CI-CD <br>
on:<br>
push:<br>
branches: ["main"]<br>
workflow_dispatch:<br>
jobs:<br>

- **bloco CI** <br>
  CI:
    runs-on: ubuntu-latest<br>
    steps:<br> 
- Action checkout para  obter o código no repositório <br>
      - name: Obtendo o codigo<br>
        uses: actions/checkout@v4.2.2<br>
- Action para logar no dockerhub <br>
      - name: Login to Docker Hub<br>
        uses: docker/login-action@v3<br>
        with:<br>
        username: \$\{\{ secrets.DOCKERHUB_USERNAME \}\} <br>
        password: \$\{\{ secrets.DOCKERHUB_TOKEN \}\} <br>
- Action build and push para logar no dockerhub <br>
      - name: Construcao e Envio da Imagem Docker <br>
        uses: docker/build-push-action@v6 <br>
        with: <br>
            context: ./src <br>
            push: true <br>
            file: ./src/Dockerfile <br>
            tags: | <br>
              muriloscastro/fake-shop-desafio:v${{ github.run_number }} <br>
              muriloscastro/fake-shop-desafio <br>

- **Bloco CD** <br>
 CD: <br>
    needs: [CI] <br>
    runs-on: ubuntu-latest <br>
    steps: <br>
- Action checkout para  obter o código no repositório <br>
      - name: Obtendo o codigo <br>
        uses: actions/checkout@v4.2.2 <br>
- Action de configuração do kubeconfig <br>
      - uses: azure/k8s\-set\-context@v4
        with: <br>
         method: kubeconfig <br>
         kubeconfig: \$\{\{ secrets.K8S_KOBECONFIG \}\} <br>
- Action para deploy do kubernets <br>
      - uses: Azure/k8s-deploy@v5 <br>
        with: <br>
            manifests: | <br>
              k8s/deployment.yaml <br>
            images: | <br>
              muriloscastro/fake-shop-desafio:v\$\{\{ github.run_number }\}\ <br>

- Observações:
  - Para não ocorrer execução em paralelo dos processo de CI/CD foi adicionado o needs em CD apontando para o  job CI.
  - Em ambos o codigos foram utilizadas variaveis para acessar as configuações de login do dockerhub e também arquivo so arquivo kube config.
  - Para a configuração e deploy da aplcação foi utilizado o modelo da azure que é compative com outras plataformas de Cloud.

# Execução da Actions 
Quando realizar o commit do código acima, é possível acompanhar a execução do pipeline. 
1) Clicar na aba Actions, selecionar a workflow **CI-CD** e ir no canto inferior direito na aba workflow run. 
2) Clicar na pipeline e acompanhar os passos e etapas da automação.

# Configuração de secrets 
Para cadastrar as secrets é necessário acessar o menu Settings do projeto
1) Clicar na aba Settings no menu superior do lado direito, proximo a Security e Insighs, no menu superior.

2) No menu lateral esquerdo na inferior, clicar no menu Secrets and variables, Selecionar Actions. Selecionara a aba secrets - New Repository secret

    2.1) Adicionar as credencias de username e token do docker hub <br>
   - DOCKERHUB_USERNAME - usuario do DockerHub <br>
   - DOCKERHUB_TOKEN - senha do DockerHub <br>

    2.2) Adicionar o conteudo do kube config da DigiralOcean <br>
   - K8S_KOBECONFIG

# Testes do Pipeline
Durante a criação do Pipeline, foram executadas diversas alterações e commits no código para certificar que o processo de CI /CD estava funcionando.

Para validar os testes, após as etapas de configuração no arquivo main.yaml, é necessário realizar  qualquer alteração no cõdigo, no nosso exemplo, foram alteradas a **versão "V2" para "V3"** nas **linhas 7 e 22** do arquivo ***src/templates/shared/nav-bar.html***, foi realizado o commit e o processo de automação inicou automaticamente e a versão do site foi atualizada de forma transparente.

Foi realizando um segundo teste alterando a **versão "V3" para "v4"** e versão do site foi atualizada em tempo real e de forma transparente. 

Em  ambos os testes, o Pipeline conectonou no docker hub, executou o processo de deployment da aplicação e enviou as imagens para o dockerhub com a tag da nova versão.

# Refências Action
Para obter e utilizar os códigos de automação, é necessário enconrar as Actions em algum lugar, na edição e setup do workflow, tem um campo de pesquisa "marktetplace" do lado esquerdo, mas também é possível encontrar no marketplace do GitHub.

- Marketplace <br>
https://github.com/marketplace

- Action Checkout <br>
https://github.com/marketplace/actions/checkout

- Action docker login <br>
https://github.com/marketplace/actions/docker-login

- Action de build <br> 
https://github.com/marketplace/actions/build-and-push-docker-images	

- Action de configuração do kubeconfig <br>
https://github.com/marketplace/actions/kubernetes-set-context 

- Action de deploy <br>
https://github.com/marketplace/actions/deploy-to-kubernetes-cluster

# Refência Estudo DevOps 
Aulas de qualidade e com excelente conteudo prático, profissionale e acessível
-  **hastag rumoaelite** <br>
https://www.youtube.com/@fabricioveronez