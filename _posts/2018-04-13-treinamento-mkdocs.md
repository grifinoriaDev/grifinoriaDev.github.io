---
layout: post
title: "Treinamento de mkdocs no Grupo Scheffer"
description: Eu tentei ensinar mkdocs pro povo lá
image: '/assets/img/tudodoido.jpg'
category: 'doidos'
---

# William

*Analista* ~~de~~ **desenvolvimento**

## O que estou apredendo?

!!! warning
    Parece que o professor não sabe o que esta falando!

> Nesse treinamento estou aprendo a utilizar o python, mkdocs para gerar sites estáticos.


A linguagem utilizada é [markdown](https://daringfireball.net/projects/markdown/syntax)

## Pra que estou aprendendo?

Com markdown, fica fácil de escrever documentações.

Eu consigo anexar imagens

![gruposcheffer](https://gruposcheffer.s3.amazonaws.com/media/ckuploads/2015/05/18/unid_001.jpg)

Eu consigo fazer uma tabela

| Aluno | Função |
|---|---|
| William | Analista de Desenvolvimento |
| Marcos | Analista de RH |
| Luader | Analista da Unemat |
| Guilherme | Auxiliar do marcos no baile da acisa |
| Ariely | Pipoqueira |
| Denizon | Tirou eu do dominio |

Eu consigo fazer uma lista não ordenada

Participantes

* Marcos
* Luander
* Guilherme
* Ariely
* Denizon

Eu consigo fazer uma lista ordenada

1. Marcos
1. Luander
1. William
1. Guilherme
1. Ariely
1. Denizon

Eu consigo fazer uma lista checada

Quem fez o site

- [X] Marcos
- [X] Luander
- [x] Guilherme
- [x] Ariely
- [ ] Denizon



Eu consigo fazer um código

```python
from django.db import models


class Unidade(models.Model):
    nome = models.CharField(max_length=50)


    def __str__(self):
        return self.nome


class UnidadeConsumidora(models.Model):

    uc = models.CharField(max_length=20)
    unidade = models.ForeignKey('core.Unidade', on_delete=models.CASCADE, null=True, blank=True)


    def __str__(self):
        return self.uc
```

```javascript
import Vue from 'vue'
import Router from 'vue-router'
import Home from './views/Home.vue'

Vue.use(Router)

export default new Router({
  routes: [
    {
      path: '/',
      name: 'home',
      component: Home
    },
    {
      path: '/colheita',
      name: 'colheita',
      component: Home
    },
    {
      path: '/patio',
      name: 'patio',
      component: Home
    },
    {
      path: '/beneficiamento',
      name: 'beneficiamento',
      component: Home
    },
    {
      path: '/classificacao',
      name: 'classificacao',
      component: Home
    },
    {
      path: '/paradas',
      name: 'paradas',
      component: Home
    }
  ]
})
```