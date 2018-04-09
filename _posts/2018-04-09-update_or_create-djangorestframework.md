---
layout: post
title: "update_or_create com DjangoRestFramework"
description: Vamos criar uma aplicação django de tarefas para servir de api REST
image: '/assets/img/update_or_create.gif'
category: 'djangorestframework'
---

## Fala galera, Beleza

Hoje só vim compartilhar uma situação que passei utilizando DjangoRestFramework em um projeto.

O meu cliente (frontend em VueJS :heart_eyes::heart_eyes:) envia um array de dados via post para salvar no banco. Por default, entendi que o ModelViewSet, recebe uma unica instância do objeto e faz a mágica.

Antes de continuar, vou mostrar minha estrutura:

Model
```python
class Item(models.Model):
    """
    Modelo de Fitas do Orçamento
    """
    orcamento = models.ForeignKey('fs_pedido.Orcamento', on_delete=models.CASCADE, related_name='orcamento_item')
    codigoerp = models.IntegerField()
    referencia = models.CharField(max_length=50)
    tipo = models.CharField(max_length=50)
    subtipo = models.CharField(max_length=50)
    quantidade = models.DecimalField(max_digits=14, decimal_places=4, default=0)
    preco = models.DecimalField(max_digits=14, decimal_places=4, default=0)
    un = models.CharField(max_length=10)
    medida = models.CharField(max_length=100)
    descricao = models.CharField(max_length=500)
    ativo = models.CharField(max_length=1, default='S')
    produto_grupo_id = models.IntegerField()
    produto_subgrupo_id = models.IntegerField()
    mostrou_video = models.BooleanField(default=False)

    @property
    def subtotal(self):
        return self.quantidade * self.preco

    def __str__(self):
        return self.codigo_erp
```

Serializer
```python
class ItemSerializer(serializers.ModelSerializer):
    """
    Serializer para modelo de Item
    """

    subtotal = serializers.DecimalField(max_digits=15, decimal_places=2, read_only=True)

    class Meta:
        model = Item
        fields = '__all__'
```

View
```python
class ItemViewSet(viewsets.ModelViewSet):
    """
    Viewset para modelo Item
    """

    serializer_class = ItemSerializer
    queryset = Item.objects.all()
```

Para receber o array e manter a estrutura do ModelViewSet, eu sobrescrevi o método create para passar a entender os dados, tanto array, quanto instancia única.

```python
class ItemViewSet(viewsets.ModelViewSet):
    """
    Viewset para modelo Item
    """

    serializer_class = ItemSerializer
    queryset = Item.objects.all()

    def create(self, request, *args, **kwargs):
        is_many = isinstance(request.data, list)

        if not is_many:
            return super(ItemViewSet, self).create(request, *args, **kwargs)

        serializers = self.get_serializer(data=request.data, many=True)
        serializers.is_valid(raise_exception=True)
        self.perform_create(serializers)
        headers = self.get_success_headers(serializers.data)
        return response.Response(serializers.data, status=201, headers=headers)
```

Simplificando, o create verifica se o tipo de dados do request é uma `list` e se caso não seja, ele chama o `super` para manter o comportamento padrão.

Caso seja, vou serializar os dados com o atributo `many` definido `True`. Efetuo as validações e chamo o `perform_create` que vai criar os dados e retorno isso.

Até ai, beleza. Agora, eu descobri que meu cliente, manda nesse mesmo POST, dados que eu já tenho no backend e dados que ainda não tenho.

De cara, já veio aquele metodo update_or_create dos models que me ajuda tanto. Depois veio a dúvida, como fazer ele funcionar no mesmo método (POST).

Lendo a documentação do [django_rest_framework](http://www.django-rest-framework.org/) verifiquei que quando um serializer é criado com o atributo `many=True` ele vai retornar um ListSerializer, o que não da pra ser iterado, mais abre um leque de implementações.

Eu fui pelo caminho que achei mais simples. No meu serializer do item, adicionei no `meta` um atributo chamado  `list_serializer_class` e criei essa classe

```python
class ItemListSerializer(serializers.ListSerializer):
    pass

class ItemSerializer(serializers.ModelSerializer):
    """
    Serializer para modelo de Item
    """

    subtotal = serializers.DecimalField(max_digits=15, decimal_places=2, read_only=True)

    class Meta:
        model = Item
        fields = '__all__'
        list_serializer_class = ItemListSerializer
```

Agora no meu ListSerializer, só adaptei minha necessidade no método create.
```python
class ItemListSerializer(serializers.ListSerializer):

    def create(self, validated_data):
        with transaction.atomic():
            itens = []
            for item in validated_data:
                obj, created = Item.objects.update_or_create(codigoerp=item['codigoerp'], orcamento=item['orcamento'], defaults=item)

                itens.append(obj)
            return itens
```

### Isso funcionou lindamente!

Alguém ai já passou por essa situação? Conseguiu encontrar outra solução?

Valeu!