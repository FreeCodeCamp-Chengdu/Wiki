---
title: 【工作坊】Web 自动化测试
date: 2022-01-03 02:31:34
categories:
  - Activity
  - Workshop
tags:
  - Web
  - automation
  - Puppeteer
  - Node.JS
toc: true
photos:

start: 2022-01-15 13:30:00
end: 2022-01-15 17:30:00
address: 成都市高新区天府五街菁蓉汇2A栋5层
mentors:
  - TechQuery
workers:
partners:
  - SKFI
  - ACE-CD
  - HDZ
---

2022 新年钟声的敲响有没有让开发小哥哥、测试小姐姐们心头一紧？马上过年了，2021 年的项目还一堆功能和 bug 没搞完，真令人头秃……

小编干了小十年开发，每到项目交付前，再美的测试小姐姐我每天也不想见到她。因为大多数团队每次为了赶进度，在一期工程基本一屁股技术债 ——

> 单元测试是不可能写的，这辈子都不会写单元测试；集成测试更不要想，只能靠测试小姐姐点点点这样维持一点质量……

而且俗话说得好：

> 码农三大懒 —— 变量命名、注释/文档、测试代码

辣这年前上线啷个弄嘛？

…… 也不是不可以抢救一下~

<!-- more -->

## 知识精讲

### Web

#### DOM 模拟库

1. JSDOM：稳健老前辈
2. HappyDOM：轻快新后生

#### 浏览器操作库

1. Selenium 开天辟地：ThoughtWorks
2. Puppeteer 改天换地：Google
3. Playwright 另起炉灶：Microsoft

#### 录制器（<em style="color: red">偷懒神器</em>）

**测试小姐姐**一边点点点，测试代码自动录制出来，\*~~代码小白的黑盒测试专员~~立马转职**会玩代码的白盒测试工程师\***，过年回家又可以吊打亲戚家孩子了~😜

#### 运行器/套件

1. Jest：使用广泛
2. Codecept：封装全面（<em style="color: red">偷懒神器</em>）
   - 中国技术宅小时候不少都玩过**易语言**中文编程，那用各国语言写测试代码你玩过吗？这又是一个**测试小姐姐**不可错过的升职加薪、吊打凤凰男/普信男的绝佳机会~

```javascript
Feature("CodeceptJS 演示");

Scenario("成功提交表单", ({ 我 }) => {
  我.在页面("/documentation");
  我.填写字段("电邮", "hello@world.com");
  我.填写字段("密码", "123456");
  我.勾选选项("激活");
  我.勾选选项("男");
  我.单击("创建用户");
  我.看到("用户名可用");
  我.在当前网址中看不到("/documentation");
});
```

### 小程序

微信开发者工具内置**操作脚本录制器**，也开发了类似浏览器操作库的 API，结合 Web 前端已有的测试框架，小程序这么中国特色的玩意儿也能做自动化测试啦~

### 后端

1. APIJSON：智能测试框架
   - 腾讯工程师基于机器学习开发的自动化 API、UI 测试框架
   - 开源中国 GVP 获奖项目
2. Apifox = Postman + Swagger + Mock + JMeter
   - 国内 API as a Service 创业厂商

### 两种自动化测试模型

#### 测试金字塔

#### 测试冠军杯

## 动手实践

<em style="color: red">课前预习，课上带电脑！</em>

> 实操案例：**Next.js + Jest**
>
> —— 来自脚手架 [@idea2app/Next-Bootstrap-ts](https://github.com/idea2app/next-bootstrap-ts)

### 安装

```shell
npm i -D jest playwright jest-playwright-preset \
    @jest/types ts-node @types/jest ts-jest
npx playwright install
```

### tsconfig.json

```json
{
  "compilerOptions": {
    "types": ["@types/jest", "jest-playwright-preset"]
  }
}
```

### jest.config.ts

```typescript
import type { value Config } from "@jest/types";

const config: Config.InitialOptions = {
  preset: "jest-playwright-preset",
  transform: {
    "^.+\\.ts$": "ts-jest"
  },
  testTimeout: 60000 // 根据后端网速调节
};
export default config;
```

### jest-playwright.config.js

```javascript
module.exports = {
  browsers: ["chromium", "firefox", "webkit"],
  launchOptions: {
    headless: false // 仅本机测试
  },
  serverOptions: {
    command: "npm run dev",
    port: 3000,
    debug: true // 仅本机测试
  }
};
```

### .vscode/launch.json

```json
{
  "version": "0.2.0",
  "configurations": [
    {
      "name": "Debug Jest",
      "type": "node",
      "request": "launch",
      "port": 9229,
      "runtimeArgs": [
        "--inspect-brk",
        "${workspaceRoot}/node_modules/jest/bin/jest.js",
        "--runInBand"
      ]
    }
  ]
}
```

### test/home.spec.ts

```typescript
describe("Home page", () => {
  beforeAll(() =>
    page.goto("http://localhost:3000", {
      timeout: 60000 // 根据后端网速调节
    })
  );

  it("should load Main framework", async () => {
    expect(await page.title()).toBe("Home page");
  });
});

export {};
```

## 活动概况

- 受众：
  - Web 前后端工程师
  - 测试专员、工程师
  - 项目经理
- 费用：￥ 0.00
