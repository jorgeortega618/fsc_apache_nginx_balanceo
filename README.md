# 🌐 Nginx - Apache Load Balancer

Este proyecto demuestra una arquitectura web de **alta disponibilidad y balanceo de carga** utilizando **Nginx** como Proxy Inverso frente a múltiples instancias de **Apache HTTP Server**, orquestado todo localmente a través de **Docker Compose**.

## 📑 Tabla de Contenidos
- [Arquitectura del Proyecto](#-arquitectura-del-proyecto)
- [Requisitos Previos](#-requisitos-previos)
- [Estructura del Repositorio](#-estructura-del-repositorio)
- [Ejecución del Proyecto](#-ejecuci%C3%B3n-del-proyecto)
- [Demostración Interactiva (Dashboard)](#-demostraci%C3%B3n-interactiva-dashboard)
- [Verificación por Consola](#-verificaci%C3%B3n-por-consola)

## 🏗 Arquitectura del Proyecto

El flujo HTTP se encuentra diseñado de la siguiente manera:

1. **El Cliente** (Navegador o utilidades como `curl`) realiza una petición al puerto `8080` de tu máquina local.
2. **Nginx** recibe la petición, procesa el ruteo mediante el algoritmo predeterminado *Round-Robin* en su bloque `upstream` y la retransmite a uno de los servidores backend disponibles.
3. **Apache 1, 2 o 3** procesan la solicitud individualmente de forma aislada, sirviendo un archivo `index.html` estático específico mediante un volumen montado que nos permite identificar qué nodo generó finalmente la respuesta.

*(Al limitar deliberadamente la directiva `worker_processes` a `1` en la configuración de Nginx para los propósitos académicos de esta demostración, garantizamos que el ruteo de peticiones se verifique de forma perfectamente cíclica).*

## ⚙ Requisitos Previos

- Tener instalado [Docker](https://www.docker.com/) o **Docker Desktop** (si utilizas Windows o macOS).
- Tener disponible en el path la utilidad de orquestación local: `docker-compose`.

## 📁 Estructura del Repositorio

```text
/
├── docker-compose.yml       # Define los 4 contenedores (1 proxy Nginx, 3 backends Apache) y su red de interconexión aislada.
├── dashboard.html           # Panel web interactivo para visualizar el comportamiento de balanceo en vivo.
├── nginx/
│   └── nginx.conf           # Configuración del proxy inverso Nginx, CORS, headers customizados e inyección en upstream.
├── apache1/
│   └── index.html           # Respuesta HTML de identidad estática para el Nodo 1.
├── apache2/
│   └── index.html           # Respuesta HTML de identidad estática para el Nodo 2.
└── apache3/
    └── index.html           # Respuesta HTML de identidad estática para el Nodo 3.
```

## 🚀 Ejecución del Proyecto

1. Accede desde una consola (Terminal, Bash, PowerShell) hasta la ruta raíz donde descargaste o clonaste este proyecto:
   ```bash
   cd ruta/del/proyecto
   ```

2. Ejecuta el archivo compose para levantar los contenedores sin anclar la terminal e instanciar la red (`-d` significa detached mode):
   ```bash
   docker-compose up -d
   ```

3. Comprueba el estado de los contenedores asegurándote de que `apache1_backend`, `apache2_backend`, `apache3_backend` y `nginx_proxy` figuren con el STATUS `Up`:
   ```bash
   docker ps
   ```

## 📊 Demostración Interactiva (Dashboard)

El proyecto incluye un panel visual moderno (con diseño Glassmorphism implementado en JavaScript Vanilla y CSS moderno) capaz de contactar al servidor e ilustrar las respuestas visualmente.

1. Navega directamente mediante el explorador de archivos hacia el documento `dashboard.html` contenido dentro de la carpeta raíz y ábrelo usando un navegador moderno (Google Chrome, Firefox, Edge, etc.).
2. Haz clic en **"▶ Iniciar Simulación"**.
3. Observarás cómo tu navegador realiza peticiones iterativas asíncronas (`fetch`) hacia `http://localhost:8080`.
4. En cada petición, uno de los contenedores de los servidores Apache se iluminará, al tiempo que irán contabilizando qué instancia backend acabó absorbiendo dicha conexión y mostrándolo todo en la bitácora local.

*(Nota: Para poder lograr que este comportamiento Cross-Origin resulte funcional desde un protocolo local `file://`, en `nginx.conf` se inyectaron los correspondientes headers `Access-Control-Allow-Origin: *` de modo que el navegador de la demostración no censure los envíos).*

## 💻 Verificación por Consola

Si deseas evidenciar la rotación excluyendo dinámicas asíncronas y omitiendo memorias cachés a nivel browser, es preferible utilizar una utilidad en tu terminal como lo es `curl`.

Ejecuta manual y repetidamente la petición fundamental para ver ciclar el valor HTML:
```bash
curl -s http://localhost:8080
```

Adicionalmente, si prefieres indagar en las entrañas de las tramas TCP respondidas, inspecciona directamente las cabeceras inyectadas:
```bash
curl -I http://localhost:8080
```

Visualizarás (entre el resto del Payload) un header referencial generado puramente por el paso tras el Nginx y añadido para este proyecto:
> `X-Backend-Server: 172.x.x.x:80`

Este último te informará siempre la IP interna de la Subred Docker que finalmente atendió la request del balanceador.
