# **DEM Backend API Documentation**

## **Base Information**

**Base URL**: `https://dem-api.road.engineering`  
**Development**: `http://localhost:8001`  
**API Version**: v1  
**Authentication**: API Key (future implementation)  
**Rate Limits**: 100 requests/hour (development), 1000 requests/hour (production)

## **Endpoints Overview**

| Endpoint | Method | Purpose | Use Case | Frontend Integration |
|----------|--------|---------|----------|---------------------|
| `/api/v1/elevation/point` | POST | Single coordinate | Individual location lookup | ✅ Direct |
| `/api/v1/elevation/points` | POST | Multiple discrete coordinates | Waypoints, markers, POIs | ✅ Direct |
| `/api/v1/elevation/line` | POST | Start/end line sampling | Sight lines, straight profiles | ✅ Direct |
| `/api/v1/elevation/path` | POST | Complex path sampling | Road alignments, GPS tracks | ✅ Direct |
| `/api/v1/elevation/contour-data` | POST | **NEW**: Grid elevation sampling | Contour generation | ✅ **Direct Only** |
| `/api/v1/elevation/sources` | GET | Available data sources | Source management | ✅ Direct |
| `/api/v1/health` | GET | Service health check | Monitoring | ✅ Direct |
| `/attribution` | GET | Data source credits | Legal compliance | ✅ Direct |

---

## **Single Point Elevation**

### `POST /api/v1/elevation/point`

Get elevation for a single coordinate.

**Request:**
```json
{
  "lat": -27.4698,
  "lon": 153.0251,
  "source": "ga_elvis"  // optional
}
```

**Response:**
```json
{
  "status": "OK",
  "results": [
    {
      "lat": -27.4698,
      "lon": 153.0251,
      "elevation": 45.2,
      "data_source": "ga_elvis",
      "resolution": 1.0,
      "interpolation": "bilinear"
    }
  ],
  "metadata": {
    "total_points": 1,
    "successful_points": 1,
    "crs": "EPSG:4326",
    "units": "meters",
    "attribution_url": "https://dem-api.road.engineering/attribution"
  }
}
```

---

## **Multiple Points (Batch)**

### `POST /api/v1/elevation/points`

Get elevations for multiple discrete coordinates.

**Request:**
```json
{
  "points": [
    {"lat": -27.4698, "lon": 153.0251},
    {"lat": -27.4705, "lon": 153.0258},
    {"lat": -27.4712, "lon": 153.0265}
  ],
  "source": "ga_elvis"  // optional
}
```

**Response:**
```json
{
  "status": "OK",
  "results": [
    {
      "lat": -27.4698, "lon": 153.0251, "elevation": 45.2,
      "data_source": "ga_elvis", "resolution": 1.0
    },
    {
      "lat": -27.4705, "lon": 153.0258, "elevation": 47.1,
      "data_source": "ga_elvis", "resolution": 1.0
    },
    {
      "lat": -27.4712, "lon": 153.0265, "elevation": 48.9,
      "data_source": "ga_elvis", "resolution": 1.0
    }
  ],
  "metadata": {
    "total_points": 3,
    "successful_points": 3,
    "crs": "EPSG:4326",
    "units": "meters"
  }
}
```

---

## **Simple Line Sampling**

### `POST /api/v1/elevation/line`

Get elevations for evenly spaced points along a straight line.

**Request:**
```json
{
  "start": {"lat": -27.4698, "lon": 153.0251},
  "end": {"lat": -27.4705, "lon": 153.0258},
  "num_points": 10,
  "source": "ga_elvis"  // optional
}
```

**Response:**
```json
{
  "status": "OK",
  "results": [
    {
      "lat": -27.4698, "lon": 153.0251, "elevation": 45.2,
      "distance_m": 0.0, "sequence": 0, "data_source": "ga_elvis"
    },
    {
      "lat": -27.4699, "lon": 153.0252, "elevation": 45.5,
      "distance_m": 89.1, "sequence": 1, "data_source": "ga_elvis"
    }
    // ... 8 more points
  ],
  "metadata": {
    "total_points": 10,
    "total_distance_m": 801.2,
    "interpolation": "geodesic"
  }
}
```

