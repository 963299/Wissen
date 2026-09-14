# Wissen — configuração e deploy

## 1. Criar o projeto Firebase (separado do app do cliente)
1. Acesse https://console.firebase.google.com → **Adicionar projeto** → nome "Wissen" (ou o que preferir).
2. Dentro do projeto, vá em **Build → Realtime Database → Criar banco de dados**.
   - Escolha uma região (ex: `us-central1`).
   - Comece em modo de teste, depois ajuste as regras (veja seção 3).
3. Vá em **Build → AI Logic** (aparece no menu lateral) → **Vamos começar**.
   - Escolha **Gemini Developer API** (tem cota gratuita, mais simples de configurar; não precisa cartão pra começar).
   - Siga o assistente — ele já habilita a API certa pro projeto.
4. Vá em **Configurações do projeto** (ícone de engrenagem) → aba **Geral** → seção "Seus apps" → clique no ícone `</>` (Web) → registre um app (não precisa hospedar nada aqui, é só pra pegar as chaves).
5. Copie o objeto que aparece, algo como:
```json
{
  "apiKey": "AIza...",
  "authDomain": "wissen-xxxx.firebaseapp.com",
  "databaseURL": "https://wissen-xxxx-default-rtdb.firebaseio.com",
  "projectId": "wissen-xxxx",
  "storageBucket": "wissen-xxxx.appspot.com",
  "messagingSenderId": "...",
  "appId": "..."
}
```
6. Abra o app Wissen pela primeira vez (no navegador) e cole esse JSON na tela de configuração. Ele fica salvo no `localStorage` do dispositivo — cada dispositivo (PC e celular) precisa colar uma vez.

## 2. Estrutura de dados (Realtime Database)
```
students/
  {studentId}/
    name: "Nome do filho"
    decks/
      {deckId}/
        name: "Matemática — Frações"
        cards/
          {cardId}/
            front: "pergunta"
            back: "resposta"
            srs: { ease, interval, reps, dueDate, lastReview }
```

## 3. Regras do Realtime Database (uso pessoal/família)
Como é uso pessoal, a forma mais simples é liberar leitura/escrita sem autenticação — mas isso deixa o banco público pra quem tiver a URL. Recomendo pelo menos travar por um "segredo" simples. Em **Realtime Database → Regras**, use:
```json
{
  "rules": {
    ".read": true,
    ".write": true
  }
}
```
Isso já sincroniza PC ↔ celular como você pediu. Se quiser mais segurança depois (ex: Auth anônimo ou por e-mail), me avise que ajusto — não é necessário pra funcionar agora.

## 4. Deploy (mesmo esquema dos outros apps)
1. Suba os arquivos (`index.html`, `manifest.json`, `sw.js`, `icon-192.png`, `icon-512.png`, `icon-maskable-192.png`, `icon-maskable-512.png`, `apple-touch-icon.png`) para um repositório no GitHub (ex: `963299/wissen`).
2. Ative o GitHub Pages (branch `main`, pasta raiz).
3. Acesse a URL do Pages, cole o `firebaseConfig` na primeira tela.
4. Use o PWABuilder normalmente pra gerar o APK, apontando pra essa URL — o manifest já está com os ícones certos.
5. Empacote com o Hermit como de costume.

## 5. Como funciona o app
- **Adicionar estudante**: toque em "+" na fileira de avatares no topo — você pode adicionar quantos quiser.
- **Criar baralho por foto**: dentro de um estudante, toque em "+" → "Baralho a partir de foto" → tire a foto da página → a IA (Gemini) lê a imagem, gera um resumo e cards de pergunta/resposta → você revisa/edita/apaga antes de salvar.
- **Revisão**: algoritmo SM-2 (o mesmo princípio usado por trás do Anki) — cada card tem "ease" e "intervalo" que ajustam sozinhos conforme a nota dada (Errei / Difícil / Bom / Fácil).
- **Offline**: se não tiver internet, o app mostra os dados da última sincronização e guarda as revisões numa fila local. Quando a conexão volta, ele sincroniza sozinho com o Realtime Database — inclusive pegando o que foi atualizado no PC enquanto o celular estava offline.

## 6. Custos
- Realtime Database: gratuito até 1GB armazenado / 10GB de transferência por mês — mais que suficiente pra esse uso.
- Gemini Developer API (via Firebase AI Logic): tem cota gratuita diária generosa para o modelo `gemini-2.0-flash`. Se um dia passar disso, dá pra trocar pro plano pago só do Gemini, sem mudar nada no app.
