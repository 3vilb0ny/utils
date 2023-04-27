## Format String with variables

```dart
extension StringModified on String {

  /// Multiline String to online String
  String toOneLine() {
    return replaceAll(RegExp(r'[\s]+'), " ").trim();
  }

  String format(List<String> params) => _interpolate(this, params);

  /// Interpolates the string with given parameters
  /// The substitution sequence is %n\$, with n being the position in the list
  /// 1 based
  String _interpolate(String string, List<String> params) {
    String result = string;
    for (int i = 1; i < params.length + 1; i++) {
      result = result.replaceAll('%$i\$', params[i - 1]);
    }

    return result;
  }

  String format(List<String> params) => _interpolate(this, params);

  /// Interpolates the string with given parameters
  /// The substitution sequence is %n\$, with n being the position in the list
  /// 1 based
  String _interpolate(String string, List<String> params) {
    String result = string;
    for (int i = 1; i < params.length + 1; i++) {
      result = result.replaceAll('%$i\$', params[i - 1]);
    }

    return result;
  }
}

/// Tests
test("String format", () {
  const String text = 'Today is %1\$ and tomorrow is %2\$';
  final List<String> placeHolders = ['Monday', 'Tuesday'];
  const String expected = 'Today is Monday and tomorrow is Tuesday';

  final String actual = text.format(placeHolders);

  expect(actual, expected);
});
```

## Iterable extension

