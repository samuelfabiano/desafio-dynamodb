# Desafio DynamoDB
DynamoDB é um serviço de banco de dados NoSQL (Not Only SQL) gerenciado pela AWS.

### DB relacional x Não relacional
Um banco de dados relacional é baseado no modelo relacional de dados para represtar os dados em tabelas e armazenam e fornecem acesso a pontos de dados relacionados entre si, tais como Microsoft SQL Server, PostgreSQL e Oracle DB.
Já um banco de dados não relacional é qualquer banco de dados que não segue o modelo relacional, e também conhecida como banco de dados NoSQL, onde os dados são armazenados no modelo par chave/valor. Por exemplo, MongoDB, DynamoDB e Redis.

![image](https://user-images.githubusercontent.com/89883269/200223203-c1b49046-ed2d-4860-acdd-80b6cf2b5dcb.png)

![image](https://user-images.githubusercontent.com/89883269/200223239-6277a203-25bc-45f6-94bc-7ee5f0071204.png)

Neste desafio foi utilizado o Amazon DynamoDB, que fornece um desempenho rápido e previsível com escabilidade integrada.

"Falar é fácil, me mostre o código!" TORVALDS, Linus.

* Criar uma tabela

```bash
aws dynamodb create-table \
    --table-name Music \
    --attribute-definitions \
        AttributeName=Artist,AttributeType=S \
        AttributeName=SongTitle,AttributeType=S \
    --key-schema \
        AttributeName=Artist,KeyType=HASH \
        AttributeName=SongTitle,KeyType=RANGE \
    --provisioned-throughput \
        ReadCapacityUnits=10,WriteCapacityUnits=5
```

Resultado:

![image](https://user-images.githubusercontent.com/89883269/200224198-f202224b-95f6-4fed-ad9d-5eb739f25392.png)


* Inserir um item

```bash
aws dynamodb put-item \
    --table-name Music \
    --item file://itemmusic.json \
```

* Inserir múltiplos itens

```bash
aws dynamodb batch-write-item \
    --request-items file://batchmusic.json
```

Resultado:

![image](https://user-images.githubusercontent.com/89883269/200224275-129456dd-ce8d-4904-9796-7df6430f0073.png)


* Criar um index global secundário baeado no título do álbum

```bash
aws dynamodb update-table \
    --table-name Music \
    --attribute-definitions AttributeName=AlbumTitle,AttributeType=S \
    --global-secondary-index-updates \
        "[{\"Create\":{\"IndexName\": \"AlbumTitle-index\",\"KeySchema\":[{\"AttributeName\":\"AlbumTitle\",\"KeyType\":\"HASH\"}], \
        \"ProvisionedThroughput\": {\"ReadCapacityUnits\": 10, \"WriteCapacityUnits\": 5      },\"Projection\":{\"ProjectionType\":\"ALL\"}}}]"
```

* Criar um index global secundário baseado no nome do artista e no título do álbum

```bash
aws dynamodb update-table \
    --table-name Music \
    --attribute-definitions\
        AttributeName=Artist,AttributeType=S \
        AttributeName=AlbumTitle,AttributeType=S \
    --global-secondary-index-updates \
        "[{\"Create\":{\"IndexName\": \"ArtistAlbumTitle-index\",\"KeySchema\":[{\"AttributeName\":\"Artist\",\"KeyType\":\"HASH\"}, {\"AttributeName\":\"AlbumTitle\",\"KeyType\":\"RANGE\"}], \
        \"ProvisionedThroughput\": {\"ReadCapacityUnits\": 10, \"WriteCapacityUnits\": 5      },\"Projection\":{\"ProjectionType\":\"ALL\"}}}]"
```

* Criar um index global secundário baseado no título da música e no ano

```bash
aws dynamodb update-table \
    --table-name Music \
    --attribute-definitions\
        AttributeName=SongTitle,AttributeType=S \
        AttributeName=SongYear,AttributeType=S \
    --global-secondary-index-updates \
        "[{\"Create\":{\"IndexName\": \"SongTitleYear-index\",\"KeySchema\":[{\"AttributeName\":\"SongTitle\",\"KeyType\":\"HASH\"}, {\"AttributeName\":\"SongYear\",\"KeyType\":\"RANGE\"}], \
        \"ProvisionedThroughput\": {\"ReadCapacityUnits\": 10, \"WriteCapacityUnits\": 5      },\"Projection\":{\"ProjectionType\":\"ALL\"}}}]"
```

Resulatdo:

![image](https://user-images.githubusercontent.com/89883269/200224493-9898d318-3b1f-4af0-b732-16e5c05bde2d.png)

* Pesquisar item por artista

```bash
aws dynamodb query \
    --table-name Music \
    --key-condition-expression "Artist = :artist" \
    --expression-attribute-values  '{":artist":{"S":"Iron Maiden"}}'
```

Resutlado:

![image](https://user-images.githubusercontent.com/89883269/200225334-d0053aca-f08e-459b-8e4b-a85ee4322987.png)


* Pesquisa pelo index secundário baseado no título do álbum

```bash
aws dynamodb query \
    --table-name Music \
    --index-name AlbumTitle-index \
    --key-condition-expression "AlbumTitle = :name" \
    --expression-attribute-values  '{":name":{"S":"Fear of the Dark"}}'
```

Resultado:

![image](https://user-images.githubusercontent.com/89883269/200225572-099a27f6-e77c-486d-9bd0-0f7fd732baa2.png)


* Pesquisa pelo index secundário baseado no nome do artista e no título do álbum

```bash
aws dynamodb query \
    --table-name Music \
    --index-name ArtistAlbumTitle-index \
    --key-condition-expression "Artist = :v_artist and AlbumTitle = :v_title" \
    --expression-attribute-values  '{":v_artist":{"S":"Iron Maiden"},":v_title":{"S":"Fear of the Dark"} }'
```

Resultado:

![image](https://user-images.githubusercontent.com/89883269/200225700-e8d45843-5e57-4846-853d-8ef1750078a8.png)


* Pesquisa pelo index secundário baseado no título da música e no ano

```bash
aws dynamodb query \
    --table-name Music \
    --index-name SongTitleYear-index \
    --key-condition-expression "SongTitle = :v_song and SongYear = :v_year" \
    --expression-attribute-values  '{":v_song":{"S":"Wasting Love"},":v_year":{"S":"1992"} }'
```

Resultado: 

![image](https://user-images.githubusercontent.com/89883269/200225807-1c848d2d-f0e1-4ca1-b107-2791ce82a312.png)

