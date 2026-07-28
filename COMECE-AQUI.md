# Do zero ao link no ar

Você não precisa instalar nada, nem abrir terminal, nem digitar comando nenhum. É tudo pelo navegador.

Reserve uns 25 minutos. Faça na ordem e não pule etapa.

---

## Antes de começar: o que é cada coisa

Só para você não ficar perdido no meio do caminho.

| | O que é | Para que serve aqui |
|---|---|---|
| **GitHub** | Um lugar onde se guardam arquivos de site | Guarda o `index.html` e transforma ele num site com endereço próprio |
| **Supabase** | Um banco de dados na nuvem | Guarda seus aportes, para que outras pessoas vejam os mesmos números que você |
| **index.html** | O painel inteiro dentro de um arquivo só | É o que você vai subir |

O fluxo é: **arquivo → GitHub → link no ar → conectar o Supabase → pronto**.

---

## Uma coisa importante sobre o Supabase

Você pode fazer só a Parte 1 e já ter um link funcionando. Mas preste atenção nisto:

**Sem o Supabase**, cada pessoa que abrir o link vê os dados de exemplo, e não os seus. Seus lançamentos ficam salvos só no seu próprio navegador. Serve para você usar, não para mostrar.

**Com o Supabase**, todo mundo que abrir o link vê os seus números de verdade — mas só você consegue alterar, porque para lançar aporte é preciso fazer login.

Como o seu objetivo é mandar o link para outras pessoas, faça as duas partes. A Parte 2 é mais fácil do que parece.

---

# PARTE 1 — Colocar o site no ar

### Passo 1. Baixe o arquivo

Baixe o `index.html` que está aqui nesta conversa e deixe ele em algum lugar fácil de achar, tipo a Área de Trabalho.

Não renomeie o arquivo. Ele precisa se chamar exatamente `index.html`, tudo minúsculo.

### Passo 2. Crie o repositório no GitHub

"Repositório" é só o nome bonito de pasta.

1. Entre em **github.com** já logado.
2. No canto superior direito, clique no **+** e escolha **New repository**.
3. Em *Repository name*, escreva: `painel`
4. Logo abaixo, deixe marcado **Public**.
   > Precisa ser público, senão o GitHub cobra para colocar no ar. Não tem problema: ninguém consegue mexer nos seus dados, você vai proteger isso na Parte 2.
5. **Não marque** nenhuma das caixinhas de "Initialize this repository".
6. Clique no botão verde **Create repository**.

### Passo 3. Suba o arquivo

Você vai cair numa página cheia de texto e comandos. **Ignore tudo isso.**

1. Procure a frase **uploading an existing file** — é um link azul no meio da página. Clique nele.
   > Se não achar, entre em `github.com/SEU-USUARIO/painel/upload` trocando SEU-USUARIO pelo seu nome de usuário.
2. Arraste o `index.html` da Área de Trabalho para dentro da área tracejada.
3. Desça a página e clique no botão verde **Commit changes**.

Pronto, o arquivo está guardado. Mas ainda não é um site.

### Passo 4. Ligue o site

1. Na página do repositório, clique em **Settings** (a engrenagem, na barra de cima).
2. No menu da esquerda, desça e clique em **Pages**.
3. Onde está escrito *Source*, escolha **Deploy from a branch**.
4. Logo abaixo, em *Branch*, mude de `None` para **main**. A pasta ao lado deixe em `/ (root)`.
5. Clique em **Save**.

### Passo 5. Pegue o seu link

Espere de **1 a 3 minutos** e recarregue a página (F5). Vai aparecer uma caixa verde no topo com o endereço:

```
https://SEU-USUARIO.github.io/painel/
```

**Esse é o link.** Abra para conferir. Se ainda der erro 404, espere mais dois minutos e recarregue — na primeira vez o GitHub demora um pouco.

Se o painel abriu com os cards dourados e os gráficos, a Parte 1 acabou. Respire.

---

# PARTE 2 — Ligar o Supabase

Agora os dados saem do seu navegador e vão para a nuvem.

### Passo 6. Crie o projeto

1. Entre em **supabase.com** e clique em **New project**.
2. Nome: `painel`. Crie uma senha para o banco e **anote num lugar seguro**.
3. Em *Region*, escolha **South America (São Paulo)**.
4. Clique em **Create new project** e espere terminar. Leva uns 2 minutos.

### Passo 7. Crie a tabela

1. No menu da esquerda, clique em **SQL Editor** e depois em **New query**.
2. Copie o bloco inteiro abaixo e cole na área branca:

```sql
create table if not exists public.carteira (
  id            uuid primary key,
  dados         jsonb not null default '{}'::jsonb,
  atualizado_em timestamptz not null default now()
);

alter table public.carteira enable row level security;

-- Qualquer visitante pode VER o painel
drop policy if exists "todos podem ver" on public.carteira;
create policy "todos podem ver" on public.carteira
  for select to anon, authenticated using (true);

-- Só quem fez login pode ALTERAR
drop policy if exists "dono pode criar" on public.carteira;
create policy "dono pode criar" on public.carteira
  for insert to authenticated with check (true);

drop policy if exists "dono pode atualizar" on public.carteira;
create policy "dono pode atualizar" on public.carteira
  for update to authenticated using (true) with check (true);

-- Cria a linha onde tudo será guardado
insert into public.carteira (id, dados)
values ('11111111-2222-3333-4444-555555555555', '{}'::jsonb)
on conflict (id) do nothing;
```

