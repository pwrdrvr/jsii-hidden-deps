# Before Adding JS/Python-only Dependency

- `npm run build`
- `npm run package`
- Result: `dotnet`, `java`, `js`, and `python` packages are created in `dist` without error

# Add JS/Python-only Dependency

- Note: this dependency is only to be consumed by the library being created and will not be exposed to consumers of the new library.
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