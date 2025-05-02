# CrewAI + Masumi Starter Kit

This **CrewAI Masumi Starter Kit** lets you quickly deploy your own CrewAI agents and integrate them with Masumi’s decentralized payment solution.
## Quick Start

Follow these steps to quickly get your CrewAI agents live and monetized on Masumi.

### Prerequesites
- [Python >=3.10 and <3.13](https://www.python.org/downloads/)
- [uv](https://docs.astral.sh/uv/getting-started/installation/)

1.  **Clone the Repository**

Clone the repository and navigate into the directory:

```bash
git clone https://github.com/masumi-network/crewai-masumi-quickstart-template.git
cd crewai-masumi-quickstart-template
```

2. **Install Dependencies:**
    ```bash
    uv sync
    ```

3. **Activate virtual environment:**

    ```bash
    source .venv/bin/activate
    ```

4. **Configure Your Environment Variables**

Copy `.env.example` to `.env` and fill with your own data (we will create them in further steps):

```bash
cp .env.example .env
```

Example `.env` configuration:

```ini
# Payment Service 
PAYMENT_SERVICE_URL=http://localhost:3001/api/v1 
PAYMENT_API_KEY=your_payment_service_api_key

# Agent Configuration
AGENT_IDENTIFIER=your_agent_identifier_from_registration
PAYMENT_AMOUNT=10000000
PAYMENT_UNIT=lovelace
SELLER_VKEY=your_selling_wallet_vkey

# OpenAI API
OPENAI_API_KEY=your_openai_api_key
```

5. **Verify that the template agent is working:**
    ```bash
    uvicorn main:app --host=0.0.0.0 --port=${PORT:-8000}
    ```
    On the first run, you might receive an error that will indicate that environment variables `PAYMENT_SERVICE_URL` and `PAYMENT_API_KEY` are missing. It's okay, we are going to set them up in the next steps. 

    If no error occurs, send a request to the server to check that it's working. For example a `/health` request or an `/input_schema`, to get the input schema of your agent. 

5. **Define Your CrewAI Agents**

Edit the file **`crew_definition.py`** to define your agents and their tasks.

When you will update your agent, make sure to import it in the `main.py` and adjust input schema, so that it works with your new agent. 

### 6. **Deploy Your Service**

When you're happy with your agentic service, deploy it.

You can use a hosting provider such as:

- **Digital Ocean** (Recommended)
- AWS, Google Cloud, Azure, etc.

Your project requires:

- **Python 3.12.x**
- **FastAPI** for the API
- **Uvicorn** ASGI server


For local testing, start the API server:

```python
python main.py api
```

Access the interactive API documentation at:
http://localhost:8000/docs

### 7. **Install the Masumi Payment Service**

The Masumi Payment Service handles all blockchain payments for your agent.

Follow the official Masumi installation guide:

👉 [Masumi Payment Installation Guide](https://docs.masumi.network/get-started/installation)

Run Masumi (recommended with Docker):

```bash
curl -X GET 'http://localhost:3001/api/v1/health/' -H 'accept: application/json'
```

You should receive:

```
{
  "status": "success",
  "data": {
    "status": "ok"
  }
}
```

### 8. **Top Up Your Wallet with Test ADA**

Get free Test ADA from Cardano Faucet:

- Copy your wallet address from the Masumi Dashboard.
- Visit the [Cardano Faucet](https://docs.cardano.org/cardano-testnets/tools/faucet).
- Request Test ADA (Preprod network).


### 9. **Register Your Crew on Masumi**

Register your CrewAI agent via Masumi’s API:

```bash
curl -X POST 'http://localhost:3001/api/v1/registry/' \
-H 'accept: application/json' \
-H 'token: your_admin_key' \
-H 'Content-Type: application/json' \
-d '{
  "network": "Preprod",
  "ExampleOutputs": [
    {
      "name": "example_output_name",
      "url": "https://example.com/example_output",
      "mimeType": "application/json"
    }
  ],
  "Tags": [
    "tag1",
    "tag2"
  ],
  "name": "Agent Name",
  "description": "Agent Description",
  "Author": {
    "name": "Author Name",
    "contactEmail": "author@example.com",
    "contactOther": "author_contact_other",
    "organization": "Author Organization"
  },
  "apiBaseUrl": "https://api.example.com",
  "Legal": {
    "privacyPolicy": "Privacy Policy URL",
    "terms": "Terms of Service URL",
    "other": "Other Legal Information URL"
  },
  "sellingWalletVkey": "wallet_vkey",
  "Capability": {
    "name": "Capability Name",
    "version": "1.0.0"
  },
  "AgentPricing": {
    "pricingType": "Fixed",
    "Pricing": [
      {
        "unit": "",
        "amount": "10000000"
      }
    ]
  }
}
```

#### Get your agent identifier:

```bash
curl -X 'GET' \
  'http://localhost:3001/api/v1/registry/?network=Preprod' \
  -H 'accept: application/json' \
  -H 'token: your_admin_key'
```

Note your `agentIdentifier` from the response and update it in your `.env` file.

---

### 10. **Run & Verify Your API**

Start your FastAPI server with integrated Masumi payments:

```bash
curl -X POST "http://localhost:8000/start_job" \
-H "Content-Type: application/json" \
-d '{
    "identifier_from_purchaser": "<put HEX of even character>",
    "input_data": {"text": "artificial intelligence trends"}
}'
```

This returns a `job_id`.

Check job status:

```
http://localhost:8000/docs
```

Test with the provided endpoints:
- `/start_job` to initiate paid AI tasks
- `/status` to check job status and payment state
- `/availability` to check service availability


## **Summary & Next Steps**

- [x] Defined your CrewAI Agents
- [x] Deployed the CrewAI FastAPI service
- [x] Installed and configured Masumi Payment Service
- [ ] **Next Step**: For production deployments, replace the in-memory store with a persistent database.


## **Useful Resources**

- [CrewAI Documentation](https://docs.crewai.com)
- [Masumi Documentation](https://docs.masumi.network)
- [FastAPI](https://fastapi.tiangolo.com)
- [Cardano Testnet Faucet](https://docs.cardano.org/cardano-testnets/tools/faucet)
