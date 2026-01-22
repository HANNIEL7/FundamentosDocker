# 🐳 Guia Completo: Docker de Forma Correta e Funcional

## 📚 Parte 1: Conceitos Fundamentais

### **O que é Docker?**
```
Imagem    → Planta/Blueprint (estática)
Container → Casa/Instância (dinâmica, rodando)

Analogia:
- Imagem = Classe em programação
- Container = Objeto instanciado da classe
```

### **Ciclo de Vida**
```
Imagem (build)
    ↓
Container (run)
    ├─ Running (app executando)
    ├─ Paused (pausado)
    └─ Exited (parado/morto)
```

---

## 🔧 Parte 2: Comandos Essenciais

### **IMAGENS (Blueprints)**

#### Listar imagens
```bash
docker images                          # Listar todas imagens
docker images | grep apiceasa          # Filtrar por nome
docker images --format "{{.Repository}}:{{.Tag}}" # Formato customizado
```

#### Construir imagem
```bash
# Básico
docker build -t apiceasa-api:v1 .     # Build com tag

# Com opções
docker build \
  --no-cache \                        # Ignora cache (rebuild total)
  -t apiceasa-api:latest \            # Tag: repository:version
  -f Dockerfile \                     # Arquivo alternativo
  .                                   # Contexto (pasta atual)
```

#### Remover imagem
```bash
docker rmi apiceasa-api:v1            # Remove imagem
docker rmi -f apiceasa-api:v1         # Force (mesmo em uso)
docker image prune                    # Remove órfãs
docker image prune -a                 # Remove TUDO (cuidado!)
```

#### Inspecionar imagem
```bash
docker inspect apiceasa-api:v1        # Info detalhada (JSON)
docker inspect apiceasa-api:v1 | grep -A 5 "Env"  # Variáveis
docker history apiceasa-api:v1        # Camadas/layers
```

---

### **CONTAINERS (Instâncias Rodando)**

#### Listar containers
```bash
docker ps                             # Containers RODANDO
docker ps -a                          # Containers RODANDO + PARADOS
docker ps -a --format "table {{.Names}}\t{{.Status}}\t{{.Ports}}"
```

#### Criar e rodar container
```bash
# Básico
docker run -d apiceasa-api:v1
# -d = detach (background)

# Com nome e porta
docker run -d \
  --name apiceasa-api \               # Nome identifiável
  -p 3000:3000 \                      # Mapear porta (host:container)
  apiceasa-api:v1

# Com variáveis de ambiente
docker run -d \
  --name apiceasa-api \
  -p 3000:3000 \
  -e NODE_ENV=production \            # Variável individual
  -e PORT=3000 \
  apiceasa-api:v1

# Com arquivo .env
docker run -d \
  --name apiceasa-api \
  -p 3000:3000 \
  --env-file .env \                   # Carregar arquivo completo
  apiceasa-api:v1

# Com volume (monta pasta local)
docker run -d \
  --name apiceasa-api \
  -p 3000:3000 \
  -v c:/Users/Hanniel/APICeasa:/app \ # host:container
  apiceasa-api:v1

# Interativo (terminal)
docker run -it apiceasa-api:v1 sh    # -i=input, -t=terminal
```

#### Iniciar/Parar/Remover
```bash
docker start apiceasa-api             # Iniciar container parado
docker stop apiceasa-api              # Parar (graceful shutdown)
docker restart apiceasa-api           # Restart
docker kill apiceasa-api              # Forçar parada (SIGKILL)
docker rm apiceasa-api                # Remover container parado
docker rm -f apiceasa-api             # Remover à força
```

#### Ver logs
```bash
docker logs apiceasa-api              # Ver logs completos
docker logs -f apiceasa-api           # Follow (tail -f)
docker logs --tail 50 apiceasa-api    # Últimas 50 linhas
docker logs --since 10m apiceasa-api  # Últimos 10 minutos
docker logs --timestamps apiceasa-api # Com timestamps
```

