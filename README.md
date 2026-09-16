# El Rosco · Palabros

App web de un solo jugador que simula "El Rosco" de Pasapalabra, usando tu propio banco
de ~55.400 definiciones. Login con Google, ajustes e historial guardados en Firestore.

## Archivos

- `index.html` — la app completa (HTML/CSS/JS, sin build ni frameworks).
- `words.json` — banco de palabras ya parseado desde tu archivo Anki (6.5 MB).
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

- Se arma un Rosco de 26 letras (A–Z, incluyendo Ñ, sin W por tener muy pocas
  entradas en el banco: solo 24 de 55.441).
- Para cada letra se elige al azar una definición de tu banco de palabras.
- Puedes **comprobar** (Enter o botón), **pasapalabra** (se guarda para un segundo
  intento más adelante en el Rosco) o **rendirte**.
- Un fallo queda marcado en rojo de forma permanente (no se puede reintentar esa
  letra), igual que en el programa original.
- El tiempo por defecto es 2 minutos, ajustable en **Ajustes** (30 s a 5 min).
- Al terminar (por tiempo, por completar todas las letras, o por rendirte) se guarda
  la partida en tu historial (`users/{uid}/history`) y se actualizan tus estadísticas
  en la pantalla de inicio (últimas 20 partidas, roscos ganados, mejor tiempo).

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

## Próximos pasos posibles (no incluidos todavía)

- Filtro de categorías/decks en Ajustes.
- Modo "silla azul" (desempate) como segunda pantalla.
- Estadísticas más completas (por letra, palabras más falladas).
- Estilo visual alternativo si el actual no te convence (paleta papel/tinta,
  tipografía Fraunces + IBM Plex Mono).
