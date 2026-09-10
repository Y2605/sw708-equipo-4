# Bitácora — la IA como guía de código ajeno (Paso 10)

## ¿En qué usé la IA?

La usé como herramienta de apoyo, en cosas puntuales:

- **Orientación inicial**: una explicación general de PetClinic (qué paquetes tiene, para
  qué sirve cada uno, cómo se atiende una petición) para tener un mapa de arranque antes
  de leer el código a fondo. Es la respuesta que se verifica más abajo.
- **Conceptos de Spring/JPA que no teníamos claros**: qué hace `JpaRepository`, qué
  significan `@OneToMany(cascade = ALL)` y `fetch = EAGER`, para qué sirve un `Formatter`,
  qué es un *runtime hint*. Preguntas sueltas para entender lo que estábamos leyendo.
- **Mensajes raros de la consola**: por qué Maven con JDK 24 imprime el warning de
  `jansi`, y por qué el build daba 74 tests en vez de los ~53 de la guía (rama sobre
  Spring Boot 4).
- **Sintaxis de Mermaid** para poder escribir el diagrama del paso 9 dentro del Markdown.
- **Recordar comandos** de Git y cómo lanzar el `.jar` en PowerShell (Windows).
- **Pulir la redacción** de algunas respuestas (el contenido y las decisiones son
  nuestras; la IA solo ayudó a dejarlas más claras).

**Lo que NO hicimos con IA** (lo hicimos leyendo el repositorio en el IDE):

- La tabla de paquetes y la columna "depende de" (mirando los `import` de cada clase).
- Agrupar las 11 clases de `owner` en roles.
- El recorrido de `GET /owners/1` clase por clase con "ir a la definición".
- El diagrama de componentes (qué cajas y qué flechas son ciertas).
- La decisión de qué es *estructural* y qué es *acabado*.
- Esta verificación afirmación por afirmación.

IA consultada: asistente de IA (ChatGPT / Claude), respondiendo **sin acceso al
repositorio**. Fecha: 2026-09-09.

---

## Prompt

> Explícame cómo está organizado el código de Spring PetClinic: paquetes,
> responsabilidades y cómo se atiende la petición `GET /owners/1`.

---

## Resumen de la respuesta de la IA

La IA describió PetClinic como una app Spring Boot **en 3 capas**:

- **Capa web** en el paquete `...petclinic.web`, con `OwnerController`, `PetController`,
  `VisitController`, `VetController`.
- **Capa de servicio** en `...petclinic.service`: interfaz `ClinicService` con su
  implementación `ClinicServiceImpl`, que centraliza la lógica de negocio.
- **Capa de repositorio** en `...petclinic.repository`: `OwnerRepository`,
  `PetRepository`, `VisitRepository`, `VetRepository` (Spring Data JPA).
- **Modelo de dominio** en `...petclinic.model`: `Owner`, `Pet`, `Visit`, `Vet`,
  `Specialty`, `PetType`, más `BaseEntity` / `NamedEntity` / `Person`.
- Base de datos **H2 en memoria** por defecto, conmutable a MySQL/PostgreSQL por perfiles.
  Vistas con **Thymeleaf**. Validación con Bean Validation (`@Valid`). i18n con `MessageSource`.
- Es el proyecto de ejemplo **oficial de Spring** y se usa como benchmark; sigue los
  principios de **Domain-Driven Design**.
- Flujo de `GET /owners/1`: `OwnerController` (con `@RequestMapping("/owners")` a nivel
  de clase) → `showOwner` → `clinicService.findOwnerById(1)` → `OwnerRepository.findById(1)`
  → Hibernate → tabla `owners` → `Owner` al modelo → vista `owners/ownerDetails` con Thymeleaf.

---

## Verificación afirmación por afirmación

