# Memory leaks in Jest with Faker

This repo is a fork of: https://github.com/CodelyTV/javascript-testing-frontend-course

I removed all the directories of the original repo to focus only at 72-test-object-factory.

To get all the context about what you will see here, you should check the course: https://pro.codely.tv/library/testing-frontend/196940/about/

If you don't know about CodelyTv, I would recommend you to check their courses: https://codely.tv/

## Summary

At this repo I want to show how Jest is leaking memory in the example project cause of faker.js (and maybe other libraries) and how to solve it.

I duplicated the tests files to do the memory leak more evident.

You just need to setup the project and execute the different commands to see how memory leak is happening.

In summary:

- To check memory leak we are using `--logHeapUsage` and `no-cache` Jest flags to see how heap usage increases each test execution.
- We solve the memory leak issue executing jest as a node script using `--expose-gc` node flag and `--logHeapUsage` jest flag. This way, jest will execute `global.gc()` after each `runTest`: https://github.com/facebook/jest/blob/ab014c140af2e10ae7c5790c2406009790787cdb/packages/jest-runner/src/runTest.ts#L318

## Project setup

```
npm install
```

## Run your unit tests

In this project we are executing test from vue-cli-service but it would not change if you execute it from jest directly.

### npm run test

This command is just executing tests with no extra info.

```
npm run test

## it is executing
vue-cli-service test:unit
```

### npm run test:leaks

This command has no workers limit.

It will show the worker heap usage on each test suite.

Each worker will increase his memory usage independently.

So, with this command, memory leak can be less evident.

```
npm run test:leaks

## it is executing:
vue-cli-service test:unit --no-cache --logHeapUsage
```

### npm run test:ci:leaks

This command is what you should execute in a CI environment or in a virtual machine where CPU could be limited.

This command use --runInBand jest flag to limit jest to one worker.

The memory leak is more evident

```
npm run test:ci:leaks

## it is executing:
vue-cli-service test:unit --runInBand --no-cache --logHeapUsage
```

### npm run test:leaks:solved

When you execute a node script with node flag `--expose-gc`, you can execute garbage collector from a global method `gc` like `global.gc()`.

This command executes jest as a node script with flag `--expose-gc` and jest flags `--logHeapUsage` and `--no-cache`.

You will see how memory leaks watched in command `test:leaks` has been solved.

```
npm run test:leaks:solved

## it is executing:
node --expose-gc ./node_modules/@vue/cli-service/bin/vue-cli-service.js test:unit --no-cache --logHeapUsage
```

### npm run test:leaks:solved

This command executes jest as a node script with flag `--expose-gc` and jest flags `--runInBand`, `--logHeapUsage` and `--no-cache`.

You will see how memory leaks watched in command `test:ci:leaks` has been solved.

```
npm run test:ci:leaks:solved

## it is executing:
node --expose-gc ./node_modules/@vue/cli-service/bin/vue-cli-service.js test:unit --runInBand --no-cache --logHeapUsage
```
