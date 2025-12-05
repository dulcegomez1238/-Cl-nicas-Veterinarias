# -Cl-nicas-Veterinarias
Sistema de Gestión de Clínicas Veterinarias
<img width="855" height="471" alt="image" src="https://github.com/user-attachments/assets/f058e7aa-adb4-44d6-a4ee-3e7de4afc33e" />




##Modelos de Base de Datos - Sistema Veterinario
python
from django.db import models
from django.core.validators import MinValueValidator, MaxValueValidator
from django.utils import timezone

    ##class Propietario(models.Model):
    """Modelo para almacenar información de los propietarios de mascotas"""
    id_propietario = models.AutoField(primary_key=True)
    nombre = models.CharField(max_length=100)
    apellido = models.CharField(max_length=100)
    direccion = models.CharField(max_length=255)
    telefono = models.CharField(max_length=20)
    email = models.EmailField(max_length=100)
    fecha_registro = models.DateField(default=timezone.now)
    dni = models.CharField(max_length=20, unique=True)
    ocupacion = models.CharField(max_length=100, blank=True, null=True)
    
    class Meta:
        verbose_name = "Propietario"
        verbose_name_plural = "Propietarios"
        ordering = ['apellido', 'nombre']
    
    def __str__(self):
        return f"{self.nombre} {self.apellido} - DNI: {self.dni}"


    ##class Veterinario(models.Model):
    """Modelo para almacenar información de los veterinarios"""
    id_veterinario = models.AutoField(primary_key=True)
    nombre = models.CharField(max_length=100)
    apellido = models.CharField(max_length=100)
    especialidad = models.CharField(max_length=100)
    telefono = models.CharField(max_length=20)
    email = models.EmailField(max_length=100)
    licencia_veterinaria = models.CharField(max_length=50, unique=True)
    fecha_contratacion = models.DateField(default=timezone.now)
    salario = models.DecimalField(max_digits=10, decimal_places=2, validators=[MinValueValidator(0)])
    
    class Meta:
        verbose_name = "Veterinario"
        verbose_name_plural = "Veterinarios"
        ordering = ['apellido', 'nombre']
    
    def __str__(self):
        return f"Dr./Dra. {self.nombre} {self.apellido} - {self.especialidad}"


    ##class Mascota(models.Model):
    """Modelo para almacenar información de las mascotas"""
    
    GENERO_OPCIONES = [
        ('M', 'Macho'),
        ('H', 'Hembra'),
    ]
    
    ESPECIE_OPCIONES = [
        ('PERRO', 'Perro'),
        ('GATO', 'Gato'),
        ('AVE', 'Ave'),
        ('ROEDOR', 'Roedor'),
        ('REPTIL', 'Reptil'),
        ('OTRO', 'Otro'),
    ]
    
    id_mascota = models.AutoField(primary_key=True)
    nombre_mascota = models.CharField(max_length=100)
    especie = models.CharField(max_length=50, choices=ESPECIE_OPCIONES)
    raza = models.CharField(max_length=50)
    fecha_nacimiento = models.DateField()
    genero = models.CharField(max_length=1, choices=GENERO_OPCIONES)
    peso_kg = models.DecimalField(
        max_digits=5, 
        decimal_places=2,
        validators=[MinValueValidator(0), MaxValueValidator(300)]
    )
    id_propietario = models.ForeignKey(
        Propietario, 
        on_delete=models.CASCADE,
        related_name='mascotas'
    )
    chip_identificacion = models.CharField(max_length=50, unique=True, blank=True, null=True)
    color = models.CharField(max_length=50)
    esterilizado = models.BooleanField(default=False)
    
    class Meta:
        verbose_name = "Mascota"
        verbose_name_plural = "Mascotas"
        ordering = ['nombre_mascota']
    
    def __str__(self):
        return f"{self.nombre_mascota} ({self.get_especie_display()})"
    
    def calcular_edad(self):
        """Calcula la edad de la mascota en años"""
        hoy = timezone.now().date()
        edad = hoy.year - self.fecha_nacimiento.year
        if (hoy.month, hoy.day) < (self.fecha_nacimiento.month, self.fecha_nacimiento.day):
            edad -= 1
        return edad


    class Vacuna(models.Model):
    """Modelo para almacenar información de las vacunas disponibles"""
    id_vacuna = models.AutoField(primary_key=True)
    nombre_vacuna = models.CharField(max_length=100)
    descripcion = models.TextField()
    laboratorio = models.CharField(max_length=100)
    fecha_vencimiento_lote = models.DateField()
    tipo_enfermedad = models.CharField(max_length=100)
    
    class Meta:
        verbose_name = "Vacuna"
        verbose_name_plural = "Vacunas"
        ordering = ['nombre_vacuna']
    
    def __str__(self):
        return f"{self.nombre_vacuna} - {self.laboratorio}"
    
    def esta_vencida(self):
        """Verifica si la vacuna está vencida"""
        return timezone.now().date() > self.fecha_vencimiento_lote


    class Consulta(models.Model):
    """Modelo para almacenar información de las consultas veterinarias"""
    id_consulta = models.AutoField(primary_key=True)
    id_mascota = models.ForeignKey(
        Mascota, 
        on_delete=models.CASCADE,
        related_name='consultas'
    )
    id_veterinario = models.ForeignKey(
        Veterinario, 
        on_delete=models.CASCADE,
        related_name='consultas'
    )
    fecha_consulta = models.DateTimeField(default=timezone.now)
    motivo_consulta = models.TextField()
    diagnostico = models.TextField()
    tratamiento = models.TextField()
    peso_mascota_consulta = models.DecimalField(
        max_digits=5, 
        decimal_places=2,
        validators=[MinValueValidator(0), MaxValueValidator(300)]
    )
    observaciones = models.TextField(blank=True, null=True)
    
    class Meta:
        verbose_name = "Consulta"
        verbose_name_plural = "Consultas"
        ordering = ['-fecha_consulta']
    
    def __str__(self):
        return f"Consulta {self.id_consulta} - {self.id_mascota.nombre_mascota} - {self.fecha_consulta.date()}"
    
    def get_peso_variacion(self):
        """Calcula la variación de peso respecto al peso actual de la mascota"""
        if self.id_mascota.peso_kg:
            return float(self.peso_mascota_consulta) - float(self.id_mascota.peso_kg)
        return 0


    class HistorialVacunacion(models.Model):
    """Modelo para almacenar el historial de vacunación de las mascotas"""
    id_historial_vac = models.AutoField(primary_key=True)
    id_mascota = models.ForeignKey(
        Mascota, 
        on_delete=models.CASCADE,
        related_name='historial_vacunacion'
    )
    id_vacuna = models.ForeignKey(
        Vacuna, 
        on_delete=models.CASCADE,
        related_name='aplicaciones'
    )
    fecha_aplicacion = models.DateField(default=timezone.now)
    fecha_proxima_dosis = models.DateField()
    id_veterinario_aplico = models.ForeignKey(
        Veterinario, 
        on_delete=models.CASCADE,
        related_name='vacunas_aplicadas'
    )
    numero_lote = models.CharField(max_length=50)
    comentarios = models.TextField(blank=True, null=True)
    
    class Meta:
        verbose_name = "Historial de Vacunación"
        verbose_name_plural = "Historiales de Vacunación"
        ordering = ['-fecha_aplicacion']
        unique_together = ['id_mascota', 'id_vacuna', 'fecha_aplicacion']
    
    def __str__(self):
        return f"{self.id_vacuna.nombre_vacuna} - {self.id_mascota.nombre_mascota} - {self.fecha_aplicacion}"
    
    def proxima_dosis_proxima(self):
        """Verifica si la próxima dosis está próxima (en los próximos 30 días)"""
        hoy = timezone.now().date()
        diferencia = (self.fecha_proxima_dosis - hoy).days
        return 0 <= diferencia <= 30

 
    class FacturaVeterinaria(models.Model):
    """Modelo para almacenar información de las facturas"""
    
    ESTADO_PAGO_OPCIONES = [
        ('PENDIENTE', 'Pendiente'),
        ('PAGADO', 'Pagado'),
        ('CANCELADO', 'Cancelado'),
        ('VENCIDO', 'Vencido'),
    ]
    
    METODO_PAGO_OPCIONES = [
        ('EFECTIVO', 'Efectivo'),
        ('TARJETA', 'Tarjeta'),
        ('TRANSFERENCIA', 'Transferencia'),
        ('CHEQUE', 'Cheque'),
    ]
    
    id_factura = models.AutoField(primary_key=True)
    id_propietario = models.ForeignKey(
        Propietario, 
        on_delete=models.CASCADE,
        related_name='facturas'
    )
    fecha_emision = models.DateField(default=timezone.now)
    total_factura = models.DecimalField(
        max_digits=10, 
        decimal_places=2,
        validators=[MinValueValidator(0)]
    )
    estado_pago = models.CharField(
        max_length=50, 
        choices=ESTADO_PAGO_OPCIONES,
        default='PENDIENTE'
    )
    metodo_pago = models.CharField(
        max_length=50, 
        choices=METODO_PAGO_OPCIONES,
        blank=True, 
        null=True
    )
    id_consulta_asociada = models.OneToOneField(
        Consulta,
        on_delete=models.SET_NULL,
        related_name='factura',
        blank=True,
        null=True
    )
    fecha_vencimiento = models.DateField()
    
    class Meta:
        verbose_name = "Factura Veterinaria"
        verbose_name_plural = "Facturas Veterinarias"
        ordering = ['-fecha_emision']
    
    def __str__(self):
        return f"Factura {self.id_factura} - {self.id_propietario.nombre} - ${self.total_factura}"
    
    def esta_vencida(self):
        """Verifica si la factura está vencida"""
        hoy = timezone.now().date()
        return hoy > self.fecha_vencimiento and self.estado_pago == 'PENDIENTE'
    
    def dias_vencimiento(self):
        """Calcula los días hasta el vencimiento"""
        hoy = timezone.now().date()
        return (self.fecha_vencimiento - hoy).days
