go to google collab and run the following commands one by one 



!apt-get update && apt-get install -y zstd  -----> install zstd


!curl -fsSL https://ollama.com/install.sh | sh         ------> install ollama

        after this you can also follow this or you can continue : https://github.com/VamsiMarriwada07/ollama-with-colab-and-ngrok/blob/main/ollama.ipynb

!wget -q https://bin.equinox.io/c/bNyj1mQVY4c/ngrok-v3-stable-linux-amd64.tgz
!tar -xzf ngrok-v3-stable-linux-amd64.tgz ngrok


-----> download and install ngrok



!./ngrok authtoken <your_auth_token>                -------> verify ngrok account


!ollama serve & ./ngrok http 11434 --host-header="localhost:11434" --log stdout & sleep 5s && ollama run llama3.2

--> run your llama specefic version.




------> now we need to consume it in our CAP backend service to do that



  const OLLAMA_BASE = 'https://nonsequestered-muhly-carrol.ngrok-free.dev';
    const MODEL       = 'llama3.2';


    const system = 'You answer questions about the provided store inventory and send data in {Item_name(name)-->item code(sku code)---> demand score} format in a simple message format not json or any other.';

    const user = `Inventory: ${JSON.stringify(snapshot)}\n\nQuestion: ${question}\nAnswer:`;

      // Free-form answer (no format: 'json' here)
      const res = await fetch(`${OLLAMA_BASE}/api/chat`, {
        method: 'POST',
        headers: { 'Content-Type': 'application/json' },
        body: JSON.stringify({
          model: MODEL,
          stream: false,
          options: { temperature: 1 },
          messages: [
            { role: 'system', content: system },
            { role: 'user',   content: user }
          ]
        })
      });
