# Elastic Search / Kibana study

## Objetivo / Motivação

Indexar uma grande massa da dados no elastic search e montar um dashboard
simples no kibana.

## Os dados

Peguei os dados do governo federal sobre os pagamentos do auxilio emergencial,
até o momento são dois csv's dos meses de abril e maio de 2020 que podem ser
baixados com o comando abaixo.

```sh
# TODO: colocar os comandos aqui
```

## Requisitos

- python
- docker

## Passo-a-passo e problemas

1 - Precisamos subir uma instância de elastic search e outra do kibana e fazer
com que uma consiga enchergar a outra. Para isso suba o docker-compose deste
projeto.

```sh
docker-compose up
```

2 - Precisamos converter os dados do formato CSV para JSON para conseguir
importá-los no elastic search.
  a - Precisamos primeiro abrir o arquivo com python para depois estruturar os 
  dados de um modo que fique mais fácil trabalhar com a linguagem de
  programação. Os arquivos foram gerados com o encoding padrão do windows
  (cp1252), que não é o padrão do python (utf-8), por isso precisamos
  informar isso no momento da abertura do arquivo.
  b - Para estruturar os dados o python tem uma lib nativa pronta chamada `csv`,
  ela estrutura os dados já em forma de dicionário que é a mesma coisa que um 
  JSON.
  c - O dado de valor do pagamento vem como texto o que é um problema para
  fazer os cálculos. Precisamo converter para número e para conseguir
  transformar este dado precisamos trocar a `vírgula` que separa os decimais
  para `ponto`.
  d - Por fim imprimimos JSON usando o método `dump` da lib `json` para que
  a saída do programa seja num formato compatível com qualquer leitor de JSON.

3 - Gerar o arquivo para ser importado no elasticsearch.
  a - o elastic search tem uma API para bulk insert (inserção em massa). Para
  usá-la precisamos gerar duas linhas para cada registro, uma com o ID e outra
  com os dados. Podemos gerar com o programa feito no primeiro passo.

```sh
./convert-csv-to-json downloaded-data.csv > es-import.json
``` 

  b - O comando acima gerará um arquivo muito grande e a funcionalidade de bulk
  insert tem uma limitação de 5M por requisição. Precisamos quebrar este arquivo
  em vários pequenos, para isso vamos usar o comando `split` do terminal. Temos
  um arquivo JSON de praticamente 2G com 10.397.530 linhas, fazendo uma regra de
  3 descobrimos que 5M é um pouco mais de 25000 linhas, para termos uma margem
  de erro vamos quebrar os arquivos a cada 25000 linhas e serpará-los em uma
  pasta.

```sh
split -l 25000 es-import.json splitted-es-import-
mkdir splitted-import
mv splitted-es-import-* splitted-import
```

4 - Para fazer a importação no elastic search vamos usar o comando `curl`, uma
vez para cada arquivo gerado.

```sh
cd splitted-import
for file in *; do; curl -H "Content-Type: application/json" -XPOST "localhost:9200/auxilio-emergencial-2020/_bulk?pretty&refresh" --data-binary "@${file}"; done
```

> Este comando demorará alguns bons minutos, talvez horas. Para importar os dois
> arquivos eu demorei quase 2h e meia só neste passo, sem contar o tempo dos
> passos 2 e 3
