<!-- TOC -->

- [alain 打包发布组件，ng-zero也可以用](#alain-打包发布组件ng-zero也可以用)
    - [安装依赖](#安装依赖)
    - [目录结构](#目录结构)
        - [文件具体内容](#文件具体内容)
            - [`ng-package.json`](#ng-packagejson)
            - [`package.json`](#packagejson)
            - [`public-api.ts`](#public-apits)
        - [要发布的组件](#要发布的组件)
    - [配置angular.json](#配置angularjson)
    - [测试模块](#测试模块)

<!-- /TOC -->

# alain 打包发布组件，ng-zero也可以用
> ######  本文参考了 https://angularfirebase.com/lessons/build-an-angular-library-with-ngpackagr/, 以及alain的的package的组件结构

## 安装依赖

打包需要安装一个angular的插件`ng-packagr`

```shell
npm install ng-packagr -save-dev
```

## 目录结构

目录结构就是没有大改动，为了方便调试自己的组件，和alain类似，src下的目录结构，如果想发布需要添加几个文件


```
│  browserslist
│  favicon.ico
│  hmr.ts
│  index.html
│  karma.conf.js
│  main.ts
│  ng-package.json     --新加
│  package.json        --新加
│  polyfills.ts
│  public-api.ts       --新加
│  styles.css
│  styles.less
│  test.ts
│  tsconfig.app.json
│  tsconfig.lib.json   --新加
│  tsconfig.spec.json
│  tslint.json
│  typings.d.ts
│
├─app
├─assets
├─environments
├─styles
└─testing

```

### 文件具体内容

- `ng-package.json` 打包用的配置
- `package.json`    发布包的package.json
- `public-api.ts`   用来导出的文件
- `tsconfig.lib.json`   ts编译配置

####  `ng-package.json`
```json
{
  "$schema": "../node_modules/ng-packagr/ng-package.schema.json",
  "dest": "../dist",
  "deleteDestPath": true,
  "lib": {
    "entryFile": "public-api.ts"
  }
}
```

#### `package.json`
```json
{
  "name": "@zhang/component",
  "version": "1.0.1",
  "peerDependencies": {
    "@angular/common": "^6.0.0-rc.0 || ^6.0.0",
    "@angular/core": "^6.0.0-rc.0 || ^6.0.0",
    "@delon/abc": "^1.1.3",
    "@delon/acl": "^1.1.3",
    "@delon/auth": "^1.1.3",
    "@delon/cache": "^1.1.3",
    "@delon/form": "^1.1.3",
    "@delon/mock": "^1.1.3",
    "@delon/theme": "^1.1.3",
    "@delon/util": "^1.1.3",
    "ng-zorro-antd": "^1.2.0",
    "ng-alain": "^1.1.3"
  }
}

```
>  所有需要的依赖写写
  
####  `public-api.ts`

```javascript
export * from './app/package/zhang-component.module';
```
>  导出主module

`src/app`目录下添加一个需要发布模块，下面需要有一个主的module文件

```
├─core
├─layout
├─package -新加
├─routes
└─shared
```
我这里叫`zhang-component.module.ts`
具体内容

```javascript
import { ModuleWithProviders, NgModule } from '@angular/core';

import { CommonModule } from "@angular/common";

import { TestCcModule } from './test-cc/test-cc.module';
import { DataChartModule } from './data-chart/data-chart.module';
import { DataCardModule } from './data-card/data-card.module';
import { ModelTreeModule } from './model-tree/model-tree.module';
/*这里就是我们新建的module*/
import {DemoModuleModule} from './demo-module/demo-module.module'

export * from './data-chart';
export * from './data-card';
export * from './test-cc';
export * from './model-tree';
/*同样将目录导出，会默认找目录下的index.ts*/
export * from './demo-module/';

const MYMODULES = [
  TestCcModule,
  DataChartModule,
  DataCardModule,
  ModelTreeModule,
  /*将Module加入到数组内*/
  DemoModuleModule
];

@NgModule({
  imports: [
    CommonModule,
    TestCcModule.forRoot(),
    DataChartModule.forRoot(),
    DataCardModule.forRoot(),
    ModelTreeModule.forRoot(),
    /*这里很重要,作为共享模块需要forRoot*/
    DemoModuleModule.forRoot()
  ],
  exports: [
    MYMODULES
  ]
})
export class ZhangComponentRootModule {

}
@NgModule({ exports: MYMODULES })
export class ZhangComponentModule {
  static forRoot(): ModuleWithProviders {
    return { ngModule: ZhangComponentRootModule };
  }
}
```

### 要发布的组件

在package目录下正常用ng命令创建module就可以

然后稍加修改

```
│  zhang-component.module.ts
│
├─demo-module  --ng g m 生成
│  │  demo-module.module.ts
│  │  index.ts   --手动添加index.ts 
│  │
│  └─demo-component
│          demo-component.component.html
│          demo-component.component.less
│          demo-component.component.ts

```

`index.ts`内容

```javascript
export * from './demo-module.module';
export * from './demo-component/demo-component.component';
```

> 将需要文件导出，上面会自动找到`index.ts`文件

`demo-module.module`文件修改,要注意组件用到的依赖包都写在这个文件里


```javascript
@NgModule({
  imports: [
    CommonModule
  ],
  declarations: [DemoComponentComponent],
  exports:[DemoComponentComponent] //将Component导出
})
export class DemoModuleModule {
  //设置为共享模块
  static forRoot(): ModuleWithProviders {
    return { ngModule: DemoModuleModule, providers: [] };
  }
 }
```

默认的`ModuleWithProviders`是没有导入的，需要在头部加入

```javascript
import { NgModule } from '@angular/core';
```
改成

```javascript
import { NgModule,ModuleWithProviders } from '@angular/core';
```

## 配置angular.json

上面都完成了，需要修改跟目录下`angular.json`来打包

在`projects`节点下加一个自己的配置

```json
"zhang-ui": {   //这里写啥都行，就是个命令的名字
      "root": "src",
      "sourceRoot": "src/app",
      "projectType": "library",
      "prefix": "lib",
      "architect": {
        "build": {
          "builder": "@angular-devkit/build-ng-packagr:build",
          "options": {
            "tsConfig": "src/tsconfig.lib.json", //读取的编译文件
            "project": "src/ng-package.json" //读取的配置
          },
          "configurations": {
            "production": {
              "project": "src/ng-package.json"
            }
          }
        }
      }
    }
```

最后在根目录下的`package.json`里面添加一个命令

```json
"scripts": {
    "ng": "ng",
    "start": "ng serve -o --port 4444",
    "build": "ng build --prod --build-optimizer",
    "test": "ng test",
    "lint": "npm run lint:ts && npm run lint:style",
    "e2e": "ng e2e",
    "package": "ng run zhang-ui:build",  //这个就是打包的命令
    "analyze": "ng build --prod --build-optimizer --stats-json",
    "test-coverage": "ng test --code-coverage --watch=false",
    "lint:ts": "tslint -p src/tsconfig.app.json -c tslint.json 'src/**/*.ts'",
    "lint:style": "stylelint \"{src}/**/*.less\" --syntax less",
    "lint-staged": "lint-staged",
    "tslint-check": "tslint-config-prettier-check ./tslint.json",
    "hmr": "ng serve -c=hmr"
  }
```

执行
```
npm run package
```
或者
```
ng run zhang-ui:build
```
都可以，最后会打包到dist目录下
然后npm publish ./dist 就可以了

## 测试模块

为了方便查看效果

在`routes.module.ts`文件
```
├─app
│  └─routes
│          routes.module.ts
```

```javascript
import {ZhangComponentModule} from "../package/zhang-component.module"; //引入

...
@NgModule({
  imports: [SharedModule, RouteRoutingModule, ZhangComponentModule],  //导进来
  declarations: [
    ...COMPONENTS,
    ...COMPONENTS_NOROUNT
  ],
  entryComponents: COMPONENTS_NOROUNT
})
export class RoutesModule {
}

```
其他就按照angular的方式该怎么写怎么写行了
