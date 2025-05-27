# 🚀 Local-AI-project
### This is a local AI environment powered by Ollama, Docker, and Open-WebUI. It lets you run and interact with AI models (like LLaMA 3) right on your machine through a ChatGPT-like interface—private, fast, and customizable in just 6 steps!


###  🎯 What You Get
###🛡️ Privacy-Focused AI: Keep your data local—no cloud required.
        → Your conversations and data stay on your machine, ensuring full control and privacy.

###  🧠 Model Flexibility: Run and switch between models like LLaMA 2, Mistral, or any Ollama-supported model.
       → Ollama supports a variety of open-source models—pick the one that fits your needs.

###  🧩 Personalization: Use Open-WebUI to give your AI memory and personality.
       → Customize how your AI behaves and interacts with you.

###  💾 Persistent Storage: Docker volumes keep your models and data safe—even after a reboot.
       → Downloaded models and settings won’t disappear when the container stops or restarts.

###  🌐 Optional Online Sharing: Use tools like ngrok to securely share your AI with others over the internet.
      → Great for remote access, demos, or collaborating with friends and teams.




# 🛠️ Installation Steps
Here’s how to set up your local AI environment from scratch. These are the exact steps I used—just follow along!


##1. 🔧 Install Docker Desktop
  Open PowerShell or CMD as Administrator.
  
  Run the following command to install Docker:
    winget install --id Docker.DockerDesktop --source winget
  
 * Reboot your machine after installation. Launch Docker Desktop, complete the first-run setup, and make sure “Use the WSL 2 based engine” is enabled. 



## 2. 📦 Create a Persistent Volume for Ollama Models
  This makes sure your downloaded models survive container stops or deletions:  
    docker volume create ollama-data
   ![image](https://github.com/user-attachments/assets/8da63b15-09b4-48e2-8716-9ec9660330b7)



## 3. 🤖 Spin Up Ollama as an HTTP Service
  This command runs Ollama and makes it available as an API:
    docker run -d --name ollama-server --gpus all -p 11434:11434 -v ollama-data:/root/.ollama -e OLLAMA_HOST="0.0.0.0:11434" ollama/ollama:latest serve
   
  ![image](https://github.com/user-attachments/assets/24ae70d9-86cd-4f16-aa0f-94c8993b39b2)


  📘 Breakdown:  
    -d: Run in background  
    --gpus all: Use your GPU if available  
    -p 11434:11434: Expose Ollama’s API port  
    -v ollama-data:/root/.ollama: Use your persistent volume  
    -e OLLAMA_HOST: Make Ollama listen for external connections  
    👉 This turns Ollama into a model-serving service that tools like Open-WebUI can talk to.

    

## 4. 📥 Pull a Model via Ollama CLI
  Download your preferred AI model (example below uses phi4-mini-reasoning):  
    docker exec -it ollama-server ollama pull phi4-mini-reasoning:latest
   
  💡 You can swap phi4-mini-reasoning:latest with any other model—like llama3.1:8b. Check out Ollama’s model library for more at https://ollama.com/search

## 5. 🌐 Spin Up Open-WebUI and Connect to Ollama
  First, create a volume for storing WebUI’s data:
    docker volume create open-webui
  
  Then run the following command to launch the interface:
    docker run -d --name open-webui --restart always --gpus all -p 3000:8080 --add-host=host.docker.internal:host-gateway -v open-webui:/app/backend/data -e PROVIDERS="ollama" -e OLLAMA_URL="http://host.docker.internal:11434" ghcr.io/open-webui/open-webui:cuda

  
  📘 Breakdown:  
    --restart always: Auto-restarts on crash or reboot  
    --gpus all: Optional, for faster performance  
    -p 3000:8080: WebUI will be available at http://localhost:3000  
    -v open-webui:/app/backend/data: Persistent WebUI settings  
    -e PROVIDERS="ollama": Set Ollama as backend  
    -e OLLAMA_URL: Tell WebUI where to find your Ollama server



## 6. 🗨️ Browse, Select, and Generate!
  Open your browser and go to:
  http://localhost:3000

![image](https://github.com/user-attachments/assets/ebc2a7e5-68c5-4e4f-9f1e-2d2e9fd0e0dd)

🌍 Optional: Sharing Online with Ngrok
  Want to let friends or colleagues use your AI remotely? Use ngrok to create a secure tunnel to your localhost.  
  Sign up for a free account here: https://ngrok.com    
  📝 A full guide on using Ngrok is coming soon, but feel free to try it out on your own!



#❓ FAQs

  ## Q: I'm getting a "failed to write file: exit status 0xffffffff" error in Ubuntu WSL. What’s the fix?
  A: This happens when Docker can’t connect to your Ubuntu WSL distro. Here's how to fix it:
  
  Unregister the broken distro:
    wsl --unregister Ubuntu
  Re-install Ubuntu:
    wsl --install -d Ubuntu
  
  Go to Docker Desktop → Settings → Resources → WSL Integration, and re-enable Ubuntu.
  
  Restart Docker Desktop.

## Q: Can I use different models?
  Absolutely! In step 4, replace phi4-mini-reasoning:latest with any other model like llama2:13b.
 ###  👉 Check Ollama’s model library for more options!

## Q: How do I stop or delete the containers?
To stop the containers:
  docker stop ollama-server open-webui
To remove them:
  docker rm ollama-server open-webui
  
### 🗂️ Note: Your data is safe in volumes, even if containers are deleted.

## Q: What if I don’t have a GPU?
  No problem! Just remove the --gpus all flag from the Docker run commands.
  ⚠️ It’ll run on your CPU—slower, but it works!

# 👏 Credits
## Thanks to the amazing tools that made this project personal project possible:

### 💡 Ollama
### 🖥️ Open-WebUI
### 🐳 Docker
### 🌐 Ngrok

💬 That’s it! You now have a full-featured, private, customizable, and extendable local AI environment. Enjoy your own ChatGPT-like assistant—on your own terms!

💡I plan on the future show how to configurate and train the local Ai models using your own data! Follow along!
