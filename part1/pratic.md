GET _cluster/health
GET _nodes/stats

PUT favorite_candy

POST favorite_candy/_doc
{
  "first_name": "Lisa",
  "candy": "Sour Skittles"
}

PUT favorite_candy/_doc/1
{
  "first_name": "John",
  "candy": "Starburst"
}
// ele atualiza/sobrescreve o documento, podemos identificar atraves do campo "_version": 2 no retorno do GET
PUT favorite_candy/_doc/1
{
  "first_name": "Sally",
  "candy": "Snickers"
}

PUT favorite_candy/_doc/2
{
  "first_name": "Rachel",
  "candy": "Rolos"
}

PUT favorite_candy/_doc/3
{
  "first_name": "Tom",
  "candy": "Sweet Tarts"
}

GET favorite_candy/_doc/1

// retorna erro 409 conflict caso ja exista o id
PUT favorite_candy/_create/1
{
  "first_name": "Finn",
  "candy": "Jolly Ranchers"
}

// para atualizar use esse 
POST favorite_candy/_update/1
{
  "doc": {
    "candy": "M&M's"
  }
}

DELETE favorite_candy/_doc/2


GET favorite_candy/_search
{
  "query": {
    "match_all": {}
  }
}