# Desplegando plantillas de Azure

## Introducción
En este laboratorio práctico utilizaremos el **portal de Azure** para **modificar una plantilla QuickStart** y agregar un disco de datos adicional antes de su implementación.  
Posteriormente, desplegaremos la plantilla y confirmaremos que los cambios realizados se aplicaron correctamente.

Tu jefe te ha pedido que implementes una **máquina virtual Windows Server 2019**, pero necesitas modificar la plantilla para agregarle un **disco de datos adicional**.

---

## Solución

### Conectarse al portal de Azure con la cuenta del laboratorio
1. Inicia sesión en el **portal de Azure** usando las credenciales proporcionadas en la página de instrucciones del laboratorio.

---

### Usar la implementación de plantillas y editar la plantilla

1. Haz clic en **+ Create (Crear)**.  
2. En la barra de búsqueda, escribe **template** y selecciona **Template deployment (deploy using custom templates)**.  
3. Haz clic en **Create > Template deployment (deploy using custom templates)**.  
4. Selecciona **Create a Windows virtual machine**.  
5. Asegúrate de que el **grupo de recursos** proporcionado esté seleccionado en la lista desplegable.  
6. Haz clic en **Edit template (Editar plantilla)**.

Busca la sección **"dataDisks"** (alrededor de la línea 290) y actualízala con lo siguiente.  
Recuerda indentar `"dataDisks"` para que quede alineado con `"osDisk"`:

```json
"dataDisks": [
  {
    "diskSizeGB": 1023,
    "lun": 0,
    "createOption": "Empty"
  },
  {
    "diskSizeGB": 1023,
    "lun": 1,
    "createOption": "Empty"
  }
]
```

Luego, encuentra la sección **"vmSize"** (alrededor de la línea 130) y cambia el valor de `"defaultValue"` a `"Standard_D2s_v3"` (con comillas).

Haz clic en **Save (Guardar)**.

---

### Editar los parámetros

1. Haz clic en **Edit parameters (Editar parámetros)**.  
2. En `"adminUsername"`, cambia el valor (`"value"`) a `"localadmin"`.  
3. En `"dnsLabelPrefix"`, cambia el valor a algo único (con comillas).  
   - Ejemplo: `"vm1446aaadd"` donde `aaa` son tus iniciales y `dd` una fecha de tu elección.  
4. Haz clic en **Save (Guardar)**.

---

### Crear la máquina virtual

1. En el campo **Admin Password**, introduce una contraseña segura.  
2. Haz clic en **Review + create (Revisar + crear)**.  
3. Luego, haz clic en **Create (Crear)**.

---

### Verificar la implementación

1. Haz clic en la **campana de notificaciones** (arriba a la derecha) para ver el estado del despliegue.  
2. Haz clic en el estado para abrir la máquina virtual.  
3. Una vez completado el despliegue, haz clic en **Home (Inicio)**.  
4. Selecciona **All resources (Todos los recursos)**.  
5. Haz clic en la máquina virtual **simple-vm**.  
6. En **Settings → Disks**, verifica que existan **dos discos de datos**: **LUN 0** y **LUN 1**.

---

## Conclusión
🎉 ¡Felicidades! Has completado este laboratorio práctico sobre **despliegue de plantillas de Azure (ARM)** y confirmaste cómo modificar una plantilla para incluir discos adicionales antes de su implementación.


# Desplegar recursos utilizando una plantilla ARM en el portal de Azure

## Introducción
Las **plantillas de Azure Resource Manager (ARM)** proporcionan una forma potente de definir recursos y configuraciones de Azure, básicamente mediante un archivo de texto.  
Esto ayuda a mantener la **consistencia y automatización** en los despliegues de recursos.  
En este laboratorio práctico aprenderás a **crear una cuenta de almacenamiento** usando una plantilla ARM y a **modificar la plantilla** para ver cómo funcionan los parámetros y las variables.

---

## Solución

### Iniciar sesión
Inicia sesión en el **portal de Azure** utilizando las credenciales proporcionadas para el laboratorio.

---

## Cargar la plantilla ARM usando el portal de Azure