| # | Afirmación de la IA | Veredicto | Evidencia en el código |
|---|---|---|---|
| 1 | Los paquetes son por capa: `web`, `service`, `repository`, `model` | **Falsa** | Los paquetes son **por funcionalidad**: `model`, `owner`, `vet`, `system` + raíz. No existe `web`, `service` ni `repository`. En `owner` conviven el controlador, el repositorio y la entidad. |
| 2 | Hay una capa de servicio: `ClinicService` / `ClinicServiceImpl` | **Falsa** | No hay ninguna clase `*Service` en `src/main/java`. Los controladores usan los repositorios **directamente** (`OwnerController` recibe `OwnerRepository owners` por constructor y llama `owners.findById(...)`). Esa capa existió en PetClinic hasta ~2016 y se quitó. |
| 3 | Existen `PetRepository` y `VisitRepository` | **Falsa** | Solo hay 3 repositorios: `OwnerRepository`, `PetTypeRepository` (paquete `owner`) y `VetRepository` (paquete `vet`). `Pet` y `Visit` se guardan **en cascada por `OwnerRepository`** (`@OneToMany(cascade = ALL)` en `Owner.pets` y `Pet.visits`). |
| 4 | `OwnerController` tiene `@RequestMapping("/owners")` a nivel de clase | **Falsa** | `OwnerController` no tiene anotación de clase; cada método define su ruta (`@GetMapping("/owners/{ownerId}")`, `@GetMapping("/owners/new")`, …). El que sí tiene `@RequestMapping` de clase es `PetController` (`"/owners/{ownerId}"`). |
| 5 | El flujo pasa por `clinicService.findOwnerById(1)` | **Falsa** | En `OwnerController.showOwner` la llamada real es `this.owners.findById(ownerId)` sobre `OwnerRepository`. No hay servicio intermedio. |
| 6 | El modelo de dominio está todo en el paquete `model` | **Falsa (parcial)** | En `model` solo están las superclases `BaseEntity`, `NamedEntity`, `Person`. Las entidades concretas (`Owner`, `Pet`, `PetType`, `Visit`) están en `owner`, y (`Vet`, `Specialty`) en `vet`. |
| 7 | Usa Spring Data JPA; los repositorios son interfaces sin implementación manual | **Cierta** | `OwnerRepository extends JpaRepository<Owner, Integer>`; consultas por nombre (`findByLastNameStartingWith`) y `@Query` (`PetTypeRepository.findPetTypes`). |
| 8 | H2 en memoria por defecto, conmutable a MySQL/PostgreSQL | **Cierta** | `application.properties`: `database=h2`, `schema-locations=classpath*:db/${database}/schema.sql`. Existen `db/h2`, `db/mysql`, `db/postgres`. |
| 9 | Vistas con Thymeleaf | **Cierta** | `src/main/resources/templates/*.html` con `xmlns:th`, `th:text`, `th:replace`; `spring.thymeleaf.mode=HTML`. |
| 10 | Validación con Bean Validation (`@Valid`) + i18n con `MessageSource` | **Cierta** | `OwnerController.processCreationForm(@Valid Owner owner, ...)`; `Owner` con `@NotBlank`, `@Pattern`. `system/WebConfiguration` registra `LocaleChangeInterceptor` (`?lang=`) y `SessionLocaleResolver`. |
| 11 | Es el proyecto de ejemplo "oficial" de Spring y se usa como benchmark de rendimiento | **No verificable** | El código no dice nada de eso; es contexto externo. En el repo solo se ve que es un `spring-projects/spring-petclinic`, no su rol como benchmark. |
| 12 | La arquitectura sigue Domain-Driven Design (agregados, etc.) | **No verificable** | Interpretación. El código no declara agregados ni límites de contexto; `Owner` sí actúa como raíz que gestiona sus `Pet`/`Visit` en cascada, pero llamar a eso "DDD" es una lectura, no un hecho comprobable en el repo. |
| 13 | ~1900 líneas, 4 paquetes | **Cierta (aprox.)** | `wc -l` sobre `src/main/java` = **1878** líneas; 4 paquetes (`model`, `owner`, `vet`, `system`) + 2 clases en la raíz. |

---

## El error encontrado (el principal)

**Qué dijo la IA:** PetClinic tiene la arquitectura clásica en 3 capas, con paquetes
`web` / `service` / `repository` y una fachada `ClinicService` por la que pasan todos los
controladores antes de tocar la base de datos.

**Qué dice el código:** No existe nada de eso. La organización es **por funcionalidad**
(`owner`, `vet`, `model`, `system`): cada feature tiene junta su entidad, su controlador y
su repositorio. Y los controladores llaman **directamente** a los repositorios de Spring
Data — no hay capa de servicio.

**Por qué falla la IA:** PetClinic sí tuvo esa estructura hace ~10 años (`ClinicService`,
paquetes por capa, repos JDBC/JPA/JDO por perfil), y eso está en miles de tutoriales
antiguos. El modelo mezcla esa versión histórica con la actual. Es justo lo que advierte
el paso 10: **la IA orienta, pero cada afirmación hay que comprobarla en el código de
esta versión.**

## Errores secundarios

- `PetRepository` y `VisitRepository` (afirmación 3): no existen; `Pet` y `Visit` se
  persisten en cascada por `OwnerRepository`.
- `@RequestMapping("/owners")` de clase en `OwnerController` (afirmación 4): lo confundió
  con `PetController`.
- Dos afirmaciones (11 y 12) no son falsas ni ciertas: son contexto/opinión que **no se
  puede verificar** contra el repositorio.

---

> Nota: si se requiere el pantallazo de la conversación con la IA, pegar aquí la captura
> de haber enviado el prompt de arriba en ChatGPT / Claude.
