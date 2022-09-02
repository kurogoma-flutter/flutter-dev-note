## Flutter ログ出力

### packageインポート
```
$ flutter pub add logger
$ flutter pub add intl
```

```dart
import 'package:logger/logger.dart';
import 'package:intl/intl.dart';

final logger = Logger(
  printer: CustomLogPrinter(),
  output: CustomLogOutput(),
);

/// シンプルな出力
class CustomLogPrinter extends LogPrinter {
  @override
  List<String> log(LogEvent event) {
    final message = event.message;

    String msg;
    if (message is Function()) {
      msg = message().toString();
    } else if (message is String) {
      msg = message;
    } else {
      msg = message.toString();
    }
    return [
      '[${event.level.name.toUpperCase()}] '
          '${DateFormat('HH:mm:ss.SSS').format(DateTime.now())}: '
          '$msg'
    ];
  }
}

class CustomLogOutput extends ConsoleOutput {
  @override
  void output(OutputEvent event) {
    super.output(event);
    if (event.level.index >= Level.error.index) {
      throw AssertionError('StackTraceを参照してください。');
    }
  }
}
```