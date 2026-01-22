# 🐳 Guia Completo de Docker


---

## 📑 Índice

1. [O que é Docker?](#o-que-é-docker)
2. [Conceitos Fundamentais](#conceitos-fundamentais)
3. [Instalação e Primeiros Passos](#instalação-e-primeiros-passos)
4. [Trabalhando com Imagens](#trabalhando-com-imagens)
5. [Trabalhando com Containers](#trabalhando-com-containers)
6. [Comandos para o Dia a Dia](#comandos-para-o-dia-a-dia)
7. [Fluxos Práticos Completos](#fluxos-práticos-completos)
8. [Problemas Comuns e Soluções](#problemas-comuns-e-soluções)
9. [Boas Práticas](#boas-práticas)
10. [Cheat Sheet](#cheat-sheet)

---

## 🎯 O que é Docker?

Docker é uma plataforma que permite **empacotar e executar aplicações** de forma isolada e consistente, independente do ambiente. 

### 🤔 Por que Docker é importante? 

**Problema comum:**
```
Desenvolvedor: "Funciona na minha máquina!" 😅
Servidor: "Erro 500" ❌
```

**Com Docker:**
```
Desenvolvedor: "Funciona no meu container!" ✅
Servidor: "Funciona no meu container!" ✅
```

### 📦 Analogia Simples

Pense no Docker como um **container de navio**: 

```
🏭 Container de Navio           🐳 Docker Container
─────────────────────           ──────────────────
│ Caixa padronizada    │   =   │ Ambiente isolado     │
│ Funciona em qualquer │   =   │ Roda em qualquer     │
│ navio/porto         │   =   │ máquina com Docker   │
│ Carrega mercadorias │   =   │ Carrega sua aplicação│
─────────────────────           ──────────────────
```

---

## 📚 Conceitos Fundamentais

### 1️⃣ **Imagem vs Container**

#### 🖼️ **IMAGEM** = Receita de Bolo (Estática)
- É um **arquivo** que contém tudo que sua aplicação precisa
- **Imutável** - não muda depois de criada
- É o "molde" para criar containers
- Pode criar **múltiplos containers** a partir de uma imagem

#### 📦 **CONTAINER** = Bolo Assado (Dinâmico)
- É uma **instância rodando** criada a partir de uma imagem
- **Mutável** - pode ter estado diferente
- É o que **realmente executa** sua aplicação
- Pode ter vários containers da mesma imagem

### 📊 Analogia em Programação

```javascript
// IMAGEM = Classe
class MinhaAPI {
  constructor() {
    this.porta = 3000;
  }
}

// CONTAINER = Instância/Objeto
const container1 = new MinhaAPI(); // Container rodando na porta 3000
const container2 = new MinhaAPI(); // Outro container independente
```

### 🔄 Ciclo de Vida Completo

```
┌─────────────┐
│  Dockerfile │  ← Receita escrita
└──────┬──────┘
       │ docker build
       ↓
┌─────────────┐
│   IMAGEM    │  ← Receita compilada
└──────┬──────┘
       │ docker run
       ↓
┌─────────────┐
│  CONTAINER  │  ← Aplicação rodando
│   Running   │
└──────┬──────┘
       │ docker stop
       ↓
┌─────────────┐
│  CONTAINER  │  ← Aplicação parada
│   Stopped   │     (pode ser reiniciada)
└──────┬──────┘
       │ docker rm
       ↓
   (deletado)
```

---

## 🚀 Instalação e Primeiros Passos

### Passo 1: Instalar Docker

**Windows/Mac:**
- Baixe o [Docker Desktop](https://www.docker.com/products/docker-desktop/)
- Instale e reinicie o computador
- Abra o Docker Desktop

**Linux:**
```bash
# Ubuntu/Debian
sudo apt update
sudo apt install docker.io
sudo systemctl start docker
```

### Passo 2: Verificar Instalação

```bash
# Verificar versão
docker --version
# Saída esperada: Docker version 24.x.x

# Testar com "Hello World"
docker run hello-world
```

**O que acabou de acontecer? **
1. Docker procurou a imagem `hello-world` localmente
2. Não achou, então baixou do Docker Hub (repositório público)
3. Criou um container a partir da imagem
4. Executou o container (que imprime uma mensagem)
5. Container terminou e parou automaticamente

---

## 🖼️ Trabalhando com Imagens

### O que são Imagens? 

Imagens são **arquivos de leitura** que contêm: 
- Sistema operacional base (ex: Ubuntu, Alpine)
- Seu código da aplicação
- Dependências (Node.js, Python, bibliotecas)
- Configurações e arquivos necessários

### 📝 Comandos Essenciais

#### 1. **Listar Imagens Locais**

```bash
# Ver todas as imagens no seu computador
docker images

# Saída exemplo:
# REPOSITORY          TAG       IMAGE ID       CREATED        SIZE
# minha-api          v1.0      abc123def      2 hours ago    150MB
# node               18        xyz789abc      1 week ago     900MB
```

**Entendendo as colunas:**
- `REPOSITORY`: Nome da imagem
- `TAG`: Versão (como v1.0, latest, etc)
- `IMAGE ID`: Identificador único
- `CREATED`: Quando foi criada
- `SIZE`: Tamanho em disco

#### 2. **Construir uma Imagem**

```bash
# Sintaxe básica
docker build -t nome-da-imagem: versao .

# Exemplo prático
docker build -t minha-api:v1.0 . 
```

**Explicando cada parte:**
- `docker build`: Comando para construir
- `-t`: Tag (nome + versão)
- `minha-api:v1.0`: Nome e versão da imagem
- `.`: Contexto (pasta atual com o Dockerfile)

**⚠️ Pré-requisito:** Você precisa ter um `Dockerfile` na pasta! 

**Exemplo de Dockerfile simples:**
```dockerfile
# Usar Node.js como base
FROM node:18

# Definir pasta de trabalho
WORKDIR /app

# Copiar arquivos de dependências
COPY package*.json ./

# Instalar dependências
RUN npm install

# Copiar código da aplicação
COPY . . 

# Expor porta
EXPOSE 3000

# Comando para iniciar
CMD ["npm", "start"]
```

#### 3. **Opções Úteis ao Construir**

```bash
# Ignorar cache (rebuild completo)
docker build --no-cache -t minha-api:v1.0 .

# Usar Dockerfile com nome diferente
docker build -f Dockerfile.prod -t minha-api:prod .

# Ver o processo detalhado
docker build --progress=plain -t minha-api: v1.0 .
```

#### 4. **Baixar Imagens Prontas**

```bash
# Baixar do Docker Hub
docker pull node:18
docker pull nginx:latest
docker pull postgres:15

# Formato:  docker pull nome:versao
```

#### 5. **Inspecionar Imagem**

```bash
# Ver detalhes completos (JSON)
docker inspect minha-api:v1.0

# Ver histórico de camadas
docker history minha-api:v1.0

# Ver apenas variáveis de ambiente
docker inspect minha-api:v1.0 | grep -A 5 "Env"
```

#### 6. **Remover Imagens**

```bash
# Remover uma imagem específica
docker rmi minha-api:v1.0

# Remover forçadamente (mesmo se em uso)
docker rmi -f minha-api:v1.0

# Remover imagens sem tag (órfãs)
docker image prune

# Remover TODAS imagens não utilizadas
docker image prune -a
```

**⚠️ Atenção:** Use `-a` com cuidado em produção!

---

## 📦 Trabalhando com Containers

### O que são Containers? 

Containers são **instâncias rodando** criadas a partir de imagens.  É onde sua aplicação **realmente executa**.

### 📝 Comandos Essenciais

#### 1. **Listar Containers**

```bash
# Ver apenas containers RODANDO
docker ps

# Ver TODOS (rodando + parados)
docker ps -a

# Formato customizado
docker ps --format "table {{.Names}}\t{{. Status}}\t{{.Ports}}"
```

**Saída exemplo:**
```
CONTAINER ID   IMAGE           STATUS          PORTS                    NAMES
abc123         minha-api:v1    Up 2 hours     0.0.0.0:3000->3000/tcp   app-container
```

#### 2. **Criar e Rodar Container**

##### 🟢 **Modo Básico**

```bash
# Rodar em background
docker run -d minha-api:v1.0

# Explicação: 
# -d = detached (background, libera o terminal)
```

##### 🔵 **Com Nome e Porta (RECOMENDADO)**

```bash
docker run -d \
  --name meu-container \
  -p 3000:3000 \
  minha-api:v1.0

# Explicação:
# --name: Nome amigável para o container
# -p:  Mapear porta (HOST:CONTAINER)
#     HOST = seu computador
#     CONTAINER = dentro do container
```

**Exemplo visual de portas:**
```
Seu Computador          Container
──────────────          ─────────
localhost:3000    →     porta 3000
   (acessa)             (aplicação)
```

##### 🟣 **Com Variáveis de Ambiente**

```bash
# Variáveis individuais
docker run -d \
  --name meu-container \
  -p 3000:3000 \
  -e NODE_ENV=production \
  -e DATABASE_URL=postgres://... \
  -e API_KEY=secret123 \
  minha-api: v1.0

# Usando arquivo . env
docker run -d \
  --name meu-container \
  -p 3000:3000 \
  --env-file .env \
  minha-api:v1.0
```

**💡 Dica:** Use `--env-file` para evitar expor secrets no terminal!

##### 🟡 **Com Volume (Sincronizar Código)**

```bash
# Desenvolvimento com hot-reload
docker run -d \
  --name meu-container \
  -p 3000:3000 \
  -v /caminho/local:/app \
  minha-api:v1.0

# Exemplo Windows: 
# -v C:/Users/SeuNome/projeto:/app

# Exemplo Linux/Mac:
# -v ~/projetos/minha-api:/app
```

**O que isso faz?**
- Conecta sua pasta local COM a pasta do container
- Alterações no código local → refletem automaticamente
- **Perfeito para desenvolvimento!**

##### 🔴 **Modo Interativo (com Terminal)**

```bash
# Rodar e entrar direto no terminal
docker run -it minha-api:v1.0 sh

# Explicação:
# -i = interativo (aceita input)
# -t = terminal (aloca um terminal)
# sh = shell/terminal dentro do container
```

#### 3. **Controlar Containers**

```bash
# Iniciar container parado
docker start meu-container

# Parar container rodando (shutdown gracioso)
docker stop meu-container

# Reiniciar
docker restart meu-container

# Pausar (congela, mas não para)
docker pause meu-container

# Despausar
docker unpause meu-container

# Matar forçadamente (não recomendado)
docker kill meu-container
```

**🕐 Diferença entre stop e kill:**
- `stop`: Envia SIGTERM → espera 10s → envia SIGKILL
- `kill`: Envia SIGKILL imediatamente (pode corromper dados)

#### 4. **Ver Logs**

```bash
# Ver todos os logs
docker logs meu-container

# Seguir em tempo real (como tail -f)
docker logs -f meu-container

# Últimas 50 linhas
docker logs --tail 50 meu-container

# Logs dos últimos 10 minutos
docker logs --since 10m meu-container

# Com timestamps
docker logs -f --timestamps meu-container
```

**💡 Atalho útil:**
```bash
# Ctrl + C para sair do modo follow (-f)
```

#### 5. **Executar Comandos no Container**

```bash
# Rodar um comando
docker exec meu-container npm run build

# Entrar no terminal do container
docker exec -it meu-container sh

# Ver arquivo dentro do container
docker exec meu-container cat /app/package.json

# Listar arquivos
docker exec meu-container ls -la /app
```

**🎯 Caso de uso comum:**
```bash
# Container está rodando mas com erro
docker exec -it meu-container sh

# Agora você está DENTRO do container
# Pode debugar, testar comandos, ver arquivos
pwd              # Ver pasta atual
ls -la           # Listar arquivos
npm test         # Rodar testes
exit             # Sair
```

#### 6. **Inspecionar Container**

```bash
# Ver todas as informações
docker inspect meu-container

# Ver apenas IP
docker inspect meu-container | grep IPAddress

# Ver variáveis de ambiente
docker inspect meu-container | grep -A 10 "Env"

# Monitorar recursos em tempo real
docker stats meu-container
```

**Saída do `docker stats`:**
```
CONTAINER    CPU %    MEM USAGE / LIMIT    MEM %    NET I/O
meu-app      0.50%    45MiB / 2GiB        2.20%    1kB / 500B
```

#### 7. **Remover Containers**

```bash
# Remover container parado
docker rm meu-container

# Remover container rodando (força)
docker rm -f meu-container

# Remover múltiplos
docker rm container1 container2 container3

# Remover todos os containers parados
docker container prune

# Remover todos (até rodando) - CUIDADO!
docker rm -f $(docker ps -aq)
```

---

## 🛠️ Comandos para o Dia a Dia

### 🔍 Cenário 1: Ver o que está rodando

```bash
# Lista rápida
docker ps

# Com detalhes de portas
docker ps --format "table {{.Names}}\t{{.Status}}\t{{.Ports}}"

# Ver tudo (rodando + parado)
docker ps -a
```

### 🐛 Cenário 2: Debugar um Problema

```bash
# 1. Ver os logs
docker logs -f meu-container

# 2. Se não ajudar, entrar no container
docker exec -it meu-container sh

# 3. Dentro do container, investigar
ls -la                    # Ver arquivos
cat /app/config.json      # Ver configuração
npm run build             # Tentar build manual
node app.js               # Tentar rodar manual
exit                      # Sair quando terminar

# 4. Ver uso de recursos
docker stats meu-container
```

### 🔄 Cenário 3: Atualizar Aplicação

```bash
# 1. Parar container atual
docker stop meu-container

# 2. Remover container
docker rm meu-container

# 3. Reconstruir imagem (com código novo)
docker build -t minha-api:v1.1 .

# 4. Rodar novo container
docker run -d --name meu-container -p 3000:3000 minha-api:v1.1

# 5. Verificar logs
docker logs -f meu-container
```

**💡 Dica Profissional:**
```bash
# Fazer tudo em um comando (use com cuidado!)
docker stop meu-container && \
docker rm meu-container && \
docker build -t minha-api:v1.1 . && \
docker run -d --name meu-container -p 3000:3000 minha-api:v1.1 && \
docker logs -f meu-container
```

### 🧹 Cenário 4: Limpeza Periódica

```bash
# Limpar containers parados
docker container prune -f

# Limpar imagens não usadas
docker image prune -f

# Limpar volumes órfãos
docker volume prune -f

# Limpar redes não usadas
docker network prune -f

# LIMPAR TUDO (use com cuidado!)
docker system prune -a --volumes -f

# Ver quanto espaço será liberado (sem deletar)
docker system df
```

---

## 🎯 Fluxos Práticos Completos

### 📦 Setup Inicial de um Projeto

```bash
# 1. Navegar até o projeto
cd ~/meu-projeto

# 2. Criar Dockerfile (se não existir)
# [Ver exemplo de Dockerfile acima]

# 3. Criar . dockerignore
cat > .dockerignore << EOF
node_modules
npm-debug.log
. env
.git
EOF

# 4. Construir imagem
docker build -t meu-projeto:v1.0 .

# 5. Verificar se criou
docker images | grep meu-projeto

# 6. Rodar container
docker run -d \
  --name meu-app \
  -p 3000:3000 \
  --env-file .env \
  meu-projeto:v1.0

# 7. Testar
curl http://localhost:3000/health
# ou abrir no navegador:  http://localhost:3000

# 8. Ver logs
docker logs -f meu-app
```

### 🔧 Desenvolvimento com Hot Reload

```bash
# Usar volume para refletir mudanças em tempo real
docker run -d \
  --name meu-app-dev \
  -p 3000:3000 \
  -v $(pwd):/app \
  --env-file .env. development \
  -e NODE_ENV=development \
  meu-projeto:v1.0

# Agora toda alteração no código local
# será refletida automaticamente no container! 

# Acompanhar logs
docker logs -f meu-app-dev

# Quando terminar
docker stop meu-app-dev
docker rm meu-app-dev
```

### 🚀 Deploy para Produção

```bash
# 1. Construir imagem otimizada
docker build \
  --no-cache \
  -t meu-projeto:v1.0-prod \
  -f Dockerfile.prod \
  .

# 2. Rodar em modo produção
docker run -d \
  --name meu-app-prod \
  -p 80:3000 \
  --env-file .env.production \
  -e NODE_ENV=production \
  --restart unless-stopped \
  meu-projeto:v1.0-prod

# Explicação:
# --restart unless-stopped: Reinicia automaticamente se cair
# -p 80:3000: Expõe na porta 80 (porta padrão HTTP)

# 3. Monitorar
docker stats meu-app-prod
docker logs -f --tail 100 meu-app-prod
```

### 🧪 Testes e CI/CD

```bash
# Rodar testes em container temporário
docker run --rm \
  -v $(pwd):/app \
  -w /app \
  node:18 \
  npm test

# Explicação:
# --rm: Remove container automaticamente após execução
# -w:  Working directory (pasta de trabalho)

# Lint + Testes + Build
docker run --rm \
  -v $(pwd):/app \
  -w /app \
  node:18 \
  sh -c "npm install && npm run lint && npm test && npm run build"
```

---

## ⚠️ Problemas Comuns e Soluções

### 🔴 Problema 1: Container não inicia

**Sintomas:**
```bash
docker ps -a
# STATUS:  Exited (0) ou Exited (1)
```

**Diagnóstico:**
```bash
# Ver os logs de erro
docker logs meu-container

# Tentar rodar manualmente
docker run -it minha-api:v1.0 sh
```

**Soluções comuns:**
- Verificar se o comando `CMD` no Dockerfile está correto
- Checar se as dependências foram instaladas
- Validar variáveis de ambiente necessárias

---

### 🟠 Problema 2: Porta já em uso

**Erro:**
```
Error: bind:  address already in use
```

**Diagnóstico:**
```bash
# Ver o que está usando a porta
# Windows: 
netstat -ano | findstr :3000

# Linux/Mac:
lsof -i :3000
```

**Soluções:**

**Opção 1: Usar porta diferente**
```bash
# Mapear 3001 no host → 3000 no container
docker run -d -p 3001:3000 minha-api:v1.0
```

**Opção 2: Parar o container anterior**
```bash
# Achar container usando a porta
docker ps --filter "publish=3000"

# Parar e remover
docker stop nome-do-container
docker rm nome-do-container
```

---

### 🟡 Problema 3: Container roda mas não responde

**Sintomas:**
```bash
curl http://localhost:3000
# curl: (7) Failed to connect to localhost port 3000
```

**Diagnóstico:**
```bash
# 1. Verificar se está realmente rodando
docker ps | grep meu-container

# 2. Ver logs
docker logs meu-container

# 3. Testar dentro do container
docker exec -it meu-container sh
curl localhost:3000    # Testar de dentro

# 4. Verificar porta exposta
docker port meu-container
```

**Soluções comuns:**
- Aplicação travou no startup (ver logs)
- Porta exposta errada no `-p`
- Firewall bloqueando (raro em desenvolvimento)

---

### 🔵 Problema 4: Erro de memória (OOMKilled)

**Erro:**
```bash
docker ps -a
# STATUS: Exited (137)
# 137 = Out of Memory Killed
```

**Soluções:**

**Opção 1: Limpar recursos**
```bash
docker system prune -a --volumes -f
```

**Opção 2: Aumentar memória do Docker Desktop**
1. Abrir Docker Desktop
2. Settings → Resources → Memory
3. Aumentar para 4GB ou mais

**Opção 3: Limitar memória do container**
```bash
docker run -d \
  --name meu-container \
  -m 2g \              # Limitar a 2GB
  --memory-swap 2g \   # Sem swap adicional
  minha-api:v1.0
```

---

### 🟢 Problema 5: Alterações não aparecem

**Cenário:** Mudei o código mas o container não reflete

**Causa:** Sem volume montado

**Solução para DEV:**
```bash
# Usar volume
docker run -d \
  -v $(pwd):/app \
  -p 3000:3000 \
  minha-api:v1.0
```

**Solução para PROD:**
```bash
# Reconstruir imagem
docker build -t minha-api:v1.1 .
docker stop meu-container
docker rm meu-container
docker run -d --name meu-container -p 3000:3000 minha-api:v1.1
```

---

### 🟣 Problema 6: Erro "no space left on device"

**Causa:** Docker consumiu todo o espaço em disco

**Solução:**
```bash
# Ver uso de espaço
docker system df

# Limpar agressivamente
docker system prune -a --volumes -f

# Ver novamente
docker system df
```

**Prevenção:**
```bash
# Configurar limpeza automática semanal
# Linux:  Adicionar ao crontab
0 3 * * 0 docker system prune -f
```

---

## ✅ Boas Práticas

### 1. **Nomenclatura Clara**

```bash
# ❌ Evite
docker run -d image:latest
docker run -d image:latest
docker run -d image:latest

# ✅ Prefira
docker run -d --name api-v1 api:v1.0
docker run -d --name api-v2 api:v2.0
docker run -d --name api-dev api:latest
```

---

### 2. **Versionamento Semântico**

```bash
# ❌ Evite em produção
docker build -t minha-api:latest . 

# ✅ Use versões explícitas
docker build -t minha-api:v1.0. 0 .
docker build -t minha-api:v1.0.1 . 

# ✅ Pode usar latest como alias
docker tag minha-api:v1.0.1 minha-api:latest
```

---

### 3. **Variáveis de Ambiente Seguras**

```bash
# ❌ Nunca exponha secrets no comando
docker run -e DB_PASSWORD=senha123 minha-api:v1

# ✅ Use arquivo . env
docker run --env-file .env minha-api:v1

# ✅ Ou Docker Secrets (swarm mode)
echo "senha123" | docker secret create db_password -
```

---

### 4. **Limpeza Regular**

```bash
# Configure uma rotina semanal
docker system prune -f

# Monitore o uso
docker system df
```

---

### 5. **Logs Estruturados**

```bash
# ❌ Evite
docker logs meu-container | grep error

# ✅ Prefira
docker logs --timestamps --since 1h meu-container

# ✅ Use ferramentas de logging
# - Configurar JSON logs
# - Enviar para ELK Stack, Datadog, etc
```

---

### 6. **Health Checks**

Adicione health check no Dockerfile:

```dockerfile
# Verifica se a aplicação responde
HEALTHCHECK --interval=30s --timeout=3s \
  CMD curl -f http://localhost:3000/health || exit 1
```

Verificar status: 
```bash
docker ps
# Veja a coluna STATUS
# - healthy
# - unhealthy
# - starting
```

---

### 7. **Multi-stage Builds**

Reduzir tamanho da imagem: 

```dockerfile
# Stage 1: Build
FROM node: 18 AS builder
WORKDIR /app
COPY package*.json ./
RUN npm install
COPY . .
RUN npm run build

# Stage 2: Runtime (mais leve!)
FROM node:18-alpine
WORKDIR /app
COPY --from=builder /app/dist ./dist
COPY --from=builder /app/node_modules ./node_modules
CMD ["node", "dist/main.js"]
```

**Resultado:**
- Imagem com build:  1.2GB
- Imagem final: 150MB ✨

---

### 8. **Dockerfile Best Practices**

```dockerfile
# ✅ Use imagens oficiais e específicas
FROM node:18-alpine

# ✅ Crie um usuário não-root
RUN addgroup -S appgroup && adduser -S appuser -G appgroup
USER appuser

# ✅ Copie dependências primeiro (cache)
COPY package*.json ./
RUN npm ci --only=production

# ✅ Copie código depois
COPY . . 

# ✅ Exponha porta documentada
EXPOSE 3000

# ✅ Use ENTRYPOINT + CMD
ENTRYPOINT ["node"]
CMD ["dist/main.js"]
```

---

### 9. **. dockerignore**

Sempre criar: 

```dockerignore
# Dependências
node_modules
npm-debug.log

# Arquivos sensíveis
.env
.env.local
*. key
*.pem

# Git
.git
.gitignore

# IDE
.vscode
.idea

# Build artifacts
dist
build
*.log

# Testes
coverage
__tests__
*. test.js
```

---

### 10. **Restart Policies**

```bash
# Desenvolvimento
docker run -d --restart no minha-api:dev

# Produção
docker run -d --restart unless-stopped minha-api:prod

# Opções: 
# - no:  Nunca reinicia (padrão)
# - on-failure: Só se falhar
# - always: Sempre (mesmo após reboot)
# - unless-stopped: Sempre, exceto se você parar manualmente
```

---

## 📋 Cheat Sheet

### 🖼️ IMAGENS

| Comando | O que faz | Quando usar |
|---------|-----------|-------------|
| `docker images` | Lista todas imagens | Ver o que tenho instalado |
| `docker build -t nome: tag .` | Constrói imagem | Após alterar Dockerfile |
| `docker pull nome:tag` | Baixa imagem | Usar imagem pública |
| `docker rmi nome:tag` | Remove imagem | Limpar espaço |
| `docker image prune` | Remove órfãs | Limpeza semanal |

### 📦 CONTAINERS

| Comando | O que faz | Quando usar |
|---------|-----------|-------------|
| `docker ps` | Lista rodando | Ver o que está ativo |
| `docker ps -a` | Lista todos | Ver inclusive parados |
| `docker run -d -p 8080:8080 img` | Cria e roda | Iniciar aplicação |
| `docker stop nome` | Para container | Desligar temporariamente |
| `docker start nome` | Inicia parado | Religar container |
| `docker restart nome` | Reinicia | Após mudança de config |
| `docker rm nome` | Remove parado | Limpar containers antigos |
| `docker rm -f nome` | Remove rodando | Forçar remoção |

### 🐛 DEBUG

| Comando | O que faz | Quando usar |
|---------|-----------|-------------|
| `docker logs nome` | Ver logs | Debugar erros |
| `docker logs -f nome` | Seguir logs | Monitorar em tempo real |
| `docker exec -it nome sh` | Entrar no container | Investigar profundo |
| `docker inspect nome` | Ver config completa | Entender setup |
| `docker stats nome` | Ver recursos | Checar CPU/RAM |

### 🧹 LIMPEZA

| Comando | O que faz | Quando usar |
|---------|-----------|-------------|
| `docker container prune` | Remove parados | Toda semana |
| `docker image prune` | Remove órfãs | Toda semana |
| `docker system prune` | Limpa tudo | Quando sem espaço |
| `docker system df` | Ver uso de espaço | Antes de limpar |

### 🚀 COMBOS ÚTEIS

```bash
# Ver logs em tempo real dos últimos 100 linhas
docker logs -f --tail 100 meu-container

# Reconstruir e rodar
docker build -t app:v1 .  && docker run -d -p 3000:3000 app:v1

# Parar e remover tudo
docker stop $(docker ps -q) && docker system prune -a -f

# Entrar no container mais recente
docker exec -it $(docker ps -q -l) sh

# Ver apenas IDs dos containers
docker ps -q

# Copiar arquivo do container para host
docker cp meu-container:/app/logs.txt ./logs.txt

# Copiar arquivo do host para container
docker cp ./config. json meu-container:/app/config.json
```

---

## 🎓 Resumo Executivo

### Para Começar
1. Instale Docker Desktop
2. Crie um `Dockerfile`
3. Construa:  `docker build -t meu-app: v1 . `
4. Rode: `docker run -d -p 3000:3000 meu-app:v1`
5. Veja logs: `docker logs -f <container-name>`

### Para Desenvolvimento
- Use volumes:  `-v $(pwd):/app`
- Use arquivo . env: `--env-file .env`
- Acompanhe logs: `docker logs -f`

### Para Produção
- Versionamento explícito:  `meu-app:v1.0.5`
- Health checks no Dockerfile
- Restart policy: `--restart unless-stopped`
- Multi-stage builds para imagens menores

### Para Troubleshooting
1. `docker logs nome` → Ver o erro
2. `docker exec -it nome sh` → Entrar e investigar
3. `docker inspect nome` → Ver configurações
4. `docker stats nome` → Ver recursos

### Para Manutenção
- Limpar semanalmente: `docker system prune -f`
- Monitorar espaço: `docker system df`
- Atualizar Docker Desktop regularmente

---

## 📚 Próximos Passos

Após dominar o básico, explore:

1. **Docker Compose** - Gerenciar múltiplos containers
2. **Docker Networks** - Comunicação entre containers
3. **Docker Volumes** - Persistência de dados
4. **Docker Registry** - Hospedar suas próprias imagens
5. **Kubernetes** - Orquestração de containers em escala

---

## 🤝 Contribuindo

Encontrou algum erro ou tem sugestões? 
- Abra uma issue no GitHub
- Envie um pull request
- Compartilhe com outros devs!

---

**Última atualização:** 22 de janeiro de 2026

**Licença:** MIT - Use e compartilhe livremente!  🚀

---

### 💡 Dica Final

> "Docker não é difícil, é diferente. Invista tempo entendendo imagens vs containers, e tudo fará sentido!" 
