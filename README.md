# Guía Definitiva de Instalación y Optimización de WebODM en Windows (Workstations de Alto Rendimiento)

Esta guía detalla la instalación, configuración avanzada, optimización de hardware (RAM/CPU/GPU) y solución de errores críticos para desplegar WebODM utilizando Docker y WSL 2 en estaciones de trabajo de alta gama. 

Está diseñada para exprimir al máximo equipos con múltiples núcleos y tarjetas gráficas dedicadas, ideal para procesar grandes volúmenes de imágenes de drones profesionales (como el Quantum-Systems Trinity Pro o DJI Matrice 300 RTK con sensores LiDAR y RGB) aplicados en el monitoreo de áreas naturales y concesiones forestales.

---

## Prerrequisitos y Requisitos del Sistema
* **Sistema Operativo:** Windows 10 o 11 Pro (64 bits).
* **Hardware Recomendado:** Procesadores multinúcleo (ej. Intel Xeon duales), memoria RAM abundante (56 GB - 64 GB+), y GPU dedicada NVIDIA con soporte CUDA (ej. NVIDIA Quadro P4000).
* **Almacenamiento:** Disco SSD NVMe con amplio espacio libre (se recomiendan al menos 50 GB a 100 GB disponibles exclusivamente para la caché de Docker y el procesamiento temporal de ortomosaicos densos).

---

## Fase 1: Instalación de WSL 2 y Docker Desktop desde PowerShell

1. Abre **PowerShell como Administrador** (haz clic derecho en el menú Inicio de Windows y selecciona *Ejecutar como administrador*).
2. Instala el Subsistema de Windows para Linux (WSL), que es el motor base para la virtualización. 
   **Copia y ejecuta el siguiente código en PowerShell:**
   ```powershell
   wsl --install
   ```

3. Instala Docker Desktop utilizando el administrador de paquetes nativo del sistema. 
   **Copia y ejecuta el siguiente código en PowerShell:**
   ```powershell
   winget install Docker.DockerDesktop
   ```

4. **Control de errores comunes (Código de salida 1):** Si el instalador de Docker se interrumpe y arroja el código de salida `1`, el sistema operativo requiere un reinicio obligatorio para aplicar los componentes de WSL. **Reinicia el equipo**. Al encender, aparecerá una consola de Ubuntu donde deberás configurar un nombre de usuario y contraseña de Linux (por seguridad, la contraseña no mostrará caracteres al teclearla).

---

## Fase 2: Habilitar la Virtualización por Hardware en la BIOS/UEFI

Si Docker Desktop muestra el error *"Virtualization support not detected"* al iniciarse, la placa base tiene deshabilitada la virtualización de fábrica.

1. Reinicia la workstation y presiona repetidamente la tecla de acceso a la BIOS (en equipos HP Z-Series Workstation, es la tecla **F10**).
2. Dirígete a la pestaña superior de **Seguridad (Security)** o **Avanzado (Advanced)**, y entra en **Seguridad del Sistema (System Security)**.
3. Asegúrate de marcar y activar las siguientes casillas:
   * **Tecnología de virtualización (VTx)** (Habilitado / Checked).
   * **Tecnología de virtualización para E/S dirigida (VTd)** (Habilitado / Checked).
4. Presiona **F10** para aceptar los cambios, ve a la pestaña Principal y selecciona guardar y salir (*Save Changes and Exit*). El equipo se reiniciará hacia Windows.

---

## Fase 3: Instalación de Git y Clonación del Repositorio

1. Abre una nueva ventana de PowerShell como Administrador e instala Git para Windows. 
   **Copia y ejecuta el siguiente código en PowerShell:**
   ```powershell
   winget install Git.Git
   ```

2. Cierra PowerShell y abre la terminal **Git Bash** (esta es la herramienta recomendada para gestionar comandos de Linux en Windows sin problemas).
3. Clona el repositorio oficial de WebODM directamente en tu carpeta de usuario local. 
   **Copia y ejecuta el siguiente código en Git Bash:**
   ```bash
   git clone https://github.com/WebODM/WebODM --config core.autocrlf=input --depth 1
   ```

4. Entra a la carpeta que acabas de descargar. 
   **Copia y ejecuta el siguiente código en Git Bash:**
   ```bash
   cd WebODM
   ```

---

## Fase 4: Optimización Extrema de Recursos (Archivo `.wslconfig`)

Por defecto, WSL 2 limita severamente el uso de la memoria RAM y los procesadores. Para liberar el potencial de tu workstation, debemos crear un archivo de configuración personalizado. 

