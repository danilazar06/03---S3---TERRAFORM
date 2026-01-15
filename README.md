# Actividad: Despliegue de S3 con Terraform y GitHub Actions

Este documento resume los pasos realizados para configurar credenciales, definir infraestructura con Terraform y configurar flujos de trabajo en GitHub Actions, incluyendo la solución a problemas con ficheros grandes.

## 1. Configuración de Credenciales AWS CLI

Configuramos las credenciales de acceso para permitir que Terraform interactúe con AWS.

```powershell
# Verificar instalación
aws --version

# Configurar credenciales (sustituyendo con tus valores)
aws configure set aws_access_key_id ASIAXK7H4Q3UJERJO3W3
aws configure set aws_secret_access_key 0jFuvBFCW+9CCpnV1vPFBfWLEtPsCVVMPZwSt3sO
aws configure set aws_session_token IQoJb3JpZ2luX2Vj...
aws configure set region us-west-2

# Verificar identidad
aws sts get-caller-identity
```

## 2. Definición de Infraestructura (Terraform)

Editamos el archivo `main.tf` para definir un bucket S3 configurado como sitio web estático con acceso público.

**Recursos añadidos:**
*   `aws_s3_bucket`: El contenedor de objetos.
*   `aws_s3_bucket_website_configuration`: Configuración para servir `index.html`.
*   `aws_s3_bucket_public_access_block`: Desbloqueo de restricciones públicas.
*   `aws_s3_bucket_policy`: Política JSON para permitir lectura pública (`s3:GetObject`).

## 3. GitHub Actions

Creamos dos flujos de trabajo en `.github/workflows/`:

1.  **`01-dispatch.yml`**: Se ejecuta manualmente (`workflow_dispatch`).
2.  **`02-push.yml`**: Se ejecuta automáticamente al hacer `push` a la rama `main` o manualmente.

## 4. Gestión del Repositorio y Solución de Errores

Al intentar subir los cambios, nos encontramos con un error porque la carpeta `.terraform` contenía binarios pesados (>100MB).

**Comando fallido:**
```bash
git add .
git commit -m '1'
git push
# Error: File ...terraform-provider-aws...exe is 628.32 MB
```

**Solución aplicada:**

1.  **Crear `.gitignore`** para excluir archivos de Terraform:
    ```gitignore
    .terraform
    .terraform.lock.hcl
    *.tfstate
    *.tfstate.*
    ```

2.  **Limpiar el commit sucio** (deshacer el commit sin borrar cambios de archivos):
    ```bash
    # Deshacer el último commit manteniendo los cambios en local
    git reset --soft HEAD~1
    
    # Sacar la carpeta .terraform del área de preparación (staging)
    git reset HEAD .terraform
    ```

3.  **Realizar el push limpio**:
    ```bash
    git status # Verificar que .terraform ya no aparece o está ignorado
    git add .
    git commit -m "Add terraform config and ignore large files"
    git push
    ```

## 5. Actualizaciones Posteriores

Añadimos el segundo workflow y actualizamos el repositorio:

```bash
git add .
git commit -m '2'
git push
```
