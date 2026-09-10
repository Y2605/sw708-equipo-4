# Entorno de desarrollo — Semana 01

Cada integrante instala y verifica su propio entorno. Los pasos 1, 3 y 5 son por
máquina; sus capturas van en `capturas/` con el apellido en el nombre del archivo.

---

## Integrante: Jhostin Leonardo Rodríguez Neyra — 20231145I

Máquina: Windows 11 Home Single Language 24H2 · terminal PowerShell.

### Paso 1 — JDK

```
> java -version
java version "24.0.1" 2025-04-15
Java(TM) SE Runtime Environment (build 24.0.1+9-30)
Java HotSpot(TM) 64-Bit Server VM (build 24.0.1+9-30, mixed mode, sharing)

> javac -version
javac 24.0.1
```

`java` y `javac` responden con la misma versión → es un **JDK**, no un JRE.
La guía pide 17 o superior; 24.0.1 cumple. (La recomendación oficial era JDK 21 LTS;
la máquina ya tenía el JDK 24 de Oracle y PetClinic construye igual.)

Captura: `capturas/p1-jdk-rodriguez.png`

### Paso 3 — Maven

```
> mvn -v
Apache Maven 3.9.9 (8e8579a9e76f7d015ee5ec7bfcdc97d260186937)
Maven home: ...\apache-maven-3.9.9
Java version: 24.0.1, vendor: Oracle Corporation, runtime: C:\Program Files\Java\jdk-24
Default locale: es_PE, platform encoding: UTF-8
OS name: "windows 11", version: "10.0", arch: "amd64", family: "windows"
```

La línea `Java version: 24.0.1` confirma que Maven usa el JDK del paso 1.
(Se usó la distribución binaria portable `apache-maven-3.9.9`; el warning
"restricted method ... jansi" que imprime Maven con JDK 24 es informativo y no afecta el build.)

Captura: `capturas/p3-maven-rodriguez.png`

### Paso 4 — IDE

**Visual Studio Code** con el *Extension Pack for Java* de Microsoft. Abre la carpeta
`spring-petclinic` y la reconoce como proyecto Maven (vista *Java Projects*, resolución
de dependencias). Comandos usados en el paso 8: ir a la definición (F12 / Ctrl+clic),
buscar referencias (Shift+F12), esquema del archivo (Ctrl+Shift+O).

### Paso 5 — Construcción de PetClinic

Clonado **fuera** de la carpeta de la entrega (es código ajeno) y construido con `mvnw.cmd`:

```
> git clone https://github.com/spring-projects/spring-petclinic.git
> cd spring-petclinic
> .\mvnw.cmd clean package

[INFO] Results:
[INFO] Tests run: 74, Failures: 0, Errors: 0, Skipped: 2
[INFO]
[INFO] BUILD SUCCESS
[INFO] Total time:  40.245 s
[INFO] Finished at: 2026-09-09T18:10:23-05:00
```

Queda `target/spring-petclinic-4.0.0-SNAPSHOT.jar`.
(El número de pruebas —74— difiere de las ~53 de la guía porque esta rama de
PetClinic ya va sobre Spring Boot 4; lo que no varía: `BUILD SUCCESS` y cero fallos.)

Captura: `capturas/p5-build-rodriguez.png` · captura del IDE del equipo: `capturas/p5-ide.png`
Log completo del build: `capturas/build-log-rodriguez.txt`

### Paso 6 — App levantada y usada

```
> java -jar target\spring-petclinic-4.0.0-SNAPSHOT.jar
...
Started PetClinicApplication in 8.505 seconds (process running for 9.101)
```

Servidor en `http://localhost:8080`. Uso como recepcionista de la clínica:

- **Find owners** (apellido vacío) → lista completa de dueños.
- Ficha del dueño 1 (George Franklin, Madison) → `http://localhost:8080/owners/1`.
- Se registró una **visita** a la mascota "Leo": fecha `2026-12-01`, descripción
  "Vacuna antirrábica anual" (`POST /owners/1/pets/1/visits/new` → 302; la visita
  aparece luego en la ficha).
- Lista de veterinarios: `http://localhost:8080/vets.html` (6 vets; especialidades
  radiology / surgery / dentistry).

Captura: `capturas/p6-owner-1-rodriguez.png`.
Respuesta HTML cruda del servidor para esa pantalla: `capturas/owners-1-response.html`
(HTTP 200; se ve `George Franklin`, `Madison`, la mascota `Leo` y la visita del `2026-12-01`).

> La `p6-owner-1-rodriguez.png` se tomó con Chrome en modo headless, así que no muestra
> la barra de direcciones. La guía la pide con la URL visible → conviene reemplazarla por
> una captura normal del navegador con `http://localhost:8080/owners/1` a la vista.

---

## Integrante: _(apellido: Fernández — completar nombre y código)_

Capturas ya subidas: `capturas/p1-jdk-ferdandez.png`, `capturas/p3-maven-fernandez.png`,
`capturas/p5-build-fernandez.png`.
Falta: pegar aquí las salidas de `java -version` / `javac -version`, `mvn -v` y las líneas
`Tests run` / `BUILD SUCCESS` / `Total time` de tu build.

## Integrante 3 — _(completar)_

- Paso 1 (JDK): `java -version` / `javac -version` → _pegar salida_ · captura `capturas/p1-jdk-APELLIDO.png`
- Paso 3 (Maven): `mvn -v` → _pegar salida_ · captura `capturas/p3-maven-APELLIDO.png`
- Paso 5 (build): líneas `Tests run` / `BUILD SUCCESS` / `Total time` → _pegar_ · captura `capturas/p5-build-APELLIDO.png`

## Integrante 4 — _(completar)_

- Paso 1 (JDK): … · `capturas/p1-jdk-APELLIDO.png`
- Paso 3 (Maven): … · `capturas/p3-maven-APELLIDO.png`
- Paso 5 (build): … · `capturas/p5-build-APELLIDO.png`

---

## Paso 2 — Git y repositorio del equipo

```
> git --version
git version 2.47.0.windows.2

> git config --global user.name
Jhostin Leonardo Rodriguez Neyra
> git config --global user.email
jhostin.rodriguez.n@uni.pe
```

Repositorio del equipo: `https://github.com/Y2605/sw708-equipo-4`
Falta: captura de Settings → Collaborators con los integrantes → `capturas/p2-collaborators.png`.
