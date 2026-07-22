import 'dart:convert';
import 'dart:io';
import 'package:flutter/material.dart';
import 'package:flutter_dotenv/flutter_dotenv.dart';
import 'package:image_picker/image_picker.dart';
import 'package:http/http.dart' as http;

// ============================================================
// 1. MODELO DE PRODUCTO
// ============================================================
class Product {
  final String name;
  final double calories;
  final double protein;
  final double carbs;
  final double fat;
  final String halalStatus; // 'halal', 'haram', 'mashbooh', 'unknown'
  final String reason;
  final String ingredientsList;
  final String imagePath;

  Product({
    required this.name,
    required this.calories,
    required this.protein,
    required this.carbs,
    required this.fat,
    required this.halalStatus,
    required this.reason,
    required this.ingredientsList,
    required this.imagePath,
  });

  Map<String, dynamic> toJson() => {
    'name': name,
    'calories': calories,
    'protein': protein,
    'carbs': carbs,
    'fat': fat,
    'halalStatus': halalStatus,
    'reason': reason,
    'ingredientsList': ingredientsList,
    'imagePath': imagePath,
  };

  factory Product.fromJson(Map<String, dynamic> json) => Product(
    name: json['name'],
    calories: json['calories'],
    protein: json['protein'],
    carbs: json['carbs'],
    fat: json['fat'],
    halalStatus: json['halalStatus'],
    reason: json['reason'],
    ingredientsList: json['ingredientsList'],
    imagePath: json['imagePath'],
  );
}

