# Documentação da API VirtuaCrop-AgriSentinel (atualizada)

A **API VirtuaCrop-AgriSentinel** disponibiliza análise geoespacial a partir de Sentinel-2:
	•	Séries temporais NDVI (mín., mediana, máx.) por data
	•	Séries por polígono (ID) e série agregada (“union”)
	•	Histograma de percentis (10 bins) e número de píxeis válidos
	•	Geração opcional de GeoTIFF COG (float32) recortado e mascarado à área
	•	Gestão de áreas do utilizador (com agregação MultiPolygon)

**URL Base**  
https://virtuacrop-agrisentinel-793092962822.europe-southwest1.run.app

---
### 1 POST /user_areas

Guarda um ou mais polígonos para um user_id e inicia o processamento NDVI.

#### Corpo do pedido

Aceita GeoJSON nas formas abaixo. IDs de parcelas devem vir em properties.wisecropID (recomendado). O campo opcional properties.crop é preservado.
	•	FeatureCollection com várias Features (Polygon/MultiPolygon)
	•	Feature única (Polygon/MultiPolygon)
	•	Geometry direta (Polygon/MultiPolygon)


#### Exemplo de Payload
```json
{
    "polygon": {
        "type": "FeatureCollection",
        "features": [
            {
                "type": "Feature",
                "properties": {
                    "uid": "poly_1",
                    "crop": "trigo"
                },
                "geometry": {
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
                }
            },
            {
                "type": "Feature",
                "properties": {
                    "uid": "poly_2",
                    "crop": "milho"
                },
                "geometry": {
                    "type": "Polygon",
                    "coordinates": [[
                        [-8.551313422321872, 39.360399584256442],
                        [-8.550708273295971, 39.360530106595355],
                        [-8.549379318572427, 39.3606487632671],
                        [-8.548465662199989, 39.36041144992361],
                        [-8.547824916172567, 39.359948688903785],
                        [-8.547516408826031, 39.359035032531352],
                        [-8.547516408826031, 39.357611152470412],
                        [-8.547919841509964, 39.356946675108638],
                        [-8.548489393534341, 39.356626302094924],
                        [-8.550613347958578, 39.356317794748385],
                        [-8.551918571347773, 39.356804287102541],
                        [-8.552405063701928, 39.357480630131491],
                        [-8.552571183042371, 39.35829936116653],
                        [-8.552642377045418, 39.359331674210715],
                        [-8.552571183042371, 39.359628315890077],
                        [-8.55222707869431, 39.359948688903785],
                        [-8.552025362352342, 39.360126673911402],
                        [-8.551645661002759, 39.360328390253372],
                        [-8.551313422321872, 39.360399584256442]
                    ]]
                }
            }
        ]
    },
    "user_id": "utilizador123",
}
```
#### Resposta (exemplo) (200)

```json
{
  "detail": "Polygon(s) saved; NDVI processing started",
  "user_id": "utilizador123",
  "added_ids": ["area001", "area002"]
}
```


### 2 POST /user_areas/<user_id>

Devolve um snapshot das áreas guardadas para o utilizador e o respetivo nó NDVI associado a cada entrada top-level.

#### Corpo do pedido

Não tem.

#### Resposta (200 – exemplo)
```json
{
  "user_id": "utilizador123",
  "areas": [
    {
      "area_id": "polygons",
      "ndvi": {}
    },
    {
      "area_id": "saved_at",
      "ndvi": {}
    },
    {
      "area_id": "ndvi",
      "ndvi": {
        "status": "completed",
        "records_union": {
          "2025-09-01": {
            "MIN": 0.12,
            "MEDIAN": 0.41,
            "MAX": 0.76,
            "total_valid_pixels": 38219,
            "percentages": { "1": 0, "2": 1.2, "3": 4.8, "4": 10.5, "5": 18.7, "6": 24.3, "7": 20.1, "8": 13.4, "9": 5.6, "10": 1.4 }
          }
        },
        "records_by_polygon": {
          "area001": {
            "2025-09-01": { "MIN": 0.11, "MEDIAN": 0.40, "MAX": 0.74, "total_valid_pixels": 10234, "percentages": { "1": 0, "2": 2.1, "...": 0 } }
          },
          "area002": {
            "2025-09-01": { "MIN": 0.13, "MEDIAN": 0.43, "MAX": 0.78, "total_valid_pixels": 27985, "percentages": { "1": 0, "2": 0.7, "...": 0 } }
          }
        },
        "images": {
          "2025-09-01": {
            "gcs_url": "https://storage.googleapis.com/virtuacrop-agrisentinel/ndvi/utilizador123/2025-09-01.tif"
          }
        }
      }
    }
  ]
}
```

### 3 Datas de NDVI (GET)
O código expõe rotas GET (em vez de POST) para consultar as datas disponíveis.
Mantemos este endpoint na documentação, mas refletindo a implementação real:

#### GET /user_areas/dates/<user_id>
Devolve as datas da série agregada (records_union) do utilizador.

#### Resposta (200)
```json
{
  "dates": ["2025-08-18", "2025-08-22", "2025-08-29", "2025-09-01"]
}
```

### 4 GET /user_areas/<user_id>/records_by_polygon/<area_id>/<date>
Devolve o registo NDVI (estatísticas) de uma área específica (area_id = wisecropID) para uma data específica.

#### Parâmetros de caminho
	•	user_id — identificador do utilizador.
	•	area_id — identificador da parcela (valor de properties.wisecropID gravado no POST /user_areas).
	•	date — data YYYY-MM-DD existente em records_by_polygon[area_id].

#### Resposta (exemplo) (200)
Um objeto com uma única chave igual à date, cujo valor são as estatísticas NDVI desse dia.
```json
{
  "2025-09-01": {
    "MIN": 0.112,
    "MEDIAN": 0.402,
    "MAX": 0.742,
    "total_valid_pixels": 10234,
    "percentages": {
      "1": 0.0,
      "2": 2.1,
      "3": 5.0,
      "4": 10.4,
      "5": 18.2,
      "6": 24.7,
      "7": 20.3,
      "8": 13.1,
      "9": 5.2,
      "10": 1.0
    }
  }
}
```
Campo percentages: histograma em 10 bins (1–10) cobrindo o intervalo 0.0–1.0 de NDVI, em percentagem de píxeis válidos na área.

### 5 DELETE /user_areas/<user_id>/polygons/<area_id>
Remove uma parcela (identificada por area_id, que corresponde ao wisecropID guardado no POST /user_areas) de um utilizador.
Esta operação também atualiza (ou limpa) os dados agregados e séries NDVI associadas.

#### Parâmetros de caminho
	•	user_id — ID do utilizador.
	•	area_id — ID da parcela (valor de properties.wisecropID).

#### Resposta (exameplo) (200)
```json
{
  "detail": "Polygon 'area-001' removed for user 'utilizador123'."
}
```

### 6. /productivity_areas

TBD

### 7. /super-resolution

TBD
