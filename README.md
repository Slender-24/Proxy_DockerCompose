# Entrega de Práctica: Proxy Inverso (Nginx) y Balanceo de Carga

Esta es la entrega correspondiente a la práctica de configuración de un Proxy Inverso con balanceo de carga y sistema de caché, haciendo uso de contenedores Docker. 

A continuación, se detallan los componentes técnicos y las instrucciones necesarias para la ejecución y evaluación del sistema.

## Arquitectura del Entorno

La topología está orquestada mediante **Docker Compose** y cuenta con los siguientes componentes:

- **Nginx (Proxy Inverso):** Expuesto en el puerto `80` del host local. Recibe todo el tráfico entrante, realiza el balanceo de carga entre los servidores web y aplica una regla de caché máxima de 500 MB (ubicada en `/var/cache/nginx`).
- **Nodos Backend (Apache 1 y Apache 2):** Dos servidores HTTP independientes operando en una red interna. Ambos reciben las peticiones delegadas del proxy inverso equilibrando la carga de trabajo a través del algoritmo por defecto *Round Robin*.
- **Volumen Compartido (`html/`):** Directorio centralizado que alimenta de forma simultánea a ambos contenedores Apache. Su objetivo es garantizar la consistencia en el contenido servido (archivos de imagen, vídeo e index original) sin necesidad de duplicar la información.

## Pasos para Evaluación (Despliegue)

Para proceder con la comprobación de la infraestructura, se requiere de un entorno Linux (o similar) que disponga de `Docker` y `Docker Compose`.

**1. Clonar el repositorio**
Ejecute el siguiente comando para descargar este proyecto:
```bash
git clone <ENLACE_DE_ESTE_REPOSITORIO_GITHUB>
cd <NOMBRE_DEL_REPOSITORIO>
```

**2. Verificación de archivos multimedia**
El sistema espera contar con los siguientes archivos estáticos dentro de la carpeta local `./html/`:
- `imagen1.jpg`, `imagen2.jpg`, `imagen3.jpg`
- `video.mp4`

**3. Despliegue de los contenedores**
Desde el directorio raíz que incluye el archivo `docker-compose.yml`, levante la infraestructura en segundo plano:
```bash
sudo docker compose up -d
```

## Verificación Práctica

Una vez arrancado, ponga a prueba el sistema siguiendo estos pasos:

1. **Acceso inicial y caché:** 
   Abra el navegador web e introduzca `http://localhost`. Deberá visualizar correctamente la web integrada con las 3 imágenes y el vídeo alojados, habiendo pasado la petición por el Nginx inicial.
   
2. **Comprobación de Tolerancia a Fallos (Balanceo):**
   Puede simular de forma sencilla una caída de cualquiera de los servidores para comprobar que el proxy mantiene la disponibilidad. En la terminal escriba:
   ```bash
   sudo docker stop apache1
   ```
   A continuación, refresque la página web en `http://localhost`. Esta seguirá cargando con normalidad de forma transparente e ininterrumpida debido a que Nginx ha distribuido la petición a `apache2`.

3. **Verificación de logs (opcional):**
   Si desea analizar cómo el proxy distribuye el tráfico, visualice los registros en directo mientras refresca la ventana web:
   ```bash
   sudo docker compose logs -f
   ```

## Finalización y Limpieza

Para apagar el entorno y limpiar la red generada, al finalizar sus comprobaciones ejecute:
```bash
sudo docker compose down
```
