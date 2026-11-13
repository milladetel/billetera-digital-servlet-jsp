# Caso 1 – Billetera Digital: Recarga y Movimientos (Servlet + JSP)

Aplicación web en Java puro (sin frameworks) que muestra el saldo y el historial de una
billetera digital y permite registrar **recargas** y **gastos**. La vista se genera en el
servidor (JSP + JSTL) y los datos se guardan en la **sesión HTTP** (`HttpSession`).

## Versiones utilizadas

| Herramienta | Versión |
|---|---|
| Java (JDK) | 17 |
| Apache Maven | 3.9.9 (incluido con Maven Wrapper `mvnw`) |
| Apache Tomcat | 10.1.x (Jakarta EE 10) |
| Jakarta Servlet API | 6.0.0 |
| Jakarta JSTL | 3.0 (API 3.0.0 / impl. GlassFish 3.0.1) |
| Bootstrap | 5.3.3 (CDN) |
| IDE | IntelliJ IDEA |

> Importante: se requiere **Tomcat 10.1** (paquetes `jakarta.*`). Tomcat 9 no funciona.

## Estructura

```
src/main/java/pe/edu/upn/billetera/
├── controller/BilleteraServlet.java   # GET muestra datos, POST procesa movimiento
└── model/
    ├── Billetera.java                 # saldo (S/ 250.00 inicial) + historial
    └── Movimiento.java                # tipo (Recarga/Gasto) + monto
src/main/webapp/WEB-INF/views/
└── billetera.jsp                      # vista con JSTL (<c:forEach>, <c:if>), sin scriptlets
```

## Funcionamiento

- `GET /billetera` → el Servlet obtiene la billetera de la sesión y hace `forward()` a la JSP.
- `POST /billetera` → el Servlet valida:
  - monto vacío → *"El monto es obligatorio."*
  - monto no numérico → *"El monto debe ser un valor numérico."*
  - monto ≤ 0 → *"El monto debe ser mayor que cero."*
  - gasto mayor al saldo → *"Saldo insuficiente para realizar el gasto."*

  Si hay error se muestra en la misma vista **sin modificar** saldo ni historial.
  Si es válido, actualiza saldo, inserta el movimiento **al inicio** del historial y aplica
  **Post-Redirect-Get** (`sendRedirect`) para evitar reenvíos al actualizar (extensión opcional).

## Pasos de ejecución

### 1. Generar el `.war`

```bash
# Windows
mvnw.cmd clean package
# Linux / macOS
./mvnw clean package
```

Se genera `target/billetera.war`.

### 2. Desplegar en Tomcat

1. Descargar Tomcat 10.1 desde https://tomcat.apache.org/download-10.cgi y descomprimirlo.
2. Copiar `target/billetera.war` a la carpeta `webapps/` de Tomcat.
3. Iniciar Tomcat (`bin/startup.bat` en Windows o `bin/startup.sh` en Linux/macOS).
   Se necesita la variable de entorno `JAVA_HOME` apuntando al JDK 17.
4. Abrir: **http://localhost:8080/billetera/**

### Alternativa: desde IntelliJ IDEA

1. *Run → Edit Configurations → + → Tomcat Server → Local*, seleccionar la carpeta de Tomcat 10.1.
2. En la pestaña *Deployment* agregar el artefacto `billetera:war` con contexto `/billetera`.
3. Ejecutar y abrir http://localhost:8080/billetera/

## Pruebas funcionales

| # | Acción | Resultado obtenido |
|---|---|---|
| 1 | Abrir la aplicación | Saldo S/ 250.00; historial vacío ("Aún no hay movimientos") |
| 2 | Recarga de S/ 100.00 | Saldo S/ 350.00; recarga en el historial |
| 3 | Gasto de S/ 50.00 | Saldo S/ 300.00; gasto aparece antes que la recarga |
| 4 | Gasto de S/ 400.00 | Rechazado: "Saldo insuficiente…"; saldo sigue en S/ 300.00 |
| 5 | Monto 0, negativo, vacío o texto | Rechazado con mensaje; no cambia saldo ni historial |
