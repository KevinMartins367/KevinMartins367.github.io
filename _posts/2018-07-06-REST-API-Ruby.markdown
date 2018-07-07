---
title: "REST API no Ruby com Postgresql e AWS S3"
date: "2018-07-06"
tags: [ Ruby on Rails, Postgresql, AWS, CRUD ]
tag: "Ruby on Rails, Postgresql, AWS, CRUD"
description: "Neste post iremos aprender a implementar uma REST API no Ruby on Rails com Postgresql e consectivamente aprender a configurar ela para ser alocada na Amazon S3"
---

Neste post iremos aprender a implementar uma REST API no Ruby on Rails com Postgresql e consectivamente aprender a configurar ela para ser alocada na Amazon S3

**Índice**  

1. [Instalações necessárias](#id1)


## <a name="id1"> Instalações necessárias: </a>

No `Gemfile` vamos adicionar as seguintes linhas

1. gem ​'devise'
2. gem 'json', '~> 1.8', '>= 1.8.3'
3. gem 'active_model_serializers', '0.9.3'
4. gem 'pg_search'
