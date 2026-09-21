# El Rosco · Palabros

> **Si ya tenías la app publicada:** esta versión agrega niveles de dificultad
> (ver más abajo) y cambia `words.json` e `index.html`. `firestore.rules` no
> cambió esta vez (la regla existente ya cubre el nuevo campo `progress`).
> Solo debes volver a subir `index.html` y `words.json` al mismo lugar donde
> los tenías (GitHub Pages o Firebase Hosting), reemplazando los anteriores.

App web de un solo jugador que simula "El Rosco" de Pasapalabra, usando tu propio banco
de ~55.400 definiciones. Login con Google, ajustes e historial guardados en Firestore.

## Archivos

- `index.html` — la app completa (HTML/CSS/JS, sin build ni frameworks).
- `words.json` — banco de palabras ya parseado desde tu archivo Anki, con nivel
  de dificultad 1–6 asignado a cada respuesta (7.7 MB).
- `firebase-config.js` — claves de tu proyecto de Firebase (debes completarlas).
- `firestore.rules` — reglas de seguridad para que cada usuario solo vea sus datos.

## 1. Crear el proyecto de Firebase

1. Ve a https://console.firebase.google.com → **Agregar proyecto**.
2. Cuando termine de crearse, entra a **Compilación → Authentication → Comenzar**.
   - En la pestaña "Sign-in method", habilita **Google**.
3. Entra a **Compilación → Firestore Database → Crear base de datos**.
   - Elige modo **producción** (ya traemos reglas propias) y la región que prefieras.
4. Ve a **Configuración del proyecto** (ícono de engranaje) → **Tus apps** → **Web (</>)**.
   - Registra la app (el nombre puede ser "Rosco Web").
   - Copia el objeto `firebaseConfig` que te muestra y pégalo en `firebase-config.js`,
     reemplazando los valores `TU_...`.

## 2. Configurar dominio autorizado

En **Authentication → Settings → Authorized domains**, agrega el dominio donde vas a
publicar la app (por defecto, si usas Firebase Hosting, tu propio dominio
`tu-proyecto.web.app` ya queda autorizado automáticamente).

## 3. Subir las reglas de Firestore

Puedes pegar el contenido de `firestore.rules` directamente en
**Firestore Database → Reglas** dentro de la consola, y publicar. (Si prefieres usar la
CLI de Firebase, `firebase deploy --only firestore:rules` funciona igual.)

## 4. Desplegar la app (Firebase Hosting)

Con Node instalado:

```bash
npm install -g firebase-tools
firebase login
firebase init hosting
# Cuando pregunte "What do you want to use as your public directory?" → escribe: .
# "Configure as a single-page app?" → No
# No sobrescribas index.html si te lo pregunta

firebase deploy --only hosting
```

Al terminar te entrega una URL tipo `https://tu-proyecto.web.app` — ábrela, inicia
sesión con Google y ya puedes jugar.

## Cómo funciona el juego

- El Rosco tiene 25 letras: 23 fijas (A–Z sin K, Ñ, Q, W) más 2 rotativas que
  van alternando entre K, Ñ, Q y W en cada partida (ver "Niveles de dificultad").
- Para cada letra se elige una definición cuya respuesta tenga el nivel de
  dificultad que corresponde según el escalón actual (ver abajo).
- Puedes **comprobar** (Enter o botón), **pasapalabra** (se guarda para un segundo
  intento más adelante en el Rosco) o **rendirte**.
- Un fallo queda marcado en rojo de forma permanente (no se puede reintentar esa
  letra), igual que en el programa original.
- El tiempo por defecto es 2 minutos, ajustable en **Ajustes** (30 s a 5 min).
- Al terminar (por tiempo, por completar todas las letras, o por rendirte) se guarda
  la partida en tu historial (`users/{uid}/history`) y se actualizan tus estadísticas
  en la pantalla de inicio (últimas 50 partidas, roscos ganados, mejor tiempo, escalón
  actual). El cuadro resumen final muestra, para cada letra, tu respuesta, la
  correcta, y **la definición completa que se te hizo** — útil para repasar por
  qué fallaste sin tener que recordar la pregunta de memoria.

## Niveles de dificultad

Cada palabra del banco tiene asignado un **nivel de 1 (muy fácil) a 6 (muy
difícil)**, según su frecuencia de uso real en español (corpus CREA de la RAE):
nivel 1–5 son palabras que sí aparecen en ese corpus de frecuencias (de más a
menos comunes), y nivel 6 son palabras válidas pero tan raras que ni siquiera
quedaron registradas ahí — a toda respuesta de tu banco de definiciones que no
coincidiera con ninguna palabra del corpus se le asignó nivel 6 por defecto,
tal como pediste.

El juego avanza por una **escalera de 50 escalones**: en el escalón 1 el Rosco
es casi todo vocabulario nivel 1–2 (muy fácil); en el escalón 50 es casi todo
nivel 4–6 (difícil/muy difícil). La proporción de cada nivel dentro de las 25
casillas sube de forma gradual escalón a escalón, sin saltos bruscos (se
calcula interpolando entre un vector "fácil" y uno "difícil" a lo largo de los
50 escalones, tal como se definió para el proyecto). Si a una letra le toca un
nivel para el que no tiene suficientes palabras disponibles (pasa sobre todo
con K, que casi no tiene palabras de nivel 1–2 en español), se usa
automáticamente el nivel más cercano que sí tenga.

