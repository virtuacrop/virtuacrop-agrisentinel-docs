# Documentacao API VirtuaCrop-AgriSentinel (v4)

URL base (producao):
`https://virtuacrop-agrisentinel-793092962822.europe-southwest1.run.app`

Esta versao suporta 4 indices:
- `ndvi`
- `evi`
- `ndmi`
- `cired_edge` (alias aceito em rotas: `cired-edge`)

## 1) Criar/guardar areas e iniciar processamento

### `POST /user_areas`
Guarda poligonos para um `user_id` e inicia processamento assincrono para:
- NDVI
- EVI
- NDMI
- CIred-edge

Corpo (exemplo):
```json
{
  "user_id": "utilizador123",
  "polygon": {
    "type": "FeatureCollection",
    "features": [
      {
        "type": "Feature",
        "properties": { "uid": "62543", "crop": "wheat" },
        "geometry": { "type": "Polygon", "coordinates": [[[-8.55,39.36],[-8.54,39.36],[-8.54,39.37],[-8.55,39.37],[-8.55,39.36]]] }
      }
    ]
  }
}
```

Resposta (exemplo):
```json
{
  "detail": "Polygon(s) saved; NDVI/EVI/NDMI/CIred-edge processing started asynchronously.",
  "user_id": "utilizador123",
  "added_ids": ["62543"],
  "skipped_ids": []
}
```

## 2) Processamento assincrono por indice

### NDVI
- `POST /process_ndvi_async`

### EVI
- `POST /process_evi_async`

### NDMI
- `POST /process_ndmi_async`

### CIred-edge
- `POST /process_cired_edge_async`

Body esperado:
```json
{
  "user_id": "utilizador123",
  "polygon": { "type": "FeatureCollection", "features": [] }
}
```

## 3) Listar areas de um utilizador

### `GET /user_areas/<user_id>`
Retorna lista de `area_id` guardados no no `polygons`.

## 4) Consultar datas disponiveis

### NDVI (legacy, mantido)
- `GET /user_areas/dates/<user_id>`
- `GET /user_areas/dates/<user_id>/<area_id>`

### Qualquer indice
- `GET /user_areas/<index_name>/dates/<user_id>`
- `GET /user_areas/<index_name>/dates/<user_id>/<area_id>`

`index_name` aceites: `ndvi`, `evi`, `ndmi`, `cired_edge`, `cired-edge`.

Resposta sem `area_id`:
```json
{ "dates": ["2026-01-18", "2026-01-28"] }
```

Resposta com `area_id`:
```json
{ "area_id": "62543", "dates": ["2026-01-18"] }
```

## 5) Registos por poligono

### NDVI (legacy, mantido)
- `GET /user_areas/<user_id>/records_by_polygon/<area_id>`
- `GET /user_areas/<user_id>/records_by_polygon/<area_id>/<date>`

### Qualquer indice
- `GET /user_areas/<user_id>/<index_name>/records_by_polygon/<area_id>`
- `GET /user_areas/<user_id>/<index_name>/records_by_polygon/<area_id>/<date>`

Resposta (exemplo):
```json
{
  "2026-01-18": {
    "MIN": 0.112,
    "MEDIAN": 0.402,
    "MAX": 0.742,
    "total_valid_pixels": 10234,
    "percentages": {
      "1": 0.0, "2": 2.1, "3": 5.0, "4": 10.4, "5": 18.2,
      "6": 24.7, "7": 20.3, "8": 13.1, "9": 5.2, "10": 1.0
    }
  }
}
```

## 6) Consulta em lote (multiple records)

### NDVI (legacy, mantido)
- `POST /user_areas/<user_id>/multiple_records_by_polygons`

### Qualquer indice
- `POST /user_areas/<user_id>/<index_name>/multiple_records_by_polygons`

Body com intervalo global:
```json
{
  "areas": ["62543", "72295"],
  "dates": ["2026-01-01", "2026-01-31"]
}
```

Body com intervalo por area:
```json
{
  "items": [
    { "area_id": "62543", "dates": ["2026-01-01", "2026-01-15"] },
    { "area_id": "72295" }
  ]
}
```

## 7) Apagar area

### `DELETE /user_areas/<user_id>/polygons/<area_id>`
Remove a area de `polygons` e remove os respetivos registos em todos os indices:
- `ndvi`
- `evi`
- `ndmi`
- `cired_edge`

Se nao restarem poligonos, o no agregado de indices e limpo.
