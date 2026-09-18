# Laboratorio: Gestión de proyectos con Git y GitHub - Desarrollo de Software <!-- omit in toc -->

Laboratorio para el curso Desarrollo de Software acerca de buenas prácticas en gestión de proyectos con Git y GitHub

- [0. Introducción](#0-introducción)
- [1. Primer Paso: Creación del Repositorio y Reporte de Inicio](#1-primer-paso-creación-del-repositorio-y-reporte-de-inicio)
- [2. Dinámica de Colaboración](#2-dinámica-de-colaboración)
- [3. Estandarización del Proyecto y Documentación](#3-estandarización-del-proyecto-y-documentación)
- [4. Automatización del Tablero Kanban](#4-automatización-del-tablero-kanban)
- [5. Las Reglas del Juego (Protección de Ramas)](#5-las-reglas-del-juego-protección-de-ramas)
- [6. Simulación: El Flujo Colaborativo](#6-simulación-el-flujo-colaborativo)
- [7. Criterios de Evaluación](#7-criterios-de-evaluación)

---

## 0. Introducción

¡Hola a todos! Soy Aldo Luna Bueno. Sé de primera mano que escribir buen código es solo una parte del trabajo; la otra mitad es saber integrarlo en equipo sin romper producción. En este laboratorio de una hora, no nos enfocaremos en programar, sino en configurar la infraestructura colaborativa de un repositorio. Aprenderemos a automatizar tableros Kanban, aplicar reglas de validación locales (Git Hooks) y proteger nuestras ramas de código.

## 1. Primer Paso: Creación del Repositorio y Reporte de Inicio

Para asegurar que todos comiencen a trabajar desde el principio, lo primero que haremos será crear nuestro espacio de trabajo y reportarlo.

1. **Crear el repositorio:** Cada estudiante debe crear un repositorio público en su cuenta personal de GitHub.
* **Nombre recomendado:** `lab-colaboracion-[apellido1]-[apellido2]`
    ```bash
    # Una vez creado en GitHub, clónalo a tu máquina local:
    git clone https://github.com/[tu-usuario]/lab-colaboracion-[apellido1]-[apellido2].git
    cd lab-colaboracion-[apellido1]-[apellido2]
    ```
1. **Reportar el inicio (Pull Request inicial):**
* Hagan un *fork* del [repositorio oficial del laboratorio](https://github.com/AldoLunaBueno/ds-lab-1-gestion.git).
* En su *fork*, abran el archivo `registro_estudiantes.md`.
* Busquen sus apellidos y nombres en la lista. En esa **misma línea** despuś de la flecha peguen el enlace del repositorio `lab-colaboracion` que acaban de crear, sin alterar ninguna otra línea para no crear conflictos.
* Creen un Pull Request (PR) hacia mi repositorio. Este PR será su "asistencia" y el punto de partida del laboratorio.
    ```bash
    # Si prefieren hacer este reporte desde la terminal en lugar de la web:
    git clone https://github.com/[tu-usuario]/ds-lab-1-gestion.git
    cd ds-lab-1-gestion

    # ... (Edita el archivo registro_estudiantes.md con tu editor favorito) ...

    git add registro_estudiantes.md
    git commit -m "chore: agregar enlace de mi repositorio"
    git push origin main
    # Finalmente, abran el PR desde la interfaz de GitHub de su fork.
    ```

## 2. Dinámica de Colaboración

El desarrollo moderno es colaborativo. Para este laboratorio no es obligatorio trabajar estrictamente en parejas; pueden agruparse con uno o más compañeros para colaborar mutuamente en sus repositorios.

Es muy importante entender la división de responsabilidades:

* **El Administrador (Dueño del repo):** Es el arquitecto del proyecto. Va a `Settings > Collaborators` e invita a su(s) compañero(s). Además, es el **único** encargado de crear los Issues, gestionar el tablero Kanban y aprobar los PRs.
* **Los Colaboradores:** Aceptan la invitación y clonan el repositorio localmente. Su trabajo es resolver los Issues asignados y enviar PRs. No crean Issues ni modifican el Kanban.

*Nota: Todos tendrán la oportunidad de ser administradores de su propio repositorio y colaboradores en el de sus compañeros.*

## 3. Estandarización del Proyecto y Documentación

Para evitar que el código desordenado llegue al repositorio, configuraremos reglas locales y plantillas. En la raíz de su repositorio, creen la siguiente estructura de carpetas y archivos básicos:

```text
├── .githooks/
│   ├── commit-msg 
│   └── pre-commit 
├── .github/
│   ├── pull_request_template.md
│   └── ISSUE_TEMPLATE/
│       └── sprint_issue.md
└── README.md

```

1. **El README.md:** Este archivo servirá como su reporte interno. Inicien el documento con un título y una sección de "Evidencias".
2. **Git Hooks:** Creen los archivos de validación que forzarán el uso de *Conventional Commits* y nombres de ramas correctos. Llenen los archivos con el código correspondiente:



`.githooks/commit-msg`:

```bash
#!/usr/bin/env bash

# Definición de colores ANSI
RED='\033[0;31m'
GREEN='\033[0;32m'
YELLOW='\033[0;33m'
NC='\033[0m' # Sin color (restablece el formato)

# Archivo que contiene el mensaje del commit
commit_file="$1"

# Extraemos solo la primera línea (el título) para validarla
commit_msg_title=$(head -n 1 "$commit_file")

commit_msg_title_max_size=72
valid_types="feat fix docs test refactor style chore build ci perf"

# Expresión regular para Conventional Commits
convention_pattern="^([a-z]+)(\(([a-zA-Z0-9_-]+)\))?(!)?:[[:space:]]+(.+)$"

if [[ $commit_msg_title =~ $convention_pattern ]]; then
    type="${BASH_REMATCH[1]}"
    scope="${BASH_REMATCH[3]}"
    breaking_change="${BASH_REMATCH[4]}"
    description="${BASH_REMATCH[5]}"

    # 1. Validar que el tipo esté en la lista permitida
    if ! echo "$valid_types" | grep -w -q "$type"; then
        echo -e "${RED}Error: '$type' no es un tipo válido para el mensaje de commit.${NC}"
        echo -e "${YELLOW}Tipos permitidos: $valid_types${NC}"
        exit 1
    fi

    # 2. Validar la longitud máxima del título
    if (( ${#commit_msg_title} > commit_msg_title_max_size )); then
        echo -e "${RED}Error: El título del commit tiene ${#commit_msg_title} caracteres.${NC}"
        echo -e "${YELLOW}El límite máximo es de $commit_msg_title_max_size caracteres.${NC}"
        exit 1
    fi
else
    # Mensaje de error si la estructura no coincide
    echo -e "${RED}Error: Formato de mensaje de commit incorrecto.${NC}"
    echo -e "${YELLOW}El formato debe seguir Conventional Commits:${NC}"
    echo -e "${YELLOW}   <tipo>[alcance opcional][! opcional]: <descripción>${NC}"
    echo ""
    echo -e "Ejemplos válidos:"
    echo -e "  - feat: agregar nueva pantalla de inicio"
    echo -e "  - fix(ui-login): corregir alineación de botones"
    echo -e "  - refactor(api)!: cambiar la estructura de la base de datos"
    exit 1 
fi

echo -e "${GREEN}Mensaje de commit válido.${NC}"
```

`.githooks/pre-commit`:

```bash
#!/usr/bin/env bash

# Definición de colores ANSI
RED='\033[0;31m'
YELLOW='\033[0;33m'
NC='\033[0m' # Sin color (restablece el formato)

# Obtener el nombre de la rama actual (funciona incluso en el primer commit)
local_branch="$(git branch --show-current)"

# Definir el patrón de ramas válidas
valid_pattern="^(main|feature/[a-z0-9\-]+|bugfix/[a-z0-9\-]+)$"

# Si branch --show-current devuelve vacío, estamos en "detached HEAD".
# Permitimos el commit en este estado (ej. durante un rebase o checkout de un tag).
if [[ -z "$local_branch" ]]; then
    exit 0
fi

# Validar el nombre de la rama contra el patrón
if [[ ! $local_branch =~ $valid_pattern ]]; then
    echo -e "${RED}Error: El nombre de la rama '$local_branch' no es válido.${NC}"
    echo -e "${YELLOW}La estrategia de ramificación requiere usar uno de los siguientes formatos:${NC}"
    echo -e "${YELLOW}  - main${NC}"
    echo -e "${YELLOW}  - feature/kebab-case (ej. feature/nuevo-login)${NC}"
    echo -e "${YELLOW}  - bugfix/kebab-case  (ej. bugfix/error-calculo)${NC}"
    exit 1
fi
```

3. **Activar los Hooks:** En su terminal, ejecuten `git config core.hooksPath .githooks` para que Git utilice estas reglas locales.


4. **Plantillas:** Agreguen las plantillas para estandarizar la creación de Issues y PRs:



`.github/pull_request_template.md`:

```markdown
## Descripción

<!-- Resume brevemente el propósito de esta PR, junto con algo de contexto -->

## Cambios introducidos

<!-- Detalla las modificaciones agrupadas por ámbito -->

## Issues relacionadas

<!-- Vincula las issues para su cierre automático en el tablero Kanban (ej. Closes #1) -->
Closes #XX
```

`.github/ISSUE_TEMPLATE/sprint_issue.md`:

```markdown
---
name: Sprint Issue
about: Plantilla para registrar issues vinculadas al plan de desarrollo por sprint
title: 'Issue [Sprint].[Épica].[N°]: [Título de la tarea]'
assignees: ''
---

- **Rol:** [Frontend | Backend | DevOps | Integraciones]
- [Criterio de aceptación / tarea concreta 1]
- [Criterio de aceptación / tarea concreta 2]
- [Criterio de aceptación / tarea concreta 3]
```

## 4. Automatización del Tablero Kanban

Dejaremos que GitHub actualice los estados por nosotros. Esta tarea la realiza el **Administrador**.

1. **Crear el Tablero:** Vayan a la pestaña `Projects` de su perfil y creen un nuevo tablero Kanban con las columnas: `Backlog`, `In progress`, `In review` y `Done`.
2. **Visibilidad:** El Administrador debe asegurarse en los *Settings* del proyecto que la visibilidad del tablero sea **Pública** para lectura. El colaborador podrá verlo, pero no debe tener permisos de edición.
3. **Configurar el Token:** Vayan a los *Settings* de su cuenta (Developer settings > Personal access tokens) y generen un token con permisos de *repo* y *project*. Luego, en su repositorio, guárdenlo como un secreto de Actions llamado `GH_TOKEN`.
4. **El Workflow:** Creen la carpeta de flujos de trabajo y el archivo correspondiente para automatizar las tarjetas:



`.github/workflows/board-automation.yml`:

```yaml
name: Board Automation (CI Gate 6)

on:
  pull_request:
    # 1. Escuchamos todos los tipos de eventos que nos interesan
    types: [opened, reopened, ready_for_review, closed]

permissions:
  contents: read
  issues: write
  pull-requests: read

jobs:
  move-project-card:
    runs-on: ubuntu-22.04
    steps:
      # 1. CONFIGURACIÓN
      - name: Checkout
        uses: actions/checkout@v3

      # 2. LÓGICA CONDICIONAL
      # Aquí determinamos el estado de destino basado en el evento que disparó el workflow.
      # Usamos $GITHUB_OUTPUT para pasar el resultado al siguiente paso.
      - name: Determine Target Status
        id: set-status # Le damos un ID a este paso para referenciar sus 'outputs'
        run: |
          if [[ "${{ github.event.action }}" == "opened" || "${{ github.event.action }}" == "reopened" ]]; then
            echo "target_status=In progress" >> $GITHUB_OUTPUT
          elif [[ "${{ github.event.action }}" == "ready_for_review" ]]; then
            echo "target_status=In review" >> $GITHUB_OUTPUT
          elif [[ "${{ github.event.action }}" == "closed" && "${{ github.event.pull_request.merged }}" == "true" ]]; then
            echo "target_status=Done" >> $GITHUB_OUTPUT
          else
            echo "target_status=" >> $GITHUB_OUTPUT # String vacío
            echo "Evento (${{ github.event.action }}) no requiere acción. Saliendo."
          fi

      # 3. EJECUCIÓN
      # Este paso solo se ejecuta si el paso anterior ('set-status') 
      # definió un 'target_status' (no está vacío).
      - name: Find and move project items
        if: steps.set-status.outputs.target_status != ''
        env:
          GH_TOKEN: ${{ secrets.GH_TOKEN }} # La CLI se autentica automáticamente con esto
          PR_NUMBER: ${{ github.event.pull_request.number }}
          PROJECT_NUMBER: 1 # número de proyecto que vamos a automatizar
          ORG: "@me" # propietario del proyecto
          # Inyectamos el estado de destino determinado en el paso anterior
          STATUS_NAME: ${{ steps.set-status.outputs.target_status }} 
        run: |
          echo "Evento: ${{ github.event.action }}. Moviendo issue a '$STATUS_NAME'..."

          echo "Buscando issue vinculado al PR #$PR_NUMBER..."
          ISSUE_NUMBER=$(gh pr view "$PR_NUMBER" --json closingIssuesReferences \
              -q ".closingIssuesReferences[0].number")
          if [ -z "$ISSUE_NUMBER" ]; then
              echo "No issue vinculated to PR"
              exit 0
          fi
          
          echo "Buscando IDs del Proyecto, Campo, y Opción..."
          PROJECT_ID=$(gh project list --format json \
            -q ".projects[]|select(.number == "$PROJECT_NUMBER").id")
          
          # Obtenemos la información del campo "Status" y sus opciones en una sola llamada
          FIELD_INFO_JSON=$(gh project field-list "$PROJECT_NUMBER" --owner "$ORG" \
              --format json --jq '.fields | map(select(.name == "Status"))[0]')
          
          FIELD_ID=$(echo $FIELD_INFO_JSON | jq -r '.id')
          
          # Usamos --arg para pasar la variable de shell a jq de forma segura
          OPTION_ID=$(echo $FIELD_INFO_JSON | jq -r --arg STATUS_NAME "$STATUS_NAME" \
              '.options | map(select(.name == $STATUS_NAME).id)[0]')

          echo "Buscando ID del elemento del issue en el proyecto..."
          ITEM_ID=$(gh project item-list "$PROJECT_NUMBER" --owner "$ORG" \
              --format json --jq ".items | map(select(.content.number == "$ISSUE_NUMBER").id)[0]")

          if [ -z "$ITEM_ID" ]; then
              echo "Error: No se pudo encontrar el Item ID para el Issue #$ISSUE_NUMBER en el proyecto."
              exit 1
          fi
          
          echo "Actualizando estado del elemento #$ITEM_ID a '$STATUS_NAME' (Option ID: $OPTION_ID)..."
          gh project item-edit --project-id "$PROJECT_ID" --id "$ITEM_ID" \
              --field-id "$FIELD_ID" --single-select-option-id "$OPTION_ID"
```

## 5. Las Reglas del Juego (Protección de Ramas)

Para evitar que alguien haga *push* directo a producción, el **Administrador** configurará la protección de la rama principal.

1. Vayan a `Settings > Branches > Add branch protection rule` y apliquen a la rama `main`:


* Requerir un Pull Request antes de fusionar (*Require a pull request before merging*).


* Requerir aprobaciones (*Require approvals*, configurar al menos 1).


* Requerir resolución de conversaciones (*Require resolution of conversations*).


* Bloquear eliminaciones y *force pushes* (*deletion* y *non_fast_forward*).




2. **Evidencia para revisión:** Tomen una captura de pantalla de estas reglas de protección guardadas y colóquenla en la sección "Evidencias" de su `README.md`.

## 6. Simulación: El Flujo Colaborativo

Ahora pondremos todo a prueba. Sigan estrictamente el orden:

1. **El Administrador** crea un *Issue* en su repositorio usando la plantilla provista y se lo asigna al **Colaborador**.


2. **El Colaborador** revisa el Issue, crea una rama local, hace un cambio en el repositorio y realiza un commit intencionalmente mal escrito.
3. **Evidencia de Hooks:** El Colaborador toma una **captura de pantalla** de su terminal mostrando el mensaje de error generado por el hook `commit-msg` al fallar, y luego otra captura con el mensaje satisfactorio tras realizar el commit con el formato correcto (ej. `docs: actualizar readme`). Añadan estas capturas al `README.md`.


4. **Constancia de colaboración:** Dentro de la misma rama, el colaborador debe modificar el archivo `README.md` agregando una sección que diga "Colaboradores" y colocando ahí su nombre y apellido.
5. El Colaborador hace *push* y abre un PR. En el cuerpo del PR, debe escribir `Closes #1` (o el número de la Issue) para vincularlo al tablero.


6. **El Administrador** revisa el PR, hace un comentario para probar los hilos de conversación, lo aprueba y fusiona.


7. **Verificación Kanban:** Ambos revisan el tablero Kanban (el Administrador como dueño, el Colaborador como lector público) para confirmar que la Issue viajó automáticamente de `Backlog` hasta `Done`.

---

## 7. Criterios de Evaluación

Este laboratorio será calificado de forma ágil basándose en evidencias directas en los repositorios entregados:

* [ ] **Criterio 1 (Reporte de Inicio - 4 pts):** El estudiante creó exitosamente el PR inicial hacia `ds-lab-1-gestion.git`, colocando el enlace de su repositorio en la misma línea que su nombre en `registro_estudiantes.md`.
* [ ] **Criterio 2 (Evidencias Documentales - 4 pts):** El archivo `README.md` incluye las capturas de pantalla requeridas: la regla de protección de la rama `main` y las salidas de consola del hook `commit-msg` (error y éxito).


* [ ] **Criterio 3 (Constancia de Colaboración - 4 pts):** El archivo `README.md` incluye los nombres de los compañeros que fungieron como colaboradores, evidenciando que se completó la dinámica de equipo con roles definidos.
* [ ] **Criterio 4 (Historial Limpio y Estructura - 4 pts):** El repositorio contiene las carpetas `.githooks` y `.github` configuradas correctamente. El historial de commits refleja el uso estricto de *Conventional Commits* demostrando la aplicación de los hooks.


* [ ] **Criterio 5 (Automatización Verificable - 4 pts):** Existe evidencia de al menos una Issue vinculada y cerrada a través del flujo de un PR (`Closes #ID`) y el historial de la pestaña *Actions* muestra ejecuciones exitosas del workflow de automatización del tablero.
