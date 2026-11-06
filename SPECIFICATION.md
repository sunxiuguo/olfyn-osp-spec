# OSP v1.1 — Open Scent Protocol Technical Specification

**Version:** 1.1  
**Status:** Draft  
**Date:** 2026-07-01  
**License:** MIT  
**Repository:** github.com/olfyn/olfyn-osp-spec  

---

## What's new in v1.1

| Change | v1.0 | v1.1 |
|---|---|---|
| Capture sensor | MQ kit (industrial gas) | BME688 (VOC-specific) |
| Capture method | Not defined | Hybrid (sensor + AI) |
| New endpoint | — | `POST /v1/capture` |
| OSP object | No capture field | `capture` field added |
| Profile categories | 3 (core) | 5 (+ wines + perfumery) |

---

## Table of Contents

1. [Introduction](#1-introduction)
2. [Data Model](#2-data-model)
3. [Hybrid Capture](#3-hybrid-capture)
4. [OSP Sessions](#4-osp-sessions)
5. [Transport Protocol](#5-transport-protocol)
6. [Device Capabilities](#6-device-capabilities)
7. [Profile Categories](#7-profile-categories)
8. [OLFYN DB](#8-olfyn-db)
9. [Error Codes](#9-error-codes)
10. [Certification](#10-certification)
11. [Versioning](#11-versioning)
12. [Governance](#12-governance)

---

## 1. Introduction

### 1.1 Purpose

OSP defines a standard way to represent, transmit, and reproduce olfactory experiences digitally. It is hardware-agnostic, extensible, and JSON-based.

### 1.2 Design Philosophy

Two types of profiles exist in OSP:

**Predefined profiles** — for scents with documented chemistry:
- Wines: aromatic chemistry documented by enologists
- Perfumery: olfactory notes published by fragrance houses
- Wellness: aromatherapy protocols published by qualified professionals

**Captured profiles** — for unique personal scent signatures:
- Memory: a person's meaningful scents (garden, home, family)
- Presence: a partner's personal scent signature
- Remembrance: preserving the scent of someone who has passed

The **hybrid capture** method (new in v1.1) is the recommended approach for captured profiles.

### 1.3 Health and safety notice

OSP is a data format, not a medical device. It makes no therapeutic or clinical claims. Profiles, sessions and categories that mention memory, wellness or therapy describe intended uses only. Any health-related use must follow the guidance of qualified professionals and the regulations that apply where it is used.

---

## 2. Data Model

### 2.1 Full OSP Profile Object

```json
{
  "osp_version":  "1.1",
  "scent_id":     "string",
  "name":         "string",
  "description":  "string",
  "author":       "string",
  "created_at":   "ISO 8601",
  "updated_at":   "ISO 8601",

  "profile": {
    "floral":    0.0,
    "frutal":    0.0,
    "herbal":    0.0,
    "leñoso":    0.0,
    "ahumado":   0.0,
    "especiado": 0.0,
    "animal":    0.0,
    "químico":   0.0,
    "marino":    0.0,
    "dulce":     0.0
  },

  "intensity":    0.75,
  "duration_ms":  5000,
  "fade_in_ms":   500,
  "fade_out_ms":  1000,
  "repeat":       1,
  "interval_ms":  0,

  "capture": {
    "method":              "hybrid|manual|predefined",
    "sensor_data": {
      "voc_index":         245,
      "voc_raw":           18500,
      "temperature":       24.5,
      "humidity":          58.2,
      "dominant_note":     "floral",
      "volatility":        "high|medium|low",
      "readings":          [18200, 18350, 18500]
    },
    "verbal_description":  "string",
    "ai_model":            "string",
    "confidence":          0.74
  },

  "metadata": {
    "category":   "memoria|presencia|recuerdo|vinos|perfumeria|general",
    "person":     "string",
    "tags":       ["string"],
    "notes":      "string",
    "language":   "es|en|..."
  },

  "extensions": {
    "x-vendor-key": "value"
  }
}
```

### 2.2 The 10 Aromatic Dimensions

| Dimension | Description | Natural Examples |
|---|---|---|
| floral | Flower aromas | Rose, jasmine, lavender |
| frutal | Fruit aromas | Citrus, mango, strawberry |
| herbal | Green and herbaceous | Mint, basil, eucalyptus |
| leñoso | Wood and earth | Cedar, sandalwood, pine |
| ahumado | Combustion | Smoke, charcoal, tobacco |
| especiado | Warm and spicy | Cinnamon, pepper, cardamom |
| animal | Body and musk | Musk, leather, amber |
| químico | Synthetic | Medicinal, plastic |
| marino | Aquatic and saline | Sea, seaweed, rain |
| dulce | Sweet | Vanilla, caramel, honey |

> **Rule:** Each dimension is independent (0.0–1.0). The sum does not need to equal 1.0. Recommended maximum total sum: 3.0.

---

## 3. Hybrid Capture

### 3.1 Overview

Hybrid capture is the recommended method for creating personal scent signature profiles. It combines three information sources:

1. **BME688 sensor** — quantitative VOC data
2. **Verbal description** — qualitative user input
3. **AI synthesis** — Claude translates both into OSP dimensions

### 3.2 Why BME688 (not MQ sensors)

| Sensor | Designed for | Relevance to aromas |
|---|---|---|
| MQ kit (v1.0) | Industrial gas safety (butane, CO, methane) | ❌ Not designed for organic aromatics |
| BME688 (v1.1) | VOC detection + air quality | ✅ Purpose-built for organic volatile compounds |

The BME688 provides a **VOC Index** (0–500) that correlates with the presence and intensity of organic volatile compounds — the chemical class that produces most pleasant aromas.

### 3.3 Capture Endpoint

```
POST /v1/capture
```

**Request:**
```json
{
  "sensor_data": {
    "voc_index":   245,
    "voc_raw":     18500,
    "temperature": 24.5,
    "humidity":    58.2,
    "readings":    [18200, 18350, 18500, 18490, 18510]
  },
  "verbal_description": "Sweet, floral, a hint of wood. It's my grandmother's perfume.",
  "category":   "memoria",
  "person":     "Abuela Rosa",
  "save":        true
}
```

**Response:** A full OSP v1.1 profile object conforming to
[`schema/osp-profile-v1.1.json`](schema/osp-profile-v1.1.json), including the required
`osp_version` and `created_at` fields. The `capture` field records how it was created.

```json
{
  "osp_version": "1.1",
  "scent_id":  "memoria_abuela_rosa_a3f2b1",
  "name":      "Perfume de la abuela Rosa",
  "created_at": "2026-07-01T12:00:00Z",
  "profile": {
    "dulce":    0.35,
    "floral":   0.30,
    "leñoso":   0.20,
    "animal":   0.10,
    "herbal":   0.05,
    "frutal":   0.00,
    "ahumado":  0.00,
    "especiado":0.00,
    "marino":   0.00,
    "químico":  0.00
  },
  "intensity":   0.60,
  "duration_ms": 180000,
  "capture": {
    "method":             "hybrid",
    "confidence":          0.72,
    "verbal_description": "Sweet, floral, a hint of wood...",
    "ai_model":           "claude-haiku-4-5"
  }
}
```

### 3.4 Complete Capture Flow

```
1. App: "Bring the object close to the OLFYN sensor"
2. BME688 reads for 30 seconds
   → voc_index, temperature, humidity, readings array
3. App: "The sensor detects [intensity] with
         [dominant_note] predominance.
         Now describe it in your own words."
4. User describes verbally
5. App sends to POST /v1/capture
6. Server calls AI with both data sources
7. AI returns OSP profile JSON
8. App shows sliders for fine-tuning
9. User confirms or adjusts
10. Profile saved to OLFYN DB
```

### 3.5 Confidence Score

The `confidence` field (0.0–1.0) indicates how well the sensor data and verbal description aligned:

| Range | Meaning |
|---|---|
| 0.8–1.0 | High — sensor and description strongly aligned |
| 0.6–0.8 | Medium — minor discrepancies, profile reliable |
| 0.4–0.6 | Low — notable gap between sensor and description |
| < 0.4 | Very low — suggest re-capture or manual adjustment |

---

## 4. OSP Sessions

A session is an ordered sequence of profiles, e.g. for a reminiscence or wellness activity.

```json
{
  "osp_version":       "1.1",
  "session_id":        "string",
  "session_name":      "string",
  "category":          "memoria|therapy|wellness|custom",
  "total_duration_ms": 1200000,
  "sequence": [
    {
      "offset_ms":          0,
      "scent_id":           "olfyn_lavanda_001",
      "intensity_override": 0.60,
      "label":              "Calming opening"
    },
    {
      "offset_ms": 300000,
      "scent_id":  "memoria_jardin_mama_001",
      "label":     "Personal memory"
    }
  ],
  "log_responses": true
}
```

---

## 5. Transport Protocol

### 5.1 REST Endpoints

| Method | Endpoint | Description |
|---|---|---|
| GET | /v1/profiles | List profiles (filter by category) |
| POST | /v1/profiles | Create profile manually |
| GET | /v1/profiles/{id} | Get specific profile |
| PUT | /v1/profiles/{id} | Update profile |
| DELETE | /v1/profiles/{id} | Delete profile |
| **POST** | **/v1/capture** | **NEW — Hybrid capture** |
| POST | /v1/emit | Activate profile |
| POST | /v1/emit/stop | Stop emission |
| GET | /v1/sessions | List sessions |
| POST | /v1/sessions | Create session |
| POST | /v1/sessions/start | Execute session |
| POST | /v1/sessions/stop | Stop session |
| GET | /v1/status | Device status |
| GET | /v1/capabilities | Device capabilities |
| GET | /v1/health | Health check |
| WS | /v1/stream | Real-time WebSocket |

### 5.2 WebSocket Events

**Client → Device:**
```json
{ "action": "emit", "scent_id": "...", "intensity": 0.7 }
{ "action": "stop" }
{ "action": "session_start", "session_id": "..." }
```

**Device → Client:**
```json
{ "event": "emission_started", "scent_id": "..." }
{ "event": "emission_ended",   "scent_id": "..." }
{ "event": "capture_ready",    "voc_index": 245 }
{ "event": "error",            "code": "E011", "message": "..." }
```

---

## 6. Device Capabilities

```json
{
  "osp_version_supported": ["1.0", "1.1"],
  "device_name":   "OLFYN One",
  "firmware":      "1.1.0",
  "hardware_mode": "simulator|raspberry|mock",

  "sensors": {
    "bme688": true,
    "voc_capture": true
  },

  "actuators": {
    "count":     6,
    "type":      "liquid",
    "mechanism": "ultrasonic-mist",
    "slots": [
      { "id": 1, "dimension": "floral",  "filled": true },
      { "id": 2, "dimension": "dulce",   "filled": true },
      { "id": 3, "dimension": "leñoso",  "filled": true },
      { "id": 4, "dimension": "animal",  "filled": true },
      { "id": 5, "dimension": "frutal",  "filled": true },
      { "id": 6, "dimension": "herbal",  "filled": true }
    ]
  },

  "features": {
    "sessions":       true,
    "hybrid_capture": true,
    "fade":           true,
    "repeat":         true
  }
}
```

---

## 7. Profile Categories

| Category | Creation method | Description |
|---|---|---|
| `memoria` | Hybrid capture | Personal meaningful scents for memory and reminiscence |
| `presencia` | Hybrid capture | Partner's personal scent signature |
| `recuerdo` | Hybrid capture | Preserved scent of someone who has passed |
| `vinos` | Predefined | Documented wine aromatic chemistry |
| `perfumeria` | Predefined | Published perfumery olfactory notes |
| `therapy` | Predefined | Aromatherapy protocols published by qualified professionals (no clinical claim by OSP) |
| `general` | Any | Open category |

> Categories are open strings — any value is valid. New categories can be added without modifying the protocol.

---

## 8. OLFYN DB

Public database of community OSP profiles.

| Prefix | Type | Description |
|---|---|---|
| `olfyn_` | Official | Validated by OLFYN team |
| `personal_` | Personal | Private user profiles |
| `comm_` | Community | Community-verified |
| `therapy_` | Therapeutic | Published by qualified professionals; source required |
| `exp_` | Experimental | Unverified, in testing |

---

## 9. Error Codes

| Code | Name | Description |
|---|---|---|
| E001 | INVALID_PROFILE | JSON does not conform to OSP schema |
| E002 | UNKNOWN_SCENT_ID | Profile not found |
| E003 | CARTRIDGE_EMPTY | Required scent slot is empty |
| E004 | DIMENSION_MISSING | Dimension not available on device |
| E005 | ALREADY_EMITTING | Device already emitting |
| E006 | SESSION_NOT_FOUND | Session not found |
| E007 | INTENSITY_OUT_OF_RANGE | Value outside 0.0–1.0 |
| E008 | DEVICE_NOT_READY | Device initializing or in error |
| E009 | VERSION_MISMATCH | OSP version not supported |
| E010 | NETWORK_TIMEOUT | Device did not respond |
| E011 | CAPTURE_FAILED | Hybrid capture failed — sensor or AI unavailable |
| E012 | SENSOR_NOT_CONNECTED | BME688 not detected on I2C bus |

---

## 10. Certification

An **"OSP v1.1 Compatible"** device must:

1. Implement all required REST endpoints (sections 5.1)
2. Support the full OSP object including the `capture` field
3. Correctly expose `GET /v1/capabilities` with `sensors.bme688` field
4. Handle error codes E001–E012
5. Support at least 4 simultaneous active scent actuators
6. Implement `POST /v1/capture` (hybrid capture pipeline)

---

## 11. Versioning

OSP follows [Semantic Versioning](https://semver.org/):

- **MAJOR** (X.0) — Breaking changes
- **MINOR** (1.X) — New backwards-compatible features
- **PATCH** (1.0.X) — Specification bug fixes

---

## 12. Governance

**License:** MIT — see [LICENSE](LICENSE)  
**Repository:** [github.com/olfyn/olfyn-osp-spec](https://github.com/olfyn/olfyn-osp-spec)

**Change process:**
1. Propose via GitHub Issues
2. Community review (30 days)
3. Incorporate in next MINOR or MAJOR version
4. Publish with detailed changelog

---

*OSP v1.1 · OLFYN · July 2026 · MIT License*
