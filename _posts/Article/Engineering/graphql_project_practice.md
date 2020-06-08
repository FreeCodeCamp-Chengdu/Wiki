---
title: 基于graphql+react+apollo（前端）、koa+mongodb（后端）的项目实践
date: 2020-06-08
authors:
  - zhangyanling77
categories:
  - Article
  - Engineering
tags:
  - GraphQL
  - Apollo
  - Koa2
  - mongodb

original: https://juejin.im/post/5ede47a45188253684677f61
toc: true
---

## 项目背景

源于2019年11月16日成都Web全栈大会上尹吉峰老师的 `GraphQL` 的分享，让我产生了浓厚的兴趣。几经研究、学习，做了个实践的小项目。

学习资料：

[graphql.cn/learn/](https://graphql.cn/learn/)

[typescript.bootcss.com/basic-types.html](https://typescript.bootcss.com/basic-types.html)

[www.apollographql.com/docs/react/](https://www.apollographql.com/docs/react/)

就代码做以下分析。

## 项目目录

项目分为前端和后端两部分（目录client和server）。如图所示，

![目录截图](https://user-gold-cdn.xitu.io/2020/6/8/172944971aa77952?imageslim)

使用技术栈：

- client：react hooks + typescript + apollo + graphql + antd

- server:  koa2 + graphql + koa-graphql + mongoose

## 项目搭建及源码实现

### 数据库部分

使用的 `mongodb` 数据库，这里对于该数据库的安装等不做赘述。

默认已经 具备 `mongodb` 的环境。启动数据库。

到 `mongodb` 安装路径下，如**C:\Program Files\MongoDB\Server\4.2\bin**

打开终端，执行命令：

```bash
mongod --dbpath=./data
```

> 创建项目总目录：react-graphql-project，并进入目录

### 后端部分

- 创建项目

```bash
mkdir server && cd server
npm init -y
```

- 安装项目依赖

```bash
yarn add koa koa-grphql koa2-cors koa-mount koa-logger graphql
```

- 配置启动命令

`package.json`

```json
{
  "name": "server",
  "version": "1.0.0",
  "description": "",
  "main": "index.js",
  "scripts": {
    "start": "nodemon index.js"
  },
  "keywords": [],
  "author": "zhangyanling",
  "license": "MIT",
  "dependencies": {
    "graphql": "^14.5.8",
    "koa": "^2.11.0",
    "koa-graphql": "^0.8.0",
    "koa-logger": "^3.2.1",
    "koa-mount": "^4.0.0",
    "koa2-cors": "^2.0.6",
    "mongoose": "^5.7.11"
  }
}
```

- 业务开发

入口文件 `index.js`

```javascript
const Koa = require('koa');
const mount = require('koa-mount');
const graphqlHTTP = require('koa-graphql');
const cors = require('koa2-cors'); // 解决跨域
const logger = require('koa-logger'); // 日志输出
const myGraphQLSchema = require('./schema');

const app = new Koa();

app.use(logger())

app.use(cors({
  origin: '*',
  allowMethods: ['GET', 'POST', 'DELETE', 'PUT', 'OPTIONS']
}))

app.use(mount('/graphql', graphqlHTTP({
  schema: myGraphQLSchema,
  graphiql: true // 开启graphiql可视化操作playground
})))

app.listen(4000, () => {
  console.log('server started on 4000')
})
```

数据库连接，创建model文件 `model.js`

```javascript
const mongoose = require('mongoose');

const Schema = mongoose.Schema;
// 创建数据库连接
const conn = mongoose.createConnection('mongodb://localhost/graphql',{ useNewUrlParser: true, useUnifiedTopology: true });

conn.on('open', () => console.log('数据库连接成功！'));

conn.on('error', (error) => console.log(error));

// 用于定义表结构
const CategorySchema = new Schema({
  name: String
});
// 增删改查
const CategoryModel = conn.model('Category', CategorySchema);

const ProductSchema = new Schema({
  name: String,
  category: {
    type: Schema.Types.ObjectId, // 外键
    ref: 'Category'  
  }
});

const ProductModel = conn.model('Product', ProductSchema);

module.exports = {
  CategoryModel,
  ProductModel
}
```

`schema.js`
> 定义查询的 schema 对象

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

const Product = new GraphQLObjectType({
  name: 'Product',
  fields: () => (
    {
      id: { type: GraphQLString },
      name: { type: GraphQLString },
      category: {
        type: Category,
        async resolve(parent){
          let result = await CategoryModel.findById(parent.category)
          return result
        }
      }
    }
  )
})


const RootQuery = new GraphQLObjectType({
  name: 'RootQuery',
  fields: {
    getCategory: {
      type: Category,
      args: {
        id: { type: new GraphQLNonNull(GraphQLString) }
      },
      async resolve(parent, args){
        let result = await CategoryModel.findById(args.id)
        return result
      }
    },
    getCategories: {
      type: new GraphQLList(Category),
      args: {},
      async resolve(parent, args){
        let result = await CategoryModel.find()
        return result
      }
    },
    getProduct: {
      type: Product,
      args: {
        id: { type: new GraphQLNonNull(GraphQLString) }
      },
      async resolve(parent, args){
        let result = await ProductModel.findById(args.id)
        return result 
      }
    },
    getProducts: {
      type: new GraphQLList(Product),
      args: {},
      async resolve(parent, args){
        let result = await ProductModel.find()
        return result 
      }
    }
  }
})

const RootMutation = new GraphQLObjectType({
  name: 'RootMutation',
  fields: {
    addCategory: {
      type: Category,
      args: {
        name: { type: new GraphQLNonNull(GraphQLString) }
      },
      async resolve(parent, args){
        let result = await CategoryModel.create(args)
        return result  
      }
    },
    addProduct: {
      type: Product,
      args: {
        name: { type: new GraphQLNonNull(GraphQLString) },
        category: { type: new GraphQLNonNull(GraphQLString) }
      },
      async resolve(parent, args){
        let result = await ProductModel.create(args)
        return result 
      }
    },
    deleteProduct: {
      type: Product,
      args: {
        id: { type: new GraphQLNonNull(GraphQLString) },
      },
      async resolve(parent, args){
        let result = await ProductModel.deleteOne({"_id": args.id})
        return result
      }
    }
  }
})

module.exports = new GraphQLSchema({
  query: RootQuery,
  mutation: RootMutation
})
```

- 启动项目

```bash
yarn start
```

访问 **http://localhost:4000/graphql** 看到数据库操作playground界面。可进行一系列数据库crud操作。

## 前端部分

- 创建项目

```bash
npx create-react-app react-graphql-project --template typescript
```

- 配置webpack

```bash
yarn add react-app-rewired customize-cra
```

更改 `package.json` 文件的 `scripts` 启动命令

```json
"scripts": {
  "start": "react-app-rewired start",
  "build": "react-app-rewired build",
  "test": "react-app-rewired test"
}
```

然后在根目录下新建 `config-overrides.js` 文件，以做 `webpack` 的相关配置。

安装前端UI组件库 `antd`，并配置按需加载、路径别名支持等。

```bash
yarn add antd babel-plugin-import 
```

`config-overrides.js`

```javascript
const { override, fixBabelImports, addWebpackAlias } = require('customize-cra');
const path = require('path')

module.exports = override(
  fixBabelImports('import', {
    libraryName: 'antd',
    libraryDirectory: 'es',
    style: 'css'
  }),
  addWebpackAlias({
    "@": path.resolve(__dirname, "src/")              
  })
)
```

> 因为ts无法识别，还需配置tconfig.json 文件。

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
    "target": "es5",
    "lib": [
      "dom",
      "dom.iterable",
      "esnext"
    ],
    "allowJs": true,
    "skipLibCheck": true,
    "esModuleInterop": true,
    "allowSyntheticDefaultImports": true,
    "strict": true,
    "forceConsistentCasingInFileNames": true,
    "module": "esnext",
    "moduleResolution": "node",
    "resolveJsonModule": true,
    "isolatedModules": true,
    "noEmit": true,
    "jsx": "react"
  },
  "include": [
    "./src/**/*"
  ],
  "extends": "./paths.json"
}
```

重启项目后生效。

- 业务开发

入口文件 `index.tsx`

```tsx
import React from 'react';
import ReactDOM from 'react-dom';
import ApolloClient from 'apollo-boost';
import { ApolloProvider } from '@apollo/react-hooks';
import App from './router';
import * as serviceWorker from './serviceWorker';
// 创建apollo客户端
const client = new ApolloClient({
  uri: 'http://localhost:4000/graphql'
})

ReactDOM.render(
  <ApolloProvider client={client}>
    <App />
  </ApolloProvider>, document.getElementById('root'));
serviceWorker.unregister();
```

路由文件 `router.js`

```javascript
import React, { Suspense, lazy } from 'react';
import { BrowserRouter as Router, Route, Switch } from 'react-router-dom';
import { Spin } from 'antd';

// 懒加载组件
const Layouts = lazy(() => import('@/components/layouts'));
const ProductList = lazy(() => import('@/pages/productlist'));
const ProductDetail = lazy(() => import('@/pages/productdetail'));


const RouterComponent = () => {
  return (
    <Router>
      <Suspense fallback={<Spin size="large" />}>
        <Layouts>
          <Switch>
            <Route path="/" exact={true} component={ProductList}></Route>
            <Route path="/detail/:id" component={ProductDetail}></Route>
          </Switch>
        </Layouts>
      </Suspense>
    </Router>
  )
};

export default RouterComponent;
```

定义类型文件 `types.tsx`

```tsx
export interface Category{
  id?: string;
  name?: string;
  products: Array<Product>
}

export interface Product{
  id?:string;
  name?: string;
  category?: Category;
  categoryId?: string | [];
}
```

开发布局组件 `src/components/layouts`

```tsx
import React from 'react';
import { Layout, Menu } from 'antd';
import { Link } from 'react-router-dom';

const { Header, Content, Footer } = Layout

const Layouts: React.FC = (props) => (
  <Layout className="layout">
    <Header>
      <div className="logo" />
      <Menu
        theme="dark"
        mode="horizontal"
        defaultSelectedKeys={['1']}
        style={{ lineHeight: '64px' }}
      >
        <Menu.Item key="1"><Link to="/">商品管理</Link></Menu.Item>
      </Menu>
    </Header>
    <Content style={{ padding: '50px 50px 0 50px' }}>
      <div style={{ background: '#fff', padding: 24, minHeight: 280 }}>
        {props.children}
      </div>
    </Content>
    <Footer style={{ textAlign: 'center' }}> ©2019 Created by zhangyanling. </Footer>
  </Layout>
)


export default Layouts;
```

定义gql查询语句文件 `api.tsx`

```tsx
import { gql } from 'apollo-boost';

export const GET_PRODUCTS = gql`
  query{
    getProducts{
      id
      name
      category{
        id
        name
        products{
          id
          name
        }
      }
    }
  }
`;

// 查询所有的上屏分类和产品
export const CATEGORIES_PRODUCTS = gql`
  query{
    getCategories{
      id
      name
      products{
        id
        name
      }
    }
    getProducts{
      id
      name
      category{
        id
        name
        products{
          id
          name
        }
      }
    }
  }
`;

// 添加产品
export const ADD_PRODUCT = gql`
  mutation($name:String!, $categoryId:String!){
    addProduct(name: $name, category: $categoryId){
      id
      name
      category{
        id
        name
      }
    }
  }
`;

// 根据id删除产品
export const DELETE_PRODUCT = gql`
  mutation($id: String!){
    deleteProduct(id: $id){
      id
      name
    }
  }
`;

// 根据id查询商品详情及相应商品分类及所属分类全部商品
export const GET_PRODUCT = gql`
  query($id: String!){
    getProduct(id: $id){
      id
      name
      category{
        id
        name
        products{
          id
          name
        }
      }
    }
  }
`;
```

开发商品列表组件 `ProductList`

> 已经实现商品列表展示、删除商品、新增商品等功能。

```tsx
import React, { useState } from 'react';
import { Table, Modal, Row, Col, Button, Divider, Tag, Form, Input, Select, Popconfirm } from 'antd';
import { Link } from 'react-router-dom';
import { useQuery, useMutation } from '@apollo/react-hooks';
import { CATEGORIES_PRODUCTS, GET_PRODUCTS, ADD_PRODUCT, DELETE_PRODUCT } from '@/api';
import { Product, Category } from '@/types';

const { Option } = Select;
/**
 * 商品列表
 */
const ProductList: React.FC = () => {
  let [visible, setVisible] = useState<boolean>(false);
  let [pageSize, setPageSize] = useState<number|undefined>(10);
  let [current, setCurrent] = useState<number|undefined>(1)
  const { loading, error, data } = useQuery(CATEGORIES_PRODUCTS);
  const [deleteProduct] = useMutation(DELETE_PRODUCT);
 
  if(error) return <p>加载发生错误</p>;
  if(loading) return <p>加载中...</p>;
  
  const { getCategories, getProducts } = data

  const confirm = async (event?:any, record?:Product) => {
    // console.log("详情",  record);
    await deleteProduct({
      variables: {
        id: record?.id
      },
      refetchQueries: [{
        query: GET_PRODUCTS
      }]
    })
    setCurrent(1)
  }
   
  const columns = [
    {
      title: "商品ID",
      dataIndex: "id"
    },
    {
      title: "商品名称",
      dataIndex: "name"
    },
    {
      title: "商品分类",
      dataIndex: "category",
      render: (text: any) => {
        let color = ''
        const tagName = text.name;
        if(tagName === '服饰'){
          color = 'red'
        } else if(tagName === '食品') {
          color = 'green'
        } else if(tagName === '数码'){
          color = 'blue'
        } else if(tagName === '母婴'){
          color = 'purple'
        }
        return (
          <Tag color={color}>{text.name}</Tag>
        )
      }
    },
    {
      title: "操作",
      render: (text: any, record: any) => (
        <span>
          <Link to={`/detail/${record.id}`}>详情</Link>
          {/* <Divider type="vertical" /> */}
          {/* <a style={{color: 'orange'}}>修改</a> */}
          <Divider type="vertical" />
          <Popconfirm
            title="确定删除吗?"
            onConfirm={(event) => confirm(event, record)}
            okText="确定"
            cancelText="取消"
          >
            <a style={{color:'red'}}>删除</a>
          </Popconfirm>
        </span>
      )
    }
  ];

  const handleOk = () => {
    setVisible(false)
  }

  const handleCancel = () => {
    setVisible(false)
  }

  const handleChange = (pagination: { current?:number, pageSize?:number}) => {
    const { current, pageSize } = pagination
    setPageSize(pageSize)
    setCurrent(current)
  }

  return (
    <div>
      <Row style={{padding: '0 0 20px 0'}}>
        <Col span={24}>
          <Button type="primary" onClick={() => setVisible(true)}>新增</Button>
        </Col>
      </Row>
      <Row>
        <Col span={24}>
          <Table 
            columns={columns} 
            dataSource={getProducts} 
            rowKey="id" 
            pagination={{
              current: current,
              pageSize: pageSize,
              showSizeChanger: true,
              showQuickJumper: true,
              total: data.length
            }}
            onChange={handleChange}
          />
        </Col>
      </Row>
      {
        visible && <AddForm handleOk={handleOk} handleCancel={handleCancel} categories={getCategories} />
      }
    </div>
  )
}

/**
 * 新增产品Modal
 */
interface FormProps {
  handleOk: any,
  handleCancel: any,
  categories: Array<Category>
}

const AddForm:React.FC<FormProps> = ({handleOk, handleCancel, categories}) => {
  let [product, setProduct] = useState<Product>({ name: '', categoryId: [] });
  let [addProduct] = useMutation(ADD_PRODUCT);

  const handleSubmit = async () => {
    // 获取表单的值
    await addProduct({
      variables: product,
      refetchQueries: [{
        query: GET_PRODUCTS
      }]
    })
    // 清空表单
    setProduct({ name: '', categoryId: [] })
    handleOk()
  }
  
  return (
    <Modal
      title="新增产品"
      visible={true}
      onOk={handleSubmit}
      okText="提交"
      cancelText="取消"
      onCancel={handleCancel}
      maskClosable={false}
    >
      <Form>
        <Form.Item label="商品名称">
          <Input 
            placeholder="请输入" 
            value={product.name} 
            onChange={event => setProduct({ ...product, name: event.target.value })} 
          />
        </Form.Item>
        <Form.Item label="商品分类">
          <Select 
            placeholder="请选择" 
            value={product.categoryId} 
            onChange={(value: string | []) => setProduct({ ...product, categoryId: value })}
          >
            {
              categories.map((item: Category) => (
                <Option key={item.id} value={item.id}>{item.name}</Option>
              ))
            }
          </Select>
        </Form.Item>
      </Form>
    </Modal>
  )
}

export default ProductList;
```

开发商品详情组件 `ProductDetail`

> 根据ID查询商品详情及其所属商品分类下的所有商品。

```tsx
import React from 'react';
import { Card, List, Spin, Alert } from 'antd';
import { useQuery } from '@apollo/react-hooks';
import { GET_PRODUCT } from '@/api';
import { Product } from '@/types';

const ProductDetail: React.FC = (props:any) => {
  let _id = props.match.params.id;
  let { loading, error, data } = useQuery(GET_PRODUCT,{
    variables: { id: _id }
  });

   if(error) return <p>加载发生错误</p>;

  if(loading) return  <Spin size="large" tip="Loading..." />;
 
  const { getProduct } = data; 
  const { id, name, category: { id: categoryId, name: categoryName, products }} = getProduct;
  
  return (
    <div>
      <Card title="商品详情" bordered={false} style={{ width:'100%' }}>
        <div>
          <p><b>商品ID：</b>{id}</p>
          <p><b>商品名称：</b>{name}</p>
        </div>
        <List
          header={
            <div>
              <p><b>分类ID：</b>{categoryId}</p>
              <p><b>分类名称：</b>{categoryName}</p>
            </div>
          }
          footer={null}
          bordered
          dataSource={products}
          renderItem={(item:Product) => (
            <List.Item>
              <p>{item.name}</p>
            </List.Item>
          )}
        >
        </List>
      </Card>
    </div>
  )
}


export default ProductDetail;
```

## 效果图展示

商品列表页

![商品列表页](https://user-gold-cdn.xitu.io/2020/6/8/172945734b46e980?imageslim)

新增商品

![新增商品](https://user-gold-cdn.xitu.io/2020/6/8/1729457a28a71754?imageslim)

删除商品

![删除商品](https://user-gold-cdn.xitu.io/2020/6/8/1729457f07d3777d?imageslim)

商品详情

![商品详情](https://user-gold-cdn.xitu.io/2020/6/8/1729458412bf9ffe?imageslim)


## 结语

通过这个项目实践，基本掌握了 GraphQL 的使用。虽然这只是一个简单的CRUD功能，但是对后端、数据库、前端都涉及到了，因此对于学习拓展来说也是不错的。

附上项目地址：[react-graphql-project](https://github.com/zhangyanling77/react-graphql-project)
