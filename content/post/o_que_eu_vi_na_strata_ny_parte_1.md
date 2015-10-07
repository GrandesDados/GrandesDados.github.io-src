+++
author = "renanoliveira"
comments = true
date = "2015-10-07T19:00:00-03:00"
draft = false
image = "/images/posts/o_que_eu_vi_na_strata_ny_parte_1/strata_ny_expo_hall.jpg"
menu = ""
share = true
slug = "o-que-eu-vi-na-strata-ny-parte-1"
tags = ["Globo.com", "BigData", "Strata", "Hardcore Data Science", "Eventos"]
title = "O que eu vi Strata + Hadoop World NY 2015 - PARTE 1"

+++

Na última semana (29-01 de Outubro) estive na [Strata](http://strataconf.com/big-data-conference-ny-2015) em Nova Iorque, o evento de Big Data mais importante do ano, pela Globo.com.


O evento foi bem interessante, foi importante a confirmação que a [arquitetura usada por nós] (http://grandesdados.com/post/bigdata-na-globocom/) do time de Big Data da Globo.com é a mesma usada nos players mais respeitados e o futuro parece ser na direção de Deep Learning, tema discutido em várias apresentações.

No primeiro dia foram os tutoriais, eu escolhi o [Hardcore Data Science](http://strataconf.com/big-data-conference-ny-2015/public/content/hardcore-data-science), na verdade era uma sala dedicada a apresentações somente do tema de Data Science que uniu profissionais do mercado (Facebook, Microsoft e Databricks) com pesquisadores das principais universidades americanas (MIT, Stanford e Berkeley), ficou claro nessa apresentação que o Spark MLlib é largamente usada, que há um tendência em cada vez usar mais algoritmos de redes neurais (Deep Learning), outro assunto que foi muito comentado foi a união de tempo real com Machine Learning (Spark Streaming e MLlib).

*Nem todas as apresentações já disponibilizaram os slides, as que estão disponíveis são acessíveis via link no título.*

## Hardcore Data Science em mais detalhes


**[GPU/CPU acceleration for matrix computations and neural networks on Spark](http://stanford.edu/~rezab/slides/stratany2015.pdf)**

Foi apresentado por [Reza Zadeh](https://twitter.com/Reza_Zadeh), que é de Stanford e da Databricks, ele trabalha nos projetos dedicados a Machine Learning e algébra linear do Spark.

Foi apresentado como o Spark pode ser usado para Deep Learning, mostrou a API de redes neurais disponível no Spark. O foco era como o Spark facilita para escalar as redes neurais, mostrando como o Spark pode ser usado para distribuir as matrizes pelo cluster. Falou que a ideia de "maquina barata" do Hadoop para o cluster precisa mudar um pouco para processar matrizes (indo até Deep Learning), precisando de mais CPU e GPU locais para aumento de performance. No final ele mostrou quais são os milestones do Spark para esse caminho, que para o 1.6 são Convolutional Neural Networks e Dropout, different layer types, e para o 1.7 Recurrent Neural Networks and LSTMs.

**[Probabilistic topic models and user behavior](http://www.cs.columbia.edu/~blei/talks/Blei_User_Behavior.pdf)**

Foi apresentado por [David Blei](http://www.cs.columbia.edu/~blei/), que é de Columbia.

Foi tratado do problema de como representar um texto por meio de palavras-chave e anotações. Foi dito que tópico é uma sequência de palavras em um determinado momento, isso diz que as palavras chave de um texto podem mudar dado o tempo, o exemplo dado foi como coisas como tecnologia viram commodites com o passar do tempo. Foi dito que [LDA](https://www.wikiwand.com/en/Latent_Dirichlet_allocation) é um ótimo algoritmo para a classificação de um documento ser automatizada. Contou o problema dos termos não terem metadados (ontologias servem para suprir esse ponto), outro ponto muito comentado foi como é importante classificar com imparcialidade dado que isso pode mudar completamente a descrição de um texto. Ele mudou a matriz de lado e mostrou como os leitores podem falar sobre o texto usando a abordagem conhecida como [Collaborative Topic Models](https://www.cs.princeton.edu/~chongw/papers/WangBlei2011.pdf). Foi uma apresentação bem densa e vale a pena a consulta sobre os detalhes apresentados.


**FBLearner Flow - Facebook machine learning platform**

Foi apresentado por [Hussein Mehana](https://www.linkedin.com/pub/hussein-mehanna/1/19b/792), que é o principal engenheiro da plataforma de Machine Learning interna do Facebook.

Foi mostrada a plataforma usada pelos cientistas de dados do Facebook tanto para fazer exploração como para rodar em produção algoritmos. Essa plataforma tem toda uma camada de Workflow junto com sampling de dados para facilitar a integração dos códigos novos bem como o tuning dos mesmos, eles chamam isso de ML-Pipeline, para um job entrar nesse workflow ele usa decoradores no código (a linguagem de exemplo foi Python), ele mostrou a interação entre dois jobs com interdependência (chamou isso de channel), sendo transparente para os usuários. Além da parte de orquestração de jobs ele mostrou uma interface gráfica que é gerada após os resultados ajudando na exploração, nesse ponto me pareceu com o [HUE](http://gethue.com/). Um ponto levantado foi a importância de ter 1 ambiente de sandbox separado da infraestrutura para o cliente final, mas que tenha a mesma carga do outro ambiente possibilitando testes sem impactar o usuário.

**KeystoneML: Building large-scale machine learning pipelines on Apache Spark**

Foi apresentado por [Ben Recht](http://www.eecs.berkeley.edu/~brecht/), que é de Berkeley.

Começou a apresentação explicando o [Keystone ML](http://keystone-ml.org/) que é um framework para criação de pipelines para Machine Learning. Pareceu muito promissor eles contribuíram fortemente com o spark.ml. Um exemplo de pipeline foi do processamento de textos com [saco de palavras](https://www.wikiwand.com/en/Bag-of-words_model), indo para [TF-IDF](https://www.wikiwand.com/en/Tf%E2%80%93idf), [LDA](https://www.wikiwand.com/en/Latent_Dirichlet_allocation) e [CRF](https://www.wikiwand.com/en/Conditional_random_field). Ele mostrou como eles tão usando essa arquitetura aplicada a Deep Learning. Esse projeto tem bind para C em vários pontos pensando em performance. Me pareceu uma biblioteca bem madura, testada e documentada. É um projeto para ficar de olho.

**Crowdsourcing your data**

Foi apresentado por [Jenn Wortman Vaughan](http://www.jennwv.com/), que é pesquisadora da Microsoft.

Nessa apresentação foi levantado o ponto de que ter muitos dados para Machine Learning é complicado, a maioria das empresas tem que validar as suas ideias com poucos usuários. O ponto é que há mecanismos como o [Amazon Mechanical Turk](https://www.mturk.com/mturk/welcome) que ajudam a coletar dados fazendo com que pessoas de verdade sejam usuários em um experimento. Algumas empresas também estão pagando a alguns usuários para preencherem pesquisas ajudando na coleta de mais informações sobre eles. Há uma grande demanda do mercado por dados, ela mostrou como empresas grandes como Facebook e Netflix já abriram parte dos seus dados anonimizados gerando uma gama de papers que podem melhorar seus próprios produtos "de graça". Ela mostrou como que a falta de conhecimento dos usuários no caso de anúncios pode gerar prejuízo para quem disponibiliza um banner, anúncios ruins podem custar mais de $1 à cada 1000 pageviews feitos.

**Minds and machines: Humans where they're best, robots for the rest**

Foi apresentado por [Adam Marcus](http://marcua.net/), que é co-fundador da Unlimited Labs.

Lançou uma plataforma Open Source chamada [ORCHESTRA](http://orchestra.unlimitedlabs.com/), essa plataforma visa a curadoria de conteúdos por especialistas, ele cria todo um workflow e permite a interação dos especialistas com o autor do trabalho, o foco é como melhorar o trabalho em equipe unindo-o com o trabalho das máquinas, como por exemplo deixando crop de imagens para o computador fazer.

**Learning with Counts: Extreme-scale featurization made easy**

Foi apresentando por [Mikhail Bilenko](http://research.microsoft.com/en-us/um/people/mbilenko/), que é pesquisador líder da divisão de Machine Learning da Microsoft (Azure).

Ele seguiu a apresentação mostrando como o estado da arte da área de Machine Learning pode sofrer com problemas de performance. Ele apresentou o [DRACULA](http://blogs.technet.com/b/machinelearning/archive/2015/02/17/big-learning-made-easy-with-counts.aspx) (Algoritmo robusto distribuído para aprendizagem baseada em contagem), que é como um objeto é referenciado olhando um histórico de contagem, por exemplo clique em matérias. O objetivo é que o [Azure ML](https://studio.azureml.net/) consiga calcular todos os dados de forma rápida com muito valor agregado, um ponto que foi destacado foi usar o máximo de dados reais para o experimento, mostrando como dados em produção podem se comportar diferente dos amostrais.

**[Sketching big data with Spark: Randomized algorithms for large-scale data analytics](http://pt.slideshare.net/databricks/sketching-big-data-with-spark-randomized-algorithms-for-largescale-data-analytics)**

Foi apresentado por [Reynold Xin](http://www.cs.berkeley.edu/~rxin/), que é co-fundador da Databricks e líder do Spark.

Ele mostrou como o caminho do Sketching é muito bom para conhecer os seus dados e fazer um processamento com resultados mais rápidos e nos ajustes dos parâmetros de tuning dos algoritmos. Falou sobre como podemos usar as técnicas de sketching no Spark (bloom filter, itens frequentes e modelo de Bernoulli, ordenação randomica) e como ele é um ótimo caminho para isso.

---

Se você gosta de Big Data e quer fazer parte de uma empresa que investe no conhecimento do profissional, como em idas a eventos internacionais e nacionais e com ótimos treinamentos internos, estamos contratando.

[http://talentos.globo.com/](http://talentos.globo.com/)
