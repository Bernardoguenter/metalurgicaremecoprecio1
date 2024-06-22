import pandas as pd
import ipywidgets as widgets
from IPython.display import display
from scipy.interpolate import interp1d

# Datos de la lista de precios para Galpón
data_galpon = {
    "Área (m²)": [50, 100, 150, 200, 250, 300, 350, 400, 500, 600, 700, 800, 900, 1000],
    "Precio (USD)": [176, 160, 155, 152, 150, 148, 147, 146, 144, 142, 140, 138, 137, 136]
}
price_list_galpon_df = pd.DataFrame(data_galpon)

# Datos de la lista de precios para Tinglado
data_tinglado = {
    "Área (m²)": [50, 75, 100, 150, 200, 250, 300, 400, 500, 600, 800, 1000],
    "Precio (USD)": [110, 105, 96, 92, 88, 86, 84, 82, 80, 78, 76, 74]
}
price_list_tinglado_df = pd.DataFrame(data_tinglado)

# Función para calcular el precio usando interpolación lineal con límites adicionales
def calcular_precio_interpolado(area, material, estructura):
    if estructura == "Galpón":
        interpolador = interp1d(price_list_galpon_df["Área (m²)"], price_list_galpon_df["Precio (USD)"], fill_value="extrapolate")
    else:
        interpolador = interp1d(price_list_tinglado_df["Área (m²)"], price_list_tinglado_df["Precio (USD)"], fill_value="extrapolate")
    
    precio_por_metro = interpolador(area)
    
    # Limitar el precio mínimo para áreas superiores a 1000 m²
    if estructura == "Tinglado" and area >= 1000:
        precio_por_metro = max(precio_por_metro, 74)
    
    # Ajustar el precio según el tipo de material
    if material == "Perfil U Ángulo":
        precio_por_metro += 24
    elif material == "Alma Llena":
        precio_por_metro += 48
    
    return precio_por_metro

# Crear un cuadro de selección para el tipo de estructura
estructura_selector = widgets.Dropdown(
    options=["Galpón", "Tinglado"],
    value="Galpón",
    description='Estructura:'
)

# Crear un cuadro de selección para el tipo de material
material_selector = widgets.Dropdown(
    options=["Hierro Torsionado", "Perfil U Ángulo", "Alma Llena"],
    value="Hierro Torsionado",
    description='Material:'
)

# Crear campos de entrada de texto para seleccionar ancho, largo y alto
ancho_input = widgets.IntText(
    value=15,
    description='Ancho (m):',
)

largo_input = widgets.IntText(
    value=10,
    description='Largo (m):',
)

alto_input = widgets.IntText(
    value=5,
    description='Alto (m):',
)

# Crear campo de entrada de texto para metros de cerramiento
cerramiento_input = widgets.FloatText(
    value=4.5,
    description='Cerramiento (m):',
)

# Crear campo de entrada de texto para el tipo de cambio
tipo_cambio_input = widgets.FloatText(
    value=1.0,
    description='Tipo de Cambio:',
)

# Crear campo de entrada de texto para el porcentaje adicional
porcentaje_adicional_input = widgets.FloatText(
    value=0.0,
    description='Porcentaje (+%):',
)

# Crear una salida para mostrar el resultado
output = widgets.Output()

# Función para actualizar el estado del campo de cerramiento
def actualizar_cerramiento(change):
    if estructura_selector.value == "Tinglado":
        cerramiento_input.disabled = True
        cerramiento_input.value = 0  # Restablecer el valor a 0 cuando está deshabilitado
    else:
        cerramiento_input.disabled = False

