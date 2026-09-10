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
- Se registró una **visita** a la mascota "Leo": fecha `2026-12-15`, descripción
  "Vacuna antirrábica"; la visita aparece luego en la ficha.
- Lista de veterinarios: `http://localhost:8080/vets.html`.

Captura: `capturas/p6-owner.png` — ficha de George Franklin con la visita del `2026-12-15`
y la barra de direcciones visible (`localhost:8080/owners/1`).
Respuesta HTML cruda del servidor para esa pantalla: `capturas/owners-1-response.html` (HTTP 200).

---

## Integrante: Yazid Fernández — 20224085D

Máquina: Windows 11 Pro · terminal CMD/PowerShell.

### Paso 1 — JDK

```
C:\Users\yazid>java -version
java version "17.0.12" 2024-07-16 LTS
Java(TM) SE Runtime Environment (build 17.0.12+8-LTS-286)
Java HotSpot(TM) 64-Bit Server VM (build 17.0.12+8-LTS-286, mixed mode, sharing)

C:\Users\yazid>javac -version
javac 17.0.12
```

`java` y `javac` responden la misma versión, así que es un **JDK**, no un JRE. La guía
pide 17 o superior: 17.0.12 LTS es exactamente el mínimo y cumple.

Captura: `capturas/p1-jdk-fernandez.png`

### Paso 3 — Maven

```
C:\Users\yazid>mvn -v
Apache Maven 3.9.10 (5f519b97e944483d878815739f519b2eade0a91d)
Maven home: C:\Program Files\apache-maven-3.9.10
Java version: 17.0.12, vendor: Oracle Corporation, runtime: C:\Program Files\Java\jdk-17
Default locale: es_PE, platform encoding: Cp1252
OS name: "windows 11", version: "10.0", arch: "amd64", family: "windows"
```

La línea `Java version: 17.0.12` confirma que Maven usa el JDK del paso 1.

Captura: `capturas/p3-maven-fernandez.png`

### Paso 4 — IDE

**Visual Studio Code** con el *Extension Pack for Java* de Microsoft; reconoce
`spring-petclinic` como proyecto Maven. Comandos de lectura usados: ir a la definición
(F12), buscar referencias (Shift+F12), esquema del archivo (Ctrl+Shift+O).

### Paso 5 — Construcción de PetClinic

Clonado **fuera** de la carpeta de la entrega (`C:\Users\yazid\Downloads\spring-petclinic`)
y construido con el wrapper:

```
> git clone https://github.com/spring-projects/spring-petclinic.git
> cd spring-petclinic
> .\mvnw.cmd clean package

[INFO] Tests run: 74, Failures: 0, Errors: 0, Skipped: 2
[INFO] --- jacoco:0.8.15:report (report) @ spring-petclinic ---
[INFO] Analyzed bundle 'petclinic' with 22 classes
[INFO] Building jar: C:\Users\yazid\Downloads\spring-petclinic\target\spring-petclinic-4.0.0-SNAPSHOT.jar
[INFO] BUILD SUCCESS
[INFO] Total time:  07:30 min
[INFO] Finished at: 2026-09-09T17:21:14-05:00
```

Mismo resultado que el de Rodríguez (74 pruebas, 0 fallos) con distinto JDK (17 frente a
24): el build no depende de la versión concreta mientras cumpla el mínimo. El tiempo mayor
(07:30 min contra 40 s) es porque esta fue la primera construcción y Maven descargó todas
las dependencias a `~/.m2`; una segunda corrida baja de un minuto.

Captura: `capturas/p5-build-fernandez.png`

## Integrante: Franz Joe Inga Champi — 20231302G

Máquina: Windows 10 (build 10.0.19045) · terminal CMD.

### Paso 1 — JDK

```
C:\Users\admin>java -version
java version "21.0.10" 2026-01-20 LTS
Java(TM) SE Runtime Environment (build 21.0.10+8-LTS-217)
Java HotSpot(TM) 64-Bit Server VM (build 21.0.10+8-LTS-217, mixed mode, sharing)

C:\Users\admin>javac -version
javac 21.0.10
```

`java` y `javac` responden la misma versión → es un **JDK**, no un JRE. Es el JDK 21 LTS,
la versión recomendada por la guía.

Captura: `capturas/p1-jdk-inga.png`

### Paso 3 — Maven

```
C:\Users\admin>mvn -v
Apache Maven 3.9.16 (2bdd9fddda4b155ebf8000e807eb73fd829a51d5)
Maven home: C:\apache-maven\apache-maven-3.9.16
Java version: 21.0.10, vendor: Oracle Corporation, runtime: C:\Program Files\Java\jdk-21.0.10
Default locale: es_PE, platform encoding: UTF-8
OS name: "windows 10", version: "10.0", arch: "amd64", family: "windows"
```

La línea `Java version: 21.0.10` confirma que Maven usa el JDK del paso 1.

Captura: `capturas/p3-maven-inga.png`

### Paso 4 — IDE

**Visual Studio Code** con el *Extension Pack for Java* de Microsoft; reconoce
`spring-petclinic` como proyecto Maven.

### Paso 5 — Construcción de PetClinic

Clonado **fuera** de la carpeta de la entrega
(`C:\26-ii\SW708U Arquitectura de Soluciones de Software\PetClinic\spring-petclinic`)
y construido con el wrapper:

```
> .\mvnw.cmd clean package

[INFO] Tests run: 74, Failures: 0, Errors: 0, Skipped: 2
[INFO] Analyzed bundle 'petclinic' with 22 classes
[INFO] Building jar: ...\PetClinic\spring-petclinic\target\spring-petclinic-4.0.0-SNAPSHOT.jar
[INFO] BUILD SUCCESS
[INFO] Total time:  02:58 min
[INFO] Finished at: 2026-09-09T23:05:05-05:00
```

Mismo resultado que Rodríguez y Fernández (74 pruebas, 0 fallos) con JDK 21. El tiempo
(02:58 min) corresponde a la primera construcción, con descarga de dependencias.

Captura: `capturas/p5-build-inga.png`

---

## Paso 2 — Git y repositorio del equipo

Jhostin Rodríguez:

```
> git --version
git version 2.47.0.windows.2

> git config --global user.name
Jhostin Leonardo Rodriguez Neyra
> git config --global user.email
jhostin.rodriguez.n@uni.pe
```

Yazid Fernández:

```
> git --version
git version 2.48.1.windows.1

> git config --global user.name
Yazid Fernandez
> git config --global user.email
yazid.fernandez.d@uni.pe
```

> Nota: los commits del 2026-09-09 salieron con el correo mal escrito
> (`yazid.ferdandez.d@uni.pe`). Ya está corregido en `git config`; los commits
> anteriores quedan así para no reescribir la historia del repositorio compartido.

Franz Inga:

```
> git config --global user.name
Franz Inga
> git config --global user.email
franz.inga.c@uni.pe
```

Repositorio del equipo: `https://github.com/Y2605/sw708-equipo-4` (propietario: Y2605).
Los 3 integrantes están agregados como colaboradores con permiso de escritura.
Captura: `capturas/p2-collaborators.png`.
