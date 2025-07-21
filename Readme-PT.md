# 🚀 Construa Seu Próprio Assistente de IA Local com Docker, Ollama & Open-WebUI

Configure seu próprio assistente privado similar ao ChatGPT localmente usando Ollama, Docker e Open-WebUI.
Este guia te leva através de uma configuração comprovada e direta em apenas 6 passos—testado tanto no Windows quanto no Linux.

---

## 📚 Índice

- [🎯 O Que Você Vai Conseguir](#-o-que-você-vai-conseguir)
- [🧰 O Que Cada Ferramenta Faz](#-o-que-cada-ferramenta-faz)
- [🛠️ Passos da Instalação](#️-passos-da-instalação)
- [🌍 Opcional: Compartilhar Online com Ngrok](#-opcional-compartilhar-online-com-ngrok)
- [❓ Perguntas Frequentes](#-perguntas-frequentes)
- [👏 Créditos](#-créditos)
- [🔮 Em Breve](#-em-breve)

---

## 🎯 O Que Você Vai Conseguir

### 🛡️ IA Focada na Privacidade
Mantenha seus dados locais—sem necessidade de nuvem. Suas conversas e dados ficam na sua máquina, garantindo controle total e privacidade.

### 🧠 Flexibilidade de Modelos
Execute e alterne entre modelos como LLaMA 2, Mistral, ou qualquer modelo suportado pelo Ollama.

### 🧩 Personalização
Use o Open-WebUI para dar memória e personalidade à sua IA. Personalize como sua IA se comporta e interage com você.

### 💾 Armazenamento Persistente
Volumes do Docker mantêm seus modelos e dados seguros—mesmo após uma reinicialização.

### 🌐 Compartilhamento Online Opcional
Use ferramentas como ngrok para compartilhar sua IA com segurança com outros pela internet.

---

## 🧰 O Que Cada Ferramenta Faz

- **Docker**: Executa todos os componentes em containers—portável e isolado.
- **Ollama**: Hospeda e serve seus modelos de IA localmente.
- **Open-WebUI**: Uma interface elegante no navegador para conversar com seus modelos.
- **Ngrok (opcional)**: Compartilha com segurança seu assistente de IA local online.

---

## 🛠️ Processo de Instalação

Estes são os passos exatos que estruturei através de múltiplas configurações de tentativa e erro (mais de cinco instalações completas tanto no Windows quanto no Linux). Apenas siga em frente! PS: Este é o processo de instalação baseado no Windows

### 🖥️ Pré-requisitos & Testado Em

Aqui estão as especificações do sistema e versões usadas durante os testes. Outras configurações podem funcionar, mas esta é uma base comprovada:

| Componente | Versão Mínima / Notas | Testado Em |
|------------|----------------------|------------|
| **Docker Desktop** | Versão estável mais recente | Windows 11 (WSL2) |
| **Docker CLI** | Vem com Docker Desktop | ✅ |
| **Ollama** | Versão estável mais recente | ✅ |
| **Open-WebUI** | Versão estável mais recente | ✅ |
| **GPU** | NVIDIA GTX 1650 4GB ou melhor | ✅ |
| **RAM** | 16 GB mínimo | 16 GB |
| **CPU** | Intel i3 (9ª Geração ou melhor) | Intel i3-9100F |

### ⚙️ Configuração

Antes de executar os containers, aqui estão as principais variáveis de ambiente usadas:

| Variável | Descrição | Valor Padrão / Exemplo |
|----------|-----------|------------------------|
| `OLLAMA_HOST` | Endereço de escuta da API do Ollama | `0.0.0.0:11434` |
| `PROVIDERS` | Especifica backend para Open-WebUI | `ollama` |
| `OLLAMA_URL` | URL para conectar WebUI ao Ollama | `http://host.docker.internal:11434` |

---

## 🛠️ Passos da Instalação

### 1. 🔧 Instalar Docker Desktop
Abra PowerShell ou CMD como Administrador.
Execute o seguinte comando para instalar o Docker:

```bash
winget install --id Docker.DockerDesktop --source winget
```

Reinicie sua máquina após a instalação. Inicie o Docker Desktop, complete a configuração da primeira execução, e certifique-se de que "Use the WSL 2 based engine" esteja habilitado.

### 2. 📦 Criar um Volume Persistente para Modelos do Ollama
Isso garante que seus modelos baixados sobrevivam a paradas ou exclusões de containers:

```bash
docker volume create ollama-data
```

![image](https://github.com/user-attachments/assets/8da63b15-09b4-48e2-8716-9ec9660330b7)

### 3. 🤖 Iniciar Ollama como um Serviço HTTP
Este comando executa o Ollama e o torna disponível como uma API:

```bash
docker run -d --name ollama-server --restart always --gpus all -p 11434:11434 -v ollama-data:/root/.ollama -e OLLAMA_HOST="0.0.0.0:11434" ollama/ollama:latest serve
```

![image](https://github.com/user-attachments/assets/24ae70d9-86cd-4f16-aa0f-94c8993b39b2)

**📘 Explicação:**
- `-d`: Executar em segundo plano  
- `--gpus all`: Usar sua GPU se disponível  
- `-p 11434:11434`: Expor porta da API do Ollama  
- `-v ollama-data:/root/.ollama`: Usar seu volume persistente  
- `-e OLLAMA_HOST`: Fazer Ollama escutar conexões externas  

👉 Isso transforma o Ollama em um serviço de servir modelos com o qual ferramentas como Open-WebUI podem se comunicar.

### 4. 📥 Baixar um Modelo via CLI do Ollama
Baixe seu modelo de IA preferido (exemplo abaixo usa qwen3:1.7b):

```bash
docker exec -it ollama-server ollama pull qwen3:1.7b
```

💡 Você pode trocar `qwen3:1.7b` por qualquer outro modelo—como `llama3.1:8b`. Confira a biblioteca de modelos do Ollama para mais em https://ollama.com/search

### 5. 🌐 Iniciar Open-WebUI e Conectar ao Ollama
Primeiro, crie um volume para armazenar dados do WebUI:

```bash
docker volume create open-webui
```

Então execute o seguinte comando para lançar a interface:

```bash
docker run -d --name open-webui --restart always --gpus all -p 3000:8080 --add-host=host.docker.internal:host-gateway -v open-webui:/app/backend/data -e PROVIDERS="ollama" -e OLLAMA_URL="http://host.docker.internal:11434" ghcr.io/open-webui/open-webui:cuda
```

**📘 Explicação:**
- `--restart always`: Reinicia automaticamente em caso de crash ou reboot  
- `--gpus all`: Opcional, para melhor performance  
- `-p 3000:8080`: WebUI estará disponível em http://localhost:3000  
- `-v open-webui:/app/backend/data`: Configurações persistentes do WebUI  
- `-e PROVIDERS="ollama"`: Definir Ollama como backend  
- `-e OLLAMA_URL`: Dizer ao WebUI onde encontrar seu servidor Ollama

### 6. 🗨️ Agora você pode conversar com seu próprio LLM Local!
Abra seu navegador e vá para:

```
http://localhost:3000
```

![image](https://github.com/user-attachments/assets/ebc2a7e5-68c5-4e4f-9f1e-2d2e9fd0e0dd)

---

## 🌍 Opcional: Compartilhar Online com Ngrok

Quer deixar amigos ou colegas usarem sua IA remotamente? Use ngrok para criar um túnel seguro para seu localhost.

Inscreva-se para uma conta gratuita aqui:
```
https://ngrok.com 
```

Crie um túnel para a porta 3000:
```bash
ngrok http 3000
```

Isso te dá um link temporário para acessar seu app local pela internet.

---

## ❓ Perguntas Frequentes

**P: Estou recebendo um erro "failed to write file: exit status 0xffffffff" no Ubuntu WSL. Qual é a solução?**

R: Isso acontece quando o Docker não consegue conectar à sua distro Ubuntu WSL. Aqui está como consertar:

1. Desregistrar a distro quebrada:
   ```bash
   wsl --unregister Ubuntu
   ```
2. Re-instalar Ubuntu:
   ```bash
   wsl --install -d Ubuntu
   ```
3. Vá para Docker Desktop → Settings → Resources → WSL Integration, e re-habilite Ubuntu.
4. Reinicie Docker Desktop.

**P: Posso usar modelos diferentes?**

Absolutamente! No passo 4, substitua `qwen3:1.7b` por qualquer outro modelo como `llama2:13b`.

👉 Confira a biblioteca de modelos do Ollama para mais opções!

**P: Como parar ou deletar os containers?**

Para parar os containers:
```bash
docker stop ollama-server open-webui
```

Para removê-los:
```bash
docker rm ollama-server open-webui
```

🗂️ **Nota:** Seus dados estão seguros nos volumes, mesmo que os containers sejam deletados.

**P: E se eu não tiver uma GPU?**

Sem problema! Apenas remova a flag `--gpus all` dos comandos Docker run.
⚠️ Vai executar na sua CPU—mais lento, mas funciona!

---

## 👏 Créditos

Obrigado às ferramentas incríveis que tornaram este projeto pessoal possível:

- 💡 **Ollama**: https://ollama.com/ -- https://github.com/ollama/ollama
- 🖥️ **Open-WebUI**: https://openwebui.com/ -- https://github.com/open-webui
- 🐳 **Docker**: https://www.docker.com/
- 🌐 **Ngrok**: https://ngrok.com/

---

## 🔮 Em Breve

- Como configurar memória e conversas persistentes através das configurações de arquivos RAG
- Como treinar modelos usando seus próprios documentos via fine-tuning usando unsloth (Em andamento)  
- Mais dicas de otimização e personalização!

---

💬 **É isso!** Você agora tem um ambiente de IA local completo, privado, personalizável e extensível. Aproveite seu próprio assistente similar ao ChatGPT—nos seus próprios termos!
