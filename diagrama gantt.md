## Diagrama de Gantt del Proyecto

```mermaid
gantt
    title Cronograma del TFG - Servidor Minecraft con Kubernetes
    dateFormat  YYYY-MM-DD
    axisFormat %d/%m

    section Planificación
    Planificación y estudio inicial     :done, p1, 2026-03-14, 6d

    section Infraestructura
    Instalación Ubuntu Server           :done, u1, 2026-03-20, 2d
    Instalación Kubernetes (K3s)        :done, k1, 2026-03-22, 5d
    Despliegue Minecraft                :done, m1, 2026-03-27, 7d

    section Monitorización
    Implementación Prometheus           :done, pr1, 2026-04-10, 3d
    Implementación Grafana              :done, g1, 2026-04-13, 7d
    Configuración Dashboards            :done, g2, 2026-04-20, 10d

    section Optimización
    Pruebas y optimización              :done, o1, 2026-05-01, 15d

    section Alertas
    Implementación Zabbix               :done, z1, 2026-05-16, 4d
    Configuración alertas               :done, z2, 2026-05-20, 8d

    section Publicación
    Contratación IP pública             :done, ip1, 2026-05-31, 1d
    Configuración NAT                   :done, nat1, 2026-06-01, 3d
    Pruebas con usuarios                :done, t1, 2026-06-04, 3d

    section Documentación
    Redacción memoria TFG               :active, doc1, 2026-03-14, 85d
