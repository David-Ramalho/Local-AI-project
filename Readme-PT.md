🚀 Projeto-IA-Local
Este é um ambiente de IA local alimentado pelo Ollama, Docker e Open-WebUI. Ele permite executar e interagir com modelos de IA (como LLaMA 3) diretamente na sua máquina através de uma interface similar ao ChatGPT—privado, rápido e personalizável em apenas 6 passos!
🎯 O Que Você Obtém
🛡️ IA Focada na Privacidade: Mantenha seus dados locais—sem necessidade de nuvem.
→ Suas conversas e dados ficam na sua máquina, garantindo controle total e privacidade.
🧠 Flexibilidade de Modelos: Execute e alterne entre modelos como LLaMA 2, Mistral, ou qualquer modelo suportado pelo Ollama.
→ O Ollama suporta uma variedade de modelos de código aberto—escolha o que melhor se adequa às suas necessidades.
🧩 Personalização: Use o Open-WebUI para dar memória e personalidade à sua IA.
→ Personalize como sua IA se comporta e interage com você.
💾 Armazenamento Persistente: Os volumes do Docker mantêm seus modelos e dados seguros—mesmo após uma reinicialização.
→ Modelos baixados e configurações não desaparecerão quando o container parar ou reiniciar.
🌐 Compartilhamento Online Opcional: Use ferramentas como ngrok para compartilhar sua IA com outros de forma segura pela internet.
→ Ótimo para acesso remoto, demonstrações, ou colaboração com amigos e equipes.
🛠️ Passos de Instalação
Aqui está como configurar seu ambiente de IA local do zero. Estes são os passos exatos que eu usei—apenas siga!
1. 🔧 Instalar Docker Desktop
Abra o PowerShell ou CMD como Administrador.
Execute o seguinte comando para instalar o Docker:
winget install --id Docker.DockerDesktop --source winget

Reinicie sua máquina após a instalação. Inicie o Docker Desktop, complete a configuração inicial e certifique-se de que "Use the WSL 2 based engine" esteja habilitado.

2. 📦 Criar um Volume Persistente para Modelos do Ollama
Isso garante que seus modelos baixados sobrevivam a paradas ou exclusões de containers:
docker volume create ollama-data
Show Image
3. 🤖 Executar o Ollama como um Serviço HTTP
Este comando executa o Ollama e o torna disponível como uma API:
docker run -d --name ollama-server --gpus all -p 11434:11434 -v ollama-data:/root/.ollama -e OLLAMA_HOST="0.0.0.0:11434" ollama/ollama:latest serve
Show Image
📘 Detalhamento:

-d: Executar em segundo plano
--gpus all: Usar sua GPU se disponível
-p 11434:11434: Expor a porta da API do Ollama
-v ollama-data:/root/.ollama: Usar seu volume persistente
-e OLLAMA_HOST: Fazer o Ollama escutar conexões externas

👉 Isso transforma o Ollama em um serviço de fornecimento de modelos com o qual ferramentas como Open-WebUI podem se comunicar.
4. 📥 Baixar um Modelo via CLI do Ollama
Baixe seu modelo de IA preferido (exemplo abaixo usa phi4-mini-reasoning):
docker exec -it ollama-server ollama pull phi4-mini-reasoning:latest
💡 Você pode substituir phi4-mini-reasoning:latest por qualquer outro modelo—como llama3.1:8b. Confira a biblioteca de modelos do Ollama para mais opções em https://ollama.com/search
5. 🌐 Executar Open-WebUI e Conectar ao Ollama
Primeiro, crie um volume para armazenar os dados do WebUI:
docker volume create open-webui
Em seguida, execute o seguinte comando para iniciar a interface:
docker run -d --name open-webui --restart always --gpus all -p 3000:8080 --add-host=host.docker.internal:host-gateway -v open-webui:/app/backend/data -e PROVIDERS="ollama" -e OLLAMA_URL="http://host.docker.internal:11434" ghcr.io/open-webui/open-webui:cuda
📘 Detalhamento:

--restart always: Reinicialização automática em caso de falha ou reinicialização
--gpus all: Opcional, para melhor performance
-p 3000:8080: WebUI estará disponível em http://localhost:3000
-v open-webui:/app/backend/data: Configurações persistentes do WebUI
-e PROVIDERS="ollama": Definir Ollama como backend
-e OLLAMA_URL: Dizer ao WebUI onde encontrar seu servidor Ollama

6. 🗨️ Navegue, Selecione e Gere!
Abra seu navegador e vá para:
http://localhost:3000
Show Image
🌍 Opcional: Compartilhamento Online com Ngrok
Quer permitir que amigos ou colegas usem sua IA remotamente? Use ngrok para criar um túnel seguro para seu localhost.
Cadastre-se para uma conta gratuita aqui: https://ngrok.com
📝 Um guia completo sobre usar Ngrok está chegando em breve, mas sinta-se à vontade para experimentar por conta própria!
❓ Perguntas Frequentes
Q: Estou recebendo um erro "failed to write file: exit status 0xffffffff" no Ubuntu WSL. Qual é a solução?
A: Isso acontece quando o Docker não consegue se conectar à sua distro Ubuntu WSL. Aqui está como corrigir:
Desregistrar a distro com problema:
wsl --unregister Ubuntu
Reinstalar o Ubuntu:
wsl --install -d Ubuntu
Vá para Docker Desktop → Settings → Resources → WSL Integration, e reative o Ubuntu.
Reinicie o Docker Desktop.
Q: Posso usar modelos diferentes?
Absolutamente! No passo 4, substitua phi4-mini-reasoning:latest por qualquer outro modelo como llama2:13b.
👉 Confira a biblioteca de modelos do Ollama para mais opções!
Q: Como paro ou excluo os containers?
Para parar os containers:
docker stop ollama-server open-webui
Para removê-los:
docker rm ollama-server open-webui
🗂️ Nota: Seus dados estão seguros nos volumes, mesmo se os containers forem excluídos.
Q: E se eu não tiver uma GPU?
Sem problemas! Apenas remova a flag --gpus all dos comandos Docker run.
⚠️ Vai executar na sua CPU—mais lento, mas funciona!
👏 Créditos
Obrigado às ferramentas incríveis que tornaram este projeto pessoal possível:
💡 Ollama
🖥️ Open-WebUI
🐳 Docker
🌐 Ngrok
💬 É isso! Você agora tem um ambiente de IA local completo, privado, personalizável e extensível. Aproveite seu próprio assistente similar ao ChatGPT—nos seus próprios termos!
💡 Planejo no futuro mostrar como configurar e treinar os modelos de IA locais usando seus próprios dados! Acompanhe!
