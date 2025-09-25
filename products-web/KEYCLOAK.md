# Keycloak Setup Guide for Products Web App

This guide explains how to configure the products-web application to use Keycloak for authentication instead of Microsoft Entra ID.

## Overview

The products-web application supports both Microsoft Entra ID and Keycloak as OAuth2 authentication providers. This flexibility allows you to integrate with your existing identity management infrastructure.

## Prerequisites

- Keycloak server (version 15+ recommended)
- Administrative access to your Keycloak realm
- Products-web application source code
- Python 3.12+ with required dependencies

## Keycloak Configuration

### 1. Create a New Client

1. Log into your Keycloak Admin Console
2. Navigate to your realm (or create a new one)
3. Go to **Clients** → **Create**
4. Configure the client:

```yaml
Client ID: products-web-client
Client Protocol: openid-connect
Root URL: http://localhost:8501
```

### 2. Client Settings

Configure the following settings for your client:

**Access Type:** `confidential`

**Standard Flow:** `Enabled` (Authorization Code Flow)

**Valid Redirect URIs:**
```
http://localhost:8501/oauth2callback
https://your-domain.com/oauth2callback  # For production
```

**Web Origins:**
```
http://localhost:8501
https://your-domain.com  # For production
```

**Advanced Settings:**
- Proof Key for Code Exchange Code Challenge Method: `S256` (recommended)

### 3. Get Client Credentials

1. Go to the **Credentials** tab of your client
2. Copy the **Secret** value - you'll need this for `CLIENT_SECRET`

### 4. Configure User Attributes (Optional)

If you want to include additional user information in JWT tokens:

1. Go to **Client Scopes**
2. Add desired scopes like `profile`, `email`
3. Configure mappers for additional claims if needed

## Application Configuration

### 1. Environment Variables

Create a `.env` file based on `.env.keycloak.example`:

```env
# Keycloak Client Configuration
CLIENT_ID=products-web-client
CLIENT_SECRET=your_actual_client_secret_here

# Keycloak Server Configuration  
KEYCLOAK_SERVER_URL=https://keycloak.yourdomain.com
KEYCLOAK_REALM=your-realm-name

# OAuth Scopes
SCOPE=openid profile email

# Redirect URI
REDIRECT_URI=http://localhost:8501/oauth2callback

# Authentication Provider
AUTH_PROVIDER=keycloak

# ProductsAgent API Configuration
PRODUCTS_AGENT_URL=http://localhost:8000

# Logging
LOG_LEVEL=INFO
```

### 2. Alternative: Manual URL Configuration

For advanced users who want to specify URLs manually:

```env
# Manual URL configuration (instead of auto-construction)
AUTH_PROVIDER=keycloak
AUTHORIZE_URL=https://keycloak.yourdomain.com/realms/your-realm/protocol/openid-connect/auth
TOKEN_URL=https://keycloak.yourdomain.com/realms/your-realm/protocol/openid-connect/token
REFRESH_TOKEN_URL=https://keycloak.yourdomain.com/realms/your-realm/protocol/openid-connect/token
REVOKE_TOKEN_URL=https://keycloak.yourdomain.com/realms/your-realm/protocol/openid-connect/revoke
```

## Running the Application

### 1. Install Dependencies

```bash
cd products-web
python3 -m venv .venv
source .venv/bin/activate
pip install -r requirements.txt
```

### 2. Set Environment Variables

```bash
# Copy and modify the Keycloak example
cp .env.keycloak.example .env

# Edit .env with your actual Keycloak configuration
nano .env
```

### 3. Start the Application

```bash
streamlit run app.py
```

### 4. Test Authentication

1. Navigate to `http://localhost:8501`
2. Click "Login with Keycloak"
3. You'll be redirected to your Keycloak login page
4. After successful authentication, you'll be returned to the application

## Troubleshooting

### Common Issues

#### 1. "Invalid redirect URI"
- Ensure the redirect URI in your `.env` exactly matches what's configured in Keycloak
- Check for trailing slashes and protocol mismatches

#### 2. "Client not found"
- Verify `CLIENT_ID` matches the client ID in Keycloak
- Ensure you're using the correct realm

#### 3. "Invalid client credentials"
- Double-check the `CLIENT_SECRET` value
- Ensure the client is set to "confidential" access type

#### 4. "Invalid scope"
- Verify the scopes in your `.env` are configured in Keycloak
- Common scopes: `openid`, `profile`, `email`

### Debugging Tips

#### Enable Debug Logging
```env
LOG_LEVEL=DEBUG
```

#### Check Configuration Display
The application shows configuration details in the "Configuration Details" expander on the authentication page.

#### Network Debugging
Use browser developer tools to inspect:
- OAuth redirect flows
- Token responses
- API calls to Keycloak endpoints

## Advanced Configuration

### Custom Claims and Scopes

To include custom user attributes in tokens:

1. **Create Custom Client Scope** in Keycloak
2. **Add Protocol Mappers** for your attributes
3. **Assign Scope** to your client
4. **Update SCOPE** environment variable

Example for custom user attributes:
```env
SCOPE=openid profile email roles groups custom-attributes
```

### SSL/TLS Configuration

For production deployments:

1. **Use HTTPS URLs** in all configuration
2. **Configure SSL certificates** in Keycloak
3. **Update redirect URIs** to use HTTPS
4. **Set secure cookie options** if needed

### Multi-Realm Support

To support multiple Keycloak realms:

1. Set up separate `.env` files per realm
2. Use different client IDs for each realm
3. Consider using realm-specific redirect URIs

## Security Considerations

### Production Checklist

- [ ] Use HTTPS for all URLs
- [ ] Set strong client secrets
- [ ] Configure appropriate token lifetimes
- [ ] Enable PKCE (Proof Key for Code Exchange)
- [ ] Review and minimize scope permissions
- [ ] Set up proper CORS policies
- [ ] Configure session timeout appropriately
- [ ] Enable audit logging in Keycloak

### Token Security

- Tokens are stored in Streamlit session state (server-side)
- JWT validation occurs on every API call
- Automatic token refresh is supported
- Logout clears all session data

## Comparison with Entra ID

| Feature | Keycloak | Microsoft Entra ID |
|---------|----------|-------------------|
| **Cost** | Free (self-hosted) | Paid per user |
| **Deployment** | Self-managed | Cloud-managed |
| **Customization** | Highly customizable | Limited customization |
| **Enterprise Features** | Full-featured | Advanced enterprise features |
| **Setup Complexity** | Medium | Low |
| **Token Standards** | Full OAuth2/OIDC | Full OAuth2/OIDC |

## Support and Resources

### Keycloak Documentation
- [Keycloak Documentation](https://www.keycloak.org/documentation)
- [OAuth2/OIDC Configuration](https://www.keycloak.org/docs/latest/server_admin/index.html#_client_credentials)

### Application Resources
- Check the main README.md for overall architecture
- See QUICKSTART.md for quick setup instructions
- Review products-agent README for backend configuration

### Getting Help

If you encounter issues:

1. Check the application logs (`LOG_LEVEL=DEBUG`)
2. Verify Keycloak server logs
3. Review network traffic in browser dev tools
4. Consult Keycloak community forums
5. Check the OAuth2 specification for protocol details