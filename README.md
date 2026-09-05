# Dashboard Finanças Pessoais

Dashboard interativo, em um único arquivo HTML, para controle mensal de despesas e receitas pessoais — quanto foi pago, quanto está a pagar/receber, separado por categoria e por mês. Roda inteiramente no navegador: sem backend, sem build, sem dependências para instalar.

## Como usar

**Opção 1 — abrir direto:**
Baixe `index.html` e abra no navegador. Pronto.

**Opção 2 — publicar no GitHub Pages:**
1. Suba este repositório no GitHub.
2. Em *Settings → Pages*, selecione a branch principal e a pasta raiz (`/`).
3. O GitHub Pages vai servir `index.html` automaticamente na URL do projeto.

**Opção 3 — publicar no Vercel:** veja o passo a passo completo na seção [Publicando no Vercel](#publicando-no-vercel), mais abaixo.

Não há nenhuma etapa de build — é HTML/CSS/JS puro em um arquivo só.

## Funcionalidades

- **Lançamento de despesas e receitas**: categoria (com sugestão automática), valor, mês, status (Pago/A pagar ou Recebido/A receber) e recorrência (repetir em todos os meses ou em um número de parcelas, até 12x).
- **Categorias de receita sugeridas**: Adiantamento quinzenal, Pagamento, 13ª 1ª parcela, 13ª 2ª parcela, Férias, PLR — além de aceitar qualquer categoria digitada livremente.
- **Filtros de período**: por ano (2026–2030), por mês individual, por trimestre (Q1–Q4), por semestre (H1/H2) ou o ano fiscal completo (FY).
- **Cards de resumo**: total de Despesas, total de Receitas e Saldo (Receitas − Despesas) do período selecionado.
- **Tabelas mensais** (categoria × mês) para despesas e para receitas, com células coloridas por status (pago/a pagar/parcial).
- **Tabela de lançamentos**: busca por categoria, filtro por tipo, edição (✎) e exclusão (✕) de qualquer lançamento.
- **Limpar dados**: apaga todo o histórico salvo, com confirmação.

## Armazenamento dos dados

Por padrão, os dados ficam salvos **somente no navegador do usuário**, via `localStorage` — sem servidor, sem conta, sem sincronização entre dispositivos.

O app também tem suporte **opcional** a sincronização em nuvem via Firebase (Firestore + login por e-mail/senha). Quando configurado, os dados passam a ficar disponíveis em qualquer navegador onde você entrar com a mesma conta — computador, celular, tablet — inclusive funcionando offline (o Firestore enfileira as mudanças localmente e sincroniza sozinho quando a conexão volta).

### Ativando a sincronização (opcional)

1. Acesse [console.firebase.google.com](https://console.firebase.google.com) e crie um projeto novo (gratuito).
2. No menu lateral, vá em **Build → Authentication → Get started**, aba **Sign-in method**, e habilite o provedor **E-mail/senha**.
3. Vá em **Build → Firestore Database → Create database**. Pode começar em modo de teste; depois, nas *Rules*, restrinja o acesso para que cada usuário só leia/escreva seus próprios dados:
   ```
   rules_version = '2';
   service cloud.firestore {
     match /databases/{database}/documents {
       match /usuarios/{userId} {
         allow read, write: if request.auth != null && request.auth.uid == userId;
       }
     }
   }
   ```
4. No painel do projeto, clique no ícone **`</>`** (Adicionar app da Web), registre um app e copie o objeto `firebaseConfig` gerado.
5. Abra o `index.html` e localize, perto do início do `<script>`, a constante `FIREBASE_CONFIG`. Substitua os valores de exemplo pelos que você copiou:
   ```js
   const FIREBASE_CONFIG = {
     apiKey: "...",
     authDomain: "...",
     projectId: "...",
     storageBucket: "...",
     messagingSenderId: "...",
     appId: "..."
   };
   ```
6. Salve e publique. Ao abrir o app, uma tela de login vai aparecer — crie sua conta com e-mail e senha (**"Criar conta"**), e use a mesma conta no celular. Os lançamentos feitos em um dispositivo aparecem automaticamente no outro.

**Se você não preencher o `FIREBASE_CONFIG`**, o app detecta isso automaticamente e roda no modo padrão (só `localStorage`, sem tela de login) — nada quebra.

> O código também tenta usar uma API de armazenamento própria do ambiente de artefatos da Claude (`window.storage`), quando disponível, como camada extra além do `localStorage`. Fora desse ambiente essa tentativa falha em silêncio sem nenhum efeito colateral.

### Chaves usadas no `localStorage`
| Chave | Conteúdo |
|---|---|
| `despesas:entries` | Array com todos os lançamentos |
| `despesas:categorias` | Lista de categorias de despesa já usadas (autocomplete) |

Com a sincronização ativada, esses mesmos dados também ficam espelhados no Firestore, na coleção `usuarios`, em um documento por conta (`usuarios/{uid}`, campos `entries` e `categorias`).

### Estrutura de um lançamento
```json
{
  "id": "e...",
  "tipo": "despesa",
  "categoria": "Condomínio",
  "valor": 745.82,
  "mes": 2,
  "ano": 2026,
  "status": "pago",
  "obs": ""
}
```
`tipo` é `"despesa"` ou `"receita"`. Lançamentos antigos sem o campo `tipo` são tratados como despesa por padrão.

## Stack técnica

- HTML + CSS + JavaScript puro (vanilla), sem frameworks, sem npm, sem build step.
- Fontes carregadas via Google Fonts (Space Grotesk, Inter, JetBrains Mono) — requer conexão com a internet para o visual completo; sem internet, cai nas fontes padrão do sistema.
- Gráficos e tabelas renderizados manualmente (SVG/HTML), sem bibliotecas externas de gráficos.

## Segurança

- Todo texto livre (nome de categoria, e-mail exibido) é escapado antes de ir para a tela — colar HTML ou script em um campo de categoria não executa nada, aparece só como texto.
- Sem Firebase configurado, não há autenticação nem rede: os dados nunca saem do navegador.
- Com Firebase configurado, siga a regra de segurança do Firestore sugerida na seção acima — ela garante que cada conta só acesse os próprios dados.

## Limitações conhecidas

- Sem sincronização entre dispositivos, a menos que a sincronização via Firebase (opcional) esteja configurada — veja acima.
- No modo padrão (sem Firebase), não há autenticação: qualquer pessoa com acesso ao navegador vê e edita os dados.
- Recorrência de N parcelas não atravessa para o ano seguinte: se o mês inicial + parcelas ultrapassar dezembro, as parcelas restantes são descartadas.
- Anos disponíveis fixos em 2026–2030 (ajustável no código, constante `ANOS_DISPONIVEIS`).

## Estrutura do repositório

```
index.html   → aplicação completa (único arquivo necessário)
README.md    → este documento
```

## Publicando no Vercel

### 1. Subir no GitHub

**Sem terminal:**
1. Acesse [github.com/new](https://github.com/new), dê um nome ao repositório e clique em **Create repository**.
2. Na página do repositório vazio, clique em **uploading an existing file**.
3. Arraste `index.html` e `README.md` e clique em **Commit changes**.

**Com terminal:**
```bash
cd pasta-do-projeto
git init
git add .
git commit -m "Dashboard finanças pessoais"
git branch -M main
git remote add origin https://github.com/SEU_USUARIO/NOME_DO_REPO.git
git push -u origin main
```

### 2. Importar no Vercel

1. Acesse [vercel.com](https://vercel.com) e entre com sua conta do GitHub.
2. Clique em **Add New... → Project**.
3. Selecione o repositório e autorize o acesso, se pedido.
4. Framework preset fica como **Other** (detectado automaticamente) — não há *Build Command* nem *Output Directory* pra configurar, é HTML puro.
5. Clique em **Deploy**. Em menos de um minuto sai uma URL tipo `https://seu-projeto.vercel.app`.

### 3. Atualizações futuras

Qualquer novo `git push` na branch principal (ou upload direto pelo GitHub) dispara um novo deploy automático na Vercel — não precisa reconfigurar nada.

### 4. Domínio próprio (opcional)

Em *Project Settings → Domains*, na Vercel, você pode apontar um domínio seu em vez de usar o subdomínio `.vercel.app`.

> Se você configurou a sincronização via Firebase, o objeto `FIREBASE_CONFIG` no código pode ficar público sem problema — são só identificadores do projeto, não segredos. Quem protege os dados de verdade são as *Firestore Rules* (seção acima), não o sigilo dessas chaves.

## Licença

Defina a licença de sua preferência ao publicar (ex.: MIT) — nenhuma licença está definida por padrão.
