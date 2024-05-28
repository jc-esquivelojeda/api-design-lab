
# Laboratorio de diseño de API's 


## Entregable archivo yaml con la especificacion openapi

```
openapi: 3.0.2
info:
  title: Ecommerce API
  description: Ecommerce API for API design workshop
  version: 1.0.0
servers:
  - url: https://api.ecommerce.com/v1
    description: This is the ecommerce api v1 for production server
  - url: https://dev.ecommerce.com/v1
    description: This is the ecommerce api v1 for Development server
tags:
  - name: Account Management
    description: This endpoints manages the user accounts creation,cancellation, login, logout and user detail retrieval
  - name: Order Processing
    description: This endpoints contains orders processing operations like order creation, update and cancellation, order details status and historic of orders by account
  - name: Products Management
    description: Products Management contains the endpoints for searching products and querying details related information
paths:
  /users:
    post:
      summary: Allows to create a user account
      description: This endpoint allow to create a new user account
      tags:
        - Account Management
      requestBody:
        required: true
        content:
          application/json:
            schema:
              type: object
              properties:
                name:
                  type: string
                  description: contains the nickname or the name of the user to be created
                email:
                  type: string
                  description: contains the email where notifications or password reset will be sent
                password:
                  type: string
                  description: contains the password with which the account will be created
                confirmation:
                  type: string
                  description: identifies that the previous password match correctly
              required:
                - name
                - email
                - password
                - confirmation
      responses:
        '201':
          description: Created - User account created successfully
          content:
            application/json:
              schema:
                $ref: '#/components/schemas/UserResponse'
        '400':
          description: Bad Request - Password and confirmation password do not match.
        '409':
          description: Conflict - A user with the given email already exists
        '422':
          description: Unprocessable content - The password doesnt comply with security criteria
        '500':
          description: Internal server error - Could not create user account due to server error. Please try again.
        '503':
          description: Service Unavailable - The server is currently unable to handle the request due to a temporary overload or maintenance of the server

  /users/login:
    post:
      summary: Allows to login User into ecommerce
      description: This endpoint allows to login user with email and password
      tags:
        - Account Management
      requestBody:
        required: true
        content:
          application/json:
            schema:
              type: object
              properties:
                email:
                  type: string
                  description: defines the email of the user that created the account
                password:
                  type: string
                  description: contains the password defined in the account creation
              required:
                - user
                - password
      responses:
        '200':
          description: OK - Successful login
          content:
            application/json:
              schema:
                $ref: '#/components/schemas/LoginResponse'
        '404':
          description: Not found -  invalid credentials
        '500':
          description: Internal server error - Could not login due to server error. Please try again.
        '503':
          description: Service Unavailable - The server is currently unable to handle the request due to a temporary overload or maintenance of the server

  /users/{user_id}:
    get:
      summary: Allows to retrieve account profile details
      description: This endpoint allow to retrieve the user profile detail
      tags:
        - Account Management
      parameters:
        - name: user_id
          in: path
          required: true
          description: The ID of the user to retrieve the profile
          schema:
            type: string
      responses:
        '200':
          description: Successful retrieved user
          content:
            application/json:
              schema:
                $ref: '#/components/schemas/UserResponse'
        '404':
          description: Not found -  user not found
        '500':
          description: Internal server error - Could not retrieve user due to server error. Please try again.
        '503':
          description: Service Unavailable - The server is currently unable to handle the request due to a temporary overload or maintenance of the server
    delete:
      summary: Allows to cancel a user account
      description: This endpoint cancel/deactivate the user account
      tags:
        - Account Management
      parameters:
        - name: user_id
          in: path
          required: true
          description: The ID of the user account to cancel
          schema:
            type: string
      responses:
        '204':
          description: Successful canceled account
        '404':
          description: Not found -  user not found
        '500':
          description: Internal server error - Could not retrieve user due to server error. Please try again.
        '503':
          description: Service Unavailable - The server is currently unable to handle the request due to a temporary overload or maintenance of the server
  /users/logout:
    get:
      summary: Allows to logout user from ecommerce
      description: This endpoint logout the user account indentifying user by the token param passed and cancels the tokens
      tags:
        - Account Management

      responses:
        '204':
          description: Successful logged out
        '404':
          description: Not found -  user not found
        '500':
          description: Internal server error - Could not retrieve user due to server error. Please try again.
        '503':
          description: Service Unavailable - The server is currently unable to handle the request due to a temporary overload or maintenance of the server

  /orders:
    get:
      summary: Allows to query a orders list page.
      description: Allows to query a orders list page by page and page size
      tags:
        - Order Processing
      parameters:
        - name: page
          in: query
          description: Page number
          required: false
          schema:
            type: integer
            default: 1
        - name: size
          in: query
          description: Number of records per page
          required: false
          schema:
            type: integer
            default: 10
      responses:
        '200':
          description: return a json array of orders paginated by page and number of records
          content:
            application/json:
              schema:
                type: array
                items:
                  $ref: "#/components/schemas/Order"
        '404':
          description: Not found -  Order not found
        '500':
          description: Internal server error - Could not retrieve order due to server error. Please try again.
        '503':
          description: Service Unavailable - The server is currently unable to handle the request due to a temporary overload or maintenance of the server

    post:
      summary:  Allows to register an order  of purchase
      description: Create a new purchase order to register required products
      tags:
        - Order Processing
      requestBody:
        $ref: "#/components/requestBodies/Order"
      responses:
        '200':
          description: Order created successfully
          content:
            application/json:
              schema:
                $ref: '#/components/schemas/Order'
        '400':
          description: Bad Request -  Order contains invalid product or inventory amount
        '409':
          description: Conflict -  Insufficient product inventory
        '500':
          description: Internal server error - Could not process the order due to server error. Please try again.
        '503':
          description: Service Unavailable - The server is currently unable to handle the request due to a temporary overload or maintenance of the server
  /orders/{order_id}:
    get:
      summary: Allows to retrieve an order detail by order id
      description: Retrieves the order detail by order id
      tags:
        - Order Processing
      parameters:
        - name: order_id
          in: path
          required: true
          description: The ID of the order to retrieve
          schema:
            type: string
      responses:
        '200':
          description: Successful retrieved order
          content:
            application/json:
              schema:
                $ref: '#/components/schemas/Order'
        '404':
          description: Not found - Order not found
        '500':
          description: Internal server error - Could not obtain the order due to server error. Please try again.
        '503':
          description: Service Unavailable - The server is currently unable to handle the request due to a temporary overload or maintenance of the server
    put:
      summary: Allows to modify existing order by order id
      description: Allow to update products and quantity of an order
      tags:
        - Order Processing
      parameters:
        - name: order_id
          in: path
          required: true
          description: The ID of the order to update
          schema:
            type: string
      requestBody:
        $ref: "#/components/requestBodies/Order"
      responses:
        '200':
          description: Successful retrieved user
          content:
            application/json:
              schema:
                $ref: '#/components/schemas/Order'
        '404':
          description: Not found -  Order to update not found
        '500':
          description: Internal server error - Could not obtain the order due to server error. Please try again.
        '503':
          description: Service Unavailable - The server is currently unable to handle the request due to a temporary overload or maintenance of the server
    delete:
      summary: Allows to cancel an existing order
      description: Cancel an order by order id when not already delivered
      tags:
        - Order Processing
      parameters:
        - name: order_id
          in: path
          required: true
          description: The ID of the order to retrieve
          schema:
            type: string
      responses:
        '200':
          description: Successful cancelled order
          content:
            application/json:
              schema:
                $ref: '#/components/schemas/Order'
        '404':
          description: Not found -  Order not found
        '409':
          description: Conflict -  could not cancel a delivered order
        '500':
          description: Internal server error - Could not cancel the order due to server error. Please try again.
        '503':
          description: Service Unavailable - The server is currently unable to handle the request due to a temporary overload or maintenance of the server

  /orders/history/{user_id}:
    get:
      summary: Retrieve the history of user orders  by user_id
      description: Retrieve the history of user orders  by user_id
      tags:
        - Order Processing
      parameters:
        - name: user_id
          in: path
          required: true
          description: The ID of the user to retrieve the orders
          schema:
            type: string
      responses:
        '200':
          description: Successful retrieved orders by user_id
          content:
            application/json:
              schema:
                type: array
                items:
                  $ref: '#/components/schemas/Order'
        '404':
          description: Not found - No orders found
        '409':
          description: Conflict -  user linked to orders not found
        '500':
          description: Internal server error - Could not search orders due to server error. Please try again.
        '503':
          description: Service Unavailable - The server is currently unable to handle the request due to a temporary overload or maintenance of the server

  /products:
    get:
      summary: Allows to retrieve a page of available products
      description: Allows to retrieve a page of available products using page and size of page, useful for displaying the products in the ecommerce site
      tags:
        - Products Management
      parameters:
        - name: page
          in: query
          description: Page number
          required: false
          schema:
            type: integer
            default: 1
        - name: size
          in: query
          description: Number of records per page
          required: false
          schema:
            type: integer
            default: 10
      responses:
        '200':
          description: A JSON array of products
          content:
            application/json:
              schema:
                type: array
                items:
                  $ref: "#/components/schemas/Product"
        '404':
          description: Not found -  Products not found
        '500':
          description: Internal server error - Could not retrieve the products due to server error. Please try again.
        '503':
          description: Service Unavailable - The server is currently unable to handle the request due to a temporary overload or maintenance of the server

  /products/{product_id}:
    get:
      summary: Allows to retrieve the details of a product by id
      description: Get the details of a product by id including  available inventory
      tags:
        - Products Management
      parameters:
        - name: product_id
          in: path
          required: true
          description: The ID of the product to retrieve
          schema:
            type: string
      responses:
        '200':
          description: Successful retrieved product
          content:
            application/json:
              schema:
                $ref: '#/components/schemas/Product'
        '404':
          description: Not found -  Products not found
        '500':
          description: Internal server error - Could not retrieve the product due to server error. Please try again.
        '503':
          description: Service Unavailable - The server is currently unable to handle the request due to a temporary overload or maintenance of the server
  /products/search:
    get:
      summary: Allows to find a product by search term query param
      description: Find a product by search term query param
      tags:
        - Products Management
      parameters:
        - name: search_term
          in: query
          description: search term for product
          required: true
          schema:
            type: string
            default: ""
      responses:
        '200':
          description: A JSON array of products
          content:
            application/json:
              schema:
                type: array
                items:
                  $ref: "#/components/schemas/Product"
        '404':
          description: Not found -  Products not found
        '500':
          description: Internal server error - Could not retrieve the products due to server error. Please try again.
        '503':
          description: Service Unavailable - The server is currently unable to handle the request due to a temporary overload or maintenance of the server


components:
  requestBodies:
    Order:
      content:
        application/json:
          schema:
            $ref: '#/components/schemas/Order'
      description: An order object that needs to be added to the store
      required: true

  schemas:
    User:
      type: object
      description: Contains the entity with the data of the user logged in the ecommerce site
      properties:
        id:
          type: string
          description: is the identifier of the user
        name:
          type: string
          description: contains the username of the user created
        email:
          type: string
          description: defines the email of the registered user
        password:
          type: string
          description: contains the encrypted password of the user
    Order:
      type: object
      description: defines the details of the order generated by the user
      properties:
        id:
          type: string
          description: is the identifier of the created order
        products:
          type: array
          description: contains a list of the ordered products in shopping cart
          items:
            $ref: "#/components/schemas/Product"
        total:
          type: number
          description: defines the total amount of the order
        user:
          $ref: "#/components/schemas/User"
        date:
          type: string
          description: contains the date on which the order was placed
        status:
          type: integer
          description: defines the status of the order
          required:
            - name
            - products
            - total
            - user

    Product:
      type: object
      description: Represents the entity of the products in the ecommerce catalog
      properties:
        id:
          type: string
          description: is the identifier of the product
        name:
          type: string
          description: describes the name of the product in catalog
        detail:
          type: string
          description: this field contain the description of the product
        price:
          type: number
          description: is the price at which the product will be sold
    UserResponse:
      type: object
      description: Represents the object of user response
      properties:
        id:
          type: string
          description: is the identifier of the user
        name:
          type: string
          description: is the name of which user was registered
        email:
          type: string
          description: contains the email of the user to login
    LoginResponse:
      type: object
      description: Represents the object returned of the login process
      properties:
        access_token:
          type: string
          description: contains de jwt access token returned from the login process
        refresh_token:
          type: string
          description: contains de jwt refresh token returned from the login process
```

