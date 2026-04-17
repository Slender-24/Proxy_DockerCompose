# Práctica: Proxy Inverso con Nginx y Apache (Balanceo de carga y Caché)

Esta práctica consiste en el despliegue de una arquitectura web en alta disponibilidad usando Docker y Docker Compose. Está diseñada para ejecutarse en entornos Linux (como Kali Linux) de forma sencilla.

## Arquitectura

- **Proxy Inverso (Nginx):** Recibe las peticiones originales en el puerto `80`. Distribuye el tráfico y aplica configuraciones de caché de hasta 500 MB.
- **Servidores Backend (Apache 1 y Apache 2):** Responden a las peticiones delegadas del proxy inverso equilibrando la carga de trabajo entre ambos mediante *Round Robin*.
- **Volumen Compartido (`html/`):** Contiene un único index.html y activos (imágenes y vídeos) para que ambos servidores Apache sirvan el mismo contenido constante, lo que garantiza la integridad y reduce el mantenimiento.

## Requisitos Previos

- Contar con un sistema Linux / máquina virtual con `Docker` y `Docker Compose` (V2) instalados (por ejemplo, en Kali Linux).
- Tener puertos `80` desocupados en la máquina anfitrión.
- Tener clonado este repositorio.

## Instrucciones de Despliegue

1. **Clonar repositorio**
   Descarga el código a tu máquina Linux desde tu GitHub:
   ```bash
   git clone <enlace-de-tu-repositorio>
   cd <nombre-del-repositorio>
   ```

2. **Añadir los archivos multimedia**
   Asegúrate de colocar los siguientes archivos multimedia dentro del directorio interno `html/`, respetando los siguientes nombres: 
   - `html/imagen1.jpg`
   - `html/imagen2.jpg`
   - `html/imagen3.jpg`
   - `html/video.mp4`

3. **Ejecutar el Clúster de Docker**
   Levanta la red de contenedores en segundo plano usando Docker Compose. Asegúrate de estar en el directorio raíz de la práctica (donde está situado el `docker-compose.yml`):
   ```bash
   sudo docker compose up -d
   ```

4. **Verificación de Funcionamiento**
   - Entra en tu navegador web de Kali Linux e introduce `http://localhost`. Deberías visualizar automáticamente la página web y sus archivos (el proxy inverso está procesando y cacheando tu petición).
   - Puedes verificar la efectividad del balanceo de carga comprobando a quién le llega la carga mirando los logs: `sudo docker compose logs -f`.
   - **Prueba de Resiliencia (Tolerancia a fallos):** Puedes simular la caída de un servidor parando el primer apache: `sudo docker stop apache1`. Si a continuación refrescas la página web (`http://localhost`), ésta seguirá sirviéndose sin interrupciones porque Nginx desvía instantáneamente el tráfico al contenedor `apache2`.

## Limpieza

Si deseas apagar los servidores y eliminar todas las redes de Docker asociadas, ejecuta en el directorio del proyecto:
```bash
sudo docker compose down
```