// ============================================================
// 2. BASE DE DATOS HALAL
// ============================================================
class HalalDatabase {
  static final Map<String, Map<String, String>> ingredients = {
    // === HARAM (Prohibidos) ===
    'gelatin': {
      'status': 'haram',
      'reason': 'Por lo general proviene de cerdo, a menos que sea certificado halal'
    },
    'gelatina': {
      'status': 'haram',
      'reason': 'Por lo general proviene de cerdo, a menos que sea certificada halal'
    },
    'alcohol': {
      'status': 'haram',
      'reason': 'El alcohol es haram en todas sus formas'
    },
    'ethanol': {
      'status': 'haram',
      'reason': 'El alcohol etílico es haram'
    },
    'e120': {
      'status': 'haram',
      'reason': 'Ácido carmínico (cochinilla) - proviene de insectos'
    },
    'e1204': {
      'status': 'haram',
      'reason': 'Alcohol polivinílico - derivado del alcohol'
    },
    'bacon': {
      'status': 'haram',
      'reason': 'Cerdo o derivados del cerdo'
    },
    'pork': {
      'status': 'haram',
      'reason': 'Cerdo o derivados del cerdo'
    },
    'lard': {
      'status': 'haram',
      'reason': 'Grasa de cerdo'
    },
    'manteca de cerdo': {
      'status': 'haram',
      'reason': 'Grasa de cerdo'
    },
    'e904': {
      'status': 'haram',
      'reason': 'Goma laca - proviene de insectos'
    },
    'e901': {
      'status': 'haram',
      'reason': 'Cera de abejas - puede ser dudosa según la escuela'
    },
    'whey': {
      'status': 'haram',
      'reason': 'Suero de leche - puede contener enzimas no halal'
    },

    // === MASHBOOH (Dudosos) ===
    'e471': {
      'status': 'mashbooh',
      'reason': 'Mono y diglicéridos - pueden ser de origen animal o vegetal'
    },
    'e472': {
      'status': 'mashbooh',
      'reason': 'Ésteres de mono y diglicéridos - origen incierto'
    },
    'e422': {
      'status': 'mashbooh',
      'reason': 'Glicerina - puede ser de origen animal o vegetal'
    },
    'glycerin': {
      'status': 'mashbooh',
      'reason': 'Glicerina - puede ser de origen animal o vegetal'
    },
    'glycerol': {
      'status': 'mashbooh',
      'reason': 'Glicerol - puede ser de origen animal o vegetal'
    },
    'e475': {
      'status': 'mashbooh',
      'reason': 'Ésteres de poliglicerol - origen incierto'
    },
    'rennet': {
      'status': 'mashbooh',
      'reason': 'Cuajo - puede ser animal (no halal) o microbiano'
    },
    'cuajo': {
      'status': 'mashbooh',
      'reason': 'Cuajo - puede ser animal (no halal) o microbiano'
    },
    'emulsifier': {
      'status': 'mashbooh',
      'reason': 'Emulsionante - origen incierto'
    },
    'emulsionante': {
      'status': 'mashbooh',
      'reason': 'Emulsionante - origen incierto'
    },
    'e476': {
      'status': 'mashbooh',
      'reason': 'Poliglicerol polirricinoleato - origen incierto'
    },
    'e433': {
      'status': 'mashbooh',
      'reason': 'Polisorbato 80 - puede ser de origen animal'
    },
    'e435': {
      'status': 'mashbooh',
      'reason': 'Polisorbato 60 - puede ser de origen animal'
    },

    // === HALAL (Permitidos) ===
    'water': {'status': 'halal', 'reason': 'Ingrediente permitido'},
    'agua': {'status': 'halal', 'reason': 'Ingrediente permitido'},
    'salt': {'status': 'halal', 'reason': 'Ingrediente permitido'},
    'sal': {'status': 'halal', 'reason': 'Ingrediente permitido'},
    'sugar': {'status': 'halal', 'reason': 'Ingrediente permitido'},
    'azúcar': {'status': 'halal', 'reason': 'Ingrediente permitido'},
    'wheat': {'status': 'halal', 'reason': 'Ingrediente permitido'},
    'trigo': {'status': 'halal', 'reason': 'Ingrediente permitido'},
    'corn': {'status': 'halal', 'reason': 'Ingrediente permitido'},
    'maíz': {'status': 'halal', 'reason': 'Ingrediente permitido'},
    'rice': {'status': 'halal', 'reason': 'Ingrediente permitido'},
    'arroz': {'status': 'halal', 'reason': 'Ingrediente permitido'},
    'soy': {'status': 'halal', 'reason': 'Ingrediente permitido'},
    'soja': {'status': 'halal', 'reason': 'Ingrediente permitido'},
    'vegetable': {'status': 'halal', 'reason': 'Ingrediente permitido'},
    'vegetal': {'status': 'halal', 'reason': 'Ingrediente permitido'},
    'fruit': {'status': 'halal', 'reason': 'Ingrediente permitido'},
    'fruta': {'status': 'halal', 'reason': 'Ingrediente permitido'},
    'milk': {'status': 'halal', 'reason': 'Ingrediente permitido (sin alcohol)'},
    'leche': {'status': 'halal', 'reason': 'Ingrediente permitido (sin alcohol)'},
    'egg': {'status': 'halal', 'reason': 'Ingrediente permitido'},
    'huevo': {'status': 'halal', 'reason': 'Ingrediente permitido'},
    'yeast': {'status': 'halal', 'reason': 'Ingrediente permitido'},
    'levadura': {'status': 'halal', 'reason': 'Ingrediente permitido'},
    'oil': {'status': 'halal', 'reason': 'Aceite vegetal - permitido'},
    'aceite': {'status': 'halal', 'reason': 'Aceite vegetal - permitido'},
    'flour': {'status': 'halal', 'reason': 'Harina - permitida'},
    'harina': {'status': 'halal', 'reason': 'Harina - permitida'},
    'cocoa': {'status': 'halal', 'reason': 'Cacao - permitido'},
    'cacao': {'status': 'halal', 'reason': 'Cacao - permitido'},
    'vanilla': {'status': 'halal', 'reason': 'Vainilla - permitida (sin alcohol)'},
    'vainilla': {'status': 'halal', 'reason': 'Vainilla - permitida (sin alcohol)'},
  };

