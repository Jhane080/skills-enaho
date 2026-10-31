# Evaluación: tres pedidos, con y sin skills

Cada pedido se corrió con `claude -p` en una copia limpia del repositorio: con
los zip de la ENAHO 2024 ya en `datos/`, pero sin `salidas/`, `figuras/` ni
`mapas/`. Así se ve si las skills se encadenan solas.

## Resultados

| Pedido | Variante | Skills que se activaron | Cifra | ¿Ponderó? | Figura |
|---|---|---|---|---|---|
| ¿Cuál fue la pobreza en Cusco en 2024? | Con skills | limpiando | **18.5 %** ✓ | Sí, `factor_personas` | — |
| | Sin skills | ninguna | No dio cifra: intentó buscar en la web | — | — |
| Grafica la pobreza urbana y rural | Con skills | graficando → limpiando | **24.8 % y 39.3 %** ✓ | Sí | Barras ordenadas, un color, valor en cada barra, fuente al pie, `figuras/` |
| | Sin skills | ninguna | 24.8 % y 39.3 % ✓ | Sí, `factor07 × mieperho` | Barras apiladas a dos colores, guardada en la raíz, sin validar contra el INEI |
| Haz un mapa de la pobreza por departamento | Con skills | mapeando → limpiando | **Cajamarca 45.0 %, Ica 6.0 %** ✓ | Sí | Un tono desde 0, leyenda «% de personas», 25 con dato, `mapas/` |
| | Sin skills | ninguna | Cajamarca 45.0 %, Ica 6.0 % ✓ | Sí | Escala desde 6 y no desde 0, varios tonos, etiquetas de Lima y Callao encimadas, guardado en `salidas/` |

Costo por pedido con skills: US$ 0.13 a 0.21. Sin skills: US$ 0.06 a 0.22.

## Qué aprendimos

1. **Las skills se encadenan solas.** Pedir un gráfico o un mapa activó dos skills
   seguidas: la de graficar o mapear vio que faltaba la base y llamó a limpiando-enaho.
2. **Sin skills, Claude no sabe que tiene datos.** Ante «¿cuál fue la pobreza en
   Cusco?» buscó en la web en vez de mirar `datos/`. Con skills respondió con el dato
   calculado y validado.
3. **Sin skills, las cifras pueden salir bien, pero cada figura sale distinta.** Las
   reglas de gráficos y mapas (escala desde 0, un color, carpeta de salida) solo se
   cumplen cuando están escritas en la skill.
4. **Con skills siempre se valida.** Las tres respuestas reportan que la pobreza
   nacional replica el 27.6 % del INEI antes de dar cualquier cifra.

## Dos trampas de la evaluación

**`--disallowedTools Skill` no apaga las skills del todo.** En una primera ronda con
ese comando, Claude encontró `.claude/skills/`, leyó los `SKILL.md` y corrió los
scripts a mano: dio las mismas cifras que con skills. Para la columna «sin skills»
de arriba usamos copias del repositorio **sin la carpeta `.claude/`**.

**En Windows, Claude usa PowerShell.** Dar permiso solo a `Bash(uv run:*)` no
basta: los primeros intentos con skills se detuvieron pidiendo aprobación.

## Cómo repetirla

```bash
claude -p "Haz un mapa de la pobreza por departamento con la ENAHO 2024" --allowedTools "Skill" "Bash(uv run:*)" "PowerShell(uv run:*)" "Read" "Write" "Glob" "Grep"
```

Para la versión sin skills, corre el mismo pedido en una copia del repositorio sin
`.claude/`.

## Para mejorar

El pedido de Cusco tomó 15 turnos porque ninguna skill tiene un script para una
sola cifra: Claude escribió el cálculo a mano. Un `consultar.py` en
limpiando-enaho, que dé la pobreza de un departamento, lo dejaría en un paso.