#### Executar comandos dentro do container
```bash
docker exec apiceasa-api npm run build      # Rodar comando
docker exec -it apiceasa-api sh             # Shell interativo
docker exec apiceasa-api cat /app/package.json
```

#### Inspecionar container
```bash
docker inspect apiceasa-api                 # Info detalhada
docker inspect apiceasa-api | grep -A 10 "Env"
docker stats apiceasa-api                   # CPU/RAM em tempo real
docker ps --no-trunc                        # Ver comando completo
```

---

### **LIMPEZA (Garbage Collection)**

```bash
# Remover containers parados
docker container prune -f

# Remover imagens órfãs (não associadas a containers)
docker image prune -f

# Remover volumes não usados
docker volume prune -f

# Remover redes não usadas
docker network prune -f

# TUDO (cuidado!)
docker system prune -a --volumes -f    # Remove TUDO não usado
```

---

## 🎯 Parte 3: Fluxo Prático com seu Projeto

### **Scenario 1: Setup Inicial**

```bash
# 1. Construir imagem
cd c:\Users\Hanniel\APICeasa
docker build -t apiceasa-api:v1.0 .

# 2. Verificar imagem criada
docker images | grep apiceasa

# 3. Rodar container
docker run -d \
  --name apiceasa-api \
  -p 3000:3000 \
  --env-file .env \
  apiceasa-api:v1.0

# 4. Testar
docker logs apiceasa-api
curl http://localhost:3000/health
```

### **Scenario 2: Desenvolvimento com Hot Reload**

```bash
# Usar volume para live-reload
docker run -d \
  --name apiceasa-api-dev \
  -p 3000:3000 \
  -v c:/Users/Hanniel/APICeasa:/app \        # Monta código
  -e NODE_ENV=development \
  apiceasa-api:v1.0

# Editar código local → reflete no container automaticamente
# (se usando tsx watch ou nodemon)
```

### **Scenario 3: Debugging**

```bash
# Ver o que está acontecendo
docker logs -f apiceasa-api              # Logs em tempo real

# Entrar no container
docker exec -it apiceasa-api sh          # Shell interativo

# Dentro do container:
# - npm run build
# - npm test
# - node dist/main.js
# - etc

# Monitorar recursos
docker stats apiceasa-api
```

### **Scenario 4: Atualizar Código**

```bash
# Opção 1: Com volume (dev)
# - Edita código local
# - Container detecta e recarrega (se configurado)

# Opção 2: Sem volume (prod)
docker stop apiceasa-api
docker rm apiceasa-api
docker build --no-cache -t apiceasa-api:v1.1 .
docker run -d \
  --name apiceasa-api \
  -p 3000:3000 \
  apiceasa-api:v1.1
```

---

## ⚠️ Parte 4: Troubleshooting Comum

### **Container não inicia**
```bash
# Ver o erro
docker logs apiceasa-api

# Se log não ajudar, entrar no container e debugar
docker run -it apiceasa-api:v1 sh
# Rodar aplicação manualmente
```

### **Porta já em uso**
```bash
# ❌ Errado
docker run -p 3000:3000 apiceasa-api    # Port 3000 já mapeada?

# ✅ Correto: Usar porta diferente
docker run -p 3001:3000 apiceasa-api    # Host:3001 → Container:3000

# Ou parar o container anterior
docker stop $(docker ps -q --filter "publish=3000")
```

### **Sem memória (OOMKilled - Error 137)**
```bash
# Limpar recursos
docker system prune -a -f

# Aumentar RAM do Docker Desktop
# Configurações → Resources → Memory

# Ou usar limit no run
docker run -m 2g apiceasa-api           # Limitar a 2GB
```

### **Container roda mas não responde**
```bash
# 1. Verificar logs
docker logs apiceasa-api

# 2. Verificar health
curl http://localhost:3000/health

# 3. Entrar e debugar
docker exec -it apiceasa-api sh

# 4. Checar porta dentro do container
docker exec apiceasa-api netstat -tuln | grep 3000
```

