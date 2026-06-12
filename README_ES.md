# Planificador de Procesos en XV6

📄 **English version:** [README.md](README.md)

## Descripción General

Este proyecto explora los mecanismos de planificación de procesos del sistema operativo **XV6 RISC-V** mediante el análisis de su planificador, modificaciones en el kernel y la evaluación de rendimiento.

El trabajo incluye la implementación de una **política de planificación basada en prioridades**, la ejecución de pruebas sobre procesos **CPU-bound** e **I/O-bound**, y el análisis de aspectos como el **cambio de contexto**, el **tamaño del quantum** y posibles escenarios de **starvation**.

Desarrollado como parte de la materia **Sistemas Operativos** en la **Facultad de Matemática, Astronomía, Física y Computación (FaMAF)** de la **Universidad Nacional de Córdoba (UNC)**.

---

## Características

* Análisis del planificador **Round Robin** utilizado por defecto en XV6.
* Implementación de un planificador personalizado basado en prioridades.
* Estudio de los estados de los procesos y los mecanismos de cambio de contexto.
* Benchmarking de cargas de trabajo CPU-bound e I/O-bound.
* Evaluación del impacto de distintos tamaños de quantum.
* Análisis de starvation en esquemas de planificación por prioridades.
* Comparación de rendimiento entre distintas políticas de planificación.

---

## Temas Abordados

* Sistemas Operativos
* Planificación de Procesos
* Algoritmos de Planificación de CPU
* Desarrollo de Kernel
* Cambio de Contexto
* Prioridades de Procesos
* Starvation
* Análisis de Rendimiento
* XV6 RISC-V

---

## Tecnologías Utilizadas

* **C**
* **XV6 RISC-V**
* **QEMU**
* **Git**
* Herramientas de Benchmarking

---

## Informe Académico

El informe completo del laboratorio puede consultarse en:

* [INFORME.md](./INFORME.md)

---

## Institución

**Facultad de Matemática, Astronomía, Física y Computación (FaMAF)**
**Universidad Nacional de Córdoba (UNC)**
**Materia Sistemas Operativos – 2024**
