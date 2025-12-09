# Nomeados ao Oscar

Contém a base de indicados ao Oscar em formato SQL para treinar comandos CRUD. 

Abaixo, algumas atividades para trabalharmos.

--- 

* Atualize os registros da tabela com os dados do Oscar 2025

Atualizado no GitHub

---

* Qual o **total** de registros na tabela indicados?

R: 49 vezes

Q: 
```js
db.filmes.countDocument({ano_cerimonia:2025})
```

---

* Qual o número de indicações por categoria agrupadas por categoria?

R: 2 vezes

Q:
```js
db.indicados_ao_oscar.aggregate([
  {
    $group: {
      _id: "$categoria",  
      total_indicacoes: { $sum: 1 }  
    }
  },
  {
    $sort: { total_indicacoes: -1 } 
  }
]);
```

---

* Quantas vezes Natalie Portman foi indicada ao Oscar?

R: 3 vezes

Q:
```js
 db.filmes.countDocuments({nome_do_indicado: "Natalie Portman"})
```

---

* Quantos Oscars Natalie Portman ganhou?

R: 1 vez

Q:
```js
db.filmes.countDocumens({nome_do_indicado: "Natalie Portman", vencedo:"true"})
```

---

* Quantas vezes Viola Davis foi indicada ao Oscar?

R: 4 vezes

Q:
```js
db.filmes.countDocuments({nome_do_indicado: "Viola Davis"})
```

---

* Quantos Oscars Viola Davis ganhou?

R: 1 vez

Q:
```js
db.filmes.countDocuments({nome_do_indicado: "Viola Davis", vencedor: "true"})
```

---

* Amy Adams já ganhou algum Oscar?

R: Nunca ganhou Oscar, mas já foi indicada 6 vezes.

Q:
```js
db.filmes.countDocuments({nome_do_indicado:"Amy Adams", vencedor:"true"})
```

---

* Quais os atores/atrizes que foram indicados mais de uma vez?

R: Metro-Goldwyn-Mayer, Walt Disney, Producer, John Williams,Warner Bros, France.

_id: 'Metro-Goldwyn-Mayer',
  total_indicacoes: 64

_id: 'Walt Disney, Producer',
  total_indicacoes: 59

_id: 'John Williams',
  total_indicacoes: 46

_id: 'Warner Bros.',
  total_indicacoes: 44

_id: 'France',
  total_indicacoes: 37

Q:
```js
db.filmes.aggregate([
  { $group: { _id: "$nome_do_indicado", total_indicacoes: { $sum: 1 } } },
  { $match: { total_indicacoes: { $gt: 1 } } },
  { $sort: { total_indicacoes: -1 } },
  { $limit: 5 }
])```


---

* A série de filmes Toy Story ganhou Oscars em quais anos?

R: Nenhuma vez

Q:
```js
db.filmes.countDocuments({nome_do_indicado:"Toy Story", vencedor:"true"})
```
---

* A partir de que ano que a categoria "Actress" deixa de existir? 

R: 1976

{
  _id: ObjectId('68f4fcb4920a6ec65f18e83d'),
  id_registro: 5313,
  ano_filmagem: 1975,
  ano_cerimonia: 1976,
  cerimonia: 48,
  categoria: 'ACTRESS',
  nome_do_indicado: 'Carol Kane',
  nome_do_filme: 'Hester Street',
  vencedor: 'false'
}

Q:
```js
db.filmes.find({categoria:"ACTRESS"}).sort({"id_registro: -1"}).limit(1)
```
---

* Quem ganhou o primeiro Oscar para Melhor Atriz? Em que ano?

R: A ganhadora do primeiro Oscar foi a Louise Dresser em 1928.

Q:
```js
db.filmes.find({categoria:"ACTRESS"}).sort({id_registro: 1}).limit(1)
```

---

* Na campo "Vencedor", altere todos os valores com "true" para 1 e todos os valores "false" para 0.

R,Q: 
```js
db.filmes.updateMany({"vencedor": "true"}, {$set: {"vencedor": 1}})

db.filmes.updateMany({"vencedor": "false"}, {$set: {"vencedor": 0}})
```
---

* Em qual edição do Oscar "Crash" concorreu ao Oscar?

R: 2006

Q:
```js
db.filmes.find(
  { nome_do_filme: "Crash" },
  { _id: 0, nome_do_filme: 1, ano_cerimonia: 1, cerimonia: 1, categoria: 1, vencedor: 1 }
)
```
---

* O filme Central do Brasil aparece no Oscar?

R: Não

Q:
```js
db.filmes.find({"nome_do_filme": /central do brasil/}).count()
```
---

* Inclua no banco 3 filmes que nunca foram nem nomeados ao Oscar, mas que merecem ser. 

---

* Denzel Washington já ganhou algum Oscar?

R: Sim, ganhou 2 vezes

Q:
```js
db.filmes.find({"nome_do_indicado": "Denzel Washington", "vencedor": 1}).count()
```
---

* Quais os filmes que ganharam o Oscar de Melhor Filme?

R: foram registrado 63 ganhadores

Q:
{nome_do_filme: "O Senhor dos Anéis: O Retorno do Rei"},
{nome_do_filme: "Titanic"},
{nome_do_filme: "Parasita"}
...

---

* Sidney Poitier foi o primeiro ator negro a ser indicado ao Oscar. Em que ano ele foi indicado? Por qual filme?
R: Ele foi indicado no ano de 1959 pelo filme, The Defiant Ones.

Q:
```js
db.filmes.find({nome_do_indicado: "Sidney Poitier"})

{ _id: ObjectId('68e92d91c7853af6fd84cd3e'),
  id_registro: 3366,
  ano_filmagem: 1958,
  ano_cerimonia: 1959,
  cerimonia: 31,
  categoria: 'ACTOR',
  nome_do_indicado: 'Sidney Poitier',
  nome_do_filme: 'The Defiant Ones',
  vencedor: 0
}

```
---

* Quais os filmes que ganharam o Oscar de Melhor Filme e Melhor Diretor na mesma cerimonia?

R: O filme Anora ganhou os prêmios de Melhor Filme e Melhor Diretor na mesma cerimônia do Oscar

Q:
```js
db.filmes.aggregate([
  {
    $match: {
      vencedor: 1,
      categoria: { $in: ["BEST PICTURE", "DIRECTING"] }
    }
  },

  {
    $group: {
      _id: { ano: "$ano_cerimonia", filme: "$nome_do_filme" },
      categorias_ganhas: { $addToSet: "$categoria" }
    }
  },
  {
    $match: {
      categorias_ganhas: { $all: ["BEST PICTURE", "DIRECTING"] }
    }
  },
  {
    $project: {
      _id: 0,
      ano_cerimonia: "$_id.ano",
      nome_do_filme: "$_id.filme",
      categorias_ganhas: 1
    }
  },
  { $sort: { ano_cerimonia: 1 } }
])
```
---

* Denzel Washington e Jamie Foxx já concorreram ao Oscar no mesmo ano?

R:Sim, Denzel Washington e Jamie Foxx já concorreram ao Oscar no mesmo ano, foi em 2005.

Q:
```js
db.filmes.aggregate([
  {
    $match: {
      nome_do_indicado: { $in: ["Denzel Washington", "Jamie Foxx"] }
    }
  },

  {
    $group: {
      _id: "$ano_cerimonia",
      indicados: { $addToSet: "$nome_do_indicado" }
    }
  },

  {
    $match: {
      indicados: { $all: ["Denzel Washington", "Jamie Foxx"] }
    }
  }
])
```