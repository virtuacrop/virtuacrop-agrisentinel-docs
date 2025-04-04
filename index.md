# VirtuaCrop-AgriSentinel API Documentation

The **VirtuaCrop-AgriSentinel API** provides geospatial analysis tools that leverage satellite imagery to compute vegetation indices and productivity zones.

**Base URL**:  
`https://virtuacrop-agrisentinel-793092962822.europe-southwest1.run.app`

---

## Endpoints

### 1. Calculate NDVI Endpoint

- **URL**: `/ndvi`  
- **Method**: `POST`

#### Description

This endpoint calculates the **median NDVI (Normalized Difference Vegetation Index)** over a specified polygon using satellite imagery. It supports three data sources:

- `sentinel2`
- `landsat8`
- `sentinel2_landsat8` (merged dataset)

Optional Firebase authentication logs request/usage data. If authentication is provided, it generates and uploads an NDVI **NetCDF** file to Google Cloud Storage.

#### Request Body

The request must be a JSON payload with the following fields:

- `polygon` (required): A GeoJSON object defining the area of interest.  
- `start_date` (required): Format `"YYYY-MM-DD"`  
- `end_date` (required): Format `"YYYY-MM-DD"`  
- `max_cloud_cover` (optional): Max cloud cover percentage (default: `50.0`)  
- `collection` (optional): `"sentinel2"`, `"landsat8"`, or `"sentinel2_landsat8"` (default: `"sentinel2"`)  
- `email` (optional): For Firebase auth  
- `password` (optional): For Firebase auth  
- `include_image` (optional): Boolean (default: `true`)  
- `include_median` (optional): Boolean (default: `true`)  

<details>
<summary>📌 Example Polygon</summary>

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

#### Response

The response includes:
- `median_values`: List of NDVI values per date
  - `DATA`: Date in "YYYY-MM-DD"
  - `VALUES`: Median NDVI (nullable if no valid data)
- `netcdf_download_url`: Signed URL (1-hour validity) for downloading the NetCDF file

Example response:
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

#### Example Usage (Python)

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
    print("NDVI API Response:")
    print(response.json())
else:
    print("Error:", response.status_code, response.text)
```

### 2. Calculate Productivity Zones Endpoint

- **URL**: `/productivity`  
- **Method**: `POST`

#### Description

Calculates productivity zones using Sentinel-2 NDVI imagery for a given polygon. Includes:
- Mean NDVI over a historical date range
- EPSG:4326 reprojection
- NDVI raster clipping
- Classification into 4 zones using NDVI percentiles (20th, 50th, 80th)
- Median filter smoothing
- Upload to Google Cloud Storage

#### Request Body

- `polygon` (required): A GeoJSON polygon
- `email` (optional): For Firebase auth
- `password` (optional): For Firebase auth

<details>
<summary>📌 Example Polygon</summary>

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

#### Processing Details

- **Date Range**: Automatically calculated based on current month:
  - If before October → Oct 1 (6 years ago) → Sep 30 (previous year)
  - If October or later → Oct 1 (5 years ago) → Sep 30 (current year)
- **NDVI**: Computed from Sentinel-2 (NIR: B08, Red: B04), cloud masked (SCL band)
- **Classification**: Based on NDVI percentiles, then smoothed

#### Response

- **200 OK**:
```json
{
  "productivity_zone_url": "https://storage.googleapis.com/..."
}
```

- **400 Bad Request**: Invalid or missing polygon
- **401 Unauthorized**: Auth failed
- **500 Internal Server Error**: Processing or upload error

#### Example Usage (Python)

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
print("Status Code:", response.status_code)
try:
    print("Response JSON:")
    print(response.json())
except Exception as e:
    print("Error decoding JSON:", e)
```

---

## Summary

This documentation outlines the usage, inputs, and outputs for the `/ndvi` and `/productivity` endpoints of the VirtuaCrop-AgriSentinel API, including example requests, expected results, and detailed behavior for each process.

Let me know if you want this saved to a file, or converted to HTML or PDF too!

