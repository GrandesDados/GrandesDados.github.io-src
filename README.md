Site Grandes Dados
==================

Blog de Big Data.

Configuração
------------

Instalar o Hugo:

    http://gohugo.io/

Clonar o projeto:

    git clone git@github.com:GrandesDados/GrandesDados.github.io-src.git grandesdados.com
    cd grandesdados.com
    git submodule update --init

Deploy:

    ./deploy.sh [mensagem do commit entre aspas simples]

Exemplo:

    hugo new post/<NOME_DO_ARQUIVO>.md
    hugo server --buildDrafts --watch
    (edita o conteúdo em content/post/<NOME_DO_ARQUIVO>.md)
    (visualiza o resultado em  http://127.0.0.1:1313/)
    (remove a configuração de draft)
    ./deploy.sh 'Novo artigo sobre ...'

Autores
-------

Criar arquivo:

    data/authors/<ID>.toml

Com os atributos:

    name = "<NOME>"
    bio = "<APRESENTAÇÃO>"
    location = "<CIDADE_ESTADO>"
    website = "<SITE_PESSOAL>"

Nos artigos:

    +++
    author = "<ID>"
    (...)
    +++
    
    <CONTEÚDO>

Exemplo:

    data/authors/cirocavani.toml
    
    name = "Ciro Cavani"
    bio = "Engenheiro de Computação."
    location = "Rio de Janeiro, RJ"
    website = "https://cirocavani.wordpress.com"

