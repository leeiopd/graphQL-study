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

      

    