## Imagen de api en editor swagger:

![ecommerce-api](https://github.com/jc-esquivelojeda/api-design-lab/assets/123103329/51fcd12b-c10d-4d88-bc73-ca1002e632de)

## Reporte
### Retos afrontados:
- Definicion de los endpoints sin tener claro el diseño o los mockups basicos para el flujo de trabajo
- Seguimiento al flujo de operacion para el ecommerce sin una guia definida
- Establecer los posibles escenarios de error que pudiera presentarse al consumir los endpoints
  
### Como se aplico el principio de api primero para diseñar un api robusta
- se aplicaron nombres descriptivos para los endpoints tomando en cuenta la
- se aplico versionamiento del API nombrando la version 1.0.0, adicionalmente se diferencio entre los endpoint para produccion de desarrollo
- Se manejo un nombrado consistente para los endpoints en base al endpoint base
- Se utilizaron los metodos http para indicar la posible accion a realizar segun el endpoint, Ej: DELETE para borrar/deactivar, PUT para actualizar, etc...

### Conocimientos adquiridos del laboratorio

- Al aplicar un estandar comun para el principio de api first permite tener claridad sobre que esperar en la documentación de un api
- Tambien mejora la colaboración ya que al seguir un estandar al cual fijarse asegura que se cumplan los criterios establecidos
- El uso de este principio permite mejorar  la comunicacion entre el equipo de back y front para establecer los requisitos del API necesarios para ambos lados
- El hecho de pensar por adelantado en los errores que se podrian presentar permite ir diseñando un API mas flexible
