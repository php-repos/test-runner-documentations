# Test Runner Package

## Run tests

### As a tool

There is no extra step required while using `test-runner` as a tool because `phpkg` will take care of everything. However, you need to `build` your project before running tests. Run:

```bash
phpkg build
cd builds/development
phpkg run https://github.com/php-repos/test-runner.git
```

### As a dependency

When you build your project, your tests also will get built.
As you can see in the `phpkg.config.json` file, there is an `executable` section in this file.
This section will get used to make a link file named `test-runner` to the Test Runner's `run` file.
Therefore, you can run your tests after the build by running:

```shell
phpkg build
cd build/development
./test-runner
```

### Filtering tests

You may need to filter the test file that you want to run. 
In this case, you can use the `filter` argument for filtering the test file or their path that you want to run.

#### As a dependency
```shell
phpkg run https://github.com/php-repos/test-runner.git run --filter=SampleFileTest
phpkg run https://github.com/php-repos/test-runner.git run --filter=Core
```

#### As a dependency
```shell
./test-runner --filter=SampleFileTest
./test-runner --filter=Core
```