**REGLA DE ORO PARA ASIGNAR RECURSOS:**
*   **Memoria (RAM):** Nunca asignes el 100% de tu RAM. Deja siempre entre **4 GB y 8 GB libres** para que Windows no colapse. (Ej: Si tienes 32 GB totales, asigna 24 GB. Si tienes 16 GB totales, asigna 12 GB).
*   **Procesadores (CPU):** Asigna el número máximo de núcleos de tu procesador menos 2. (Ej: Si tienes un procesador de 8 núcleos, asigna 6).

**Pasos para configurarlo:**

1. Presiona las teclas `Windows + R`, escribe `%USERPROFILE%` y presiona Enter para abrir la carpeta de tu usuario (`C:\Users\TuUsuario`).
2. Crea un nuevo documento de texto y nómbralo exactamente **`.wslconfig`** (incluyendo el punto inicial. Confirma el cambio de extensión si Windows lo solicita).
3. Abre el archivo con el Bloc de notas. 
   **Copia y pega el siguiente código adaptando los números a tu equipo (el ejemplo de abajo es para una PC con 64 GB de RAM y 20 núcleos):**
   ```ini
   [wsl2]
   memory=56GB
   processors=18
   swap=128GB
   ```
   *Nota: El valor `swap` (memoria virtual de respaldo en el disco duro) puedes dejarlo en 128GB o 64GB dependiendo del espacio libre en tu disco.*

4. Guarda los cambios del bloc de notas y ciérralo. Para forzar a Windows a leer esta nueva configuración, abre PowerShell. 
   **Copia y ejecuta el siguiente código en PowerShell:**
   ```powershell
   wsl --shutdown
   ```

5. Abre **Docker Desktop** desde el menú de inicio y verifica que la esquina inferior izquierda indique **"Engine running"** en color verde.

5. Abre **Docker Desktop** desde el menú de inicio y verifica que la esquina inferior izquierda indique **"Engine running"** en color verde.

---

## Fase 5: Resolución de Cuelgues por Espacio en Disco

Si durante el procesamiento masivo de imágenes Docker se detiene abruptamente arrojando un error en el archivo `init.log` (*Espacio en disco insuficiente*), el sistema se congela por seguridad.

1. Libera espacio físico en tu unidad de almacenamiento (borra archivos temporales o mueve proyectos antiguos).
2. Cierra Docker haciendo clic derecho en el ícono de la ballena en la bandeja del sistema (junto al reloj) y selecciona **Quit Docker Desktop**.
3. Abre PowerShell para forzar el apagado de cualquier proceso virtual colgado. 
   **Copia y ejecuta el siguiente código en PowerShell:**
   ```powershell
   wsl --shutdown
   ```

4. Vuelve a abrir **Docker Desktop** desde el menú Inicio. El error desaparecerá y el motor volverá a estar operativo.

---

## Fase 6: Arranque de WebODM con Aceleración por GPU (NVIDIA)

Para utilizar los núcleos CUDA de la tarjeta gráfica dedicada (ej. NVIDIA Quadro) y acelerar dramáticamente la generación de nubes de puntos y mallas 3D:

1. Abre tu terminal de **Git Bash**.
2. Entra al directorio del proyecto. 
   **Copia y ejecuta el siguiente código en Git Bash:**
   ```bash
   cd WebODM
   ```

3. Inicia el servidor de WebODM añadiendo el parámetro de aceleración de hardware de video. 
   **Copia y ejecuta el siguiente código en Git Bash:**
   ```bash
   ./webodm.sh start --gpu
   ```
   *(Nota: En la primera ejecución con este parámetro, el sistema descargará los contenedores con los controladores CUDA. Puede demorar varios minutos dependiendo de la conexión).*

4. Cuando la terminal indique que los servicios están corriendo, abre tu navegador web e ingresa a: `http://localhost:8000`

---

## Fase 7: Confirmar y Maximizar el Uso de la GPU en la Interfaz

Para asegurar que los cálculos topográficos pesados se deriven efectivamente a la tarjeta de video:

1. Inicia sesión en la interfaz web de WebODM (`http://localhost:8000`).
2. En el panel izquierdo, selecciona **Processing Nodes** (Nodos de procesamiento).
3. Verifica que el nodo activo (`nodeodm-1`) esté conectado y muestre indicadores de soporte para GPU.
4. Al configurar un nuevo procesamiento de imágenes, despliega el menú de **Opciones (Options)** de la tarea.
5. Selecciona **Edit** para personalizar los parámetros y asegúrate de habilitar funciones de alta demanda (como generación densa de nubes de puntos). El sistema distribuirá automáticamente la carga de trabajo hacia los núcleos CUDA de la GPU, reduciendo drásticamente los tiempos de procesamiento.
