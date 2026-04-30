# Simulador de Destilación Binaria (Método McCabe-Thiele)

Herramienta interactiva para el cálculo del número de etapas teóricas en una columna de destilación fraccionada para mezclas binarias. 

## Descripción
Este simulador permite visualizar gráficamente el método de McCabe-Thiele. A partir de las composiciones de alimentación, destilado y fondos, así como la relación de reflujo y la condición térmica de la alimentación ($q$), el algoritmo traza las líneas de operación y la curva de equilibrio para determinar las etapas ideales requeridas.

## Fundamento Teórico
El cálculo de la curva de equilibrio asume una volatilidad relativa constante ($\alpha$), calculada mediante la siguiente relación empírica para el sistema binario:

$$ y = \frac{\alpha x}{1 + (\alpha - 1)x} $$

La intersección de la línea de operación de enriquecimiento (LOE) y la línea de operación de agotamiento (LOA) se determina mediante la línea $q$, cuya pendiente se define como:

$$ m_q = \frac{q}{q - 1} $$

## Tecnologías Utilizadas
* **Frontend:** HTML5, Tailwind CSS para el diseño de la interfaz.
* **Lógica y Gráficos:** JavaScript puro (Vanilla JS) y HTML5 Canvas API para el renderizado del diagrama de McCabe-Thiele.

## Uso
Para ejecutar el simulador, simplemente abre el archivo `index.html` en cualquier navegador web moderno, o visita la página desplegada a través de GitHub Pages.