---

## **Complex Path Sampling**

### `POST /api/v1/elevation/path`

Get elevations for evenly spaced points along a complex multi-segment path.

**Request:**
```json
{
  "path": [
    {"lat": -27.4698, "lon": 153.0251},
    {"lat": -27.4705, "lon": 153.0258},
    {"lat": -27.4712, "lon": 153.0265},
    {"lat": -27.4720, "lon": 153.0275}
  ],
  "num_points": 50,
  "interpolation": "geodesic",  // "geodesic" or "linear"
  "source": "ga_elvis"  // optional
}
```

**Response:**
```json
{
  "status": "OK",
  "results": [
    {
      "lat": -27.4698, "lon": 153.0251, "elevation": 45.2,
      "distance_m": 0.0, "sequence": 0, "data_source": "ga_elvis"
    },
    {
      "lat": -27.4699, "lon": 153.0252, "elevation": 45.5,
      "distance_m": 57.3, "sequence": 1, "data_source": "ga_elvis"
    }
    // ... 48 more evenly-spaced points
  ],
  "metadata": {
    "total_points": 50,
    "successful_points": 50,
    "total_distance_m": 2847.3,
    "path_segments": 3,
    "interpolation": "geodesic"
  }
}
```

---

## **Source Management**

### `GET /api/v1/elevation/sources`

List available elevation data sources.

**Response:**
```json
{
  "sources": {
    "ga_elvis": {
      "description": "Geoscience Australia ELVIS (1m LiDAR)",
      "resolution": 1.0,
      "coverage": "Australia",
      "priority": 1
    },
    "srtm_global": {
      "description": "NASA SRTM Global (30m)",
      "resolution": 30.0,
      "coverage": "Global",
      "priority": 3
    }
  },
  "default_source": "ga_elvis",
  "total_sources": 2
}
```

---

## **Service Status**

### `GET /api/v1/health`

Check service health and availability.

**Response:**
```json
{
  "status": "healthy",
  "service": "DEM Backend API",
  "version": "v1.0.0",
  "uptime_seconds": 86400,
  "sources_available": 5,
  "last_check": "2025-07-16T14:30:00Z"
}
```

---

## **Attribution**

### `GET /attribution`

View data source attribution and licensing information.

**Response**: HTML page with Creative Commons compliance details for all elevation data sources.

---

## **Error Responses**

**Standard Error Format:**
```json
{
  "status": "ERROR",
  "error": {
    "code": 400,
    "message": "Invalid coordinates",
    "details": "Latitude must be between -90 and 90",
    "timestamp": "2025-07-16T14:30:00Z"
  }
}
```

