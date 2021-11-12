# 🚗 Rentx
API Restful to rent cars


## ❓ Why?
This project was created during the Rocketseat Bootcamp. Therefore, it’s for my personal portfolio, so, I really appreciate any feedback that you can give me about the project, code, architecture, design pattern, or anything else that you could report because this makes me a better developer. To help me with that, you can email me: [joaovictorramalho7@gmail.com](mailto:joaovictorramalho7@gmail.com), or connect with me on [LinkedIn](https://www.linkedin.com/in/404jv/), or even open an issue [here](https://github.com/404jv/rentx/issues/new).

## 👀 Observation
At the moment that I write this project, my current computer can’t run Docker (it’s not that bad, tho). Therefore, I won't be able to use Docker like the project developed in the Bootcamp but I still learning Docker and how it helps us.

## 🔧 Requirements
The functions requirements and nonfunctional requirements are on this [file](func.md)

## 📃 Notes
Some of my answers to questions are on this [file](caderno.md).

## 🎖 Extra mile
- [X] Delete images of a car.
- [ ] Fix error when create a new car with a non-existent category.

<!-- ## ⚖ Restful -->

## 🔨 Architecture
First of all, I used Clean Architecture, to learn some architecture that is used in the real world, so I don’t know a lot of architecture yet. However, I will try to explain why is Clean Architecture in this project.

1. The code is more testable than other architectures like MVC (Model - View - Controller).

2. The project is separated into layers that have one purpose. Therefore, it’s easy to navigate between them, and if I have something to do with the database, I know that I have to go to the Data Layer, or if I have to fix some business rule, I go to the Use Case layer, and so on…

3. When the project is already structured is easy to implement a new feature, since everything is separated, we don’t have to worry if we are going to break something that has nothing to do with the new feature. Of course, it’s not impossible to do that but it’s not frequent as other architectures.

4. If the project needs to change some dependencies, the process to do that is going to be easier, since the layers don’t affect each other. For example, in this project, I’m using [dayjs](src/shared/container/providers/DateProvider/implementations/DayjsDateProvider.ts) to compare dates in days, hours, and so on… If the project needs to change this lib to [date-fns](https://github.com/date-fns/date-fns), what I have to do is create a new class called `DatefnsDateProvider` that implements the interface [IDateProvider](src/shared/container/providers/DateProvider/IDateProvider.ts) that have all methods needed for the project, I implement those methods using date-fns, I register a singleton (in this [file](src/shared/container/providers/index.ts)) for the `DatefnsDateProvider` like this: 

```ts
...
container.registerSingleton<IDateProvider>(
  "DayjsDateProvider",
  DayjsDateProvider
);

container.registerSingleton<IDateProvider>(
  "DatefnsDateProvider",
  DatefnsDateProvider
);
...
```

And then start to immigrate all the injections of DayjsDateProvider to `DatefnsDateProvider`. For example, in the CreateRentalUseCase:

```typescript
...
@injectable()
class CreateRentalUseCase {
  constructor(
    @inject("RentalsRepository")
    private rentalsRepository: IRentalsRepository,
    @inject("DayjsDateProvider") // Here would be like @inject("DatefnsDateProvider")
    private dateProvider: IDateProvider,
...
```
it must be emphasized that this project does not follow every single detail of the Clean Architecture, so, some files are in different places. For example, the unity tests and integration tests are together on their use case:

    .
    ├── useCases...
    │   ├── CreateCategory                        # Use Case
    │   │   ├── CreateCategoryController.spec.ts  # Integration test
    │   │   ├── CreateCategoryController.ts
    │   │   ├── CreateCategoryUseCase.spec.ts     # Unit test
    │   │   └── CreateCategoryUseCase.ts
    │   └── ... 
    └── ...

I think this is better because when I see them I know that those tests are for `CreateCategory`, and If I want to search for a specific test I know that it’s in the same folder as its use case. Furthermore, there are other details that do not follow exactly the Clean Architecture, but it’s fine because architecture like this is created to a lot of different scenarios and for some projects, it need to adapt some things.

## 🚀 Run project

**Clone Repository**
```bash
$ git clone https://github.com/404jv/rentx
```

**Enter directory**
```bash
$ cd rentx
```

**Install dependencies, if you use npm**
```bash
$ npm install
```

<p align="center">or<p>

**Install dependencies, if you use yarn**
```bash
$ yarn
```

**Open `psql` and create the database**
```sql
$ CREATE DATABASE rentx;
```

**Open the file `ormconfig.json` and change the config for your database**
```json
{
  "type": "postgres",
  "host": "localhost",
  "port": 5432,
  "username": "postgres",
  "password": "123",
  ...
}
```

**Then, open your bash/terminal again and run the migrations**
```bash
# if you use yarn
$ yarn typeorm migration:run
```
<p align="center">or<p>

```bash
# if you use npm
$ npm run typeorm migration:run
```

**Run project, if you use yarn**
```bash
$ yarn dev
```

<p align="center">or<p>

**Run project, if you use npm**
```bash
$ npm run dev
```

## 🔷 Database Diagram
<img src="public/diagram.png" />
