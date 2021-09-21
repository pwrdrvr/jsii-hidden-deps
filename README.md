# Overview

This repository demonstrates a possible issue / design limitation in [jsii](https://github.com/aws/jsii/) (used for building multi-language CDK modules in TypeScript / JavaScript / C# / DotNet / Python / Java) in that private modules must provide support for all target languages of the consuming module, else the private module must be listed as a `bundledDependencies`, causing it and all of it's dependencies to be bundled with the new module being created.

`jsii` appears to be using inter-process communication to marshall messages to `node` to execute the module code and the generated stubs are just that: stubs that allow the marshalling to be strongly typed from non-node languages.  As such, it seems that a `jsii` module should be able to privately use `jsii` modules with partial or missing language stub support so long as no types from that module are exposed in the public api of the new module.  If this is possible then the private module should no longer need to be listed in `bundledDependencies` and can be installed at runtime instead.

This seems to be related to, but slightly different than, a similar issue for private modules needing to be `peerDependencies` even if they are not exposed in the public API: [jsii issue #2942](https://github.com/aws/jsii/issues/2942).

# Before Adding JS/Python-only Dependency

- `npm run build`
- `npm run package`
- Result: `dotnet`, `java`, `js`, and `python` packages are created in `dist` without error

# Add JS/Python-only Dependency

- This dependency is only to be consumed by the library being created and will not be exposed to consumers of the new library's public API.
- Adding this dependency immediately causes `package` to fail.
- CDK-Deletable-Bucket only lists Python in it's JSII Targets: https://github.com/cloudcomponents/cdk-constructs/blob/375ecd9eff5557636401f848d2b7685389e727a2/packages/cdk-deletable-bucket/package.json#L43-L59
- `npm i @cloudcomponents/cdk-deletable-bucket`
- `npm run build`
- `npm run package`
- Result: `TypeError: Cannot read property 'namespace' of undefined at DotNetTypeResolver.resolveNamespacesDependencies (/jsii-hidden-deps/node_modules/jsii-pacmak/lib/targets/dotnet/dotnettyperesolver.js:71:46)`
- [Line of failure](https://github.com/aws/jsii/blob/00c583586d832d77c372ea0d7ddb62b85a9ca336/packages/jsii-pacmak/lib/targets/dotnet/dotnettyperesolver.ts#L88)


## Complete Error

```
> jsii-hidden-deps@1.0.0 package /Users/huntharo/pwrdrvr/jsii-hidden-deps
> jsii-pacmak

[jsii-pacmak] [WARN] Exception occurred, not cleaning up 
[jsii-pacmak] [WARN] dotnet failed
[jsii-pacmak] [WARN] Exception occurred, not cleaning up 
[jsii-pacmak] [WARN] java failed
TypeError: Cannot read property 'namespace' of undefined
    at DotNetTypeResolver.resolveNamespacesDependencies (/Users/huntharo/pwrdrvr/jsii-hidden-deps/node_modules/jsii-pacmak/lib/targets/dotnet/dotnettyperesolver.js:71:46)
    at DotNetGenerator.generate (/Users/huntharo/pwrdrvr/jsii-hidden-deps/node_modules/jsii-pacmak/lib/targets/dotnet/dotnetgenerator.js:43:27)
    at Dotnet.generateCode (/Users/huntharo/pwrdrvr/jsii-hidden-deps/node_modules/jsii-pacmak/lib/target.js:29:28)
    at async DotnetBuilder.generateModuleCode (/Users/huntharo/pwrdrvr/jsii-hidden-deps/node_modules/jsii-pacmak/lib/targets/dotnet.js:94:9)
    at async /Users/huntharo/pwrdrvr/jsii-hidden-deps/node_modules/jsii-pacmak/lib/targets/dotnet.js:62:30
    at async Function.make (/Users/huntharo/pwrdrvr/jsii-hidden-deps/node_modules/jsii-pacmak/lib/util.js:186:36)
    at async DotnetBuilder.buildModules (/Users/huntharo/pwrdrvr/jsii-hidden-deps/node_modules/jsii-pacmak/lib/targets/dotnet.js:35:35)
    at async Promise.all (index 0)
    at async Object.pacmak (/Users/huntharo/pwrdrvr/jsii-hidden-deps/node_modules/jsii-pacmak/lib/index.js:56:9)
npm ERR! code ELIFECYCLE
npm ERR! errno 1
npm ERR! jsii-hidden-deps@1.0.0 package: `jsii-pacmak`
npm ERR! Exit status 1
npm ERR! 
npm ERR! Failed at the jsii-hidden-deps@1.0.0 package script.
npm ERR! This is probably not a problem with npm. There is likely additional logging output above.

npm ERR! A complete log of this run can be found in:
npm ERR!     /Users/huntharo/.npm/_logs/2021-09-21T00_55_23_423Z-debug.log
```

# Work-Around - Adding to bundledDepdencies

- Remove `@cloudcomponents/cdk-deletable-bucket` from `peerDepdencies` in `package.json`
- Add `"bundledDependencies": ["@cloudcomponents/cdk-deletable-bucket"]` to `package.json`
- `npm run build`
- `npm run package`
- Result: These both work now, but the generated packages now contain not only `@cloudcomponents/cdk-deletable-bucket`, but all of it's dependencies as well
- Expected / Hoped For: A private, non-exposed in the public API, dependency should be able to lack support in a particular target JSII language and it's dependencies should be able to be installed at runtime, just as if `@aws-cdk/aws-s3` had been listed as a dependency/peerDenendency (in which case `dist/js/package` contains no `node_modules` directory and deps are installed by the consumer).