Neste desafio vamos precisar instalar o CLI da AWS que é a linha de comando para manipular o ambiente da ASW. 
## Passo 1: Criar uma tabela através dos seguintes comandos
aws dynamodb create-table \
    --table-name Musicas \
    --attribute-definitions \
        AttributeName=Artist,AttributeType=S \
        AttributeName=SongTitle,AttributeType=S \
    --key-schema \
        AttributeName=Artist,KeyType=HASH \
        AttributeName=SongTitle,KeyType=RANGE \
    --provisioned-throughput \
        ReadCapacityUnits=5,WriteCapacityUnits=5 \
    --table-class STANDARD
    
    # Como podemos conferir, foi criada uma tabela de Musicas, com campos para artista e nome da música, no formato String
    
## Passo 2: Inserir uma informação na tabela, através do arquivo do tipo json chamado itemmusic
aws dynamodb put-item \
    --table-name Music \
    --item file://itemmusic.json \
    
    # Obs: Podemos inserir vários items aoo mesmo tempo também, com o arquivo batchmusic.json
    aws dynamodb batch-write-item \
    --request-items file://batchmusic.json
    
    
    ## Podemos criar um index que permite procurar através do nome do álbum por exemplo:
    aws dynamodb update-table \
    --table-name Music \
    --attribute-definitions AttributeName=AlbumTitle,AttributeType=S \
    --global-secondary-index-updates \
        "[{\"Create\":{\"IndexName\": \"AlbumTitle-index\",\"KeySchema\":[{\"AttributeName\":\"AlbumTitle\",\"KeyType\":\"HASH\"}], \
        \"ProvisionedThroughput\": {\"ReadCapacityUnits\": 10, \"WriteCapacityUnits\": 5      },\"Projection\":{\"ProjectionType\":\"ALL\"}}}]"
        
## Passo 3: Realizar buscas através dos dados inseridos na tabela
# Procurar por artista:
aws dynamodb query \
    --table-name Music \
    --key-condition-expression "Artist = :artist" \
    --expression-attribute-values  '{":artist":{"S":"Iron Maiden"}}'
    
# Procurar por index secundário no titulo da musica e ano
aws dynamodb query \
    --table-name Music \
    --index-name SongTitleYear-index \
    --key-condition-expression "SongTitle = :v_song and SongYear = :v_year" \
    --expression-attribute-values  '{":v_song":{"S":"Wasting Love"},":v_year":{"S":"1992"} }'
