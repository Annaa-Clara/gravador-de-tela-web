# Gravador de Tela

Aplicação web full-stack para gravar tela, microfone e webcam diretamente do navegador, sem instalar nada. Projeto de portfólio focado em backend, com frontend em React/TypeScript e um backend Node/Express/PostgreSQL com propósito real (contas, sincronização e compartilhamento), não decorativo.

O projeto foi pensado para reduzir a necessidade de instalação de softwares de gravação, permitindo que o processamento da captura aconteça diretamente no navegador. Isso é especialmente útil em dispositivos com pouco espaço em disco ou em ambientes nos quais o usuário não pode instalar aplicações.

**A gravação em si acontece 100% no navegador.** Nenhum frame de vídeo passa pelo backend a menos que o usuário ative explicitamente o backup em nuvem (recurso opcional).

---

## Sumário

- [Demonstração](#demonstração)
- [Funcionalidades](#funcionalidades)
- [Arquitetura](#arquitetura)
- [Tecnologias](#tecnologias)
- [Estrutura de pastas](#estrutura-de-pastas)
- [Como rodar localmente](#como-rodar-localmente)
- [Privacidade e Segurança](#privacidade-e-segurança)
- [Decisões arquiteturais](#decisões-arquiteturais)

---

## Demonstração

A aplicação tem quatro áreas principais:

1. **Landing page** (`/`) — apresentação do produto, como funciona, recursos, privacidade e stack.
2. **Gravar** (`/gravar`) — tela de configuração e gravação (qualidade, microfone, webcam, áudio do sistema).
3. **Minhas gravações** (`/gravacoes`) — biblioteca local com busca, filtros, player, download e exclusão.
4. **Compatibilidade** (`/compatibilidade`) — o que o seu navegador suporta agora, e uma matriz comparativa entre navegadores.

---

## Funcionalidades

- Captura de tela inteira, janela ou aba (via `getDisplayMedia`)
- Microfone e webcam opcionais (via `getUserMedia`)
- Webcam sobreposta em bolha, com posição e tamanho configuráveis (composição em `<canvas>`)
- Áudio do sistema quando o navegador/SO permitir, com aviso claro quando não permitir
- Qualidade ajustável (Automática/Alta/Média/Baixa), com detecção de codecs suportados
- Iniciar, pausar, retomar, finalizar e cancelar a gravação, com contador em tempo real
- Preview ao vivo durante a gravação
- Biblioteca local (IndexedDB): busca, ordenação, filtros, seleção múltipla e exclusão em lote
- Geração automática de thumbnail a partir de um frame real do vídeo
- Player customizado (play/pause, seek, volume, velocidade, tela cheia)
- Download com nome de arquivo padronizado (`gravacao-tela-AAAA-MM-DD-HH-mm.webm`)
- Painel de armazenamento local (uso estimado via `navigator.storage.estimate()`)
- Painel de compatibilidade ao vivo + matriz comparativa Chrome/Edge/Firefox/Safari
- Avisos contextuais no momento certo (não um único aviso genérico)
- PWA instalável (o "casco" da aplicação funciona offline; a gravação em si depende das APIs de mídia ao vivo do navegador, então nunca é uma funcionalidade "offline")
- **Backend opcional**: contas de usuário, sincronização de metadados, backup em nuvem do vídeo e links de compartilhamento com expiração

---

## Arquitetura

O processamento da gravação ocorre 100% no navegador. O vídeo gravado não é trafegado para o backend durante a gravação. O upload ocorre exclusivamente caso o usuário solicite a sincronização em nuvem.

> Diagramas detalhados do fluxo de dados e schema do banco estão no arquivo [`ARCHITECTURE.md`](./ARCHITECTURE.md).

---

## Tecnologias

**Frontend**
- React 19 + TypeScript
- Vite 8
- Tailwind CSS v4
- React Router
- Dexie.js (IndexedDB)
- MediaRecorder API, Screen Capture API (`getDisplayMedia`), `getUserMedia`, Web Audio API
- Vitest + Testing Library

**Backend** (opcional)
- Node.js + TypeScript
- Express 5
- PostgreSQL (via `pg`, sem ORM com binários nativos — ver [Decisões arquiteturais](#decisões-arquiteturais))
- node-pg-migrate (migrations)
- JWT (`jsonwebtoken`) + `bcryptjs`
- Zod (validação)
- Multer (upload) + AWS SDK v3 (armazenamento S3-compatível opcional)
- Vitest + Supertest

---

## Estrutura de pastas

```
gravador-de-tela/
├── frontend/
│   ├── src/
│   │   ├── app/
│   │   ├── components/
│   │   ├── hooks/
│   │   ├── lib/
│   │   ├── pages/
│   │   └── types/
│   └── public/
├── backend/
│   ├── src/
│   │   ├── routes/
│   │   ├── middleware/
│   │   ├── storage/
│   │   ├── lib/
│   │   └── __tests__/
│   ├── migrations/
│   └── Dockerfile
├── docker-compose.yml
└── ARCHITECTURE.md
```

---

## Como rodar localmente

## Frontend (Stand-alone)
```bash
cd frontend
npm install
npm run dev
```

Acesse http://localhost:5173. O app funciona 100% local sem necessidade do backend.

## Backend (opcional)
```bash
docker compose up -d

cd backend
cp .env.example .env
npm install
npm run migrate:up
npm run dev
```

Para conectar o frontend à API, defina no frontend/.env:
VITE_API_URL=http://localhost:4000

---

## Testes

Execute a suíte de testes de cada ecossistema:

```bash
cd frontend && npm run test
```

```bash
cd backend && npm run test
```

---

## Privacidade e Segurança

- Processamento Local: Vídeos gravados não trafegam pela rede sem autorização prévia.
- Criptografia e Validação: Senhas armazenadas via bcrypt (cost factor 12) e validação de entrada estrita com Zod.
- Segurança na API: Sanitização de uploads, cabeçalhos de proteção via helmet, rate-limiting e queries parametrizadas prevenindo SQL Injection.
  
---

## Decisões arquiteturais

- **Por que o backend é opcional?** Gravação de tela é um processo totalmente client-side por padrão de segurança dos navegadores. O backend foi desenhado estritamente para adicionar valor ao produto (autenticação, sincronização entre múltiplos dispositivos e compartilhamento).
- **Por que pg puro em vez de um ORM (como Prisma)?** O uso do driver nativo pg com node-pg-migrate elimina dependências de binários específicos da plataforma e garante controle total sobre as consultas executadas, mantendo a instalação rápida e sem atritos de build.
- **Composição de Webcam com Canvas:** A fusão do fluxo da webcam e da tela em tempo real via <canvas> entrega um único arquivo WebM/MP4 pronto para compartilhamento, evitando o processamento pesado de edição pós-gravação.
