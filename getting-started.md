# Test Runner Package

## Introduction

Test Runner Package is a simple solution to define and run tests for a PHP library. You can either add it as your project dependency or simply use `run` command to use it as a tool.

## Installation

### As a tool

There is no installation required when using `test-runner` as a tool.

> **Note**  
> When you use the `test-runner` as a tool, you still need writing your tests using the `PhpRepos\TestRunner\Runner\test` and assert using `PhpRepos\TestRunner\Assertions`.
> While doing so, your IDE may not be able to resolve their definition.
> There will not be any problem when running as `test-runner` load those files before running tests.
> There will soon be some extensions for each IDE to be able to resolve those definitions.

### As a dependency

You can simply install this package by running the following command:

```shell
phpkg add https://github.com/php-repos/test-runner.git
```

## Usage

By default, Test Runner looks up the `Tests` directory on your project directory to find tests.
Your test files should have a filename ending with `Test.php`. For example, `HomeTest.php` marks this file as a test.

### Define tests

In your files, you can simply define tests by calling the `test` function.
The signature of the `test` function is like so:

```php
use function PhpRepos\TestRunner\Runner\test;

function test(string $title, Closure $case, ?Closure $before = null, ?Closure $after = null, ?Closure $finally = null)
```

In the following, we are going to explain each parameter:

#### title

The `title` is your test's title that will be printed on the output while the Test Runner runs your test.

#### case

The `case` is a closure that should contain your main test functionality.

Example:

```php
use function PhpRepos\TestRunner\Assertions;
use function PhpRepos\TestRunner\Runner\test;

test(
    title: 'it should return true when the string starts with the given substring',
    case: function () {
        assert_true(str_starts_with('Hello World', 'Hello'));
        assert_false(str_starts_with('Hello World', 'World'));
    }
);
```

#### before

Sometimes you may need to do some stuff before starting your tests.
In this case, you can use the `before` parameter to define a closure that will be run before running your `case`.

```php
use function PhpRepos\TestRunner\Assertions;
use function PhpRepos\TestRunner\Runner\test;

test(
    title: 'it should assert directory exists',
    case: function () {
        assert_true(file_exists(__DIR__ . '/TestDirectory'));
    },
    before: function () {
        mkdir(__DIR__ . '/TestDirectory');
    }
);
```

It may be needed to pass some variables from the `before` to the `case`.
You can pass variables by returning them from the `before` closure:

```php
use function PhpRepos\TestRunner\Assertions;
use function PhpRepos\TestRunner\Runner\test;

test(
    title: 'it should assert directory exists',
    case: function ($directory) {
        assert_true(file_exists($directory));
    },
    before: function () {
        $directory = __DIR__ . '/TestDirectory' 
        mkdir($directory);
        
        return $directory
    }
);
```

If you need to pass more than one variable from the `before` to the `case`, you can return an array:

```php
use function PhpRepos\TestRunner\Assertions;
use function PhpRepos\TestRunner\Runner\test;

test(
    title: 'it should assert directory and file exist',
    case: function (Directory $directory, File $file) {
        assert_true(file_exists($directory));
        assert_true(file_exists($file));
    },
    before: function () {
        $directory = __DIR__ . '/TestDirectory' 
        mkdir($directory);
        $file = $directory->file('filename')->create('file content');
        
        return [$directory, $file];
    }
);
```

#### after

Just like the `before`, you may need to execute some functions after the test `case` finished successfully.
For example, you may need to clean up some files created during your test run.
In this case, you can use the `after` closure:

```php
use function PhpRepos\TestRunner\Assertions;
use function PhpRepos\TestRunner\Runner\test;

test(
    title: 'it should assert directory exists',
    case: function () {
        assert_true(file_exists(__DIR__ . '/TestDirectory'));
    },
    before: function () {
        mkdir(__DIR__ . '/TestDirectory');
    },
    after: function () {
        unlink(__DIR__ . '/TestDirectory');
    }
);
```

And of course, you can pass variables from the `case` to the `after` functions:

```php
use function PhpRepos\TestRunner\Assertions;
use function PhpRepos\TestRunner\Runner\test;

test(
    title: 'it should assert directory exists',
    case: function ($directory) {
        assert_true(file_exists($directory));
        
        return $directory;
    },
    before: function () {
        $directory = __DIR__ . '/TestDirectory' 
        mkdir($directory);
        
        return $directory
    },
    after: function ($directory) {
        unlink($directory);
    }
);
```

If the `after` hook needs inputs and the `case` does not return any value, outputs of the `before` hook will be passed to the `after` hook directly.

```php
use function PhpRepos\TestRunner\Assertions;
use function PhpRepos\TestRunner\Runner\test;

test(
    title: 'it should pass data from before to after',
    case: function () {
        assert_true(true);
    },
    before: function () {
        $directory = __DIR__ . '/TestDirectory' 
        mkdir($directory);
        
        return $directory
    },
    after: function ($directory) {
        unlink($directory);
    }
);
```

#### finally

In some tests, you may need to do some stuff even when a test fails.
For this case, you can use the `finally` command.
Test runner always runs this function after running the `case`:

```php
use function PhpRepos\TestRunner\Assertions;
use function PhpRepos\TestRunner\Runner\test;

test(
    title: 'it should assert directory not exists',
    case: function () {
        assert_false(file_exists(__DIR__ . '/TestDirectory'));
    },
    before: function () {
        mkdir(__DIR__ . '/TestDirectory');
    },
    finally: function () {
        unlink(__DIR__ . '/TestDirectory');
    }
);
```

> **Note**  
> You can not pass any parameter to the `finally` closure.

### Build and Run

#### As a tool

There is no extra step required while using `test-runner` as a tool because `phpkg` will take care of everything. However, you need to `build` your project before running tests. Run:

```bash
phpkg build
cd builds/development
phpkg run https://github.com/php-repos/test-runner.git
```

#### As a dependency

When you build your project, your tests also will get built.
As you can see in the `phpkg.config.json` file, there is an `executable` section in this file.
This section will get used to make a link file named `test-runner` to the Test Runner's `run` file.
Therefore, you can run your tests after the build by running:

```shell
phpkg build
cd build/development
./test-runner
```

The `test-runner` file is an executable file so there is no need to call `php` before it.