```dart
extension Iterables<E> on Iterable<E> {
  /// Groups an iterable by an Object property
  Map<K, List<E>> groupBy<K>(K Function(E element) keyFunction) => fold(
      <K, List<E>>{},
      (Map<K, List<E>> map, E element) =>
          map..putIfAbsent(keyFunction(element), () => <E>[]).add(element));

  /// Returns the index of given element founded first
  int indexOf(E element) {
    for (int i = 0; i < length; i++) {
      if (elementAt(i).toString() == element.toString()) return i;
    }
    return -1;
  }

  /// Normal map function with the element index as an argument too
  Iterable<T> mapWithIndex<T>(T Function(E e, int i) toElement) sync* {
    int index = 0;
    for (var value in this) {
      yield toElement(value, index++);
    }
  }

  /// Split an iterable into multi iterables based on a test function
  Iterable<Iterable<E>> toChunks(bool Function(E element) test) sync* {
    if (length <= 0) {
      yield [];
      return;
    }
    final chunk = <E>[];
    for (E item in this) {
      chunk.add(item);
      if (test(item)) {
        yield chunk;
        chunk.clear();
      }
    }
    if (chunk.isNotEmpty) yield chunk;
  }

  /// Insterts an element between every element in the Iterable
  Iterable<E> insertBetween<T>(E element) sync* {
    final int length = this.length;

    if (length == 0) {
      return;
    }

    Iterator<E> iterator = this.iterator;
    iterator.moveNext();

    for (int i = 0; i < length - 1; i++) {
      yield iterator.current;
      yield element;
      iterator.moveNext();
    }

    yield iterator.current;
  }

  /// Converts an Iterable<MapEntry<K, V>> to Map<K, V>
  Map<K, V> toMap<K, V>() {
		if (this is! Iterable<MapEntry<K, V>>) {
      throw TypeError();
    }
    return Map.fromEntries(this as Iterable<MapEntry<K, V>>);
  }

  /// Combine an iterable into an iterable of List of desired length
  ///
  /// [1, 2, 3].combinations(2) ==> [[1, 2], [1, 3], [2, 3]]
  Iterable<List<E>> combinations([int n = 2]) sync* {
    if (n == 0) {
      yield [];
    } else {
      int i = 0;
      for (E element in this) {
        if (i + n > length) {
          return;
        }
        if (n == 1) {
          yield [element];
        } else {
          for (List<E> combination in skip(i + 1).combinations(n - 1)) {
            yield [element, ...combination];
          }
        }
        i++;
      }
    }
  }

  /// Largest absolute value of the difference of each element, divided by the
  /// length
  ///
  /// Each value of the iterable must be [num] type
  double weightedAverage() {
    num ret = 0;
    if (length < 2) {
      return ret / mean();
    }

    ret = combinations(2)
        .map((pair) => ((pair[0] as num) - (pair[1] as num)).abs())
        .findMax();
    ret = ret / mean();

    return ret as double;
  }

  /// Mean ("average") for an iterable of any data type.
  ///
  /// The mean is calculated as the sum of all the elements divided by the
  /// number of elements.
  ///
  /// Each value of the iterable must be [num] type
  double mean() {
    if (length == 0) {
      return 0.0;
    }
    double sum = 0;
    for (E element in this) {
      sum += (element as num).toDouble();
    }
    return sum / length;
  }

  /// Find the maximum value in an iterable of any data type.
  ///
  /// Each value of the iterable must be [num] type
  E findMax() {
    if (isEmpty) {
      throw StateError('Cannot find maximum value in an empty iterable');
    }
    E max = first;
    for (E element in skip(1)) {
      if ((element as num) > (max as num)) {
        max = element;
      }
    }
    return max;
  }
}

/// Tests
test("groupBy", () {
  List<MapEntry<int, int>> x = [
    const MapEntry<int, int>(1, 1),
    const MapEntry<int, int>(1, 2),
    const MapEntry<int, int>(1, 3),
    const MapEntry<int, int>(2, 4),
    const MapEntry<int, int>(2, 5),
    const MapEntry<int, int>(2, 6),
  ];

	expect(
    x.groupBy((MapEntry<int, int> element) => element.key),
    {
      1: [const MapEntry(1, 1), const MapEntry(1, 2), const MapEntry(1, 3)],
      2: [const MapEntry(2, 4), const MapEntry(2, 5), const MapEntry(2, 6)]
    },
	);

	expect([].groupBy((element) => element), {});
});

test("indexOf", () {
	expect([].indexOf(0), -1);
	expect([1, 2, 3].indexOf(2), 1);
	expect([1, 2, 2].indexOf(2), 1);
});

test("mapWithIndex", () {
  expect([].mapWithIndex((e, i) => e * i), []);
  expect([1, 2, 3].mapWithIndex((e, i) => e * i), [0, 2, 6]);
});

test("toChunks", () {
  List<int> xw = [1, 2, 3, 4, 5];
  Iterable<Iterable<int>> rw = xw.toChunks((int element) {
    return element == 2;
  });

  List<int> xy = [1, 2, 3, 2, 5];
  Iterable<Iterable<int>> ry = xy.toChunks((int element) {
    return element == 2;
  });

  List<int> xz = [1, 2, 2, 4, 5];
  Iterable<Iterable<int>> rz = xz.toChunks((int element) {
    return element == 2;
  });

  List<int> xv = [1, 2, 2, 4, 2];
  Iterable<Iterable<int>> rv = xv.toChunks((int element) {
    return element == 2;
  });

  expect([].toChunks((element) => true), [[]]);

  expect(rw, [
    [1, 2],
    [3, 4, 5]
  ]);

  expect(ry, [
    [1, 2],
    [3, 2],
    [5]
  ]);

  expect(rz, [
    [1, 2],
    [2],
    [4, 5]
  ]);

  expect(rv, [
    [1, 2],
    [2],
    [4, 2],
  ]);
});

test('toMap', () {
  Iterable<MapEntry<String, int>> input = [
    const MapEntry('a', 1),
    const MapEntry('b', 2),
    const MapEntry('c', 3)
  ];
  Map<String, int> output = input.toMap();
  expect(output, {'a': 1, 'b': 2, 'c': 3});
});

test('toMap error', () {
  Iterable<String> input = ['a', 'b', 'c'];
  expect(() => input.toMap(), throwsA(isA<TypeError>()));
});

test("Combinations", () {
  final values = [1, 2, 3];
  expect(values.combinations(2), [
    [1, 2],
    [1, 3],
    [2, 3]
  ]);
});

test("Mean (Average)", () {
  final values = [1.0, 2.0, 3.0, 4.0, 5];
  expect(values.mean(), 3.0);
});

test("Max", () {
  final values = [0, -4, 6, 2];
  expect(values.findMax(), 6);
});

(test("Weighted Average", () {
  final values = [1, 2, 3];
  expect(values.weightedAverage(), 1.0);
}));
```

## Double extension

```dart
extension DoubleModified on double {
  /// This is a way to cut the decimal places of a double,
  /// but still being as double
  double toFixed({int round = 2}) {
    return double.parse(toStringAsFixed(round));
  }
}

/// Tests
test('toFixed', () {
  expect(3.14159.toFixed(2), 3.14);
  expect(1.23456.toFixed(3), 1.235);
  expect(0.123456.toFixed(4), 0.1235);
  expect(123.45.toFixed(0), 123.0);
});
```

## DateTime formatters

```dart
/// dd/mm/YYYY
String dateToString(DateTime date) {
  return "${date.day.toString().padLeft(2, '0')}/${date.month.toString().padLeft(2, '0')}/${date.year.toString()}";
}

/// hh:MM
String timeToString(DateTime date) {
  return "${date.hour.toString().padLeft(2, '0')}:${date.minute.toString().padLeft(2, '0')}";
}

/// dd/mm/YYYY hh:MM
String dateTimeToString(DateTime date) {
  return "${dateToString(date)} ${timeToString(date)}";
}

```
