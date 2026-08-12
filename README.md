# Code Review AGLS

Revisión técnica cruzada del PFC de **BCEL** (SGA — Sistema de Gestión Académica Distribuido, Escuela Provincias Unidas), realizada por el equipo **AGLS**, según la guía FORM-TL-05 (principios SOLID, patrones de diseño, separación de capas, manejo de excepciones distribuidas y observabilidad).

## Equipo

**Revisores (AGLS):**
- Jhinson Stalyn Aucatoma Celorio — Bloques A, B y C
- Andy Paul Sánchez Pilaloa — Bloques D y E

**Repositorio revisado (BCEL):**
- `https://github.com/LEO23as/sga-sistema-distribuido`
- Commit auditado: `ff6f8b0` (completo: `ff6f8b0bf3f4200ae4deb9d6d7833e8bce7330f4`)
- Verificado: 11 de agosto de 2026, 08:30

## Contenido de este repositorio

| Archivo | Descripción |
|---|---|
| `informe.tex` | Informe completo en LaTeX: mapa de arquitectura, tabla consolidada de hallazgos, análisis por bloque (A–E), tabla de issues abiertos, preguntas de análisis obligatorias, conclusiones y declaración de uso de IA. |
| `referencias.bib` | Base bibliográfica (biblatex) citada en el informe. |
| `FORM_TL_06.pdf` | Documento de entrega/carátula para el LMS. |
| `issues_plan.md` | *(añadir)* Cuerpo completo de los 13 issues listos para copiar en GitHub, con el mapeo contra los números reales ya creados en el repo de BCEL. |

## Compilación del informe (`informe.tex`)

**Dependencias:** TeX Live 2021+ o MiKTeX, con **biber** (no BibTeX clásico). Paquetes: `babel` (spanish), `geometry`, `booktabs`, `tabularx`, `xltabular`, `tcolorbox`, `listings`, `xcolor`, `xurl`, `enumitem`, `tikz`, `fancyhdr`, `titlesec`, `longtable`, `biblatex` (backend biber, estilo ieee), `hyperref`, `microtype`.

En TeX Live sobre Debian/Ubuntu, si falta algo: `sudo apt install texlive-bibtex-extra biber texlive-lang-spanish`. En MiKTeX se instala automáticamente al compilar.

```bash
pdflatex -interaction=nonstopmode informe.tex
biber informe
pdflatex -interaction=nonstopmode informe.tex
pdflatex -interaction=nonstopmode informe.tex
```

Cuatro pasos en ese orden: el primer `pdflatex` genera el `.bcf` que necesita `biber` para resolver las citas contra `referencias.bib`; los dos `pdflatex` siguientes insertan la bibliografía y estabilizan referencias cruzadas (índice, `\ref` a secciones).

## Issues abiertos en el repositorio de BCEL

12 de 13 issues ya están creados en `https://github.com/LEO23as/sga-sistema-distribuido/issues` (#2–#9 y #11–#14), con responsable propuesto por bloque temático:

| Responsable | Issues |
|---|---|
| Keyla Bedón Viteri | #2, #3, #4, #5 (Bloque A — SOLID) |
| Pedro Castro López | #6, #7, #8 (BD compartida, secretos, resiliencia gRPC) |
| Juliana Emanuel Pino | #9, #11, #12 (manejo de errores y trazabilidad) |
| Ernesto Luna Mora | #13, #14, y (pendiente) el issue de limpieza de repositorio |

## Estado de la entrega

- [x] Datos de los dos revisores de AGLS completos en la portada del informe.
- [x] SHA y fecha del commit auditado verificados.
- [x] 12 de 13 issues creados en el repositorio de BCEL, con enlaces reales en el informe.
- [x] Responsable propuesto (BCEL) asignado a cada issue en el informe.
- [ ] El issue **#10** de GitHub quedó duplicado del **#9**; falta editarlo con el contenido del issue de higiene de repositorio (hallazgo H14) en vez de dejarlo repetido.
- [ ] Crear las etiquetas (`solid`, `patrones`, `capas`, `excepciones`, `logging`, `seguridad`, `severidad:critica`, `severidad:mayor`, `severidad:menor`, `sugerencia`) y aplicarlas a cada issue.
- [ ] Que cada integrante de BCEL se autoasigne (*assignee* de GitHub) el issue que le corresponde.
- [ ] Compilar el PDF final y confirmar que no queden advertencias `Citation ... undefined` ni `Empty bibliography` en el log.
- [ ] Verificar que el conteo de páginas de contenido (sin portada ni bibliografía) esté entre 6 y 12, como exige FORM-TL-05.
- [ ] Subir el PDF compilado al LMS junto con `FORM_TL_06.pdf`.
