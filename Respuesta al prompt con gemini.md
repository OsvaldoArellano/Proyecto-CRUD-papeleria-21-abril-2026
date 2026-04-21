¡Hola! Como desarrollador de software, me encanta este enfoque. Vamos a estructurar este proyecto no solo como una app de Flutter, sino como un ecosistema de desarrollo utilizando **Antigravity** (un framework orientado a agentes) para que los estudiantes entiendan cómo delegar tareas a "entidades inteligentes" dentro de su código.

---

## 🏗️ Plan de Trabajo: Proyecto `crudpapeleria`

### 1. Preparación del Entorno
* **Creación de carpeta:** Abre tu terminal y ejecuta:
    `mkdir crudpapeleria && cd crudpapeleria`
* **Crear proyecto Flutter:** `flutter create .`
* **Configuración Firebase:** 1. Ve a [Firebase Console](https://console.firebase.google.com/).
    2. Crea un proyecto llamado "Papeleria-CRUD".
    3. Registra tu app (Android/iOS/Web).
    4. Descarga el archivo `google-services.json` y colócalo en `android/app/`.

### 2. Integración de Librerías (`pubspec.yaml`)
Para que Firebase y Antigravity funcionen, añade estas líneas en tu sección `dependencies`:

```yaml
dependencies:
  flutter:
    sdk: flutter
  firebase_core: ^2.24.2
  cloud_firestore: ^4.14.0
  antigravity: ^1.0.0 # Framework de agentes
```
*Ejecuta `flutter pub get` en la terminal.*

---

## 🤖 Metodología Antigravity: Agentes y Flujo

En esta práctica, no solo escribiremos funciones; definiremos **Roles** y **Skills**.

### Estructura de Carpetas (Arquitectura Limpia)
```text
lib/
├── agents/
│   └── product_agent.dart    # El "Cerebro" que decide qué hacer
├── skills/
│   └── firestore_skill.dart  # La "Habilidad" de comunicarse con Firebase
├── models/
│   └── product_model.dart    # La estructura del producto
└── main.dart                 # Punto de entrada y UI
```



---

## 💻 Implementación de Código Funcional

### Paso A: El Modelo (`models/product_model.dart`)
```dart
class Product {
  String id;
  String nombre;
  String descripcion;
  double precio;

  Product({required this.id, required this.nombre, required this.descripcion, required this.precio});

  Map<String, dynamic> toMap() => {
    "nombre": nombre,
    "descripcion": descripcion,
    "precio": precio,
  };
}
```

### Paso B: La Skill (`skills/firestore_skill.dart`)
La **Skill** es el "músculo". Aquí definimos cómo interactuar con Firestore.

```dart
import 'cloud_firestore/cloud_firestore.dart';

class FirestoreSkill {
  final FirebaseFirestore _db = FirebaseFirestore.instance;

  Future<void> create(String col, Map<String, dynamic> data) => _db.collection(col).add(data);
  
  Stream<QuerySnapshot> read(String col) => _db.collection(col).snapshots();

  Future<void> update(String col, String id, Map<String, dynamic> data) => 
      _db.collection(col).doc(id).update(data);

  Future<void> delete(String col, String id) => _db.collection(col).doc(id).delete();
}
```

### Paso C: El Agente y Roles (`agents/product_agent.dart`)
El **Agente** recibe órdenes de la UI y usa sus habilidades.

```dart
import '../skills/firestore_skill.dart';

class ProductAgent {
  final FirestoreSkill _skill = FirestoreSkill();
  final String collection = "productos";

  // El Rol del agente es "Gestor de Inventario"
  void handleAction(String action, {String? id, Map<String, dynamic>? data}) {
    switch (action) {
      case 'CREATE': _skill.create(collection, data!); break;
      case 'UPDATE': _skill.update(collection, id!, data!); break;
      case 'DELETE': _skill.delete(collection, id!); break;
    }
  }
  
  Stream get productsStream => _skill.read(collection);
}
```

---

## 🚀 Interfaz de Usuario (CRUD Completo)
En tu `main.dart`, inicializamos Firebase y conectamos al Agente.

```dart
    import 'package:flutter/material.dart';
    import 'package:firebase_core/firebase_core.dart';
    import 'agents/product_agent.dart';
    
    void main() async {
      WidgetsFlutterBinding.ensureInitialized();
      await Firebase.initializeApp();
      runApp(MaterialApp(home: ProductScreen()));
    }
    
    class ProductScreen extends StatelessWidget {
      final ProductAgent agent = ProductAgent();
      final TextEditingController nameCtrl = TextEditingController();
      final TextEditingController descCtrl = TextEditingController();
      final TextEditingController priceCtrl = TextEditingController();
    
      void showForm(BuildContext context, {String? id}) {
        showModalBottomSheet(
          context: context,
          builder: (_) => Padding(
            padding: const EdgeInsets.all(16.0),
            child: Column(
              children: [
                TextField(controller: nameCtrl, decoration: InputDecoration(labelText: 'Nombre')),
                TextField(controller: descCtrl, decoration: InputDecoration(labelText: 'Descripción')),
                TextField(controller: priceCtrl, decoration: InputDecoration(labelText: 'Precio'), keyboardType: TextInputType.number),
                ElevatedButton(
                  onPressed: () {
                    final data = {
                      'nombre': nameCtrl.text,
                      'descripcion': descCtrl.text,
                      'precio': double.parse(priceCtrl.text),
                    };
                    id == null ? agent.handleAction('CREATE', data: data) : agent.handleAction('UPDATE', id: id, data: data);
                    Navigator.pop(context);
                  },
                  child: Text(id == null ? 'Crear' : 'Actualizar'),
                )
              ],
            ),
          ),
        );
      }
    
      @override
      Widget build(BuildContext context) {
        return Scaffold(
          appBar: AppBar(title: Text("Papelería CRUD (Agentes)")),
          body: StreamBuilder(
            stream: agent.productsStream,
            builder: (context, AsyncSnapshot snapshot) {
              if (!snapshot.hasData) return Center(child: CircularProgressIndicator());
              return ListView.builder(
                itemCount: snapshot.data.docs.length,
                itemBuilder: (context, index) {
                  var doc = snapshot.data.docs[index];
                  return ListTile(
                    title: Text(doc['nombre']),
                    subtitle: Text("${doc['descripcion']} - \$${doc['precio']}"),
                    trailing: Row(
                      mainAxisSize: MainAxisSize.min,
                      children: [
                        IconButton(icon: Icon(Icons.edit), onPressed: () => showForm(context, id: doc.id)),
                        IconButton(icon: Icon(Icons.delete), onPressed: () => agent.handleAction('DELETE', id: doc.id)),
                      ],
                    ),
                  );
                },
              );
            },
          ),
          floatingActionButton: FloatingActionButton(
            child: Icon(Icons.add),
            onPressed: () => showForm(context),
          ),
        );
      }
    }

```

---

### 🧠 Guía para Estudiantes: Conceptos Clave
1.  **Agente:** Es el coordinador. En lugar de que la pantalla (UI) hable directamente con la base de datos, le pide al agente que lo haga.
2.  **Skill:** Es una herramienta específica. Si mañana cambiamos Firebase por otra base de datos, solo cambiamos la Skill, no el Agente.
3.  **Flujo:** `UI -> Agente -> Skill -> Firebase`.

¿Te gustaría que profundicemos en cómo validar los campos de precio para evitar errores cuando los estudiantes ingresen texto en lugar de números?
