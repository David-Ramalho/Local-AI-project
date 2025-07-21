# 🚀 Build Your Own Local AI Assistant with Docker, Ollama & Open-WebUI

Set up your own **private ChatGPT-like assistant** locally using Ollama, Docker, and Open-WebUI.  
This guide walks you through a proven, no-nonsense setup in just 6 steps—tested on both Windows and Linux.

---

## 📚 Table of Contents
- [What You Get](#-what-you-get)
- [What Each Tool Does](#-what-each-tool-does)
- [Installation Steps](#️-installation-steps)
- [Optional: Sharing Online with Ngrok](#-optional-sharing-online-with-ngrok)
- [FAQs](#-faqs)
- [Credits](#-credits)
- [Coming Soon](#-coming-soon)

---

## 🎯 What You Get

- 🛡️ Privacy-Focused AI
     Keep your data local—no cloud required. Your conversations and data stay on your machine, ensuring full control and privacy.

-🧠 Model Flexibility  
   Run and switch between models like LLaMA 2, Mistral, or any Ollama-supported model.

- 🧩 Personalization  
   Use Open-WebUI to give your AI memory and personality. Customize how your AI behaves and interacts with you.

- 💾 Persistent Storage  
   Docker volumes keep your models and data safe—even after a reboot.

- 🌐 Optional Online Sharing  
   Use tools like ngrok to securely share your AI with others over the internet.

---

## 🧰 What Each Tool Does

- **Docker**: Runs all components in containers—portable and isolated.
- **Ollama**: Hosts and serves your AI models locally.
- **Open-WebUI**: A sleek browser interface to chat with your models.
- **Ngrok (optional)**: Securely share your local AI assistant online.

---


# 🛠️ Installation Steps   
   These are the exact steps I structured through multiple trial-and-error setups (over five full installs on both Windows plus on Linux). Just follow along!
   PS: This is Windows' based installation process 

## 🖥️ Prerequisites & Tested On

Here are the system specs and versions used during testing. Other setups may work, but this is a proven baseline:

| Component         | Minimum Version / Notes                      | Tested On              |
|------------------|-----------------------------------------------|------------------------|
| Docker Desktop    | Latest stable  | Windows 11 (WSL2)           |  ✅                    |
| Docker CLI        | Comes with Docker Desktop                    |  ✅                    |
| Ollama            |  Latest stable                               |  ✅                    |
| Open-WebUI        |  Latest stable                               |  ✅                    |
| GPU               | NVIDIA GTX 1650 4GB or better                |  ✅                    |
| RAM               | 16 GB minimum                                | 16 GB                   |
| CPU               | Intel i3 (9th Gen or better)                 | Intel i3-9100F          |


### ⚙️ Configuration

Before running the containers, here are the key environment variables used:

| Variable       | Description                          | Default / Example Value       |
|----------------|--------------------------------------|-------------------------------|
| `OLLAMA_HOST`  | Ollama API listening address         | `0.0.0.0:11434`               |
| `PROVIDERS`    | Specifies backend for Open-WebUI     | `ollama`                      |
| `OLLAMA_URL`   | URL to connect WebUI to Ollama       | `http://host.docker.internal:11434` |



## 1. 🔧 Install Docker Desktop
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
   
    docker run -d --name ollama-server --restart always --gpus all -p 11434:11434 -v ollama-data:/root/.ollama -e OLLAMA_HOST="0.0.0.0:11434" ollama/ollama:latest serve
           
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
   
     docker exec -it ollama-server ollama pull qwen3:1.7b
           
   💡 You can swap qwen3:1.7b with any other model—like llama3.1:8b. Check out Ollama’s model library for more at https://ollama.com/search

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

## 🌍 Optional: Sharing Online with Ngrok
   Want to let friends or colleagues use your AI remotely? Use ngrok to create a secure tunnel to your localhost.  
    Sign up for a free account here: 
    
        https://ngrok.com 
        
   Create a tunnel to port 3000:
   
        ngrok http 3000

It gives you a temporary link to access your local app over the internet.
        
   📝 A full guide on using Ngrok is coming soon, but feel free to try it out on your own!


# ❓ FAQs

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

#  👏 Credits
   ## Thanks to the amazing tools that made this project personal project possible:
        
      💡 Ollama

      🖥️ Open-WebUI
      
      🐳 Docker
      
      🌐 Ngrok

#  🔮 Coming Soon
    How to configure memory and persistent conversations trough Rag files settings
    
    How to train models using your own documents via finetunin using unsloth(On going)  

    More optimization and customization tips!
    

💬 That’s it! You now have a full-featured, private, customizable, and extendable local AI environment. Enjoy your own ChatGPT-like assistant—on your own terms!
