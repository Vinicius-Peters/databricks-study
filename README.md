# Consumir dados do Data Lake através do Databricks
Guia para consumir dados do Data lake storage através o Databricks.

## Requisitos mínimos
- Resource group devidamente criado
- Storage account que você ira consumir os dados
- Databricks criado e vinculado a sua conta
  
## Passo a passo do guia
- Criação do App Registration
  - Parâmetros e onde encontrá-los
- Atribuir permissão ao App Registration dentro do Storage Account
- Import dos dados para o notebook no Databricks

## Criação do App Registration
### Para quê você precisa de um App Registration?
O App Registration irá fazer essa "ponte" entre o data lake e o databricks, servindo de autenticação entre os dois serviços.

### Como criar?
Primeiramente digite App Registration na barra de pesquisa do Azure e selecione-o.

Clique em + New Registration
<br>
<img alt="image" src="https://github.com/Vinicius-Peters/databricks-study/blob/main/Images/1.png" style="width: 700px; height: 300px;">

De um nome com as iniciais "ap" ou "app" para manter as boas práticas: 
<br>
<img alt="image" src="https://github.com/Vinicius-Peters/databricks-study/blob/main/Images/2.png" style="width: 700px; height: 300px;">

Após isso, clique em Register e finalize a criação sem precisar alterar nenhum parâmetro.

Depois de criado, você vera essa tela a seguir.
<br>
<img alt="image" src="https://github.com/Vinicius-Peters/databricks-study/blob/main/Images/3.png" style="width: 700px; height: 300px;">

Em seguida, salve esses códigos (Client ID e Tenant ID) em um bloco de notas, pois você precisára deles mais tarde.

Agora no menu lateral esquerdo, vá em *Certificates e Secrets*, pois você precisará criar uma chave para seu registro.

Clique em *New Client Secret* 
<br>
<img alt="image" src="https://github.com/Vinicius-Peters/databricks-study/blob/main/Images/4.png" style="width: 700px; height: 300px;">

De um nome para sua Key e clique em *ok*
<br>
<img alt="image" src="https://github.com/Vinicius-Peters/databricks-study/blob/main/Images/5.png" style="width: 700px; height: 180px;">

Você vera sua Key criada logo abaixo. Salve o campo **Value** em um bloco de notas pois você ira precisar mais tarde. 
<br>
<img alt="image" src="https://github.com/Vinicius-Peters/databricks-study/blob/main/Images/6.png" style="width: 700px; height: 300px;">

Pronto, até aqui você devera estar com os 3 parâmetros necessários: **Client ID**, **Tenant ID** e o **Value ID** da sua chave.

## Atribuir permissão ao App Registration dentro do Storage Account

Após você ter criado uma identidade para uma aplicação (App Registration), você precisará atribuir permissões necessárias dentro do Storage Account.

Para isso, abra seu Storage account o qual você deseja acessar via Databricks, e vá na opção de **Containers**
<br>
<img alt="image" src="https://github.com/Vinicius-Peters/databricks-study/blob/main/Images/7.png" style="width: 700px; height: 400px;">

No meu caso, quero acessar a camada "Bronze" que está dentro desse container via Databricks

Clique na opção *Acess Control (IAM)* e logo em seguida em *+ADD* onde você ira atribuir as permissões ao AppRegistration que você criou anteriormente. 
<br>
<img alt="image" src="https://github.com/Vinicius-Peters/databricks-study/blob/main/Images/8.png" style="width: 700px; height: 300px;">

Você precisará atribuir essas permissões: *Storage Blob Contributor e Storage Blob Reader* <br>
<img alt="image" src="https://github.com/Vinicius-Peters/databricks-study/blob/main/Images/9.png" style="width: 700px; height: 300px;">

Clique em + Select Members e encontre seu AppRegistration que você criou no passo anterior.
Exemplo:<br>
<img alt="image" src="https://github.com/Vinicius-Peters/databricks-study/blob/main/Images/10.png" style="width: 700px; height: 300px;"><br>

Após realizado esse passo para *Storage Blob Contributor*, repita o passo para *Storage Blob Reader*

Finalizado essa etapa, você precisara voltar para o container denro do seu Storage account como mostra a imagem abaixo:<br>

<img alt="image" src="https://github.com/Vinicius-Peters/databricks-study/blob/main/Images/11.png" style="width: 700px; height: 300px;">

Após selecionar a camada desejada, vá na opção lateral *Manage ACL*<br>

<img alt="image" src="https://github.com/Vinicius-Peters/databricks-study/blob/main/Images/12.png" style="width: 700px; height: 300px;">

Atribuia a opção de Reader e Write para o *Security Principal* e logo em seguida clique em *+ADD Principal*<br>

<img alt="image" src="https://github.com/Vinicius-Peters/databricks-study/blob/main/Images/13.png" style="width: 700px; height: 300px;">

Novamente, selecione seu AppRegistration e clique em *Select*
<br>
<img alt="image" src="https://github.com/Vinicius-Peters/databricks-study/blob/main/Images/14.png" style="width: 700px; height: 300px;">

Atribuia a opção de Reader e Write para o *Security Principal* para seu Registro.

Pronto, todas as permissões necessárias foram atribuidas!

## Import dos dados para o notebook no Databricks

Em seu workspace, crie um notebook e cole os seguintes parâmetros na celula:

**%python
configs = {"fs.azure.account.auth.type":"OAuth",<br>
       "fs.azure.account.oauth.provider.type": "org.apache.hadoop.fs.azurebfs.oauth2.ClientCredsTokenProvider",<br>
       "fs.azure.account.oauth2.client.id": "(COLE AQUI SEU CLIENT_ID)",<br>
       "fs.azure.account.oauth2.client.secret": "(COLE AQUI SEU VALUE_ID)",<br>
       "fs.azure.account.oauth2.client.endpoint": "https://login.microsoftonline.com/(COLE AQUI SEU TENANT_ID)/oauth2/token",<br>
       "fs.azure.createRemoteFileSystemDuringInitialization": "true"}<br>**

**dbutils.fs.mount(<br>
source = "abfss://(NOME_DO_SEU_CONTAINER>@(NOME_DO_SEU_STORAGE_ACCOUNT).dfs.core.windows.net/",<br>
mount_point = "/mnt/(Container)/(Folder)/",<br>
extra_configs = configs)<br>**
<br>
Se tudo ocorrer bem, ao executar esse trecho corretamente você vera uma mensagem de sucesso no console.log.

Na célula seguinte você precisará montar a unidade para o diretorio o qual você quer acessar:<br>

**dbutils.fs.ls("/mnt/(Seu Container)/(Seu diretorio de pastas)/")**<br>

Execute a celula para ver se o acesso deu certo ao diretorio e se seus arquivos estão sendo listados corretamente.<br>

Por fim, use esse código para acessar seu container e atribui-lo a uma variavel e em seguida de um display:<br>
(Vou utilizar como exemplo esse trecho)<br>

**DF = spark.read.format("csv").options(header="true", inferschema="true").load("/mnt/bronze/STUDY/job_sample.csv")<br>
DF.display()<br>**

Dessa forma, você vera seus dados listados!