Dos modos, elegibles en **Ajustes → Dificultad del Rosco**:

- **Automático** (por defecto): empiezas en el escalón 1 y subes +1 escalón cada
  vez que **completas** un Rosco (aciertas o fallas las 25, no importa cuántas
  fallaste — lo que cuenta es no rendirte ni quedarte sin tiempo a mitad de
  camino). Tu progreso queda guardado en tu cuenta y se muestra en Inicio.
- **Manual**: eliges directamente con un control deslizante en qué escalón (1
  a 50) quieres jugar, sin que tu progreso avance solo. Útil para practicar un
  nivel de dificultad específico repetidamente.

El escalón jugado en cada partida queda visible en el marcador durante el
juego y en el resumen final, y se guarda en tu historial.

## Ajuste de texto aplicado al banco de palabras

Tu archivo original (formato Anki) tenía variantes complejas en las respuestas.
Se procesó así:

- **Sinónimos separados por espacio** (ej. `CUSCURRO CURRUSCO CORRUSCO`) → los tres
  se aceptan como respuesta correcta.
- **Contenido entre paréntesis** (ej. `FALCADO, DA (FALCIFORME)`) → se extrae como
  respuesta alternativa adicional (`FALCIFORME`).
- **Variantes de género/número con coma** (ej. `ARTERO, RA`, `ALADAR, ES`) → se
  simplificó aceptando solo la forma principal (`ARTERO`, `ALADAR`); no se derivan
  automáticamente las formas femeninas/plurales porque la regla ortográfica no es
  uniforme en español (`artero→artera` pero `dorado→dorada` funciona distinto).
  Si te importa cubrir estos casos, puedo generarlas a mano por lote más adelante.
- **Tildes**: por defecto la app acepta la respuesta con o sin tilde (como pediste).
  Esto es un ajuste en **Ajustes**, así que si luego prefieres exigir tilde exacta,
  puedes desactivarlo ahí.
- Se descartaron los campos `guid`, `deck` y `tags` del Anki original (no se usan en
  el juego). Si más adelante quieres jugar solo con ciertas categorías (por ejemplo
  excluir `ZZ::*`, o practicar solo `Matasania` = medicina), puedo agregar un filtro
  por categoría en Ajustes — quedó fuera de esta versión porque elegiste usar todo
  el banco.

## Funcionalidades agregadas

1. **Filtro de categorías** (Ajustes → "Categorías incluidas"): puedes desmarcar
   categorías completas (ej. `ZZ`, `Matasania`) para excluirlas del banco de
   palabras usado en cada partida. Hay un botón rápido "Excluir ZZ".
2. **Repaso de errores** (botón en Inicio): junta las palabras falladas de tus
   últimas 15 partidas de Rosco (sin repetir), y te las presenta una por una,
   sin límite de tiempo, hasta un máximo de 30 palabras.
4. **Estadísticas por letra** (botón "Estadísticas"): calcula, sobre tus últimas
   50 partidas, qué letras fallas con más frecuencia (mínimo 2 intentos para
   aparecer en la lista).
5. **Sonido**: pitidos generados en el navegador (sin archivos de audio) para
   acierto, fallo, los últimos 10 segundos del reloj, y la alarma de fin de
   tiempo. Se puede desactivar en Ajustes.
6. **Derivación de género/plural mejorada**: se amplió el procesamiento de
   `words.json` para reconstruir automáticamente formas como `ARTERO`→`ARTERA`,
   `ALADAR`→`ALADARES`, `FALCADO`→`FALCADA`, etc. No es 100% infalible en casos
   irregulares (ej. `AUTOMOTOR` genera además una forma poco natural
   `AUTOMOTRA` junto a la correcta `AUTOMOTRIZ`) — si encuentras casos que
   convenga corregir a mano, dímelo y los ajusto puntualmente.
7. **Modo práctica** (botón "Modo práctica (sin tiempo)"): mismo Rosco completo,
   sin cronómetro, ideal para aprender vocabulario nuevo sin presión. No queda
   guardado en tu historial ni afecta el ranking.
8. **Pista** (botón "Pista (−5s)" durante la partida): revela una letra más de
   la respuesta cada vez que se usa. En el Rosco normal descuenta 5 segundos del
   reloj por cada uso; en modo práctica y repaso es gratuita.
9. **Tolerancia a erratas** (Ajustes → "Tolerar pequeñas erratas"): si tu
   respuesta difiere en 1 sola letra de la correcta (por un error de tipeo), se
   acepta como válida y queda marcada como "(con tolerancia a errata)" en el
   resumen final.
10. **Ranking** (botón "Ranking"): tabla pública con el mejor tiempo de cada
    jugador que haya completado un Rosco (top 10). Se actualiza sola cuando
    logras un tiempo mejor que tu marca anterior guardada.

## Próximos pasos posibles (no incluidos todavía)

- Modo "silla azul" (desempate) como segunda pantalla.
- Filtro de categorías también disponible dentro del modo repaso.
- Estilo visual alternativo si el actual no te convence.