  static Map<String, String> classifyProduct(String ingredientsText) {
    if (ingredientsText.isEmpty) {
      return {
        'status': 'unknown',
        'reason': 'No se pudo leer la lista de ingredientes'
      };
    }

    String lowerText = ingredientsText.toLowerCase();
    List<String> foundIssues = [];
    String overallStatus = 'halal';

    ingredients.forEach((keyword, data) {
      if (lowerText.contains(keyword)) {
        String status = data['status']!;
        String reason = data['reason']!;

        if (status == 'haram') {
          overallStatus = 'haram';
          foundIssues.add('🔴 $keyword: $reason');
        } else if (status == 'mashbooh' && overallStatus != 'haram') {
          if (overallStatus != 'haram') {
            overallStatus = 'mashbooh';
          }
          foundIssues.add('🟡 $keyword: $reason');
        }
      }
    });

    if (foundIssues.isEmpty) {
      return {
        'status': 'halal',
        'reason': '✅ No se encontraron ingredientes prohibidos o dudosos'
      };
    }

    String reasonMessage = foundIssues.join('\n');
    
    if (overallStatus == 'halal' && foundIssues.isNotEmpty) {
      List<String> dudosos = foundIssues.where((issue) => issue.contains('🟡')).toList();
      if (dudosos.isNotEmpty) {
        reasonMessage = '⚠️ Ingredientes dudosos encontrados:\n${dudosos.join('\n')}';
      }
    }

    return {
      'status': overallStatus,
      'reason': reasonMessage,
    };
  }

  static Color getStatusColor(String status) {
    switch (status) {
      case 'halal': return Colors.green;
      case 'haram': return Colors.red;
      case 'mashbooh': return Colors.orange;
      default: return Colors.grey;
    }
  }

  static IconData getStatusIcon(String status) {
    switch (status) {
      case 'halal': return Icons.check_circle;
      case 'haram': return Icons.cancel;
      case 'mashbooh': return Icons.warning;
      default: return Icons.help;
    }
  }

  static String getStatusEmoji(String status) {
    switch (status) {
      case 'halal': return '✅';
      case 'haram': return '❌';
      case 'mashbooh': return '⚠️';
      default: return '❓';
    }
  }
}

// ============================================================
// 3. SERVICIO API (GOOGLE GEMINI)
// ============================================================
class ApiService {
  static final String? _apiKey = dotenv.env['GEMINI_API_KEY'];
  static const String _baseUrl = 'https://generativelanguage.googleapis.com/v1beta/models/gemini-pro-vision:generateContent';

  static Future<Map<String, dynamic>> analyzeImage(File imageFile) async {
    if (_apiKey == null || _apiKey!.isEmpty) {
      print('⚠️ API Key no configurada. Usando datos de prueba...');
      return _getTestData();
    }

    try {
      List<int> imageBytes = await imageFile.readAsBytes();
      String base64Image = base64Encode(imageBytes);

      String prompt = '''
      Analiza esta imagen de un producto alimenticio y devuelve un JSON con esta estructura EXACTA:
      {
        "name": "Nombre del producto",
        "calories": número (calorías por 100g o por porción),
        "protein": número (gramos de proteína),
        "carbs": número (gramos de carbohidratos),
        "fat": número (gramos de grasa),
        "ingredients": "Lista completa de ingredientes separados por comas"
      }
      Si no puedes identificar algún campo, pon 0 o "Desconocido". Solo devuelve el JSON, sin texto adicional.
      ''';

      final url = Uri.parse('$_baseUrl?key=$_apiKey');
      
      final request = {
        'contents': [
          {
            'parts': [
              {'text': prompt},
              {'inline_data': {
                'mime_type': 'image/jpeg',
                'data': base64Image
              }}
            ]
          }
        ]
      };

      final response = await http.post(
        url,
        headers: {'Content-Type': 'application/json'},
        body: jsonEncode(request),
      );

      if (response.statusCode == 200) {
        final data = jsonDecode(response.body);
        final textResponse = data['candidates'][0]['content']['parts'][0]['text'];
        
        String cleanJson = textResponse
            .replaceAll('```json', '')
            .replaceAll('```', '')
            .trim();
        
        final result = jsonDecode(cleanJson);
        
        return {
          'name': result['name'] ?? 'Producto desconocido',
          'calories': (result['calories'] as num?)?.toDouble() ?? 0.0,
          'protein': (result['protein'] as num?)?.toDouble() ?? 0.0,
          'carbs': (result['carbs'] as num?)?.toDouble() ?? 0.0,
          'fat': (result['fat'] as num?)?.toDouble() ?? 0.0,
          'ingredients': result['ingredients'] ?? '',
        };
      } else {
        print('Error API: ${response.statusCode}');
        return _getTestData();
      }
    } catch (e) {
      print('Error al analizar imagen: $e');
      return _getTestData();
    }
  }

