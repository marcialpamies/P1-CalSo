# Práctica 1: Revisiones estáticas de código con SonarCloud, SonarQube y GitHub

## Índice

- [0. Requisitos previos](#0-requisitos-previos)
- [1. Objetivo](#1-objetivo)
- [2. Introducción a las revisiones estáticas de código](#2-introducción-a-las-revisiones-estáticas-de-código)
- [3. SonarQube Cloud: concepto y creación de una cuenta gratuita](#3-sonarqube-cloud-concepto-y-creación-de-una-cuenta-gratuita)
  - [3.1. Creación de cuenta en SonarQube Cloud](#31-creación-de-cuenta-en-sonarqube-cloud)
- [4. Creación de un repositorio en GitHub para la práctica y conexión con SonarCloud](#4-creación-de-un-repositorio-en-github-para-la-práctica-y-conexión-con-sonarcloud)
  - [4.1. Crear repositorio vacío en GitHub](#41-crear-repositorio-vacío-en-github)
  - [4.2. Crear un proyecto de análisis y asociarlo a un repositorio de GitHub desde SonarCloud](#42-crear-un-proyecto-de-análisis-y-asociarlo-a-un-repositorio-de-github-desde-sonarcloud)
  - [4.3. Configurar el proyecto de SonarCloud y vincularlo con las acciones del repositorio de GitHub](#43-configurar-el-proyecto-de-sonarcloud-y-vincularlo-con-las-acciones-del-repositorio-de-github)
- [5. Creación de un proyecto Maven en Eclipse](#5-creación-de-un-proyecto-maven-en-eclipse)
- [6. Plugin de SonarQube para Eclipse](#6-plugin-de-sonarqube-para-eclipse)
  - [6.1. Instalación del plugin](#61-instalación-del-plugin)
  - [6.2. Conexión con SonarCloud](#62-conexión-con-sonarcloud)
- [7. Forma de trabajo](#7-forma-de-trabajo)
- [8. Ejercicios a realizar](#8-ejercicios-a-realizar)
  - [1. Configuración inicial](#1-configuración-inicial)
  - [2. Integración en Eclipse](#2-integración-en-eclipse)
  - [3. Gestión de ramas](#3-gestión-de-ramas)
  - [4. Resolución de disconformidades](#4-resolución-de-disconformidades)
  - [5. Documentación de las correcciones](#5-documentación-de-las-correcciones)
  - [6. Integración final](#6-integración-final)
- [9. Entregables](#9-entregables)
- [10. Evaluación](#10-evaluación)
- [11. FAQ: errores comunes](#11-FAQ-errores-comunes)

## 0. Requisitos previos
Antes de comenzar la práctica, cada alumno debe tener instalado y configurado en su equipo:

  - Java 17 (JDK) o posterior.
  - Eclipse IDE versión 2021-12 o posterior.
  - Maven (se puede comprobar con mvn -v).
  - Git (se puede comprobar con git --version).
  - Cuenta en GitHub.

Además, es recomendable configurar Git con nombre y correo global (si trabajas en tu propia máquina):

```
git config --global user.name "TuNombre"
git config --global user.email "tuemail@dominio.com"

```
⚠️ **Importante si trabajas en un laboratorio compartido:**
Si usas un ordenador al que acceden más personas, **no uses la opción `--global`** al configurar Git, ya que el nombre y el correo quedarán guardados para todos los usuarios de ese equipo.

En su lugar, configura Git solo para tu repositorio local:

```bash
git config user.name "TuNombre"
git config user.email "tuemail@dominio.com"
```
Esto guardará los datos de usuario solo en ese repositorio.
Cuando termines la sesión, recuerda cerrar tu sesión de GitHub en el navegador y borrar cualquier credencial guardada en el sistema para evitar que otros usen tu identidad.

**Gestión de credenciales en equipos compartidos**

Cuando uses un ordenador compartido, es fundamental borrar las credenciales de GitHub al terminar tu sesión para evitar que otros puedan hacer push en tu nombre.

  - **Windows**

    1. Abre el menú de inicio y busca Administrador de credenciales.
    2. Entra en Credenciales de Windows.
    3. Busca entradas relacionadas con git: o github.com.
    4. Selecciónalas y pulsa Quitar.

  - **macOS**
    1. Abre la aplicación Acceso a llaveros (Keychain Access).
    2. Busca github.com.
    3. Selecciona las credenciales almacenadas y bórralas.

  - **Linux**
    - Si usas el helper de credenciales de Git (cache o store), puedes limpiar con:
    ```bash
    git credential-cache exit
    git credential-cache --timeout=1
    git credential reject
    ```
    - Si usas ~/.git-credentials (modo store), edita o borra ese fichero:
    ```bash
    nano ~/.git-credentials
    # elimina la línea con https://usuario:token@github.com
    rm ~/.git-credentials  # si quieres borrarlo completo
    ```
**En todos los casos**

  - Cierra la sesión en https://github.com desde el navegador.
  - Si usaste un **token personal (PAT)**, recuerda que puedes revocarlo desde tu perfil de GitHub: *Settings > Developer settings > Personal access tokens*.

## 1. Objetivo
Introducir el uso de herramientas de análisis estático de código y su integración en el flujo de desarrollo de software, utilizando SonarQube Cloud como plataforma principal.

## 2. Introducción a las revisiones estáticas de código
Las **revisiones estáticas de código** consisten en analizar el software **sin ejecutarlo**, con el objetivo de detectar defectos, malas prácticas o riesgos de seguridad. Este tipo de análisis forma parte del proceso de **aseguramiento de la calidad** y permite localizar problemas en fases tempranas del desarrollo.

Entre los beneficios de la revisión estática destacan:
- Identificación de **bugs potenciales** (uso incorrecto de variables, estructuras incompletas, excepciones no tratadas).
- Control de **estilo y convenciones de codificación** (nombres de variables, estructura de clases, redundancias).
- Detección de **código duplicado** o mal estructurado.
- **Seguridad**: vulnerabilidades comunes como inyecciones o uso inseguro de librerías.
- Mejora de la **mantenibilidad** y reducción de la **deuda técnica**.

La automatización de este proceso es posible gracias a herramientas como **SonarQube**, que integran motores de análisis estático con la posibilidad de establecer métricas de calidad y gates (umbrales mínimos que el código debe cumplir antes de ser aceptado).

En esta práctica llamaremos **disconformidades** a los issues (problemas) detectados por SonarCloud: bugs, vulnerabilidades o code smells.

---

## 3. SonarQube Cloud: concepto y creación de una cuenta gratuita
**SonarQube Cloud** es la versión en la nube del servidor SonarQube. Permite:
- Analizar proyectos directamente conectados a repositorios de GitHub, GitLab, Azure DevOps o Bitbucket. En la versión gratuita con limitaciones
- Definir **Quality Profiles** (conjuntos de reglas activas) y **Quality Gates** (criterios de aceptación). En la versión gratuita se pueden definir pero el uso de los perfiles y criterios modificados solo está permitido si se adopta la versión de pago.
- Generar paneles de control con métricas de calidad, seguridad y cobertura de tests. Incluido en la versión gratuita.
- Integrar resultados de los análisis en los flujos de integración continua. Incluido en la versión gratuita con limitaciones

### 3.1. Creación de cuenta en SonarQube Cloud
1. Si no se dispone de una, crearemos una cuenta de **GitHub**, usando la cuenta de correo del alumno que ejerce como coordinador del grupo. Accedemos a la cuenta de github. 
2. Con la cuenta de GitHub abieta se accede a [https://sonarcloud.io/login](https://sonarcloud.io/login).  
3. Inicia sesión usando tu cuenta de **GitHub**.  
4. Autoriza a SonarCloud a acceder a tus repositorios. Durante este procedimiento importaremos una organización que coincidirá con el nombre del usuario propietario de la cuenta de GitHub utilizada.

![Proceso de creación de cuenta en SonarCloud](imagenes/01_practica_01.png)

---

## 4. Creación de un repositorio en GitHub para la práctica y conexión con SonarCloud
Para automatizar el análisis en cada interacción con **GitHub**, necesitamos vincular nuestro repositorio en GitHub con un proyecto de análisis en SonarCloud.

### 4.1. Crear repositorio vacío en GitHub
1. Inicia sesión en [https://github.com](https://github.com).  
2. Clic en **New repository**.  
3. Define el nombre (ej. `CalSo2526-grupoXX`, donde XX representará el número de grupo).
4. Marca el repositorio como **Privado**  
5. No marques la opción de inicializar con README para evitar conflictos iniciales ni crees un fichero .gitignore.  
6. Copia la URL del repositorio (HTTPS).

![Creación de un nuevo repositorio en GitHub](imagenes/02_practica_01.png)

### 4.2. Crear un proyecto de análisis y asociarlo a un repositorio de GitHub desde SonarCloud
1. En SonarCloud, dentro de tu organización, selecciona **Analyze new project**.  
2. Escoge el repositorio creado en GitHub y completa los pasos del proceso en las páginas sucesivas.  

![Creación de un proyecto de análisis de SonarCloud](imagenes/03_practica_01.png)


### 4.3. Configurar el proyecto de SonarCloud y vincularlo con las acciones del repositorio de GitHub
1. Configurar el proyecto utilizando las acciones de GitHub. Este procedimiento proporcionará un **SONAR_TOKEN** para autenticar los análisis de las acciones.  
2. En el repositorio de GitHub:
   - Ve a **Settings > Secrets and variables > Actions**.
   - Crea un secreto llamado `SONAR_TOKEN` con el valor generado en el paso 1.
5. Añade un workflow de GitHub Actions:
   - Crear una carpeta que contenga el proyecto local (en nuestro ordenador)
   - Iniciar git en la carpeta local del proyecto y crear el archivo `build.yml` en la carpeta del proyecto local `.github/workflows`:
   ```shell
   cd RUTA_CARPETA_LOCAL_PROYECTO
   git init
   mkdir .github
   mkdir .github/workflows
   touch .github/workflows/build.yml
   ```
   - Copiar, en el archivo `.github/workflows/build.yml`, el contenido indicado, para un proyecto Maven, en el paso 2 de la configuración de SonarCloud (ver imagen siguiente). Un ***workflow*** es un fichero YAML de **GitHub Actions** que define qué pasos se ejecutan automáticamente cuando haces un push o pull request.
   - **IMPORTANTE:** Si en nuestro repositorio tenemos el proyecto maven (archivo pom.xml) dentro de una carpeta contenida en el repositorio (el archivo `pom.xml` no está en el directorio raiz de nuestro repositorio), por ejemplo en la carpeta `carpeta-poroy`, tendremos que añadir a la ejecución de maven la siguiente opción: `-f carpeta-proy/pom.xml`con el fin de que la acción encuentre el archivo del proyecto.

  ![Configuración de un workflow en GitHub Actions](imagenes/04_practica_01.png)

  - Realizar el primer commit a nuestro repositorio remoto:
   ```shell
   cd RUTA_CARPETA_LOCAL_PROYECTO
   git config user.name "USERNAME" //Si no se tiene configurado.
   git config user.email "USER@EMAIL" //Si no se tiene configurado.
   git remote add origin https://github.com/... // Sustituir por la dirección de nuestro repositorio.
   git add .
   git commit -m "Commit Inicial"
   git branch -M main
   git push -u origin main //Se solicitará el nombre de usuario y el token (clásico) de desarrollo de github.
   ```
   [Crear un token (clásico) de desarrollo de github](https://docs.github.com/es/authentication/keeping-your-account-and-data-secure/managing-your-personal-access-tokens#creating-a-personal-access-token-classic)

---

## 5. Creación de un proyecto Maven en Eclipse
- Descarga en la carpeta de nuetro proyecto se encuentra en la subcarpeta de este repositorio nombrada como `p1-calso`. Esta carpeta contiene los archivos de proyecto Maven con el que trabajeremos en la práctica.
- Crea, en la carpeta de tu repositorio local (no en la del proyecto Maven), el fichero `.gitignore` con el siguiente contenido:
```
### Eclipse ###
.metadata
bin/
tmp/
*.tmp
*.bak
*.swp
*~.nib
local.properties
.settings/
.loadpath
.recommenders

# External tool builders
.externalToolBuilders/

# Locally stored "Eclipse launch configurations"
*.launch


# CDT- autotools
.autotools

# Java annotation processor (APT)
.factorypath

# PDT-specific (PHP Development Tools)
.buildpath

# sbteclipse plugin
.target

# Tern plugin
.tern-project

# TeXlipse plugin
.texlipse

# STS (Spring Tool Suite)
.springBeans

# Code Recommenders
.recommenders/

# Annotation Processing
.apt_generated/
.apt_generated_test/

# Scala IDE specific (Scala & Java development for Eclipse)
.cache-main
.scala_dependencies
.worksheet

### Eclipse Patch ###
# Spring Boot Tooling
.sts4-cache/

### Java ###
# Compiled class file
*.class

# Log file
*.log

# BlueJ files
*.ctxt

# Mobile Tools for Java (J2ME)
.mtj.tmp/

# Package Files #
*.jar
*.war
*.nar
*.ear
*.zip
*.tar.gz
*.rar

# virtual machine crash logs, see http://www.java.com/en/download/help/error_hotspot.xml
hs_err_pid*
replay_pid*

### Linux ###
*~

# temporary files which can be created if a process still has a handle open of a deleted file
.fuse_hidden*

# KDE directory preferences
.directory

# Linux trash folder which might appear on any partition or disk
.Trash-*

# .nfs files are created when an open file is removed but is still being accessed
.nfs*

### macOS ###
# General
.DS_Store
.AppleDouble
.LSOverride

# Icon must end with two \r
Icon


# Thumbnails
._*

# Files that might appear in the root of a volume
.DocumentRevisions-V100
.fseventsd
.Spotlight-V100
.TemporaryItems
.Trashes
.VolumeIcon.icns
.com.apple.timemachine.donotpresent

# Directories potentially created on remote AFP share
.AppleDB
.AppleDesktop
Network Trash Folder
Temporary Items
.apdisk

### macOS Patch ###
# iCloud generated files
*.icloud

### Maven ###
target/
pom.xml.tag
pom.xml.releaseBackup
pom.xml.versionsBackup
pom.xml.next
release.properties
dependency-reduced-pom.xml
buildNumber.properties
.mvn/timing.properties
# https://github.com/takari/maven-wrapper#usage-without-binary-jar
.mvn/wrapper/maven-wrapper.jar

# Eclipse m2e generated files
# Eclipse Core
.project
# JDT-specific (Eclipse Java Development Tools)
.classpath

### Windows ###
# Windows thumbnail cache files
Thumbs.db
Thumbs.db:encryptable
ehthumbs.db
ehthumbs_vista.db

# Dump file
*.stackdump

# Folder config file
[Dd]esktop.ini

# Recycle Bin used on file shares
$RECYCLE.BIN/

# Windows Installer files
*.cab
*.msi
*.msix
*.msm
*.msp

# Windows shortcuts
*.lnk
```
- Abre el IDE de Eclipse y selecciona como workspace la carpeta de tu repositorio local (la carpeta que contiene `p1-calso`)
- Importa el proyecto Maven que se encuentra en la carpeta `p1-calso`

---

## 6. Plugin de SonarQube para Eclipse
Para trabajar en local con las **mismas reglas y configuraciones** que tengamos en SonarCloud, se utiliza el plugin oficial **SonarQube for IDE** (antes conocido como SonarLint).

### 6.1. Instalación del plugin
- En Eclipse: **Help > Eclipse Marketplace…**.  
- Buscar “SonarQube” e instalar **SonarQube for IDE**.  
- Reiniciar Eclipse.

### 6.2. Conexión con SonarCloud
1. En **Eclipse**, Ir a *Window > Preferences > SonarQube > Connected Mode*.  
2. Crear una nueva conexión con **SonarCloud**.  
3. Autenticarse con un token personal de SonarCloud.  
4. Enlazar (bind) el proyecto local de Eclipse con su proyecto correspondiente en SonarCloud.  

De este modo, Eclipse descarga el **Quality Profile** activo en SonarCloud y lo aplica a los análisis locales.  Al editar un fichero y guardar, los **issues** aparecen en la vista de SonarQube del IDE.

---
## 7. Forma de trabajo

Cada miembro del grupo trabajará sobre su propia rama del repositorio de trabajo del grupo. 

```mermaid
flowchart TD
    A[main]
    B[rama_user_A]
    C[rama_user_B]
    Bc[Commits A]
    Cc[Commits B]
    Bp[Push rama_A]
    Cp[Push rama_B]
    PRa[Pull Request A]
    PRb[Pull Request B]
    M[Merge a main]

    A --> B
    A --> C
    B --> Bc --> Bp --> PRa --> M
    C --> Cc --> Cp --> PRb --> M
```

Tened en cuenta que para ejecutar el workflow al hacer `push` o `pull request` en cada rama (por defecto sólo estará en la rama `main`) hay que indicar en el build.yml que así lo queremos. En el siguiente código mínimo se indica la forma:

```
on:
  push:
    branches:
      - main
      - rama_user_A
      - rama_user_B
  pull_request:
    types: [opened, synchronize, reopened]
    branches:
      - main
      - rama_user_A
      - rama_user_B
```

En función del análisis inicial del código el grupo de trabajo se repartirá el código de análisis y se procederá a el análisis de la resolución de disconformidades y su documentación, que se incorporará en un fichero `.md`. 

Cada participante, cuando realice las acciones para resolver una disconformidad realizará un commit a su repositorio local y al concluir cada sesión de trabajo realizarán un push a su rama. Cuando ambos miembros concluiyan con la resolución de todas las disconformidades que aparecen en el código procederán a establecer los `pull request` necesarios para unificar todos las acciones en la rama `main` donde se configurará la entrega final.

IMPORTANTE: para la entrega de la práctica se deben haber resuelto la totalidad de las disconformidades y haber realizado la documentación de la disconformidad encontrada, su descripción, su localización en el código original y las modificaciones realizadas para su solución (No está permitida la eliminación de funcionalidad para la corrección de disconformidades). Toda esta documentación de organizará en uno o varios archivos `.md`  de la rama `main` que se organizarán según las clases del proyecto original.

## 8. Ejercicios a realizar

Para consolidar los conocimientos de la práctica, cada grupo deberá completar los siguientes ejercicios:

### 1. Configuración inicial

- Crear la cuenta en GitHub y SonarCloud, vinculando una organización y un repositorio privado del grupo.
- Configurar el build.yml en la carpeta .github/workflows del repositorio para ejecutar el análisis de SonarCloud sobre el proyecto Maven proporcionado.
- Verificar que el análisis se ejecuta correctamente al realizar un primer push a la rama main.
- En el archivo `README.md`de la rama `main` deberá constar el nombre y correo electrónico de todos los miembros del grupo.
- Incorporar al profesor como usuario con los permisos adecuados pen el repositorio y en la organización de SonarCloud (La cuenta de usuario que hay que añadir aparece en el enunciado de la tarea en el Aula Virtual)

### 2. Integración en Eclipse

- Importar el proyecto Maven (p1-calso) en Eclipse seleccionando como workspace la carpeta local del repositorio.
- Instalar y configurar el plugin SonarQube for IDE, vinculando el proyecto local con el proyecto remoto de SonarCloud.
- Realizar un análisis local con SonarQube for IDE y comprobar que los issues detectados coinciden con los mostrados en la interfaz de SonarCloud.

### 3. Gestión de ramas

- Crear una rama individual por cada miembro del grupo (ej. rama_user_A, rama_user_B).
- Configurar el build.yml para que también ejecute análisis en esas ramas.
- Cada miembro trabajará en su rama para resolver un subconjunto de disconformidades detectadas en el análisis inicial.

### 4. Resolución de disconformidades

- Seleccionar varias disconformidades detectadas por SonarCloud (bugs, code smells, vulnerabilidades).
- Modificar el código en Eclipse para resolverlas, asegurando que no se elimina funcionalidad.
- Realizar commit y push a la rama personal, verificando que SonarCloud muestra la disminución de disconformidades en los análisis posteriores.

### 5. Documentación de las correcciones

- Crear un archivo Markdown (docs/disconformidades.md o varios organizados por clase) en el repositorio donde se documente cada disconformidad:
   - Localización: archivo y línea donde aparece.
   - Descripción: texto del issue detectado.
   - Modificación aplicada: cambios realizados en el código para resolverla.
- Cada miembro debe documentar las disconformidades que haya corregido.

### 6. Integración final

- Abrir pull requests desde las ramas individuales hacia main.
- Resolver posibles conflictos y fusionar los cambios cuando el análisis en SonarCloud sea satisfactorio.
- Verificar en la rama main que se han resuelto todas las disconformidades detectadas originalmente.
- Asegurar que los ficheros .md de documentación estén completos y actualizados en la rama main.

## 9. Entregables
Al entregar la tarea en el aula virtual se indicará el enlace al repositorio de GitHub que contendrá, en la rama main: el proyecto completo, la documentación solicitada en el archivo `.md`, la identificación de los miembros del grupo en el archivo `README.md` y, en las ramas personales de cada participante la información sobre los commits/PR realizados por cada participante junto con su versión de la documentación.

**IMPORTANTE:** El profesor debe tener acceso tanto al repositorio como a la organización en la que se configuren los proyectos SonarQube con la cuenta de usuario indicada en el enunciado de la tarea asociada a la práctica en el aula virtual.

**Fecha de entrega máxima:** 10/11/2025

## 10. Evaluación

**IMPORTANTE:** Si el profesor no pudiera acceder al repositorio o al proyecto de SonarCloud desde la cuenta indicada en el enunciado de la tarea asociada a la práctica en el aula virtual, se considerará la práctica como no superada con puntuación 0 puntos.

La práctica se considerará superada si se cumplen los siguientes criterios:

#### 1. Configuración y entorno (2 puntos)
- [ ] Se ha creado correctamente el repositorio privado en GitHub.
- [ ] Se ha creado correctamente la organización en SonarCloud vinculada con GitHub.  
- [ ] Se ha añadido al profesor en el repositorio de GitHub y en la organización de SonarCloud con la cuenta indicada en la tarea del aula virtual asociada a la práctica. 
- [ ] El workflow de GitHub Actions (`build.yml`) ejecuta los análisis automáticamente al hacer *push* en las ramas configuradas.  

#### 2. Análisis estático (3 puntos)
- [ ] Se han detectado y documentado las disconformidades iniciales en el código.  
- [ ] El análisis de SonarCloud muestra métricas de *bugs*, vulnerabilidades y *code smells*.  

#### 3. Resolución de disconformidades (3 puntos)
- [ ] Cada miembro del grupo ha trabajado en su rama individual.
- [ ] El trabajo de los miembros del grupo se ha equilibrado de forma que sus miembros han realizado un trabajo equitativo.   
- [ ] Se han realizado modificaciones en el código resolviendo las disconformidades sin eliminar funcionalidad.  
- [ ] Los *pull requests* se han integrado correctamente en `main`, reduciendo a cero las disconformidades reportadas.  

#### 4. Documentación (2 puntos)
- [ ] Se han creado archivos `.md` con la documentación de cada disconformidad.  
- [ ] Cada documento incluye: localización, descripción y modificación aplicada.  
- [ ] La documentación final está organizada y accesible en la rama `main`.  

## 11. FAQ: errores comunes

- `mvn`: command not found → Maven no está instalado o no está en el PATH.
- POM file not found → Asegúrate de usar `-f carpeta/pom.xml` si tu proyecto está en subcarpeta.
- **Error** refname `refs/heads/master not found` **al renombrar rama** → tu rama inicial ya se llama `main`. No necesitas renombrar.
- **Error 403 en GitHub Actions al hacer commit automático** → añade un *Personal Access Token (PAT)* o revisa permisos de `GITHUB_TOKEN`.
- **Conflictos al hacer** `git pull` → abre los archivos marcados con `<<<<<<<`, elige qué cambios conservar, guarda, `git add`, y ejecuta `git rebase --continue`.
