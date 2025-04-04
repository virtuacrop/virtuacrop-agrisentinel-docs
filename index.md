# Documentação da API VirtuaCrop-AgriSentinel

A **API VirtuaCrop-AgriSentinel** fornece ferramentas de análise geoespacial que utilizam imagens de satélite para calcular índices de vegetação e zonas de produtividade.

**URL Base**:  
`https://virtuacrop-agrisentinel-793092962822.europe-southwest1.run.app`

---

## Endpoints

### 1. Endpoint de Cálculo NDVI

- **URL**: `/ndvi`  
- **Método**: `POST`

#### Descrição

Este endpoint calcula o **NDVI mediano (Índice de Vegetação da Diferença Normalizada)** sobre um polígono especificado utilizando imagens de satélite. Suporta três fontes de dados:

- `sentinel2`
- `landsat8`
- `sentinel2_landsat8` (conjunto de dados combinado)

A autenticação opcional do Firebase regista dados de pedidos/utilização. Se for fornecida autenticação, gera e carrega um ficheiro NDVI **NetCDF** para o Google Cloud Storage.

#### Corpo do Pedido

O pedido deve ser um payload JSON com os seguintes campos:

- `polygon` (obrigatório): Um objeto GeoJSON que define a área de interesse.  
- `start_date` (obrigatório): Formato `"YYYY-MM-DD"`  
- `end_date` (obrigatório): Formato `"YYYY-MM-DD"`  
- `max_cloud_cover` (opcional): Percentagem máxima de cobertura de nuvens (predefinição: `50.0`)  
- `collection` (opcional): `"sentinel2"`, `"landsat8"`, ou `"sentinel2_landsat8"` (predefinição: `"sentinel2"`)  
- `email` (opcional): Para autenticação Firebase  
- `password` (opcional): Para autenticação Firebase  
- `include_image` (opcional): Booleano (predefinição: `true`)  
- `include_median` (opcional): Booleano (predefinição: `true`)  

<details>
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
  ]
}
```
</details>

#### Resposta

A resposta inclui:
- `median_values`: Lista de valores NDVI por data
  - `DATA`: Data no formato "YYYY-MM-DD"
  - `VALUES`: NDVI mediano (pode ser nulo se não houver dados válidos)
- `netcdf_download_url`: URL assinada (válida por 1 hora) para descarregar o ficheiro NetCDF

Exemplo de resposta:
```json
{
  "median_values": [
    {"DATA": "2024-10-30", "VALUES": 0.1767304837703704},
    {"DATA": "2024-11-19", "VALUES": null},
    {"DATA": "2025-03-29", "VALUES": 0.460008829832077}
  ],
  "netcdf_download_url": "https://storage.googleapis.com/virtuacrop-agrisentinel/ndvi_..."
}
```

#### Exemplo de Utilização (Python)

```python
import requests

BASE_URL = "https://virtuacrop-agrisentinel-793092962822.europe-southwest1.run.app"
endpoint = f"{BASE_URL}/ndvi"

payload = {
    "polygon": {
        "type": "Polygon",
        "coordinates": [[
            [-8.5530436, 39.3620312],
            [-8.5530436, 39.3620312]
        ]]
    },
    "start_date": "2024-10-01",
    "end_date": "2025-03-31",
    "include_median": True,
    "include_image": True,
    "max_cloud_cover": 30,
    "collection": "sentinel2_landsat8"
    # "email": "tiago@virtuacrop.com",
    # "password": "tt1234"
}

response = requests.post(endpoint, json=payload)
if response.status_code == 200:
    print("Resposta da API NDVI:")
    print(response.json())
else:
    print("Erro:", response.status_code, response.text)
```

### 2. Endpoint de Cálculo de Zonas de Produtividade

- **URL**: `/productivity`  
- **Método**: `POST`

#### Descrição

Calcula zonas de produtividade utilizando imagens NDVI do Sentinel-2 para um polígono dado. Inclui:
- NDVI médio ao longo de um período histórico
- Reprojeção EPSG:4326
- Recorte do raster NDVI
- Classificação em 4 zonas utilizando percentis NDVI (20º, 50º, 80º)
- Suavização com filtro mediano
- Carregamento para o Google Cloud Storage

#### Corpo do Pedido

- `polygon` (obrigatório): Um polígono GeoJSON
- `email` (opcional): Para autenticação Firebase
- `password` (opcional): Para autenticação Firebase

<details>
<summary>📌 Exemplo de Polígono</summary>

```json
{
  "type": "Polygon",
  "coordinates": [[
    [-8.5530436, 39.3620312],
    [-8.5530436, 39.3620312]
  ]]
}
```
</details>

#### Detalhes do Processamento

- **Período**: Calculado automaticamente com base no mês atual:
  - Se antes de Outubro → 1 de Outubro (há 6 anos) → 30 de Setembro (ano anterior)
  - Se Outubro ou depois → 1 de Outubro (há 5 anos) → 30 de Setembro (ano atual)
- **NDVI**: Calculado a partir do Sentinel-2 (NIR: B08, Vermelho: B04), com máscara de nuvens (banda SCL)
- **Classificação**: Baseada em percentis NDVI, depois suavizada

#### Resposta

- **200 OK**:
```json
{
  "productivity_zone_url": "https://storage.googleapis.com/..."
}
```

- **400 Bad Request**: Polígono inválido ou em falta
- **401 Unauthorized**: Falha na autenticação
- **500 Internal Server Error**: Erro no processamento ou no carregamento

#### Exemplo de Utilização (Python)

```python
import requests

BASE_URL = "https://virtuacrop-agrisentinel-793092962822.europe-southwest1.run.app"
endpoint = f"{BASE_URL}/productivity"

payload = {
    "polygon": {
        "type": "Polygon",
        "coordinates": [[
            [-8.5530436, 39.3620312],
            [-8.5530436, 39.3620312]
        ]]
    }
}

response = requests.post(endpoint, json=payload)
print("Código de Estado:", response.status_code)
try:
    print("Resposta JSON:")
    print(response.json())
except Exception as e:
    print("Erro ao decodificar JSON:", e)
```

---

## Resumo

Esta documentação descreve a utilização, entradas e saídas dos endpoints `/ndvi` e `/productivity` da API VirtuaCrop-AgriSentinel, incluindo exemplos de pedidos, resultados esperados e comportamento detalhado para cada processo.

Let me know if you want this saved to a file, or converted to HTML or PDF too!
