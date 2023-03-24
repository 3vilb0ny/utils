## Multiline String to one line String

```dart
/// Multiline String to online String
String toOneLine() {
	return replaceAll(RegExp(r'[\s]+'), " ").trim();
}

/// Tests
test("String toOneLine", () {
    const String text = """This
	is a
	text""";
	const String expected = "This is a text";

	final String actual = text.toOneLine();
	final String actual2 = expected.toOneLine();

	expect(actual, expected);
	expect(actual2, expected);
});
```

## Format String with variables

```dart
extension StringModified on String {

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
extension IterableCustom<E> on Iterable<E> {
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
```

## Double extension

```dart
extension DoubleModified on double {
  /// This is a way to cut the decimal places of a double,
  /// but still being as double
  double toPrecision({int round = 2}) {
    return double.parse(toStringAsFixed(round));
  }
}
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
