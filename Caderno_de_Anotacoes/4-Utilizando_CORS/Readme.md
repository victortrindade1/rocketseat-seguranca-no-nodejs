# Utilizando CORS

O CORS protege a conexão entre frontend e backend.

O CORS protege apenas do frontend. Se vc quiser acessar uma API via Insomnia ou outra coisa, o CORS não vai funcionar pra este tipo de entrada.

## src/app.js

```diff
import 'dotenv/config';

import express from 'express';
import * as Sentry from '@sentry/node';
import path from 'path';
import cors from 'cors';
import helmet from 'helmet';
import redis from 'redis';
import RateLimit from 'express-rate-limit';
import RateLimitRedis from 'rate-limit-redis';
import Youch from 'youch';
import 'express-async-errors';

import sentryConfig from './config/sentry';
import routes from './routes';

import './database';

// O professor disse não gostar de usar "class" no frontend, mas no backend é
// muito bom de usar, usa bastante.
class App {
  constructor() {
    this.server = express();

    Sentry.init(sentryConfig);

    this.middlewares();
    this.routes();
    this.exceptionHandler();
  }

  middlewares() {
    this.server.use(Sentry.Handlers.requestHandler());
    this.server.use(helmet());

-    this.server.use(cors());
+    this.server.use(
+      cors({
+        origin: process.env.FRONT_URL,
+      })
+    );

    this.server.use(express.json());
    // Para o express aceitar acessar arquivos estáticos por url
    this.server.use(
      '/files',
      express.static(path.resolve(__dirname, '..', 'tmp', 'uploads'))
    );

    if (process.env.NODE_ENV !== 'development') {
      this.server.use(
        new RateLimit({
          store: new RateLimitRedis({
            client: redis.createClient({
              host: process.env.REDIS_HOST,
              port: process.env.REDIS_PORT,
            }),
          }),
          windowMs: 1000 * 60 * 15, // 15 minutos
          max: 100, // máximo de 100 requests em 15 minutos
        })
      );
    }
  }

  routes() {
    this.server.use(routes);
    this.server.use(Sentry.Handlers.errorHandler());
  }

  exceptionHandler() {
    this.server.use(async (err, req, res, next) => {
      if (process.env.NODE_ENV === 'development') {
        const errors = await new Youch(err, req).toJSON();

        return res.status(500).json(errors);
      }

      return res.status(500).json({ error: 'Internal server error' });
    });
  }
}

// Exporto apenas o server da classe, e não a classe toda, pois traz segurança
export default new App().server;
```

## .env

```diff
#APP_URL=http://localhost:3333
APP_URL=http://10.0.2.2:3333
+FRONT_URL=https://frontdodiego.com

NODE_ENV=development

# Auth
...
```
