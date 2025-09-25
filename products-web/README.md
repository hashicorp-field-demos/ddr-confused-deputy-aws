# Products Web - Streamlit Application

A modern Streamlit web application that provides a secure interface for OAuth authentication integrated with an intelligent ProductsAgent chat system. This application serves as the frontend for interacting with AI-powered product management capabilities while demonstrating end-to-end authentication flows in a zero-trust architecture.

## Supported Authentication Providers

This application supports multiple OAuth2/OpenID Connect providers:

- **Microsoft Entra ID** (Azure AD) - Default provider
- **Keycloak** - Open-source identity and access management
- **Custom OAuth2/OIDC** - Any compliant OAuth2 provider

📖 **For Keycloak setup instructions, see [KEYCLOAK.md](./KEYCLOAK.md)**

## Prerequisites

- Python 3.12 or higher
- [uv](https://github.com/astral-sh/uv) package manager
- Docker and Docker Compose (for containerized deployment)
- Authentication provider:
  - **Microsoft Entra ID**: tenant with configured application registration, OR
  - **Keycloak**: server with configured client, OR  
  - **Custom OAuth2**: provider with client credentials

## Local Development Setup

### 1. Install uv Package Manager

```bash
# Install uv (if not already installed)
curl -LsSf https://astral.sh/uv/install.sh | sh

# Verify installation
uv --version
```

### 2. Setup Development Environment

```bash
# Navigate to the products-web directory
cd products-web

# Create virtual environment using uv
uv venv

# Activate the virtual environment
source .venv/bin/activate  # On Unix/macOS
# or .venv\Scripts\activate on Windows

# Install dependencies using uv
uv pip install -r requirements.txt

# Alternative: Install from pyproject.toml
uv pip install -e .
```

### 3. Authentication Provider Configuration

Choose one of the supported authentication providers and create the appropriate configuration:

#### Option A: Microsoft Entra ID (Default)

Create a `.env` file with your Microsoft Entra ID configuration:

```env
# Microsoft Entra ID OAuth Configuration
CLIENT_ID=your_application_client_id_here
CLIENT_SECRET=your_client_secret_here
TENANT_ID=your_directory_tenant_id_here

# OAuth Settings
SCOPE=openid profile email  api://<ad_domain>.onmicrosoft.com/products-agent/Agent.Invoke
REDIRECT_URI=http://localhost:8501/oauth2callback
BASE_URL=https://login.microsoftonline.com

# Authentication Provider (optional, defaults to entra_id)
AUTH_PROVIDER=entra_id

# ProductsAgent API Configuration
PRODUCTS_AGENT_URL=http://localhost:8001
```

#### Option B: Keycloak

Create a `.env` file with your Keycloak configuration:

```env
# Keycloak Client Configuration
CLIENT_ID=products-web-client
CLIENT_SECRET=your_keycloak_client_secret_here

# Keycloak Server Configuration
KEYCLOAK_SERVER_URL=https://keycloak.yourdomain.com
KEYCLOAK_REALM=your-realm-name

# OAuth Settings
SCOPE=openid profile email
REDIRECT_URI=http://localhost:8501/oauth2callback

# Authentication Provider
AUTH_PROVIDER=keycloak

# ProductsAgent API Configuration  
PRODUCTS_AGENT_URL=http://localhost:8001
```

📖 **For detailed Keycloak setup instructions, see [KEYCLOAK.md](./KEYCLOAK.md)**

#### Option C: Custom OAuth2 Provider

For other OAuth2/OIDC compliant providers, manually specify the endpoints:

```env
# Custom OAuth2 Provider Configuration
CLIENT_ID=your_client_id
CLIENT_SECRET=your_client_secret

# Manual URL Configuration
AUTHORIZE_URL=https://your-provider.com/oauth2/authorize
TOKEN_URL=https://your-provider.com/oauth2/token
REFRESH_TOKEN_URL=https://your-provider.com/oauth2/token
REVOKE_TOKEN_URL=https://your-provider.com/oauth2/revoke

# OAuth Settings
SCOPE=openid profile email
REDIRECT_URI=http://localhost:8501/oauth2callback

# Authentication Provider
AUTH_PROVIDER=custom

# ProductsAgent API Configuration
PRODUCTS_AGENT_URL=http://localhost:8001
```

### 4. Run the Application

```bash
# Ensure virtual environment is activated
source .venv/bin/activate

# Start the Streamlit application
uv run streamlit run app.py

# Alternative: Direct execution
streamlit run app.py
```

The application will be available at `http://localhost:8501`

## Docker Deployment

### Build and Run with Docker

This project includes a multi-stage Dockerfile optimized for both development and production use. Use the provided `docker-build.sh` script for streamlined container management.

#### Production Builds

**Build for current architecture:**
```bash
# Build production image for your current platform
./docker-build.sh

# Build for specific architecture
./docker-build.sh amd64   # For Intel/AMD processors  
./docker-build.sh arm64   # For Apple Silicon (M1/M2)
```

**Multi-architecture build:**
```bash
# Build for both AMD64 and ARM64 using Docker buildx
./docker-build.sh multi
```

#### Running Containers

```bash
# Run with default settings (requires .env file)
./docker-build.sh run

# Run specific architecture build
./docker-build.sh run amd64
./docker-build.sh run arm64
```

#### Manual Docker Commands

If you prefer manual Docker commands over the build script:

```bash
# Build production image
docker build --target production -t products-web:latest .

# Build development image  
docker build --target development -t products-web:dev .

# Run production container
docker run -p 8501:8501 --env-file .env products-web:latest

# Run development container with volume mounting
docker run -p 8501:8501 --env-file .env -v $(pwd):/app products-web:dev
```

#### Registry Deployment

```bash
# Build and push to container registry
./docker-build.sh push your-registry.com/products-web

# This builds multi-architecture images and pushes to the specified registry
```

## API Integration

### ProductsAgent Chat

The application integrates with the ProductsAgent API to provide intelligent product management capabilities:

**Supported Operations:**
- List all products in the catalog
- Search for products by name or attributes
- Create new products with natural language
- Update existing product information
- Delete products by ID or name
- Sort and filter products by various criteria

**Example Queries:**
- "Show me all products"
- "Find laptops under $1000"
- "Create a new MacBook Pro priced at $2499"
- "Update product XYZ to cost $1999"
- "Delete the product called 'Old Laptop'"

### Authentication Flow

1. **Login**: User clicks "Login with Microsoft" button
2. **OAuth Redirect**: User is redirected to Microsoft Entra ID for authentication
3. **Token Exchange**: Application receives authorization code and exchanges for tokens
4. **Token Storage**: JWT tokens are securely stored in Streamlit session state
5. **API Calls**: Authenticated requests are made to ProductsAgent API with Bearer token
6. **Chat Interface**: User can interact with AI agent through natural language queries

## Usage Examples

### Basic Chat Interaction

```python
# Example of how the chat interface processes queries
user_input = "List all products sorted by price"
response = chat_with_agent(user_input, jwt_token)
# Returns structured response from ProductsAgent API
```


## Integration with Other Components

This application integrates with the broader confused-deputy-aws architecture:

- **products-agent**: FastAPI backend providing AI agent capabilities
- **products-mcp**: Model Context Protocol server for database operations  
- **Infrastructure**: AWS services, HCP Vault, and Microsoft Entra ID

The web application serves as the user-facing interface that orchestrates secure communication between these components while providing an intuitive chat experience for product management tasks.
