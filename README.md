# Basic-Calculator
A simple yet fully functional calculator app built with Flutter. Supports decimals, %, backspace, and basic operations — with a modern UI. ✅ Runs on Android &amp; iOS ✅ No external dependencies required ✅ Built-in expression parser (pure Dart)
import 'package:flutter/material.dart';
import 'dart:math';

void main() {
  runApp(const CalculatorApp());
}

class CalculatorApp extends StatelessWidget {
  const CalculatorApp({super.key});

  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      title: 'Calculator by Muhammad Umar',
      debugShowCheckedModeBanner: false,
      themeMode: ThemeMode.system,
      theme: ThemeData.light(useMaterial3: true),
      darkTheme: ThemeData.dark(useMaterial3: true),
      home: const CalculatorHome(),
    );
  }
}

class CalculatorHome extends StatefulWidget {
  const CalculatorHome({super.key});

  @override
  State<CalculatorHome> createState() => _CalculatorHomeState();
}

class _CalculatorHomeState extends State<CalculatorHome> {
  String input = '';
  String result = '';

  final List<String> buttons = [
    'C', '⌫', '%', '/',
    '7', '8', '9', '×',
    '4', '5', '6', '-',
    '1', '2', '3', '+',
    '00', '0', '.', '='
  ];

  void onButtonPressed(String btn) {
    setState(() {
      if (btn == 'C') {
        input = '';
        result = '';
      } else if (btn == '⌫') {
        input = input.isNotEmpty ? input.substring(0, input.length - 1) : '';
      } else if (btn == '=') {
        calculateResult();
      } else {
        input += btn;
      }
    });
  }

  void calculateResult() {
    try {
      String finalInput = input.replaceAll('×', '*').replaceAll('÷', '/');
      finalInput = finalInput.replaceAll('%', '/100');
      result = _evaluate(finalInput);
    } catch (e) {
      result = 'Error';
    }
  }

  String _evaluate(String expression) {
    try {
      expression = expression.replaceAll(RegExp(r'[^0-9\.\+\-\*/]'), '');
      Parser parser = Parser(expression);
      double value = parser.parse();
      return value.toStringAsFixed(value.truncateToDouble() == value ? 0 : 4);
    } catch (e) {
      return 'Error';
    }
  }

  @override
  Widget build(BuildContext context) {
    bool isDark = Theme.of(context).brightness == Brightness.dark;

    return Scaffold(
      body: SafeArea(
        child: Column(
          children: [
            // ✅ Title at the top
            Padding(
              padding: const EdgeInsets.symmetric(vertical: 16),
              child: Text(
                'This Calculator is Made by Muhammad Umar',
                style: TextStyle(
                  fontSize: 18,
                  fontWeight: FontWeight.w600,
                  color: isDark ? Colors.white70 : Colors.black87,
                ),
                textAlign: TextAlign.center,
              ),
            ),

            // ✅ Display Area
            Expanded(
              flex: 2,
              child: Container(
                padding: const EdgeInsets.all(24),
                alignment: Alignment.bottomRight,
                child: Column(
                  mainAxisAlignment: MainAxisAlignment.end,
                  crossAxisAlignment: CrossAxisAlignment.end,
                  children: [
                    Text(input, style: const TextStyle(fontSize: 28, color: Colors.grey)),
                    const SizedBox(height: 10),
                    Text(result, style: const TextStyle(fontSize: 48, fontWeight: FontWeight.bold)),
                  ],
                ),
              ),
            ),

            // ✅ Button Grid with better spacing
            Expanded(
              flex: 4,
              child: GridView.builder(
                padding: const EdgeInsets.all(12),
                itemCount: buttons.length,
                gridDelegate: const SliverGridDelegateWithFixedCrossAxisCount(
                  crossAxisCount: 4,
                  mainAxisSpacing: 12,
                  crossAxisSpacing: 12,
                ),
                itemBuilder: (context, index) {
                  final btn = buttons[index];
                  return ElevatedButton(
                    onPressed: () => onButtonPressed(btn),
                    style: ElevatedButton.styleFrom(
                      backgroundColor: _getColor(btn, isDark),
                      shape: RoundedRectangleBorder(
                        borderRadius: BorderRadius.circular(16),
                      ),
                      padding: const EdgeInsets.all(16),
                    ),
                    child: FittedBox(
                      child: Text(
                        btn,
                        style: TextStyle(
                          fontSize: 24,
                          fontWeight: FontWeight.bold,
                          color: btn == '=' ? Colors.white : (isDark ? Colors.white70 : Colors.black87),
                        ),
                      ),
                    ),
                  );
                },
              ),
            ),
          ],
        ),
      ),
    );
  }

  Color _getColor(String btn, bool isDark) {
    if (btn == '=') return Colors.orange;
    if (['+', '-', '×', '/', '%'].contains(btn)) return isDark ? Colors.grey[800]! : Colors.grey[300]!;
    if (btn == 'C' || btn == '⌫') return isDark ? Colors.red[400]! : Colors.red[100]!;
    return isDark ? Colors.grey[900]! : Colors.white;
  }
}

// ✅ Dart Expression Parser (No external packages)
class Parser {
  final String exp;
  int pos = -1;
  late int ch;

  Parser(this.exp);

  double parse() {
    _nextChar();
    double x = _parseExpression();
    if (pos < exp.length) throw Exception("Unexpected: ${String.fromCharCode(ch)}");
    return x;
  }

  void _nextChar() {
    ch = (++pos < exp.length) ? exp.codeUnitAt(pos) : -1;
  }

  bool _eat(int charToEat) {
    while (ch == 32) _nextChar();
    if (ch == charToEat) {
      _nextChar();
      return true;
    }
    return false;
  }

  double _parseExpression() {
    double x = _parseTerm();
    while (true) {
      if (_eat('+'.codeUnitAt(0))) x += _parseTerm();
      else if (_eat('-'.codeUnitAt(0))) x -= _parseTerm();
      else return x;
    }
  }

  double _parseTerm() {
    double x = _parseFactor();
    while (true) {
      if (_eat('*'.codeUnitAt(0))) x *= _parseFactor();
      else if (_eat('/'.codeUnitAt(0))) x /= _parseFactor();
      else return x;
    }
  }

  double _parseFactor() {
    if (_eat('+'.codeUnitAt(0))) return _parseFactor();
    if (_eat('-'.codeUnitAt(0))) return -_parseFactor();

    double x;
    int startPos = pos;
    if (_eat('('.codeUnitAt(0))) {
      x = _parseExpression();
      _eat(')'.codeUnitAt(0));
    } else if ((ch >= '0'.codeUnitAt(0) && ch <= '9'.codeUnitAt(0)) || ch == '.'.codeUnitAt(0)) {
      while ((ch >= '0'.codeUnitAt(0) && ch <= '9'.codeUnitAt(0)) || ch == '.'.codeUnitAt(0)) _nextChar();
      x = double.parse(exp.substring(startPos, pos));
    } else {
      throw Exception("Unexpected: ${String.fromCharCode(ch)}");
    }

    return x;
  }
}