3. Clique em **Run** (ou Ctrl+Enter). Deve aparecer *Success*.

### Passo 8. Crie o seu usuário

É com ele que você vai fazer login no painel para lançar os aportes.

1. Menu da esquerda: **Authentication** → **Users**.
2. Botão **Add user** → **Create new user**.
3. Coloque seu e-mail e invente uma senha. **Marque a caixinha "Auto Confirm User"** — sem isso o login não funciona.
4. Clique em **Create user**.

### Passo 9. Copie as duas chaves

1. Menu da esquerda, lá embaixo: **Project Settings** → **API**.
2. Anote dois valores (pode colar num bloco de notas):
   - **Project URL** — parece com `https://abcdefgh.supabase.co`
   - A chave pública, chamada **anon public** ou **publishable key** — é um texto bem comprido começando com `eyJ...`

### Passo 10. Coloque as chaves no painel

Isso é feito direto no GitHub, sem baixar nada.

1. Volte para `github.com/SEU-USUARIO/painel`.
2. Clique no arquivo **index.html**.
3. No canto direito, clique no **ícone de lápis** (Edit this file).
4. Ali pelas primeiras 20 linhas você vai achar este trecho:

```js
const CONFIG = {
  SUPABASE_URL: "",
  SUPABASE_ANON_KEY: "",
  CARTEIRA_ID: "00000000-0000-0000-0000-000000000001",
};
```

5. Preencha assim, mantendo as aspas e as vírgulas:

```js
const CONFIG = {
  SUPABASE_URL: "https://abcdefgh.supabase.co",
  SUPABASE_ANON_KEY: "eyJhbGciOiJIUzI1NiIsInR5cCI6...",
  CARTEIRA_ID: "11111111-2222-3333-4444-555555555555",
};
```

> O `CARTEIRA_ID` tem que ser **exatamente igual** ao que está no SQL do Passo 7. Se você usou o do exemplo, é esse mesmo.

6. Botão verde **Commit changes** no canto superior direito, e **Commit changes** de novo na janelinha que abrir.

Espere 1 minuto e recarregue o seu link. Agora aparece "Somente leitura · Entrar" no topo.

### Passo 11. Faça o primeiro login

1. Abra o seu link e clique em **Entrar**.
2. Use o e-mail e a senha que você criou no Passo 8.
3. O botão vira **Novo lançamento**. Vá em **Ajustes → Limpar lançamentos** para apagar os exemplos e comece a registrar os seus de verdade.

Terminou. Manda o link para quem quiser.

---

# Como funciona no dia a dia

**Você**, depois de entrar com e-mail e senha: lança aportes, apaga, muda a taxa. Tudo salva sozinho na nuvem.

**As outras pessoas**, abrindo o link: veem os números, os gráficos, o extrato, tudo atualizado. Não conseguem alterar nada — os botões nem aparecem.

O login fica guardado no seu navegador, então você não precisa digitar a senha toda vez.

---

# Problemas comuns

**"404 — There isn't a GitHub Pages site here"**
Normal nos primeiros minutos. Espere 3 minutos e recarregue. Se persistir, confira no Settings → Pages se a branch está em `main` e a pasta em `/ (root)`.

**A página abre em branco**
Quase sempre é erro de digitação no Passo 10 — uma aspa faltando ou uma vírgula a mais. Volte no arquivo e compare com o exemplo. Se quiser, apague o bloco todo e cole de novo.

**Entrei, lancei um aporte, mas no celular não aparece**
Confira se você preencheu as chaves do Passo 10. Sem elas, cada aparelho guarda os dados separadamente.

**"E-mail ou senha não conferem"**
Volte no Supabase, Authentication → Users, e veja se o usuário está lá com status *Confirmed*. Se não estiver, apague e crie de novo marcando **Auto Confirm User**.

**Quero mudar alguma coisa no painel depois**
Sempre pelo mesmo caminho: GitHub → arquivo `index.html` → lápis → editar → Commit changes. O site atualiza sozinho em cerca de 1 minuto.

**Quero um endereço melhor, tipo meupainel.com.br**
Dá para fazer. Compre o domínio (registro.br, por exemplo) e me chame que eu te explico como apontar para o GitHub Pages.

---

# Sobre segurança, com franqueza

A chave que você colou no Passo 10 fica visível para quem abrir o site. Isso é normal e esperado — o Supabase foi feito assim. Quem protege os dados são as regras do Passo 7: elas dizem ao banco que visitante só pode ler, e que gravar exige login.

O que você **não** deve fazer é colocar no arquivo a senha do banco (a do Passo 6) ou a chave `service_role` do Supabase. Essas duas nunca saem do painel de administração.
