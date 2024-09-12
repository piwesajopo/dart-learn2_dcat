# A Dart Console App Example: dcat

This is the dcat app provided at https://dart.dev/tutorials/server/cmdline

Here I'll describe the process for creating a console app as described by the article. 
You can either follow the steps described in this README file, or you can clone this
repository and skip to the [Get Packages](#Get-Packages) section.

## Creating the app

The first step for creating this app was running the following command:
```shell
dart create dcat
```

The previous command will create the directory structure including a simple README.md with the following contents:

```markdown
A sample command-line application with an entrypoint in `bin/`, library code
in `lib/`, and example unit test in `test/`.
```

While this is the command provided on the article, a more specific command is:
```shell
# Explicitly specify the console app template
dart create -t console dcat
```

By specifying this option you make sure that this template is used 
(currently it is the default so you can omit it, but this is a way
to avoid problems if this behavior changes in 
the future.

Also, we can delete the following file which is not really used in the provided example:
```shell
rm lib/dcat.dart
```

## Modifying the entrypoint of the app

The next step in the tutorial is to add the code provided.
Modify the file located at `bin/dcat.dart`

I'll include it here for your convenience:
```dart
import 'dart:convert';
import 'dart:io';

import 'package:args/args.dart';

const lineNumber = 'line-number';

void main(List<String> arguments) {
  exitCode = 0; // Presume success
  final parser = ArgParser()..addFlag(lineNumber, negatable: false, abbr: 'n');

  ArgResults argResults = parser.parse(arguments);
  final paths = argResults.rest;

  dcat(paths, showLineNumbers: argResults[lineNumber] as bool);
}

Future<void> dcat(List<String> paths, {bool showLineNumbers = false}) async {
  if (paths.isEmpty) {
    // No files provided as arguments. Read from stdin and print each line.
    await stdin.pipe(stdout);
  } else {
    for (final path in paths) {
      var lineNumber = 1;
      final lines = utf8.decoder
          .bind(File(path).openRead())
          .transform(const LineSplitter());
      try {
        await for (final line in lines) {
          if (showLineNumbers) {
            stdout.write('${lineNumber++} ');
          }
          stdout.writeln(line);
        }
      } catch (_) {
        await _handleError(path);
      }
    }
  }
}

Future<void> _handleError(String path) async {
  if (await FileSystemEntity.isDirectory(path)) {
    stderr.writeln('error: $path is a directory');
  } else {
    exitCode = 2;
  }
}
```

## Get Packages {#Get-Packages}

In the tutorial they show you how to add the args package to your project by using this command:
```shell
dart pub add args
```

While using `dart pub add args` was more specific, it requires you to list all the packages you are using.

A better way to just let dart download what you need is running the following command:
```shell
dart pub get
```

## Running your program

Use the `dart run` command as follows:
```shell
dart run bin/dcat.dart [-n] somefile 
```

For example to dcat the file `pubspec.yaml`:
```shell
dart run bin/dcat.dart -n pubspec.yaml
```

Please note the -n command is optional. It will preface each line with a line number.


## Generating a binary

You can create a binary with the following command:

```shell
dart compile exe bin/dcat.dart -o dcat
```

The `-o dcat` is optional and is used on Linux/MacOs where you don't want the file to have a .exe extension.
You can omit it if you are using Windows, or even if you don't care about the filename being `dcat.exe`.
Also, please note that by default the file will be located at the same directory as the source file
if you don't specify the `-o path/filename` option.


## Additional information

Remember to visit https://dart.dev/tutorials/server/cmdline where the source code is explained in detail so you can learn more about the `async` and `await` language features, which rely on the **Future** and **Stream** classes for asynchronous support.
