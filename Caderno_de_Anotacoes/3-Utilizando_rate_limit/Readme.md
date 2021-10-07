# Utilizando rate limit

O `express-rate-limit` é uma lib q limita o número de acesso às rotas. Bem, mas o express-brute tb faz isso, né?! Faz mas faz diferente. O Brute é focado em login. Os bloqueios são a partir de 5 requests. O rate limit previne não ataque de invasão, mas de ataque de queda do servidor. Se um usuário faz 5 mil requests de determinada rota em poucos segundos, o servidor cai. O rate-limit já é pronto pra proteger todas as rotas.

> O express-rate-limit exige q tenha a lib `redis`. Por mais q estejamos já usando o redis aqui no app a um tempão, ainda não foi instalada essa lib. Instale-a.

Instale `express-rate-limit`, `rate-limit-redis` e `redis`:

`yarn add express-rate-limit redis rate-limit-redis`

Abaixo, vou configurar pro usuário poder acessar até 100 vezes em 15 minutos. Mas fique atento com isso. Pode ter dashboard fazendo 20 requests a cada entrada, daí o cara só poderia entrar 5 vezes. O ideal é geranciar bem as requests pra existir o mínimo possível de requests, e deixar sim o backend mais limitado.

## src/app.js

```diff
import 'dotenv/config';

import express from 'express';
import * as Sentry from '@sentry/node';
import path from 'path';
import cors from 'cors';
import helmet from 'helmet';
+import redis from 'redis';
+import RateLimit from 'express-rate-limit';
+import RateLimitRedis from 'rate-limit-redis';
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
    /**
     * Em produção, o cors fica assim:
     * this.server.use(cors({ origin: 'https://foobar.com.br' }));
     */
    this.server.use(cors());
    this.server.use(express.json());
    // Para o express aceitar acessar arquivos estáticos por url
    this.server.use(
      '/files',
      express.static(path.resolve(__dirname, '..', 'tmp', 'uploads'))
    );

+    if (process.env.NODE_ENV !== 'development') {
+      this.server.use(
+        new RateLimit({
+          store: new RateLimitRedis({
+            client: redis.createClient({
+              host: process.env.REDIS_HOST,
+              port: process.env.REDIS_PORT,
+            }),
+          }),
+          windowMs: 1000 * 60 * 15, // 15 minutos
+          max: 100, // máximo de 100 requests em 15 minutos
+        })
+      );
+    }
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

Nos headers diz a qnt limite de requests (`X-RateLimit-Limit`) e quantas ainda faltam (`X-RateLimit-Remaining`).
