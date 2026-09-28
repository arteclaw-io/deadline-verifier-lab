---
name: verificar-cita
description: Verifica si una cita textual que Claude dice haber extraído de un documento existe de verdad en el texto fuente (grounding), y si una fecha resultante es futura respecto a una fecha de referencia dada. Usar cuando alguien pide chequear si una respuesta de Claude sobre un plazo/cita legal es confiable antes de mostrarla como tal.
---

# Verificar cita y fecha límite

Esta skill envuelve la lógica de `src/04-verify.js` de este repo
(`verificarGrounding` y `verificarFecha`) para que se pueda invocar como
chequeo puntual dentro de una sesión de Claude Code, sin tener que correr
`index.js` completo.

## Cuándo usar esta skill

- Alguien pegó un texto fuente + una cita que un LLM dice haber sacado de
  ahí, y quiere saber si esa cita es real (grounding) antes de confiar en
  cualquier conclusión basada en ella.
- Alguien tiene una fecha límite extraída y quiere confirmar que sea
  posterior a una fecha de referencia dada (no un plazo ya vencido).

## Cómo verificar

**Preferí SIEMPRE ejecutar el código real en vez de comparar los textos
vos mismo.** Si estás en una sesión de Claude Code con este repo
disponible, corré algo como:

```bash
node -e '
import("./src/04-verify.js").then(({ verificarGrounding, verificarFecha }) => {
  const g = verificarGrounding("TEXTO FUENTE COMPLETO ACÁ", "FRAGMENTO CITADO ACÁ");
  const f = verificarFecha("YYYY-MM-DD o null", new Date("FECHA DE REFERENCIA YYYY-MM-DD"));
  console.log(JSON.stringify({ g, f }, null, 2));
});
'
```

Esto usa la comparación de substring exacto real (`verificarGrounding`) y
la validación de fecha real (`verificarFecha`) del código del lab — nada
de "me parece que sí coincide". Solo si no podés ejecutar código, aplicá
el criterio de reserva de abajo y aclará que es una aproximación:

*Criterio de reserva (sin ejecutar código):*
1. **Grounding** — el fragmento citado tiene que existir TEXTUALMENTE
   (no parafraseado) dentro del texto fuente completo. Si no aparece tal
   cual, la cita es falsa aunque "suene" coherente con el resto.
2. **Fecha** — tiene que ser una fecha real (parseable) y posterior o
   igual a la fecha de referencia dada (no la fecha real del sistema,
   salvo que se indique lo contrario explícitamente — ver el bug real
   documentado en el historial de commits de este repo sobre por qué la
   fecha de referencia no puede ser `new Date()` a ciegas).

Con cualquiera de las dos vías: si CUALQUIERA de los dos chequeos falla,
la conclusión completa es "NO CONFIABLE" — no hay términos medios ni
"confiable con reservas".

## Output esperado

Reportar en este formato:

```
Grounding: [OK / FALLÓ] — <motivo si falló>
Fecha: [OK / FALLÓ] — <motivo si falló>
Veredicto: [CONFIABLE / NO CONFIABLE]
```

## Por qué existe esta skill (nota para la clase)

Esto es exactamente el punto pedagógico del lab convertido en una Skill
reutilizable: separar "está bien formado" (evaluarFormato, Associate) de
"es verdad" (esta skill, Architect Professional). Una Skill de Claude Code
es, en esencia, instrucciones reutilizables versionadas junto al código —
el mismo criterio de "no confiar a ciegas en el output" se puede aplicar
tanto corriendo `node index.js` como pidiéndole a Claude Code, en una
sesión interactiva, que use esta skill sobre un texto que le pegues en el
chat.