##Instalación y Configuración
Requisitos previos:

bash
pip install django
Agregar al settings.py:

python
INSTALLED_APPS = [
    ...
    'veterinaria',  # Nombre de tu aplicación
    ...
]

# Configuración de base de datos (ejemplo para PostgreSQL)
DATABASES = {
    'default': {
        'ENGINE': 'django.db.backends.postgresql',
        'NAME': 'veterinaria_db',
        'USER': 'tu_usuario',
        'PASSWORD': 'tu_contraseña',
        'HOST': 'localhost',
        'PORT': '5432',
    }
}
Ejecutar migraciones:

bash
python manage.py makemigrations
python manage.py migrate
Características del Modelo
✅ Relaciones implementadas:
Mascota → Propietario (ForeignKey)

Consulta → Mascota (ForeignKey)

Consulta → Veterinario (ForeignKey)

HistorialVacunacion → Mascota (ForeignKey)

HistorialVacunacion → Vacuna (ForeignKey)

HistorialVacunacion → Veterinario (ForeignKey)

FacturaVeterinaria → Propietario (ForeignKey)

FacturaVeterinaria → Consulta (OneToOneField)

✅ Validaciones incluidas:
Campos únicos: DNI, licencia veterinaria, chip

Validadores de rango para peso y salario

Elecciones predefinidas para género, especie, estado de pago

Fechas automáticas donde corresponde

✅ Métodos útiles:
Cálculo de edad de mascotas

Verificación de vencimientos

Cálculo de variación de peso

Validación de próximas dosis

✅ Configuraciones de Django:
Nombres plurales y singulares en español

Ordenamiento por defecto

Claves primarias autoincrementales

Relaciones con related_name para acceso inverso

Personalización Adicional
Puedes agregar los siguientes campos según necesidades específicas:

python
# En el modelo Mascota
foto = models.ImageField(upload_to='mascotas/', blank=True, null=True)
alergias = models.TextField(blank=True, null=True)

# En el modelo Propietario
notificaciones_email = models.BooleanField(default=True)

# En el modelo FacturaVeterinaria
iva = models.DecimalField(max_digits=5, decimal_places=2, default=21.00)
