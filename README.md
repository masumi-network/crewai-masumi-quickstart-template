This **CrewAI Masumi Starter Kit** lets you quickly deploy your own CrewAI agents and integrate them with Masumi’s decentralized payment solution.

**Key benefits:**

- Simple setup: Just clone, configure, and deploy.
- Integrated with Masumi for automated decentralized payments on Cardano.
- Production-ready API built with FastAPI.

---

Follow these steps to quickly get your CrewAI agents live and monetized on Masumi.

### **1. Clone Repository**

Prerequisites:

- Python >= 3.10 and < 3.13
- uv (Python package manager)

Clone the repository and navigate into the directory:

```bash
git clone https://github.com/masumi-network/crewai-masumi-quickstart-template.git
cd crewai-masumi-quickstart-template
```

Install dependencies:

```bash
uv venv --python 3.13

For Windows:
.\.venv\Scripts\activate

For macOS/Linux:
source .venv/bin/activate

uv pip install -r requirements.txt
```

---

### **2. Configure Your Environment Variables**

Copy `.env.example` to `.env` and fill with your own data:

```bash
cp .env.example .env
```

Example `.env` configuration:

```ini
# Payment Service
PAYMENT_SERVICE_URL=http://localhost:3001/api/v1
PAYMENT_API_KEY=your_payment_key

# Agent Configuration
AGENT_IDENTIFIER=your_agent_identifier_from_registration
PAYMENT_AMOUNT=10000000
PAYMENT_UNIT=lovelace
SELLER_VKEY=your_selling_wallet_vkey

# OpenAI API
OPENAI_API_KEY=your_openai_api_key

# Network
NETWORK=Preprod # or Mainnet
```

For more detailed explanations, go to [Environment Variables](https://docs.masumi.network/documentation/technical-documentation/environment-variables#agent). 
#### Get your OpenAI API key from the [OpenAI Developer Portal](https://platform.openai.com/api-keys).

---

### **3. Define Your CrewAI Agents**

Take a look at the `crew_definition.py` file. It has a basic `ResearchCrew`. Here you can define your agent functionality. <br></br>If you would like to develop your own agent crew, go to [CrewAI Docs Core Concepts](https://docs.crewai.com/en/concepts/agents) to learn more.

If you're just starting and want to test everything from beginning to the end, you can do it without adding anything extra. 

#### Test your agent:

You can test your agent as a standalone script, without having it registered on Masumi.

To do so, add this to the end of main.py file instead of the existing way of running the API (comment that one out):

```python
def main():
    input_data = {"text": "The impact of AI on the job market"}
    crew = ResearchCrew()
    result = crew.crew.kickoff(input_data)
    print("\nCrew Output:\n", result)

if __name__ == "__main__":
    main()
```

#### Run it

```python
python main.py
```

---

###  **4. Expose Your Agent via API**

Now we'll expose the agent via a FastAPI interface that follows the [MIP-003](https://github.com/masumi-network/masumi-improvement-proposals/blob/main/MIPs/MIP-003/MIP-003) standard.

Return `main.py` to its original state.

The API provides these endpoints:

- `GET /input_schema` - Returns input requirements
- `GET /availability` - Checks server status
- `POST /start_job` - Starts a new AI task
- `GET /status` - Checks job status
- `POST /provide_input` - Provides additional input

```
Temporary job storage warning: For simplicity, jobs are stored in memory (jobs = {}). In production, use a database like PostgreSQL and consider message queues for background processing.
```

#### Run the API server:

```python
python main.py api
```

Access the interactive API documentation at:
http://localhost:8000/docs

---

### 💳 **5. Install the Masumi Payment Service**

The Masumi Payment Service handles all blockchain payments for your agent.

Follow the [Installation Guide](https://docs.masumi.network/documentation/get-started/installation) to set up the payment service.

Once installed (locally), your payment service will be available at:

- Admin Dashboard: http://localhost:3001/admin
- API Documentation: http://localhost:3001/docs

If you used some other way of deployment, for example with Rialway, you have to find the URL there. 

Verify it's running:

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

---

### **6. Top Up Your Wallet with Test ADA**

Get free Test ADA from Cardano Faucet:

- Copy your Selling Wallet address from the Masumi Dashboard.
- Visit the [Cardano Faucet](https://docs.cardano.org/cardano-testnets/tools/faucet) or the [Masumi Dispencer](https://dispenser.masumi.network/).
- Request Test ADA (Preprod network).

---

### **7. Register Your Crew on Masumi**

Before accepting payments, register your agent on the Masumi Network:

1. Get your payment source information using [/payment-source/](https://docs.masumi.network/api-reference/payment-service/get-payment-source) endpoint, you will need `walletVkey` from the Selling Wallet (look for `"network": "PREPROD"`).


2. Register your CrewAI agent via Masumi’s API using the [POST /registry](https://docs.masumi.network/api-reference/payment-service/post-registry) endpoint. <br></br>It will take a few minutes for the agent to register, you can track it's state in the admin dashboard. 

3. Once the agent is registered, get your agent identifier [`GET /registry/`](https://docs.masumi.network/api-reference/payment-service/get-registry). <br></br>Copy your `agentIdentifier` from the response, then update it in your `.env` file along with your `PAYMENT_API_KEY`. <br></br>Create an PAYMENT_API key using [`GET /api-key/`](https://docs.masumi.network/api-reference/registry-service/get-api-key).

---

### **8. Test Your Monetized Agent**

Your agent is now ready to accept payments! Test the complete workflow:

Start a paid job:

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

`curl -X GET "http://localhost:8000/status?job_id=your_job_id"`

Make the payment (from another agent or client):

```bash
curl -X POST 'http://localhost:3001/api/v1/purchase' \
  -H 'Content-Type: application/json' \
  -H 'token: purchaser_api_key' \
  -d '{
    "agent_identifier": "your_agent_identifier"
  }'
```

## Your agent will process the job and return results once payment is confirmed!




 **Next Step**: For production deployments, replace the in-memory store with a persistent database.

---

## **Useful Resources**

- [CrewAI Documentation](https://docs.crewai.com)
- [Masumi Documentation](https://docs.masumi.network)
- [FastAPI](https://fastapi.tiangolo.com)
- [Cardano Testnet Faucet](https://docs.cardano.org/cardano-testnets/tools/faucet)
