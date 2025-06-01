# Running a Local LLM on Cursor IDE via SSH Port Forwarding  

I successfully set up a local LLM on Cursor IDE using Ollama with SSH port forwarding.  
Initially, I encountered three issues, but after some quick fixes, I got the green slider on.  
Here’s how.

## Issues & Fixes

### 1. Ollama Only Serving `localhost`  
By default, Ollama refuses external connections. Fix this by setting:  
```
    export OLLAMA_HOST=0.0.0.0
```

### 2. CORS Issue (`403 Forbidden`)  
Ollama blocks requests from non-local origins. Allow all origins with:  
```
    export OLLAMA_ORIGINS='*'
```

#### Validate CORS Fix  
Run:  
```
    curl -X GET http://localhost:11434 \
      -H "Origin: http://example.com" \
      -H "Access-Control-Request-Method: GET" -I
```
If you see `HTTP 200 OK`, CORS is properly configured.

### 3. Cursor Verification Failure (Requires `default` Model)  
Cursor expects a model named `default` for API key verification. Copy your model:  
```
    ollama cp <your_model_name> default
```

## Exposing Ollama via SSH Port Forwarding  

Run:  
```
    ssh -R 80:localhost:11434 serveo.net
```
This provides a forwarding URL like:  
```
    Forwarding HTTP traffic from https://your-serveo-url.serveo.net
```

## Configuring Cursor IDE  

1. Open **Settings** (⚙️).  
2. Under **Models**, add a new model (from `ollama list`). Remember to press enter, instead of clicking 'Add model' again (as of version `0.46.11` it doesn't work)   
3. In **OpenAI API Key**:  
   - Click **Override OpenAI Base URL** and enter:  
```
         https://your-serveo-url.serveo.net/v1
```
   - Enter any value in **API Key** and click **Verify**.  

If setup is correct, you’ll see a green slider turn on.

---

## TL;DR (Quick Steps)  

    export OLLAMA_HOST=0.0.0.0
    export OLLAMA_ORIGINS='*'
    ollama cp <your_model_name> default
    ssh -R 80:localhost:11434 serveo.net

- **Set OpenAI Base URL**: `https://your-serveo-url.serveo.net/v1`  
- **Enter random API Key & Verify**   

You're set!

## If you have llama.cpp (more performant than ollama) on Cursor IDE via Reverse proxy (installed on raspberry pi)
Run llama.cpp with a model

```
brew install llama.cpp

llama-server -m ~/repo/llamacpp-models/unsloth_Qwen3-32B-GGUF_Qwen3-32B-UD-Q4_K_XL.gguf --port 1234 --api-key abcdefgh --host 0.0.0.0
```

The raw llm has started on your machine. Now we'll setup reverse proxy server using nginx (on a raspberry pi).

Install nginx on raspberry pi 

```
sudo apt-get remove --purge nginx nginx-full nginx-common
sudo apt -y update
sudo apt install nginx
```

Add the key and cert files:
```
sudo mkdir -p /etc/nginx/ssl/
sudo openssl req -x509 -nodes -days 365  -newkey rsa:2048 -keyout /etc/nginx/ssl/llm.mylocal.com.key -out /etc/nginx/ssl/llm.mylocal.com.crt -subj "/CN=llm.mylocal.com"
```

Add the sites config file: `sudo vi /etc/nginx/sites-available/llm.mylocal.com` 

Then add the following block to it (take care of changing the IP):
```
server {
    listen 443 ssl;
    server_name llm.mylocal.com;

    ssl_certificate     /etc/nginx/ssl/llm.mylocal.com.crt;
    ssl_certificate_key /etc/nginx/ssl/llm.mylocal.com.key;

    location / {
        proxy_pass http://xx.xx.xx.xx:1234; #change the ip

        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
    }
}

# Optional: Redirect HTTP → HTTPS
server {
    listen 80;
    server_name llm.mylocal.com;
    return 301 https://$host$request_uri;
}
```

Create a soft link to the new site:
```
sudo ln -s /etc/nginx/sites-available/llm.mylocal.com /etc/nginx/sites-enabled/
```

Test nginx config
```
sudo nginx -t
```

Restart nginx
```
sudo systemctl reload nginx
```
