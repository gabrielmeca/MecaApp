# Segurança do MecaApp: como ativar

O app passa a exigir login (Firebase Authentication), e o banco só abre para quem tem acesso liberado.
Os formulários públicos (`chamado.html` e `meca-contratacao.html`) continuam funcionando sem login,
mas só conseguem **criar** um chamado na caixa `entrada/`. Eles não conseguem ler nada.

Siga os passos **nesta ordem**. Se publicar as regras (passo 5) antes de criar o seu acesso (passo 3),
ninguém consegue entrar.

## 1. Ligar o login por e-mail e senha

[console.firebase.google.com](https://console.firebase.google.com) → projeto **meca-laundry-solutions** →
**Authentication** → **Começar** → aba **Método de login** → **E-mail/senha** → ativar → **Salvar**.

## 2. Criar a sua conta (Master)

**Authentication** → aba **Usuários** → **Adicionar usuário**:

- E-mail: `gabriel@usuarios.mecaapp.com.br` (no app você entra só com `gabriel`)
- Senha: uma senha forte, com pelo menos 6 caracteres

Depois de criar, copie o **UID do usuário** (a coluna "UID", uma sequência de letras e números).

> O login do app vira `login@usuarios.mecaapp.com.br`. É só um identificador, nenhum e-mail é enviado.

## 3. Liberar o seu acesso no banco

**Realtime Database** → aba **Dados** → passe o mouse sobre a raiz → **+** e crie:

```
acesso
  └─ COLE_O_UID_AQUI
       ├─ name:   "Gabriel"
       ├─ login:  "gabriel"
       ├─ role:   "master"
       └─ active: true
```

Atenção: `active` precisa ser o valor booleano `true`, não o texto `"true"`.

## 4. Publicar o app (merge do PR) e migrar os usuários

1. Faça o merge do PR. Em 1 ou 2 minutos o site atualiza.
2. Abra o app e entre com `gabriel` e a senha do passo 2.
3. Na aba **Usuários**, clique em **Migrar usuários antigos**. Isso cria as contas com o mesmo login e senha
   de antes e apaga do banco as senhas antigas. Senhas com menos de 6 caracteres não podem ser migradas:
   crie esses usuários de novo com **Novo Usuário**.

## 5. Fechar o banco (regras)

**Realtime Database** → aba **Regras** → apague o que estiver lá → cole o conteúdo do arquivo
[`database.rules.json`](database.rules.json) → **Publicar**.

Faça isso logo depois do passo 4: até as regras serem publicadas, o banco continua aberto.

## 6. Conferir

- Abra o app numa janela anônima: só deve aparecer a tela de login.
- Envie um chamado de teste pelo `meca-contratacao.html` e veja se ele chega em **Preventivas → Chamados**.
- Envie um chamado de teste pelo `chamado.html` e veja se ele chega em **Agendamento**.

## Como funciona no dia a dia

- **Master** vê todas as abas e gerencia a aba **Usuários**: cria logins, escolhe as abas de cada pessoa
  e pode desativar ou remover o acesso, que é cortado na hora.
- Cada pessoa troca a própria senha em **Alterar Senha**. Se alguém esquecer a senha, remova o acesso
  dessa pessoa e crie um login novo (por exemplo, `ana2`).
- Quem está inativo, foi removido ou não tem login não consegue ler nem gravar nada no banco.
- A separação por aba é feita pelo app. As regras do banco bloqueiam quem está de fora, mas um usuário
  ativo tecnicamente consegue ler os dados de todas as abas.
- A antiga senha master fixa (`Meca@1010`) deixa de existir.
