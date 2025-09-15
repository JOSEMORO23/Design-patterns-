# Instituto Tecnológico del Azuay

**Asignatura:** Patrones de Diseño
**Docente:** Veronica Segarra*
**Entrega:** **Subir a GitHub hasta el *30 de septiembre de 2025***

---

## Instrucciones Generales

* Lenguaje sugerido: **Java** (puedes adaptar a C# o Python si lo prefieres).
* Crea un repositorio en GitHub con el nombre: `patrones-diseno-nombre-del-estudiante`.
* Estructura del repo:

  ```
  /src
    /ej01_singleton
    /ej02_factory_method
    /ej03_adapter
    /ej04_observer
    /ej05_decorator
    /ej06_command
    /ej07_composite
    /ej08_strategy
    /ej09_chain_of_responsibility
  /README.md
  ```
* Cada ejercicio debe compilar y ejecutarse de forma **independiente** (con su `main`).
* Documenta con comentarios **qué patrón** usas, **por qué** y **cómo**.
* Incluye al menos **1 prueba mínima** o salida por consola que demuestre el patrón.
* **Fecha límite inamovible:** 30/09/2025 (23:59, hora local).

---

##  Ejercicios Fáciles

### EJ01 — Singleton básico

**Objetivo:** Garantizar una sola instancia de una clase.

**Descripción:** Implementa `ConexionBD` con método estático `getInstance()` y prueba que dos llamadas devuelven el mismo objeto.

**Requisitos mínimos:**

* Constructor **privado**.
* Asegurar **thread-safety** (usa *holder* o `synchronized`).
* Método `conectar()` que imprima un mensaje.

**Código base (Java):**

```java
public final class ConexionBD {
    private static class Holder {
        private static final ConexionBD INSTANCE = new ConexionBD();
    }
    private ConexionBD() { /* TODO: configurar conexión */ }
    public static ConexionBD getInstance() { return Holder.INSTANCE; }
    public void conectar() { System.out.println("Conectado a la BD (única instancia)"); }
}

class Main {
    public static void main(String[] args) {
        ConexionBD a = ConexionBD.getInstance();
        ConexionBD b = ConexionBD.getInstance();
        System.out.println("Misma instancia: " + (a == b));
        a.conectar();
    }
}
```

---

### EJ02 — Factory Method simple

**Objetivo:** Crear objetos sin acoplarse a clases concretas.

**Descripción:** Fábrica de figuras: `Circulo`, `Cuadrado`. La fábrica retorna según un `tipo`.

**Código base:**

```java
interface Figura { double area(); }
class Circulo implements Figura { private final double r; Circulo(double r){this.r=r;} public double area(){return Math.PI*r*r;} }
class Cuadrado implements Figura { private final double l; Cuadrado(double l){this.l=l;} public double area(){return l*l;} }

abstract class CreadorFiguras {
    public abstract Figura crear(String tipo);
}
class CreadorFigurasConcreto extends CreadorFiguras {
    public Figura crear(String tipo){
        return switch (tipo.toLowerCase()) {
            case "circulo" -> new Circulo(2);
            case "cuadrado" -> new Cuadrado(3);
            default -> throw new IllegalArgumentException("Tipo no soportado: "+tipo);
        };
    }
}

class MainFM {
    public static void main(String[] args){
        CreadorFiguras creador = new CreadorFigurasConcreto();
        Figura f1 = creador.crear("circulo");
        Figura f2 = creador.crear("cuadrado");
        System.out.println("Area círculo: "+f1.area());
        System.out.println("Area cuadrado: "+f2.area());
    }
}
```

---

### EJ03 — Adapter sencillo

**Objetivo:** Hacer compatibles interfaces incompatibles.

**Descripción:** `AudioPlayer` reproduce `.mp3`. Agrega un `Mp4Adapter` para reproducir `.mp4`.

**Código base:**

```java
interface Reproductor { void play(String archivo); }
class AudioPlayer implements Reproductor {
    public void play(String archivo) {
        if (archivo.endsWith(".mp3")) System.out.println("Reproduciendo MP3: "+archivo);
        else throw new UnsupportedOperationException("Formato no soportado");
    }
}
class Mp4Lib { void playMp4(String archivo){ System.out.println("MP4 -> "+archivo); } }
class Mp4Adapter implements Reproductor {
    private final Mp4Lib lib = new Mp4Lib();
    public void play(String archivo){
        if (archivo.endsWith(".mp4")) lib.playMp4(archivo);
        else new AudioPlayer().play(archivo);
    }
}
class MainAdapter {
    public static void main(String[] args){
        Reproductor r = new Mp4Adapter();
        r.play("cancion.mp3");
        r.play("video.mp4");
    }
}
```

---

##  Ejercicios Normales

### EJ04 — Observer (Publicador/Suscriptor)

**Objetivo:** Notificar cambios a múltiples observadores.

**Descripción:** `Canal` notifica a `Usuario` cuando hay un nuevo video.

**Código base:**

```java
import java.util.*;
interface Observador { void actualizar(String titulo); }
interface Sujeto {
    void suscribir(Observador o);
    void desuscribir(Observador o);
    void notificar(String titulo);
}
class Canal implements Sujeto {
    private final List<Observador> obs = new ArrayList<>();
    public void suscribir(Observador o){ obs.add(o);}
    public void desuscribir(Observador o){ obs.remove(o);}
    public void notificar(String titulo){ obs.forEach(o -> o.actualizar(titulo)); }
    public void subirVideo(String titulo){ System.out.println("Nuevo video: "+titulo); notificar(titulo);}    
}
class Usuario implements Observador {
    private final String nombre;
    public Usuario(String n){this.nombre=n;}
    public void actualizar(String titulo){ System.out.println(nombre+" recibió notificación: "+titulo); }
}
class MainObs {
    public static void main(String[] args){
        Canal c = new Canal();
        Observador u1 = new Usuario("Ana");
        Observador u2 = new Usuario("Luis");
        c.suscribir(u1); c.suscribir(u2);
        c.subirVideo("Patrón Observer en 5 minutos");
        c.desuscribir(u1);
        c.subirVideo("Refactorizando con patrones");
    }
}
```

---

### EJ05 — Decorator

**Objetivo:** Añadir responsabilidades dinámicamente a un objeto.

**Descripción:** `Bebida` con decoradores `ConLeche`, `ConAzucar` que alteran costo y descripción.

**Código base:**

```java
interface Bebida { String descripcion(); double costo(); }
class Cafe implements Bebida { public String descripcion(){return "Café";} public double costo(){return 1.0;} }
abstract class BebidaDecorator implements Bebida { protected final Bebida base; BebidaDecorator(Bebida b){this.base=b;} }
class ConLeche extends BebidaDecorator { ConLeche(Bebida b){super(b);} public String descripcion(){return base.descripcion()+" + leche";} public double costo(){return base.costo()+0.3;} }
class ConAzucar extends BebidaDecorator { ConAzucar(Bebida b){super(b);} public String descripcion(){return base.descripcion()+" + azúcar";} public double costo(){return base.costo()+0.1;} }
class MainDeco {
    public static void main(String[] args){
        Bebida b = new ConAzucar(new ConLeche(new Cafe()));
        System.out.println(b.descripcion()+" => $"+b.costo());
    }
}
```

---

### EJ06 — Command (con deshacer)

**Objetivo:** Encapsular solicitudes como objetos y permitir *undo*.

**Descripción:** Control remoto con comandos `Encender`/`Apagar` para una `Lampara`.

**Código base:**

```java
import java.util.*;
interface Command { void ejecutar(); void deshacer(); }
class Lampara { boolean on=false; void encender(){on=true; System.out.println("Lámpara ON");} void apagar(){on=false; System.out.println("Lámpara OFF");} }
class EncenderCmd implements Command { private final Lampara l; EncenderCmd(Lampara l){this.l=l;} public void ejecutar(){ l.encender(); } public void deshacer(){ l.apagar(); }}
class ApagarCmd implements Command { private final Lampara l; ApagarCmd(Lampara l){this.l=l;} public void ejecutar(){ l.apagar(); } public void deshacer(){ l.encender(); }}
class ControlRemoto {
    private final Deque<Command> historial = new ArrayDeque<>();
    public void presionar(Command c){ c.ejecutar(); historial.push(c);}
    public void undo(){ if(!historial.isEmpty()) historial.pop().deshacer(); }
}
class MainCmd {
    public static void main(String[] args){
        Lampara l = new Lampara();
        ControlRemoto ctrl = new ControlRemoto();
        ctrl.presionar(new EncenderCmd(l));
        ctrl.presionar(new ApagarCmd(l));
        ctrl.undo(); // vuelve a ON
    }
}
```

---

## Ejercicios Difíciles

### EJ07 — Composite (Sistema de archivos)

**Objetivo:** Tratar objetos individuales y compuestos de forma uniforme.

**Descripción:** `Carpeta` contiene `Archivo` o `Carpeta`. Recorre e imprime estructura y tamaño total.

**Código base:**

```java
import java.util.*;
interface Nodo { String nombre(); long tamano(); void imprimir(String prefijo); }
class Archivo implements Nodo {
    private final String n; private final long t;
    public Archivo(String n, long t){this.n=n;this.t=t;}
    public String nombre(){return n;} public long tamano(){return t;}
    public void imprimir(String p){ System.out.println(p+"- "+n+" ("+t+"B)"); }
}
class Carpeta implements Nodo {
    private final String n; private final List<Nodo> hijos = new ArrayList<>();
    public Carpeta(String n){this.n=n;} public String nombre(){return n;}
    public long tamano(){ return hijos.stream().mapToLong(Nodo::tamano).sum(); }
    public Carpeta agregar(Nodo x){ hijos.add(x); return this; }
    public void imprimir(String p){
        System.out.println(p+"+ "+n+"/ (") ;
        for (Nodo h: hijos) h.imprimir(p+"  ");
        System.out.println(p+") total="+tamano()+"B");
    }
}
class MainComposite {
    public static void main(String[] args){
        Carpeta root = new Carpeta("root")
            .agregar(new Archivo("a.txt", 120))
            .agregar(new Carpeta("img").agregar(new Archivo("logo.png", 2048)));
        root.imprimir("");
    }
}
```

---

### EJ08 — Strategy (Pagos)

**Objetivo:** Intercambiar algoritmos en tiempo de ejecución.

**Descripción:** Estrategias de pago: `PayPal`, `Tarjeta`, `Cripto`.

**Código base:**

```java
interface EstrategiaPago { void pagar(double monto); }
class PagoPayPal implements EstrategiaPago { public void pagar(double m){ System.out.println("Pagando $"+m+" con PayPal"); } }
class PagoTarjeta implements EstrategiaPago { public void pagar(double m){ System.out.println("Pagando $"+m+" con Tarjeta"); } }
class PagoCripto implements EstrategiaPago { public void pagar(double m){ System.out.println("Pagando $"+m+" con Cripto"); } }
class Checkout {
    private EstrategiaPago estrategia;
    public void setEstrategia(EstrategiaPago e){ this.estrategia = e; }
    public void procesar(double monto){ if(estrategia==null) throw new IllegalStateException("Estrategia no definida"); estrategia.pagar(monto);}
}
class MainStrategy {
    public static void main(String[] args){
        Checkout c = new Checkout();
        c.setEstrategia(new PagoPayPal()); c.procesar(15.5);
        c.setEstrategia(new PagoTarjeta()); c.procesar(20);
        c.setEstrategia(new PagoCripto()); c.procesar(5.75);
    }
}
```

---

### EJ09 — Chain of Responsibility (Aprobación de gastos)

**Objetivo:** Encadenar manejadores con límites de aprobación.

**Descripción:** `Jefe` (≤1000), `Gerente` (≤5000), `Director` (sin límite).

**Código base:**

```java
abstract class Aprobador {
    protected Aprobador siguiente;
    public Aprobador enlazar(Aprobador s){ this.siguiente = s; return s; }
    public void procesar(double monto){
        if (puedeAprobar(monto)) aprobar(monto);
        else if (siguiente != null) siguiente.procesar(monto);
        else System.out.println("Nadie pudo aprobar $"+monto);
    }
    protected abstract boolean puedeAprobar(double monto);
    protected abstract void aprobar(double monto);
}
class Jefe extends Aprobador {
    protected boolean puedeAprobar(double m){ return m <= 1000; }
    protected void aprobar(double m){ System.out.println("Jefe aprueba $"+m); }
}
class Gerente extends Aprobador {
    protected boolean puedeAprobar(double m){ return m <= 5000; }
    protected void aprobar(double m){ System.out.println("Gerente aprueba $"+m); }
}
class Director extends Aprobador {
    protected boolean puedeAprobar(double m){ return true; }
    protected void aprobar(double m){ System.out.println("Director aprueba $"+m); }
}
class MainChain {
    public static void main(String[] args){
        Aprobador cadena = new Jefe();
        cadena.enlazar(new Gerente()).enlazar(new Director());
        cadena.procesar(500);
        cadena.procesar(3000);
        cadena.procesar(20000);
    }
}
```

---

##  Checklist de Entrega (GitHub)

* [ ] Repo creado: `patrones-diseno-veronica-segarra`.
* [ ] Cada ejercicio en su carpeta con `Main` ejecutable.
* [ ] Comentarios explicando el **patrón** usado.
* [ ] `README.md` con instrucciones de compilación/ejecución.
* [ ] Último *commit* antes del **30/09/2025 23:59**.

---

##  Guía Rápida para Subir a GitHub

1. **Inicializar y primer push**

   ```bash
   git init
   git add .
   git commit -m "Inicio del proyecto de patrones"
   git branch -M main
   git remote add origin https://github.com/<tu-usuario>/patrones-diseno-veronica-segarra.git
   git push -u origin main
   ```
2. **Trabajar por ejercicio**

   ```bash
   # Editas, compilas y pruebas
   git add src/ej01_singleton
   git commit -m "EJ01: Singleton implementado"
   git push
   ```
3. **Antes de la fecha límite (30/09/2025)**

   * Verifica que el repositorio **sea público o compartido con el docente**.
   * Revisa que cada `Main` se ejecute.

---

**¡Éxitos,  Recuerda: claridad de diseño > cantidad de código. Documenta decisiones y mantén el repositorio ordenado.