  static Map<String, dynamic> _getTestData() {
    return {
      'name': 'Producto de prueba',
      'calories': 250.0,
      'protein': 10.0,
      'carbs': 30.0,
      'fat': 8.0,
      'ingredients': 'agua, azúcar, gelatina, e471, sal, harina, leche',
    };
  }
}

// ============================================================
// 4. PANTALLA DE INICIO (HOME)
// ============================================================
class HomeScreen extends StatelessWidget {
  const HomeScreen({super.key});

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      body: Container(
        decoration: const BoxDecoration(
          gradient: LinearGradient(
            begin: Alignment.topCenter,
            end: Alignment.bottomCenter,
            colors: [Color(0xFF1B5E20), Color(0xFF43A047)],
          ),
        ),
        child: SafeArea(
          child: Center(
            child: Padding(
              padding: const EdgeInsets.all(24.0),
              child: Column(
                mainAxisAlignment: MainAxisAlignment.center,
                children: [
                  Container(
                    padding: const EdgeInsets.all(20),
                    decoration: BoxDecoration(
                      color: Colors.white.withOpacity(0.2),
                      borderRadius: BorderRadius.circular(20),
                    ),
                    child: const Icon(
                      Icons.restaurant_menu,
                      size: 80,
                      color: Colors.white,
                    ),
                  ),
                  const SizedBox(height: 30),
                  const Text(
                    'Halal & Calorie AI',
                    style: TextStyle(
                      fontSize: 36,
                      fontWeight: FontWeight.bold,
                      color: Colors.white,
                      letterSpacing: 1.5,
                    ),
                  ),
                  const SizedBox(height: 10),
                  const Text(
                    'Escanea · Analiza · Descubre',
                    style: TextStyle(
                      fontSize: 18,
                      color: Colors.white70,
                      letterSpacing: 2,
                    ),
                  ),
                  const SizedBox(height: 60),
                  ElevatedButton(
                    onPressed: () {
                      Navigator.push(
                        context,
                        MaterialPageRoute(
                          builder: (context) => const ScanScreen(),
                        ),
                      );
                    },
                    style: ElevatedButton.styleFrom(
                      backgroundColor: Colors.white,
                      foregroundColor: const Color(0xFF1B5E20),
                      padding: const EdgeInsets.symmetric(
                        horizontal: 50,
                        vertical: 20,
                      ),
                      shape: RoundedRectangleBorder(
                        borderRadius: BorderRadius.circular(30),
                      ),
                      elevation: 8,
                    ),
                    child: const Text(
                      'Comenzar Escaneo',
                      style: TextStyle(
                        fontSize: 20,
                        fontWeight: FontWeight.bold,
                      ),
                    ),
                  ),
                  const SizedBox(height: 40),
                  Row(
                    mainAxisAlignment: MainAxisAlignment.center,
                    children: [
                      _buildFeatureIcon(Icons.camera_alt, 'Cámara'),
                      const SizedBox(width: 40),
                      _buildFeatureIcon(Icons.analytics, 'IA'),
                      const SizedBox(width: 40),
                      _buildFeatureIcon(Icons.check_circle, 'Halal'),
                    ],
                  ),
                  const SizedBox(height: 20),
                  Text(
                    'v${dotenv.env['APP_VERSION'] ?? '1.0.0'}',
                    style: const TextStyle(
                      color: Colors.white54,
                      fontSize: 12,
                    ),
                  ),
                ],
              ),
            ),
          ),
        ),
      ),
    );
  }

  Widget _buildFeatureIcon(IconData icon, String label) {
    return Column(
      children: [
        Container(
          padding: const EdgeInsets.all(14),
          decoration: BoxDecoration(
            color: Colors.white.withOpacity(0.2),
            borderRadius: BorderRadius.circular(14),
          ),
          child: Icon(
            icon,
            color: Colors.white,
            size: 30,
          ),
        ),
        const SizedBox(height: 8),
        Text(
          label,
          style: const TextStyle(
            color: Colors.white70,
            fontSize: 13,
          ),
        ),
      ],
    );
  }
}

