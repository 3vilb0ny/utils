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