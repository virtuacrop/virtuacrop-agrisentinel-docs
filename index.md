# Documentação da API VirtuaCrop-AgriSentinel

A **API VirtuaCrop-AgriSentinel** fornece ferramentas de análise geoespacial com base em imagens de satélite para:

- Cálculo de séries temporais NDVI (mínimo, mediana, máximo)
- Geração de zonas de produtividade (com base em percentis)
- Processamento de super-resolução (assíncrono)
- Gestão de áreas de utilizador com histórico NDVI
- Armazenamento em nuvem com URLs de download temporárias

**URL Base**  
https://virtuacrop-agrisentinel-793092962822.europe-southwest1.run.app

## 📍 Lista de Endpoints

- [`/user_areas`](#1-user_areas-post) — Guardar área do utilizador e iniciar NDVI
- [`/user_areas/<user_id>`](#2-user_areasuser_id-post) — Listar áreas guardadas
- [`/user_areas/dates`](#3-user_areasdates-post) — Datas disponíveis por área
- [`/productivity_areas`](#4-productivity-post) — Zonas de produtividade NDVI
- [`/super-resolution`](#5-super-resolution-post) — Pedido de super-resolução (assíncrono)

---

### 1. /user_areas

- **URL**: `/user_areas`  
- **Método**: `POST`

#### Descrição

Guarda um polígono associado a um user_id e inicia o cálculo de NDVI.

#### Corpo do Pedido

O pedido deve ser um payload JSON com os seguintes campos:

- `polygon` (obrigatório): Um objeto GeoJSON que define a área de interesse.  
- `user_id` (obrigatório): Identificador único do utilizador.  

<summary>📌 Exemplo de Polígono</summary>

```json
{
  "type": "Polygon",
  "coordinates": [
    [
      [-8.553043599579647, 39.362031162768943],
      [-8.553213381033897, 39.363064707371684],
      [-8.553043599579647, 39.362031162768943]
    ]
  ],
  "user_id": "tmorais",
}
```


### 2. /user_areas/<user_id> POST

- **URL**: `/user_areas`  
- **Método**: `POST`
  
#### Descrição

Lista todas as áreas guardadas para um determinado utilizador, com os valores NDVI por data (sem a geometria original).

#### Corpo do Pedido

Não necessita de corpo.


#### Exemplo de resposta:

```json
{
  "user_id": "utilizador123",
  "areas": [
    {
      "area_id": "area987",
      "saved_at": "2025-07-14T12:00:00Z",
      "ndvi": {
        "records": [
          {
            "DATA": "2025-06-15",
            "MIN": 0.1,
            "MEDIAN": 0.3,
            "MAX": 0.6,
            "url": "https://..."
          }
        ]
      }
    }
  ]
}
```

### 3. /user_areas/dates

#### Descrição

Obtém as datas disponíveis com valores NDVI para múltiplas áreas de um utilizador.

#### Corpo do Pedido

O pedido deve ser um payload JSON com os seguintes campos:

- `user_id` (obrigatório): Identificador único do utilizador.  
- `area_ids` (obrigatório): Identificador único das parcela e interesse.  

#### Exemplo de resposta:

```json
{
  "area1": ["2025-06-01", "2025-06-15"],
  "area2": ["2025-06-03"]
}
```

### 4. /productivity_areas

TBD

### 5. /super-resolution

TBD
