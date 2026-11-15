import 'package:flutter/material.dart';

void main() {
  // Test conversion functions
  assert((convertValue(1, 'km', 'm', lengthUnits) - 1000).abs() < 0.0001);
  assert((convertValue(1, 'kg', 'g', weightUnits) - 1000).abs() < 0.0001);
  assert((convertValue(100, '°C', '°F', {}) - 212).abs() < 0.0001);

  runApp(
    const MaterialApp(debugShowCheckedModeBanner: false, home: HomePage()),
  );
}

// Length conversion factors (base unit: metre)
const Map<String, double> lengthUnits = {
  'm': 1,
  'km': 1000,
  'cm': 0.01,
  'mile': 1609.344,
};

// Weight conversion factors (base unit: kilogram)
const Map<String, double> weightUnits = {'kg': 1, 'g': 0.001, 'lb': 0.45359237};

// Conversion function
double convertValue(
  double value,
  String from,
  String to,
  Map<String, double> units,
) {
  if (units.isEmpty) {
    double celsius = from == '°F' ? (value - 32) * 5 / 9 : value;

    return to == '°F' ? celsius * 9 / 5 + 32 : celsius;
  }

  double base = value * units[from]!;
  return base / units[to]!;
}

// Home screen
class HomePage extends StatelessWidget {
  const HomePage({super.key});

  void openConverter(
    BuildContext context,
    String title,
    Map<String, double> units,
  ) {
    Navigator.push(
      context,
      MaterialPageRoute(
        builder: (_) => ConverterPage(title: title, units: units),
      ),
    );
  }

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(
        title: const Text('Unit Converter'),
        backgroundColor: Colors.blue,
      ),
      body: ListView(
        padding: const EdgeInsets.all(16),
        children: [
          converterCard(context, 'Length', Icons.straighten, lengthUnits),
          converterCard(context, 'Weight', Icons.monitor_weight, weightUnits),
          converterCard(context, 'Temperature', Icons.thermostat, {}),
        ],
      ),
    );
  }

  Widget converterCard(
    BuildContext context,
    String title,
    IconData icon,
    Map<String, double> units,
  ) {
    return Card(
      child: ListTile(
        leading: Icon(icon, color: Colors.blue),
        title: Text(title),
        trailing: const Icon(Icons.arrow_forward),
        onTap: () => openConverter(context, title, units),
      ),
    );
  }
}

// Reusable converter screen
class ConverterPage extends StatefulWidget {
  final String title;
  final Map<String, double> units;

  const ConverterPage({super.key, required this.title, required this.units});

  @override
  State<ConverterPage> createState() => _ConverterPageState();
}

class _ConverterPageState extends State<ConverterPage> {
  final TextEditingController controller = TextEditingController();

  String fromUnit = '';
  String toUnit = '';

  List<String> get unitList =>
      widget.units.isEmpty ? ['°C', '°F'] : widget.units.keys.toList();

  @override
  void initState() {
    super.initState();
    fromUnit = unitList[0];
    toUnit = unitList[1];
  }

  double? get convertedValue {
    double? value = double.tryParse(controller.text);

    if (value == null) return null;

    return convertValue(value, fromUnit, toUnit, widget.units);
  }

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(
        title: Text('${widget.title} Converter'),
        backgroundColor: Colors.blue,
      ),
      body: Padding(
        padding: const EdgeInsets.all(20),
        child: Column(
          children: [
            TextField(
              controller: controller,
              keyboardType: const TextInputType.numberWithOptions(
                decimal: true,
                signed: true,
              ),
              decoration: const InputDecoration(
                labelText: 'Enter value',
                border: OutlineInputBorder(),
              ),
              onChanged: (_) => setState(() {}),
            ),

            const SizedBox(height: 20),

            const Text('From Unit'),
            DropdownButton<String>(
              isExpanded: true,
              value: fromUnit,
              items: unitList.map((unit) {
                return DropdownMenuItem(value: unit, child: Text(unit));
              }).toList(),
              onChanged: (value) {
                if (value != null) {
                  setState(() => fromUnit = value);
                }
              },
            ),

            const SizedBox(height: 10),

            IconButton(
              onPressed: () {
                setState(() {
                  String temp = fromUnit;
                  fromUnit = toUnit;
                  toUnit = temp;
                });
              },
              icon: const Icon(Icons.swap_vert),
            ),

            const Text('To Unit'),
            DropdownButton<String>(
              isExpanded: true,
              value: toUnit,
              items: unitList.map((unit) {
                return DropdownMenuItem(value: unit, child: Text(unit));
              }).toList(),
              onChanged: (value) {
                if (value != null) {
                  setState(() => toUnit = value);
                }
              },
            ),

            const SizedBox(height: 30),

            const Text('Conversion Result', style: TextStyle(fontSize: 18)),

            const SizedBox(height: 10),

            Text(
              convertedValue == null
                  ? 'Enter a valid number'
                  : '${convertedValue!.toStringAsFixed(4)} $toUnit',
              style: const TextStyle(
                fontSize: 25,
                fontWeight: FontWeight.bold,
                color: Colors.blue,
              ),
            ),
          ],
        ),
      ),
    );
  }

  @override
  void dispose() {
    controller.dispose();
    super.dispose();
  }
}
