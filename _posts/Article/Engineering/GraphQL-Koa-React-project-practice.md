---
title: GraphQL + Koa + React 项目实践
date: 2020-06-08 22:32:39
updated: 2020-06-16
authors:
  - zhangyanling77
categories:
  - Article
  - Engineering
tags:
  - GraphQL
  - Apollo
  - Koa
  - React
  - MongoDB

original: https://juejin.im/post/5ede47a45188253684677f61
toc: true
---

## 项目背景

源于 2019 年 11 月 16 日 [FCC 成都社区][1]主办的 [Web 全栈大会][2]上尹吉峰老师的 GraphQL 的分享，让我产生了浓厚的兴趣。GraphQL 是一个用于 API 的查询语言，是使用基于类型系统来执行查询的服务端运行时（类型系统由你的数据定义）。一个 GraphQL 服务是通过定义类型和类型上的字段来创建的，然后给每个类型上的每个字段提供解析函数。

参考学习资料：

- https://graphql.cn/learn/
- https://typescript.bootcss.com/basic-types.html
- https://www.apollographql.com/docs/react/

基于以上的一番学习，做了个实践的小项目，就代码做以下分析。

（附上项目地址：[react-graphql-project](https://github.com/zhangyanling77/react-graphql-project)）

<!-- more -->

## 项目目录

项目分为前端和后端两部分（目录 client 和 server），如图所示：

![目录截图](9fj3stocj2s.webp)

使用技术栈：

- client：React hooks + TypeScript + Apollo + GraphQL + Ant Design

- server: Koa 2 + GraphQL + koa-graphql + Mongoose

## 项目搭建及源码实现

### 数据库部分

使用的 MongoDB 数据库，这里对于该数据库的安装等不做赘述。

默认已经具备 MongoDB 的环境。接下来，启动数据库。

到 MongoDB 安装路径下，如 **C:\Program Files\MongoDB\Server\4.2\bin**

打开终端，执行命令：

```bash
mongod --dbpath=./data
```

创建项目总目录：react-graphql-project，并进入目录。

### 后端部分

#### 创建项目，并初始化

```bash
mkdir server && cd server
npm init -y
```

#### 安装项目依赖

```bash
yarn add koa koa-grphql koa2-cors koa-mount koa-logger graphql
```

#### 配置启动命令

`package.json`

```json
{
  "scripts": {
    "start": "nodemon index.js"
  }
}
```

#### 业务开发

##### 入口文件

`index.js`

> 这里我们启动一个 Koa 服务，进行日志监听，支持跨域操作，并将 GraphQL 服务挂到 Koa 服务上。
> 通过 koa-graphql 提供 HTTP 服务，传入 schema，并启动 graphiql。
> graphiql 在测试和开发过程中都非常有用，但生产环境下应禁用它。

```javascript
const Koa = require("koa");
const mount = require("koa-mount");
const graphqlHTTP = require("koa-graphql");
const cors = require("koa2-cors");
const logger = require("koa-logger");
const myGraphQLSchema = require("./schema");

const app = new Koa();
// 日志
app.use(logger());
// 跨域支持
app.use(
  cors({
    origin: "*",
    allowMethods: ["GET", "POST", "DELETE", "PUT", "OPTIONS"]
  })
);

app.use(
  mount(
    "/graphql",
    graphqlHTTP({
      schema: myGraphQLSchema,
      graphiql: true // 开启graphiql可视化操作playground
    })
  )
);

app.listen(4000, () => {
  console.log("server started on 4000");
});
```

##### 数据库连接

创建 model 文件 `model.js`

> 这里我们建立数据链接，定义 schema，并生成对应的 model 导出。
> schema 是 mongoose 里会用到的一种数据模式，可以理解为表结构的定义。每个 schema 会映射到 MongoDB 中的一个 collection，它并不具备操作数据库的能力。
> model 是由 schema 生成的模型，可以对数据库进行操作。

```javascript
const mongoose = require("mongoose");

const Schema = mongoose.Schema;
// 创建数据库连接
const conn = mongoose.createConnection("mongodb://localhost/graphql", {
  useNewUrlParser: true,
  useUnifiedTopology: true
});

conn.on("open", () => console.log("数据库连接成功！"));

conn.on("error", error => console.log(error));

// 用于定义表结构
const CategorySchema = new Schema({
  name: String
});
// 增删改查
const CategoryModel = conn.model("Category", CategorySchema);

const ProductSchema = new Schema({
  name: String,
  category: {
    type: Schema.Types.ObjectId, // 外键
    ref: "Category"
  }
});

const ProductModel = conn.model("Product", ProductSchema);

module.exports = {
  CategoryModel,
  ProductModel
};
```

`schema.js`

> 定义查询的 schema 对象。在 GraphGL 中有许多内置的 Schema Types 可供我们用来定义字段名类型。
> 这里我们通过定义查询对象类型，通过 model 就可以对数据库进行增、删、改、查等相应操作了。

```javascript
const graphql = require('graphql');
const { CategoryModel, ProductModel } = require('./model');

const {
  GraphQLObjectType,
  GraphQLString,
  GraphQLSchema,
  GraphQLList,
  GraphQLNonNull
}  = graphql
// 分类类型定义
const Category = new GraphQLObjectType({
  name: 'Category',
  fields: () => (
    {
      id: { type: GraphQLString },
      name: { type: GraphQLString },
      products: {
        type: new GraphQLList(Product),
        async resolve(parent){
          let result = await ProductModel.find({ category: parent.id })
          return result
        }
      }
    }
  )
})
// 商品类型定义
const Product = new GraphQLObjectType({
  name: 'Product',
  fields: () => (
    // ...
  )
})
// 根查询对象
const RootQuery = new GraphQLObjectType({
  name: 'RootQuery',
  fields: {
    getCategory: { // 通过id获取分类
      type: Category,
      args: {
        id: { type: new GraphQLNonNull(GraphQLString) }
      },
      async resolve(parent, args){
        let result = await CategoryModel.findById(args.id)
        return result
      }
    },
    // ... 其他查询定义
  }
})
// 根变更对象
const RootMutation = new GraphQLObjectType({
  name: 'RootMutation',
  fields: {
    addCategory: { //根据name添加分类
      type: Category,
      args: {
        name: { type: new GraphQLNonNull(GraphQLString) }
      },
      async resolve(parent, args){
        let result = await CategoryModel.create(args)
        return result
      }
    },
    // ... 其他变更定义
  }
})

module.exports = new GraphQLSchema({
  query: RootQuery,
  mutation: RootMutation
})
```

#### 启动项目

```bash
yarn start
```

访问 http://localhost:4000/graphql 看到数据库操作 playground 界面。可进行一系列数据库 CRUD 操作。

### 前端部分

#### 创建项目

```bash
npx create-react-app client --template typescript
```

#### 配置 webpack

```bash
yarn add react-app-rewired customize-cra
```

更改 `package.json` 文件的 `scripts` 启动命令

```json
{
  "scripts": {
    "start": "react-app-rewired start",
    "build": "react-app-rewired build",
    "test": "react-app-rewired test"
  }
}
```

然后在根目录下新建 `config-overrides.js` 文件，以添加 webpack 的相关配置。

安装前端 UI 组件库 `antd`，并配置按需加载、路径别名支持等。

```Shell
yarn add antd babel-plugin-import
```

`config-overrides.js`

```javascript
const { override, fixBabelImports, addWebpackAlias } = require("customize-cra");
const path = require("path");

module.exports = override(
  fixBabelImports("import", {
    libraryName: "antd",
    libraryDirectory: "es",
    style: "css"
  }),
  addWebpackAlias({
    "@": path.resolve(__dirname, "src/")
  })
);
```

> 因为 TS 无法识别，还需配置 tconfig.json 文件。

新建 `paths.json` 文件

```json
{
  "compilerOptions": {
    "baseUrl": ".",
    "paths": {
      "@/*": ["src/*"]
    }
  }
}
```

更改 `tconfig.json`

```json
{
  "compilerOptions": {
    "target": "ES5",
    "lib": ["DOM", "DOM.Iterable", "ESNext"],
    "allowJs": true,
    "jsx": "react"
  },
  "include": ["./src/**/*"],
  "extends": "./paths.json"
}
```

重启项目后生效。

#### 业务开发

##### 入口文件

`index.tsx`

> 首先我们需要创建 Apollo 客户端，传入启动的后端地址作为 uri 的值，将生成的客户端实例通过 context 注入到整个单页应用中。

```javascript
// ...
import ApolloClient from "apollo-boost";
import { ApolloProvider } from "@apollo/react-hooks";
import App from "./router";
// 创建apollo客户端
const client = new ApolloClient({
  uri: "http://localhost:4000/graphql"
});

ReactDOM.render(
  <ApolloProvider client={client}>
    <App />
  </ApolloProvider>,
  document.getElementById("root")
);
```

##### 路由文件

`router.js`

> 主要包括商品列表页、商品详情等路由的配置。

```javascript
import React, { Suspense, lazy, memo } from "react";
import { BrowserRouter as Router, Route, Switch } from "react-router-dom";
import { Spin } from "antd";

const Layouts = lazy(() => import("@/components/layouts"));
const ProductList = lazy(() => import("@/pages/productlist"));
const ProductDetail = lazy(() => import("@/pages/productdetail"));

const RouterComponent = () => {
  return (
    <Router>
      <Suspense fallback={<Spin size="large" />}>
        <Layouts>
          <Switch>
            <Route path="/" exact={true} component={ProductList} />
            <Route path="/detail/:id" component={ProductDetail} />
            <Route render={() => <h1>404 Not Found</h1>} />
          </Switch>
        </Layouts>
      </Suspense>
    </Router>
  );
};
```

##### 定义类型文件

`types.ts`

```typescript
export interface Category {
  id?: string;
  name?: string;
  products: Array<Product>;
}

export interface Product {
  id?: string;
  name?: string;
  category?: Category;
  categoryId?: string | [];
}
```

##### 布局组件

`src/components/layouts`

```javascript
import React from "react";
import { Layout, Menu } from "antd";
import { Link } from "react-router-dom";

const { Header, Content, Footer } = Layout;

const Layouts = props => (
  <Layout className="layout">
    <Header>
      <div className="logo" />
      <Menu
        theme="dark"
        mode="horizontal"
        defaultSelectedKeys={["1"]}
        style={{ lineHeight: "64px" }}
      >
        <Menu.Item key="1">
          <Link to="/">商品管理</Link>
        </Menu.Item>
      </Menu>
    </Header>
    <Content style={{ padding: "50px 50px 0 50px" }}>
      <div style={{ background: "#fff", padding: 24, minHeight: 280 }}>
        {props.children}
      </div>
    </Content>
    <Footer style={{ textAlign: "center" }}>
      ©2019 - {new Date().getFullYear()} Created by zhangyanling77.{" "}
    </Footer>
  </Layout>
);
export default Layouts;
```

##### GraphQL 查询语句文件

`api.ts`

```typescript
import { gql } from "apollo-boost";
// 获取所有的商品
export const GET_PRODUCTS = gql`
  query {
    getProducts {
      id
      name
      category {
        id
        name
        products {
          id
          name
        }
      }
    }
  }
`;
// ... 其他查询语句定义
```

##### 商品列表组件 ProductList

> 实现商品列表展示、删除商品、新增商品等功能。

![商品列表页](1patw3gmssb.webp)

![删除商品](xnh3kuvujl.webp)

![新增商品](1tgd9h2osm8.webp)

```javascript
// ... 其他依赖引入
import { useQuery, useMutation } from "@apollo/react-hooks";
import {
  CATEGORIES_PRODUCTS,
  GET_PRODUCTS,
  ADD_PRODUCT,
  DELETE_PRODUCT
} from "@/api";
// ...

const ProductList = () => {
  // ... 其他状态定义
  const { loading, error, data } = useQuery(CATEGORIES_PRODUCTS);
  const [deleteProduct] = useMutation(DELETE_PRODUCT);

  // ... 错误处理，加载中处理

  const { getCategories, getProducts } = data;

  const confirm = async (event, record) => {
    // 删除商品
    await deleteProduct({
      variables: {
        id: record?.id
      },
      refetchQueries: [
        {
          query: GET_PRODUCTS
        }
      ]
    });
    setCurrent(1);
  };

  const columns = [
    // ...
  ];

  // ...

  return (
    <div>
      <Row style={{ padding: "0 0 20px 0" }}>
        <Col span={24}>
          <Button type="primary" onClick={() => setVisible(true)}>
            新增
          </Button>
        </Col>
      </Row>
      {/* ... */}
      {visible && (
        <AddForm
          handleOk={handleOk}
          handleCancel={handleCancel}
          categories={getCategories}
        />
      )}
    </div>
  );
};

const AddForm = ({ handleOk, handleCancel, categories }) => {
  // ... 其他状态定义
  let [addProduct] = useMutation(ADD_PRODUCT);

  const handleSubmit = async () => {
    await addProduct({
      variables: product,
      refetchQueries: [
        {
          // 添加成功后执行查询全部商品的操作，是个回调
          query: GET_PRODUCTS
        }
      ]
    });
    setProduct({ name: "", categoryId: [] }); // 清空表单
    handleOk();
  };

  return (
    <Modal
      title="新增产品"
      // ...
    >
      <Form>{/* ... */}</Form>
    </Modal>
  );
};

export default ProductList;
```

##### 商品详情组件 ProductDetail

> 根据 ID 查询商品详情及其所属商品分类下的所有商品。

![商品详情](8bfps0iinjw.webp)

```javascript
// ...
import { useQuery } from "@apollo/react-hooks";
import { GET_PRODUCT } from "@/api";
import { Product } from "@/types";

const ProductDetail = props => {
  let _id = props.match.params.id;
  let { loading, error, data } = useQuery(GET_PRODUCT, {
    variables: { id: _id }
  });

  // ... 错误处理

  const { getProduct } = data;
  const {
    id,
    name,
    category: { id: categoryId, name: categoryName, products }
  } = getProduct;

  return (
    <div>
      <Card title="商品详情" bordered={false} style={{ width: "100%" }}>
        <div>
          <p>
            <b>商品ID：</b>
            {id}
          </p>
          <p>
            <b>商品名称：</b>
            {name}
          </p>
        </div>
        // ... 商品列表展示
      </Card>
    </div>
  );
};

export default ProductDetail;
```

## 结语

通过这个项目实践，基本掌握了 GraphQL 的使用。虽然这个项目只包含了简单的 CRUD 功能，但是对后端、数据库、前端都涉及到了，因此对于学习拓展来说也是不错的。后续也继续实现了登录验证、个人中心等功能，这里不做详细介绍，可自行查看项目代码了解。

[1]: https://fcc-cd.dev/
[2]: https://web-conf.dev/#2019
