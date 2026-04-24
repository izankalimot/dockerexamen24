# Infraestructura de Balanceo de Carga con Docker

Este proyecto despliega una arquitectura web de alta disponibilidad utilizando Docker y Nginx como proxy inverso para balancear la carga entre dos servidores de backend.

## 🏗️ Esquema de la Arquitectura

```mermaid
graph TD
    User([Usuario Externo]) -- Puerto 80 --> Proxy[Nginx Proxy Inverso]
    
    subgraph "Red Interna Aislada (backend_net)"
        Proxy -- Balanceo Round Robin --> S1[Web Server 1]
        Proxy -- Balanceo Round Robin --> S2[Web Server 2]
        
        S1 -- Montaje Volumétrico --> Vol[(Contenido Web Compartido)]
        S2 -- Montaje Volumétrico --> Vol
    end
    
    subgraph "Red Externa (proxy_net)"
        Proxy
    end
```

## 🖼️ Elección de las Imágenes

Para este proyecto se ha seleccionado la imagen oficial de **Nginx (`nginx:latest`)** por las siguientes razones:

1.  **Rendimiento**: Extremadamente eficiente en el manejo de conexiones simultáneas y bajo consumo de recursos.
2.  **Versatilidad**: Funciona perfectamente tanto como **Proxy Inverso** (con capacidades de balanceo de carga nativas) como **Servidor Web** de contenido estático.
3.  **Configuración**: Su sintaxis es clara y permite añadir cabeceras personalizadas (`add_header`) de forma sencilla para el seguimiento de peticiones.
4.  **Estándar de la Industria**: Es la opción más robusta y documentada para entornos de producción.

## 🌐 Estructura de Red

La infraestructura utiliza un modelo de doble red para garantizar la seguridad:

*   **`proxy_net`**: Red pública (o expuesta al host) donde reside el proxy inverso. Es el único punto de entrada desde el exterior.
*   **`backend_net`**: Red configurada como `internal: true`. Esto significa que los servidores backend pueden comunicarse con el proxy, pero **no tienen acceso directo a la red externa** ni pueden ser contactados directamente desde fuera del entorno Docker.

## 🛠️ Comandos Necesarios

### Levantar el entorno
Para desplegar toda la infraestructura en segundo plano:
```powershell
docker-compose up -d
```

### Verificar el estado
Comprobar que los contenedores están en ejecución:
```powershell
docker-compose ps
```

### Verificar el Balanceo y Cabeceras
Ejecuta el siguiente comando varias veces para ver cómo cambia la IP del backend en la cabecera `X-Served-By`:
```powershell
curl.exe -I http://localhost
```

### Ver logs en tiempo real
Para monitorizar el tráfico que llega al proxy:
```powershell
docker-compose logs -f nginx-proxy
```

## 📁 Estructura del Proyecto
- `docker-compose.yml`: Orquestación de servicios y redes.
- `nginx-proxy.conf`: Configuración del balanceo y cabeceras.
- `web/`: Carpeta compartida con el código HTML.
- `web/imagenes/`: Almacén de contenido multimedia (imágenes y vídeo).
