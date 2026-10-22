# Taller: Inferencia Bayesiana con Cobaya y BOBE

Este taller tiene como objetivo introducir el uso de **Cobaya** y **BOBE (Bayesian Optimization for Bayesian Evidence)** para realizar inferencia de parámetros cosmológicos y estimar la evidencia bayesiana.

Durante el taller utilizaremos el código cosmológico **CLASS**, a través de su interfaz de Python `classy`, para calcular predicciones teóricas.

Se compararán dos metodologías:

- **MCMC con Cobaya:** exploración de la distribución posterior mediante cadenas de Markov.
- **BOBE:** aproximación de la función de verosimilitud mediante procesos gaussianos y optimización bayesiana para estimar distribuciones posteriores y evidencias bayesianas.

Los ejercicios se ejecutarán mediante **Jupyter Notebook**.

---

## 1. Requisitos previos

Se recomienda utilizar:

- Conda o Miniconda.
- Git.
- Compilador de C/C++.


Sistemas operativos recomendados:

- macOS.
- Linux (Ubuntu).
- Windows mediante WSL2 (Ubuntu).

Para Windows se recomienda utilizar WSL2, ya que algunas dependencias científicas requieren herramientas de compilación compatibles con Linux.

---

## 2. Instalación del entorno de trabajo

Para facilitar la instalación y garantizar la compatibilidad de las dependencias, el taller incluye un archivo `environment.yml` que define el entorno Conda.

### 2.1. Clonar el repositorio del taller

```bash
git clone <URL_DEL_REPOSITORIO_DEL_TALLER>
cd bobe_workshop
```

### 2.2. Crear el entorno Conda

Desde el directorio que contiene `environment.yml`, ejecutar:

```bash
conda env create -f environment.yml
```

Este archivo configura el entorno `bobe_ws`, con Python 3.11 y las dependencias científicas principales y cobaya.

Activar el entorno:

```bash
conda activate bobe_ws
```

Verificar:

```bash
python --version
```
Verificamos la instalación de cobaya:

```bash
python -c "import cobaya; print('Cobaya instalado correctamente')"
```

También podemos comprobar que su comando principal está disponible:

```bash
cobaya-run --help
```
### 2.3. Instalar CLASS y classy

Con el entorno activado, instalar la interfaz de Python para CLASS:

```bash
python -m pip install classy
```

Verificar:

```bash
python -c "from classy import Class; print('CLASS/classy funcionando')"
```

### 2.4. Instalar BOBE

**BOBE (Bayesian Optimization for Bayesian Evidence)** es una herramienta para realizar inferencia bayesiana mediante procesos gaussianos y optimización bayesiana.

Existen dos métodos de instalación.

#### Opción 1: Instalación desde PyPI (recomendada)

Con el entorno `bobe_ws` activado, ejecutar:

```bash
python -m pip install BOBE
```

Para instalar BOBE junto con sus dependencias opcionales de cosmología:

```bash
python -m pip install "BOBE[cosmo]"
```

Esta última opción es recomendable para el taller, ya que trabajaremos con modelos cosmológicos utilizando Cobaya y CLASS.

#### Opción 2: Instalación desde el código fuente (desarrollo)

Si se desea utilizar la versión de desarrollo de BOBE o modificar su código fuente, clonar el repositorio oficial:

```bash
git clone https://github.com/Ameek94/BOBE.git
cd BOBE
```

Instalar en modo editable:

```bash
python -m pip install -e .
```

Para incluir las dependencias opcionales de cosmología:

```bash
python -m pip install -e ".[cosmo]"
```

El modo editable permite realizar modificaciones al código fuente sin tener que reinstalar el paquete después de cada cambio.



#### Verificar la instalación

Ejecutar:

```bash
python -c "from BOBE import BOBE; print('BOBE instalado correctamente')"
```

Si el comando termina sin errores, BOBE está disponible en el entorno.

Para comprobar la integración con las herramientas cosmológicas:

```bash
python -c "from classy import Class; from cobaya.model import get_model; from BOBE import BOBE; print('CLASS, Cobaya y BOBE disponibles')"
```

#### Regresar al directorio del taller

Si se utilizó la instalación desde el código fuente, regresar al directorio del taller antes de continuar con los ejercicios.

### 2.5. Configurar Jupyter

El entorno incluye JupyterLab e `ipykernel`.

Registrar el kernel:

```bash
python -m ipykernel install --user --name bobe_ws --display-name "Python (bobe_ws)"
```

Iniciar JupyterLab:

```bash
jupyter lab
```

Si se prefiere la interfaz clásica de Jupyter Notebook, instalarla mediante:

```bash
python -m pip install notebook
```

Y ejecutar:

```bash
jupyter notebook
```

Seleccionar el kernel **Python (bobe_ws)**.

---

## 3. Instalar  MPI (opcional)

Para ejecutar varias cadenas MCMC en paralelo, necesitamos una implementación de MPI y su interfaz para Python.

Una opción con Conda es:

```bash
conda install -c conda-forge openmpi mpi4py
```

Verificamos:

```bash
mpirun --version
```

La instalación de MPI es opcional para los ejercicios básicos, pero recomendable para comparar cadenas MCMC.

---

## 4. Instalar Jupyter Notebook

