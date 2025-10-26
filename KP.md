# KP Multiple Mahasiswa Implementation

## Overview

This document outlines the changes made to support multiple mahasiswa per KP (Kerja Praktek) instead of the previous one-to-one relationship.

## Key Changes Made

### 1. Model Updates

- **File**: `internal/domain/models/kp.go`
- **Changes**:
  - Added `Kuota` field support in `NewKp()` function
  - KP model already had `Mahasiswa []Mahasiswa` field for one-to-many relationship
  - Updated constructor to accept `kuota` parameter

### 2. DTO Updates

- **File**: `internal/application/dto/kp_dto.go`
- **Changes**:
  - Added `Kuota` field to `CreateKpRequest` and `CreateKpResponse`
  - Added `Kuota` field to `UpdateKpRequest` and `UpdateKpResponse`
  - Created new `MahasiswaInfo` struct for mahasiswa details
  - Updated `GetKpByIdResponse` and `GetAllKpResponse` to include:
    - `Mahasiswa []MahasiswaInfo` instead of single mahasiswa fields
    - `JumlahMahasiswa int` field
    - `Kuota uint` field
  - Added new DTOs:
    - `AssignMultipleMahasiswaRequest`
    - `UnassignMahasiswaRequest`
    - `GetKpMahasiswaResponse`

### 3. Repository Updates

- **File**: `internal/domain/repository/kp_repository.go`
- **Changes**:
  - Updated repository interface with new methods:
    - `AssignMultipleMahasiswa()`
    - `UnassignMahasiswa()` (now requires mahasiswaId parameter)
    - `UnassignAllMahasiswa()`
    - `GetKpMahasiswa()`
  - Updated implementation to work with mahasiswa table directly
  - Changed assignment logic to update mahasiswa's `KPId` field instead of KP's mahasiswa_id field

### 4. Service Updates

- **File**: `internal/application/services/kp_service.go`
- **Changes**:
  - Updated service interface with new methods:
    - `AssignMultipleMahasiswa()`
    - `UnassignMahasiswa()` (now requires UnassignMahasiswaRequest)
    - `UnassignAllMahasiswa()`
    - `GetKpMahasiswa()`
  - Updated `CreateKp()` to handle kuota parameter
  - Updated `UpdateKp()` to handle kuota updates
  - Updated mapping functions to handle multiple mahasiswa:
    - `mapKpToGetAllResponse()` now maps all mahasiswa
    - `mapKpToGetByIdResponse()` now maps all mahasiswa
  - Added proper null safety for all operations

### 5. Controller Updates

- **File**: `internal/presentation/controllers/kp_controller.go`
- **Changes**:
  - Updated controller interface with new methods:
    - `AssignMultipleMahasiswa()`
    - `UnassignAllMahasiswa()`
    - `GetKpMahasiswa()`
  - Updated `UnassignMahasiswa()` to accept request body
  - Added implementations for all new controller methods

### 6. Router Updates

- **File**: `internal/router/kp_router.go`
- **Changes**:
  - Added new routes:
    - `POST /:id/assign-multiple` - Assign multiple mahasiswa to KP
    - `DELETE /:id/assign-all` - Unassign all mahasiswa from KP
    - `GET /:id/mahasiswa` - Get all mahasiswa assigned to KP
  - Updated existing routes with better documentation

### 7. Seeder Updates

- **File**: `database/seeder/kp/kp_seeder.go`
- **Changes**:
  - Added `kuota` field to seeder data
  - Updated KP creation to include kuota values (2-5 mahasiswa per KP)
  - Fixed status field type casting

## API Endpoints

### Existing Endpoints (Updated)

```
POST   /api/kp                    - Create KP (now includes kuota)
GET    /api/kp                    - Get all KP (now shows all mahasiswa)
GET    /api/kp/:id                - Get KP by ID (now shows all mahasiswa)
PUT    /api/kp/:id                - Update KP (now includes kuota)
DELETE /api/kp/:id                - Delete KP
POST   /api/kp/:id/assign         - Assign single mahasiswa
DELETE /api/kp/:id/assign         - Unassign single mahasiswa (now requires body)
```

### New Endpoints

```
POST   /api/kp/:id/assign-multiple  - Assign multiple mahasiswa
DELETE /api/kp/:id/assign-all       - Unassign all mahasiswa
GET    /api/kp/:id/mahasiswa        - Get KP mahasiswa details
```

## Request/Response Examples

### Create KP (Updated)

```json
POST /api/kp
{
  "judul": "Web Development Project",
  "deskripsi": "Full-stack web development using Go and React",
  "id_dosen": "uuid-here",
  "kuota": 3,
  "start_at": "2025-01-01T00:00:00Z",
  "end_at": "2025-06-01T00:00:00Z"
}
```

### Assign Multiple Mahasiswa

```json
POST /api/kp/:id/assign-multiple
{
  "id_mahasiswa": [
    "mahasiswa-uuid-1",
    "mahasiswa-uuid-2",
    "mahasiswa-uuid-3"
  ]
}
```

### Unassign Single Mahasiswa

```json
DELETE /api/kp/:id/assign
{
  "id_mahasiswa": "mahasiswa-uuid-1"
}
```

### Get KP Response (Updated)

```json
{
  "id_kp": "kp-uuid",
  "judul": "Web Development Project",
  "deskripsi": "Full-stack web development...",
  "status": "Tersedia",
  "kuota": 3,
  "id_dosen": "dosen-uuid",
  "nama_dosen": "Dr. John Doe",
  "mahasiswa": [
    {
      "id_mahasiswa": "mahasiswa-uuid-1",
      "nrp_mahasiswa": "5025201001",
      "nama_mahasiswa": "Alice Johnson"
    },
    {
      "id_mahasiswa": "mahasiswa-uuid-2",
      "nrp_mahasiswa": "5025201002",
      "nama_mahasiswa": "Bob Smith"
    }
  ],
  "jumlah_mahasiswa": 2,
  "start_at": "2025-01-01",
  "end_at": "2025-06-01"
}
```

## Database Schema Implications

The existing schema already supports this relationship:

- `KP` table has a `kuota` field
- `Mahasiswa` table has a `kp_id` field that references KP
- The relationship is properly set up as one-to-many

## Benefits of This Implementation

1. **Scalability**: KP can now accommodate multiple mahasiswa based on kuota
2. **Flexibility**: Easy to assign/unassign individual or multiple mahasiswa
3. **Data Integrity**: Proper validation of kuota limits
4. **Performance**: Efficient queries for mahasiswa management
5. **Maintainability**: Clean separation of concerns in all layers

## Usage Examples

1. **Create a KP with kuota of 3 students**
2. **Assign multiple mahasiswa at once**
3. **Check current mahasiswa assignments**
4. **Unassign specific mahasiswa when they drop out**
5. **Unassign all mahasiswa when KP is cancelled**

The implementation maintains backward compatibility where possible and provides a robust foundation for managing multiple mahasiswa per KP.
