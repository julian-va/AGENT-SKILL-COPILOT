# GitHub Copilot Instructions

## Objetivo

Proveer reglas claras, reproducibles y automatizables para el uso de GitHub Copilot en este repositorio: estilo de commits, idioma, previsualización, pruebas, seguridad y excepciones.

## Alcance

- Se aplica a todos los cambios propuestos por Copilot que afecten al código fuente, tests o infra del repositorio.
- Excepciones: documentación de producto en otros idiomas, archivos de configuración específicos (documentar cada excepción en la PR).

## Reglas generales

1. Commits
   - Nunca usar el usuario **"Copilot"** como autor de commits. El commit debe realizarse siempre con el usuario Git configurado por la persona responsable.
   - Convención de mensaje recomendada: `<type>(<scope>): <short summary>` (ej.: `feat(auth): add JWT refresh endpoint`).
   - Tipo comunes: `feat`, `fix`, `chore`, `docs`, `test`, `refactor`.

2. Idioma
   - **Código fuente y docstrings**: English.
   - **Mensajes de PR, previsualizaciones y la comunicación de Copilot hacia el mantenedor**: Español.
   - Documentación extensa puede incluir versiones en ambos idiomas si procede.

3. Comentarios en el código
   - No insertar comentarios explicativos que describan la lógica de forma redundante. Las explicaciones deben ir en la previsualización de la PR.
   - Docstrings públicos (APIs) pueden escribirse en English.

4. Previsualización (Preview) obligatoria
   - Antes de proponer cambios, Copilot debe generar una previsualización con esta plantilla mínima:
     - **Resumen:** breve descripción del cambio.
     - **Archivos modificados:** lista corta.
     - **Tests incluidos/afectados:** sí/no + resumen.
     - **Cómo validar localmente:** comandos (ej.: `npm test`, `pytest`).
     - **Riesgos conocidos / rollback:** breve.
     - **Checklist:** tests ✅, linter ✅, CI ✅.
   - Copilot no debe aplicar cambios no triviales sin aprobación humana.

5. Estructura de respuestas (obligatorio)
   - Cuando la respuesta incluya pasos, opciones o escenarios:
     - Estar **numerados**.
     - Ser **claros y concisos** (2–4 líneas por paso cuando sea posible).
     - Seguir un **orden lógico** y priorizar la acción recomendada.
   - Si hay alternativas, enumerarlas y marcar la opción recomendada con una breve justificación.
   - Ejemplo de formato:
     1. Paso 1: Breve acción y motivo (1–2 líneas).
     2. Paso 2: Acción siguiente y comando de verificación (1–2 líneas).
     3. Resultado esperado / nota de excepción (opcional).
   - Plantilla rápida:
     1. [Acción breve] — [Comando / comprobación]
     2. [Acción breve] — [Comando / comprobación]

6. Pull Requests y flujo
   - Copilot debe proponer cambios mediante PRs; no se permite push directo a ramas protegidas.
   - Checklist mínimo para PRs:
     1. Descripción clara y referencia a issue (si aplica).
     2. Previsualización incluida (ver plantilla).
     3. Tests añadidos/actualizados cuando corresponda.
     4. Linters/formatters aplicados.
     5. CI passing.
     6. Aprobación de al menos un mantenedor en cambios funcionales.

7. Tests, linters y hooks
   - Todo cambio funcional debe incluir tests automáticos.
   - Incluir comandos para ejecutar tests y linters localmente (ej.: `npm test`, `pytest`, `npm run lint`).
   - Usar pre-commit hooks (`pre-commit`, `husky`) para ejecutar checks básicos antes de commit.

8. Seguridad y dependencias
   - Prohibido incluir secretos o credenciales en el código. Usar herramientas de secret-scanning en CI.
   - Escaneo de vulnerabilidades a la hora de añadir dependencias (Dependabot/Snyk o similar).

9. Licencias y terceros
   - Verificar compatibilidad de licencias al copiar snippets de terceros y documentar la fuente en la PR.

10. Cuándo escalar a revisión humana
    - Cambios en infra, arquitectura, seguridad, performance o que afecten contratos públicos requieren aprobación explícita de un mantenedor.

11. Automatización de cumplimiento (CI)
    - CI debe validar al menos:
      - Que el autor del commit no sea "Copilot".
      - Que la PR incluya la previsualización obligatoria.
      - Que tests y linters pasen (cuando existan).
      - Que no se añadan comentarios explicativos en el código (donde sea aplicable por linters).

## Plantillas y ejemplos

### Ejemplo de mensaje de commit

```
feat(auth): add JWT refresh endpoint

- Add route POST /auth/refresh
- Add unit tests for refresh token logic
```

### Plantilla mínima para previsualización (Preview)

- Resumen:
- Archivos modificados:
- Tests incluidos:
- Cómo validar localmente:
  - `npm test` / `pytest`
- Riesgos y mitigación:
- Checklist:
  - [ ] Tests
  - [ ] Linter
  - [ ] CI passed

### Ejemplo de checklist de PR

- [ ] Descripción y contexto
- [ ] Previsualización incluida
- [ ] Tests añadidos/actualizados
- [ ] Linters/formatters ok
- [ ] CI verde
- [ ] Aprobación de mantenedor (si aplica)

---

Estas instrucciones deben aplicarse **en todas las interacciones y sugerencias de código** realizadas por GitHub Copilot en este repositorio.
