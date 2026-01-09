#  Homelab Infrastructure

Documentação completa da infraestrutura do homelab, incluindo todos os serviços, projetos e configurações.

##  Visão Geral

| Componente | Status | Porta | Descrição |
|------------|--------|-------|-----------|
| Ollama |  Running | 11434 | LLM Server (qwen2.5-coder:7b) |
| GitHub Agent |  Running | 8502 | Agente AI para GitHub |
| RAG Dashboard |  Running | 8501 | Dashboard de monitoramento RAG |
| RAG API |  Running | 8001 | Backend API do RAG |
| Nginx |  Running | 80 | Proxy IPv6IPv4 para WireGuard |
| WireGuard |  Running | - | Túnel para Fly.io |

##  URLs de Acesso

### Acesso Local (LAN)
- **Ollama API**: http://192.168.15.2:11434
- **GitHub Agent**: http://192.168.15.2:8502
- **RAG Dashboard**: http://192.168.15.2:8501
- **RAG API**: http://192.168.15.2:8001

### Acesso Externo (Internet via Fly.io)
**URL Base**: https://homelab-tunnel-sparkling-sun-3565.fly.dev

| Endpoint | Serviço |
|----------|---------|
| \/\ | Health Check |
| \/health\ | Status do túnel |
| \/api/ollama/*\ | Ollama API |
| \/v1/*\ | OpenAI-compatible API |
| \/github/*\ | GitHub Agent |
| \/rag/*\ | RAG Dashboard |
| \/api/rag/*\ | RAG API |

### Acesso via Cloudflare Tunnel
- **Ollama**: https://container-restored-remains-configure.trycloudflare.com

##  Arquitetura

\                    
                                     INTERNET                     
                    
                                        
                    
                                                          
                                                          
                  
               Fly.io           Cloudflare         LAN         
               (Caddy)           Tunnel           192.168.15.x 
                  
                                                          
                     WireGuard IPv6                       
                                                          
                                          
                Nginx                                    
              IPv6IPv4                                  
                                          
                                                          
                    
                                        
                    
                               HOMELAB SERVER              
                               192.168.15.2                
                                                           
                         
                       Ollama    GitHub      RAG    
                       :11434    Agent     Backend  
                                 :8502      :8001   
                         
                                                           
                                                
                                     RAG                 
                                  Dashboard              
                                    :8501                
                                                
                    
\
##  Projetos

### 1. GitHub Agent
- **Repositório**: https://github.com/eddiejdi/github-agent
- **Porta**: 8502
- **Tecnologia**: Streamlit + Ollama
- **Serviço**: \github-agent.service
Agente AI que interage com repositórios GitHub usando LLM local.

### 2. RAG Dashboard
- **Repositório**: https://github.com/eddiejdi/rag-dashboard
- **Porta**: 8501
- **Tecnologia**: Streamlit
- **Serviço**: \ag-dashboard.service
Dashboard de monitoramento e métricas do sistema RAG.

### 3. RAG API (PersonaIDE)
- **Porta**: 8001
- **Tecnologia**: FastAPI + ChromaDB
- **Serviço**: \personaide-rag.service
Backend API para Retrieval-Augmented Generation.

### 4. Fly.io Tunnel
- **Repositório**: https://github.com/eddiejdi/flyio-tunnel
- **URL**: https://homelab-tunnel-sparkling-sun-3565.fly.dev
- **Tecnologia**: Caddy + WireGuard

Túnel seguro para expor serviços na internet.

### 5. GitHub MCP Server
- **Repositório**: https://github.com/eddiejdi/github-mcp-server
- **Tecnologia**: Python MCP

Servidor MCP (Model Context Protocol) para integração com GitHub.

### 6. Homelab Scripts
- **Repositório**: https://github.com/eddiejdi/homelab-scripts
- **Conteúdo**: Scripts de automação e documentação

##  Serviços Systemd

### Ollama
\\ash
sudo systemctl status ollama
# Porta: 11434
# Modelos: qwen2.5-coder:7b, github-agent:latest, codestral:22b, deepseek-coder-v2:16b
\
### GitHub Agent
\\ash
sudo systemctl status github-agent
# Porta: 8502
# Arquivo: /etc/systemd/system/github-agent.service
\
### RAG Dashboard
\\ash
sudo systemctl status rag-dashboard
# Porta: 8501
# Arquivo: /etc/systemd/system/rag-dashboard.service
\
### RAG API
\\ash
sudo systemctl status personaide-rag
# Porta: 8001
# Arquivo: /etc/systemd/system/personaide-rag.service
\
### WireGuard (Fly.io Tunnel)
\\ash
sudo systemctl status wg-quick@fly0
# Interface: fly0
# IP: fdaa:3b:60e0:a7b:8cfe:0:a:102/120
# Config: /etc/wireguard/fly0.conf
\
### Nginx (Proxy IPv6IPv4)
\\ash
sudo systemctl status nginx
# Config: /etc/nginx/modules-enabled/flyio-proxy.conf
# Função: Traduz conexões IPv6 do WireGuard para serviços IPv4 locais
\
### Cloudflare Tunnel
\\ash
sudo systemctl status cloudflare-ollama
# Expõe Ollama via Cloudflare
\
##  Modelos LLM Disponíveis

| Modelo | Tamanho | Uso |
|--------|---------|-----|
| qwen2.5-coder:7b | 7.6B | Modelo padrão (rápido) |
| qwen2.5-coder:1.5b | 1.5B | Ultra rápido |
| github-agent:latest | 22.2B | Customizado para GitHub |
| codestral:22b | 22.2B | Codificação avançada |
| deepseek-coder-v2:16b | 15.7B | Alternativa |
| nomic-embed-text:latest | 137M | Embeddings |

##  Comandos Úteis

### Verificar Status
\\ash
# Todos os serviços
systemctl list-units --type=service --state=running | grep -E 'github|rag|ollama|nginx|fly'

# Portas em uso
ss -tlnp | grep -E '8001|8501|8502|11434'

# WireGuard
wg show fly0

# Logs
journalctl -u github-agent -f
journalctl -u rag-dashboard -f
journalctl -u ollama -f
\
### Testar Conectividade
\\ash
# Fly.io via WireGuard
ping6 fdaa:3b:60e0:a7b:5be:73ef:1e7a:2

# Endpoints
curl https://homelab-tunnel-sparkling-sun-3565.fly.dev/health
curl https://homelab-tunnel-sparkling-sun-3565.fly.dev/api/ollama/api/tags
\
### Reiniciar Serviços
\\ash
sudo systemctl restart github-agent
sudo systemctl restart rag-dashboard
sudo systemctl restart personaide-rag
sudo systemctl restart ollama
sudo systemctl restart wg-quick@fly0
sudo systemctl restart nginx
\
##  Configurações de Rede

### WireGuard (fly0)
- **IP Local**: fdaa:3b:60e0:a7b:8cfe:0:a:102/120
- **Fly.io IP**: fdaa:3b:60e0:a7b:5be:73ef:1e7a:2
- **Rede**: fdaa:3b:60e0::/48

### Nginx Stream Proxy
Configurado em \/etc/nginx/modules-enabled/flyio-proxy.conf\:
- Porta 8501 (RAG Dashboard)
- Porta 8502 (GitHub Agent)
- Porta 8001 (RAG API)

##  Histórico de Alterações

### 2026-01-09
-  Criado túnel Fly.io com WireGuard
-  Configurado Nginx como proxy IPv6IPv4
-  Criado serviço rag-dashboard.service
-  Documentação completa do homelab

### 2026-01-08
-  Organização dos projetos em /home/homelab/projects/
-  Push de todos os projetos para GitHub
-  Correções no GitHub Agent

### 2026-01-07
-  Setup inicial do GitHub Agent
-  Configuração do Ollama
-  Cloudflare Tunnel para Ollama

##  Licença

MIT