// ============================================================
// 5. PANTALLA DE ESCANEO (SCAN)
// ============================================================
class ScanScreen extends StatefulWidget {
  const ScanScreen({super.key});

  @override
  State<ScanScreen> createState() => _ScanScreenState();
}

class _ScanScreenState extends State<ScanScreen> {
  bool _isLoading = false;
  String _errorMessage = '';

  Future<void> _pickImage(ImageSource source) async {
    try {
      final ImagePicker picker = ImagePicker();
      final XFile? image = await picker.pickImage(
        source: source,
        maxWidth: 1024,
        imageQuality: 80,
      );

      if (image == null) return;

      setState(() {
        _isLoading = true;
        _errorMessage = '';
      });

      final analysisResult = await ApiService.analyzeImage(File(image.path));

      final halalResult = HalalDatabase.classifyProduct(
        analysisResult['ingredients'] ?? ''
      );

      final product = Product(
        name: analysisResult['name'] ?? 'Producto desconocido',
        calories: analysisResult['calories'] ?? 0.0,
        protein: analysisResult['protein'] ?? 0.0,
        carbs: analysisResult['carbs'] ?? 0.0,
        fat: analysisResult['fat'] ?? 0.0,
        halalStatus: halalResult['status'] ?? 'unknown',
        reason: halalResult['reason'] ?? 'No se pudo determinar',
        ingredientsList: analysisResult['ingredients'] ?? '',
        imagePath: image.path,
      );

      setState(() {
        _isLoading = false;
      });

      if (mounted) {
        Navigator.push(
          context,
          MaterialPageRoute(
            builder: (context) => ResultScreen(product: product),
          ),
        );
      }
    } catch (e) {
      setState(() {
        _isLoading = false;
        _errorMessage = 'Error: ${e.toString()}';
      });
      ScaffoldMessenger.of(context).showSnackBar(
        SnackBar(
          content: Text('Error: $e'),
          backgroundColor: Colors.red,
          duration: const Duration(seconds: 3),
        ),
      );
    }
  }

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(
        title: const Text('Escanear Producto'),
        backgroundColor: const Color(0xFF1B5E20),
        foregroundColor: Colors.white,
        elevation: 0,
        leading: IconButton(
          icon: const Icon(Icons.arrow_back),
          onPressed: () => Navigator.pop(context),
        ),
      ),
      body: Container(
        color: Colors.grey[50],
        child: _isLoading
            ? const Center(
                child: Column(
                  mainAxisAlignment: MainAxisAlignment.center,
                  children: [
                    CircularProgressIndicator(
                      valueColor: AlwaysStoppedAnimation<Color>(Color(0xFF1B5E20)),
                    ),
                    SizedBox(height: 30),
                    Text(
                      'Analizando producto...',
                      style: TextStyle(
                        fontSize: 20,
                        fontWeight: FontWeight.w600,
                      ),
                    ),
                    SizedBox(height: 10),
                    Text(
                      'Esto puede tomar unos segundos',
                      style: TextStyle(
                        fontSize: 14,
                        color: Colors.grey,
                      ),
                    ),
                    SizedBox(height: 20),
                    LinearProgressIndicator(
                      backgroundColor: Colors.grey[200],
                      valueColor: AlwaysStoppedAnimation<Color>(Color(0xFF43A047)),
                    ),
                  ],
                ),
              )
            : Center(
                child: Padding(
                  padding: const EdgeInsets.all(30.0),
                  child: Column(
                    mainAxisAlignment: MainAxisAlignment.center,
                    children: [
                      Container(
                        padding: const EdgeInsets.all(30),
                        decoration: BoxDecoration(
                          color: Colors.white,
                          borderRadius: BorderRadius.circular(20),
                          boxShadow: [
                            BoxShadow(
                              color: Colors.grey.withOpacity(0.3),
                              spreadRadius: 5,
                              blurRadius: 20,
                              offset: const Offset(0, 5),
                            ),
                          ],
                        ),
                        child: const Icon(
                          Icons.camera_alt,
                          size: 80,
                          color: Color(0xFF1B5E20),
                        ),
                      ),
                      const SizedBox(height: 30),
                      const Text(
                        'Toma una foto de la etiqueta',
                        style: TextStyle(
                          fontSize: 22,
                          fontWeight: FontWeight.bold,
                        ),
                      ),
                      const SizedBox(height: 10),
                      const Text(
                        'Enfoca la lista de ingredientes',
                        style: TextStyle(
                          fontSize: 16,
                          color: Colors.grey,
                        ),
                      ),
                      const SizedBox(height: 40),
                      Row(
                        mainAxisAlignment: MainAxisAlignment.center,
                        children: [
                          _buildScanButton(
                            icon: Icons.camera,
                            label: 'Cámara',
                            color: const Color(0xFF1B5E20),
                            onPressed: () => _pickImage(ImageSource.camera),
                          ),
                          const SizedBox(width: 20),
                          _buildScanButton(
                            icon: Icons.photo_library,
                            label: 'Galería',
                            color: Colors.blue[700]!,
                            onPressed: () => _pickImage(ImageSource.gallery),
                          ),
                        ],
                      ),
                      if (_errorMessage.isNotEmpty) ...[
                        const SizedBox(height: 20),
                        Text(
                          _errorMessage,
                          style: const TextStyle(color: Colors.red),
                          textAlign: TextAlign.center,
                        ),
                      ],
                      const SizedBox(height: 30),
                      const Text(
                        '🔍 La IA analizará calorías e ingredientes',
                        style: TextStyle(
                          fontSize: 13,
                          color: Colors.grey,
                        ),
                      ),
                    ],
                  ),
                ),
              ),
      ),
    );
  }

  Widget _buildScanButton({
    required IconData icon,
    required String label,
    required Color color,
    required VoidCallback onPressed,
  }) {
    return ElevatedButton.icon(
      onPressed: onPressed,
      icon: Icon(icon, size: 24),
      label: Text(label),
      style: ElevatedButton.styleFrom(
        backgroundColor: color,
        foregroundColor: Colors.white,
        padding: const EdgeInsets.symmetric(horizontal: 30, vertical: 16),
        shape: RoundedRectangleBorder(
          borderRadius: BorderRadius.circular(15),
        ),
        elevation: 4,
      ),
    );
  }
}

