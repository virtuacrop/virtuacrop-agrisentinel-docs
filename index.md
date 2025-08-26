# Documentação da API VirtuaCrop-AgriSentinel

A **API VirtuaCrop-AgriSentinel** fornece ferramentas de análise geoespacial com base em imagens de satélite para:

- Cálculo de séries temporais NDVI (mínimo, mediana, máximo)
- Geração de zonas de produtividade (com base em percentis)
- Processamento de super-resolução (assíncrono)
- Gestão de áreas de utilizador com histórico NDVI
- Armazenamento em nuvem com URLs de download temporárias

**URL Base**  
https://virtuacrop-agrisentinel-793092962822.europe-southwest1.run.app

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

#### Exemplo de Payload

```json
{
    "polygon": {
        "type": "Polygon",
        "coordinates": [[
            [-8.5530436, 39.3620312],
            [-8.5532134, 39.3630647],
            [-8.5532452, 39.3635040],
            [-8.5531646, 39.3638096],
            [-8.5526934, 39.3643275],
            [-8.5520865, 39.3646373],
            [-8.5516026, 39.3648113],
            [-8.5509914, 39.3648665],
            [-8.5499048, 39.3648113],
            [-8.5492341, 39.3646373],
            [-8.5487418, 39.3643614],
            [-8.5483895, 39.3641704],
            [-8.5481390, 39.3638351],
            [-8.5480329, 39.3635083],
            [-8.5478631, 39.3631814],
            [-8.5477740, 39.3624684],
            [-8.5479311, 39.3620863],
            [-8.5478886, 39.3617977],
            [-8.5480966, 39.3613011],
            [-8.5490728, 39.3606984],
            [-8.5499090, 39.3606474],
            [-8.5512885, 39.3605201],
            [-8.5517108, 39.3606283],
            [-8.5524112, 39.3610273],
            [-8.5528611, 39.3614157],
            [-8.5529736, 39.3615940],
            [-8.5530521, 39.3619739],
            [-8.5530436, 39.3620312]
        ]]
    },
    "user_id": "utilizador123",
}
```


### 2. /user_areas/<user_id> POST

- **URL**: `/user_areas/<user_id>`  
- **Método**: `POST`
  
#### Descrição

Lista todas as áreas guardadas para um determinado utilizador, com os valores NDVI por data (sem a geometria original).

#### Exemplo de Payload

Não necessita de corpo.

#### Exemplo de Resposta

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

#### Exemplo de Resposta

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
