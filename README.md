# media_center_k8s

Este repositorio contiene manifestos Kubernetes y utilidades para desplegar un stack multimedia (Jellyfin, Sonarr, Radarr, Lidarr, Ombi, Whisparr, Stash, etc.) en un clúster con Longhorn u hostPath.

Resumen rápido
- Archivos de configuración Kubernetes (PVCs, Deployments, Services, Ingress): [k8s/00-media-stack-pvc.yaml](k8s/00-media-stack-pvc.yaml), [k8s/media-stack-hostpath.yaml](k8s/media-stack-hostpath.yaml), [k8s/01-media-stack-longhorn.yaml](k8s/01-media-stack-longhorn.yaml), [k8s/02-media-ingress.yaml](k8s/02-media-ingress.yaml), [k8s/LimitRange_quota.yalm](k8s/LimitRange_quota.yalm).
- Job de migración para copiar datos host → PVC: [k8s/migrator.yaml](k8s/migrator.yaml).

Descripción
Este proyecto ofrece manifiestos para:
- Provisionar PVCs y PVs (hostPath o Longhorn) — ver [k8s/00-media-stack-pvc.yaml](k8s/00-media-stack-pvc.yaml).
- Desplegar aplicaciones del media-stack con opciones para Longhorn ([k8s/01-media-stack-longhorn.yaml](k8s/01-media-stack-longhorn.yaml)) o usando hostPath ([k8s/media-stack-hostpath.yaml](k8s/media-stack-hostpath.yaml)).
- Configurar Ingress/TLS para los servicios expuestos (Traefik + cert-manager): [k8s/02-media-ingress.yaml](k8s/02-media-ingress.yaml).
- Aplicar límites por defecto y cuotas para el namespace media: [k8s/LimitRange_quota.yalm](k8s/LimitRange_quota.yalm).
- Ejecutar una job de migración segura con rsync para mover datos al PVC: [k8s/migrator.yaml](k8s/migrator.yaml).

Requisitos previos
- Un cluster Kubernetes funcional (kubeconfig).
- Si usa Longhorn: Longhorn instalado y storageClass configurada.
- Traefik y cert-manager instalados si quiere TLS automático.
- Asegúrese de tener permisos para crear PV/PVC/Ingress/Deployments.

Cómo desplegar (mínimo)
1. Crear el namespace y PVCs/configs:
   kubectl apply -f k8s/00-media-stack-pvc.yaml
2. Desplegar el stack (Longhorn):
   kubectl apply -f k8s/01-media-stack-longhorn.yaml
   - O, si prefiere hostPath: kubectl apply -f k8s/media-stack-hostpath.yaml
3. Configurar ingress/TLS:
   kubectl apply -f k8s/02-media-ingress.yaml
4. Opcional: ajustar límites y cuotas:
   kubectl apply -f k8s/LimitRange_quota.yalm

Comandos útiles
- Aplicar todo en orden:
  kubectl apply -f k8s/00-media-stack-pvc.yaml && \
  kubectl apply -f k8s/01-media-stack-longhorn.yaml && \
  kubectl apply -f k8s/02-media-ingress.yaml
- Revisar pods:
  kubectl -n media get pods
- Logs de un pod:
  kubectl -n media logs deploy/whisparr

Notas de configuración importantes
- Cambie nombres y secretos:
  - Revise [k8s/01-media-stack-longhorn.yaml](k8s/01-media-stack-longhorn.yaml) y [k8s/media-stack-hostpath.yaml](k8s/media-stack-hostpath.yaml) para ajustar:
    - externalIPs, storageClassName, size de PVCs, credenciales de MySQL (Ombi).
- Ingress:
  - Modifique hostnames y secretName en [k8s/02-media-ingress.yaml](k8s/02-media-ingress.yaml) para apuntar a su dominio.
- Seguridad:
  - No deje contraseñas por defecto en los manifests (ej. MYSQL_PASSWORD).
  - Rotar API keys y revisar permisos de los Deployments (securityContext existente en algunos manifests).
- Storage:
  - Si usa Longhorn, los PVCs en [k8s/00-media-stack-pvc.yaml](k8s/00-media-stack-pvc.yaml) están configurados para longhorn; ajuste tamaños según uso.
  - Si usa hostPath, editar las rutas locales en [k8s/media-stack-hostpath.yaml](k8s/media-stack-hostpath.yaml).

Migración de datos
- Para copiar datos del host al PVC de Longhorn/hostPath use la job de migrador:
  kubectl apply -f k8s/migrator.yaml
  - Ajuste en el manifest la ruta hostPath /src y el claimName /dest si hace falta.

Estructura de archivos (k8s/)
- [k8s/00-media-stack-pvc.yaml](k8s/00-media-stack-pvc.yaml) — Namespace + PV/PVCs (Longhorn/hostPath).
- [k8s/01-media-stack-longhorn.yaml](k8s/01-media-stack-longhorn.yaml) — Deployments y Services para Longhorn.
- [k8s/media-stack-hostpath.yaml](k8s/media-stack-hostpath.yaml) — Alternativa con hostPath.
- [k8s/02-media-ingress.yaml](k8s/02-media-ingress.yaml) — Ingress + TLS hostnames.
- [k8s/LimitRange_quota.yalm](k8s/LimitRange_quota.yalm) — LimitRange y ResourceQuota defaults.
- [k8s/migrator.yaml](k8s/migrator.yaml) — Job para migración de datos.

Consejos rápidos
- Cambie PUID/PGID en los contenedores si sincroniza permisos con host.
- Validar cambios localmente con kubectl apply --dry-run=client -f <file>.
- Use kubecontext correcto: kubectl config use-context <context>

Licencia
- Este repositorio contiene archivos con licencia Apache-2.0 (ver [LICENSE](LICENSE)).