**Common Error Codes:**
- `400` - Bad Request (invalid parameters)
- `404` - Not Found (endpoint doesn't exist)
- `429` - Too Many Requests (rate limit exceeded)
- `500` - Internal Server Error
- `503` - Service Unavailable (maintenance)

---

## **Request Parameters**

### **Common Parameters**

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `lat` | float | Yes | Latitude (-90 to 90) |
| `lon` | float | Yes | Longitude (-180 to 180) |
| `source` | string | No | Specific elevation source ID |

### **Path-Specific Parameters**

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `num_points` | integer | Yes | Number of points to generate (2-1000) |
| `interpolation` | string | No | "geodesic" (default) or "linear" |

---

## **Response Fields**

### **Elevation Result Object**

| Field | Type | Description |
|-------|------|-------------|
| `lat` | float | Latitude of point |
| `lon` | float | Longitude of point |
| `elevation` | float | Elevation in meters (null if unavailable) |
| `data_source` | string | Source ID used for this point |
| `resolution` | float | Data resolution in meters |
| `distance_m` | float | Distance from start (path/line only) |
| `sequence` | integer | Point order in sequence |

### **Metadata Object**

| Field | Type | Description |
|-------|------|-------------|
| `total_points` | integer | Total points requested |
| `successful_points` | integer | Points with valid elevations |
| `crs` | string | Coordinate reference system |
| `units` | string | Elevation units |
| `attribution_url` | string | Link to data attribution page |

---

## **Usage Examples**

### **Frontend Integration**
```javascript
// Single point lookup
const response = await fetch('/api/v1/elevation/point', {
  method: 'POST',
  headers: { 'Content-Type': 'application/json' },
  body: JSON.stringify({ lat: -27.4698, lon: 153.0251 })
});

// Road profile generation
const profile = await fetch('/api/v1/elevation/path', {
  method: 'POST',
  headers: { 'Content-Type': 'application/json' },
  body: JSON.stringify({
    path: roadCoordinates,
    num_points: 100,
    interpolation: 'geodesic'
  })
});
```

### **Batch Processing**
```javascript
// Multiple waypoints
const elevations = await fetch('/api/v1/elevation/points', {
  method: 'POST',
  headers: { 'Content-Type': 'application/json' },
  body: JSON.stringify({
    points: [
      { lat: -27.4698, lon: 153.0251 },
      { lat: -27.4705, lon: 153.0258 }
    ]
  })
});
```

---

## **Grid Elevation Data (NEW)**

### `POST /api/v1/elevation/contour-data`

**Purpose**: Generate grid elevation data within polygon bounds for contour generation.

**Use Case**: Frontend contour map generation, terrain visualization.

**Request Body**:
```json
{
  "area_bounds": {
    "polygon_coordinates": [
      {"latitude": -27.4698, "longitude": 153.0251},
      {"latitude": -27.4705, "longitude": 153.0258},
      {"latitude": -27.4712, "longitude": 153.0265},
      {"latitude": -27.4698, "longitude": 153.0251}
    ]
  },
  "grid_resolution_m": 10.0,
  "source": "local_dtm_gdb"
}
```

**Response**:
```json
{
  "status": "OK",
  "dem_points": [
    {
      "latitude": -27.4698,
      "longitude": 153.0251,
      "elevation_m": 45.2,
      "x_grid_index": 0,
      "y_grid_index": 0,
      "grid_resolution_m": 10.0
    }
  ],
  "total_points": 1000,
  "dem_source_used": "local_dtm_gdb",
  "grid_info": {
    "total_width": 50,
    "total_height": 20,
    "grid_resolution_m": 10.0,
    "dem_native_resolution_m": 1.0,
    "bounds": {
      "min_lat": -27.4712,
      "max_lat": -27.4698,
      "min_lon": 153.0251,
      "max_lon": 153.0265
    }
  },
  "crs": "EPSG:4326",
  "message": "Contour data generated successfully."
}
```

**Request Parameters**:

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `area_bounds.polygon_coordinates` | array | Yes | Polygon boundary coordinates (min 3 points) |
| `grid_resolution_m` | float | No | Grid spacing in meters (default: 10.0) |
| `source` | string | No | Specific DEM source to use |

**Response Fields**:

| Field | Type | Description |
|-------|------|-------------|
| `status` | string | "OK" or "ERROR" |
| `dem_points` | array | Grid elevation points with coordinates and indices |
| `total_points` | integer | Number of grid points returned |
| `dem_source_used` | string | DEM source that provided the data |
| `grid_info` | object | Grid metadata (dimensions, bounds, resolution) |
| `crs` | string | Coordinate reference system |
| `message` | string | Success or error message |

**Error Response**:
```json
{
  "status": "ERROR",
  "error": {
    "code": 400,
    "message": "Invalid coordinates",
    "details": "Polygon must have at least 3 coordinates",
    "timestamp": "2025-07-16T14:30:00Z"
  }
}
```

### **Frontend Integration Example**
```javascript
// Generate contour data for map display
const contourData = await fetch('http://localhost:8001/api/v1/elevation/contour-data', {
  method: 'POST',
  headers: { 'Content-Type': 'application/json' },
  body: JSON.stringify({
    area_bounds: {
      polygon_coordinates: mapBounds
    },
    grid_resolution_m: 5.0
  })
});

const result = await contourData.json();
if (result.status === 'OK') {
  // Use result.dem_points for contour generation
  generateContourLines(result.dem_points, result.grid_info);
}
```

---

**This API provides comprehensive elevation services for road engineering applications with support for single points, batch processing, and complex path analysis.**