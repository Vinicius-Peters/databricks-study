# Databricks
Guia para consumir dados do Data lake storage para o Databricks.
-- exemplo de diretorio: <img alt="image" src="">
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
![image](https://github.com/Vinicius-Peters/databricks-study/assets/49006283/313e4308-2807-4dc6-bf91-ebf647be419f)

De um nome com as iniciais "ap" para manter as boas práticas: 
<br>
![image](https://github.com/Vinicius-Peters/databricks-study/assets/49006283/21d0d0a0-0c8f-4f1c-8bf6-51810f05d089)

Após isso, clique em Register e finalize a criação sem precisar alterar nada.

Depois de criado, você vera essa tela a seguir.
<br>
![image](https://github.com/Vinicius-Peters/databricks-study/assets/49006283/164a3465-a3a6-44f2-b5d6-29dc65e23fe5)

Em seguida, salve esses códigos (Client ID e Tenant ID) em um bloco de notas, pois você precisára deles mais tarde.

Agora no menu lateral esquerdo, vá em *Certificates e Secrets*, pois você precisará criar uma chave para seu registro.

Clique em *New Client Secret* 
<br>
![image](https://github.com/Vinicius-Peters/databricks-study/assets/49006283/a0ad7cc0-5e0c-437e-b291-f7a784e9f64b)

De um nome para sua Key e clique em *ok*
<br>
![image](https://github.com/Vinicius-Peters/databricks-study/assets/49006283/1bce6c29-7110-4bae-8d1e-6c6a711894f4)

Você vera sua Key criada logo abaixo. Salve o campo *Value* em um bloco de notas poir você ira precisar mais tarde. 
<br>
![image](https://github.com/Vinicius-Peters/databricks-study/assets/49006283/bb62753a-c58f-47ba-9a79-be01cdd85178)

Pronto, até aqui você devera estar com os 3 parâmetros necessários: Client ID, Tenant ID e o Value ID da sua chave.

## Atribuir permissão ao App Registration dentro do Storage Account

Após você ter criado uma identidade para uma aplicação (App Registration), você precisará atribuir permissões necessárias dentro do Storage Account.

Para isso, abra seu Storage account o qual você deseja acessar via Databricks, e vá na opção de *Containers* 
<br>
![image](https://github.com/Vinicius-Peters/databricks-study/assets/49006283/2978899d-d169-43c8-8ff8-497e3bb76f5b)

No meu caso, quero acessar a camada "Bronze" que está dentro desse container via Databricks

Clique na opção *Acess Control (IAM)* e logo em seguida em *+ADD* onde você ira atribuir as permissões ao AppRegistration que você criou anteriormente. 
<br>
![image](https://github.com/Vinicius-Peters/databricks-study/assets/49006283/5708d9e7-e509-467b-a3e9-a4baad3daf25)

Escolha as opções: *Storage Blob Contributor e Storage Blob Reader* e clique em next
![image](https://github.com/Vinicius-Peters/databricks-study/assets/49006283/e8a3a714-cc74-4c14-911b-0c3f2afb1f7e)

Clique em + Select Members e encontre seu AppRegistration que você criou no passo anterior.
Exemplo:
![image](https://github.com/Vinicius-Peters/databricks-study/assets/49006283/cbf4d37c-8658-468d-9947-70a602225dac)

Após realizado esse passo para *Storage Blob Contributor*, repita o passo para *Storage Blob Reader*

Finalizado essa etapa, você precisara voltar para o container denro do seu Storage account como mostra a imagem abaixo:
<br>
![image](https://github.com/Vinicius-Peters/databricks-study/assets/49006283/8916fdfa-6a22-4c4a-9c49-e1b69cbd0e60)

Após selecionar a camda desejada, vá na opção lateral *Manage ACL*
<br>
]![image](https://github.com/Vinicius-Peters/databricks-study/assets/49006283/1d670f7a-b442-48ff-a576-7cf75d9fdac4)

Atribuia a opção de Reader e Write para o *Security Principal* e logo em seguida clique em *+ADD Principal*
<br>
![image](https://github.com/Vinicius-Peters/databricks-study/assets/49006283/0fa6cd9a-5b67-449e-9cdb-2fe9add56ff7)

Novamente, selecione seu AppRegistration e clique em *Select*
<br>
![image](https://github.com/Vinicius-Peters/databricks-study/assets/49006283/11f0b534-2c03-4297-9d8e-84d471a3ecfd)

Atribuia a opção de Reader e Write para o *Security Principal* para seu Registro.

Pronto, todas as permissões necessárias foram atribuidas!

## Import dos dados para o notebook no Databricks

Em seu workspace, crie um notebook e cole os seguintes parâmetros na celula:

"%python
configs = {"fs.azure.account.auth.type":"OAuth",
       "fs.azure.account.oauth.provider.type": "org.apache.hadoop.fs.azurebfs.oauth2.ClientCredsTokenProvider",
       "fs.azure.account.oauth2.client.id": "<COLE AQUI SEU CLIENT_ID>",
       "fs.azure.account.oauth2.client.secret": "<COLE AQUI SEU VALUE_ID>",
       "fs.azure.account.oauth2.client.endpoint": "https://login.microsoftonline.com/<COLE AQUI SEU TENANT_ID>/oauth2/token",
       "fs.azure.createRemoteFileSystemDuringInitialization": "true"}

dbutils.fs.mount(
source = "abfss://<NOME_DO_SEU_CONTAINER>@<NOME_DO_SEU_STORAGE_ACCOUNT>.dfs.core.windows.net/",
mount_point = "/mnt/bronze/study/",
extra_configs = configs)"

Se tudo ocorrer bem, ao executar esse trecho corretamente você vera uma mensagem de sucesso no console.log.

Na célula seguinte você precisará montar a unidade para o diretorio o qual você quer acessar:

dbutils.fs.ls("/mnt/<Seu Container>/<Seu diretorio de pastas>/")

Execute a celula para ver se o acesso deu certo ao diretorio e se seus arquivos estão sendo listados corretamente.

Por fim, use esse código para acessar seu container e atribui-lo a uma variavel e em seguida display:
(Vou utilizar como exemplo esse trecho)

DF = spark.read.format("csv").options(header="true", inferschema="true").load("/mnt/bronze/STUDY/job_sample.csv")
DF.display()

Dessa forma, você vera seus dados listados!






