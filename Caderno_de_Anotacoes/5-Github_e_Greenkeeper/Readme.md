# Github e Greenkeeper

O Github fornece uns mecanismos de segurança ao seu código pra qnd libs q vc possui como dependência te deixam vulnerável. O github escaneia seu código, monitora, e te dá o alerta. Tb fixa as vulnerabilidades sozinho, só q vc tem q autorizar a `pull request` pra injetar no código original a alteração.

Pra cada repositório, vc precisa configurar separado a segurança do github.

No repositório, vá em `Settings` > `Security and analysis`.

Habilite `Dependabot alerts`, `Dependabot security updates`. Quando vc for alertado de uma vulnerabilidade, vá em `Pull Requests` e veja se lá existe a solução pronta pra vc commitar no código. Todo update automático de vulnerabilidade vc tem q aceitar antes no pull requests pra alterar o original.

> Existe tb o `Code Scaning` q é bem legal mas não existia no github na época do curso. No Code Scaning, vc tem disponível umas 20 diferentes ferramentas disponíveis pra monitorar seu código, e é bem fácil de configurar.

## Greenkeeper

O objetivo do `greenkeeper` é atualizar sozinho as versões das libs pra versões mais estáveis e protegidas. O professor não chegou a instalar, pq pra usar o Greenkeeper, vc precisa estar usando alguma estratégia de CI (integração contínua), como por exemplo o buddy, codesheep, travis CI, circle CI, q faça o teste da aplicação. O greenkeeper atualiza as libs qnd os testes com as novas libs é aprovado. O código atualizado pelo greenkeeper, assim como faz o dependabot do github, fica disponível no `Pull Request` do repositório, esperando sua autorização pra entrar no código.
