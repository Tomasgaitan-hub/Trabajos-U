# Trabajos-U

//en este espacio se presneta el codigo utilizado para resolver el problema propuesto en la guia 

import javax.swing.*;
import java.awt.*;


class Nodo {
    String dato;
    Nodo siguiente;
    Nodo anterior;

    public Nodo(String dato) {
        this.dato = dato;
    }
}


class ListaSimple {
    private Nodo cabeza;
    private Nodo cola;

    public void agregar(String dato) {
        Nodo nuevo = new Nodo(dato);
        if (cabeza == null) {
            cabeza = nuevo;
            cola = nuevo;
        } else {
            cola.siguiente = nuevo;
            cola = nuevo;
        }
    }

    public String obtenerTexto() {
        if (cabeza == null) return "(historial vacío)\n";
        StringBuilder sb = new StringBuilder();
        for (Nodo actual = cabeza; actual != null; actual = actual.siguiente) {
            sb.append(actual.dato).append("\n");
        }
        return sb.toString();
    }
}


class ListaDoble {
    private Nodo cabeza;
    private Nodo cola;

    public void agregar(String dato) {
        Nodo nuevo = new Nodo(dato);
        if (cabeza == null) {
            cabeza = nuevo;
            cola = nuevo;
        } else {
            cola.siguiente = nuevo;   
            nuevo.anterior = cola;    
            cola = nuevo;
        }
    }

    
    public String obtenerTexto() {
        if (cabeza == null) return "(sin registros)\n";
        StringBuilder sb = new StringBuilder();
        for (Nodo actual = cabeza; actual != null; actual = actual.siguiente) {
            sb.append(actual.dato).append("\n");
        }
        return sb.toString();
    }

    
    public String obtenerTextoInverso() {
        if (cola == null) return "(sin registros)\n";
        StringBuilder sb = new StringBuilder();
        for (Nodo actual = cola; actual != null; actual = actual.anterior) {
            sb.append(actual.dato).append("\n");
        }
        return sb.toString();
    }
}


class ListaCircular {
    private Nodo ultimo;
    private Nodo actual;   

    public void agregar(String dato) {
        Nodo nuevo = new Nodo(dato);
        if (ultimo == null) {
            nuevo.siguiente = nuevo;          
            ultimo = nuevo;
            actual = nuevo;
        } else {
            nuevo.siguiente = ultimo.siguiente; 
            ultimo.siguiente = nuevo;
            ultimo = nuevo;
        }
    }

    public String actual() {
        return actual == null ? null : actual.dato;
    }

    public String avanzar() {
        if (actual == null) return null;
        actual = actual.siguiente;
        return actual.dato;
    }
}


class Pila {
    private static class NodoPila {
        double[] estado;
        NodoPila siguiente;

        NodoPila(double[] estado) {
            this.estado = estado;
        }
    }

    private NodoPila cima;

    public void push(double[] estado) {
        NodoPila nuevo = new NodoPila(estado.clone());
        nuevo.siguiente = cima;
        cima = nuevo;
    }

    public double[] pop() {
        if (cima == null) return null;
        double[] estado = cima.estado;
        cima = cima.siguiente;
        return estado;
    }

    public boolean estaVacia() {
        return cima == null;
    }
}


class Cola {
    private static class NodoCola {
        Lote dato;
        NodoCola siguiente;

        NodoCola(Lote dato) {
            this.dato = dato;
        }
    }

    private NodoCola frente;
    private NodoCola finalCola;

    public void ofrecer(Lote dato) {
        NodoCola nuevo = new NodoCola(dato);
        if (finalCola == null) {
            frente = nuevo;
        } else {
            finalCola.siguiente = nuevo;
        }
        finalCola = nuevo;
    }

    public Lote extraer() {
        if (frente == null) return null;
        Lote dato = frente.dato;
        frente = frente.siguiente;
        if (frente == null) finalCola = null;
        return dato;
    }

