# Databricks
Guia para consumir dados do Data lake storage para o Databricks.

## Requisitos mínimos
- Resource group devidamente criado
- Storage account que você ira consumir os dados
- Databricks criado e vinculado a sua conta
  
## Passo a passo do guia
- Criação do App Registration
- Atribuir permissão ao App Registration dentro do Storage Account
- Paramêtros e onde encontrá-los
- Import dos dados para o notebook

## Criação do App Registration
Para quê você precisa de um App Registration? O App Registration irá fazer essa "ponte" entre o data lake e o databricks, servindo de autenticação entre os dois serviços.

### Onde encontrar?
Primeiramente digite App Registration na barra de pesquisa do Azure e selecione-o.

Clique em + New Registration
![image](https://github.com/Vinicius-Peters/databricks-study/assets/49006283/313e4308-2807-4dc6-bf91-ebf647be419f)
