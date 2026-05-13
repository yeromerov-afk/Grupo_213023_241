# Grupo_213023_241
Grupo de trabajo solución ejercicio 
**import logging
from abc import ABC, abstractmethod

# 1. CONFIGURACIÓN DE LOGS (Archivo de registro de eventos y errores)
# Se registrarán todos los errores para mantener la aplicación activa [cite: 18, 31]
logging.basicConfig(

    filename='errores_sistema.log', 
    level=logging.ERROR, 
    format='%(asctime)s - %(levelname)s - %(message)s'
)

# 2. EXCEPCIONES PERSONALIZADAS [cite: 17, 19]
class SoftwareFJException(Exception):
    """Clase base para excepciones del sistema."""
    pass

class DatosInvalidosError(SoftwareFJException):
    """Se lanza cuando los datos de entrada no cumplen las validaciones."""
    pass

class ReservaInvalidaError(SoftwareFJException):
    """Se lanza ante intentos de reserva incorrectos o fallidos."""
    pass

# 3. ARQUITECTURA DE CLASES (POO) [cite: 20]

class EntidadBase(ABC):
    """Clase abstracta que representa entidades generales[cite: 21]."""
    @abstractmethod
    def obtener_detalles(self):
        pass

class Cliente(EntidadBase):
    """Clase Cliente con encapsulación y validaciones[cite: 22]."""
    def __init__(self, id_cliente, nombre, email):
        if not email or "@" not in email:
            raise DatosInvalidosError(f"Email inválido para el cliente: {nombre}")
        self.__id = id_cliente  # Atributo privado (Encapsulación)
        self.__nombre = nombre
        self.__email = email

    def obtener_detalles(self):
        return f"ID: {self.__id} | Nombre: {self.__nombre} | Email: {self.__email}"

    @property
    def nombre(self):
        return self.__nombre

class Servicio(ABC):
    """Clase abstracta Servicio[cite: 23]."""
    def __init__(self, nombre_servicio, costo_base):
        self.nombre_servicio = nombre_servicio
        self.costo_base = costo_base

    @abstractmethod
    def calcular_costo(self, cantidad):
        """Método para polimorfismo[cite: 24]."""
        pass

    def describir(self):
        return f"Servicio: {self.nombre_servicio}"

# 4. IMPLEMENTACIÓN DE SERVICIOS ESPECIALIZADOS (Herencia y Polimorfismo) [cite: 23, 24]

class ReservaSala(Servicio):
    def calcular_costo(self, horas, recargo_limpieza=50):
        # Ejemplo de método con parámetros opcionales/sobrecargados [cite: 26]
        if horas <= 0:
            raise DatosInvalidosError("Las horas de reserva deben ser mayores a cero.")
        return (self.costo_base * horas) + recargo_limpieza

class AlquilerEquipo(Servicio):
    def calcular_costo(self, dias):
        if dias <= 0:
            raise DatosInvalidosError("Los días de alquiler deben ser positivos.")
        return self.costo_base * dias

class AsesoriaEspecializada(Servicio):
    def calcular_costo(self, sesiones, es_urgente=False):
        # Lógica polimórfica con condicionales [cite: 24, 26]
        total = self.costo_base * sesiones
        if es_urgente:
            total *= 1.20 # 20% de recargo
        return total

# 5. GESTIÓN DE RESERVAS [cite: 25]

class Reserva:
    def __init__(self, cliente, servicio, cantidad):
        self.cliente = cliente
        self.servicio = servicio
        self.cantidad = cantidad
        self.estado = "Pendiente"

    def procesar(self):
        print(f"--- Procesando reserva para {self.cliente.nombre} ---")
        try:
            # Uso de bloques try/except/else/finally [cite: 17]
            costo_final = self.servicio.calcular_costo(self.cantidad)
        except DatosInvalidosError as e:
            self.estado = "Fallida"
            logging.error(f"Falla en reserva: {e}")
            print(f"ERROR: {e}")
            raise ReservaInvalidaError("No se pudo confirmar la reserva por datos inconsistentes.")
        except Exception as e:
            self.estado = "Error Técnico"
            logging.error(f"Error inesperado: {e}")
            print("Ha ocurrido un error inesperado en el sistema.")
        else:
            self.estado = "Confirmada"
            print(f"ÉXITO: Reserva de '{self.servicio.nombre_servicio}' procesada.")
            print(f"Total a pagar: ${costo_final}")
        finally:
            print(f"Estado final del proceso: {self.estado}\n")

# 6. SIMULACIÓN DE OPERACIONES (10 casos: válidos e inválidos) [cite: 32]

def ejecutar_simulacion():
    print("SISTEMA INTEGRAL DE GESTIÓN - SOFTWARE FJ\n" + "="*40)
    
    # Listas internas para gestión [cite: 12, 13]
    servicios_disponibles = [
        ReservaSala("Sala de Juntas", 100),
        AlquilerEquipo("Laptop Core i7", 45),
        AsesoriaEspecializada("Consultoría Cloud", 200)
    ]
    
    # Datos de prueba (Mezcla de válidos y erróneos)
    casos_prueba = [
        # (Nombre, Email, IndiceServicio, Cantidad)
        ("Juan Perez", "juan@mail.com", 0, 3),      # Válido
        ("Ana Rios", "ana_at_mail.com", 1, 2),      # Inválido: Email sin @ [cite: 19]
        ("Luis Gomez", "luis@mail.com", 0, -1),     # Inválido: Horas negativas [cite: 19]
        ("Maria Luz", "maria@mail.com", 2, 5),      # Válido
        ("Carlos Paz", "carlos@mail.com", 1, 0),    # Inválido: Días cero
        ("Sofía Mar", "sofia@mail.com", 0, 10),     # Válido
        ("Pedro Pic", "pedro@mail.com", 2, 1),      # Válido
        ("Invalido", "", 0, 5),                    # Inválido: Email vacío
        ("Jose Jose", "jose@mail.com", 1, 7),      # Válido
        ("Elena No", "elena@mail.com", 2, -5),     # Inválido: Sesiones negativas
    ]

    for nombre, email, idx_srv, cant in casos_prueba:
        try:
            # 1. Crear Cliente
            user = Cliente(101, nombre, email)
            # 2. Crear Reserva
            res = Reserva(user, servicios_disponibles[idx_srv], cant)
            # 3. Procesar
            res.procesar()
        except (DatosInvalidosError, ReservaInvalidaError) as e:
            logging.error(f"Operación fallida para {nombre}: {e}")
            print(f"AVISO: La operación para {nombre} no pudo completarse. Revise el log.\n")
        except Exception as e:
            logging.error(f"Error crítico: {e}")
            print("Error grave detectado. El sistema continúa operando...\n")

if __name__ == "__main__":
    ejecutar_simulacion()
    print("="*40 + "\nSimulación finalizada. Revise 'errores_sistema.log' para detalles técnicos.")**
