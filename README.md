<p align="center">
  <img src="https://i.imgur.com/oUAKMC5.png" />
  <h2 align="center">API to rent cars 🚗</h2>
</p>

## ❓ Why?
This project was created during the Rocketseat Bootcamp. Therefore, it’s for my portfolio, so, I really appreciate any feedback that you can give me about the project, code, architecture, design pattern, or anything else that you could report because this makes me a better developer. To help me with that, you can email me: [joaovictorramalho7@gmail.com](mailto:joaovictorramalho7@gmail.com), or connect with me on [LinkedIn](https://www.linkedin.com/in/404jv/), or even open an issue [here](https://github.com/404jv/rentx/issues/new).

## 🔧 Requirements
The functions requirements and nonfunctional requirements are on this [file](func.md)

## 📃 Notes
Some of my answers to questions are on this [file](caderno.md).

## 🎖 Extra mile
Extra mile are all the work that I did by myself without a Bootcamp class to explain it.

- [X] Add [ioredis](https://github.com/luin/ioredis) to cache data.
- [X] Delete images of a car.
- [X] Fix error when create a new car with a non-existent category.

## ⚖ Rest
The goal of the rest is basic improve some details in a web service. There are a lot of benefits that Rest gives to us, for example, performance and reliability. Performance is one of the factors that make the users use an APP, so, the higher the speed the better, and the reliability is important to the service, since, other applications will consume the API, clear communications between them must happen.

Now, about this API. It is separated from the client, the API is stateless. Therefore, every request is different from each other, for example, the route `cars/images` needs a bearer token to authenticate, and so if a user does two requests to this route, both requests must have the user’s token. 

The Uniform Interface is applied here, in all the routes, messages, and resources, I tried to make it clear possible. So, when the token was not sent in a request, the error is `“Token missing”` and the status code is 401, also, some routes are self-explanatory, for example, the route to upload an avatar is `users/avatar`, and the methods HTTP is used to describe the communication as well. The route to list categories (“categories/”) is a GET because I just want to retrieve data about the resource (categories) and the route to create a category is a POST. 

I’m sure there are some details that Rest has and this API does not follow or even more, some break of the Rest’s rules. That’s because I’m not familiar with it, and my ignorance around this topic doesn’t let me correct it. Of course, I always will correct the mistakes, so, if you see some let me know it (you can open an issue [here](https://github.com/404jv/rentx/issues/new)) 😉.

## 🔨 Architecture
First of all, I used Clean Architecture, to learn some architecture that is used in the real world, so I don’t know a lot of architecture yet. However, I will try to explain why is Clean Architecture in this project.

1. The code is more testable than other architectures like MVC (Model - View - Controller).

2. The project is separated into layers that have one purpose. Consequently, it’s easy to navigate between them, and if I have something to do with the database, I know that I have to go to the Data Layer, or if I have to fix some business rule, I go to the Use Case layer, and so on…

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
    ├── useCases
    │   ├── CreateCategory                        # Use Case
    │   │   ├── CreateCategoryController.spec.ts  # Integration test
    │   │   ├── CreateCategoryController.ts
    │   │   ├── CreateCategoryUseCase.spec.ts     # Unit test
    │   │   └── CreateCategoryUseCase.ts
    │   └── ... 
    └── ...

I think this is better because when I see them I know that those tests are for `CreateCategory`, and If I want to search for a specific test I know that it’s in the same folder as its use case. Furthermore, other details that do not follow exactly the Clean Architecture, but it’s fine because architecture like this is created for a lot of different scenarios, and for some projects, it needs to adapt some things.


## 🌵 Cache
I used Redis to set a rate limiter, and also to cache data. Some of the routes like the session route set users in Redis: ([AuthenticateUserUseCase](src/modules/accounts/useCases/authenticateUser/AuthenticateUserUseCase.ts))
```ts
await setRedis(`user-${user.id}`, JSON.stringify(user));
```

And others routes use this data to improve the velocity since a connection to the database is really costly. In the route to get user profile is used Redis to get this information: ([ProfileUseCase](src/modules/accounts/useCases/profileUser/ProfileUserUseCase.ts))
```ts
...
class ProfileUserUseCase {
  constructor(
    @inject("UsersRepository")
    private usersRepository: IUsersRepository
  ) {}

  async execute(id: string): Promise<IUserResponseDTO> {
    let user = await this.findUserInCache(id);

    if (!user) {
      user = await this.usersRepository.findById(id);
    }

    return UserMapper.toDTO(user);
  }

  async findUserInCache(id: string): Promise<User> {
    const userRedis = await getRedis(`user-${id}`);

    return JSON.parse(userRedis as string) as User;
  }
}
...
```
In this use case, it’s trying to find the user in the cache and if it does not exist, try to find the user on the database. The difference is 30.128ms (database) VS 3.65ms (Redis), it’s a huge difference and in a production product, this improves the system a lot:

<img src="public/cache.gif" />

However, in some of the use cases are not very recommended to apply cache because the data changes very much and that’s a problem since the copy in cache is going to be out of date soon, and if I store the value but never read it from the cache, then the cache is not helping at all, actually, it’s a problem though because Redis consumes hardware from the server.

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

Copy the file `ormconfig.example.json`
```bash
$ cp ormconfig.example.json ormconfig.json
```

**Open the file `ormconfig.json` and change the config for your database**
```json
{
  "type": "postgres",
  "host": "localhost",
  "port": 5432,
  "username": "docker",
  "password": "ignite",
  ...
}
```
**Open the file `docker-compose.yml` and change the config as well**
```yml
environment:
  - POSTGRES_USER=docker
  - POSTGRES_PASSWORD=ignite
  - POSTGRES_DB=rentx
```

Copy the file `.env.example`
```bash
$ cp .env.example .env
```

Open the `.env`, and set the config
```
FORGOT_MAIL_URL=

## AWS credentials
AWS_ACCESS_KEY_ID=
AWS_SECRET_ACCESS_KEY=
AWS_BUCKET=
AWS_BUCKET_REGION=
AWS_BUCKET_URL=

APP_API_URL=

DISK=local
```

**Now run docker**
```bash
$ docker-compose up
```

**Finally, run the migrations**
```
$ yarn typeorm migration:run
```

## 🔷 Database Diagram
<img src="public/diagram.png" />