    public boolean estaVacia() {
        return frente == null;
    }
}


class Lote {
    static final double UMBRAL_HUMEDAD = 35.0;

    String nombre;
    double humedad;

    Lote(String nombre, double humedad) {
        this.nombre = nombre;
        this.humedad = humedad;
    }

    boolean necesitaRiego() {
        return humedad < UMBRAL_HUMEDAD;
    }
}


public class SimuladorRiegoGaussSeidel extends JFrame {

    
    private final double[][] matriz = {
        {4.0, -1.0, 0.0},
        {-1.0, 4.0, -1.0},
        {0.0, -1.0, 3.0}
    };

    
    private double[] presiones = {10.0, 8.0, 5.0};
    private final double[] objetivo = {15.0, 10.0, 5.0};

    private static final double TOLERANCIA = 0.001;
    private static final int MAX_ITER = 100;

    private final ListaSimple historial = new ListaSimple();
    private final ListaDoble fugas = new ListaDoble();
    private final ListaCircular sectores = new ListaCircular();
    private final Pila estados = new Pila();
    private final Cola colaHumedad = new Cola();

    private JTextArea area;
    private JLabel lblSector;

    public SimuladorRiegoGaussSeidel() {
        setTitle("Simulador de Riego - Gauss Seidel");
        setSize(1050, 650);
        setDefaultCloseOperation(JFrame.EXIT_ON_CLOSE);
        setLocationRelativeTo(null);
        setLayout(new BorderLayout(10, 10));

        sectores.agregar("Sector Norte (Invernadero 1)");
        sectores.agregar("Sector Central (Cultivo Principal)");
        sectores.agregar("Sector Sur (Área Frutal)");

        colaHumedad.ofrecer(new Lote("Lote 1", 42));
        colaHumedad.ofrecer(new Lote("Lote 2", 28));
        colaHumedad.ofrecer(new Lote("Lote 3", 30));

        fugas.agregar("Nodo 1 - Fricción estable");
        fugas.agregar("Nodo 2 - Pérdida de carga menor");

        armarInterfaz();
    }

    private void armarInterfaz() {
        JPanel arriba = new JPanel(new GridLayout(2, 1, 5, 5));
        arriba.setBorder(BorderFactory.createEmptyBorder(10, 10, 10, 10));

        JLabel titulo = new JLabel("Control de Riego y Balance Hídrico", JLabel.CENTER);
        titulo.setFont(new Font("Arial", Font.BOLD, 16));
        arriba.add(titulo);

        lblSector = new JLabel("Sector actual: " + sectores.actual(), JLabel.CENTER);
        lblSector.setFont(new Font("Arial", Font.ITALIC, 12));
        arriba.add(lblSector);

        add(arriba, BorderLayout.NORTH);

        area = new JTextArea();
        area.setEditable(false);
        area.setFont(new Font("Monospaced", Font.PLAIN, 12));
        JScrollPane scroll = new JScrollPane(area);
        scroll.setBorder(BorderFactory.createTitledBorder("Consola"));
        add(scroll, BorderLayout.CENTER);

        JPanel botones = new JPanel(new FlowLayout(FlowLayout.CENTER, 15, 10));

        JButton btnGauss = new JButton("Calcular presiones");
        JButton btnRotar = new JButton("Siguiente sector");
        JButton btnHumedad = new JButton("Procesar humedad");
        JButton btnUndo = new JButton("Deshacer");
        JButton btnHistorial = new JButton("Ver historial");
        JButton btnVerFugas = new JButton("Ver Trazabilidad Fugas");

        botones.add(btnGauss);
        botones.add(btnRotar);
        botones.add(btnHumedad);
        botones.add(btnUndo);
        botones.add(btnHistorial);
        botones.add(btnVerFugas);

        add(botones, BorderLayout.SOUTH);

        btnGauss.addActionListener(e -> calcularPresiones());
        btnRotar.addActionListener(e -> siguienteSector());
        btnHumedad.addActionListener(e -> procesarHumedad());
        btnUndo.addActionListener(e -> deshacer());

        btnHistorial.addActionListener(e -> {
            area.append("--- Historial de aproximaciones  ---\n");
            area.append(historial.obtenerTexto() + "\n");
        });

        btnVerFugas.addActionListener(e -> {
            area.append("--- Trazabilidad de Fugas  ---\n");
            area.append(fugas.obtenerTexto());
            area.append("--- Trazabilidad de Fugas  ---\n");
            area.append(fugas.obtenerTextoInverso() + "\n");
        });

        area.append("Sistema iniciado\n");
        area.append(String.format("Presiones actuales: [%.2f, %.2f, %.2f]\n\n",
                presiones[0], presiones[1], presiones[2]));
    }

    
    private boolean esDiagonalDominante() {
        for (int i = 0; i < matriz.length; i++) {
            double resto = 0;
            for (int j = 0; j < matriz.length; j++) {
                if (j != i) resto += Math.abs(matriz[i][j]);
            }
            if (Math.abs(matriz[i][i]) <= resto) return false;
        }
        return true;
    }