1. Descarga la plantilla ARM a tu computadora o copia el texto de la plantilla desde el repositorio de GitHub indicado [aquí](./labstoragetemplate.json).  
2. En el portal de Azure, haz clic en **Create a resource (Crear un recurso)**.  
3. En la barra de búsqueda, escribe **“template”** y selecciona la opción **Template deployment (deploy using custom templates)** del menú contextual.  
4. Haz clic en **Create (Crear)**.  
5. En la página **Custom deployment (Implementación personalizada)**, haz clic en **Build your own template in the editor (Crea tu propia plantilla en el editor)**.  
6. En la página **Edit template (Editar plantilla)**, pega el texto de la plantilla copiado o haz clic en **Load file (Cargar archivo)** y selecciona el archivo **labstoragetemplate.json** descargado en el primer paso.

---

## Modificar los parámetros y variables de la plantilla ARM

### Agregar un parámetro para el tipo de cuenta de almacenamiento y modificar la definición del recurso

1. En la sección **parameters** de la plantilla, copia el parámetro `location` y pégalo directamente debajo.  
2. Cambia el nombre del parámetro `location` a **storageAccountKind**.  
3. Establece el valor por defecto (`defaultValue`) en **StorageV2**.  
4. En la sección **resources**, copia el valor `[parameters('location')]` de la propiedad `location` y pégalo como valor para la propiedad `kind`.  
5. En la propiedad `kind`, cambia el valor a `[parameters('storageAccountKind')]`.

---

### Crear una variable para generar un nombre único de cuenta de almacenamiento

1. Por encima de la sección **resources**, agrega una nueva sección llamada **variables**.  
2. Agrega una nueva variable llamada **uniqueStorageAccountName**.  
3. Asígnale el valor:  
   ```json
   [concat(parameters('storageAccountName'), uniqueString(resourceGroup().id))]
   ```  
   Esto combina el parámetro `storageAccountName` con una cadena única de 13 caracteres basada en el ID del grupo de recursos.  
4. En la sección **resources**, cambia el valor de la propiedad `name` a `variables('uniqueStorageAccountName')`.  
5. **Nota:** asegúrate de agregar una **coma final** al final de la configuración de variables (después de la llave de cierre) para que sea válida.

---

### Fijar la ubicación (hard-code) del recurso de almacenamiento

1. En la sección **resources**, cambia el valor de la propiedad `location` a **West US**.  
2. En la sección **parameters**, elimina el código correspondiente al parámetro `location`:

   ```json
   "location": {
     "type": "string",
     "defaultValue": "[resourceGroup().location]"
   },
   ```
3. Haz clic en **Save (Guardar)** para guardar los cambios en la plantilla.

---

## Desplegar la plantilla ARM utilizando el archivo de parámetros

1. En la página **Custom deployment**, haz clic en **Edit parameters (Editar parámetros)**.  
2. En el archivo de parámetros:
   - Establece el valor del parámetro `storageAccountName` en **azurelab**.  
   - Deja el valor del parámetro `storageAccountKind` como **StorageV2**.  
3. Haz clic en **Save (Guardar)**.  
4. Verifica que los valores que configuraste ahora aparezcan en los campos **Storage Account Name** y **Storage Account Kind** bajo la sección **Parameters**.  
5. Deja el campo **Subscription** con la selección predeterminada (la suscripción del laboratorio).  
6. En el menú desplegable **Resource group**, selecciona el grupo de recursos existente provisto para el laboratorio.  
7. Observa que, una vez seleccionados la suscripción y el grupo de recursos, la **región se actualiza a West US** y no puede modificarse.  
8. Haz clic en **Review + create (Revisar + crear)**.  
9. Cuando la validación haya pasado, haz clic en **Create (Crear)**.  
10. Una vez completado el despliegue, haz clic en **Go to resource (Ir al recurso)**.

---

## Verificación
Observa los siguientes valores definidos en la plantilla y el archivo de parámetros:

- El **nombre de la cuenta de almacenamiento** debe ser `azurelab` con una cadena única agregada al final.  
- La **ubicación** es **West US**.  
- El **tipo de cuenta (Account kind)** es **StorageV2**.

---

## Conclusión
¡Felicidades! Has completado este laboratorio práctico sobre el uso de **plantillas ARM para desplegar recursos en Azure**.
