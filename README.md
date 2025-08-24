# Microframeworks-WEB

Mini framework HTTP en Java (sin frameworks externos) para:

* Servir **archivos estáticos**.
* Definir **rutas GET** con **lambdas**.
* Leer **parámetros de consulta** (`?name=...`).
* Endpoint demo **/stocks** (proxy a AlphaVantage) que siempre devuelve **JSON**.

> Proyecto Maven (Java 21). Probado en Windows con Git Bash / IntelliJ.

---

## 📁 Estructura

```
src/
 └─ main/
     ├─ java/
     │   ├─ com/mycompany/httpserver/
     │   │   ├─ HttpServer.java       # núcleo del framework
     │   │   ├─ HttpRequest.java      # acceso a query params
     │   │   ├─ HttpResponse.java     # placeholder para futuro (headers/estado)
     │   │   └─ Service.java          # interfaz funcional (lambda)
     │   └─ com/mycompany/webapplication/
     │       └─ WebAplication.java    # ejemplo de uso
     └─ resources/
         └─ static/                   # web app (frontend)
             ├─ index.html
             ├─ css/styles.css
             ├─ js/app.js
             └─ images/logo.png
```

---

## 🚀 Cómo correr

### Opción 1: Maven + `java`

```bash
# en la raíz del proyecto
mvn -q -DskipTests package
# puerto por defecto 8080; aquí lo fijamos a 35000
java -cp target/classes com.mycompany.webapplication.WebAplication 35000
```

### Opción 2: Scripts

```bash
# Windows
./run.bat
# Linux/macOS
./run.sh
```

La app quedará en: `http://localhost:35000/`
(usa tu puerto si pasaste otro argumento o 8080 por defecto).

---

## 🧩 Endpoints y ejemplos

### Estáticos

* `GET /` → `index.html`
* `GET /css/styles.css`, `/js/app.js`, `/images/logo.png`

> `staticfiles("/static")` copia desde `src/main/resources/static` → `target/classes/static` y sirve desde allí (si no existe, sirve directo desde `resources` en modo dev).

### REST (GET)

```java
// WebAplication.java
HttpServer.staticfiles("/static");

HttpServer.get("/app/hello", (req, resp) ->
    "{\"message\":\"Hello " + escape(req.getValues("name")) + "\"}"
);

HttpServer.get("/app/pi", (req, resp) ->
    "{\"pi\":\"" + Math.PI + "\"}"
);

// Proxy demo a AlphaVantage (si no hay API key usa "demo" y fuerza IBM)
HttpServer.get("/stocks", (req, resp) -> { /* ...ver código del repo... */ });

HttpServer.startServer(new String[]{"35000"});
```

* `GET /app/hello?name=John` → `{"message":"Hello John"}`
* `GET /app/pi` → `{"pi":"3.141592653589793"}`
* `GET /stocks?symbol=fb`

  * Con **tu API key**: devuelve el JSON de AlphaVantage.
  * Con **demo key**: AlphaVantage solo permite **IBM**. El framework devuelve:

    ```json
    {
      "notice":"demo key in use; AlphaVantage solo permite 'IBM'. Requested='fb', served='IBM'.",
      "data": { ... JSON de AlphaVantage ... }
    }
    ```

### cURL rápido

```bash
curl -i http://localhost:35000/index.html
curl -i "http://localhost:35000/app/hello?name=John"
curl -i http://localhost:35000/app/pi
curl -i "http://localhost:35000/stocks?symbol=IBM"
```

---

## 🔑 API Key (AlphaVantage)

Para datos reales en `/stocks` (cualquier símbolo), define `ALPHAVANTAGE_API_KEY`.

**Windows PowerShell**

```powershell
$env:ALPHAVANTAGE_API_KEY="TU_KEY"
```

**CMD**

```cmd
set ALPHAVANTAGE_API_KEY=TU_KEY
```

**IntelliJ IDEA**
Run → Edit Configurations → Environment variables →
`ALPHAVANTAGE_API_KEY=TU_KEY`

---

## 🛠️ Diseño / Detalles técnicos

* **Router case-insensitive**: las rutas se guardan y buscan en minúsculas (`/App/Hello` ≡ `/app/hello`).
* **Lambdas**: `Service` es una interfaz funcional (`String handle(HttpRequest, HttpResponse)`).
* **Query params**: `HttpRequest#getValues("name")` (URL-decoded).
* **Binarios seguros**: para estáticos se escribe con `OutputStream` y `Content-Length`.
* **MIME**: detección por extensión con fallback `application/octet-stream`.
* **Contenido JSON**: respuestas de handlers se envían con `application/json; charset=utf-8`.

---

## 🧭 Roadmap (Lab #2 sugerido)

* [ ] Pool de hilos (concurrency).
* [ ] Soporte **HEAD** y **POST**.
* [ ] Cache-Control para estáticos.
* [ ] 404.html personalizada.
* [ ] Middleware sencillo (logs/headers).

---

## 🧹 Git

`.gitignore` incluye:

```
target/
.idea/
*.iml
out/
nbproject/
dist/
build/
```

---

## 🩺 Troubleshooting

* **404 en `/app/hello`**: revisa que la ruta esté registrada y usa minúsculas en la URL (router case-insensitive, pero mejor mantener consistencia).
* **Mensaje “Information: demo…” en `/stocks`**: estás usando la key `demo`. Configura tu API key o prueba con `symbol=IBM`.
* **Puerto ocupado**: cambia el argumento (p.ej. `9090`) o cierra la instancia previa.

---

## 📄 Licencia

Este proyecto incluye un archivo `LICENSE`. úsalo conforme corresponda.
