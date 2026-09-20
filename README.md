# Sintetización Monte Carlo de la silueta de un agujero negro con Ray Tracing (Aproximación Eikonal)

**Autor:** David Alejandro Pérez Múnera

## Descripción General

Este repositorio contiene una simulación numérica que sintetiza las imágenes que observaría un observador lejano de un agujero negro rodeado por un disco de acreción. La simulación se basa en *ray tracing inverso*: se lanzan rayos desde la pantalla del observador (plano $z-y$, en posición $x=-100$) hacia el agujero negro y, para cada uno, se calcula su trayectoria completa.

Para cada rayo se determina:
- Si su trayectoria **toca el disco de acreción** (el píxel se ilumina)
- Si **cae al horizonte de eventos** (absorbido por el agujero negro)
- Si **escapa al infinito** (el píxel queda oscuro)

## Aproximación Física

La simulación no resuelve las ecuaciones completas de la geometría diferencial, sino que utiliza la **aproximación Eikonal**. En esta aproximación, los rayos de luz se tratan como si se propagaran a través de un medio en el que la velocidad de la luz es inversamente proporcional a un índice de refracción $n(\vec{r})=n(x,y)$.

### Índice de Refracción

El espacio-tiempo alrededor de un agujero negro se modela mediante un índice de refracción radial:

$$n(r)= \frac{1}{1-\frac{A}{r}}, \quad r=\sqrt{x^2 + y^2}$$

donde $A$ es una constante correspondiente al radio gravitacional del agujero negro.

A medida que $r \to A$, los rayos de luz se curvan fuertemente hacia el centro. Para mejorar la aproximación a lo predicho por la relatividad general, se introduce un factor de corrección:

$$\text{corr} = \left|1 - \frac{9}{8} \left(\frac{A}{r}\right)^2\right|$$

### Ecuaciones de Movimiento

Las ecuaciones que describen la trayectoria $(x, y)$ y la velocidad $(v_x, v_y)$ de un rayo son:

$$\frac{dx}{dt} = \frac{v_x}{n(x, y)} \quad \wedge \quad \frac{dy}{dt} = \frac{v_y}{n(x,y)}$$

$$\frac{d v_x}{dt} = \frac{\partial n}{\partial x}(x, y) \quad \wedge \quad \frac{d v_y}{dt} = \frac{\partial n}{\partial y}(x, y)$$

Estas ecuaciones se integran numéricamente usando `scipy.integrate.solve_ivp` con un criterio de detención para el horizonte de eventos.

## Estructura del Repositorio

```
proyecto/
├── Proyecto_1.ipynb              # Notebook principal con la simulación completa
├── simulacion_nolte.py           # Referencia: implementación alternativa (Nolte, 2019)
├── imagenes_bacanas/             # Carpeta de imágenes generadas
│   └── BH_images/                # Imágenes del agujero negro con diferentes inclinaciones
└── README.md                      # Este archivo
```

## Archivo Principal: `Proyecto_1.ipynb`

El notebook está estructurado en las siguientes secciones:

### 1. **Generación de Posiciones Iniciales**
Se generan $N$ puntos uniformemente distribuidos dentro de un círculo de radio $R$ en el plano $z-y$. Estos representan píxeles en la pantalla del observador.

Parámetros:
- `N = 17000`: número de rayos a simular
- `R = 70`: radio de la pantalla del observador

### 2. **Integración de Trayectorias de Rayos**
Se resuelven las ecuaciones diferenciales para todos los rayos usando integración numérica. La simulación:
- Comienza en $x = -100$ (posición del observador)
- Integra en el intervalo de tiempo $t \in [0, 400]$
- Detiene la integración cuando un rayo alcanza el horizonte de eventos ($r = A$)

Parámetros:
- `A = 10`: radio del agujero negro (horizonte de eventos)
- `tspan = [0, 400]`: intervalo de integración

### 3. **Construcción del Disco de Acreción**
Se genera un disco de acreción muestreando $N_d$ puntos uniformemente distribuidos dentro de un anillo de radios interno $R_{in}$ y externo $R_{ext}$.

