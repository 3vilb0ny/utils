## Double extension

```dart
import 'package:flutter/material.dart';
import 'package:pdf/widgets.dart' as pdf_widget;

extension DoubleCustom on double {
  /// Vertical spacer
  SizedBox get vertical {
    return SizedBox(
      height: this,
    );
  }

  /// Horizontal spacer
  SizedBox get horizontal {
    return SizedBox(
      width: this,
    );
  }

  /// Vertical spacer for PDF
  pdf_widget.SizedBox get verticalPDF {
    return pdf_widget.SizedBox(
      height: this,
    );
  }

  /// Horizontal spacer for PDF
  pdf_widget.SizedBox get horizontalPDF {
    return pdf_widget.SizedBox(
      width: this,
    );
  }
}
```

## Thousand Separator Input Formatter

```dart
import 'package:flutter/services.dart';

class ThousandsSeparatorInputFormatter extends TextInputFormatter {
  @override
  TextEditingValue formatEditUpdate(
    TextEditingValue oldValue,
    TextEditingValue newValue,
  ) {
    String text = newValue.text;

    // Only format if the input is not empty
    if (text.isNotEmpty) {
      // Remove any non-numeric except the dot (".") characters from the new value
      text = text.replaceAll(RegExp(r'[^\d\.]'), '');

      if (text.isNotEmpty) {
        text = _formatWithThousandsSeparator(text);
      }
    }

    // Return the new value with the formatted number and the new selection
    return TextEditingValue(
      text: text,
      selection: TextSelection.collapsed(offset: text.length),
    );
  }

  String _formatWithThousandsSeparator(String value) {
    // Divide the decimal places from the integer part
    List<String> parts = value.split(".");

    String integerPart = parts.first;
    List<String> integerParts = [];

    // Split the value into groups of 3 digits (starting from the right)
    for (int i = integerPart.length; i > 0; i -= 3) {
      int start = i - 3;
      if (start < 0) {
        start = 0;
      }
      String part = integerPart.substring(start, i);
      integerParts.add(part);
    }

    // Reverse the list of parts and join them with the separator
    String ret = integerParts.reversed.join(',');

    // If the original number is a float, add the decimal places
    if (parts.length > 1) {
      ret += ".${parts.last}";
    }

    // Return the new value
    return ret;
  }
}
```

Usage

```dart
TextField(
  keyboardType: TextInputType.number,
  inputFormatters: [
    ThousandsSeparatorInputFormatter()
  ],
),
```