// ============================================================
// 6. PANTALLA DE RESULTADOS (RESULT)
// ============================================================
class ResultScreen extends StatelessWidget {
  final Product product;

  const ResultScreen({super.key, required this.product});

  @override
  Widget build(BuildContext context) {
    Color statusColor = HalalDatabase.getStatusColor(product.halalStatus);
    IconData statusIcon = HalalDatabase.getStatusIcon(product.halalStatus);
    String statusEmoji = HalalDatabase.getStatusEmoji(product.halalStatus);
    
    String statusText = '';
    switch (product.halalStatus) {
      case 'halal': statusText = '✅ HALAL'; break;
      case 'haram': statusText = '❌ HARAM'; break;
      case 'mashbooh': statusText = '⚠️ MASHBOOH'; break;
      default: statusText = '❓ DESCONOCIDO';
    }

    return Scaffold(
      appBar: AppBar(
        title: const Text('Resultado del Análisis'),
        backgroundColor: const Color(0xFF1B5E20),
        foregroundColor: Colors.white,
        elevation: 0,
        actions: [
          IconButton(
            icon: const Icon(Icons.share),
            onPressed: () {
              ScaffoldMessenger.of(context).showSnackBar(
                const SnackBar(
                  content: Text('Compartir resultado (próximamente)'),
                  backgroundColor: Colors.blue,
                ),
              );
            },
          ),
        ],
      ),
      body: SingleChildScrollView(
        padding: const EdgeInsets.all(16.0),
        child: Column(
          crossAxisAlignment: CrossAxisAlignment.start,
          children: [
            // Imagen del producto
            if (product.imagePath.isNotEmpty)
              ClipRRect(
                borderRadius: BorderRadius.circular(16),
                child: Image.file(
                  File(product.imagePath),
                  height: 200,
                  width: double.infinity,
                  fit: BoxFit.cover,
                ),
              ),
            const SizedBox(height: 20),

            // Nombre del producto
            Container(
              padding: const EdgeInsets.all(16),
              decoration: BoxDecoration(
                color: Colors.white,
                borderRadius: BorderRadius.circular(12),
                boxShadow: [
                  BoxShadow(
                    color: Colors.grey.withOpacity(0.1),
                    spreadRadius: 2,
                    blurRadius: 10,
                  ),
                ],
              ),
              child: Row(
                children: [
                  const Icon(Icons.shopping_bag, color: Color(0xFF1B5E20)),
                  const SizedBox(width: 12),
                  Expanded(
                    child: Text(
                      product.name,
                      style: const TextStyle(
                        fontSize: 20,
                        fontWeight: FontWeight.bold,
                      ),
                    ),
                  ),
                ],
              ),
            ),
            const SizedBox(height: 20),

            // === SECCIÓN HALAL ===
            Container(
              padding: const EdgeInsets.all(20),
              decoration: BoxDecoration(
                color: statusColor.withOpacity(0.1),
                borderRadius: BorderRadius.circular(16),
                border: Border.all(
                  color: statusColor,
                  width: 2,
                ),
              ),
              child: Row(
                children: [
                  Container(
                    padding: const EdgeInsets.all(12),
                    decoration: BoxDecoration(
                      color: statusColor.withOpacity(0.2),
                      borderRadius: BorderRadius.circular(12),
                    ),
                    child: Icon(
                      statusIcon,
                      color: statusColor,
                      size: 36,
                    ),
                  ),
                  const SizedBox(width: 16),
                  Expanded(
                    child: Column(
                      crossAxisAlignment: CrossAxisAlignment.start,
                      children: [
                        Text(
                          statusText,
                          style: TextStyle(
                            fontSize: 22,
                            fontWeight: FontWeight.bold,
                            color: statusColor,
                          ),
                        ),
                        const SizedBox(height: 6),
                        Text(
                          product.reason,
                          style: TextStyle(
                            fontSize: 14,
                            color: Colors.grey[800],
                            height: 1.4,
                          ),
                        ),
                      ],
                    ),
                  ),
                ],
              ),
            ),
            const SizedBox(height: 20),

            // === SECCIÓN NUTRICIONAL ===
            const Text(
              '📊 Información Nutricional',
              style: TextStyle(
                fontSize: 18,
                fontWeight: FontWeight.bold,
              ),
            ),
            const SizedBox(height: 10),
            Container(
              decoration: BoxDecoration(
                color: Colors.white,
                borderRadius: BorderRadius.circular(12),
                boxShadow: [
                  BoxShadow(
                    color: Colors.grey.withOpacity(0.1),
                    spreadRadius: 2,
                    blurRadius: 10,
                  ),
                ],
              ),
              child: Column(
                children: [
                  _buildNutrientRow(Icons.local_fire_department, 'Calorías', 
                      '${product.calories.toStringAsFixed(1)} kcal', Colors.red),
                  _buildDivider(),
                  _buildNutrientRow(Icons.fitness_center, 'Proteínas', 
                      '${product.protein.toStringAsFixed(1)} g', Colors.blue),
                  _buildDivider(),
                  _buildNutrientRow(Icons.grain, 'Carbohidratos', 
                      '${product.carbs.toStringAsFixed(1)} g', Colors.orange),
                  _buildDivider(),
                  _buildNutrientRow(Icons.egg, 'Grasas', 
                      '${product.fat.toStringAsFixed(1)} g', Colors.amber),
                ],
              ),
            ),
            const SizedBox(height: 20),

            // === LISTA DE INGREDIENTES ===
            const Text(
              '📝 Ingredientes Detectados',
              style: TextStyle(
                fontSize: 18,
                fontWeight: FontWeight.bold,
              ),
            ),
            const SizedBox(height: 10),
            Container(
              padding: const EdgeInsets.all(16),
              decoration: BoxDecoration(
                color: Colors.grey[50],
                borderRadius: BorderRadius.circular(12),
                border: Border.all(color: Colors.grey[200]!),
              ),
              child: Text(
                product.ingredientsList.isEmpty
                    ? 'No se pudieron detectar ingredientes'
                    : product.ingredientsList,
                style: const TextStyle(
                  fontSize: 14,
                  height: 1.6,
                ),
              ),
            ),
            const SizedBox(height: 30),

            // Botones de acción
            Row(
              children: [
                Expanded(
                  child: ElevatedButton.icon(
                    onPressed: () {
                      ScaffoldMessenger.of(context).showSnackBar(
                        const SnackBar(
                          content: Text('✅ Producto guardado en historial'),
                          backgroundColor: Colors.green,
                          duration: Duration(seconds: 2),
                        ),
                      );
                    },
                    icon: const Icon(Icons.save),
                    label: const Text('Guardar'),
                    style: ElevatedButton.styleFrom(
                      backgroundColor: Colors.blue[700],
                      foregroundColor: Colors.white,
                      padding: const EdgeInsets.symmetric(vertical: 16),
                      shape: RoundedRectangleBorder(
                        borderRadius: BorderRadius.circular(12),
                      ),
                    ),
                  ),
                ),
                const SizedBox(width: 12),
                Expanded(
                  child: ElevatedButton.icon(
                    onPressed: () {
                      Navigator.pushReplacement(
                        context,
                        MaterialPageRoute(
                          builder: (context) => const ScanScreen(),
                        ),
                      );
                    },
                    icon: const Icon(Icons.camera_alt),
                    label: const Text('Escanear otro'),
                    style: ElevatedButton.styleFrom(
                      backgroundColor: const Color(0xFF1B5E20),
                      foregroundColor: Colors.white,
                      padding: const EdgeInsets.symmetric(vertical: 16),
                      shape: RoundedRectangleBorder(
                        borderRadius: BorderRadius.circular(12),
                      ),
                    ),
                  ),
                ),
              ],
            ),
            const SizedBox(height: 20),

            // Botón para ir al inicio
            Center(
              child: TextButton.icon(
                onPressed: () {
                  Navigator.popUntil(context, (route) => route.isFirst);
                },
                icon: const Icon(Icons.home, size: 18),
                label: const Text('Ir al inicio'),
                style: TextButton.styleFrom(
                  foregroundColor: Colors.grey[600],
                ),
              ),
            ),
            const SizedBox(height: 20),
          ],
        ),
      ),
    );
  }

