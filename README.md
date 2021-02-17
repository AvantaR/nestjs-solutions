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
