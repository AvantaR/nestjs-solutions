# Issues that NestJS docs don't solve

## OpenAPI

### Hiding schema section

There is no special switch to turn off and turn on schema section module in Swagger implementation for NestJS. You can simply set `defaultModelsExpandDepth` param in `swaggerOptions`, while setting up SwaggerModule.

```TypeScript
const options = new DocumentBuilder()
  .setTitle('Docs without schema section')
  .build();
const document = SwaggerModule.createDocument(app, options);
SwaggerModule.setup('api', app, document, {
  swaggerOptions: { defaultModelsExpandDepth: -1 },
});
```

## Validation

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
