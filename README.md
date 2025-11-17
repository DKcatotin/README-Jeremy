# Demo de CI/CD con Python, Docker y GitHub Actions

Este repositorio demuestra paso a paso cómo implementar un **flujo CI/CD moderno** para una aplicación Python utilizando **Docker** y **GitHub Actions**, donde el *package final* generado por el pipeline es una **imagen Docker publicada automáticamente en GitHub Container Registry (GHCR)**.

Este README explica:

- Cómo funciona todo el ciclo **CI/CD**.
- Cómo se construye y publica la imagen Docker.
- Cómo se ejecutan pruebas automatizadas con `pytest`.
- Qué archivos componen el proyecto.
- Qué hace exactamente el workflow de GitHub Actions.

---

## 1. Objetivo del proyecto

El propósito de este proyecto es mostrar un flujo completo de CI/CD donde:

1. **Python ejecuta pruebas unitarias**.
2. Se construye una **imagen Docker** usando un `Dockerfile`.
3. Esa imagen es subida automáticamente a:

```
ghcr.io/<usuario>/<nombre-de-imagen>
```

En este caso:

```
ghcr.io/dkcatotin/readme-jeremy
```

Este enfoque cumple perfectamente con la rúbrica de CI/CD donde el “package” generado es la **imagen Docker**, no un wheel de Python.

---

## 2. ¿Qué es CI/CD?

### 🔵 Integración Continua (CI)

La Integración Continua garantiza que cada cambio de código enviado al repositorio:

- Se valide automáticamente.
- Instale dependencias.
- Ejecute pruebas unitarias.
- Detecte errores antes de llegar a producción.

En este proyecto, CI incluye:

- `pip install -r requirements.txt`
- `pytest`

---

### 🟢 Entrega Continua (CD)

La Entrega Continua prepara automáticamente artefactos listos para despliegue.

En este caso:

- Se construye la **imagen Docker** usando `docker build`.
- Se publica a **GitHub Container Registry** (`ghcr.io`) mediante `docker/build-push-action`.

El resultado final es un **package** instalable y ejecutable: la imagen Docker.

---

## 3. Estructura del proyecto

```text
.
├── .github/
│   └── workflows/
│       └── python-docker-ci.yml
├── Dockerfile
├── app.py
├── requirements.txt
└── tests/
    └── test_app.py
```

---

## 4. Código Python del proyecto

### `app.py`

```python
def saludar(nombre: str) -> str:
    return f"Hola, {nombre} desde python 🐍"


if __name__ == "__main__":
    print(saludar("Mundo"))
```

---

## 5. Dependencias (`requirements.txt`)

```txt
pytest
```

---

## 6. Pruebas automatizadas (`tests/test_app.py`)

```python
from app import saludar

def test_saludar():
    assert saludar("Jeremy") == "Hola, Jeremy desde python 🐍"
```

Estas pruebas son ejecutadas automáticamente por GitHub Actions.

---

## 7. Dockerfile (el origen del PACKAGE final)

```dockerfile
FROM python:3.10-slim

WORKDIR /app

COPY requirements.txt .
RUN pip install --no-cache-dir -r requirements.txt

COPY app.py .

CMD ["python", "app.py"]
```

La imagen generada incluirá tu aplicación Python lista para ejecutarse.

---

## 8. CI/CD completo (workflow)

Archivo: `.github/workflows/python-docker-ci.yml`

```yaml
name: Python Docker Image Build

on:
  push:
    branches: [ "catota" ]
  pull_request:
    branches: [ "catota" ]

permissions:
  contents: read
  packages: write   # Necesario para subir imágenes a GHCR

env:
  IMAGE_NAME: ghcr.io/dkcatotin/readme-jeremy

jobs:
  build:
    runs-on: ubuntu-latest

    steps:
      - name: Checkout del código
        uses: actions/checkout@v4

      - name: Set up Python 3.10
        uses: actions/setup-python@v5
        with:
          python-version: "3.10"

      - name: Install dependencies
        run: |
          python -m pip install --upgrade pip
          pip install -r requirements.txt

      - name: List directory for debug
        run: |
          echo "Listing root folder:"
          ls -R .
          echo "Listing tests folder:"
          ls tests

      - name: Run tests
        run: |
          echo "PYTHONPATH antes:"
          echo $PYTHONPATH

          # FIX FINAL → agrega la raíz al path de Python
          export PYTHONPATH=.

          echo "PYTHONPATH después:"
          echo $PYTHONPATH
          
          pytest

      - name: 🔐 Login to GitHub Container Registry
        uses: docker/login-action@v3
        with:
          registry: ghcr.io
          username: ${{ github.actor }}
          password: ${{ secrets.GITHUB_TOKEN }}

      - name: 🏗️ Build and push Docker image
        uses: docker/build-push-action@v6
        with:
          context: .
          push: true
          tags: |
            ${{ env.IMAGE_NAME }}:latest
            ${{ env.IMAGE_NAME }}:${{ github.sha }}
```

---

## 9. ¿Qué produce exactamente este pipeline?

Cuando haces un **push a main**, el pipeline:

1. instala dependencias  
2. ejecuta pruebas  
3. construye una imagen Docker  
4. la sube automáticamente a GHCR  

Al terminar, puedes ver tu “package” (la imagen Docker) aquí:

```
https://github.com/dkcatotin?tab=packages
```

---

## 10. Ejecutar la imagen Docker publicada

Una vez publicada, puedes ejecutarla desde cualquier máquina con Docker:

```bash
docker pull ghcr.io/dkcatotin/jeremy-catota:latest
docker run --rm ghcr.io/dkcatotin/jeremy-catota:latest
```

Salida:

```
Hola, Mundo desde python 🐍
```

---

## 11. Ventajas de este flujo CI/CD
 
✔ Usa Python como base del proyecto  
✔ Usa Docker como empaquetado profesional  
✔ Genera un artefacto real y reutilizable  
✔ El pipeline es automático y confiable  
✔ Imagen accesible desde cualquier entorno

---

## 12. Resumen final

Este proyecto implementa un flujo de CI/CD moderno donde:

- GitHub Actions prueba, construye y despliega tu app  
- Docker empaqueta tu aplicación como contenedor  
- GHCR almacena y distribuye la imagen final  