# Función para actualizar el resultado basado en los valores de las entradas de texto y el selector
def update_result(change):
    with output:
        output.clear_output()
        ancho = ancho_input.value
        largo = largo_input.value
        alto = alto_input.value
        cerramiento = cerramiento_input.value
        tipo_cambio = tipo_cambio_input.value
        porcentaje_adicional = porcentaje_adicional_input.value
        
        # Área del piso
        area_piso = ancho * largo
        
        # Perímetro del galpón
        perimetro = 2 * (ancho + largo)
        
        # Área de las paredes
        area_paredes = perimetro * alto
        
        # Área total
        area_total = area_piso + area_paredes
        
        material = material_selector.value
        estructura = estructura_selector.value
        precio_por_metro = calcular_precio_interpolado(area_piso, material, estructura)
        
        # Calcular el número de columnas del lado del largo
        num_columnas_largo = (largo // 5) + 1
        total_columnas = num_columnas_largo * 2  # Dos lados del galpón
        
        # Precio por metro de columna según el material
        if material == "Hierro Torsionado":
            precio_columna = 38
        elif material == "Perfil U Ángulo":
            precio_columna = 76
        else:  # Alma Llena
            precio_columna = 160
        
        # Calcular el costo de las columnas según la altura
        if alto > 5:
            costo_columnas = total_columnas * (alto - 5) * precio_columna  # Añadir costo si la altura es superior a 5m
        elif alto < 5:
            costo_columnas = total_columnas * (5 - alto) * (-precio_columna)  # Restar costo si la altura es inferior a 5m
        else:
            costo_columnas = 0  # Costo es 0 si la altura es exactamente 5m
        
        # Calcular el costo del cerramiento si no está deshabilitado
        if not cerramiento_input.disabled:
            if cerramiento > 4.5:
                costo_cerramiento = (cerramiento - 4.5) * perimetro * 27
            elif cerramiento < 4.5:
                costo_cerramiento = (4.5 - cerramiento) * perimetro * (-27)
            else:
                costo_cerramiento = 0  # Costo es 0 si la medida es exactamente 4.5m
        else:
            costo_cerramiento = 0  # No se descuenta nada del precio si está deshabilitado
        
        costo_piso = area_piso * precio_por_metro
        precio_total_usd = costo_piso + costo_columnas + costo_cerramiento
        
        # Convertir a pesos argentinos y aplicar porcentaje adicional
        precio_total_ars = precio_total_usd * tipo_cambio
        precio_total_final = precio_total_ars * (1 + porcentaje_adicional / 100)
        
        print(f"Área del piso: {area_piso} m²")
        print(f"Perímetro del galpón: {perimetro} m")
        print(f"Número total de columnas: {total_columnas}")
        print(f"Área de las paredes: {area_paredes} m²")
        print(f"Área total: {area_total} m²")
        print(f"Estructura seleccionada: {estructura}")
        print(f"Material seleccionado: {material}")
        print(f"Precio por metro cuadrado de piso: {precio_por_metro:.2f} USD/m²")
        print(f"Costo adicional/reducción por columnas: {costo_columnas:.2f} USD")
        print(f"Costo del cerramiento: {costo_cerramiento:.2f} USD")
        print(f"Precio total en USD: {precio_total_usd:.2f} USD")
        print(f"Precio total en ARS: {precio_total_ars:.2f} ARS")
        print(f"Precio final con porcentaje adicional: {precio_total_final:.2f} ARS")

# Configurar los eventos de cambio de valor de las entradas de texto y el selector
ancho_input.observe(update_result, names='value')
largo_input.observe(update_result, names='value')
alto_input.observe(update_result, names='value')
cerramiento_input.observe(update_result, names='value')
tipo_cambio_input.observe(update_result, names='value')
porcentaje_adicional_input.observe(update_result, names='value')
material_selector.observe(update_result, names='value')
estructura_selector.observe(update_result, names='value')

# Configurar el evento de cambio de valor del selector de estructura para actualizar el estado del campo de cerramiento
estructura_selector.observe(actualizar_cerramiento, names='value')

# Mostrar el selector, las entradas de texto y la salida
display(estructura_selector, material_selector, ancho_input, largo_input, alto_input, cerramiento_input, tipo_cambio_input, porcentaje_adicional_input, output)

# Mostrar el precio inicial y actualizar el estado del campo de cerramiento
actualizar_cerramiento