---

## 📋 Parte 5: Boas Práticas

### **✅ DO (Faça)**

```bash
# 1. Sempre nomear containers e imagens
docker run --name app -p 3000:3000 image:v1

# 2. Usar tags versionadas
docker build -t app:v1.0 .              # ✓ Versionado
docker build -t app:latest .            # ✓ Referência atual

# 3. Limpar regularmente
docker system prune -f                  # 1x por semana

# 4. Usar --env-file para secrets
docker run --env-file .env app:v1       # ✓ Seguro

# 5. Volumes em desenvolvimento
docker run -v /local:/container app:v1  # ✓ Hot-reload

# 6. Verificar antes de remover
docker ps -a                            # Ver antes de rm
```

### **❌ DON'T (Evite)**

```bash
# 1. Deixar containers órfãos acumulando
# ❌ docker run image:old
# ❌ docker run image:old
# ❌ ... 50x

# 2. Usar :latest em produção
# ❌ docker run app:latest
# ✅ docker run app:v1.0.5

# 3. -v sem entender mapeamento
# ❌ docker run -v /data app         # Ambíguo!
# ✅ docker run -v /local:/app app   # Claro

# 4. Rodar como root
# ❌ USER root
# ✅ USER appuser (no Dockerfile)

# 5. Não usar .dockerignore
# ❌ context= 500MB (node_modules incluído)
# ✅ context= 10MB (excluindo via .dockerignore)
```

---

## 🚀 Parte 6: Cheat Sheet Rápido

```bash
# IMAGENS
docker build -t name:tag .              # Construir
docker images                           # Listar
docker rmi name:tag                     # Remover

# CONTAINERS
docker run -d -p 8080:8080 name:tag    # Criar e rodar
docker ps -a                           # Listar tudo
docker logs -f container               # Ver logs
docker exec -it container sh           # Entrar
docker stop/start/restart container    # Controlar
docker rm container                    # Remover

# LIMPEZA
docker system prune -a -f              # Limpar tudo
docker container prune -f              # Limpar containers
docker image prune -f                  # Limpar imagens

# DEBUG
docker inspect container               # Info detalhada
docker stats container                 # Recursos
docker logs --tail 100 container       # Últimas 100 linhas
```

---

## 📖 Seu Projeto: Exemplos Reais

### **Setup APICeasa**
```bash
# Build
cd c:\Users\Hanniel\APICeasa
docker build -t apiceasa-api:v1 .

# Dev com hot-reload
docker run -d \
  --name apiceasa-dev \
  -p 3000:3000 \
  -v c:/Users/Hanniel/APICeasa:/app \
  --env-file .env \
  apiceasa-api:v1

# Testar
docker exec apiceasa-dev npm run check    # TypeScript check
docker exec apiceasa-dev npm test         # Run tests
docker logs -f apiceasa-dev               # Ver logs

# Produção
docker stop apiceasa-dev
docker rm apiceasa-dev
docker run -d \
  --name apiceasa-api \
  -p 3000:3000 \
  -e NODE_ENV=production \
  --env-file .env \
  apiceasa-api:v1
```

---

## 🎓 Resumo Executivo

| Operação | Comando | Uso |
|----------|---------|-----|
| Construir imagem | `docker build -t name:tag .` | Setup/Build |
| Rodar container | `docker run -d --name x -p 8080:8080 image:tag` | Iniciar app |
| Ver logs | `docker logs -f container` | Debugging |
| Entrar no container | `docker exec -it container sh` | Troubleshooting |
| Parar container | `docker stop container` | Shutdown graceful |
| Listar tudo | `docker ps -a` | Visibilidade |
| Limpar | `docker system prune -a -f` | Manutenção |
| Remover | `docker rm/rmi name` | Cleanup |

---

**Última atualização**: 21 de janeiro de 2026

Pronto para usar Docker como um profissional! 🚀
