This **CrewAI Masumi Starter Kit** lets you quickly deploy your own CrewAI agents and integrate them with Masumi's decentralized payment solution.

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

<Tabs items={[ 'macOS/Linux', 'Windows']}>
<Tab>
```bash
uv venv --python 3.13  
source .venv/bin/activate  
uv pip install -r requirements.txt
```
</Tab>
<Tab>
```bash
uv venv --python 3.13  
.\.venv\Scripts\activate  
uv pip install -r requirements.txt
```
</Tab>
</Tabs>

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

### **3. Define and Test Your CrewAI Agents**

Take a look at the `crew_definition.py` file. It has a basic `ResearchCrew`. Here you can define your agent functionality.

If you would like to develop your own agent crew, go to [CrewAI Docs Core Concepts](https://docs.crewai.com/en/concepts/agents) to learn more.

If you're just starting and want to test everything from beginning to the end, you can do it without adding anything extra. 

#### Running Your Agents:

The application supports two modes:

**1. Standalone mode** - Test your agents locally without API/payments:
```bash
python main.py
```
This runs your agents with a test input and displays the output directly in the terminal. Perfect for development and testing.

**2. API mode** - Run with full Masumi payment integration:
```bash
python main.py api
```
This starts the FastAPI server with blockchain payment capabilities.

---

###  **4. API Mode with Masumi Integration**

When running in API mode (`python main.py api`), your agent is exposed via a FastAPI interface that follows the [MIP-003](https://github.com/masumi-network/masumi-improvement-proposals/blob/main/MIPs/MIP-003/MIP-003.md) standard for Masumi-compatible services.

Access the interactive API documentation at:
http://localhost:8000/docs

#### Available Endpoints:

- `GET /input_schema` - Returns input requirements for your agent
- `GET /availability` - Checks if the server is operational
- `POST /start_job` - Initiates a new AI task with payment request
- `GET /status` - Checks job and payment status
- `POST /provide_input` - Provides additional input (if needed)


<Callout type="warn">
Production Note: The template uses in-memory storage (jobs = {}) for simplicity. 
In production, implement proper database storage (e.g., PostgreSQL) and consider 
message queues for background processing.
</Callout>

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


2. Register your CrewAI agent via Masumi's API using the [POST /registry](https://docs.masumi.network/api-reference/payment-service/post-registry) endpoint.

   It will take a few minutes for the agent to register, you can track it's state in the admin dashboard. 

3. Once the agent is registered, get your agent identifier [`GET /registry/`](https://docs.masumi.network/api-reference/payment-service/get-registry).

   Copy your `agentIdentifier` from the response, then update it in your `.env` file along with your `PAYMENT_API_KEY`.

   Create a PAYMENT_API key using [`GET /api-key/`](https://docs.masumi.network/api-reference/registry-service/get-api-key).

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
