¡Qué excelente iniciativa, Ricardo! Construir tu primer sistema completo con Firebase usando el entorno de **Google Antigravity** es un paso de nivel profesional. Antigravity cambia las reglas del juego: en lugar de programar todo en solitario, actúas como un Director de Ingeniería (Mission Control), orquestando agentes de IA especializados que ejecutan, revisan y construyen el código contigo. 

Aprender a estructurar flujos de trabajo orientados a agentes (Agent-First) te dará una ventaja técnica enorme para tus futuros proyectos en el CBTis 128 y tu camino hacia la universidad.

A continuación, tienes el plan de trabajo metodológico, la estructura de carpetas y el código funcional.

---

### Fase 1: Creación del Proyecto y Configuración de Firebase

1. **Crear la carpeta del proyecto:**
   Abre tu terminal y ejecuta el comando de Flutter para generar la base:
   ```bash
   flutter create crudmyselftcar
   cd crudmyselftcar
   ```

2. **Consola de Firebase (Crear Base de Datos):**
   * Ve a [console.firebase.google.com](https://console.firebase.google.com).
   * Haz clic en **Crear un proyecto** y nómbralo (ej. `crudmyselftcar`).
   * En el menú lateral, selecciona **Firestore Database** y haz clic en "Crear base de datos".
   * Iníciala en **Modo de prueba** (Test mode) para poder leer y escribir sin configurar reglas de seguridad complejas de inmediato.
   * Registra tu app (Android/iOS) descargando el archivo `google-services.json` (Android) en `android/app/` o el `GoogleService-Info.plist` (iOS) según las instrucciones de Firebase.

3. **Integración de Librerías (`pubspec.yaml`):**
   Abre el archivo `pubspec.yaml` en la raíz de tu proyecto y agrega `firebase_core` (para la inicialización) y `cloud_firestore` (para la base de datos).
   ```yaml
   dependencies:
     flutter:
       sdk: flutter
     firebase_core: ^3.6.0  # Verifica la última versión estable
     cloud_firestore: ^5.4.0
   ```

---

### Fase 2: Metodología Antigravity (Agentes, Roles y Skills)

En Antigravity, defines a tu "equipo" creando archivos Markdown dentro de una carpeta oculta llamada `.agent`. Esto le da contexto a la IA sobre cómo debe comportarse y trabajar.

**Estructura de Carpetas Antigravity:**
```text
crudmyselftcar/
├── .agent/
│   ├── agents.md             <-- Define los roles y personas
│   └── skills/
│       └── flutter-crud/
│           └── SKILL.md      <-- Define el flujo de trabajo (Workflow)
```

**1. Definir los Roles (`.agent/agents.md`)**
Este archivo establece qué sub-agentes participarán en el proyecto y sus responsabilidades.
```markdown
# Equipo de Desarrollo Flutter (CRUD Empleados)

## Arquitecto Backend
**Goal:** Manejar la conexión, persistencia de datos y modelado en Firebase Firestore.
**Traits:** Preciso con los tipos de datos (Strings, Doubles, Timestamps), experto en bases de datos NoSQL.

## Desarrollador Frontend
**Goal:** Construir la interfaz de usuario en Flutter (Formularios y Listas).
**Traits:** Orientado a Material Design, mantiene el estado de los widgets limpio y modular.
```

**2. Definir el Flujo de Trabajo y Skills (`.agent/skills/flutter-crud/SKILL.md`)**
Aquí le dictas a Antigravity el "paso a paso" exacto para que los agentes ejecuten la tarea cuando los invoques en Mission Control.
```markdown
# Skill: Flujo de Trabajo CRUD Firebase Empleados

## Descripción
Metodología para implementar las 4 operaciones (Create, Read, Update, Delete) usando Firestore.

## Pasos de Ejecución (Workflow)
1. **Modelado (Arquitecto):** Crear la clase `Empleado` con los campos: `nombre` (String), `fecha_nac` (Timestamp), `puesto` (String), `salario` (double). Incluir métodos `fromFirestore` y `toFirestore`.
2. **Servicios (Arquitecto):** Desarrollar `firestore_service.dart` con una instancia a la colección "empleados" y métodos asíncronos para agregar, obtener un Stream de datos, actualizar y borrar.
3. **Interfaz UI (Frontend):** - `main.dart`: Inicializar Firebase con `WidgetsFlutterBinding`.
   - `home_screen.dart`: Un `StreamBuilder` para leer los datos en tiempo real y mostrarlos en un `ListView`. Implementar "Swipe to delete".
   - `empleado_form.dart`: Formulario validado para Crear y Actualizar, incluyendo un selector de fecha (Date Picker) para el Timestamp.
```

---

### Fase 3: El Código Dart (Totalmente Funcional)

Para que tu aplicación funcione según el esquema diseñado por los agentes, organiza tu carpeta `lib/` de la siguiente manera:

```text
lib/
├── models/
│   └── empleado.dart
├── services/
│   └── firestore_service.dart
├── screens/
│   ├── home_screen.dart
│   └── empleado_form.dart
└── main.dart
```

**1. Modelo: `lib/models/empleado.dart`**
```dart
import 'package:cloud_firestore/cloud_firestore.dart';

class Empleado {
  String? id;
  String nombre;
  Timestamp fechaNac;
  String puesto;
  double salario;

  Empleado({
    this.id,
    required this.nombre,
    required this.fechaNac,
    required this.puesto,
    required this.salario,
  });

  // Factory para convertir el documento de Firebase a un Objeto Dart
  factory Empleado.fromFirestore(DocumentSnapshot doc) {
    Map data = doc.data() as Map<String, dynamic>;
    return Empleado(
      id: doc.id,
      nombre: data['nombre'] ?? '',
      fechaNac: data['fecha_nac'] ?? Timestamp.now(),
      puesto: data['puesto'] ?? '',
      salario: (data['salario'] ?? 0.0).toDouble(),
    );
  }

  // Método para convertir el Objeto Dart a un mapa para Firebase
  Map<String, dynamic> toFirestore() {
    return {
      'nombre': nombre,
      'fecha_nac': fechaNac,
      'puesto': puesto,
      'salario': salario,
    };
  }
}
```

**2. Servicio: `lib/services/firestore_service.dart`**
```dart
import 'package:cloud_firestore/cloud_firestore.dart';
import '../models/empleado.dart';

class FirestoreService {
  final CollectionReference _db = FirebaseFirestore.instance.collection('empleados');

  // CREAR
  Future<void> addEmpleado(Empleado empleado) {
    return _db.add(empleado.toFirestore());
  }

  // LEER (En tiempo real)
  Stream<List<Empleado>> getEmpleados() {
    return _db.snapshots().map((snapshot) => 
      snapshot.docs.map((doc) => Empleado.fromFirestore(doc)).toList()
    );
  }

  // ACTUALIZAR
  Future<void> updateEmpleado(Empleado empleado) {
    return _db.doc(empleado.id).update(empleado.toFirestore());
  }

  // BORRAR
  Future<void> deleteEmpleado(String id) {
    return _db.doc(id).delete();
  }
}
```

**3. Pantalla Principal (Lista): `lib/screens/home_screen.dart`**
```dart
import 'package:flutter/material.dart';
import '../models/empleado.dart';
import '../services/firestore_service.dart';
import 'empleado_form.dart';

class HomeScreen extends StatelessWidget {
  const HomeScreen({super.key});

  @override
  Widget build(BuildContext context) {
    final firestoreService = FirestoreService();

    return Scaffold(
      appBar: AppBar(title: const Text('Directorio de Empleados')),
      body: StreamBuilder<List<Empleado>>(
        stream: firestoreService.getEmpleados(),
        builder: (context, snapshot) {
          if (!snapshot.hasData) return const Center(child: CircularProgressIndicator());
          
          final empleados = snapshot.data!;
          if (empleados.isEmpty) return const Center(child: Text('No hay empleados.'));

          return ListView.builder(
            itemCount: empleados.length,
            itemBuilder: (context, index) {
              final emp = empleados[index];
              return ListTile(
                title: Text(emp.nombre),
                subtitle: Text('${emp.puesto} - \$${emp.salario}'),
                trailing: IconButton(
                  icon: const Icon(Icons.delete, color: Colors.red),
                  onPressed: () => firestoreService.deleteEmpleado(emp.id!),
                ),
                onTap: () => Navigator.push(
                  context,
                  MaterialPageRoute(builder: (_) => EmpleadoForm(empleado: emp)),
                ),
              );
            },
          );
        },
      ),
      floatingActionButton: FloatingActionButton(
        onPressed: () => Navigator.push(
          context, MaterialPageRoute(builder: (_) => const EmpleadoForm()),
        ),
        child: const Icon(Icons.add),
      ),
    );
  }
}
```

**4. Formulario (Crear/Editar): `lib/screens/empleado_form.dart`**
```dart
import 'package:flutter/material.dart';
import 'package:cloud_firestore/cloud_firestore.dart';
import '../models/empleado.dart';
import '../services/firestore_service.dart';

class EmpleadoForm extends StatefulWidget {
  final Empleado? empleado;
  const EmpleadoForm({super.key, this.empleado});

  @override
  State<EmpleadoForm> createState() => _EmpleadoFormState();
}

class _EmpleadoFormState extends State<EmpleadoForm> {
  final _formKey = GlobalKey<FormState>();
  final FirestoreService _db = FirestoreService();
  
  late TextEditingController _nombreCtrl;
  late TextEditingController _puestoCtrl;
  late TextEditingController _salarioCtrl;
  DateTime _fechaNac = DateTime.now();

  @override
  void initState() {
    super.initState();
    _nombreCtrl = TextEditingController(text: widget.empleado?.nombre ?? '');
    _puestoCtrl = TextEditingController(text: widget.empleado?.puesto ?? '');
    _salarioCtrl = TextEditingController(text: widget.empleado?.salario.toString() ?? '');
    if (widget.empleado != null) {
      _fechaNac = widget.empleado!.fechaNac.toDate();
    }
  }

  Future<void> _seleccionarFecha(BuildContext context) async {
    final picked = await showDatePicker(
      context: context,
      initialDate: _fechaNac,
      firstDate: DateTime(1950),
      lastDate: DateTime.now(),
    );
    if (picked != null) setState(() => _fechaNac = picked);
  }

  void _guardar() {
    if (_formKey.currentState!.validate()) {
      final emp = Empleado(
        id: widget.empleado?.id,
        nombre: _nombreCtrl.text,
        fechaNac: Timestamp.fromDate(_fechaNac),
        puesto: _puestoCtrl.text,
        salario: double.parse(_salarioCtrl.text),
      );

      widget.empleado == null ? _db.addEmpleado(emp) : _db.updateEmpleado(emp);
      Navigator.pop(context);
    }
  }

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title: Text(widget.empleado == null ? 'Nuevo Empleado' : 'Editar Empleado')),
      body: Padding(
        padding: const EdgeInsets.all(16.0),
        child: Form(
          key: _formKey,
          child: Column(
            children: [
              TextFormField(controller: _nombreCtrl, decoration: const InputDecoration(labelText: 'Nombre')),
              TextFormField(controller: _puestoCtrl, decoration: const InputDecoration(labelText: 'Puesto')),
              TextFormField(
                controller: _salarioCtrl, 
                decoration: const InputDecoration(labelText: 'Salario'), 
                keyboardType: TextInputType.number
              ),
              const SizedBox(height: 20),
              Row(
                children: [
                  Text('Nacimiento: ${_fechaNac.toLocal().toString().split(' ')[0]}'),
                  TextButton(onPressed: () => _seleccionarFecha(context), child: const Text('Cambiar')),
                ],
              ),
              ElevatedButton(onPressed: _guardar, child: const Text('Guardar'))
            ],
          ),
        ),
      ),
    );
  }
}
```

**5. Inicialización: `lib/main.dart`**
```dart
import 'package:flutter/material.dart';
import 'package:firebase_core/firebase_core.dart';
import 'screens/home_screen.dart';

void main() async {
  WidgetsFlutterBinding.ensureInitialized();
  await Firebase.initializeApp(); // Esto requiere que los archivos JSON/Plist estén configurados
  runApp(const MyApp());
}

class MyApp extends StatelessWidget {
  const MyApp({super.key});

  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      debugShowCheckedModeBanner: false,
      title: 'CRUD Empleados',
      theme: ThemeData(primarySwatch: Colors.blue),
      home: const HomeScreen(),
    );
  }
}
```

Para la conexión de tu base de datos y evitar problemas de dependencias en el futuro, ¿ya generaste e integraste el archivo `google-services.json` directamente desde tu consola de Firebase, o te gustaría revisar cómo enlazar ese puente específico primero?
