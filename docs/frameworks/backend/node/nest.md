# [NestJS](https://docs.nestjs.com/)

## Introduction

Nest is a framework for building efficient, scalable Node.js server-side applications. It uses modern JavaScript, is built with TypeScript (preserves compatibility with pure JavaScript) and combines elements of OOP (Object Oriented Programming), FP (Functional Programming), and FRP (Functional Reactive Programming).

## Basics 

The basics of NestJS.

### [Controllers](https://docs.nestjs.com/controllers)

The controller layer is responsible for handling incoming **Requests** and returning a **response** to a client.

Creating a basic controller requires attaching **metadata** to a classusing **decorators**, for instance: `@Controller('someroute')`.

```typescript

import { Controller, Get } from '@nestjs/common';

@Controller('cats')
export class CatsController {
    @Get()
    findAll() {
        return [];
    }

    @Get('all')
    all() {
        return getAllCats();
    }
}
```

#### Metadata

The `@Controller('...')` decorator is **obligatory** for controllers to be defined. The optional prefix is to simplify the controller by not having to repeat it when the routes share a common prefix.

The response can be manipulated in one of two ways:

> **Standard**
> When a Javascript object or array is returned, it'll automatically be parsed to JSON. When a string is returned, Nest will just send the string without trying to parse it into JSON.
>
>Furthermore the status code is always 200 by default, except for POST requests which use 201. This behvaiour can be changed by adding the decorator `@HttpCode(...)` at the handler level.

> **Express**
> The express response object can be used by injecting the `@Res()` decorator in the function signature, such as `findAll(@Res() response)`.

#### Request Object

Many endpoints need access to the client **request** details. Next can easily inject the express request object into a handler using the `@Req()` decorator.

**It is strongly recommended to install the `@types/express` package and use it**

#### Decorators vs their Express equivalents

| Decorator | Express |
| --- | --- |
| `@Request()`  |   `req`   |
| `@Response()` |   `res`   |
| `@Next()`     |   `next`  |
| `@Session()`  |   `req.session` |
| `@Param(param?: string)`  | `req.params` / `req.params[param]` |
| `@Body(param?: string)` | `req.body` / `req.body[param]` |
| `@Quest(param?: string)` | `req.query` / `req.query[param]` |
| `@Headers(param?: string)` | `req.headers` / `req.headers[param]` |

#### HTTP Methods

- `@Post()`
- `@Get()`
- `@Put()`
- `@Delete()`
- `@Patch()`
- `@Options()`
- `@Head()`
- `@All()`

#### Status Codes

All responses use the status code 200 by default, except for POST requests when it's 201. This can be changed by using the `@HttpCode(...)` decorator at the handler level

```typescript
import { Controller, Get, Post, HttpCode } from '@nestjs/common';

@Controller('cats')
export class CatsController {
  @HttpCode(204)
  @Post()
  create() {
    // TODO: Add some logic here
  }

  @Get()
  findAll() {
    return [];
  }
}
```

#### Route Parameters

To define routes with parameters, simply specify the route parameters in the path of the route:

```typescript
@Get(':id')
findOne(@Param() params) {
    console.log(params.idf);
    return {};
}
```

#### Async / await

Nest supports `async` functions! Each async function has to return a `Promise`. This means that a function can return a deferred value and Nest will resolve it by itself.

```typescript
@Get()
async findAll(): Promise<any[]> {
    return [];
}
```

- More on [`async` / `await`](https://kamilmysliwiec.com/typescript-2-1-introduction-async-await)

##### Advanced: RxJS Obervable Streams

Nest route handlers are even more powerful and support RxJS [observable streams](http://reactivex.io/rxjs/class/es6/Observable.js~Observable.html).

```typescript
@Get()
findAll(): Observable<any[]> {
    return Observable.of([]);
}
```

#### Post Handler

POST request parameters can be retrieved using the `@Body()` decorator. First the **DTO (Data Transfer Object)**  schema needs to be defined. This object defines how the data will be sent over the network. This can be done using TypeScript interfaces, or by simple classes.

```typescript
export class CreateCatDto {
  readonly name: string;
  readonly age: number;
  readonly breed: string;
}
``` 

This schema can now be used inside the `CatsController`:

```typescript
@Post()
async create(@Body() createCatDto: CreateCatDto){
    ...
}
```


#### Final Configuration

Nest needs to be configured to use the custom controller. Controllers always belong to a module, that's why the `controllers` array is in the `@Module()` decorator.

```typescript
// app.module.ts

import { Module } from '@nestjs/common';
import { CatsController } from './cats/cats.controller';

@Module({
    controllers: [CatsController],
})
export class ApplicationModule {}
```


### Components

Almost everything is a component (Service, Repository, Factory, Helper ...) and they can be **injected** intro controllers or into other components through `constructor`.

Controllers should be designe to only handle HTTP requests and delegate the more complex tasks to the **components**. Components are plain TypeScript classes with a `@Component()` decorator.

#### Service Component

Suppose a service component like `cats.service.ts`:

```typescript
import { Component } from '@nestjs/common';
import { Cat } from './interfaces/cat.interface';

@Component()
export class CatsService {
  private readonly cats: Cat[] = [];

  create(cat: Cat) {
    this.cats.push(cat);
  }

  findAll(): Cat[] {
    return this.cats;
  }
}
```

`CatService` is a basic class with one property and two methods. However it has the `@Component()` decorator which attaches metadata, letting Nest know that this class is a Nest component.

Then given the appropriate controller (like `cats.controller.ts`):

```typescript
import { Controller, Get, Post, Body } from '@nestjs/common';
import { CreateCatDto } from './dto/create-cat.dto';
import { CatsService } from './cats.service';
import { Cat } from './interfaces/cat.interface';

@Controller('cats')
export class CatsController {
  constructor(private readonly catsService: CatsService) {}

  @Post()
  async create(@Body() createCatDto: CreateCatDto) {
    this.catsService.create(createCatDto);
  }

  @Get()
  async findAll(): Promise<Cat[]> {
    return this.catsService.findAll();
  }
}
```

#### Dependency Injection

It's simple to manage dependencies with TypeScript because Nest will resolve dependencies just by **injecting** them into a controllers constructor.

```typescript
constructor(private readonly catsSergice: CatsService) {}
```

### Configuration

Last the module needs to be linked to the custom service, this is done by editing the module file and putting the sergice into hte `components` array of the `@Module()` decorator.

```typescript
// app.module.ts

import { Module } from '@nestjs/common';
import { CatsController } from './cats/cats.controller';
import { CatsService } from './cats/cats.service';

@Module({
    controllers: [CatsController],
    components: [CatsService],
})
export class ApplicationModule {}
```


### [Exception Handling](https://docs.nestjs.com/exception-filters)

## Fundamentals

### [Dependency Injection](https://docs.nestjs.com/fundamentals/dependency-injection)

