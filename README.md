# Issues that NestJS docs don't solve

## OpenAPI

### Hiding schema section

There is no special switch to turn off and turn on schema section module in OpenAPI implementation for NestJS. You can simply set `defaultModelsExpandDepth` param in `swaggerOptions`, while setting up SwaggerModule.

```TypeScript
const options = new DocumentBuilder()
  .setTitle('Docs without schema section')
  .build();
const document = SwaggerModule.createDocument(app, options);
SwaggerModule.setup('api', app, document, {
  swaggerOptions: { defaultModelsExpandDepth: -1 },
});
```

### Sorting tags and endpoints

OpenAPI allows handy grouping with tags. You just need to use `@ApiTags()` or `@ApiOperational({ tags: ['YourTag']})` decorators. If you wish to sort tags or endpoints alphabetically, you need to pass `tagsSorter` and `operationsSorter` as option in `swaggerOptions`.

```TypeScript
const options = new DocumentBuilder()
  .setTitle('Docs with sorted tags and endpoints')
  .build();
const document = SwaggerModule.createDocument(app, options);

SwaggerModule.setup('api', app, document, {
  swaggerOptions: { tagsSorter: 'alpha', operationsSorter: 'alpha' },
});
```

## Validation

### Validation of nested objects with class-validator

If you want to validate nested object in your DTO with class-validator, use `@ValidateNested()` decorator and specify typing with `@Type()` decorator.

```Typescript
export class Message {
  @ValidateNested()
  @Type(() => Author)
  author: Author;
}
```

If you need to validate array of objects, use `each: true` option for `@ValidateNested()` decorator.

```Typescript
export class User {
  @ValidateNested({ each: true })
  @Type(() => Message)
  messages: Message[];
}
```
You can also read more about it in [my article](https://dev.to/avantar/validating-nested-objects-with-class-validator-in-nestjs-1gn8).

### Dependency Injection in custom validation class

If you want to create your own validation class and inject into it your dependencies, you need to:

1. Create custom validation class as official class-validator documentation says. E.g.:

```Typescript
@ValidatorConstraint({ name: 'userName', async: true })
@Injectable()
export class UserExists implements ValidatorConstraintInterface {
  constructor(private usersRepository: UsersRepository) {}

  async validate(value: number, arg: ValidationArguments) {
    try {
      await this.userRepository.find({id});
    } catch (e) {
      return false;
    }

    return true;
  }

  defaultMessage(args: ValidationArguments) {
    return `User doesn't exist!`;
  }
}
```
2. Then you must add your validation class as provider in the module, where you want to use it. E.g.:
```Typescript
@Module({
  imports: [
    TypeOrmModule.forFeature([
      UsersRepository
    ]),
  ],
  controllers: [UsersController],
  providers: [UserExists],
})
export class UserModule {}
```
3. Last but not least, you must set Nest container for class-validator library in your main.ts file:

```Typescript
async function bootstrap() {
  const app = await NestFactory.create(AppModule);
  useContainer(app.select(AppModule), { fallbackOnErrors: true });

  await app.listen(3000);
}

```
Now you can use your services and repository in custom validation classes. I don't need to say how useful this is!
