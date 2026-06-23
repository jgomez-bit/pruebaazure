# pruebaazure

Aquí está `files.zip`, el entregable de la prueba técnica para el rol de Líder de Ciberseguridad. Este README es solo para que sepas qué vas a encontrar adentro antes de descargarlo.

## Qué hay en el zip

Al descomprimir `files.zip` salen dos cosas:

- `ARQUITECTURA.md`: el documento donde explico la solución, las decisiones que tomé y los problemas con los que me topé al desplegarla.
- `prueba-agentes-efimeros.zip`: el proyecto completo (código, infraestructura y pipeline).

Dentro de ese segundo zip está el repo real:

```
app/                FastAPI mínima de demo (build → test → scan → deploy)
  main.py           endpoints de salud + un echo validado con Pydantic
  tests/            pruebas con pytest
  Dockerfile        imagen no-root, base con tag fijo
agent/              imagen del agente efímero
  start.sh          registra el agente, corre un job en modo --once y se desregistra
infra/              toda la infra en Bicep
  main.bicep        + módulos: red, ACR, Key Vault, identidad, observabilidad, job del agente
azure-pipelines.yml pipeline que corre sobre los agentes efímeros
deploy.sh           despliegue de punta a punta
ARQUITECTURA.md     misma doc, dentro del proyecto
README.md           README propio del proyecto, con más detalle
```

## De qué trata

La idea es reemplazar los agentes de CI/CD montados sobre VMs dedicadas (más un Bastion permanente) por **agentes efímeros**: cada vez que se encola un pipeline nace un contenedor limpio, hace un solo job y se destruye. No queda estado entre ejecuciones, no hay nada encendido cuando no hay builds y no hay máquinas que parchear.

Corre sobre Azure DevOps + Azure Container Apps Jobs, con KEDA escalando de cero a N según la cola del pool. Todo vive dentro de una red privada: ACR y Key Vault solo se alcanzan por Private Endpoint, el entorno de Container Apps es interno y la API no tiene IP pública. La autenticación va por Managed Identity para no arrastrar secretos de larga vida.

No es un diseño de pizarra. Lo desplegué completo en una suscripción de Azure, así que en `ARQUITECTURA.md` también quedan anotados los tropiezos reales (el Key Vault privado que no resolvía la referencia del PAT, el huevo-y-gallina de la imagen del agente contra un ACR recién creado, las limitaciones del plan trial) y cómo los resolví.

## Por dónde empezar

Si solo quieres entender la propuesta, lee `ARQUITECTURA.md`. Si quieres reproducirla, descomprime `prueba-agentes-efimeros.zip` y sigue las instrucciones de su README y de `deploy.sh`.
