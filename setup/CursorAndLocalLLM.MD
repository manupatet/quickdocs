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
