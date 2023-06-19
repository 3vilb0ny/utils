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
  /// Groups an [Iterable] by an Object property.
  Map<K, List<E>> groupBy<K>(K Function(E element) keyFunction) => fold(
      <K, List<E>>{},
      (Map<K, List<E>> map, E element) =>
          map..putIfAbsent(keyFunction(element), () => <E>[]).add(element));

  /// Returns the index of given element founded first.
  int indexOf(E element) {
    for (int i = 0; i < length; i++) {
      if (elementAt(i).toString() == element.toString()) return i;
    }
    return -1;
  }

  /// Normal map function with the element index as an argument too.
  Iterable<T> mapWithIndex<T>(T Function(E e, int i) toElement) sync* {
    int index = 0;
    for (var value in this) {
      yield toElement(value, index++);
    }
  }

  /// Split an [Iterable] into multi [Iterable]s based on a test function.
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

  /// Insterts an element between every element in the Iterable.
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

  /// Converts an Iterable<MapEntry<K, V>> to Map<K, V>.
  Map<K, V> toMap<K, V>() {
    if (this is! Iterable<MapEntry<K, V>>) {
      throw TypeError();
    }
    return Map.fromEntries(this as Iterable<MapEntry<K, V>>);
  }

  /// Combine an [Iterable] into an [Iterable] of List of desired length.
  ///
  /// [1, 2, 3].combinations(2) ==> [[1, 2], [1, 3], [2, 3]].
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
  /// length.
  ///
  /// Each value of the [Iterable] must be [num] type.
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

  /// Mean ("average") for an [Iterable] of any data type.
  ///
  /// The mean is calculated as the sum of all the elements divided by the
  /// number of elements.
  ///
  /// Each value of the [Iterable] must be [num] type
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

  /// Find the maximum value in an [Iterable] of any data type.
  ///
  /// Each value of the [Iterable] must be [num] type.
  E findMax() {
    if (isEmpty) {
      throw StateError('Cannot find maximum value in an empty [Iterable]');
    }
    E max = first;
    for (E element in skip(1)) {
      if ((element as num) > (max as num)) {
        max = element;
      }
    }
    return max;
  }

  /// Applies an operation between the elements of one [Iterable] and the elements
  /// of the [other] [Iterable].
  ///
  /// [1, 2, 3].operateWith([1, 2, 3], (a, b) => a + b) ==> [2, 4, 6]
  Iterable<R?> operateWith<R>(
      Iterable<E?> other, R Function(E, E) operation) sync* {
    final iterator1 = iterator;
    final iterator2 = other.iterator;

    while (iterator1.moveNext() && iterator2.moveNext()) {
      final current1 = iterator1.current;
      final current2 = iterator2.current;

      if (current1 == null || current2 == null) {
        yield null;
        continue;
      }
      yield operation(current1, current2);
    }
  }

  /// Performs an operation in the same index of the elements in a
  /// bidimensional [Iterable].
  ///
  /// [[1, 2, 3], [4, 5, 6]].calulateInColumn((a, b) => a + b, 3)
  /// ==> [5, 7, 9]
  Iterable<num> calculateInColumn(num Function(num, num) operation) sync* {
    if (isEmpty) {
      yield* [];
      return;
    }
    List<num> result =
        ((this as Iterable<Iterable<num>>).elementAt(0)).toList();

    for (E e in skip(1)) {
      result = result
          .operateWith(e as Iterable<num>, operation)
          .cast<num>()
          .toList();
    }
    yield* result;
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

group('calculateInColumn', () {
    test('should calculate sum in each column', () {
      final data = [
        [1, 2, 3],
        [4, 5, 6],
        [7, 8, 9],
      ];

      final columnSums = data.calculateInColumn((a, b) => a + b).toList();

      expect(columnSums, equals([12, 15, 18]));
    });

    test('should calculate product in each column', () {
      final data = [
        [1, 2, 3],
        [4, 5, 6],
        [7, 8, 9],
      ];

      final columnProducts = data.calculateInColumn((a, b) => a * b).toList();

      expect(columnProducts, equals([28, 80, 162]));
    });

    test('should return zeros if data is empty', () {
      final emptyData = <List<num>>[];

      final columnSums = emptyData.calculateInColumn((a, b) => a + b).toList();

      expect(columnSums, equals([]));
    });
  });

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

## File Service & File cache service

1. File Service

   ```dart
   import 'dart:io';

   import 'package:flutter_dotenv/flutter_dotenv.dart';
   import 'package:followit/falvors.dart';
   import 'package:path_provider/path_provider.dart';

   abstract class FileService {
     static Future<String> get localPath async {
       final directory = await getApplicationDocumentsDirectory();
       return directory.path;
     }

     static Future<Directory> get tempDirectory async {
       return await getTemporaryDirectory();
     }

     static Future<String> get tempPath async {
       return (await tempDirectory).path;
     }

     static Future<File> get localStorageActionsFile async {
       final path = await localPath;
       // Environment variables
       if (!dotenv.isInitialized) {
         await dotenv.load(fileName: ".env");
       }

       String baseFile = dotenv.env["LOCAL_STORAGE_ACTIONS"]!;

       File file = File(
           '$path/${baseFile}_${FlavorsE.instance.getFlavor().name}.json');
       if (!await file.exists()) {
         file.create();
       }
       return file;
     }

     static Future<void> writeLocalStorageActionsFile(String s) async {
       await (await FileService.localStorageActionsFile)
           .writeAsString(s, mode: FileMode.write);
     }
   }
   ```

2. File Cache Service

   ```dart
   import 'dart:io';

   import 'package:flutter/services.dart';
   import 'package:followit/src/services/file_service.dart';

   class FileCacheService {
     static FileCacheService? _instance;
     final String tempPath;

     FileCacheService({required this.tempPath});

     static Future<FileCacheService> getInstance() async {
       _instance ??= FileCacheService(tempPath: await FileService.tempPath);
       return _instance!;
     }

     String _storePath(String url) {
       return url
           .trim()
           .toLowerCase()
           .replaceAll('/', '_')
           .replaceAll('.', '_')
           .replaceAll(':', '_');
     }

     Future<Uint8List?> getFileBytes(String url) async {
       String storeUrl = _storePath(url);

       String path = "$tempPath/$storeUrl";
       if (File(path).existsSync()) {
         return File(path).readAsBytesSync();
       }
       return null;
     }

     void storeFileBytes(String url, Uint8List bytes) {
       String storeUrl = _storePath(url);

       String path = "$tempPath/$storeUrl";

       File(path).writeAsBytes(bytes);
     }
   }
   ```

3. Usage [Ex: HTTPService]

   ```dart
   /// Endpoint request
   String path = "${dotenv.env['API_URL']}/$endpoint";
   FileCacheService fcs = await FileCacheService.getInstance();
   Uint8List? fb = await fcs.getFileBytes(path);
   if (fb != null) {
     return fb;
   }
   /// Make the request

   /// Store into FileCacheService
   if (response.statusCode == 200) {
     fb = response.bodyBytes;
     fcs.storeFileBytes(path, fb);
     return fb;
   }
   ```

## Isolate Task With Arguments & Environment Variables

```dart
import 'dart:async';
import 'dart:isolate';

import 'package:aico/services/env.dart';

class IsolateData {
  final SendPort sendPort;
  final Map<String, String> env;
  final List<dynamic>? args;
  final Function task;

  const IsolateData({
    required this.sendPort,
    required this.env,
    required this.args,
    required this.task,
  });
}

Future<T> runTaskIsolated<T>(
  Function(List<dynamic>? args) task, {
    List<dynamic>? args,
}) async {
  Completer<T> completer = Completer<T>();

  // Communication channel for success response
  ReceivePort isolateToMainStream = ReceivePort();

  // Communication channel for error response
  ReceivePort isolateToMainStreamError = ReceivePort();

  // Data to send to isolate
  IsolateData auxD = IsolateData(
    sendPort: isolateToMainStream.sendPort,
    env: Env.data,
    args: args,
    task: task,
  );

  // Spawn Task
  await Isolate.spawn((IsolateData data) {
    Env(data.env);
    data.sendPort.send(data.task(data.args));
  }, auxD, onError: isolateToMainStreamError.sendPort);

  // End future
  isolateToMainStream.listen((message) {
    completer.complete(message);
  });

  // Handle errors
  isolateToMainStreamError.listen((e) {
    List errors = e as List;
    completer.completeError(errors.first);
  });

  return completer.future;
}

```

**Env Class**

```dart
class Env {
  Env._privateConstructor(Map<String, String> data) {
    _data = data;
  }

  static late final Env _instance;
  static late final Map<String, String>? _data;

  factory Env(Map<String, String> data) {
    _instance = Env._privateConstructor(data);
    return _instance;
  }

  static Map<String, String> get data => _data ?? {};
}

```

**Usage**

```dart
final result = await runTaskIsolated<ReturnedDataType>((args) => someTask(args), args: [...]);
```
