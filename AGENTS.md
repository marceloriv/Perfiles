# AGENTS.md

**Para el agente que trabaje en este repo:**
- Sin emojis en commits fuera del esquema Gitmoji ya en uso (sección 11), nunca como decoración libre.
- Este repo no tiene código de aplicación. No copiar boilerplate de proyecto de software (tests, CI, deploy) que no aplica acá — ver sección 0.

## 0. Jerarquía de reglas
1. Seguridad y corrección (no corromper JSON, no filtrar datos personales) — nunca se sacrifica.
2. Convenciones observadas en el repo (estructura de archivos, formato de commit real) — se siguen salvo instrucción explícita en contrario.
3. Minimalismo (sección 6) — se aplica solo después de 1 y 2.

## 1. Resumen del proyecto
Backup/versionado de **VS Code Profiles** (`.code-profile`) del usuario, uno por stack de trabajo (Java, Python, Web, Markdown, DevOps) más uno base (`predeterminado`). Permite restaurar tema, settings, keybindings y lista de extensiones en cualquier máquina vía `Profiles: Import Profile`. Uso individual, no colaborativo.

## 2. Formato técnico
- No hay lenguaje de aplicación ni framework — son exports JSON de VS Code.
- Edición de settings/extensions vía Node.js inline (`node -e "..."`) cuando el cambio es puntual y programático; edición manual solo para cambios triviales de una clave.
- Sin package manager, sin dependencies propias.

## 3. Estructura del proyecto
```
Java.code-profile          # perfil Spring Boot/Maven/Gradle
Python.code-profile        # perfil Jupyter/Ruff/Pylance
Web.code-profile           # perfil React/Angular/TypeScript, Tailwind/Bootstrap
Markdown.code-profile      # perfil docs/mermaid/markdownlint
DevOps.code-profile        # perfil Terraform/AWS, Docker Compose, K8s, MongoDB
predeterminado.code-profile # perfil base, mínimo de extensiones
```
Todos comparten tema `Monokai Pro (Filter Spectrum)`, `material-icon-theme`, fuente `Hack Nerd Font`, lang pack ES.

## 4. Comandos
```bash
# validar que un perfil sigue siendo JSON válido tras editarlo
node -e "JSON.parse(require('fs').readFileSync('Markdown.code-profile','utf8')); console.log('ok')"

# leer settings reales (doble-encoded, ver sección 5)
node -e "
const d=JSON.parse(require('fs').readFileSync('Markdown.code-profile','utf8'));
const s=JSON.parse(JSON.parse(d.settings).settings);
console.log(JSON.stringify(s,null,1));
"

# listar extensiones de un perfil
node -e "
const d=JSON.parse(require('fs').readFileSync('Markdown.code-profile','utf8'));
console.log(JSON.parse(d.extensions).map(e=>e.identifier.id).join('\n'));
"
```
No hay build, test suite ni entorno local que levantar — son archivos de configuración estáticos.

## 5. Estilo / formato de datos
**Gotcha del formato:** cada `.code-profile` es JSON con doble-encoding. `settings` es un string que contiene JSON, y ese JSON tiene a su vez una clave `settings` con otro string JSON adentro:

```js
const outer = JSON.parse(d.settings);        // { settings: "..." }
const real  = JSON.parse(outer.settings);    // objeto real de VS Code settings.json
```

`extensions` es un array JSON-string normal: `JSON.parse(d.extensions)` → lista de `{identifier:{id,...}, displayName, disabled?, applicationScoped}`.

**Al editar:** re-serializar con `JSON.stringify` en el mismo orden de encoding (real → outer.settings → d.settings), nunca escribir el objeto real directamente en `d.settings`. Ver bug real ya encontrado: claves de `editor.unicodeHighlight.allowedCharacters` pueden corromperse a mojibake (`├®` en vez de `é`) si se edita con encoding incorrecto — siempre verificar que tildes/ñ sobrevivan el roundtrip.

Preservar EOL `\r\n` si el archivo ya los usa (VS Code en Windows los genera así); no forzar `\n`.

## 6. Disciplina anti-sobreingeniería (Ponytail)
Escalera antes de tocar un perfil:
1. ¿Hace falta este cambio? (¿no alcanza con editarlo a mano desde VS Code UI?)
2. ¿Ya existe un setting/extensión equivalente en otro perfil de este repo? Copiar el patrón, no reinventar.
3. ¿Se resuelve con un solo `node -e` de la sección 4? Hazlo así, sin script nuevo en el repo.

No crear scripts, `package.json`, linters ni tooling nuevo para este repo — es un repositorio de datos de configuración, no de código.

## 7. Verificación mínima (reemplaza "Pruebas")
No hay test suite. Antes de dar por cerrado un cambio a un `.code-profile`:
1. `node -e "JSON.parse(...)"` parsea sin error (comando de sección 4).
2. Diff (`git diff`) toca solo las claves que se querían cambiar — nada de reformateo masivo del archivo por un save completo de VS Code.
3. Si se tocó texto con acentos/ñ, confirmar que no se corrompió el encoding (grep visual del bloque afectado).

## 8. Seguridad
Nunca commitear tokens, API keys o credenciales dentro de un `.code-profile` (revisar `globalState` y `settings` antes de commitear — ahí a veces quedan URLs internas, paths con nombre de usuario real, o tokens de extensiones como GitHub/MongoDB pegados por accidente al exportar).

Repo es público-potencial: si algún perfil tiene datos que identifiquen infraestructura interna (IPs, hostnames de empresa) fuera de settings genéricos de editor, señalarlo antes de commitear.

## 9. Commits y PR
Formato observado en el historial real de este repo: `tipo: :emoji: sujeto` (Conventional Commits + Gitmoji, con el emoji **después** del tipo — orden invertido respecto al Gitmoji estándar `:emoji: tipo:`). Mantener este orden, no "corregirlo" al estándar sin que el usuario lo pida.

Ejemplos reales: `docs: :sparkles: actualizacion del perfil de Markdown`, `feat: :sparkles: Actualizar los perfiles`.

Tipos vistos: `feat`, `docs`. Sujeto en español, minúscula tras el emoji, sin punto final.

Nunca agregar trailers de autoría de agente/IA (`Co-Authored-By`, "Generated with…", session links) salvo pedido explícito puntual.

## 10. Límites del agente
**Siempre** (sin pedir permiso): editar claves de un `.code-profile` cuando el pedido es explícito y acotado; validar JSON tras editar; crear commits locales.

**Preguntar primero**: cambiar tema/tipografía/keybindings de forma masiva (son preferencias personales, no bugs — no "mejorar" sin que se pida), agregar/quitar extensiones por iniciativa propia, `git push`.

**Nunca sin aprobación explícita**: reescribir un perfil completo desde cero, borrar un `.code-profile`, fusionar dos perfiles en uno.

## 11. Mantenimiento
Repo chico y de bajo cambio — no necesita revisión periódica de este archivo. Actualizar solo si cambia la convención de commits, se agrega un perfil nuevo, o el formato de export de VS Code cambia de estructura.