  Widget _buildNutrientRow(IconData icon, String label, String value, Color color) {
    return Padding(
      padding: const EdgeInsets.symmetric(horizontal: 16, vertical: 12),
      child: Row(
        mainAxisAlignment: MainAxisAlignment.spaceBetween,
        children: [
          Row(
            children: [
              Icon(icon, color: color, size: 22),
              const SizedBox(width: 12),
              Text(
                label,
                style: const TextStyle(
                  fontSize: 16,
                  fontWeight: FontWeight.w500,
                ),
              ),
            ],
          ),
          Text(
            value,
            style: TextStyle(
              fontSize: 18,
              fontWeight: FontWeight.bold,
              color: color,
            ),
          ),
        ],
      ),
    );
  }

  Widget _buildDivider() {
    return Divider(
      height: 1,
      color: Colors.grey[200],
      indent: 16,
      endIndent: 16,
    );
  }
}

// ============================================================
// 7. APLICACIÓN PRINCIPAL (MAIN)
// ============================================================
void main() async {
  WidgetsFlutterBinding.ensureInitialized();
  await dotenv.load(fileName: '.env');
  runApp(const MyApp());
}

class MyApp extends StatelessWidget {
  const MyApp({super.key});

  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      title: dotenv.env['APP_NAME'] ?? 'Halal & Calorie AI',
      theme: ThemeData(
        useMaterial3: true,
        primaryColor: const Color(0xFF1B5E20),
        colorScheme: const ColorScheme.light(
          primary: Color(0xFF1B5E20),
          secondary: Color(0xFF43A047),
        ),
        appBarTheme: const AppBarTheme(
          backgroundColor: Color(0xFF1B5E20),
          foregroundColor: Colors.white,
          elevation: 0,
        ),
        elevatedButtonTheme: ElevatedButtonThemeData(
          style: ElevatedButton.styleFrom(
            shape: RoundedRectangleBorder(
              borderRadius: BorderRadius.circular(12),
            ),
          ),
        ),
      ),
      home: const HomeScreen(),
      debugShowCheckedModeBanner: false,
    );
  }
}
