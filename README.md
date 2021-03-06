# GraphQL

* GraphQL

    * 페이스북에서 만든 API로 쿼리를 호출 할 수 있게 함

    * 하나의 endpoint 를 통하여 사용

    * 백엔드 개발자에게 요청 할 필요 없이, 프론트에서 자유롭게 호출 가능

    * example

        ```javascript
        // url - /graphql
        
        {
          product(id:1){
            id
            price
            name
          },
            cart(user_id:10){
              thumbnail
              price
            }
        }
        ```

         

    * 타입 - 기본
      * Int : 부호가 있는 32비트 정수
      * Float : 부호가 있는 부동소수점 값
      * String : UTP-8 문자열
      * Boolean : true / false

    * RESTAPI 와 비교

        ```
        // REST API
        
        * GET - /products/1
        * GET - /users/10/cart
        ```

        

    * 설치

        ```shell
        npm install graphql
        ```

        

    * 사용예제

        ```javascript
        // start.js
        
        const { graphql, buildSchema } = require("graphql");
        
        // 쿼리 생성
        const schema = buildSchema(`
          type Query {
            hello: String,
            nodejs: Int
          }
        `);
        
        // 응답 생성
        const root = {
          hello: () => "Hello world!",
          nodejs: () => 20,
        };
        
        // 쿼리 요청
        graphql(schema, "{ hello }", root).then((response) => {
          console.log(response);
        });
        
        ```

    

* Express 연동

    *  설치

        ```shell
        npm install --save express express-graphql
        ```

    

    * graphql 서버 구동

      ```javascript
      const express = require("express");
      const { graphqlHTTP } = require("express-graphql");
      const { graphql } = require("graphql");
      const { buildSchema } = require("graphql");
      
      // query 생성
      const schema = buildSchema(`type Query{
          hello: String,
          nodejs: Int
      }`);
      
      // query 응답 생성
      const root = {
        hello: () => "hello world",
        nodejs: () => 20,
      };
      
      const app = express();
      
      // graphql endpoint 통일
      app.use(
        "/graphql",
        graphqlHTTP({
          schema: schema,
          rootValue: root,
          // graphql gui
          graphiql: true,
        })
      );
      
      app.listen(4000, () => {
        console.log("running server port 4000");
      });
      
      ```




* DATA READ

  * 데이터 구조 / 쿼리 작성

    ```javascript
    const schema = buildSchema(`
    	## '!' 는 필수 항목 의미
    	type Product {
    		id: ID!
    		name: String
    		price: Int
    		description: String
    		}
    
    	## 쿼리
    	type Query{
    		getProduct(id: ID!): Product
    	}`);
    ```

  * 임시 데이터 작성

    ```javascript
    const products = [
      {
        id: 1,
        name: "첫번째 제품",
        price: 2000,
        description: "히히히",
      },
      {
        id: 2,
        name: "두번째 제품",
        price: 4000,
        description: "허허허",
      },
    ];
    ```

  * 동작/응답 작성

    ```javascript
    const root = {
      getProduct: ({ id }) =>
        products.find((product) => product.id === parseInt(id)),
    };
    ```

  * 데이터 읽기

    ```javascript
    ## 데이터를 불러오기 위한 쿼리
    {
      getProduct(id: 2){
        name
        price
      }
    }
    ```

  * 결과

    ```javascript
    {
      "data": {
        "getProduct":{
          "name": "두번째 제품",
          "price": 4000
        }
      }
    }
    ```

    

* DATA WRITE

  * 데이터 입력 스키마 작성

    ```javascript
    const schema = buildSchema(`
      ## 데이터 입력 Form
      input ProductInput{
        name: String
        price: Int
        description: String
      }
    
      type Product {
        id: ID!
        name: String
        price: Int
        description: String
      }
    
      type Query{
        getProduct(id : ID!) : Product
      }
    
      ## Mutation - 새로운 데이터 추가나 수정시 사용하는 graphQL 문
      type Mutation{
        addProduct( input: ProductInput) : Product
      }
    `);
    ```

  * 응답 작성

    ```javascript
    const root = {
      getProduct: ({ id }) =>
        products.find((product) => product.id === parseInt(id)),
    
      addProduct: ({ input }) => {
        input.id = parseInt(products.length + 1);
        products.push(input);
        return root.getProduct({ id: input.id });
      },
    };
    ```

  * 실제 요청

    ```
    POST - http://localhost:4000/graphql
    
    {
        "query":"mutation addProduct($input: ProductInput) {addProduct(input: $input) {id}}",
        "variables":{"input":{"name":"세번째 상품", "price":3000, "description":"후후후"}}
    }
    ```

  * 응답

    ```
    데이터 작성 완료 후 응답,
    
    {
        "data": {
            "addProduct": {
                "id": "3"
            }
        }
    }
    ```

    

* DATA UPDATE/DELETE

  * 스키마 작성

    ```javascript
    const schema = buildSchema(`
    input ProductInput{
      name: String
      price: Int
      description: String
    }
    
    type Product {
      id: ID!
      name: String
      price: Int
      description: String
    }
    
    type Query{
      getProduct(id : ID!) : Product
    }
    
    
    ## Mutation - 데이터의 수정 시 사용하는 graphQL문
    type Mutation{
      addProduct( input: ProductInput) : Product
      
    	## 수정 스키마
    	updateProduct(id: ID!, input: ProductInput!): Product
    	
    	## 삭제 스미카
      deleteProduct(id: ID!): String
    }
    `);
    ```

  * 동작/응답 작성

    ```javascript
    
    const root = {
      getProduct: ({ id }) =>
        products.find((product) => product.id === parseInt(id)),
    
      addProduct: ({ input }) => {
        input.id = parseInt(products.length + 1);
        products.push(input);
        return root.getProduct({ id: input.id });
      },
    
      ## 수정
      updateProduct: ({ id, input }) => {
        const index = products.findIndex((product) => product.id === parseInt(id));
        products[index] = {
          id: parseInt(id),
          ...input,
          // ... - 확산연산자
        };
        return products[index];
      },
    
    	## 삭제  
      deleteProduct: ({ id }) => {
        const index = product.findIndex((product) => product.id === parse(id));
        products.splice(index, 1);
        return "remove success";
      },
    };
    ```

  * 실제 요청

    ```
    ## 수정
    POST - http://localhost:4000/graphQL
    
    {
        "query": "mutation updateProduct( $id : ID! , $input: ProductInput! ) { updateProduct( id : $id  , input: $input) { id } }",
        "variables": { "id" : 1 ,"input" : { "name" : "수정상품" , "price" : 1000 , "description" : "후후후" } }
    }
    
    ## 삭제
    POST - http://lcoalhost:4000/graphQL
    
    {
        "query": "mutation deleteProduct( $id : ID! ) { deleteProduct( id : $id  )  }",
        "variables": { "id" : 1  }
    }
    ```

    