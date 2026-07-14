# 🦊 ROK Tauri YouTube Downloader ⚡🤖

Un cliente de escritorio ultra elegante y de alto rendimiento para descargar videos y audio de YouTube. Desarrollado con la velocidad de **Rust (Tauri v2)**, un **sidecar de Python** personalizado y una interfaz web moderna y responsiva[cite: 3].

Este proyecto utiliza el patrón seguro **Sidecar** de Tauri para delegar las tareas pesadas de extracción y procesamiento de video a un backend local de Python compilado, manteniendo la aplicación principal sumamente ligera y rápida[cite: 3].

---

## 📐 Arquitectura y Funcionamiento

┌──────────────────────────────────────┐
│          Frontend (HTML/CSS/JS)      │  <-- Interfaz Cyberpunk con estilo
└──────────────────┬───────────────────┘
│  (Tauri IPC / Comandos)
┌──────────────────▼───────────────────┐
│          Backend en Rust (Tauri)     │  <-- Orquestador y Gestor de Ventanas Nativo
└──────────────────┬───────────────────┘
│  (Ejecución del Sidecar mediante std::process)
┌──────────────────▼───────────────────┐
│     Binario Sidecar de Python (.exe) │  <-- Extractor y descargador compilado
└──────────────────┬───────────────────┘
│  (Llamadas a Subprocesos)
┌──────────────────▼───────────────────┐
│         FFmpeg y FFprobe (.exe)      │  <-- Mezclado de flujos de Audio/Video (Muxing)
└──────────────────────────────────────┘

Fragmento de código

---

## 🛠️ Requisitos del Sistema

Antes de compilar o ejecutar el proyecto en modo de desarrollo, asegúrate de tener instalado lo siguiente en tu sistema:

*   **Rust (MSRV 1.75+)**: Instalado a través de [rustup](https://rustup.rs/)[cite: 3].
*   **Python 3.10+**: Necesario únicamente para compilar el script descargador[cite: 3].
*   **Herramientas de compilación de Microsoft C++**: Requerido por Rust para compilar binarios nativos de Windows[cite: 3].

---

## 🚀 Cómo Generar los Ejecutables (.exe)

Por motivos de seguridad, optimización de peso y para respetar el límite de almacenamiento, los archivos ejecutables binarios (`.exe`) se encuentran excluidos del repositorio de Git. Debes generarlos o colocarlos manualmente en sus respectivas rutas antes de compilar la aplicación[cite: 3].

### 1. Compilar el Sidecar de Python (`descargador-yt.py`)

Tauri requiere que los binarios externos (sidecars) sigan una convención de nomenclatura estricta que incluya la "triple diana" del sistema objetivo (por ejemplo, `x86_64-pc-windows-msvc` para Windows de 64 bits)[cite: 3].

Para compilar tu script de Python en un ejecutable independiente:

1.  Abre la terminal en la carpeta raíz del proyecto[cite: 3].
2.  Instala **PyInstaller** si aún no lo tienes:
    ```bash
    pip install pyinstaller
    ```[cite: 3]
3.  Navega a la carpeta de binarios y ejecuta PyInstaller:
    ```bash
    cd src-tauri/binaries
    pyinstaller --onefile --noconsole --name descargador-yt-x86_64-pc-windows-msvc descargador-yt.py
    ```[cite: 3]
4.  Una vez finalizada la compilación:
    *   Ve a la carpeta temporal `src-tauri/binaries/dist/`[cite: 3].
    *   Mueve el archivo generado `descargador-yt-x86_64-pc-windows-msvc.exe` directamente a la carpeta raíz de binarios: `src-tauri/binaries/`[cite: 3].
    *   Puedes borrar con total seguridad las carpetas temporales `build/`, `dist/` y el archivo `.spec` creados por PyInstaller[cite: 3].

### 2. Configurar los Recursos de FFmpeg y FFprobe

Para poder unir flujos de video de alta calidad (1080p, 4K) con sus respectivas pistas de audio sin perder definición, la aplicación necesita las herramientas de **FFmpeg**[cite: 3].

1.  Descarga la compilación estática de FFmpeg para Windows desde un sitio oficial (como [Gyan.dev](https://www.gyan.dev/ffmpeg/builds/))[cite: 3].
2.  Descomprime el archivo y localiza los ejecutables `ffmpeg.exe` y `ffprobe.exe` dentro de la carpeta `bin/`[cite: 3].
3.  Copia y pega ambos archivos dentro del directorio de recursos de tu proyecto:
    ```text
    src-tauri/resources/ffmpeg.exe
    src-tauri/resources/ffprobe.exe
    ```[cite: 3]

---

## 💻 Desarrollo y Compilación

Una vez que tengas todos los binarios necesarios en sus carpetas correspondientes, puedes ejecutar la aplicación de forma local o empaquetarla para su distribución usando **Cargo** sin depender de Node.js ni npm[cite: 3].

### Ejecutar en Modo de Desarrollo
```bash
# Iniciar Tauri en modo de desarrollo con recarga en vivo
cargo tauri dev
```[cite: 3]

### Compilar la Versión de Producción
```bash
# Compila todo el código en un instalador optimizado y ligero (.msi / .exe)
cargo tauri build
```[cite: 3]
El instalador generado se ubicará en la ruta `src-tauri/target/release/bundle/`[cite: 3].

---

## 🔮 Ruta de Desarrollo Futuro (Expansión Multiplataforma)

Para transformar esta herramienta local en un software compatible con múltiples sistemas operativos, se han planteado los siguientes objetivos de desarrollo:

### 🐧 1. Compatibilidad con Linux
*   **Compilación de Sidecars**: Compilar el sidecar de Python dentro de un entorno Linux usando PyInstaller para apuntar al objetivo `x86_64-unknown-linux-gnu`:
    ```bash
    pyinstaller --onefile --name descargador-yt-x86_64-unknown-linux-gnu descargador-yt.py
    ```[cite: 3]
*   **Dependencias del Sistema**: Sustituir los recursos estáticos de FFmpeg por una verificación local mediante los gestores de paquetes del sistema (`apt`, `pacman`, `dnf`) para consumir el comando global `ffmpeg`[cite: 3].
*   **Empaquetado**: Generar paquetes instalables nativos como archivos `.deb` o `.AppImage`[cite: 3].

### 🤖 2. Adaptación a Android (Tauri v2 Mobile)
*   **Cambio Arquitectónico**: Los dispositivos móviles Android no pueden ejecutar directamente archivos `.exe` de escritorio. La lógica de extracción escrita en Python se migrará hacia:
    *   Un módulo embebido en **Rust** usando bindings nativos (como la librería `pyo3` si se requiere conservar Python estrictamente)[cite: 3].
    *   O bien, reescribir toda la lógica de obtención directamente en **módulos nativos de Rust** (utilizando librerías de red y concurrencia como `tokio` y procesadores de HTTP personalizados)[cite: 3].
*   **Integración del NDK de Android**: Compilar el motor de procesamiento multimedia adaptando los targets nativos de Android (`aarch64-linux-android`, `armv7-linux-androideabi`)[cite: 3].
*   **Optimización de Interfaz**: Adaptar el panel de control Cyberpunk para que sea completamente responsivo y cómodo en pantallas táctiles y móviles[cite: 3].

---

## 📜 Licencia e Intención

Desarrollado como una aplicación de escritorio local, rápida y segura. Bajo una optimización constante guiada por un enfoque de desarrollo activo y libre, disfrutando de la creación pura de software sin límites[cite: 3]. 

*Creado por Roberto Ricardo Araujo Ari - ROK.*[cite: 3]