Parámetros:
- `Nd = 11000`: número de partículas en el disco
- `Rin = 15`: radio interno del disco
- `Rext = 50`: radio externo del disco
- `i_rot = 20`: ángulo de inclinación del disco respecto a la línea de visión (en grados)

El disco se rota alrededor del eje $x$ (línea de visión del observador) para simular diferentes ángulos de inclinación.

### 4. **Intersecciones Rayo-Disco**
Para cada rayo, se busca el primer punto de intersección con el plano del disco. La función `intersecta_disco()` calcula:
- La normal del plano del disco usando descomposición en valores singulares (SVD)
- Los puntos donde cada segmento de rayo cruza el plano
- El radio del punto de cruce (medido desde el agujero negro)
- Verifica que el cruce esté dentro del anillo $[R_{in}, R_{ext}]$

### 5. **Síntesis de Imágenes**
Se generan imágenes coloreadas donde:
- Los píxeles **blancos** representan impactos cercanos al agujero negro
- Los píxeles **rojos** representan impactos más alejados (cerca del borde exterior del disco)
- Los píxeles **negros** representan rayos que no golpean el disco (absorbidos o escapan)

Se generan imágenes para tres ángulos de inclinación: 20°, 45° y 70°.

## Cómo Usar

### Requisitos
- Python 3.7+
- NumPy
- SciPy
- Matplotlib
- Jupyter Notebook

### Instalación

```bash
pip install numpy scipy matplotlib jupyter
```

### Ejecución

1. Abre Jupyter Notebook:
```bash
jupyter notebook
```

2. Navega a `Proyecto_1.ipynb` y abre el archivo.

3. Ejecuta todas las celdas en orden. El notebook está diseñado para ejecutarse de forma secuencial.

4. Las imágenes generadas se guardarán en la carpeta `imagenes_bacanas/BH_images/`.

## Parámetros Configurables

Puedes modificar los parámetros en la celda de configuración para cambiar el comportamiento de la simulación:

```python
# Para los rayos
N = 17000          # número de posiciones iniciales
R = 70              # radio de la pantalla del observador
A = 10             # radio del agujero negro

# Para el disco de acreción
Nd = 11000         # número de partículas en el disco
Rin = 15            # radio interno del disco
Rext = 50          # radio externo del disco
i_rot = 20         # ángulo de inclinación (en grados)
```

**Nota:** Aumentar `N` y `Nd` proporciona mejores imágenes pero requiere más tiempo de cálculo.

## Archivos Generados

Las imágenes se guardan en formato PNG en `imagenes_bacanas/BH_images/`:
- `agujero_negro_20.png` – Disco inclinado a 20°
- `agujero_negro_45.png` – Disco inclinado a 45°
- `agujero_negro_70.png` – Disco inclinado a 70°

## Referencias

- **Nolte, D. D.** (2019). *Introduction to Modern Dynamics: Chaos, Networks, Space and Time* (2nd ed.). Oxford University Press.
  - El archivo `simulacion_nolte.py` contiene una implementación referencial basada en este trabajo.

## Correcciones y Mejoras Recientes

- **Función `intersecta_disco()`**: Se corrigió para usar intersecciones exactas del plano en lugar de proximidad a partículas aleatorias. Esto reduce falsos positivos del ~15% al <1% en la detección de impactos.
- **Convención de ejes**: Se ajustó la rotación del disco para usar la convención correcta ($R_z(i_{rot})$) donde el ángulo 0° significa disco de frente y 90° significa disco de canto.
- **Geometría**: El plano del disco ahora se define correctamente centrado en el agujero negro.

## Notas

- La simulación es computacionalmente intensiva. Con `N = 17000` y `Nd = 11000`, el tiempo total es de aproximadamente 2-5 minutos.
- Los resultados son imágenes 2D que proyectan la estructura 3D del disco sobre la pantalla del observador.
- La punteadura visible en las imágenes es inherente al muestreo Monte Carlo; se puede reducir aumentando `N`.

## Autor

**David Alejandro Pérez Múnera**  
Estudiante de Astronomía Moderna  
Universidad [Institution Name]

---

Para más detalles sobre la implementación, consulte el notebook `Proyecto_1.ipynb`.
