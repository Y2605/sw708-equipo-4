# Bitácora — la IA como guía de código ajeno

Objetivo: pedirle a una IA que explique la estructura de PetClinic y **verificar cada
afirmación contra el código que ya recorrimos** (pasos 7 y 8).

---

## Prompt

> Explícame cómo está organizado el código de Spring PetClinic: paquetes,
> responsabilidades y cómo se atiende la petición `GET /owners/1`.

IA consultada: modelo de lenguaje general, respondiendo **sin acceso al repositorio**
(de memoria / datos de entrenamiento). Fecha: 2026-09-09.

---

## Resumen de la respuesta de la IA

La IA describió PetClinic como una app Spring Boot en 3 capas:

- **Capa web** en el paquete `...petclinic.web`, con controladores como `OwnerController`,
  `PetController`, `VisitController`, `VetController`.
- **Capa de servicio** en `...petclinic.service`, con una interfaz `ClinicService` y su
  implementación `ClinicServiceImpl`, que centraliza la lógica de negocio.
- **Capa de repositorio** en `...petclinic.repository`, con `OwnerRepository`,
  `PetRepository`, `VisitRepository`, `VetRepository` (Spring Data JPA).
- **Modelo de dominio** en `...petclinic.model`: `Owner`, `Pet`, `Visit`, `Vet`,
  `Specialty`, `PetType`, más `BaseEntity` / `NamedEntity` / `Person`.
- Base de datos **H2 en memoria** por defecto, conmutable a MySQL/PostgreSQL por perfiles.
  Vistas con **Thymeleaf**. Validación con Bean Validation (`@Valid`). i18n con `MessageSource`.
- Flujo de `GET /owners/1`: `OwnerController` (con `@RequestMapping("/owners")` a nivel de
  clase) → `showOwner` → `clinicService.findOwnerById(1)` → `OwnerRepository.findById(1)` →
  Hibernate → tabla `owners` → se pone el `Owner` en el modelo → vista `owners/ownerDetails`
  con Thymeleaf.

---

## Verificación afirmación por afirmación

| # | Afirmación de la IA | Veredicto | Evidencia en el código |
|---|---|---|---|
| 1 | Los paquetes son por capa: `web`, `service`, `repository`, `model` | **Falsa** | Los paquetes son **por funcionalidad**: `model`, `owner`, `vet`, `system` + raíz. No existe `web`, `service` ni `repository`. Controlador, repositorio y entidad de dueños viven todos juntos en `owner`. |
| 2 | Hay una capa de servicio: `ClinicService` / `ClinicServiceImpl` | **Falsa** | No existe ninguna clase `*Service` en `src/main/java`. Los controladores usan los repositorios **directamente** (`OwnerController` recibe `OwnerRepository owners` por constructor y llama `owners.findById(...)`). Esa capa existía en PetClinic hasta ~2016; se eliminó hace años. |
| 3 | Existen `PetRepository` y `VisitRepository` | **Falsa** | Solo hay 3 repositorios: `OwnerRepository`, `PetTypeRepository` (paquete `owner`) y `VetRepository` (paquete `vet`). `Pet` y `Visit` se guardan **en cascada a través de `OwnerRepository`** (`@OneToMany(cascade = ALL)` en `Owner.pets` y `Pet.visits`). |
| 4 | `OwnerController` tiene `@RequestMapping("/owners")` a nivel de clase | **Falsa** | `OwnerController` **no** tiene anotación a nivel de clase; cada método define su ruta (`@GetMapping("/owners/{ownerId}")`, `@GetMapping("/owners/new")`, …). El que sí tiene `@RequestMapping` de clase es `PetController` (`"/owners/{ownerId}"`). |
| 5 | El modelo de dominio está en el paquete `model` | **Parcialmente falsa** | En `model` solo están las superclases `BaseEntity`, `NamedEntity`, `Person`. Las entidades concretas (`Owner`, `Pet`, `PetType`, `Visit`) están en `owner`, y (`Vet`, `Specialty`) en `vet`. |
| 6 | Usa Spring Data JPA; los repositorios son interfaces sin implementación manual | **Cierta** | `OwnerRepository extends JpaRepository<Owner, Integer>`; consultas por convención de nombre (`findByLastNameStartingWith`) y `@Query` (`PetTypeRepository.findPetTypes`). |
| 7 | H2 en memoria por defecto, conmutable a MySQL/PostgreSQL | **Cierta** | `application.properties`: `database=h2` y `schema-locations=classpath*:db/${database}/schema.sql`. Existen `db/h2`, `db/mysql`, `db/postgres`. |
| 8 | Vistas con Thymeleaf | **Cierta** | `src/main/resources/templates/*.html` con `xmlns:th`, `th:text`, `th:replace`; `spring.thymeleaf.mode=HTML`. |
| 9 | Validación con Bean Validation (`@Valid`) | **Cierta** | `OwnerController.processCreationForm(@Valid Owner owner, ...)`; `Owner` tiene `@NotBlank`, `@Pattern(regexp="\\d{10}")`. (Además hay validación en Java pura: `PetValidator`.) |
| 10 | i18n con `MessageSource` / cambio de idioma por URL | **Cierta** | `system/WebConfiguration` registra un `LocaleChangeInterceptor` con `paramName = "lang"` y un `SessionLocaleResolver` por defecto en inglés. |
| 11 | El flujo pasa por `clinicService.findOwnerById(1)` | **Falsa** | En `OwnerController.showOwner` la llamada real es `this.owners.findById(ownerId)` sobre `OwnerRepository`. No hay servicio intermedio. |
| 12 | ~1900 líneas, 4 paquetes | **Cierta (aprox.)** | `wc -l` sobre `src/main/java` = **1878** líneas; 4 paquetes (`model`, `owner`, `vet`, `system`) + 2 clases en la raíz. |

---

## El error encontrado (el principal)

**Qué dijo la IA:** PetClinic tiene una arquitectura clásica en 3 capas con paquetes
`web` / `service` / `repository`, y una fachada `ClinicService` por la que pasan todos
los controladores antes de tocar la base de datos.

**Qué dice el código:** No existe ninguna de esas cosas. La organización es **por
funcionalidad** (`owner`, `vet`, `model`, `system`): cada feature guarda junta su
entidad, su controlador y su repositorio. Y los controladores hablan **directamente**
con los repositorios de Spring Data — no hay capa de servicio.

**Por qué falla la IA:** PetClinic sí tuvo esa estructura hace ~10 años (`ClinicService`,
paquetes por capa, repos JDBC/JPA/JDO por perfil). El modelo mezcla esa versión histórica
—muy documentada en blogs y tutoriales antiguos— con la actual. Es exactamente el riesgo
que advierte el paso 10: **la IA orienta, pero cada afirmación hay que comprobarla en el
código de esta versión.**

## Errores secundarios

- Inventó `PetRepository` y `VisitRepository` (afirmación 3): plausible por simetría, pero
  en esta versión `Pet` y `Visit` se persisten en cascada por `OwnerRepository`.
- Puso `@RequestMapping("/owners")` a nivel de clase en `OwnerController` (afirmación 4):
  lo mezcló con `PetController`, que sí lo tiene.
