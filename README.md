# 📚 DOCUMENTACIÓN - MÓDULO RESERVAS, CHECK-IN/OUT Y PASES MENSUALES

**Autor:** Santiago Abuawad 
**Fecha:** Diciembre 2025  
**Tecnología:** Spring Boot 3.2.0 + Java 17

---

## 📖 ÍNDICE

1. [Resumen Ejecutivo](#resumen)
2. [Arquitectura](#arquitectura)
3. [Patrones de Diseño](#patrones)
4. [Componentes Implementados](#componentes)
5. [Endpoints REST](#endpoints)
6. [Sistema QR](#sistema-qr)
7. [Validaciones](#validaciones)
8. [Instalación y Uso](#instalacion)

---

## 1. RESUMEN EJECUTIVO <a id="resumen"></a>

### 1.1 Alcance del Módulo

Este módulo implementa la funcionalidad completa de:
- ✅ Gestión de Reservas
- ✅ Sistema de Check-In/Check-Out con cálculo de tiempo
- ✅ Pases Mensuales con 3 tipos (Básico, Premium, Empresarial)
- ✅ Generación y validación de códigos QR con seguridad

### 1.2 Métricas del Proyecto

| Métrica | Cantidad |
|---------|----------|
| **Endpoints REST** | 19 |
| **Modelos** | 4 |
| **DTOs** | 10 |
| **Services** | 4 |
| **Controllers** | 3 |
| **Mappers** | 4 |
| **Patrones de Diseño** | 5 |

---

## 2. ARQUITECTURA <a id="arquitectura"></a>

### 2.1 Arquitectura en Capas

```
┌─────────────────────────────────────┐
│         CAPA PRESENTACIÓN           │
│   (Controllers - REST API)          │
├─────────────────────────────────────┤
│         CAPA APLICACIÓN             │
│   (DTOs + Mappers)                  │
├─────────────────────────────────────┤
│         CAPA NEGOCIO                │
│   (Services + Validaciones)         │
├─────────────────────────────────────┤
│         CAPA DOMINIO                │
│   (Models + Strategy)               │
├─────────────────────────────────────┤
│         CAPA UTILIDADES             │
│   (QRCodeGenerator)                 │
└─────────────────────────────────────┘
```

### 2.2 Estructura de Paquetes

```
com.parking.system/
├── controller/
│   ├── ReservaController.java
│   ├── AccesoController.java
│   └── PaseMensualController.java
├── service/
│   ├── ReservaService.java
│   ├── CheckInService.java
│   ├── CheckOutService.java
│   └── PaseMensualService.java
├── model/
│   ├── Reserva.java
│   ├── CheckIn.java
│   ├── CheckOut.java
│   └── PaseMensual.java
├── dto/
│   ├── CrearReservaRequest.java
│   ├── ReservaResponse.java
│   ├── CheckInRequest.java
│   ├── CheckInResponse.java
│   ├── CheckOutRequest.java
│   ├── CheckOutResponse.java
│   ├── CrearPaseMensualRequest.java
│   └── PaseMensualResponse.java
├── mapper/
│   ├── ReservaMapper.java
│   ├── CheckInMapper.java
│   ├── CheckOutMapper.java
│   └── PaseMensualMapper.java
├── strategy/
│   ├── PrecioPaseStrategy.java (interface)
│   ├── PrecioBasicoStrategy.java
│   ├── PrecioPremiumStrategy.java
│   ├── PrecioEmpresarialStrategy.java
│   └── PrecioPaseContext.java
└── util/
    └── QRCodeGenerator.java
```

---


## 🎨 Diagramas UML

### Diagrama 1: Módulo de Acceso y Reservas
**Archivo:** [`acceso_reservas.drawio`](./acceso_reservas.drawio)

**Muestra:**
- Modelos: Reserva, CheckIn, CheckOut
- Services y Controllers
- Sistema QR
- Relaciones entre componentes

### Diagrama 2: Módulo de Pases Mensuales
**Archivo:** [`pase_mensual.drawio`](./pase_mensual.drawio)

**Muestra:**
- Modelo PaseMensual
- Implementación del Strategy Pattern
- Cálculo dinámico de precios
- Tipos: Básico, Premium, Empresarial

> 💡 **Para visualizar:** Abrir los archivos `.drawio` en [draw.io](https://app.diagrams.net/)

---

## 3. PATRONES DE DISEÑO <a id="patrones"></a>

### 3.1 Strategy Pattern ⭐

**Propósito:** Calcular precios de pases mensuales según el tipo.

**Implementación:**
```java
// Interfaz Strategy
public interface PrecioPaseStrategy {
    BigDecimal calcularPrecio();
    String getTipo();
}

// Estrategia Concreta
@Component
public class PrecioPremiumStrategy implements PrecioPaseStrategy {
    public BigDecimal calcularPrecio() {
        return new BigDecimal("300.00");
    }
}

// Context
@Component
public class PrecioPaseContext {
    public BigDecimal calcularPrecio(String tipo) {
        return strategies.get(tipo).calcularPrecio();
    }
}
```

**Beneficios:**
- ✅ Fácil agregar nuevos tipos de pases
- ✅ Desacopla lógica de cálculo
- ✅ Cumple Open/Closed Principle

---

### 3.2 DTO Pattern

**Propósito:** Separar representación API de modelo de dominio.

**Ejemplo:**
```java
// Request (entrada)
public class CrearReservaRequest {
    private Long usuarioId;
    private Long espacioId;
    private LocalDateTime fechaInicio;
    private LocalDateTime fechaFin;
}

// Response (salida)
public class ReservaResponse {
    private Long id;
    private String qrCode;
    private String estado;
    // ... más campos
}
```

**Beneficios:**
- ✅ Control sobre datos expuestos en API
- ✅ Validación independiente
- ✅ Evolución independiente de API y dominio

---

### 3.3 Mapper Pattern

**Propósito:** Convertir entre Models y DTOs.

**Ejemplo:**
```java
@Component
public class ReservaMapper {
    public ReservaResponse toResponse(Reserva reserva) {
        ReservaResponse response = new ReservaResponse();
        response.setId(reserva.getId());
        response.setQrCode(reserva.getQrCode());
        response.setEstado(reserva.getEstado());
        return response;
    }
}
```

**Beneficios:**
- ✅ Responsabilidad única
- ✅ Reusabilidad
- ✅ Fácil testing

---

### 3.4 Service Layer Pattern

**Propósito:** Centralizar lógica de negocio.

**Características:**
- Todas las validaciones en Services
- Controllers solo delegan
- Transaccionalidad (preparado para @Transactional)

---

### 3.5 Dependency Injection

**Propósito:** Inversión de control y bajo acoplamiento.

**Ejemplo:**
```java
@Service
public class ReservaService {
    private final QRCodeGenerator qrCodeGenerator;
    
    // Constructor injection
    public ReservaService(QRCodeGenerator qrCodeGenerator) {
        this.qrCodeGenerator = qrCodeGenerator;
    }
}
```

---

## 4. COMPONENTES IMPLEMENTADOS <a id="componentes"></a>

### 4.1 Modelos

#### **Reserva**
```java
public class Reserva {
    private Long id;
    private Long usuarioId;
    private Long espacioId;
    private LocalDateTime fechaInicio;
    private LocalDateTime fechaFin;
    private String estado; // CONFIRMADA, EN_USO, COMPLETADA, CANCELADA
    private String qrCode;
    private LocalDateTime fechaCreacion;
}
```

**Estados:**
```
CONFIRMADA → (check-in) → EN_USO → (check-out) → COMPLETADA
           ↘ (cancelar) → CANCELADA
```

#### **CheckIn**
```java
public class CheckIn {
    private Long id;
    private Long reservaId;
    private LocalDateTime horaEntrada;
    private String dispositivoId;
}
```

#### **CheckOut**
```java
public class CheckOut {
    private Long id;
    private Long reservaId;
    private LocalDateTime horaSalida;
    private Long tiempoTotalMinutos;
}
```

#### **PaseMensual**
```java
public class PaseMensual {
    private Long id;
    private Long usuarioId;
    private String tipo; // BASICO, PREMIUM, EMPRESARIAL
    private Long espacioAsignado;
    private LocalDateTime fechaInicio;
    private LocalDateTime fechaVencimiento;
    private BigDecimal precio;
    private String estado; // ACTIVO, VENCIDO, CANCELADO
}
```

---

### 4.2 Servicios

#### **ReservaService**

**Responsabilidades:**
- Crear reservas con validaciones
- Generar códigos QR automáticamente
- Verificar disponibilidad de espacios
- Gestionar estados de reserva

**Validaciones Implementadas:**
1. Fecha inicio < fecha fin
2. Mínimo 1 hora de anticipación
3. Máximo 12 horas de duración
4. Espacio disponible (sin solapamiento)

**Método clave:**
```java
public Reserva crearReserva(Long usuarioId, Long espacioId, 
                           LocalDateTime fechaInicio, LocalDateTime fechaFin) {
    // Validaciones
    validarFechas(fechaInicio, fechaFin);
    validarAnticipacion(fechaInicio);
    validarDuracion(fechaInicio, fechaFin);
    verificarDisponibilidad(espacioId, fechaInicio, fechaFin);
    
    // Crear y generar QR
    Reserva reserva = new Reserva();
    reserva.setQrCode(qrCodeGenerator.generarCodigo(id, fechaInicio));
    return reserva;
}
```

---

#### **CheckInService**

**Responsabilidades:**
- Registrar entrada de vehículo
- Validar estado de reserva
- Actualizar estado a EN_USO
- Soportar entrada por QR

**Flujos:**
1. Check-in por ID de reserva
2. Check-in por código QR (lector físico)

---

#### **CheckOutService**

**Responsabilidades:**
- Registrar salida de vehículo
- Calcular tiempo total de estancia
- Actualizar estado a COMPLETADA

**Cálculo de tiempo:**
```java
Duration duracion = Duration.between(checkIn.getHoraEntrada(), LocalDateTime.now());
long minutos = duracion.toMinutes();
```

---

#### **PaseMensualService**

**Responsabilidades:**
- Crear pases con Strategy Pattern
- Calcular precio según tipo
- Gestionar vigencia (30 días)
- Renovar pases

**Uso de Strategy:**
```java
public PaseMensual crearPase(Long usuarioId, String tipo, Long espacioAsignado) {
    // Strategy Pattern calcula precio
    BigDecimal precio = precioPaseContext.calcularPrecio(tipo);
    
    pase.setPrecio(precio);
    pase.setFechaVencimiento(LocalDateTime.now().plusDays(30));
    return pase;
}
```

---

## 5. ENDPOINTS REST <a id="endpoints"></a>

### 5.1 Reservas (7 endpoints)

| Método | Endpoint | Descripción |
|--------|----------|-------------|
| POST | `/api/reservas` | Crear reserva |
| GET | `/api/reservas` | Listar todas |
| GET | `/api/reservas/{id}` | Obtener por ID |
| GET | `/api/reservas/usuario/{id}` | Por usuario |
| GET | `/api/reservas/activas` | Solo EN_USO |
| GET | `/api/reservas/disponibilidad/{id}` | Verificar disponibilidad |
| DELETE | `/api/reservas/{id}` | Cancelar |

**Ejemplo Request:**
```json
POST /api/reservas
{
  "usuarioId": 1,
  "espacioId": 10,
  "fechaInicio": "2025-12-15T10:00:00",
  "fechaFin": "2025-12-15T18:00:00"
}
```

**Ejemplo Response:**
```json
{
  "id": 1,
  "qrCode": "PARKING-1-20251215100000-a3f2c1",
  "estado": "CONFIRMADA",
  "fechaCreacion": "2025-12-08T19:00:00"
}
```

---

### 5.2 Accesos (5 endpoints)

| Método | Endpoint | Descripción |
|--------|----------|-------------|
| POST | `/api/accesos/check-in` | Registrar entrada |
| POST | `/api/accesos/check-out` | Registrar salida |
| POST | `/api/accesos/validar-qr` | Validar QR (lector) |
| GET | `/api/accesos/check-ins` | Listar entradas |
| GET | `/api/accesos/check-outs` | Listar salidas |

**Flujo Check-In:**
```json
POST /api/accesos/check-in
{
  "reservaId": 1,
  "dispositivoId": "LECTOR-PUERTA-A"
}

Response:
{
  "id": 1,
  "horaEntrada": "2025-12-15T10:05:00",
  "mensaje": "Check-in realizado exitosamente"
}
```

**Flujo Check-Out:**
```json
POST /api/accesos/check-out
{
  "reservaId": 1
}

Response:
{
  "id": 1,
  "horaSalida": "2025-12-15T18:00:00",
  "tiempoTotalMinutos": 475,
  "mensaje": "Check-out realizado. Tiempo: 7h 55min"
}
```

---

### 5.3 Pases Mensuales (7 endpoints)

| Método | Endpoint | Descripción |
|--------|----------|-------------|
| POST | `/api/pases-mensuales` | Crear pase |
| GET | `/api/pases-mensuales` | Listar todos |
| GET | `/api/pases-mensuales/{id}` | Por ID |
| GET | `/api/pases-mensuales/usuario/{id}` | Por usuario |
| GET | `/api/pases-mensuales/vigentes` | Solo vigentes |
| PUT | `/api/pases-mensuales/{id}/renovar` | Renovar (+30 días) |
| DELETE | `/api/pases-mensuales/{id}` | Cancelar |

**Tipos de Pases:**
- **BASICO**
- **PREMIUM**
- **EMPRESARIAL**

**Ejemplo:**
```json
POST /api/pases-mensuales
{
  "usuarioId": 1,
  "tipo": "PREMIUM",
  "espacioAsignado": 10
}

Response:
{
  "id": 1,
  "tipo": "PREMIUM",
  "precio": 300.00,
  "fechaVencimiento": "2026-01-08T19:00:00",
  "vigente": true
}
```

---

## 6. SISTEMA QR <a id="sistema-qr"></a>

### 6.1 Formato del Código QR

```
PARKING-{id}-{timestamp}-{hash}
```

**Ejemplo:**
```
PARKING-1-20251215100000-a3f2c1
```

**Componentes:**
- `PARKING`: Prefijo fijo
- `1`: ID de reserva
- `20251215100000`: Timestamp (yyyyMMddHHmmss)
- `a3f2c1`: Hash de seguridad (6 caracteres)

---

### 6.2 Generación del Hash

```java
public String generarCodigo(Long reservaId, LocalDateTime fecha) {
    String timestamp = fecha.format(DateTimeFormatter.ofPattern("yyyyMMddHHmmss"));
    String data = reservaId + timestamp + SECRET_KEY;
    String hash = SHA256(data).substring(0, 6);
    
    return "PARKING-" + reservaId + "-" + timestamp + "-" + hash;
}
```

**Seguridad:**
- Hash SHA-256 para evitar QR falsificados
- Timestamp único por reserva
- Validación de formato en backend

---

### 6.3 Flujo de Validación QR

```
1. Usuario crea reserva
   ↓
2. Backend genera: "PARKING-1-20251215100000-a3f2c1"
   ↓
3. Frontend genera imagen QR del código
   ↓
4. Usuario llega al parking
   ↓
5. Lector escanea QR → lee código
   ↓
6. POST /api/accesos/validar-qr {"codigoQR": "..."}
   ↓
7. Backend valida:
   - Formato correcto
   - Hash válido
   - Reserva existe
   - Estado CONFIRMADA
   ↓
8. Response: {"valido": true, "accion": "ABRIR_BARRERA"}
   ↓
9. Barrera se abre automáticamente
```

**Endpoint de Validación:**
```json
POST /api/accesos/validar-qr
{
  "codigoQR": "PARKING-1-20251215100000-a3f2c1",
  "dispositivoId": "LECTOR-001"
}

Response (válido):
{
  "valido": true,
  "accion": "ABRIR_BARRERA",
  "mensaje": "Acceso permitido",
  "reservaId": 1,
  "checkInId": 1
}

Response (inválido):
{
  "valido": false,
  "accion": "DENEGAR_ACCESO",
  "mensaje": "Código QR inválido",
  "razon": "Hash incorrecto"
}
```

---

## 7. VALIDACIONES <a id="validaciones"></a>

### 7.1 Validaciones de Reserva

| Validación | Regla | Error |
|------------|-------|-------|
| **Fechas** | inicio < fin | "Fecha inicio debe ser anterior" |
| **Anticipación** | inicio > now + 1h | "Mínimo 1h de anticipación" |
| **Duración** | (fin - inicio) ≤ 12h | "Máximo 12h de duración" |
| **Disponibilidad** | Sin solapamiento | "Espacio no disponible" |

### 7.2 Detección de Conflictos

```java
private boolean hayConflictoHorario(Reserva r, LocalDateTime inicio, LocalDateTime fin) {
    // Conflicto si:
    // - Nueva empieza antes que existente termine
    // - Nueva termina después que existente empiece
    
    boolean terminaAntes = fin.isBefore(r.getFechaInicio());
    boolean empiezaDespues = inicio.isAfter(r.getFechaFin());
    
    return !(terminaAntes || empiezaDespues);
}
```

---

### 7.3 Validaciones de Check-In

- Reserva debe existir
- Estado debe ser CONFIRMADA
- No puede haber check-in previo

### 7.4 Validaciones de Check-Out

- Reserva debe existir
- Estado debe ser EN_USO
- Debe existir check-in previo

---

## 8. INSTALACIÓN Y USO <a id="instalacion"></a>

### 8.1 Requisitos

- Java 17+
- Gradle 8.x
- Puerto 8080 disponible

### 8.2 Dependencias

```gradle
dependencies {
    implementation 'org.springframework.boot:spring-boot-starter-web'
    implementation 'org.springframework.boot:spring-boot-starter-data-jpa'
    runtimeOnly 'com.h2database:h2'
}
```

### 8.3 Ejecutar

```bash
# Compilar
./gradlew clean build -x test

# Ejecutar
./gradlew bootRun

# El servidor arranca en http://localhost:8080
```

### 8.4 Probar Endpoints

```bash
# Crear reserva
curl -X POST http://localhost:8080/api/reservas \
  -H "Content-Type: application/json" \
  -d '{
    "usuarioId": 1,
    "espacioId": 10,
    "fechaInicio": "2025-12-15T10:00:00",
    "fechaFin": "2025-12-15T18:00:00"
  }'

# Listar reservas
curl http://localhost:8080/api/reservas

# Crear pase premium
curl -X POST http://localhost:8080/api/pases-mensuales \
  -H "Content-Type: application/json" \
  -d '{
    "usuarioId": 1,
    "tipo": "PREMIUM",
    "espacioAsignado": 10
  }'
```

---

## 📊 RESUMEN TÉCNICO

### Patrones de Diseño
- ✅ Strategy Pattern (precios pases)
- ✅ DTO Pattern (API/domain separation)
- ✅ Mapper Pattern (conversiones)
- ✅ Service Layer Pattern (business logic)
- ✅ Dependency Injection (IoC)

### Funcionalidades
- ✅ 19 endpoints REST
- ✅ Sistema QR con seguridad
- ✅ Validaciones de negocio
- ✅ Gestión de estados
- ✅ Cálculo automático de tiempos
- ✅ 3 tipos de pases mensuales


---

## 🎯 CONCLUSIÓN

Este módulo implementa un sistema completo y profesional de gestión de reservas de estacionamiento con:
- Arquitectura en capas bien definida
- Patrones de diseño aplicados correctamente
- Código limpio y mantenible
- Validaciones robustas
- Sistema QR seguro y funcional
- API REST completa y documentada