Instalamos Jupyter Notebook y el kernel de Python:

```bash
python -m pip install notebook ipykernel
```

Registramos nuestro entorno Conda:

```bash
python -m ipykernel install --user --name bobe_ws --display-name "Python (bobe_ws)"
```

Iniciamos Jupyter Notebook:

```bash
jupyter notebook
```

En Jupyter seleccionamos el kernel:

**Python (bobe_ws)**

Esto garantiza que los notebooks utilicen las bibliotecas instaladas en el entorno del taller.


---

## 5. Estructura del taller

La estructura sugerida del proyecto es:

```text
bobe_workshop/
│
├── input/
│   ├── lcdm_simple.yaml
│   ├── lcdm_full.yaml
│   └── results/
│
├── notebooks/
│   ├── TestExample.ipynb
│   ├── LCDM_simple.ipynb
│   └── LCDM_full.ipynb
│
├── results/
│       ├── lcdm_simple_partial4.1.txt
│       ├── lcdm_simple_partial4.2.txt
│       ├── lcdm_simple_partial4.3.txt
│       ├── lcdm_simple_partial4.4.txt
│       │
│       ├── lcdm_full.1.txt
│       ├── lcdm_full.2.txt
│       ├── lcdm_full.3.txt
│       ├── lcdm_full.4.txt
│       │
│       └── ...
├── results/
│   ├── H0_mock_BOBE/
│   ├── lcdm_simple_test_00/
│   └── lcdm_full_test_00/
│
└── README.md
```

---

## 6. Ejercicio 1: Verificar instalacción de Cobaya y BOBE
El primer ejercicio tiene como objetivo verificar que las herramientas principales del taller estén instaladas correctamente y familiarizarse con el funcionamiento básico de BOBE.

Utilizaremos el notebook:
```bash
notebooks/TestExample.ipynb
```
---

## 7. Ejercicio 2: Modelo simplificado de dos parámetros

El primer ejercicio utiliza un modelo cosmológico simplificado con dos parámetros libres:

$$
\theta = (H_0,\omega_b).
$$

Se consideran dos funciones de likelihood gaussianas:

$$
\ln \mathcal{L}_{H_0}
=
-\frac{1}{2}
\left(
\frac{H_0-\mu_{H_0}}{\sigma_{H_0}}
\right)^2,
$$

$$
\ln \mathcal{L}_{\omega_b}
=
-\frac{1}{2}
\left(
\frac{\omega_b-\mu_{\omega_b}}{\sigma_{\omega_b}}
\right)^2.
$$

La función de verosimilitud conjunta es:

$$
\ln \mathcal{L}(\theta)
=
\ln \mathcal{L}_{H_0}
+
\ln \mathcal{L}_{\omega_b}.
$$

Los parámetros restantes del modelo cosmológico se mantienen fijos.

### Probar Cobaya
Desde el directorio `input/`:

```bash
cd input
```

Ejecutamos:

```bash
cobaya-run lcdm_simple.yaml --test

### Ejecutar Cobaya

Desde el directorio `input/`:

```bash
cd input
```

Ejecutamos:

```bash
cobaya-run lcdm_simple.yaml
```

Para ejecutar cuatro cadenas en paralelo:

```bash
mpirun -np 4 cobaya-run lcdm_simple.yaml
```

Los resultados se guardarán en el directorio indicado por la opción `output` del archivo YAML.

---

## 8. Ejercicio 3: Modelo cosmológico ΛCDM

En el segundo ejercicio consideramos los seis parámetros cosmológicos habituales:

```math
\theta_{\Lambda \mathrm{CDM}}
=
\{
\log(10^{10}A_s),
n_s,
\tau_{\mathrm{reio}},
H_0,
\omega_b,
\omega_{\mathrm{cdm}}
\}.
```

Utilizamos datos cosmológicos de:

- Planck 2018 low-$\ell$ TT.
- Planck 2018 low-$\ell$ EE.
- Planck NPIPE CamSpec TTTEEE.
- Planck 2018 CMB lensing.
- DESY5 supernovas.
- DESI BAO DR2.

Los likelihoods de Planck también pueden introducir parámetros nuisance relacionados con calibración y foregrounds.

Por tanto, el espacio completo de parámetros puede tener una dimensionalidad mayor que seis.

### Importante: instalación de datos observacionales

Los likelihoods cosmológicos requieren archivos de datos adicionales que no se instalan necesariamente junto con Cobaya.

Antes de ejecutar este ejercicio, deben instalarse y configurarse los datos correspondientes a Planck, CamSpec, DESY5 y DESI DR2.

Las rutas y procedimientos de instalación dependen de las implementaciones concretas de los likelihoods utilizadas en el taller.

### Verificar la configuración

```bash
cobaya-run lcdm_full.yaml --test
```

Este comando permite comprobar que los componentes se inicializan correctamente.

### Ejecutar MCMC

```bash
cobaya-run lcdm_full.yaml
```

O mediante cuatro procesos MPI:

```bash
mpirun -np 4 cobaya-run lcdm_full.yaml
```

---


## Referencias

- **CLASS:** https://github.com/lesgourg/class_public
- **Cobaya:** https://cobaya.readthedocs.io/
- **BOBE:** https://github.com/BOBE-Sampler/BOBE/tree/main