    private void calcularPresiones() {
        if (!esDiagonalDominante()) {
            area.append("Aviso: la matriz no es diagonalmente dominante; "
                    + "no se garantiza la convergencia.\n\n");
        }

        int n = objetivo.length;
        estados.push(presiones);   

        area.append("--- Calculando Gauss-Seidel ---\n");

        int iter = 0;
        double error;
        do {
            iter++;
            error = 0;
            for (int i = 0; i < n; i++) {
                double suma = objetivo[i];
                for (int j = 0; j < n; j++) {
                    if (j != i) {
                        suma -= matriz[i][j] * presiones[j];
                    }
                }
                double nuevo = suma / matriz[i][i];
                error = Math.max(error, Math.abs(nuevo - presiones[i]));
                presiones[i] = nuevo;
            }

            String linea = String.format("  Iter %d -> [%.4f, %.4f, %.4f]  error = %.5f",
                    iter, presiones[0], presiones[1], presiones[2], error);
            historial.agregar(linea);
            area.append(linea + "\n");
        } while (error > TOLERANCIA && iter < MAX_ITER);

        if (error <= TOLERANCIA) {
            area.append("Convergió en " + iter + " iteraciones (tolerancia " + TOLERANCIA + ").\n\n");
        } else {
            area.append("No convergió en " + MAX_ITER + " iteraciones.\n\n");
        }
    }

    private void siguienteSector() {
        String nuevo = sectores.avanzar();
        lblSector.setText("Sector actual: " + nuevo);
        area.append("Cambié al sector: " + nuevo + "\n");
    }

    private void procesarHumedad() {
        Lote lote = colaHumedad.extraer();
        if (lote == null) {
            area.append("Ya no quedan lotes por revisar.\n\n");
            return;
        }

        area.append(String.format("Revisando -> %s: humedad al %.0f%%\n", lote.nombre, lote.humedad));
        if (lote.necesitaRiego()) {
            area.append(String.format("   Bajo el umbral (%.0f%%): activando riego...\n", Lote.UMBRAL_HUMEDAD));
            lote.humedad = 75;
            area.append(String.format("   Humedad subió a %.0f%%.\n\n", lote.humedad));
        } else {
            area.append("   Humedad ok, no hace falta regar.\n\n");
        }
    }

    private void deshacer() {
        if (!estados.estaVacia()) {
            presiones = estados.pop();
            area.append(String.format("Deshice el cambio. Presiones: [%.2f, %.2f, %.2f]\n\n",
                    presiones[0], presiones[1], presiones[2]));
        } else {
            area.append("No hay nada que deshacer.\n\n");
        }
    }

    public static void main(String[] args) {
        SwingUtilities.invokeLater(() -> new SimuladorRiegoGaussSeidel().setVisible(true));
    }
}
