# MIIX 310-ICR

## LENOVO MIIX 310 - 10ICR

### Datos del dispositivo

#### Detalles de funcionalidad

| **Hardware**  | **PCI/USB ID** | **¿Funciona?** |
| ------------- | -------------- | -------------- |
| Teclado       | 258a:1015      | Si             |
| Touchpad      |                | SI             |
| Tactil        |                | Si             |
| GPU           | 8086:22b0      | Si             |
| Pantalla      |                | Depende\*      |
| Webcam        | 8086:22b8      | Sin probar     |
| Audio         |                | Yes            |
| Wifi          | 024c-b723      | Depende\*      |
| Modem         | 12d1:15c1      | Sin probar     |
| Bluetooth     |                | Depende\*      |
| Lector TF/SIM |                | Sin probar     |
| TPM           |                | Si             |
| Rotación      |                | Depende\*      |

\*Depende de la distribución, del kernel y de otros factores. Puede buscar a continuación si tiene solución.

### Ubuntu / Debian / Linux

#### Sensor Movimiento / Rotación de pantalla:

El sensor de movimiento o giroscopio funciona de forma predeterminada en varias distribuciones / kernels, sin embargo, el sensor reporta de forma errónea al sistema operativo sobre su orientación. Para obtener una configuración correcta para el sensor de movimiento dentro de Linux debe aplicar los siguientes cambios:

* Crear archivo / Añadir la siguiente linea al archivo **/etc/udev/hwdb.d/60-sensor.hwdb** :

>

```
sensor:modalias:*
```

```
  ACCEL_MOUNT_MATRIX=0, 1, 0; 1, 0, 0; 0, 0, 1
```

* Ejecutar `sudo systemd-hwdb update`para actualizar la configuración del hardware.
* Reiniciar el sistema.

#### Evitar que la luz de la pantalla se desconecte del sistema:

Para evitar problemas de intermitencia con la pantalla debe hacer lo siguiente:

* Añadir la siguiente linea a **/etc/initramfs-tools/modules** (es un archivo)

>

```
pwm_lpss_platform
```

* Actualizar initramfs con `sudo update-initramfs -k all -u`
* Reiniciar el sistema.

#### Configuración de GRUB

Para solución de problemas o solución con la rotación de la pantalla en GRUB o en el modo texto puede usar las siguientes soluciones:

* Edite el archivo **/etc/default/grub** y añada a la linea **GRUB\_CMDLINE\_LINUX\_DEFAULT** una de las siguientes opciones dependiendo de sus necesidades (mezcle las opciones de ser necesario):

>

```
quiet splash fbcon=rotate:1 video=efifbh
```

>

```
video=DSI-1:800x1280e acpi_osi= i915.modeset=1 fbcon=rotate:1 video.use_native_backlight=1 i915.enable_fbc=1 i915.enable_rc6=1 i915.semaphores=1 nospalsh quiet
```

Esta es especifica si tiene problemas para tener una salida de monitor:

>

```
pwm-lpss pwm-lpss-platform
```

* Ejecutar `sudo update-grub` para actualizar su configuración de GRUB
* Reinicie el sistema.

#### Extras de pantalla:

Si desea rotar la pantalla puede usar uno de los siguientes comandos ( Solo para Xorg):

>

```
xrandr -o right
```

>

```
xrandr --output DSI-1 --rotate right
```

Es recomendable usarlo junto a un script de arranque que sea soportado por su gestor de ventanas en Linux, por ejemplo:

Para Linux mint 21 usando Xorg y lightdm:

* Cree un archivo de configuración para lightdm usando: `sudo nano /etc/lightdm/lightdm.conf.d/80-display-setup.conf`
* Indexe la siguiente configuración bajo la sección **\[SeatDefaults]** quedando de la siguiente manera:

>

```
[SeatDefaults]
```

```
# Otras configuraciones si es que hay
display-setup-script=xrandr -o right
```

* Reinicie su sitema.

Si tiene problemas donde su pantalla se congela, puede intentar lo siguiente (Solo para Xorg):

* Añadir o crear en la configuración de gráficos intel: `sudo nano /etc/X11/xorg.conf.d/20-intel-graphics.conf`:

>

```
Section "Module"
```

```
    Load "dri3"
EndSection

Section "Device"
    Identifier  "Intel Graphics"
    Driver      "intel"
    Option      "DRI"   "3"
EndSection
```

* Reinicie su dispositivo.

Luego de reiniciar, la aceleración por GPU funcionará mejor, no tendrá "screen tearing" y su dispositivo no se congelará mas. Podrá ver las resoluciones reales del dispositivo en su gestor de pantallas.

**Advertencia:**

Activar DRI 3 en su grafica aumentara la latencia de entrada en sus aplicaciones 3D

#### Otros extras:

* Dispositivos que requieren versiones del kernel especificas o drivers adicionales:

>

```
 rtl8723bs | Wifi y Bluetooth | Funcional bajo kernel 5.15.0-75-generic
```

### Fuentes

Estas son las fuentes que ayudaron a solucionar los problemas o a guiar parte de la solución:

* https://forums.lenovo.com/t5/Ubuntu/Xubuntu-successfully-installed-on-Miix-310-10ICR-everything-but-camera-works/m-p/5208218
* https://gist.github.com/dotsh/7e647e2f4fec06d085a6902ea0f0a4cb
* https://wiki.archlinux.org/title/Lenovo\_IdeaPad\_Miix\_310-10ICR
* https://askubuntu.com/questions/1072645/screen-rotation-on-ubuntu-18-04-lts-lenovo-ideapad-miix-510
* https://community.intel.com/t5/Graphics/Difficulty-Enabling-DRI3-on-Debian-bullseye/td-p/1344602?profile.language=es\&countrylabel=Argentina
